# CoinVarsity Academy architecture

## Status and purpose

This is the baseline architecture for the implementation phase. It deliberately
defines boundaries and contracts without selecting a hidden managed backend,
inventing credentials, creating schema migrations, or implementing product features.
CoinVarsity is an educational and practice platform. It provides no financial advice,
execution of real-money orders, or promise of returns.

## Architecture goals

- Retain the continuous learning loop: learn, assess, practice, simulate, journal,
  analyze, detect mistakes, mentor, and target the next learning activity.
- Make the journal the durable feedback layer rather than an isolated trade log.
- Support Android, iOS, Windows, and Linux from one Flutter/Dart client. Web is out
  of scope.
- Keep content, AI, and market-data providers replaceable, and preserve privacy and
  auditability for sensitive user records.

## Client architecture

Use a feature-first Flutter workspace with a layered implementation inside every
feature. The exact state-management package is intentionally deferred, but the
selected option must support testable immutable state, dependency injection through
constructors, cancellation/loading/error states, and scoped observation. A Riverpod
implementation is the recommended decision because it meets those criteria without
putting service locators in widgets; its adoption needs explicit approval and a
dependency record first.

```text
lib/
  app/                 composition root, routing, theme, bootstrap
  core/                errors, result types, identifiers, time, telemetry, security
  features/
    academy/           presentation | application | domain | data
    assessment/        presentation | application | domain | data
    practice/          presentation | application | domain | data
    journal/           presentation | application | domain | data
    performance/       presentation | application | domain | data
    mentor/            presentation | application | domain | data
    personalization/   presentation | application | domain | data
    profile/           presentation | application | domain | data
  infrastructure/      authenticated API client, local store, secure store, platform adapters
```

Domain code contains entities, value objects, policies, and repository ports; data
implements those ports with API/local sources; application coordinates use cases and
state; presentation renders state and dispatches intents. Feature-to-feature access
uses domain/application contracts, not another feature's data source or widgets.

## Domain boundaries and ownership

| Boundary | Owns | Key outputs to the loop |
| --- | --- | --- |
| Identity and profile | external identity link, profile, roles, preferences | user context and consent |
| Academy | versioned courses, modules, lessons, concepts, paths, prerequisites | content and concept coverage |
| Assessment | reusable items, attempts, scores, explanations, mastery evidence | concept-level evidence |
| Practice and simulation | scenarios, portfolios, orders, positions, calculations, challenges | simulated execution evidence |
| Journal | journal entries, trade rationale, plan snapshots, observations, lessons | central durable practice memory |
| Performance and rules | metrics, plan/risk rule evaluation, mistake detections | patterns and weakness signals |
| Personalization | recommendations and learning-plan state | ranked next activities |
| Mentor/AI | educational conversations, analyses, generated drafts with provenance | explanatory, non-advisory assistance |
| Market data | normalized assets, prices, candles, provider observations | read-only input to simulation/learning |

Use stable opaque IDs, UTC timestamps, money/quantity decimal values (never binary
floating point for persisted financial calculations), explicit currency/asset units,
and an immutable `content_version` or rule/algorithm version on derived records.

## Backend and API boundary

Build a modular backend (initially a modular monolith) with independently testable
modules matching the boundaries above. It exposes a versioned HTTPS JSON API, e.g.
`/v1/academy`, `/v1/assessments`, `/v1/simulations`, `/v1/journal`,
`/v1/performance`, `/v1/recommendations`, and `/v1/mentor`. Use OpenAPI as the
contract source; generated client code is permitted only at the transport boundary.

The backend is authoritative for identity-linked data, scoring, risk calculations,
simulation state, journal records, performance results, recommendations, and audit
events. Validate authorization and business rules server-side. Use asynchronous jobs
for market-data ingestion, performance recomputation, recommendation refreshes, AI
requests, content publishing, notifications, and reconciliation. Jobs must be
idempotent, retry safely, and carry correlation IDs.

Production hosting and backend language/framework are intentionally not selected in
this phase. Choose them only after an ADR covering operations, team capability,
regional/privacy needs, cost, background work, and observability. Do not use
Supabase.

## Relational persistence model

Use PostgreSQL (or a production-equivalent managed PostgreSQL service) as the
relational system of record. Object storage holds lesson media and generated assets;
the database stores metadata and access control. A cache/message queue may be added
for rate limits, short-lived market reads, and jobs, but neither replaces the system
of record.

Core tables should include:

- `users`, `external_identities`, `roles`, `user_roles`, `profiles`, `consents`, and
  `sessions`/token metadata as appropriate to the chosen identity provider.
- `courses`, `course_versions`, `modules`, `lessons`, `content_blocks`, `concepts`,
  `content_concepts`, `learning_paths`, `path_items`, and `prerequisites`.
- `assessment_definitions`, `assessment_versions`, `assessment_items`,
  `assessment_options`, `assessment_attempts`, `attempt_responses`, and
  `mastery_evidence`.
- `practice_scenarios`, `simulated_portfolios`, `simulated_orders`,
  `simulated_fills`, `simulated_positions`, `simulation_snapshots`, and `challenges`.
- `trading_plans`, `plan_rules`, `journal_entries`, `journal_trade_links`,
  `journal_observations`, `journal_lessons`, and attachment metadata.
