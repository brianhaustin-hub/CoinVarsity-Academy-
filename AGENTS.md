# CoinVarsity Academy engineering instructions

## Product and scope

CoinVarsity Academy is a native, multi-platform education and practice product for
Bitcoin, cryptocurrency, financial markets, trading, risk management, and security.
It is **not** a demo, a web application, a social-media clone, a financial-advice
service, or a profit-guarantee service. Preserve the learning loop:

`LEARN → TEST → PRACTICE → SIMULATE → JOURNAL → ANALYZE → DETECT MISTAKES → AI MENTOR → TARGETED LEARNING → REPEAT`

The trading journal is the central memory and feedback system. Changes to academy,
assessment, simulation, journal, analysis, or AI work must preserve their explicit
relationships; do not implement them as disconnected features.

## Technology and architecture constraints

- The official clients are Flutter/Dart for Android, iOS, Windows, and Linux. Do
  not create, enable, or target a Web client, and isolate platform-specific code.
- Do **not** use Supabase for any concern, including authentication or persistence.
- Do not implement real-money trading. Simulation and paper trading are educational
  only.
- Keep Flutter presentation, application/state, domain, data, and infrastructure
  concerns separate. Widgets must not own database, authentication, AI, or
  market-data implementation.
- Use modular features and stable domain contracts. Read
  `docs/architecture.md` before changing cross-cutting architecture.
- Academy content must be structured and versionable; do not encode lessons as
  one-off screens. Reusable assessments, provider-agnostic market data, and
  replaceable AI providers are required boundaries.
- Keep identity separate from profiles and business data. Never put API keys,
  secrets, or privileged market/AI credentials in the Flutter client.
- Treat AI output solely as educational assistance, not financial advice. Preserve
  auditability and privacy for journal and AI-interaction data.
- Add dependencies only with a documented responsibility, replacement path, and
  lock-in assessment. Do not silently replace agreed technology choices.

## Delivery discipline

- Inspect the current repository and existing instructions before changing code.
- Keep changes focused; do not remove established requirements or unrelated work.
- Before committing: run formatting, static analysis, relevant tests, inspect
  `git status`, and review the changed files. Report failures honestly.
- Never force-push, rewrite history, or push unless Brian explicitly authorizes it.
- Do not use speculative debugging loops. Read an error, make the smallest justified
  fix, and rerun the affected verification.
- For runnable UI changes, capture a screenshot when practical. Do not fabricate
  implementation status or verification results.
