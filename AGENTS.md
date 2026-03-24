# Repository Guidelines

## Project Structure & Module Organization
This repository is infrastructure-as-code for NixOS deployments, not an application codebase.

- `.github/workflows/deploy.yml`: main deployment workflow (manual `workflow_dispatch`, matrix deploy to Vultr hosts).
- `.github/workflows/claude.yml`: Claude Code automation for issue/PR interactions.
- `README.md`: operator-facing setup and deployment usage.
- `CLAUDE.md`: agent context and architecture notes.

Keep deployment logic in `deploy.yml` and keep operational documentation in `README.md`.

## Build, Test, and Development Commands
There is no local build artifact; changes are validated by workflow runs.

- `gh workflow run deploy.yml -f target=vultr-02`: deploy one host.
- `gh workflow run deploy.yml -f target=all`: deploy all hosts in parallel.
- `gh workflow run deploy.yml -f target=vultr-03 -f build_on_target=true`: remote build on target.
- `gh run watch`: stream workflow progress.
- `gh run list --workflow deploy.yml`: check recent deployment history.

## Coding Style & Naming Conventions
- Use 2-space indentation in YAML files.
- Keep workflow step names short, imperative, and specific (for example, `Verify deployment`).
- Use uppercase snake case for env and secret names (`SSH_PRIVATE_KEY`, `NIX_CONFIG_REPO`).
- Follow existing host naming: `vultr-01` to `vultr-04`; keep flake target names aligned (`.#vultr-02`).
- Update docs in the same PR whenever behavior or required secrets change.

## Testing Guidelines
No unit test framework exists in this repository. Validate by running the workflow safely.

- Prefer single-host validation before `target=all`.
- Confirm the `Verify deployment` step succeeds (`nixos-version && uptime`).
- For workflow-only edits, inspect job logs for SSH setup, clone, and `nixos-rebuild` output.

## Commit & Pull Request Guidelines
Commit style in history follows concise typed subjects (`feat:`, `docs:`, `ci:`, `Fix:`). Prefer:

- `<type>: <brief imperative summary>` (example: `ci: add vultr-01 to deploy matrix`).

PRs should include:

- What changed and why.
- Deployment impact (which hosts/environments are affected).
- Any secret/config prerequisites.
- Link to a successful GitHub Actions run (or reason it was not run).

## Security & Configuration Tips
- Never commit private keys; use GitHub Actions secrets only.
- Respect `.gitignore` secret patterns (`*.pem`, `*.key`, `id_*`).
- When adding a host, update workflow input options, matrix logic, and setup docs together.