- `performance_snapshots`, `rule_evaluations`, `mistake_detections`,
  `weakness_signals`, `recommendations`, and `recommendation_items`.
- `assets`, `markets`, `market_instruments`, `market_prices`, `candles`,
  `market_provider_observations`, and provider ingestion metadata.
- `mentor_conversations`, `mentor_messages`, `ai_runs`, `ai_artifacts`,
  `prompt_versions`, and `audit_events`.

Foreign keys connect evidence to `user_id`, concepts, content versions, journal
entries, scenarios, and recommendation reasons. Keep original user-entered journal
facts immutable or revisioned; store analyses as versioned derived records so a new
algorithm does not overwrite history.

## Authentication and authorization

Google sign-in is required, using OAuth 2.0/OIDC with PKCE on native clients. The
client receives only short-lived tokens and stores refresh/session material in each
platform's secure credential storage. It exchanges or validates tokens with the
backend; the backend maps the verified external subject to a separate internal user
and profile. Authorization is policy/role based on the backend and defaults to
owner-only access for journals, conversations, portfolios, and records. Add provider
adapters so Apple, enterprise, or other identity providers can be introduced later.

## Provider abstractions

Define ports in domain/application code and implementations only in infrastructure:

- `MarketDataProvider`: asset metadata, latest prices, candles, historical ranges,
  and optional stream subscriptions. Normalize provider symbols, timestamps, units,
  provenance, and staleness before consumers see them.
- `AiProvider`: request/response, streaming if needed, model metadata, safety
  controls, and usage accounting. The backend owns credentials, prompt templates,
  retrieval, redaction, and output persistence. Every response carries an
  educational/non-advice framing and provenance.
- `IdentityProvider`: verifies identity assertions and exposes provider-neutral
  subject and claims.

No provider SDK, secret, API key, or provider-specific model leaks into Flutter
features or core domain models.

## Simulation, journal, analysis, and personalization

Simulation is an educational ledger: scenario/configuration plus deterministic order,
fill, position, cash, P&L, and risk calculations. It consumes normalized market data
or a frozen historical scenario and explicitly marks data source, latency, and
calculation version. It never routes orders to an exchange.

The journal links to simulated trades but can also record a manual practice entry. It
captures asset, market, timeframe, setup, thesis, entry/stop/target, position size,
risk, expected reward and risk/reward, execution, outcome, mistakes, emotional or
behavioral observations, lessons, and timestamps. A journal entry can reference the
user's plan and preserve the plan/rule snapshot that existed at the time.

The performance module derives metrics and evaluates the snapshot against explicit
best-practice/risk rules. It writes explainable detections with evidence, severity,
confidence, and rule version, then aggregates them into concept-tagged weakness
signals. Personalization combines mastery, prerequisites, practice evidence,
weaknesses, and user goals to create explainable recommendations linking directly to
lessons, exercises, assessments, or scenarios. AI Mentor consumes authorized
read-models of these facts; it cannot silently mutate them or make financial advice.

## Offline and synchronization

Offline-first is scoped, not assumed. Cache published content, downloaded media,
user progress drafts, and in-progress assessment/journal drafts locally. Keep market
quotes, live streams, scoring authority, simulation ledger finalization,
recommendations, and AI interactions online/server-authoritative. Queue eligible
mutations with idempotency keys and a client timestamp; synchronize when connected.
Use per-entity revisions and expose conflicts rather than overwriting an edited
journal entry. Encrypt local sensitive caches where supported and allow users to
clear them.

## Security, privacy, and observability

Use TLS, secure native storage, server-side secret management, authenticated API
authorization, schema/input validation, rate limits, least-privilege service roles,
encryption at rest, backups, and structured audit events. Redact sensitive journal
content from logs and AI prompts unless consent and a legitimate feature need exist.
Record access to sensitive records, AI-provider calls, administrative changes, and
rule/content publication. Instrument client/server errors, latency, job outcomes,
market-data freshness, and recommendation/AI safety events using correlation IDs;
never put sensitive free text in telemetry.

## Testing and deployment

Test domain policies and calculations with deterministic unit tests; test application
use cases with fake ports; test API contract/authorization and persistence integration
against PostgreSQL; and test key native client journeys with Flutter integration tests
on supported platforms. Contract-test market and AI adapters. Include migration,
backup-restore, rate-limit, and security tests when those systems are introduced.

CI should format, analyze, test, build Android/iOS/Windows/Linux artifacts as the
relevant runners become available, scan dependencies/secrets, and publish signed
artifacts from protected release workflows. Deploy backend changes through reviewed,
versioned migrations and observable canary/rollback procedures. Secrets belong in the
deployment secret manager, never source control or the client.

## Decisions and approvals needed

1. Approve the backend language/framework, hosting, region/data-residency posture,
   and managed PostgreSQL operator.
2. Approve the Flutter state-management dependency (recommended: Riverpod) and
   local persistence/encryption approach after dependency review.
3. Select Google OIDC implementation and session model, including whether a managed
   identity broker is acceptable (Supabase is not).
4. Select market-data vendors, entitlement policy, snapshot/realtime behavior, and
   historical-data retention.
5. Select AI provider(s), content/RAG safety policy, retention/redaction controls,
   human review for generated tutorials, and usage-cost limits.
6. Define the initial trading-plan rule catalogue, simulation-market assumptions,
   journal retention/deletion policy, and supported locales/currencies.
