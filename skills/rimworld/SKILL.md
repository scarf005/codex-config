---
name: rimworld
description: RimWorld mod XML definitions, patches, load order, C# and F# conventions, keyed localization, dependency setup, formatting, and build validation. Use whenever modifying or reviewing a RimWorld mod.
---

# RimWorld

## RimWorld 1.6 reference source

- Use the local RimWorld 1.6 decompiled reference clone at `/home/scarf/repo/rw/RimworldDecompile` to verify native APIs, implementation details, and efficient vanilla call patterns before guessing
- Reuse this clone instead of creating temporary duplicate clones; treat it as read-only reference material and do not copy decompiled game code into mods
- For performance work, inspect both the relevant native type and vanilla call sites using that API, then prefer cached managers, grids, region listers, and direct lists over broad enumerable scans

## XML compatibility and load order

- Before changing an XML compatibility definition, inspect the target-version source of every named parent `ThingDef`, required workbench, research, race definition, and patched list
- For `Could not find parent node` errors, inspect `XmlInheritance.GetBestParentFor`, the add-on's `ModsConfig.xml` position, and the provider mod's `About.xml` before changing Defs or `LoadFolders.xml`. A parent from another mod is eligible only when its `loadOrder` is less than or equal to the child mod's load order
- When a compatible parent exists but loads after the add-on, preserve the original conditional `LoadFolders.xml` entries and declare the parent package IDs in `<forceLoadAfter>`. Use `<loadAfter>` only when the author or evidence establishes that enforced ordering causes a conflict
- Do not duplicate existing Def XML into `PatchOperationAdd` as a workaround for a load-order issue. Copy or replace a parent-dependent Def only after target-version source proves that no compatible parent exists
- Treat a user-provided Workshop comment, issue, log, or mod link as primary evidence: inspect it before proposing a substitute mechanism

## Task-specific references

- For C# or F# source changes, read [code conventions](references/code.md).
- For keyed strings, DefInjected translations, or localization validation, read [localization](references/localization.md).
- For DLLs, builds, releases, installation, runtime control, save inspection, local paths, or a mod `justfile`, read [build and runtime](references/build-runtime.md).
- For Combat Extended APIs, explosions, wounds, custom defs, settings, or stat calibration, read [Combat Extended](references/combat-extended.md).
