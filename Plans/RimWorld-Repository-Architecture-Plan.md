# RimWorld Modding Repository — Initial Architecture & Development Plan

**Repository:** `RimworldDEV`  
**Owner / development namespace:** ThelianTech  
**Baseline established:** 2026-11-19  
**Status:** Initial architectural decisions recorded; implementation of common build configuration and utility toolchain remains pending.  
**Suggested repository location for this document:** `Wiki-Docs/RimWorld-Repository-Architecture-Plan.md`

> **Purpose:** Preserve the agreed architecture, naming, version-support policy, development workflows, and remaining implementation work so the repository can be resumed after an extended absence. This is a **plan and decision record**, not a claim that every described file or utility has already been implemented.

## 1. Executive decision summary

| Area | Decision |
|---|---|
| Repository organization | **One Git repository** containing individual mod projects, utilities, documentation, and multiple solutions. |
| Top-level application solution | Keep existing `ThelianTech_Rimworld-Mods.slnx` at the repository root. Membership is **manual**. |
| Physical domain root | Keep `Directory/` with child directories **`Mods/`, `Projects/`, and `Utilities/`**. |
| Mods solution | `Directory/Mods/Mods.slnx` — working solution for creating and maintaining RimWorld mod projects. |
| Utilities solution | `Directory/Utilities/Utilities.slnx` — working solution for development and automation tools. |
| Purpose-specific solutions | `Directory/Projects/<InitiativeName>.slnx` — **blank on creation**, then manually populated with existing mod/utility projects as needed. |
| Solution relationships | **No actual nested `.slnx`**. The same `.csproj` may be referenced by multiple independent solutions. |
| Project ownership | One physical mod directory and, when C# is required, one initial `.csproj` per mod. Utility projects live under `Utilities/`. |
| Game baseline | **RimWorld 1.7 onward**, Waylith/.NET 10; **no committed requirement** to support 1.6 or earlier. |
| C# target | `net10.0` for RimWorld 1.7 mod class libraries. SDK version and target framework are different concerns. |
| SDK selection | **No `global.json`** by preference. Target framework is explicit; installed SDK selection is handled by .NET/Visual Studio. |
| Source versioning | Start with **one 1.7 source implementation** per mod. Do not create duplicate `Source/1.6` trees without an actual support requirement. |
| Shared mod configuration | Planned `Directory.Build.props`, `Directory.Packages.props`, and `Directory.Build.targets` physically under `Directory/Mods/`. |
| Project registration | **Manually curated**. Creating a mod in `Mods.slnx` must **not automatically add** it to the repository master or purpose-specific solutions. |
| Packaging | Future utility workflow separates **build**, **validation**, **in-game testing**, and **release packaging**. |

## 2. Verified starting layout versus planned additions

The screenshot of the repository at the time of planning showed:

```text
RimworldDEV/
├── .git/
├── .vs/
├── Directory/
│   ├── Mods/
│   ├── Projects/
│   ├── Utilities/
│   └── README.md
├── Wiki-Docs/
├── .gitignore
├── LICENSE
├── README.md
└── ThelianTech_Rimworld-Mods.slnx
```

The three domain directories were visible, but the screenshots **did not confirm** that any new domain `.slnx` files or MSBuild configuration files had yet been created. The script `New-RimWorldSolution.ps1` was provided separately; place it as indicated below.

### Target layout (selected illustrative projects)

