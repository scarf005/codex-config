# Localization

## Keyed translations

- Use semantic named arguments for every interpolated keyed translation. Pass values with `Named("argumentName")` and use the exact matching `{argumentName}` placeholder in every language file; do not use positional placeholders such as `{0}`. For example, use `Good: {name}` with `value.Named("name")`, never `Bad: {0}`.
- Keep placeholder names stable across translations and verify every affected `Translate(...)` call against its keyed XML entry.

## Translation validation

Run the structure validator from the mod root:

```bash
deno run --allow-read ~/.config/codex/skills/rimworld/scripts/validate-mod.ts
```

It catches `defInjected` lowercase, missing `About.xml`, and unrecognized `Languages/` subdirectories.

## Translations (DefInjected)

- Every new mod must include English and Korean translations. Add player-facing keyed strings in both `Languages/English/Keyed/` and `Languages/Korean (한국어)/Keyed/`; add both DefInjected translations when the mod introduces translatable defs.
- Folder: `Languages/[langFolderName]/DefInjected/[DefType]/` — `DefInjected` must use this exact case (capital D, capital I). RimWorld's `DefInjectionsFolderName` constant is `"DefInjected"` and the match is case-sensitive on Linux.
  `langFolderName` must match `Prefs.xml` `<langFolderName>` EXACTLY, including parentheses. Korean: `Languages/Korean (한국어)/DefInjected/`.
- Do not create the folder as `defInjected` — the lowercase variant will be silently ignored.
- Subdirectories named by Def type with trailing 's' (`DamageDefs`, `HediffDefs`, `ThingDefs`). RimWorld strips trailing 's' if exact type match fails, then retries.
- For custom Def types in custom namespaces, use the def type's full name as the folder (e.g. `CombatExtended.AmmoCategoryDef/`). Standard Verse defs use bare type names.
- Translation files are simple key-value XML; `LoadFolders.xml` does not need a Languages entry.

### Key format — path resolution

The translation key is a dot-separated path that RimWorld navigates through the def's object graph. Three patterns exist:

1. **Direct def fields** — fields on the def class itself (e.g. `label`, `labelNoun`, `description`, `prettyLabel`):
   `<defName.fieldName>text</defName.fieldName>`
   e.g. `<CEOP_Barotrauma.description>폭발 충격파로 인한 내부 장기 손상입니다.</CEOP_Barotrauma.description>`

2. **Nested object fields** — fields on sub-objects referenced directly by the def (e.g. `injuryProps.destroyedLabel`, `injuryProps.destroyedOutLabel`):
   `<defName.parentField.childField>text</defName.parentField.childField>`
   e.g. `<CEOP_Barotrauma.injuryProps.destroyedLabel>파열됨</CEOP_Barotrauma.injuryProps.destroyedLabel>`

3. **Comp properties** — fields inside list elements in the `comps` list, accessed via `[TranslationHandle]`:
   `<defName.comps.handleFieldName.propertyName>text</defName.comps.handleFieldName.propertyName>`
   e.g. `<CEOP_Barotrauma.comps.HediffComp_GetsPermanent.permanentLabel>만성 압력손상</CEOP_Barotrauma.comps.HediffComp_GetsPermanent.permanentLabel>`

   **Crucially, the handle is determined by `[TranslationHandle]` on `HediffCompProperties.compClass` (a `Type` field), so the handle value is the runtime comp class name** (e.g. `HediffComp_GetsPermanent`, `HediffComp_TendDuration`), **not** the properties class name (`HediffCompProperties_GetsPermanent`).

   Common comp properties and their correct paths:

   | Property | Properties class | Runtime comp (handle) | Full path |
   |---|---|---|---|
   | `permanentLabel` | `HediffCompProperties_GetsPermanent` | `HediffComp_GetsPermanent` | `{defName}.comps.HediffComp_GetsPermanent.permanentLabel` |
   | `instantlyPermanentLabel` | `HediffCompProperties_GetsPermanent` | `HediffComp_GetsPermanent` | `{defName}.comps.HediffComp_GetsPermanent.instantlyPermanentLabel` |
   | `labelTendedWell` | `HediffCompProperties_TendDuration` | `HediffComp_TendDuration` | `{defName}.comps.HediffComp_TendDuration.labelTendedWell` |
   | `labelTendedWellInner` | `HediffCompProperties_TendDuration` | `HediffComp_TendDuration` | `{defName}.comps.HediffComp_TendDuration.labelTendedWellInner` |
   | `labelSolidTendedWell` | `HediffCompProperties_TendDuration` | `HediffComp_TendDuration` | `{defName}.comps.HediffComp_TendDuration.labelSolidTendedWell` |

   **Wrong**: `<CEOP_Barotrauma.permanentLabel>만성 압력손상</CEOP_Barotrauma.permanentLabel>` — RimWorld cannot find `permanentLabel` on the def class itself and silently discards the translation (logs an error).
   **Correct**: `<CEOP_Barotrauma.comps.HediffComp_GetsPermanent.permanentLabel>만성 압력손상</CEOP_Barotrauma.comps.HediffComp_GetsPermanent.permanentLabel>`

   To verify correct paths, run the game's language report or inspect the `DefInjectionUtility.ForEachPossibleDefInjectionInDefRecursive` traversal — it enumerates every translatable field with its suggested key path.

### Diagnostic: Checking translation key validity

If a translation does not appear in-game despite a seemingly correct XML entry:

- Check RimWorld's `Player.log` for `Couldn't inject` errors — these indicate path resolution failures.
- Verify the key format path: the game can navigate through to the exact field. Comp properties must use the `comps.CompClassName.propertyName` format because they live inside list elements, not on the def class.
- Run the RimWorld language report generator or compare against the output of `DefInjectionUtility.ForEachPossibleDefInjection` to list all valid translation keys for a def.
