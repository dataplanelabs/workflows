# dataplanelabs/workflows

Org-wide reusable GitHub Actions workflows. Central home for shared CI —
successor to `dataplanelabs/.github` (the rename will happen later).

This repo is **public** so that both public and private org repos can call its
reusable workflows. It contains only workflow YAML, templates, and docs — **no
secrets**.

## Organization repositories

All repos under [dataplanelabs](https://github.com/orgs/dataplanelabs/repositories)
— 22 total (9 public, 13 private). The **Code review** column shows whether a repo
has adopted the [code-review workflow](#code-review-opencodereview) below:
`tailored` = project-specific `rule.json`, `minimal` = starter rules, `—` = not
yet adopted.

| Repo | Vis | Code review | Description |
|------|-----|-------------|-------------|
| [.github](https://github.com/dataplanelabs/.github) | private | — | Legacy org reusable workflows — superseded by this repo |
| [annhien](https://github.com/dataplanelabs/annhien) | private | tailored (Python / Bruin / FastAPI) | — |
| [design](https://github.com/dataplanelabs/design) | private | — | Brand identity: logo, color tokens, typography, usage guidelines |
| [dpl-web](https://github.com/dataplanelabs/dpl-web) | private | tailored (Astro / TS) | — |
| [excalidraw](https://github.com/dataplanelabs/excalidraw) | public | — | Self-hosted Excalidraw with MCP control (fork of mcp-excalidraw-local) |
| [gcplane](https://github.com/dataplanelabs/gcplane) | public | tailored (Go CLI) | Declarative GitOps control plane for GoClaw (YAML manifests) |
| [goclaw](https://github.com/dataplanelabs/goclaw) | public | tailored (Go + UI) | Multi-agent AI gateway — single Go binary, 11+ LLM providers, 5 channels |
| [goclaw-charts](https://github.com/dataplanelabs/goclaw-charts) | public | — | Helm charts for GoClaw deployment |
| [goclaw-config](https://github.com/dataplanelabs/goclaw-config) | private | minimal | GCPlane GitOps config for GoClaw multi-tenant deployment |
| [gws-cli](https://github.com/dataplanelabs/gws-cli) | public | — | Read-only Google Workspace CLI for goclaw skill integration |
| [infra](https://github.com/dataplanelabs/infra) | private | tailored (Terraform / Flux / Ansible / Vault) | IaC for dataplanelabs.com — K3S cluster with FluxCD GitOps |
| [infra-template](https://github.com/dataplanelabs/infra-template) | private | — | Infrastructure template repository for DataPlaneLabs projects |
| [miu-bot](https://github.com/dataplanelabs/miu-bot) | public | — | Durable AI assistant framework — nanobot fork w/ Postgres + Temporal |
| [opencode](https://github.com/dataplanelabs/opencode) | private | — | — |
| [runnerclubs](https://github.com/dataplanelabs/runnerclubs) | private | — | Platform for runner clubs — members, races, results, PRs, achievements |
| [scry](https://github.com/dataplanelabs/scry) | public | minimal | Persistent, logged-in, agent-drivable browser (Chromium + CDP + noVNC) |
| [skills](https://github.com/dataplanelabs/skills) | private | — | — |
| [ssh-agent](https://github.com/dataplanelabs/ssh-agent) | public | — | GitHub Action to set up `ssh-agent` with a private key |
| [telegram-bot](https://github.com/dataplanelabs/telegram-bot) | private | — | Telegram–GitHub bot for issue creation and AlertManager alerts |
| [website](https://github.com/dataplanelabs/website) | private | — | Marketing website — Next.js + shadcn/ui |
| [workflows](https://github.com/dataplanelabs/workflows) | public | n/a | Org-wide reusable workflows — **this repo** |
| [zalo-cli](https://github.com/dataplanelabs/zalo-cli) | private | — | — |

`minimal` repos (goclaw-config, scry) carry starter rules — refine via a tracked issue.

## Code Review (OpenCodeReview)

AI code review on pull requests, powered by
[OpenCodeReview](https://github.com/alibaba/open-code-review) (OCR). Posts inline
review comments on PRs. Triggered on PR open/reopen, or on demand by commenting
`/ocr` (or `@ocr`) on a PR.

**Bot-authored PRs are skipped** on the auto-trigger (release-please, dependabot,
etc. — no value reviewing version bumps). Comment `/ocr` to force a review on one.

### Adopt it in a repo

1. Add the caller workflow — copy [`templates/code-review.caller.yml`](templates/code-review.caller.yml)
   to `.github/workflows/code-review.yml`:

   ```yaml
   name: Code Review
   on:
     pull_request_target:
       types: [opened, reopened]
     issue_comment:
       types: [created]
   permissions:
     contents: read
     pull-requests: write
   jobs:
     review:
       uses: dataplanelabs/workflows/.github/workflows/code-review.yml@main
       secrets:
         OCR_LLM_URL: ${{ secrets.OCR_LLM_URL }}
         OCR_LLM_AUTH_TOKEN: ${{ secrets.OCR_LLM_AUTH_TOKEN }}
         OCR_LLM_MODEL: ${{ secrets.OCR_LLM_MODEL }}
         OCR_LLM_USE_ANTHROPIC: ${{ secrets.OCR_LLM_USE_ANTHROPIC }}
         GH_APP_ID: ${{ secrets.GH_APP_ID }}
         GH_APP_MUNMIU_PRIVATE_KEY: ${{ secrets.GH_APP_MUNMIU_PRIVATE_KEY }}
   ```

   The trigger keyword (`/ocr`, `@ocr`), the keyword guard, and concurrency all
   live in the reusable workflow — to change them, edit this repo only. Three
   things must stay in each caller:
   - `on:` triggers — GitHub requires them in the caller.
   - `permissions: pull-requests: write` — a reusable workflow cannot exceed the
     caller's token, and the org default is read-only.
   - **explicit `secrets:`** — `secrets: inherit` does **not** pass secrets from a
     private repo into a public reusable workflow, nor across owner boundaries
     (personal account → org). Passing each secret explicitly works everywhere.

2. Add project rules (optional) — drop a `rule.json` at
   `.opencodereview/rule.json`. It is **auto-loaded** by OCR; no flag needed. See
   [`templates/rule.json`](templates/rule.json) for the format.

### Secrets

The reusable workflow reads these via `secrets: inherit`. They are set as
**organization secrets** (visibility: all repos):

| Secret | Required | Notes |
|--------|----------|-------|
| `OCR_LLM_URL` | yes | LLM endpoint (Anthropic-compatible) |
| `OCR_LLM_AUTH_TOKEN` | yes | API token |
| `OCR_LLM_MODEL` | no | default `glm-5.2` |
| `OCR_LLM_USE_ANTHROPIC` | no | default `true` |
| `GH_APP_ID` + `GH_APP_MUNMIU_PRIVATE_KEY` | no | post comments as `munmiu[bot]` (falls back to `github-actions[bot]`) |

`GITHUB_TOKEN` is provided automatically (`pull-requests: write`).

### Reuse from another account/org

The reusable workflow is public, so any repo can call it with **its own** secrets —
no fork/copy needed. Set `OCR_LLM_*` (and optionally `GH_APP_*`) on the calling
repo/org. The secrets always resolve in the **caller's** context, never this repo's.

**Always pass secrets explicitly** (see the caller snippet above). `secrets: inherit`
only works when the caller is a public repo with the same owner as this one — it
silently passes nothing when the caller is private (private → public reusable) or
under a different owner (personal account / another org). Explicit passing works
in every case, so use it everywhere.

### Review rules

OCR resolves rules with a first-match-wins priority chain:

1. `--rule` flag (not used here)
2. `<repo>/.opencodereview/rule.json` — **per-project, committed to git**
3. `~/.opencodereview/rule.json` — user global
4. Built-in `system_rules.json`

Rule file format:

```json
{
  "rules": [
    { "path": "**/*.go", "rule": "Wrap errors with %w; handle every returned error." }
  ],
  "include": ["src/**"],
  "exclude": ["**/vendor/**", "**/*.lock"]
}
```

- `path` globs: `**` recursive, `*` single-segment, `{a,b}` brace expansion, case-insensitive.
- Rules are first-match-wins in declaration order — put specific paths first.
- `exclude` always wins over `include`; `include` bypasses built-in default excludes (e.g. test files).

### Rollout status

See the [Organization repositories](#organization-repositories) table above — the
**Code review** column tracks per-repo adoption (`tailored` / `minimal` / `—`).

## Security note

The review job uses `pull_request_target`, so org secrets are available even on
fork PRs. This is safe here because OCR only reads the diff — it does not execute
PR code. Do not add steps that run untrusted PR scripts in this job.
