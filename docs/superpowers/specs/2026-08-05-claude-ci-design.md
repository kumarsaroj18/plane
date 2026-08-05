# Claude-Integrated CI for the Plane Fork

**Date:** 2026-08-05
**Branch:** `feat/workshop`
**Repo:** `kumarsaroj18/plane` (fork of `makeplane/plane`)
**Status:** Approved, ready for implementation planning

## Goal

Build a GitHub Actions CI pipeline from scratch as a learning exercise, with Claude
integrated as the centerpiece so arbitrary prompts can be run against the project.

The fork already inherits nine upstream workflows. This design does not modify or
delete them. It adds a parallel, self-contained set that is readable end to end.

## Context

- **Monorepo:** pnpm 11 + turbo. Apps: `web`, `admin`, `space`, `live`, `api`, `proxy`.
  Packages: `ui`, `editor`, `types`, `i18n`, and others.
- **Toolchain:** Node >= 22.18.0, Python 3.12 for `apps/api`.
- **Linting:** oxlint via `.oxlintrc.json`; formatting via oxfmt.
- **Existing skills:** `.claude/skills/` ships `branch-name`, `create-pull-request`,
  `react-doctor`, `release-notes`, `translate`.
- **Actions status:** enabled on the fork (`allowed_actions: all`).
- **Auth:** `CLAUDE_CODE_OAUTH_TOKEN` repo secret, already configured. Uses the
  Claude subscription rather than per-token API billing.

### Turbo task graph

Verified in `turbo.json`:

- `check:lint` declares no `dependsOn`. It runs standalone and is cheap.
- `check:types` declares `dependsOn: ["^build"]`. Turbo builds dependency packages
  automatically before type checking.

This drives the CI job split in section 2. It also means no explicit build job is
needed: turbo resolves the build dependency itself.

## Decisions

| Question | Decision |
|---|---|
| Relationship to inherited CI | Build fresh workflows alongside; leave the nine upstream files untouched |
| CI scope | Lean: lint + types on changed code only |
| Auth | `CLAUDE_CODE_OAUTH_TOKEN` (subscription), not an API key |
| Triggers | On-demand only: manual dispatch, `@claude` mention, auto PR review. No cron. |
| Permissions | Write for deliberate triggers (dispatch, mention); comment-only for automatic PR review |
| Prompt library | Versioned `.github/prompts/*.md` files plus a free-text override |
| Layout | One workflow file per trigger |

### Why one file per trigger

The permission split is the deciding factor. `permissions:` is scoped per job, and
`claude-review.yml` must not have `contents: write`: it fires automatically on every
PR, including on code Claude itself just pushed. A single consolidated workflow would
need a permissions block that is the union of all modes, which hands write access to
that automatic path and defeats the guardrail. Separate files express the distinction
structurally rather than by convention.

The cost is roughly fifteen lines of repeated checkout and auth boilerplate per file.
For a workshop artifact that repetition is a feature: each file reads top to bottom
with no indirection. Consolidating into a `workflow_call` reusable workflow is the
natural refactor once the duplication becomes annoying, and is explicitly out of
scope here.

## 1. Architecture

```
.github/workflows/ci.yml                # lint + types
.github/workflows/claude-dispatch.yml   # manual prompt runner   WRITE
.github/workflows/claude-mention.yml    # @claude in comments    WRITE
.github/workflows/claude-review.yml     # automatic PR review    COMMENT-ONLY
.github/prompts/*.md                    # versioned prompt library
```

All Claude workflows pin `anthropics/claude-code-action@v1` and authenticate with
`claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}`.

Every job sets a `concurrency` group with `cancel-in-progress: true` and an explicit
`timeout-minutes`:

| Job | `timeout-minutes` |
|---|---|
| `ci.yml` lint | 10 |
| `ci.yml` types | 20 |
| `claude-dispatch.yml` | 30 |
| `claude-mention.yml` | 30 |
| `claude-review.yml` | 20 |

## 2. `ci.yml`

**Triggers:** `pull_request` targeting `preview`, plus `workflow_dispatch`.

Push events are deliberately excluded. `turbo --affected` needs a base ref to diff
against; `github.event.pull_request.base.sha` provides a correct one, whereas a push
event does not and would require guessing a merge base.

The inherited workflows gate on `github.event.pull_request.requested_reviewers != null`.
That condition is omitted here, since it would prevent the workflow from ever firing
on an ordinary fork PR.

**Jobs (parallel):**

- **lint** — `pnpm turbo run check:lint --affected`. Caches the pnpm store. No build
  dependency. Expected 2-3 minutes.
