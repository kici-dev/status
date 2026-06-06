DO NOT BE LAZY

# KiCI status page (Upptime)

This repo IS the public status page at <https://status.kici.dev> — an
[Upptime](https://upptime.js.org) instance: GitHub Actions probes the
production endpoints every ~5 minutes, GitHub Pages serves the static site
from `gh-pages`, GitHub Issues store incidents. The operator runbook
(monitored URLs, incident workflow, secrets, DNS) lives in the main KiCI
repo at `docs/internal/status-page.md`.

## Ground rules

- **Default branch is `main`.**
- **The repo must stay public** — GitHub Pages and unlimited Actions minutes
  both depend on it. Flipping it private silently stops the checks and
  unpublishes the page.
- **Monitors live in `.upptimerc.yml`** — adding/changing one is a
  one-commit change. Do not edit `README.md`, `history/`, or `graphs/` by
  hand; Upptime's workflows own them.

## Action pinning (MANDATORY)

Every `uses:` in `.github/workflows/` MUST be pinned to a full-length commit
SHA with a version comment: `owner/repo@<40-hex-sha> # vX.Y.Z`.

- Enforced server-side: the repo setting **"Require actions to be pinned to
  a full-length commit SHA"** is ON — GitHub refuses to RUN any workflow
  containing an unpinned `uses:`.
- Enforced client-side: the committed `.githooks/pre-commit` runs
  `pinact run` on every commit ([pinact](https://github.com/suzuki-shunsuke/pinact)
  is version-pinned in `mise.toml`). On a fresh clone, enable it with:

  ```bash
  git config core.hooksPath .githooks
  ```

- Never hand-write a SHA; never commit a floating tag (`@v6`, `@master`).

## Upgrading dependencies

```bash
mise exec -- pinact run -u   # bump all actions to latest releases AND pin SHAs
git commit -am "chore: bump <action> to vX.Y.Z"
git push
```

The main repo's weekly `/version-audit` watches `upptime/uptime-monitor`
(the generator) by fetching `.github/workflows/uptime.yml` from `main` and
reading the pinact version comment. Renaming/moving that workflow or
reformatting the pin comment breaks the audit's regex loudly (ERROR row) —
update `hack/lib/remote-pin-sources.ts` in the main repo when restructuring.

## Known hazard: template auto-updates

Upptime's "Update Template CI" (`updates.yml`) rewrites the workflows from
the upstream template and may write floating refs. Because of the
SHA-required setting, workflows then refuse to run. Fix: in a hooked clone,
run `mise exec -- pinact run`, commit, push.

## Secrets (names only — values live in the main repo's sops / GitHub)

- `GH_PAT` — fine-grained PAT, scoped to only this repo (Contents, Issues,
  Workflows, Pages, Actions — all read/write), expiry capped at 366 days by
  org policy. On expiry the workflows fail loudly; mint a new PAT with the
  same scopes and `gh secret set GH_PAT --repo kici-dev/status`.
- `NOTIFICATION_TELEGRAM`, `NOTIFICATION_TELEGRAM_BOT_KEY`,
  `NOTIFICATION_TELEGRAM_CHAT_ID` — operator alerting. Note the `_BOT_KEY`
  name: Upptime silently ignores the `_BOT_TOKEN` spelling.

DO NOT BE LAZY
