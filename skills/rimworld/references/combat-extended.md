# Combat Extended

## Wound cause display (health tab tooltip)

- The "원인: ..." tooltip on hediffs in the health tab comes from `hediff.combatLogEntry` / `hediff.combatLogText`, which are set by `DamageResult.AssociateWithLog()` — never by `DamageInfo` alone.
- `DamageInfo.weapon` sets `Hediff_Injury.sourceLabel` / `sourceDef` for the bracketed label (e.g. "타박상 (볼레)"), but the full cause line requires a battle log entry.
- When applying damage outside the standard explosion/projectile pipeline (e.g. custom postfix damage), you must:
  1. Create a `BattleLogEntry_ExplosionImpact(instigator, recipient, weaponDef, projectileDef, damageDef)`.
  2. `Find.BattleLog.Add(battleLogEntry)`.
  3. Capture `pawn.TakeDamage(dinfo)` result and call `damageResult.AssociateWithLog(battleLogEntry)`.
- `BattleLogEntry_ExplosionImpact` handles both Pawn and non-Pawn initiators (falls back to `initiatorThing = initiator.def`).
- `DamageResult.AssociateWithLog` sets `combatLogEntry` (WeakReference) and `combatLogText` (pre-generated string) on every hediff in the result. Without this, the cause tooltip will be empty regardless of `DamageInfo.weapon`.
- `Hediff_MissingPart` (destroyed body parts) is created inside `AddHediff`, not added to `DamageResult.hediffs`. `AssociateWithLog` only iterates `result.hediffs`, so destroyed parts won't get combat log info. After `AssociateWithLog`, check `pawn.health.hediffSet.PartIsMissing(part)` and manually set `combatLogEntry` / `combatLogText` on the matching `Hediff_MissingPart`.
- For CE-specific overpressure/explosion postfix damage: resolve the firing Pawn from the CE `instigator` parameter via `(instigator as ProjectileCE)?.launcher` before creating `BattleLogEntry_ExplosionImpact`, since CE passes the projectile object, not the Pawn.
- Reference: `DamageWorker.ExplosionDamageThing` in the decompiled source shows the canonical pattern.

## CE-specific APIs

