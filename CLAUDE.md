# CLAUDE.md

Guidance for Claude Code (and any AI agent) working in this repository.

## What this project is

**TrustLayer** — an explainable, real-time risk engine for UPI transactions. It scores every transaction against behavioral signals and turns India's mandatory (post-April 2026) second verification step from a blind, flat prompt into a transparent, risk-proportional decision that always tells the user *why*.

Built for the **Paytm Build for India** hackathon under the **AI-Powered Fintech Innovation** theme. Extends prior work (**Risk Radar** from *Asli Meesho*).

Read `README.md` for the full product narrative before making product-shaping changes.

## Core principles — do not break these

1. **Explainability is the product, not a feature.** Every risk score MUST carry human-readable reasons. Never return a score without the `reasons[]` that produced it. A black-box output is a regression here.
2. **Reasons are user-facing plain language.** "New payee, amount 3x above your typical send" — not "feature_7 exceeded threshold 0.82". Write reasons a non-technical user would understand.
3. **Three decision bands, always.** `low → approve`, `medium → step-up`, `high → hold`. Keep the router's mapping explicit and testable. Don't collapse or add bands without updating docs + tests.
4. **The scoring seam stays clean.** Scoring logic lives behind one interface so rule weights can later be swapped for an ML model with no change to the router or API. Don't leak scoring internals into the router or handlers.
5. **Deterministic demo.** The seeded dataset must keep producing the canonical demo: one normal txn auto-approves, one risky txn is held with a specific explanation. Don't break these two paths — they are the pitch.

## Architecture

```
Transaction → Signal Extraction → Risk Scoring Engine → Decision Router → Response
                                   → Explainability Layer ↗
```

- **Signal Extraction** — derive: payee novelty, amount deviation, timing/device anomaly, payee reputation.
- **Risk Scoring Engine** — combine signals into a 0–100 score.
- **Explainability Layer** — turn contributing signals into plain-language reasons.
- **Decision Router** — map score → band → action.

Keep these as separable modules. A change to how a signal is computed should not require touching the router.

## Tech stack

- Python 3.11+, FastAPI, Pydantic
- Seeded transaction data (no external DB required for the demo)

## Conventions

- Type-hint everything; validate all request/response bodies with Pydantic models.
- New signals: add to signal extraction + give each a plain-language reason template. A signal with no reason template is incomplete.
- Keep endpoints thin — orchestration only. Logic lives in the engine/router/explainability modules.
- Match the existing code's naming and comment density; don't over-comment obvious code.

## Working agreements for agents

- **Don't invent a scoring model.** The current engine is rule-weighted by design (auditable for a fintech trust use case). ML is a roadmap item behind the existing seam — don't swap it in unprompted.
- **Preserve the two canonical demo transactions.** If a change alters their outcome, that's a red flag — call it out.
- **Run/verify before claiming done.** Start the app (`uvicorn app.main:app --reload`), hit `/score` with the low- and high-risk payloads from the README, confirm the bands and reasons. Report actual output, not assumptions.
- No secrets, no real user data in the repo. Seeded/synthetic data only.
- This is a solo hackathon project — favor clear, demoable, well-explained code over premature scale/abstraction.

## Commands

```bash
# setup
python -m venv .venv && source .venv/bin/activate    # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# run
uvicorn app.main:app --reload        # docs at http://localhost:8000/docs

# test (once tests exist)
pytest -q
```