- **types** — `pnpm turbo run check:types --affected`. Turbo resolves `^build`
  internally, so dependency packages are built as part of the task graph. Caches both
  the pnpm store and `.turbo`. Expected 8-12 minutes cold, substantially less warm.

Both jobs set `TURBO_SCM_BASE` and `TURBO_SCM_HEAD` so `--affected` resolves, use
`actions/checkout@v6` with `fetch-depth: 50` and `filter: blob:none`, and enable pnpm
through corepack.

## 3. Claude workflows

| File | Trigger | Permissions |
|---|---|---|
| `claude-dispatch.yml` | `workflow_dispatch` | `contents: write`, `pull-requests: write`, `issues: write` |
| `claude-mention.yml` | `issue_comment`, `pull_request_review_comment`, `issues` | `contents: write`, `pull-requests: write`, `issues: write` |
| `claude-review.yml` | `pull_request` (opened, synchronize, reopened, ready_for_review) | `contents: read`, `pull-requests: write` |

### `claude-dispatch.yml`

Inputs:

- `prompt_file` — choice, names a file in `.github/prompts/`
- `prompt` — free text; takes precedence when non-empty
- `model` — choice of `sonnet`, `opus`, `haiku`; defaults to `sonnet`
- `scope` — optional path to focus the run, for example `apps/web`

A resolve step selects free text over the file, reads the file when free text is
empty, and writes the result to `$GITHUB_OUTPUT` for the action's `prompt` input.

Known limitation: Actions `choice` inputs are static, so adding a prompt file also
requires adding its name to the dropdown list. The file remains the source of truth;
the dropdown is only a menu. This is accepted rather than worked around, because the
alternatives (dynamic enumeration via a matrix, or free-text-only) either add
indirection or lose reproducibility.

Model is passed through `claude_args` as `--model`, along with `--max-turns 30`.

### `claude-mention.yml`

Pre-gated with `if: contains(github.event.comment.body, '@claude')`. The action
performs its own trigger-phrase detection, but the guard avoids starting a runner for
unrelated comments.

### `claude-review.yml`

Sets `use_sticky_comment: true` so repeated pushes update one comment instead of
accumulating new ones, and `track_progress: true` for visible progress.

`claude_args` restricts the tool set to read-only tools. Comment-only is therefore
enforced at two layers: the workflow `permissions` block and the tool allowlist.

## 4. Prompt library

Plain markdown files, no frontmatter. Initial set:

- `code-review.md`
- `security-audit.md`
- `i18n-drift.md`
- `dependency-audit.md`
- `test-coverage-gaps.md`

Each references the conventions in `AGENTS.md` so output matches house style.

## 5. Fork-specific constraints

**Pull requests originating from other forks do not receive repository secrets**, so
`claude-review.yml` will not run Claude on them. This is acceptable here because the
working pattern is `feat/*` to `preview` within a single fork.

The constraint around scheduled workflows being disabled by default on forks no longer
applies, since nothing runs on a cron. See "Dropped from scope" below.

## 6. Prerequisites

Completed before implementation:

```
claude setup-token
gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo kumarsaroj18/plane
```

Verified present on 2026-08-05 via `gh secret list`.

## 7. Verification

1. `actionlint` across all four workflow files for YAML and expression validity.
2. `gh workflow run claude-dispatch.yml` with a trivial prompt.
3. A throwaway PR into `preview` to exercise `ci.yml` and `claude-review.yml`.
4. A comment containing `@claude` to exercise `claude-mention.yml`.

Each workflow is considered done only when observed green in an actual run, not when
the YAML is merely written.

## Dropped from scope

A scheduled `claude-scheduled.yml` running prompts on a weekly cron was designed and
then cut before implementation. Claude now runs only when explicitly invoked: a
dispatch button, an `@claude` comment, or opening a PR.

Consequences of the cut:

- No `schedule:` trigger exists, so the fork-specific rules about cron workflows being
  disabled by default and auto-disabling after 60 days of inactivity are irrelevant.
- Subscription usage is bounded by deliberate actions rather than a recurring job.
- The `dependency-audit.md`, `i18n-drift.md`, and `test-coverage-gaps.md` prompts stay
  in the library. They were the intended cron payloads and remain useful on demand
  through `claude-dispatch.yml`.

Re-adding a cron later is additive: one new workflow file reusing an existing prompt.

## Out of scope

- Modifying or removing the nine inherited workflows
- Python and pytest coverage in the new CI
- Refactoring to a `workflow_call` reusable workflow
- Turbo remote caching (`remoteCache.enabled` is `false` upstream)
