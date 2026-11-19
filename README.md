# RimWorld Modding

A centralized repository for developing, maintaining, improving, and experimenting with mods for RimWorld.

The primary focus is on compatibility patches, consolidated mod replacements, performance optimizations, and maintaining or modernizing existing community mods where their licensing permits. The repository may also include independently developed mods and experimental projects.

This repository serves as a personal development workspace and a publicly accessible collection of RimWorld modding projects.

## Development Scope

The repository supports several categories of mod development:

- **Compatibility Patches:** Resolve conflicts, behavioral inconsistencies, and compatibility issues between existing mods.
- **Combined & Replacement Mods:** Consolidate overlapping functionality from multiple mods into unified implementations, reducing redundancy, conflicts, or unnecessary overhead where possible.
- **Maintenance & Modernization:** Maintain, update, or adapt existing mods for newer RimWorld versions when permitted by their original licenses or authors.
- **Performance Optimization:** Investigate and address inefficient mod behavior, unnecessary processing, and performance-related compatibility problems.
- **Original & Experimental Mods:** Develop new features, mechanics, and experimental modifications independently of existing projects.

Individual projects may serve multiple purposes and are not necessarily restricted to a single category.

## Supported Game Versions

**Primary Development Target:** RimWorld 1.7

Initial development targets RimWorld 1.7, including its experimental development period and eventual stable release.

Compatibility with earlier or later game versions is not guaranteed and will be determined independently for each mod.

## Repository Organization

The repository uses a modular, project-oriented structure.

**Primary directories:**

- `mods/compatibility/` — Compatibility patches and conflict-resolution projects.
- `mods/combined/` — Consolidated or replacement mod implementations.
- `mods/maintenance/` — Maintained, updated, or modernized existing mods.
- `mods/original/` — Independently developed and experimental mods.

Each mod resides in its own subdirectory and maintains its own relevant project files.

**Individual mod directories may contain:**

- `About/` — RimWorld mod metadata and configuration.
- `Defs/` — XML-based game definitions.
- `Patches/` — XML patch operations.
- `Source/` — C# source code and project configuration.
- `Assemblies/` — Compiled assemblies when applicable.
- `Textures/` — Mod-specific graphical assets.
- `docs/` — Technical documentation, development notes, and references.
- `README.md` — Mod overview, installation instructions, dependencies, and compatibility information.
- `LICENSE` — Licensing terms applicable to the individual mod.

Not every mod requires every directory. XML-only patches, C# assemblies, and hybrid implementations may use different structures.

Repository-level configuration files, including `.gitignore`, workspace configuration, and general documentation, are maintained at the repository root.

## Development Environment

Development primarily uses C#, .NET, XML, and the RimWorld modding framework and APIs.

The repository is intended to support lightweight development environments, including Visual Studio Code.

Each mod requiring compiled C# code maintains its own `.csproj` and associated source files within its individual project directory.

A repository-wide Visual Studio solution is not required.

Build requirements, framework targets, dependencies, and development instructions may differ between individual projects.

## Licensing

The repository includes a root-level GNU General Public License version 3.0 (GPL-3.0) governing original repository content unless otherwise specified.

Individual mods may contain their own licensing terms, identified in their respective directories.

The following principles apply:

- Each mod's licensing requirements must be documented independently.
- Existing mods and externally sourced code retain their original copyright and licensing obligations.
- Modifications, redistributions, and derivative works must comply with applicable upstream licenses and permissions.
- Independently developed projects may use different compatible or otherwise appropriate licenses where legally permissible.
- Third-party content is not automatically relicensed under the repository's root license.

Consult the relevant mod's licensing documentation before modifying, redistributing, or incorporating its contents into another project.

## Project Status

This repository is an ongoing development workspace.

Projects may be experimental, incomplete, under active maintenance, or developed primarily for personal use.

Individual mods may have different levels of stability, documentation, support, and public availability.

## Disclaimer

This repository is an independent community modding project and is not affiliated with or endorsed by Ludeon Studios.

RimWorld and related intellectual property belong to their respective owners.

Software and other materials are provided without warranty, subject to their applicable license terms.
