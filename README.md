# CoinVarsity Academy

CoinVarsity Academy is a native education and practice platform for Bitcoin,
cryptocurrency, financial markets, trading, risk management, and security. It helps
learners build critical thinking and realistic expectations; it is not a
financial-advice service, profit-guarantee product, or real-money trading platform.

## Product direction

The product is built around a continuous feedback loop:

**Learn → Test → Practice → Simulate → Journal → Analyze → Detect mistakes → AI
Mentor → Targeted learning → Repeat**

The Trading Journal is the central memory layer in that loop. It connects simulated
and practical work with structured rationale, risk, execution, outcome, observations,
and lessons. Performance analysis evaluates that evidence against a user's plan and
best-practice rules. Detected weaknesses feed directly into relevant lessons,
assessments, exercises, practice scenarios, and educational AI mentoring.

## Supported clients

- Android
- iOS
- Windows
- Linux

Flutter and Dart are the primary client technologies. A Web target is intentionally
out of scope.

## Core systems

- **Academy:** versioned courses, modules, lessons, concepts, prerequisites, paths,
  and progress—not hard-coded lesson screens.
- **Assessment:** reusable quizzes, scenarios, practical questions, attempts,
  explanations, scoring, and mastery evidence.
- **Practice and simulation:** paper portfolios, scenarios, orders, positions, P&L,
  risk calculations, and challenges. No real-money order routing is permitted.
- **Journal and performance:** structured journal entries, trading-plan snapshots,
  rule evaluation, explainable mistake detection, and performance metrics.
- **Personalization and AI Mentor:** concept-tagged recommendations and
  provider-agnostic educational assistance. AI output is never financial advice.
- **Market data:** a provider-neutral boundary for prices, candles, asset metadata,
  historical data, and future streaming.

## Architecture principles

- Flutter features separate presentation, application/state, domain, data, and
  infrastructure responsibilities.
- A modular backend and relational database own authoritative user records,
  calculations, scoring, journal data, and audit history. Supabase is not used.
- Google authentication is required, while identity remains separate from profiles and
  business logic. Secrets and privileged provider credentials never enter the client.
- Content, AI, identity, and market-data integrations use replaceable contracts.
- Offline support is intentional: published content and drafts can be cached, while
  live prices, AI, authoritative scoring, and finalized simulation records require
  synchronization.
- Security, privacy, authorization, validation, auditability, and observability are
  first-class requirements.

## Developer guidance

Read [the architecture baseline](docs/architecture.md) before introducing a feature
or cross-cutting dependency, and follow the repository's [permanent engineering
instructions](AGENTS.md). Important technology selections—backend, hosting, database
operator, market-data provider, AI provider, and Flutter state-management package—are
deliberately pending documented approval rather than being silently chosen.

## Current repository status

This repository currently contains architecture and project-governance documentation
only. Product implementation, backend services, database migrations, AI integrations,
market-data integrations, and Flutter client scaffolding have not yet been added.
