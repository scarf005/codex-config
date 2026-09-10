---
name: github-workflow
description: GitHub issue, pull request, branch, worktree, commit metadata, checks, review follow-up, and external distribution workflow. Use for GitHub contribution work or external distribution artifacts.
---

# GitHub workflow

- Use `gh`; do not comment unless requested
- For registry, package-index, or external distribution work, keep vendor content out of the main application tree
  unless explicitly requested.
- For external reports about bundled components, identify whether the fix belongs upstream or in the host repository
  before branching or opening a PR
- Branch from the repository's actual target branch, using the matching `upstream` remote when available and otherwise
  `origin`; name branches
  `{type}/{issue-number}/{issue-slug}` or `{type}/{slug}`
- For requested new worktrees, use the installed `git-gtr` through `git gtr`; run `git gtr version` before considering
  installation or cloning another copy
- Inspect the specific PR/commit file list and diff before claiming whether it changed a file
- Read repository PR templates, title/changelog guidelines, and existing comparable PRs before drafting
- Before editing an existing issue or PR, fetch its current title and body with `gh`; patch a saved copy minimally and
  inspect a body-only diff that preserves human prose, artifacts, screenshots, generated blocks, checklist state, and
  links
- Before creating or updating a PR body, run task-specific issue/PR searches and add direct related links when found;
  include known source issues/PRs/branches/commits even when search misses them
- Put `close #NUMBER` before full URL references when the PR resolves an issue; close applicable open subissues
  explicitly
- Keep PR titles and bodies terse and concrete; omit optional filler and CI-redundant testing prose
- When a template asks for testing or reviewer guidance, include concrete steps and expected results
- For web UI PRs, upload `playwright-cli` screenshots to a GitHub gist and link raw gist URLs; do not commit media
  unless requested
- For external reports, link the report and rerun timed-out validation with sufficient timeout
- Treat `CHANGES_REQUESTED`, `still broken`, `does not work`, and follow-up screenshots as a new bug report: inspect the
  latest review/comment and artifact, reproduce the exact symptom, and do not rely on build/tests/CI alone
- Watch Cataclysm-BN checks at 10-minute intervals unless inspecting a failure; do not watch checks after metadata-only
  edits unless requested
- For CI permission/auth failures, compare the same step on recent comparable PRs before calling it PR-specific
- After follow-up implementation changes, replace stale cumulative PR bullets and update relevant
  compatibility/testing/checklist details
- Reuse authorization already granted for the current issue or PR; ask before a remote operation that expands its scope
  or changes the requested target.
- After opening a PR, push follow-up commits to the current PR branch when already authorized; require explicit
  authorization for force-push or a different remote or branch