- `GenExplosionCE.DoExplosion` is a standalone static method (not an override of `GenExplosion`). Patch with `[<HarmonyPatch(typeof<GenExplosionCE>, nameof GenExplosionCE.DoExplosion)>]`.
- CE explosion armor penetration: `GetExplosionArmorPenetration()` = `damageAmountBase * ExplosiveArmorPenetrationMultiplier` where multiplier = 0.4f. This is blunt penetration in MPa.
- CE explosion damage may be scaled by the firing weapon's `RangedWeapon_DamageMultiplier` stat. Radius comes from XML `explosionRadius` unscaled.
- Thermobaric detection: check `DamageDef.isExplosive && typeof<DamageWorker_Flame>.IsAssignableFrom(damageDef.workerClass)`. CE's `Thermobaric` DamageDef inherits from `Bomb` and uses `DamageWorker_Flame`.
- Howitzer thermobaric shells: projectile uses `PrometheumFlame` (non-explosive) → CompExplosiveCE triggers secondary explosion with `explosiveDamageType=Thermobaric`. The overpressure postfix on `GenExplosionCE.DoExplosion` fires for the comp explosion.
- `RegionTraverser.BreadthFirstTraverse` accepts an `entryCondition` predicate (e.g. check `region.door == null || region.door.Open`) and a `regionProcessor`. Prefer over `map.mapPawns.AllPawnsSpawned`.
- **Never mutate the map or apply damage inside a `RegionTraverser.BreadthFirstTraverse` callback.** `pawn.TakeDamage()`, `pawn.Kill()`, `thing.DeSpawn()`, and similar side effects can invalidate the region graph mid-traversal, corrupting the BFS worker's internal state and causing a native SIGSEGV crash. Collect pawns/things into a pooled collection during the traversal, then apply effects in a separate loop after traversal completes. This is the pattern used by vanilla `DamageWorker.ExplosionDamageThing` and every safe explosion handler.
- `Region.ListerThings.ThingsInGroup(ThingRequestGroup.Pawn)` returns pawns in a region. Deduplicate with a pooled `HashSet<Pawn>` from `SimplePool<T>`.
- `Room.UsesOutdoorTemperature` is true when touching map edge or `OpenRoofCount >= ceil(CellCount * 0.25)`. Use as a proxy for enclosed-space detection.
- CE stores additional explosion stats in `ExplosionCE` (extends `Explosion`): `height`, `armorPenetration`, `damAmount`, `weapon`, `projectile`, `FalloffCurve`, `GetArmorPenetrationAtCE(IntVec3)`.
- CE explosion `instigator` resolution: `CompExplosiveCE.Explode()` receives `instigator` from the caller — `ProjectileCE` passes `this` (the projectile itself, not the Pawn), `Projectile_FireTrail` passes `launcher` (the Pawn), `AmmoThing` passes `this`. When using the `instigator` parameter from a `GenExplosionCE.DoExplosion` postfix, resolve the actual Pawn via `(instigator as ProjectileCE)?.launcher` — `ProjectileCE.launcher: Thing` is the firing pawn/weapon.
- `GenExplosionCE.DoExplosion` sets `explosion.projectile = parent.def` from `CompExplosiveCE.Explode` (the explosive thing's ThingDef). The `weapon` parameter is explicitly null in typical calls. For battle log entries, use `projectile` (ThingDef) as both `weaponDef` and `projectileDef`.

## Custom def creation

- Custom `DamageDef`: inherit from `BluntBase` or `Bomb` via `ParentName`. Override `workerClass` to `DamageWorker_AddInjury` to avoid vanilla Blunt stun/penetration mechanics when handling effects yourself. Set custom `<hediff>`, `<hediffSkin>`, `<hediffSolid>` to reference custom `HediffDef`s for distinguishable injuries.
- Custom `HediffDef` for injuries: inherit from `InjuryBase` (abstract, provides `hediffClass>Hediff_Injury</hediffClass>` and other base properties). Include `HediffCompProperties_TendDuration`, `HediffCompProperties_Infecter`, `HediffCompProperties_GetsPermanent` comps as needed. Set `injuryProps` with `painPerSeverity`, `bleedRate`, `canMerge`, `destroyedLabel`.
- Custom mod extension: subclass `DefModExtension` with mutable fields. Declare with `[<AllowNullLiteral>]`. Add via `<li Class="Namespace.ExtensionClass">` in XML or via `PatchOperationAddModExtension` in patches.
- Use `PatchOperationFindMod` with `<mods><li>Combat Extended</li></mods>` guard when patching CE defs.
- Use `PatchOperationAddModExtension` to add extensions to specific defs by xpath.

## Mod Settings (player-facing configuration)

- When the user asks for a toggle or setting that the **player** should control at runtime (e.g. enable logging, adjust thresholds), implement `Mod` + `ModSettings` — never bury it as an XML def field that requires manual file edits.
- XML def fields (`OverpressureSettingsDef`, `ThingDef`, etc.) are for **mod developer** tuning, not player configuration. They ship with reasonable defaults and are only changed when recalibrating the mod itself.
- Pattern:
  - Subclass `Mod(content: ModContentPack)` with `override SettingsCategory()` and `override DoSettingsWindowContents(Rect)`.
  - Subclass `ModSettings()` with `let mutable` fields, public members, and `override ExposeData()` calling `Scribe_Values.Look`.
  - In F#, place `ModSettings` subclass **before** `Mod` subclass in the same file (F# type ordering). Use `let mutable` + property wrappers instead of `[<DefaultValue>] val mutable` to avoid CLR init issues.
  - Do not call `GetSettings<T>()` from the constructor body; defer to `LongEventHandler.ExecuteWhenFinished`.

## CE stats calibration (project-specific working notes)

The numeric calibration notes in this section are repository-specific working values.
Their source and provenance are not recorded here, so treat them as unverified examples
until the target project supplies data, source references, or in-game measurements.
Do not generalize them to other CE versions, weapons, or mods without recalibration.

- CE explosion `damageAmountBase` and `explosionRadius` are game-design values, not physical quantities. Do not treat them as direct TNT/kg mappings without calibration.
- For HE artillery/mortar rounds, a log-linear power-law model `tntKg = k · damage^a · radius^b` fits real-world data with R² ≈ 0.96. Airburst variants (same filler, wider radius) help disentangle damage vs radius contributions.
- Thermobaric weapons have CE stats that are inconsistent with real blast yields — a separate regression model fails (R² ≈ 0.40). Use explicit per-weapon `tntEquivalentKg` override values instead.
- Small-caliber thermobarics (30-40mm) produce ~0.03-0.05 kg TNT blast equivalent; RPG-7 thermobaric ~1.8-3.0 kg; howitzer thermobaric comp explosions ~4-9 kg.
- HE regression-derived coefficients for `OverpressureSettingsDef`: `damagePerKgTnt ≈ 236`, `damageYieldWeight ≈ 0.82`, `YieldExponent ≈ 1.60`, `RadiusMetresPerKgTnt = 2.0`.
- CE caliber multiplier: damage may be scaled by `RangedWeapon_DamageMultiplier` stat on the firing weapon. Radius is not scaled. Always verify in-game values via UI or debug log before using in calibration.
