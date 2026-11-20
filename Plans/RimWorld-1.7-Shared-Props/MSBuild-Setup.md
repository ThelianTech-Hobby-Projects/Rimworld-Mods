# RimWorld 1.7 - Shared Mod Dependencies

**Status:** Initial development-only configuration, prepared 2026-11-19.  
**Scope:** C# class library projects physically located below `src/Mods/`.  
**Does not cover:** releasing, publishing, incrementing mod versions, packaging, copying DLLs to `1.7/Assemblies`, or adding projects to additional solutions.

## 1. Files to install

Place both files in the **physical** Mods directory:

```text
RimworldDEV/
├── src/
│   ├── Mods/
│   │   ├── Mods.slnx
│   │   ├── Directory.Build.props
│   │   ├── Directory.Packages.props
│   │   └── ExampleMod/
│   │       └── ExampleMod.csproj
│   └── Utilities/
│       └── Utilities.slnx
├── Projects/
└── ThelianTech_Rimworld-Mods.slnx
```

If the repository move from `Directory/Mods` to `src/Mods` is not yet complete, place these two files under the *current physical Mods directory* instead. Their behavior depends on the location of each `.csproj`, not the `.slnx` location.

**Only two shared files are required for this baseline.** No `global.json`, `Directory.Build.targets`, per-mod `.props`, or release automation is needed yet.

## 2. What is inherited automatically

- **Default target framework:** `net10.0` when the project does not already declare `TargetFramework` or `TargetFrameworks`.
- **`Krafs.Rimworld.Ref`** supplies RimWorld 1.7 compile-time reference assemblies.
- **`Krafs.Rimworld.Ref.Local`** is included automatically for every mod: in Debug, when the installed game exactly matches the pinned RimRef version, it substitutes local game assemblies, enabling navigation to real implementation code. Without an exact match, it falls back to portable RimRef; Release always uses RimRef.
- **`Lib.Harmony.Ref`** supplies Harmony 3 compile-time reference assemblies.
- **`PrivateAssets="all"`** keeps inherited packages private to the mod build. Both `.Ref` packages use **`ExcludeAssets="runtime"`**; the `.Local` package deliberately retains build assets because it imports MSBuild targets. The game and enabled Workshop Harmony mod supply runtime implementations.
- **Central package versions:** `Directory.Packages.props` controls both package versions; `Directory.Build.props` actually adds the references.
- **Build outputs:** ordinary `bin/Debug/net10.0` and `bin/Release/net10.0` as managed by .NET, with no automatic copying to the RimWorld mod directory.

Versions pinned in this initial baseline:

| Package | Initial pinned version | Role |
| --- | --- | --- |
| `Krafs.Rimworld.Ref` | `1.7.5014-unstable` | RimWorld 1.7 game reference assemblies |
| `Lib.Harmony.Ref` | `3.0.0-rc.7` | Harmony 3 compile-time API |
| `Krafs.Rimworld.Ref.Local` | `1.0.0` | Debug-only local game assembly substitution |

**Version caution:** RimWorld 1.7 is experimental in this baseline; verify that your installed RimWorld build matches the RimRef version before treating a successful compilation as compatible. Update the pinned versions intentionally; don't assume a NuGet version equals the game/Workshop runtime on your PC.

## 3. Creating a new mod project