```text
RimworldDEV/
├── .git/
├── .gitignore
├── LICENSE
├── README.md
├── ThelianTech_Rimworld-Mods.slnx       # Existing master; membership is optional/manual
│
├── Directory/
│   ├── README.md                       # Directory index, not replaced by utility docs
│   │
│   ├── Mods/
│   │   ├── Mods.slnx
│   │   ├── Directory.Build.props       # PLANNED: mod-wide MSBuild defaults
│   │   ├── Directory.Packages.props    # PLANNED: centralized package versions
│   │   ├── Directory.Build.targets     # PLANNED: common build/deployment behavior
│   │   ├── ExampleMod/                 # ILLUSTRATIVE — not verified as present
│   │   │   ├── About/
│   │   │   │   └── About.xml
│   │   │   ├── Source/
│   │   │   │   └── ModEntry.cs
│   │   │   ├── 1.7/
│   │   │   │   ├── Assemblies/          # Generated DLL deployment target
│   │   │   │   ├── Defs/               # Only if needed
│   │   │   │   └── Patches/            # Only if needed
│   │   │   ├── LoadFolders.xml         # If custom load order is necessary
│   │   │   └── ExampleMod.csproj
│   │   └── <AdditionalMods>/
│   │
│   ├── Projects/
│   │   ├── PawnSystemsOverhaul.slnx     # ILLUSTRATIVE composition solution
│   │   └── <MoreDevelopmentInitiatives>.slnx
│   │
│   └── Utilities/
│       ├── Utilities.slnx
│       ├── New-RimWorldSolution.ps1
│       ├── Rw-S-S.md                   # Dedicated script usage guide
│       ├── README.md                   # Reserved for utilities directory overview
│       └── <FutureToolProjects>/
│
└── Wiki-Docs/
    └── RimWorld-Repository-Architecture-Plan.md
```

**Important:** `Mods`, `Projects`, and `Utilities` are under **`Directory/`**, not directly under the Git root. Preserve these exact physical names unless a later architectural change is deliberate.

## 3. Solution model: four distinct roles

### A. Repository-wide master solution

- **File:** `ThelianTech_Rimworld-Mods.slnx` (existing).
- **Purpose:** Optional high-level development workspace containing whichever projects are intentionally added to it.
- **Registration policy:** Manual; no synchronization from `Mods.slnx`, `Utilities.slnx`, or `Projects/`.
- **Crucial distinction:** It contains references to `.csproj` files, not functioning child solution objects.

### B. Mods domain solution

- **File:** `Directory/Mods/Mods.slnx`.
- **Purpose:** Normal entry point for **creating and developing individual RimWorld mods**.
- **New class libraries:** Create under `Directory/Mods/<ModName>/`, with `<ModName>.csproj` in the mod's physical root.
- **Game target:** RimWorld 1.7 / `net10.0`.
- **Configuration:** Inherit shared references and build defaults via files physically located in `Directory/Mods/` (planned; see §6).

### C. Utilities domain solution

- **File:** `Directory/Utilities/Utilities.slnx`.
- **Purpose:** Develop build automation, validation, release packaging, diagnostics, and other developer tools.
- **Project types:** Console apps, libraries, or other appropriate .NET project types; **not automatically RimWorld mod DLLs**.
- **Dependency isolation:** Utilities should not automatically inherit RimWorld mod reference packages simply because they appear in a custom composition solution.

### D. Purpose-specific development solutions

- **Files:** `Directory/Projects/<InitiativeName>.slnx`.
- **Purpose:** Named, visual groupings of **existing** mod and utility projects for a particular mashup, compatibility initiative, research effort, or overhaul.
- **Creation:** Begin empty; manually choose exactly which existing `.csproj` files to add.
- **Example:** `PawnSystemsOverhaul.slnx` might include hauling, pathfinding, door-permissions, a new unified mod project, and a release/validation utility.
- **No duplication:** Adding a `.csproj` to another `.slnx` references the same physical project.
- **No automatic merging:** Building this solution does **not** fuse its projects or DLLs into one game mod.

**Operational model:** Each solution is an independent **workspace definition**, not a child solution, separate Git repository, or default release package definition.

## 4. Creation and daily workflows

Assume commands are run from:

```powershell
Set-Location 'T:\ThelianTech\ThelianTech-Studios\ModProjects\RimworldDEV'
```

### 4.1 Create the two domain solutions (one-time action)

