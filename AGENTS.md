# AGENTS.md

Canonical AI collaboration rules for this repo.

## Source of truth checks

- `AGENTS.md` is canonical; `.github/copilot-instructions.md` and `.github/instructions/*.instructions.md` must stay aligned.
- `CLAUDE.md` must be a symlink to `AGENTS.md` (enforced by `python scripts/check_ai_assets.py`).
- Required AI assets are enforced in CI (`ai-governance` job): `.github/instructions/{backend,client,governance}.instructions.md` and `.claude/skills/**`.
- If you edit AI governance files, run: `python scripts/check_ai_assets.py`.

## Repo map (only major boundaries)

- Python runtime entrypoints: `main.py` (CLI/scheduler) and `server.py` (FastAPI app export for uvicorn).
- Backend implementation: `src/`, `data_provider/`, `api/`, `bot/`.
- Web client: `apps/dsa-web` (Vite + React + TypeScript).
- Desktop client: `apps/dsa-desktop` (Electron; bundles backend artifact from `dist/backend/stock_analysis`).
- CI/workflows: `.github/workflows/`; local automation scripts: `scripts/`.

## Commands that match CI

- Backend CI-equivalent gate: `./scripts/ci_gate.sh`.
- Phase-specific backend checks:
  - `./scripts/ci_gate.sh syntax`
  - `./scripts/ci_gate.sh flake8`
  - `./scripts/ci_gate.sh deterministic`
  - `./scripts/ci_gate.sh offline-tests`
- Offline pytest suite used by CI: `python -m pytest -m "not network"`.
- Network tests are separate and non-blocking in CI: `python -m pytest -m network -q` and `./test.sh quick --no-notify`.
- Frontend gate (runs only when `apps/dsa-web/**` changes): `cd apps/dsa-web && npm ci && npm run lint && npm run build`.

## Focused local verification

- Single pytest file/test: `python -m pytest path/to/test_file.py -k "test_name"`.
- Syntax-only quick check for edited Python files: `python -m py_compile <files>`.
- `pytest` markers are defined in `setup.cfg`: `unit`, `integration`, `network`.

## Desktop build order and constraints

- Desktop packaging depends on backend build output at `dist/backend/stock_analysis`; do not run desktop build first.
- One-command desktop builds:
  - Windows: `powershell -ExecutionPolicy Bypass -File scripts/build-all.ps1`
  - macOS: `bash scripts/build-all-macos.sh`
- Backend desktop build scripts build web static assets first (`apps/dsa-web npm run build`) before PyInstaller.
- Windows local desktop builds require Developer Mode unless `DSA_SKIP_DEVMODE_CHECK=true`.

## Release/workflow quirks

- Auto-tag is opt-in: only commits to `main` with `#patch`, `#minor`, or `#major` in the commit message trigger `.github/workflows/auto-tag.yml`.
- `create-release.yml` triggers on pushed `v*.*.*` tags and uses tag annotation text as the release body fallback source.
- Desktop release workflow also triggers on `v*.*.*` tags and reads changelog content from `docs/CHANGELOG.md`.

## Change guardrails specific to this codebase

- Keep fail-open behavior for optional providers/channels unless task explicitly requires fail-fast (core pipeline is designed to continue on partial provider failure).
- If behavior/config changes affect users, update docs and `docs/CHANGELOG.md` in the same change.
- When updating only one language variant of docs, note why the counterpart was not synchronized.
