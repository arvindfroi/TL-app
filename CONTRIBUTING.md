# Contributing

Welcome to the TL app (WAD?FC). This is a small, friends-only project — keep it simple and have fun.

**Read [`AGENTS.md`](AGENTS.md) first** — it's the full guide (also written so you can paste it into, or point your AI assistant at, this repo). This file is the quick human version.

## Getting oriented
- The plan: [`docs/MVP-and-Roadmap.md`](docs/MVP-and-Roadmap.md)
- The architecture: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)
- Current progress: [`STATUS.md`](STATUS.md) (auto-updated)
- What to work on: the [Issues](../../issues), filtered by **phase** label

## Workflow
1. Pick an open issue from the lowest open phase and **assign yourself**.
2. Branch from `main` (e.g. `phase-1/group-chat`).
3. Commit in present tense and reference the issue: `Add poll results view (#11)`.
4. Open a PR into `main` and link the issue (`Closes #11`). Keep PRs small.

## Labels
- **Phase:** `phase-0: foundations` → `phase-4: money` (+ `decision`)
- **Area:** `backend`, `ui`, `ios-native`, `ai`, `infra`

## Ground rules
- iOS-only; TestFlight-only distribution for now.
- Never commit secrets (keys, provisioning profiles) — see `.gitignore`.
- Betting feature: no money held/routed by the app (points-only first).
- Don't hand-edit `STATUS.md`.

## Seeing who did what
GitHub already tracks this: **Insights → Contributors** and **Insights → Pulse** show commits, PRs, and issue activity per person. Use issue **assignees** to show ownership.