After placing `New-RimWorldSolution.ps1` under `Directory/Utilities/`:

```powershell
.\Directory\Utilities\New-RimWorldSolution.ps1 -Name Mods -Area Mods
.\Directory\Utilities\New-RimWorldSolution.ps1 -Name Utilities -Area Utilities
```

Equivalent direct CLI commands:

```powershell
dotnet new sln --name Mods --format slnx --output .\Directory\Mods
dotnet new sln --name Utilities --format slnx --output .\Directory\Utilities
```

**Do not run both methods for the same solution.** The script refuses to overwrite an existing target solution. Neither approach creates a further solution-named subfolder.

### 4.2 Create a new mod project

Preferred Visual Studio workflow:

1. Open **`Directory/Mods/Mods.slnx`**.
2. **Add → New Project → C# Class Library**.
3. Name it for the mod and ensure the physical output is `Directory/Mods/<ModName>/<ModName>.csproj` (no extra `<ModName>/<ModName>/` nesting).
4. Select **.NET 10** / `net10.0`.
5. Create mod directories such as `About/`, `Source/`, and `1.7/` **within the physical C# project directory** as needed.
6. Verify inherited RimWorld references and deployment after the shared configuration files are installed.

CLI equivalent (example):

```powershell
dotnet new classlib --name ExampleMod --framework net10.0 `
  --output .\Directory\Mods\ExampleMod

dotnet sln .\Directory\Mods\Mods.slnx add `
  .\Directory\Mods\ExampleMod\ExampleMod.csproj --in-root
```

This creates and registers **one** mod project in the Mods domain solution. It does **not** register that project in the master solution or any `Projects/` solution.

**XML-only mod exception:** A mod with no compiled C# code may not need a `.csproj`. Its content can still live under `Directory/Mods/<ModName>/` and be tracked in Git; whether/how to expose it in Visual Studio is a separate choice.

### 4.3 Create a development-initiative solution

```powershell
.\Directory\Utilities\New-RimWorldSolution.ps1 -Name PawnSystemsOverhaul
```

Creates **`Directory/Projects/PawnSystemsOverhaul.slnx`**, initially blank.

Next open that `.slnx` in Visual Studio, use **Add → Existing Project**, and select the relevant `.csproj` files from `Directory/Mods/` and `Directory/Utilities/`. Use virtual solution folders if useful for visual organization. This selection is **deliberately manual**.

See [`Rw-S-S.md`](../Directory/Utilities/Rw-S-S.md) for the full script usage and optional `-ProjectPaths` behavior.

### 4.4 Build a domain or initiative solution

```powershell
# Mods domain
dotnet build .\Directory\Mods\Mods.slnx -c Release

# Utilities domain
dotnet build .\Directory\Utilities\Utilities.slnx -c Release

# Example custom initiative
dotnet build .\Directory\Projects\PawnSystemsOverhaul.slnx -c Release
```

Building a solution compiles its registered projects. **It does not automatically package, test, or merge them.** A future utility will define those orchestration rules explicitly.

## 5. Mod structure and version-support policy

### 5.1 Chosen baseline

- Develop **for RimWorld 1.7 onward**, not 1.6 backwards compatibility.
- Build C# mod assemblies targeting **`net10.0`** against RimWorld 1.7-compatible reference assemblies.
- Existing RimWorld 1.6 mods may be studied or adapted when licensing permits; that is **not** a promise to maintain a 1.6 build.
- Start with **one source tree**, rather than parallel `Source/1.6` and `Source/1.7` trees.
- Future version-specific builds should be introduced only when testing or API differences actually require them.
- A modern SDK may compile a `net10.0` project, but SDK selection, target framework, and the customized RimWorld runtime are separate concepts.
- **Do not create `global.json`** solely to force SDK 10: the stated choice is to avoid it.

### 5.2 A mod root is not a C# project

