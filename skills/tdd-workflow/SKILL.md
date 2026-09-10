---
name: tdd-workflow
description: Test-driven bug fixes and reproductions. Use when the user requests TDD, asks to begin with a reproducer, or a bugfix explicitly starts by adding a failing test.
---

# TDD workflow

- Inspect the relevant contract and existing tests, then add a focused behavior test that reproduces the requested
  failure
- Run it and verify the expected failure before implementing
- Commit the red test separately only when the repository workflow or user request calls for intermediate commits; a
  failing test is evidence, not a reason to leave a shared or unvalidated tree committed
- Implement the general fix, then rerun the same focused test and relevant adjacent tests
- Cover boundaries, negative paths, relevant state combinations, and adjacent regressions without bloating fixtures