Open `src/Mods/Mods.slnx` in Visual Studio and use **Add > New Project > Class Library (C#)**. Create it physically under `src/Mods/<ModName>/<ModName>.csproj` and choose **.NET 10**.

Or run from the repository root:

```powershell
dotnet new classlib --name ExampleMod --framework net10.0 --output .\src\Mods\ExampleMod
dotnet sln .\src\Mods\Mods.slnx add .\src\Mods\ExampleMod\ExampleMod.csproj
```

An ordinary project file is enough:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>
</Project>
```

The packages need **no additional `PackageReference` entries** in the new `.csproj`: they are inherited automatically. If Visual Studio creates the project targeting `net11.0`, **change it to `net10.0`**: project-local properties override early `Directory.Build.props` defaults.

## 4. Restore and verify

Run from the repository root:

```powershell
# Confirm available SDKs (the .NET 10 targeting pack / SDK must be installed).
dotnet --list-sdks

# See the effective target framework and inherited package references.
dotnet msbuild .\src\Mods\ExampleMod\ExampleMod.csproj -getProperty:TargetFramework -getItem:PackageReference

# Download NuGet packages and create the assets file.
dotnet restore .\src\Mods\ExampleMod\ExampleMod.csproj

# Compile the mod DLL, using only the reference assemblies.
dotnet build .\src\Mods\ExampleMod\ExampleMod.csproj -c Debug

dotnet build .\src\Mods\ExampleMod\ExampleMod.csproj -c Release
```

Once restore succeeds, Visual Studio's **Dependencies** node and C# IntelliSense should resolve the packages' API declarations. A missing SDK, failed restore, unmatched game build, or API absent from that reference package may still produce squiggles/compiler errors. Compiling does not prove a Harmony patch works at runtime; test with the corresponding RimWorld and Harmony versions.

## 5. Reference packages vs. runtime packages

**Harmony:** For a mod loaded by RimWorld, use **`Lib.Harmony.Ref` alone** for IntelliSense, type/method signatures, and compilation, including Debug and Release. Do **not** add `Lib.Harmony` as another mod-project dependency just to make Debug builds work; the official RimWorld Harmony Workshop mod supplies the executable library at game runtime. Only mods that *actually use* Harmony APIs should declare the Harmony mod as a runtime requirement in their `About/About.xml` metadata.

**RimRef and RimRef Local:** Both packages are inherited automatically. `Krafs.Rimworld.Ref` alone supplies sufficient type signatures for IntelliSense and compiling in both configurations. `Krafs.Rimworld.Ref.Local` additionally attempts to use **your installed game assemblies in Debug** so **Go to Definition** can show real implementation code, not stripped signatures. It uses **strict version matching** by default: the local game build must match the pinned RimRef version (`1.7.5014-unstable` in this sample). Otherwise, Debug falls back to portable reference assemblies. Release and CI use RimRef only. This is the package's own documented behavior; it is **not** necessary to make a Debug build compile. See https://github.com/krafs/RimRefLocal .

If game discovery fails, optionally specify a `RimWorldDirectory` **environment variable** on your computer pointing to the actual RimWorld installation; avoid hardcoding your private path in committed repository configuration. Package discovery also checks Steam libraries. To see why local assemblies were or were not selected, inspect the build output (try `dotnet build -tl:off`). Do not disable strict comparison simply to suppress a version mismatch, because Debug and Release may then compile against different game APIs.

**Build configurations:** `UseLocalRimWorldAssemblies` is explicitly `true` in Debug and `false` in other configurations. `UseStrictRimWorldVersionComparison` is `true`. These settings apply to all mod projects but can be overridden individually if a special case needs it.

**Harmony source inspection:** `Lib.Harmony.Ref` is sufficient for compile-time API access, but does not provide Harmony implementation method bodies. The normal Harmony `.Ref` package is not equivalent to RimRef Local's optional local-code inspection feature.

## 6. Repository conventions and limitations

- Keep mod-specific `Version`, `AssemblyVersion`, game `About.xml` metadata and any special dependencies in each mod project or its future release configuration, not the shared baseline.
- Do not automatically copy DLLs into `1.7/Assemblies` or create release archives on every solution build. Those features belong to the future Build & Release utility.
- Do not add a project to the umbrella/master or initiative solutions automatically; solution memberships are deliberately maintained manually.
- A nested `Directory.Build.props` in one mod's project directory replaces the automatically discovered parent file unless it **explicitly imports** `../Directory.Build.props`.
- The `.Ref` packages are compile-time API surfaces. They are not drop-in replacements for runtime game libraries when launching or executing code outside RimWorld.
- RimRef currently does **not** expose every special API in Ludeon's customized .NET runtime (for example, certain `WaylithClr` APIs). Some advanced cases may require alternate access techniques.

## 7. References verified 2026-11-19

- RimRef: https://github.com/krafs/RimRef
- RimRef Local: https://github.com/krafs/RimRefLocal
- RimRef Local package: https://www.nuget.org/packages/Krafs.Rimworld.Ref.Local/1.0.0
- RimRef package: https://www.nuget.org/packages/Krafs.Rimworld.Ref/1.7.5014-unstable
- Harmony reference package: https://www.nuget.org/packages/Lib.Harmony.Ref/3.0.0-rc.7
- Official RimWorld Harmony loader: https://github.com/pardeike/HarmonyRimWorld
- MSBuild directory imports: https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory
- Central Package Management: https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management
