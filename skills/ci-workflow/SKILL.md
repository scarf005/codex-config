---
name: ci-workflow
description: CI/CD workflow rules. Use when creating or modifying GitHub Actions, scheduled workflows, cron jobs, build/test/release automation, or Deno setup in CI.
---

# CI Workflow

- Match existing workflow style, permissions, triggers, package manager, local scripts, and release/platform event class
  before adding patterns
- Treat repository presets, package-manager scripts, and CI jobs as the contract; invoke them directly, use
  fresh/reconfigure options for stale state, and validate observable outputs instead of duplicating cache
  variables/options
- When CI fails after an intentional behavior change, verify whether expectations or fixtures are stale before changing
  intended implementation behavior
- Run the matching local workflow when feasible and state exact unverified event/platform/release paths
- When using `denoland/setup-deno@v2`, use `with: { cache: true }`.
- If a scheduled GitHub Actions cron job would otherwise be disabled by repository inactivity, add
  `gautamkrishnar/keepalive-workflow@v2`; do not add it to schedules that do not need that keepalive behavior.
- Prefer keepalive API mode in a separate job with `permissions: { actions: write }`, `actions/checkout@v4`, and no
  dummy commits.
- Use `workflow_files` when a dedicated keepalive workflow maintains other scheduled workflow files.
