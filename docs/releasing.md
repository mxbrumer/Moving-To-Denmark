# Releasing

One SemVer for the whole repo (D7), driven by [release-please](https://github.com/googleapis/release-please) from Conventional Commits.

## Flow
1. Work on a branch, open a PR. Its **title** must be a Conventional Commit (`feat: …`, `fix: …`); the `pr-title` check enforces it. PRs are squash-merged, so the title becomes the commit on `main`.
2. `main` accepts changes only through PRs whose `ci-passed` and `pr-title` checks pass. Direct and force pushes are rejected.
3. On every push to `main`, `release.yml` runs release-please. It opens or updates a **release PR** (`chore(main): release X.Y.Z`) with the version bump and `CHANGELOG.md`.
4. Merge the release PR when you want to ship. release-please then tags `vX.Y.Z` and creates the GitHub Release, and the same workflow:
   - pushes `ghcr.io/<owner>/ajs-backend` and `ajs-frontend` as `:X.Y.Z` and `:latest` (only once their Dockerfiles exist);
   - attaches `langflow-flows-vX.Y.Z.zip` (all `langflow/flows/*.json`).

## Version bumps (pre-1.0)
`feat` → minor (0.1.0 → 0.2.0), `fix`/`perf`/`docs` → patch, `feat!` or `BREAKING CHANGE:` → minor while < 1.0. Only `feat`/`fix`/`perf`/`docs` appear in `CHANGELOG.md`; `chore`, `ci`, `test`, `refactor` and `build` are hidden. Config: `release-please-config.json`. The version is stamped into `.release-please-manifest.json`, `backend/pyproject.toml` and `frontend/package.json` (once they exist).

## Release PR checks
Workflows do not run on PRs opened with the default `GITHUB_TOKEN`, so the release PR shows no checks. Either:
- merge it as a repo admin (the ruleset lets admins bypass checks on PRs only), or
- add a fine-grained PAT (contents + pull-requests: write on this repo) as the secret `RELEASE_PLEASE_TOKEN`; `release.yml` then uses it and CI runs on the release PR.

To force a version, add `Release-As: X.Y.Z` to a commit body.
