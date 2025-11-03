# Repository Guidelines

## Project Structure & Module Organization
Root manifests (`Dockerfile`, `Dockerfile.alpine`, `Caddyfile_global`, `Caddyfile_per_site`) stay at the top level for quick builds and config swaps. GitHub Actions definitions in `.github/workflows/` drive automated image builds and release checks. Utility scripts belong in `scripts/`, currently housing `check_caddy_status.py`; add new helpers there to keep the layout predictable.

## Build, Test, and Development Commands
Use `docker build -t ghcr.io/caddybuilds/caddy-cloudflare:dev -f Dockerfile .` for the standard image and swap in `Dockerfile.alpine` when targeting Alpine. Smoke-test locally with `docker run --rm -p 80:80 -p 443:443 -v $PWD/Caddyfile_global:/etc/caddy/Caddyfile ghcr.io/caddybuilds/caddy-cloudflare:dev` and run `caddy version` inside. For release automation, execute `python3 scripts/check_caddy_status.py` to confirm the upstream matrix; workflows mirror these steps.

## Coding Style & Naming Conventions
Keep Dockerfiles to one instruction per logical step with uppercase directives. Caddyfiles rely on two-space alignment; mirror `Caddyfile_global` and use explicit blocks for each site. Python utilities should use 4-space indentation, f-strings, and short docstrings for public functions. Name scripts in lowercase with underscores and document required environment variables near the top.

## Testing Guidelines
There is no dedicated unit test suite; rely on container-level validation. Build both Docker variants and invoke `caddy version` inside each container to confirm the bundled binary. For Python helpers, run `python3 -m compileall scripts` to catch syntax errors and add lightweight asserts near entry points. New GitHub Actions should expose a dry-run path or a commented `if: github.event_name == 'workflow_dispatch'` guard.

## Commit & Pull Request Guidelines
Follow conventional commits (`feat:`, `fix:`, `docs:`) as seen in the log, and keep subject lines under 72 characters. Bundle related Docker, workflow, and config changes, but open separate PRs for unrelated updates. Each PR should describe the image variant(s) touched, capture manual validation commands, and link issues or discussions. Provide logs or screenshots when altering workflows so reviewers can verify the CI effect. 

## Security & Configuration Tips
Never commit API tokens or `.env` files; reference them via `CLOUDFLARE_API_TOKEN` and other secrets configured in GitHub Actions. When introducing new credentials, document them in `README.md` and add masked secrets to workflow `env:` blocks rather than inline literals. Review Docker layers for sensitive artifacts, and prefer `--platform` flags over bundling architecture-specific binaries to maintain multi-arch support.