A RimWorld mod is the **physical directory** that the game can load. A `.csproj` is the **build definition** for its optional DLL. The solution is the **IDE grouping**.

```text
Directory/Mods/ExampleMod/
├── About/
│   └── About.xml
├── Source/
│   └── ModEntry.cs
├── 1.7/
│   └── Assemblies/
│       └── ExampleMod.dll       # Build output, not source
├── ExampleMod.csproj            # Located in mod root by decision
└── LoadFolders.xml              # Only where custom loading rules are needed
```

The actual folders `Defs/`, `Patches/`, `Textures/`, and localization are created only when the mod requires them. `bin/` and `obj/` are temporary SDK-generated directories, not additional mod projects. The build should keep compilation intermediates separate from release outputs.

### 5.3 Game-version directories and future migration

The `1.7/` directory holds game-loadable, version-specific content. It is **not** a separate Visual Studio project. Later releases (1.8+) may call for another output directory and/or separate conditional compilation, but **future compatibility is assessed, not automatically assumed**.

## 6. Planned automatic mod-domain build configuration

**Status: AGREED DESIGN; NOT YET VERIFIED AS INSTALLED OR TESTED.**

Place these **inside `Directory/Mods/`**, not at the Git root:

| File | Role |
|---|---|
| `Directory.Build.props` | Inherited properties, mod-specific compile defaults, and a shared RimWorld reference `PackageReference`. |
| `Directory.Packages.props` | Central Package Management: one chosen version of `Krafs.Rimworld.Ref` for the mod domain. |
| `Directory.Build.targets` | Shared post-build deployment/validation behavior (e.g., copy the successful mod DLL to that mod's `1.7/Assemblies/`). |

**Why it works:** MSBuild/NuGet discover these files by walking **up the physical project directory hierarchy**. Thus `Directory/Mods/ExampleMod/ExampleMod.csproj` inherits them no matter which `.slnx` references it. Projects physically under `Directory/Utilities/` do not inherit the Mods-specific files.

### 6.1 Intended configuration pattern (illustrative, requires validation)

`Directory/Mods/Directory.Packages.props`:

```xml
<Project>
  <PropertyGroup>
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <!-- Select and pin a version matching the actual RimWorld 1.7 build. -->
    <PackageVersion Include="Krafs.Rimworld.Ref" Version="1.7.5013-unstable" />
  </ItemGroup>
</Project>
```

**Version note:** `1.7.5013-unstable` is an example of a published package, **not a permanently approved version**. Confirm the actual installed RimWorld build and package availability before selecting and locking a reference version. Avoid floating package versions for reproducible releases.

`Directory/Mods/Directory.Build.props`:

```xml
<Project>
  <PropertyGroup>
    <TargetFramework Condition="'$(TargetFramework)' == '' and '$(TargetFrameworks)' == ''">net10.0</TargetFramework>
    <IsPackable>false</IsPackable>
    <GenerateDependencyFile>false</GenerateDependencyFile>
    <CopyLocalLockFileAssemblies>false</CopyLocalLockFileAssemblies>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Krafs.Rimworld.Ref" PrivateAssets="all" />
  </ItemGroup>
</Project>
```

**Important:** A Visual Studio-generated `.csproj` with an explicitly set `TargetFramework` overrides the **default** property above; always choose `net10.0` when creating the class library. The `PackageReference` comes from the inherited props file, while the `PackageVersion` comes from the NuGet central version file. Both roles are required.

`Directory/Mods/Directory.Build.targets` (conceptual starter):

```xml
<Project>
  <Target Name="DeployRimWorldMod" AfterTargets="Build"
          Condition="'$(TargetFramework)' == 'net10.0'">
    <PropertyGroup>
      <ModAssembliesPath>$(MSBuildProjectDirectory)\1.7\Assemblies\</ModAssembliesPath>
    </PropertyGroup>
    <MakeDir Directories="$(ModAssembliesPath)" />
    <Copy SourceFiles="$(TargetPath)"
          DestinationFolder="$(ModAssembliesPath)"
          SkipUnchangedFiles="true"
          Condition="Exists('$(TargetPath)')" />
  </Target>
</Project>
```

**Validation required:** Test the deploy target with at least one actual mod; review how build configuration, native dependencies, debugging assets, and packaging should behave. A successful compile is only the first gate, not proof that RimWorld can load the mod.

### 6.2 Package policy

- Baseline reference: [`Krafs.Rimworld.Ref`](https://www.nuget.org/packages/Krafs.Rimworld.Ref) (RimRef compile-time references).
- Select the package build corresponding to the **actual game build** under test; pin it.
- Do not assume all mods need Harmony or a particular Harmony major version. Add patching and third-party API packages based on actual needs and verified 1.7 support.
- Do not copy proprietary RimWorld game assemblies into the public repository without appropriate permissions.

## 7. Planned utility toolchain and release architecture

**Status: Future capabilities, not yet implemented.** Utility project names below are descriptive placeholders, not mandatory projects.

Potential components:

- **ModBuildTool:** discover eligible mod project(s), invoke `dotnet build` with declared settings, and capture diagnostics.
- **ModValidationTool:** check mod metadata, dependencies, XML, assembly output, and package layout; add richer automated checks over time.
- **ModReleasePackager:** stage and archive eligible mod content; produce a manifest and release artifacts.
- **Solution helper:** maintain purpose-specific `.slnx` workspace files when explicitly requested (not automatic registration into all solutions).

### Release gates (keep separate)

```text
Select intended mod / initiative
           ↓
Build project(s)
           ↓
Static validation + dependency checks
           ↓
Automated tests (where available)
           ↓
In-game testing / explicit release approval
           ↓
Stage distributable mod folder → produce ZIP and report
```

**Do not confuse:**

- **Build succeeded** ≠ automatically **tests passed**.
- **Tests passed** ≠ automatically **safe for existing saves**.
- **Solution contains multiple mod projects** ≠ a merged DLL or unified release.
- **Combination mod** ≠ automatically improved performance; benchmark and profile the actual integration.

A future optional initiative manifest, e.g. `Directory/Projects/PawnSystemsOverhaul.build.json`, could specify what to package. This is **a proposed custom format**, not a RimWorld/.NET requirement. The `.slnx` stays a workspace/grouping definition.

## 8. Integrating and adapting existing mods

Purpose-specific `Projects/` solutions are useful for studying, comparing, and composing functionality from separate mod projects. For an actual combined RimWorld mod:

1. Identify original behavior, patch points, and performance concerns.
2. Review each source mod's **license, permissions, and dependencies** before copying or redistributing code/assets.
3. Create or maintain a **dedicated unified mod project** under `Directory/Mods/` for what will actually ship.
4. Register relevant reference projects and that unified project in a manually curated `Projects/<Initiative>.slnx`.
5. Consolidate conflicting implementations deliberately; do not assume combining them eliminates performance or save risks.
6. Package **only the explicitly selected output**, with licensing and attribution notices as required.

Old 1.6 source may be retained for comparison under appropriate licensing, but it is not a mandatory compiled target in the chosen 1.7-forward policy.

## 9. Repository documentation and Git conventions

- Root `README.md`: overview and entry points for the entire repository.
- `Directory/README.md`: overview/index for the Mods, Projects, and Utilities domains.
- `Directory/Utilities/README.md`: index of tools and scripts; **do not overwrite with script instructions**.
- `Directory/Utilities/Rw-S-S.md`: operational instructions for `New-RimWorldSolution.ps1`.
- `Wiki-Docs/`: longer-lived repository and mod documentation, including this architecture plan if desired.
- `.slnx` files: normally version-controlled; manual project membership intentionally maintained.
- `.gitignore`: ignore `.vs/`, `bin/`, `obj/`, tool caches, and transient release/staging output, with explicit decisions about whether compiled mod DLLs are tracked or generated only for releases.
- **One Git root:** no new repositories or submodules are required simply to create additional solutions.

## 10. Implementation checklist and status

The following separates **decisions** from **verified execution**:

| Item | Status as of this plan |
|---|---|
| Root Git repository, `Directory/`, `Wiki-Docs/`, master `.slnx` | **Observed in screenshots** |
| `Directory/Mods`, `Directory/Projects`, `Directory/Utilities` | **Observed in screenshots** |
| `New-RimWorldSolution.ps1` | **Provided as separate download**; placement in repository not verified |
| `Rw-S-S.md` usage guide | **Created as downloadable document**; repository placement pending |
| `Directory/Mods/Mods.slnx` | **To create / verify** |
| `Directory/Utilities/Utilities.slnx` | **To create / verify** |
| `Directory/Projects/<Initiative>.slnx` | **Create on demand** |
| Shared `Directory.Build.props` / `.targets` / `Directory.Packages.props` | **Design agreed; implement and test** |
| First mod class library inheriting settings | **Verify after MSBuild files are created** |
| RimRef package version matched to installed RimWorld build | **Verify and pin** |
| Mod build and validation utilities | **Future work** |
| Release pipeline and manifest | **Future design/implementation** |

### Suggested next execution sequence

1. Place `New-RimWorldSolution.ps1` and `Rw-S-S.md` into `Directory/Utilities/`.
2. Create or verify `Mods.slnx` and `Utilities.slnx` using the script.
3. Implement the three Mods-specific MSBuild/NuGet files and confirm package restore and compilation for a test mod.
4. Confirm a newly created mod automatically inherits its references **without** any per-project manual NuGet installation.
5. Open a new empty solution under `Directory/Projects/`; manually add an existing mod and, if available, a utility project; confirm paths and solution behavior.
6. Build one mod to `1.7/Assemblies/`, inspect the output, and test load in the intended RimWorld 1.7 build.
7. Design release manifests, validation stages, and packaging **after** the basic build process is dependable.

## 11. Things not to re-decide without a new requirement

- Do **not** introduce 1.6 backward-compatibility targets by default.
- Do **not** create parallel source trees or multiple `.csproj` files per mod preemptively.
- Do **not** create a `global.json` merely to force SDK 10; the project targets `net10.0` independently.
- Do **not** automatically add new mod projects to the root master solution or every initiative solution.
- Do **not** create actual nested solutions; keep standalone `.slnx` workspaces with shared project references.
- Do **not** use a composition `.slnx` as an implicit release or DLL-merging rule.
- Do **not** move `Directory/Mods`, `Directory/Projects`, and `Directory/Utilities` to the Git root without an explicit restructuring decision.

## 12. References for future maintenance

- [Microsoft — `dotnet sln`](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-sln)
- [Microsoft — SDK templates and `dotnet new sln`](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new-sdk-templates)
- [Microsoft — Customize the build by folder (`Directory.Build.props` / `.targets`)](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory)
- [Microsoft / NuGet — Central Package Management](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management)
- [NuGet — Krafs.Rimworld.Ref](https://www.nuget.org/packages/Krafs.Rimworld.Ref)
- [RimWorld Wiki — Modding Tutorials](https://rimworldwiki.com/wiki/Modding_Tutorials)

---

**Revision protocol:** When an architectural decision changes, update the executive summary, affected implementation steps, and checklist together. Record the date and rationale in a change log below rather than silently replacing the original intent.

### Change log

| Date | Revision |
|---|---|
| 2026-11-19 | Initial baseline: one Git repository; root master `.slnx`; domain Mods/Utilities solutions; manually populated initiative solutions in `Directory/Projects/`; RimWorld 1.7 / `net10.0` focus; inherited mod-domain build configuration planned; no automatic solution membership. |
