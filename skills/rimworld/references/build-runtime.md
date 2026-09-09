# Build and runtime

## DLL management (binary artifacts)

- **Never commit `.dll` files to the repository.** Compiled assemblies are build artifacts, not source.
- Every mod repository must include `*.dll` in its `.gitignore`. Also add `Source/**/bin/` and `Source/**/obj/`.
- If a repository already tracks `.dll` files, stop tracking them in an explicitly authorized cleanup. Use history rewriting and force-push only when the user explicitly requests history cleanup.
- Distribute compiled DLLs via GitHub Releases, not via git history. Copy the reference CI workflow from `ci-build-release.yml` into `.github/workflows/release.yml` and adjust the env vars.
- The CI workflow: builds the .dll on tag push → packages the full mod tree (About/, Assemblies/, Defs/, Languages/, Patches/, Textures/) into a zip → attaches to a GitHub Release.

## Build

1. Read the repository's build instructions and project files, then invoke the documented project command directly.
2. On this machine, when the repository does not pin a .NET SDK, build SDK-style RimWorld projects with:

   ```sh
   mise exec dotnet@8.0.422 -- dotnet build <path-to-project-file> -c Release
   ```

3. Do not change the global mise default merely because bare `dotnet` reports that no shim version is set.
4. Verify zero build errors and inspect the project-defined output path for the expected mod DLL. If the build fails, stop before installation and report the blocker.
5. Install only the exact DLL produced by the verified target-source build. Never patch, rewrite, decompile/recompile, or substitute a DLL to bypass a source build failure, including in a local test mod.
6. Do not copy or overwrite build artifacts (such as DLLs) in the Steam Workshop directory (`steamapps/workshop/content/...`). This directory is managed automatically by Steam. Mod installation, local testing, and debugging must only target the local RimWorld `Mods` directory (`steamapps/common/RimWorld/Mods/...`) or user-managed directories. A running game does not prohibit copying a verified build into a local mod; it continues using its already loaded DLL until restart. Do not require exit merely to install that DLL.

## Runtime control

- Never launch the interactive RimWorld game client or open a visible game window unless the user explicitly authorizes that specific launch. Build, install, enable, or an earlier launch does not authorize a later interactive launch.
- Automated test-runner invocations are allowed when they are non-interactive, use isolated config/save/log paths, do not alter the user's normal runtime state, and exit automatically. If the test runner requires or may open a visible game window, obtain explicit authorization first.
- The user controls interactive game startup, save selection, and in-game verification. When the user reports a runtime result, use that result as evidence and do not independently repeat the interactive check.

## Local runtime paths

- RimWorld install: `/media/scarf/@steam/SteamLibrary/steamapps/common/RimWorld`
- Mods directory: `/media/scarf/@steam/SteamLibrary/steamapps/common/RimWorld/Mods`
- Player log: `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Player.log`
- Save files: `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Saves`
- Mods config: `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Config/ModsConfig.xml`
- Prefs (language, resolution): `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Config/Prefs.xml`
- CE Steam Workshop: `/media/scarf/@steam/SteamLibrary/steamapps/workshop/content/294100/2890901044`
- RimWorld 1.6 reference decompile: `/home/scarf/repo/rw/RimworldDecompile`

## Save file inspection

- `.rws` save files are XML. Always use `rg -F` (fixed-string) for literal tag/def name search — regex patterns silently fail on `<tag>` angle brackets.
- Prefer `rg -F "pattern" save.rws` over `rg "pattern"` for any XML file.

## First-party mod justfile

- Every mod authored in its own repository must include a tracked root `justfile`. Once present, treat its recipes as the project contract instead of invoking their underlying commands separately.
- In third-party repositories, keep agent-created local workflow files untracked and locally ignored. Do not commit them unless the user explicitly requests that repository-facing change.
- Provide these public recipes:
  - `just fmt`: format all source using tracked configuration—K&R via `dotnet format` for C#, or a repository-pinned Fantomas tool for F#.
  - `just build`: compile the mod and produce every runtime assembly in its project-defined version directory.
  - `just install`: depend on `build`, then synchronize the complete runtime mod tree into a dedicated directory under the local RimWorld `Mods` directory. Exclude repository and development-only files, and constrain deletion to that dedicated destination.
  - `just enable`: depend on `install`, then idempotently add the `packageId` from `About/About.xml` to `activeMods` in `ModsConfig.xml`. Preserve every existing entry and its order, never add a duplicate, and leave valid XML.
- Make local paths environment-overridable and use these defaults on this machine:
  - RimWorld: `/media/scarf/@steam/SteamLibrary/steamapps/common/RimWorld`
  - Config: `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Config/ModsConfig.xml`
  - Log: `/home/scarf/.config/unity3d/Ludeon Studios/RimWorld by Ludeon Studios/Player.log`
- Do not edit `ModsConfig.xml` while RimWorld is running because the game can overwrite it on exit.
- Validate a new justfile with `just --list`, `just fmt`, and `just build`; run `just fmt` a second time to verify idempotence. Run `just install` or `just install-enable` only when the task explicitly requests the corresponding local state change.
