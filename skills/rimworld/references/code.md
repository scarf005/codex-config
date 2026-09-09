# Code conventions

## C#

- Use K&R braces: place each opening brace on the same line as its declaration, control statement, or object initializer, and keep `else`, `catch`, and `finally` on the same line as the preceding closing brace.
- Use `var` for every local variable whose type the compiler can infer. Keep explicit types for members, parameters, return types, and declarations where inference is unavailable.

## F#

- Target `net472` for RimWorld 1.6 and explicitly order every source file with `<Compile Include="..." />` in the `.fsproj`.
- Pin the shared runtime ABI with `<PackageReference Update="FSharp.Core" Version="4.7.2" ExcludeAssets="runtime" />`. Declare `latta.fsharp.core` as both a mod dependency and `loadAfter` entry, and verify the mod output does not contain a duplicate `FSharp.Core.dll`.
- Use modules and functions by default. Use scoped `internal` extension members when repeated member syntax materially improves a .NET-facing API, follow .NET PascalCase naming for members, and do not hide type conversions behind property-like extensions.
- Define Harmony patch arguments in tupled form so they compile to ordinary CLR method parameters, and initialize `PatchAll()` from a `[<StaticConstructorOnStartup>]` type. Prefer RimWorld 1.6 native extension points such as `FloatMenuOptionProvider` when their selection and invocation gates satisfy the requested behavior; otherwise patch the narrowest gate that does.
- Preserve C# API behavior explicitly at interop boundaries. Use pattern matching and `Option` for nullable results in non-hot paths. F# can apply exact, type-directed `op_Implicit` conversions, but it does not chain every C# upcast-plus-conversion sequence; construct `LocalTargetInfo`, `LookTargets`, and similar targets explicitly when inference fails.
- Prefer idiomatic pipelines and clarity before speculative optimization. `Seq.filter`, `Seq.sortBy`, tuples, and options are appropriate for infrequent user-triggered paths; reserve direct loops, allocation avoidance, and single-pass selection for measured or per-tick hot paths.
- Pin Fantomas as a repository-local .NET tool, store its settings in `.editorconfig`, and use it for all `.fs`, `.fsi`, and `.fsx` formatting.
- Only add `///` XML docstrings when they supply information the code does not already express: mathematical models, design intent, non-obvious side effects, contracts, or category groupings of large member lists. Never add docstrings that merely paraphrase a function name, its parameter names, its return type, or a self-documenting attribute such as `[<HarmonyPatch>]` or `[<StaticConstructorOnStartup>]`. A one-line private helper with a descriptive name and a self-explanatory body rarely needs a docstring.
- When a docstring references an uncommon concept (mathematical model, algorithm, domain-specific term), include a URL to the canonical reference so readers can learn more without guessing search terms.
