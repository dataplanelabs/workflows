# dataplanelabs/workflows

Org-wide reusable GitHub Actions workflows. Central home for shared CI —
successor to `dataplanelabs/.github` (the rename will happen later).

This repo is **public** so that both public and private org repos can call its
reusable workflows. It contains only workflow YAML, templates, and docs — **no
secrets**.

## Code Review (OpenCodeReview)

AI code review on pull requests, powered by
[OpenCodeReview](https://github.com/alibaba/open-code-review) (OCR). Posts inline
review comments on PRs. Triggered on PR open/reopen, or on demand by commenting
`/open-code-review` (or `@open-code-review`) on a PR.

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
   jobs:
     review:
       if: >
         github.event_name == 'pull_request_target' ||
         (github.event_name == 'issue_comment' && github.event.issue.pull_request != null &&
           (startsWith(github.event.comment.body, '/open-code-review') ||
            startsWith(github.event.comment.body, '@open-code-review')))
       uses: dataplanelabs/workflows/.github/workflows/code-review.yml@main
       secrets: inherit
   ```

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

`GITHUB_TOKEN` is provided automatically (`pull-requests: write`).

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

| Repo | rule.json |
|------|-----------|
| annhien | tailored (Python / Bruin / FastAPI) |
| infra | tailored (Terraform / Flux / Ansible / Vault) |
| goclaw | tailored (Go + UI) |
| gcplane | tailored (Go CLI) |
| dpl-web | tailored (Astro / TS) |
| goclaw-config | minimal — refine via tracked issue |
| scry | minimal — refine via tracked issue |

## Security note

The review job uses `pull_request_target`, so org secrets are available even on
fork PRs. This is safe here because OCR only reads the diff — it does not execute
PR code. Do not add steps that run untrusted PR scripts in this job.
