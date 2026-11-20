# RimWorld Solution Manager — Concept & Future Development Handoff

**Repository:** ThelianTech `RimworldDEV`  
**Project:** *RimWorld Solution Manager* (working name; final name not selected)  
**Document type:** Concept baseline, decision register, and new-chat planning handoff  
**Original concept recorded:** 2026-11-19  
**Revision:** 2026-11-19 — WPF single-project architecture and future-chat protocol incorporated  
**Implementation status:** **Backlog / not implemented**  
**Development approach:** Concept-first; explicit decision approval before technical design or implementation  
**Related documents:** `RimWorld-Repository-Restructure-Proposal.md`; `RimWorld-Repository-Architecture-Plan.md`; `Rw-S-S.md`; `New-RimWorldSolution.ps1`

> **Read this first in a future chat:** This is an **initial, approved-direction concept**, not a completed technical specification or evidence of a working application. Preserve the settled requirements, distinguish proposals from verified repository facts, review open decisions with the user in order, and **do not start implementing code or generating Codex execution prompts until the architecture and implementation specifications have been reviewed and approved**. The existing PowerShell workflow remains the fallback until a tested application explicitly replaces it.

---

## 1. Purpose and product identity

Develop a **small internal Windows desktop utility** that replaces repetitive PowerShell operations for creating Visual Studio `.slnx` files and selecting which existing C# projects they contain. A specialized solution is a **development initiative/workspace**, not a mod pack, merge operation, or separate Git repository.

The utility should eventually support a second workflow: creating a new RimWorld 1.7 C# mod project in the repository's Mods source domain, registering it with the Mods solution, and **optionally** attaching it to a chosen specialized solution. The user is intentionally maintaining different solutions for different work contexts; the utility must not turn those contexts into an automatically synchronized hierarchy.

### Primary outcomes

1. Create `Projects/<ChosenName>.slnx` directly in the repository-root `Projects/` folder, **without a solution-named subfolder**.
2. Show the existing mod and utility C# projects in an interactive directory browser or tree.
3. Let the user choose **zero, one, or multiple** existing `.csproj` projects using mouse and keyboard.
4. Preview the exact project references and file operations; create the solution only after explicit confirmation.
5. Preserve one physical copy of each project while allowing it to appear in many independent solutions.
6. Later, scaffold a new RimWorld 1.7 mod and add it to `src/Mods/Mods.slnx`; add it to a specialized solution **only if expressly selected**.

### Non-goals

The Solution Manager does **not** compile, merge, profile, test inside RimWorld, validate gameplay compatibility, package releases, distribute third-party assets, or infer which projects belong to the master solution. Other future Utilities projects may do some of those jobs.

---

## 2. Decision register — what is settled versus still open

Treat **Locked concept** as approved product direction; treat **Proposed**, **Pending verification**, and **Deferred** as *not* implementation approvals.

| Topic | Status | Baseline or current position |
|---|---|---|
| One Git repository | **Locked concept** | All mod and utility source projects stay within `RimworldDEV`; no duplicated repositories required. |
| Solution model | **Locked concept** | Independent `.slnx` files reference existing `.csproj` files; a solution is not functionally nested inside another solution. |
| Domain solutions | **Locked concept** | Mods projects belong in the Mods solution; internal utility projects belong in the Utilities solution. |
| Specialized solutions | **Locked concept** | `Projects/` stores named initiative-specific `.slnx` files containing manually selected project references. |
| Solution membership | **Locked concept** | No auto-synchronization to the repository master, other initiative solutions, or every domain solution. User selection governs membership. |
| Interactive application technology | **Locked initial direction** | **Minimal WPF desktop application** for Windows, written in C#. A console frontend is **not** required for the initial release. |
| Application project topology | **Locked initial direction** | **One WPF `.csproj`**, with UI, ViewModels, services, models, and infrastructure in that same project/assembly. No separate reusable core class library initially. |
| Logical architecture | **Locked initial direction** | Lightweight **MVVM** and clear separation of concerns *within* the WPF project; keep service logic independent of WPF types where practical. |
| Target framework | **Locked initial direction** | WPF utility: `net10.0-windows`. New RimWorld 1.7 mod libraries: `net10.0`. These have separate roles. |
| RimWorld support | **Locked concept** | New mods target RimWorld 1.7 onward/Waylith. The utility does not promise 1.6 backwards compatibility. |
| Framework vs SDK policy | **Locked concept** | No `global.json` requirement. The target framework does not select or pin the installed .NET SDK. |
| New mod defaults | **Planned dependency** | `src/Mods/Directory.Build.props`, `Directory.Build.targets`, `Directory.Packages.props` will establish RimWorld mod defaults; they are not confirmed implemented. |
| Current repository migration | **Proposed / pending verification** | Move `Directory/Mods` and `Directory/Utilities` to `src/Mods` and `src/Utilities`; move `Directory/Projects` to repository-root `Projects`. Do not presume this move already happened. |
| Existing PowerShell tool | **Locked transition rule** | Retain `New-RimWorldSolution.ps1` as the fallback until the WPF replacement is validated; update its paths if migration occurs. |
| Publishing/distribution | **Deferred** | An internal `.exe` is desired. Framework-dependent vs self-contained and single-file publishing remain packaging decisions. |
| Future reuse | **Deferred trigger** | Extract services into a class library only when another frontend/utility genuinely requires the shared assembly or the boundary proves useful. |

**Decision discipline:** A recommendation is not a locked decision. A code sketch is not evidence that a project has been implemented. New conversations should not silently revise these statuses.

---

## 3. Repository topology and existing realities

### 3.1 Last directly observed physical layout

Screenshots from 2026-11-19 showed this baseline:

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

The existence or relocation of additional `.slnx`, `.csproj`, MSBuild configuration files, or scripts on the user's **local machine** is **not verified** by those screenshots. Files provided as downloads should not be assumed to have been placed in the repository. A future chat should inspect the actual current tree or user-provided files before forming an implementation plan.

### 3.2 Proposed target topology

```text
RimworldDEV/
├── ThelianTech_Rimworld-Mods.slnx     # Root master solution; independently curated
├── Projects/                          # Specialized initiative solutions, not .csproj source
│   ├── PawnSystemsOverhaul.slnx       # Illustrative, not confirmed present
│   └── ColonyQoL.slnx                 # Illustrative, not confirmed present
├── src/
│   ├── README.md
│   ├── Mods/
│   │   ├── Mods.slnx
│   │   ├── Directory.Build.props       # Planned
│   │   ├── Directory.Build.targets     # Planned
│   │   ├── Directory.Packages.props    # Planned
│   │   └── ExampleMod/                # Illustrative
│   │       ├── ExampleMod.csproj
│   │       ├── About/
│   │       ├── Source/
│   │       └── 1.7/
│   └── Utilities/
│       ├── Utilities.slnx
│       ├── New-RimWorldSolution.ps1    # Legacy fallback; must be path-updated
│       ├── Rw-S-S.md                   # Legacy usage guide; must be path-updated
│       └── RimWorldSolutionManager/    # Future WPF app root
│           └── RimWorldSolutionManager.csproj
├── Wiki-Docs/
│   ├── RimWorld-Solution-Manager-Concept-Plan.md
│   └── RimWorld-Repository-Restructure-Proposal.md
├── .gitignore
├── LICENSE
└── README.md
```

**Important:** `Projects/` is a folder of **development initiative solutions**. The actual mod and utility `.csproj` source lives under `src/`. Being part of a solution does not change a project's physical location, NuGet inheritance, runtime target, or publishing policy.

### 3.3 Required migration coordination

The existing PowerShell script was written for `Directory/Utilities/` and previously resolved all three target areas under `Directory/`. After the proposed move to `src/Utilities/`, it must instead route:

```text
Default / -Area Projects  -> RimworldDEV/Projects/<Name>.slnx
-Area Mods                -> RimworldDEV/src/Mods/<Name>.slnx
-Area Utilities           -> RimworldDEV/src/Utilities/<Name>.slnx
```

Also recheck all relative solution-to-project paths, Visual Studio/VS Code references, `.gitignore`, CI/scripts, and documentation. Do not assume relocating the script alone corrects its destination logic. See the separate restructuring proposal for the migration checklist.

---

## 4. Architecture decision — a single WPF project

### 4.1 Decision and rationale

**Use one WPF C# project for the initial application.** The service layer does **not** require a separate class library. Internal reuse and testability come from logical boundaries and interfaces, not necessarily from multiple compiled assemblies.

Reasons:

- The application is an internal Windows-only tool primarily serving this repository.
- One `.csproj` keeps project setup, builds, dependency management, and delivery straightforward.
- No independent console frontend or separately consumable service API is required at launch.
- Services can remain plain C# classes with no WPF dependencies, allowing later extraction if truly needed.
- An additional DLL/project boundary should solve a demonstrated requirement, not be introduced preemptively.

A typical framework-dependent WPF build can still produce a Windows executable, a .NET application DLL, and supporting runtime/configuration files. **One `.csproj` does not mean one physical output file.** Single-file publishing may be evaluated later; it is not required by the concept.

### 4.2 Recommended logical folders — all inside one `.csproj`

```text
src/Utilities/RimWorldSolutionManager/
├── RimWorldSolutionManager.csproj
├── App.xaml
├── App.xaml.cs
├── Views/
│   ├── MainWindow.xaml               # Could instead live at project root
│   └── MainWindow.xaml.cs
├── ViewModels/
│   ├── MainViewModel.cs
│   └── ProjectSelectionViewModel.cs   # Illustrative
├── Services/
│   ├── SolutionService.cs            # Illustrative
│   ├── ProjectDiscoveryService.cs    # Illustrative
│   └── ModProjectCreationService.cs  # Future capability; illustrative
├── Infrastructure/
│   ├── DotNetCliRunner.cs            # Illustrative
│   └── RepositoryLocator.cs          # Illustrative
└── Models/
    ├── ProjectEntry.cs               # Illustrative
    └── SolutionPlan.cs               # Illustrative
```

These are **suggested responsibilities and names**, not approved class contracts or a mandate to create an empty file for each entry.

### 4.3 Architectural boundaries

```text
WPF View (XAML / input / presentation)
       ↓
ViewModel (UI state, commands, selection, progress)
       ↓
Application Services (validation, discovery, solution operations)
       ↓
Infrastructure (filesystem, dotnet CLI / MSBuild execution, process output)
```

- **Views:** display data and receive user actions; avoid `dotnet` commands or filesystem orchestration in code-behind.
- **ViewModels:** coordinate interactive state and invoke application services; do not know `.slnx` serialization internals.
- **Services:** enforce solution/project workflows and rules; keep public contracts free from WPF-specific types where practical.
- **Infrastructure:** locate repository, execute external processes, enumerate files, and report observable outcomes.
- **Models:** hold validated inputs, discovered project metadata, and planned operations. Define concrete types after reviewing workflow needs.

Dependencies should flow toward application behavior, not from domain services back into windows or XAML. Constructor injection or a simple composition root is sufficient initially; a DI package is **optional**, not a mandatory decision.

### 4.4 Initial project configuration — illustrative, not finalized

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net10.0-windows</TargetFramework>
    <UseWPF>true</UseWPF>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
</Project>
```

**Do not** apply `Krafs.Rimworld.Ref` or RimWorld mod-specific build targets to the WPF project merely because it manages mod projects. Those belong to `.csproj` files physically under `src/Mods/`. Dependencies and tooling for the WPF app should be approved based on its own needs.

### 4.5 When to reconsider a library

A separate `RimWorldSolutionManager.Core` class library is **not** in the initial design. Reconsider only if the service layer is needed by a second console frontend, a separate utility, an independently versioned component, or if testing/maintenance demonstrates an actual physical assembly boundary would help. Extraction should preserve behavior and require its own review and approval.

---

## 5. Core user workflows and acceptance criteria

### Workflow A — Create a specialized solution with selected existing projects (MVP)

1. Identify or request the repository root; verify the expected locations exist **in the current layout**.
2. Enter the new solution name, such as `PawnSystemsOverhaul`.
3. Validate the name, destination, duplicate filenames, case-insensitive collisions on Windows, and invalid path segments.
4. Preview exactly `Projects/<Name>.slnx`—not `Projects/<Name>/<Name>.slnx`.
5. Browse actual `.csproj` files under the physical Mods and Utilities domains; support directory navigation and mouse/keyboard selection.
6. Choose **zero, one, or multiple** projects, including projects from both domains.
7. Display the planned solution path and selected project paths; require explicit confirmation.
8. Create the blank `.slnx`; register only the selected projects; show operation outcome and errors.
9. Allow an optional action to open the resulting solution in Visual Studio.

**Acceptance:** The solution opens, references only the selected projects, and contains no copied/moved source or extra solution-named directory. Master and domain solution membership are unchanged.

### Workflow B — Create an intentionally blank solution (MVP)

- Use the same flow but select **zero** projects.
- Create `Projects/<Name>.slnx` and nothing else other than expected SDK/process side effects.
- Keep later manual registration through Visual Studio possible.

### Workflow C — Create a new mod project (later phase)

1. Enter and validate a mod/project name, namespace, and only such metadata as the approved template requires.
2. Create a C# class library under `src/Mods/<ModName>/<ModName>.csproj` targeting RimWorld 1.7 / `net10.0`.
3. Apply the repository's actual (verified) `src/Mods/` inherited MSBuild/NuGet settings; do not duplicate them in every generated project without reason.
4. Generate only the standardized RimWorld mod folders and files agreed during technical design.
5. Register the new project in **`src/Mods/Mods.slnx`**.
6. Optionally include the new mod project in **one explicitly chosen** `Projects/*.slnx` initiative if the user elects to.
7. Never automatically add it to the root master solution, every specialized solution, or unrelated utility solutions.
8. Preview filesystem and solution edits; present success and partial-failure reports.

**Acceptance:** One physical C# project exists in `src/Mods/`, appears in `Mods.slnx`, and appears in any custom initiative only if the user requested it.

### Workflow D — Manage existing initiative solutions (later phase)

- List existing `Projects/*.slnx`; show registered projects and missing references.
- Add or remove **solution references** with confirmation; never delete a project's source by removing it from a solution.
- Handle rename/move or broken path reports as a separate reviewed capability, not silent auto-repair.

### Workflow E — Cross-tool integration (deferred)

- Future build, testing, validation, and packaging tools may receive selected project or solution paths.
- This app is **not itself** the release orchestrator and does not certify a successful RimWorld test.

---

## 6. Interaction and usability requirements

The initial user interface should be **minimal rather than feature-heavy**, but the following matter:

- Solution-name entry, repository/destination preview, and an interactive project browser.
- Expand/collapse project directories; select/deselect projects without losing the current selections.
- Mouse operation and keyboard access (including practical navigation and toggling).
- Selected-project list, selected count, and a final review screen or confirmation state.
- Useful validation errors and actionable results; disable unsupported/destructive actions.
- Zero-project solutions are valid.
- Do not create or overwrite files without explicit approval.
- Avoid deleting or moving existing projects; solution references are distinct from physical project files.
- Avoid blocking the UI while scanning or running SDK commands; cancellation/progress strategy remains to be designed.
- Preserve actual CLI error details for troubleshooting while surfacing concise user-readable messages.

**Interface choice locked:** WPF. **Not yet locked:** exact window layout, control composition, tree behavior, MVVM toolkit, async command model, theming, and detailed validation/error presentation.

---

## 7. Services, execution strategy, and safeguards — design topics to finalize

### 7.1 Intended capabilities

| Component / responsibility | Expected behavior | Status |
|---|---|---|
| Repository discovery | Identify correct Git repository and `src/` / `Projects/` locations; handle pre-migration layout intentionally. | Proposed contract |
| Project discovery | Enumerate existing `.csproj` files under Mods and Utilities; ignore `bin`, `obj`, `.git`, temporary and generated paths. | Proposed contract |
| Input/path policy | Validate names and destinations; prevent traversal outside intended folders and accidental overwrites. | Required behavior, contract pending |
| Solution operations | Create, list, add, and eventually remove project membership using supported tooling. | Required behavior, implementation pending |
| External-process runner | Execute the .NET CLI with correctly separated arguments; capture exit codes, stdout/stderr, and cancellation. | Recommended technique |
| Operation preview | Describe planned filesystem and `.slnx` changes before confirmation. | Required behavior, contract pending |
| Mod scaffolding | Later generate compliant `net10.0` class libraries in the Mods domain. | Deferred feature |
| Audit/logging | Describe success, failure, and partial completion accurately. | Required behavior, level pending |

### 7.2 CLI baseline

For the current manual workflow, supported commands include:

```powershell
# From repository root AFTER the proposed relocation (do not assume it is complete)
dotnet new sln --name PawnSystemsOverhaul --format slnx --output .\Projects

dotnet sln .\Projects\PawnSystemsOverhaul.slnx add .\src\Mods\ExampleMod\ExampleMod.csproj
```

The WPF tool may orchestrate the supported `dotnet` commands, subject to technical verification, or use a validated library. **Do not manually rewrite `.slnx` XML without first evaluating supported APIs and format semantics.** Prefer safe process invocation with structured argument lists, not shell-string concatenation.

### 7.3 Non-negotiable safeguards

- Explicitly selected projects **only**; no automatic master-solution sync.
- No silent overwrite of existing `.slnx` or `.csproj` files.
- No copying entire project trees solely to add solution membership.
- No modification to unrelated solutions.
- No auto-Git commit, automatic third-party code import, or project merger.
- Report partial outcomes: e.g., created solution but failed adding one project.
- Plan recovery/idempotence rules for partial failures during technical design.
- A custom solution is **not** a deployable combined mod; a separate integration `.csproj` would implement actual merged functionality.

---

## 8. Incremental development roadmap (planning baseline only)

**Phase 0 — Prerequisites and audit**

- Verify repository's actual tree and all relevant `.slnx`/`.csproj` files.
- Confirm whether `Directory/` -> `src/` and root `Projects/` migration has occurred.
- Verify/repair legacy PowerShell script documentation and paths before depending on it.
- Verify Visual Studio/.NET SDK compatibility, the `.slnx` CLI operations, and initial WPF tooling.
- Audit existing utility projects to avoid duplicating implemented logic.

**Phase 1 — Approved architecture and service contracts**

- Define root discovery, project discovery, names/paths, command execution, transaction/partial-failure behavior, and file/solution operations.
- Make explicit decisions about interfaces, tests, error handling, and minimal UI data models.
- Document accepted architecture; do not treat the illustrative classes as mandatory.

**Phase 2 — WPF MVP: solution creation**

- Create the single `net10.0-windows` WPF project.
- Implement lightweight MVVM and service boundaries in that same `.csproj`.
- Support blank solution creation, interactive existing-project browsing/selection, preview/confirm, and reliable `.slnx` project registration.
- Verify no unintended source copies or solution-membership changes.

**Phase 3 — Mod project creation**

- Create new `net10.0` mod class libraries under `src/Mods/`.
- Apply verified shared RimWorld 1.7 build configuration and approved mod template.
- Register with `Mods.slnx`; allow **explicit opt-in** to an initiative solution.

**Phase 4 — Existing solution management**

- Inspect initiative solution memberships; safely add/remove references, report missing paths, and improve discovery/search.

**Phase 5 — Optional integration & packaging**

- Consider launching Visual Studio, bridging to separate build/validation tools, and optional single-file/self-contained publishing **only after MVP stability**.

No implementation phases have been started or completed merely because they appear here. Phases and MVP boundaries remain subject to formal review.

---

## 9. Open decisions for the dedicated planning chat

Review these deliberately; do **not** ask the user to reconfirm locked decisions unless a real conflict or new evidence demands it.

1. Final application name, namespace, and assembly name.
2. Actual current repository layout, root-discovery approach, and whether to support pre-migration paths.
3. Project browser UX: tree, filters, search, checkbox selection, duplicate handling, and keyboard behavior.
4. Source-of-truth for project discovery: filesystem `.csproj` enumeration, solution lists, or a combination.
5. Supported `.slnx` creation/list/add/remove method: `dotnet` CLI first versus an appropriate library.
6. Validation and error recovery: atomicity expectations, partial failure, retries, logging, cancellation.
7. Lightweight MVVM implementation choice: built-in WPF primitives versus a toolkit; DI/composition approach.
8. Exact models/interfaces and dependency boundaries, to be specified after workflows are approved.
9. Mod project scaffolding: custom `.NET` template versus application-owned generator; initial `About/` and source structure.
10. Testing approach: pure-service tests, potential separate **test project** (distinct from a reusable service library), and verification fixtures.
11. Whether project selection may include `.csproj` files outside the repository (default proposal: restricted to Mods/Utilities).
12. Packaging and distribution of the internal WPF utility (framework-dependent, self-contained, single-file) and any update strategy.
13. Whether additional initiative manifests are useful or unnecessary for this tool's responsibilities.

Do not expand into game-mod merging, save-file compatibility, runtime pathfinding, or release certification unless separately requested.

---

## 10. Instructions for a future ChatGPT concept-planning conversation

**This section is intended to be read and followed when the user uploads this Markdown file in a new, dedicated chat.** It carries forward the user's established concept-first, decision-driven planning process used for other software projects (including CalradiaForge and modding work).

### 10.1 How to initialize the future conversation

1. **Read this entire document before advising.** If available, also read the repository restructure proposal, architecture plan, and any actual `.slnx`, `.csproj`, scripts, or source files the user supplies.
2. **State the product identity and status accurately:** internal Windows WPF utility, one `.csproj`, lightweight MVVM, no separate service library initially, concept/backlog rather than implementation.
3. **Produce a compact status inventory** with four explicit groups: **Locked Decisions**, **Verified Current State**, **Proposed or Unverified Design**, and **Open Decisions**. Do not collapse these categories.
4. **Check for new evidence or contradictions** between the uploaded files and these decisions. The user's actual current repository/codebase is authoritative for *what exists*, and this document is the baseline for *intended behavior*.
5. **Ask only targeted unresolved questions**; do not re-ask decisions already established or assume that a proposed path migration was performed.
6. **Do not immediately generate an entire codebase, adopt a UI framework, split projects, or issue a Codex implementation prompt.** Start by resolving architecture and UX questions.

### 10.2 Required planning method

Use **sequential, decision-by-decision reviews**, rather than dumping many speculative design choices into a single undifferentiated plan. For each architectural topic:

1. Define the topic and the behavior it must provide.
2. Cite the **approved concept baseline** and any **verified existing code or file structure** relevant to it.
3. Identify conflicts, assumptions, dependencies, edge cases, and failure/recovery paths.
4. Present credible alternatives with engineering tradeoffs, including maintenance/testability and future extensibility.
5. Give a clearly identified recommendation and, where useful, an illustrative design or focused code sketch. **Label proposals as proposals.**
6. Define acceptance criteria and the evidence/tests that would validate the decision.
7. Seek the user's **explicit approval or refinement** before marking the decision locked.
8. Update a decision ledger so later discussions do not contradict or re-litigate established decisions without new evidence.

**Important:** Recommendations by the assistant are **not approvals**. A deferred question remains deferred until the user resolves it. Do not silently convert a suggestion, example class name, or draft code snippet into a requirement.

### 10.3 Suggested topic order

1. Verify actual repository paths and current script/solution state.
2. Finalize MVP scope and the create-solution user journey.
3. Finalize project browser/selection UI and confirmation behavior.
4. Establish path policies, project discovery, `.slnx` management, and process execution.
5. Resolve partial failure, cancellation, logging, and recovery.
6. Specify single-project MVVM composition and concrete service contracts.
7. Specify test strategy and acceptance tests.
8. Specify new-mod scaffolding as a later capability.
9. Review final architecture for contradictions, duplication, scope creep, and missing dependencies.
10. Only then establish phased implementation tasks and any future Codex prompts.

If the user changes the order, follow the user's explicit priorities while preserving the decision ledger.

### 10.4 Required documentation progression

Maintain separate, clearly versioned artifacts as the project evolves:

**A. Concept baseline (this document)**  
What the product must do; architectural direction; constraints; settled versus pending decisions.

**B. Technical architecture and specification (future)**  
Approved modules/classes/contracts, data flow, lifecycle, UI states, CLI interactions, error handling, test contracts, and exact on-disk paths. Ground claims about pre-existing code in an actual repository audit.

**C. Phased implementation project plan (future)**  
Scoped work packages, prerequisites, dependency order, acceptance criteria, validation procedures, regression risks, and documentation updates.

**D. Sequential Codex/implementation prompts (future)**  
Only after the user approves the technical design and phase plan. Prompts must reference verified files, specify boundaries, tests and stop conditions, and avoid unrelated refactors.

**E. Post-implementation verification and as-built documents (future)**  
Record actual implementation decisions, tested outcomes, deviations from the plan, and remaining risks.

When the conversation becomes lengthy, regenerate a **complete, consolidated Markdown baseline** separating locked decisions, open decisions, rejected or superseded ideas, and verified implementation state. Use it as the handoff for any subsequent chat rather than relying on conversational memory alone.

### 10.5 Source-of-truth and research rules

- **For intended scope and constraints:** use this current approved concept and the user's newer explicit decisions.
- **For actual implementation:** inspect the real files/repository before claiming a service, project, or feature exists.
- **For framework/tool capabilities:** verify against current official Microsoft/.NET documentation when necessary; clearly distinguish verified facts from inferences.
- **For RimWorld/NuGet compatibility:** verify appropriate current game/ref-package data separately when implementing mod scaffolding; do not assume illustrative version strings will remain valid.
- **For historical artifacts:** preserve old documents as versioned references or explicitly supersede them; do not mix obsolete `Directory/` examples into implemented `src/` instructions.
- **For code changes:** no unapproved, broad refactors, destructive file operations, or claims that local machine files were changed unless actual tools executed and verified those changes.

### 10.6 Expected response when this file is first uploaded

The first response in the dedicated planning chat should, at minimum:

1. Identify **RimWorld Solution Manager** and summarize its intended user workflow in a paragraph.
2. Confirm the locked **single WPF project, `net10.0-windows`, lightweight MVVM, internal service layer** decision.
3. Explain that `src/` and `Projects/` are proposed target paths until their existence is verified.
4. Present a short, precise **decision-status inventory** and name the **first unresolved architectural decision** to examine.
5. Start the architectural discussion rather than producing a full app implementation or repeating the already-settled console-vs-WPF decision.

### 10.7 Copy-paste activation prompt for the future chat

> **RimWorld Solution Manager — Concept Planning Chat**  
> Read the attached `RimWorld-Solution-Manager-Concept-Plan.md` in full and use it as the concept and decision baseline. This is a future internal tool, not an implemented application. We have already selected a single-project .NET 10 WPF application with lightweight MVVM and internal C# services; do not propose a separate class library or console frontend as the default. Preserve the explicitly manual solution-membership policy. First summarize locked decisions, verified facts, proposed/unverified items, and open decisions; inspect any current repository files I provide rather than assuming the proposed directory migration is complete. Then work with me through the remaining architecture and UX decisions one at a time, giving alternatives, tradeoffs, recommendations, acceptance criteria, and approval checkpoints. Do not begin implementation, write a Codex task, or lock unapproved decisions. Once the concept is finalized, help produce an approved technical architecture/specification, a phased implementation plan, and only then implementation prompts and verification documentation.

---

## 11. Risks and boundaries to remember

| Risk / ambiguity | Required handling |
|---|---|
| New solution references multiple mods | Explain clearly that **solution membership does not merge DLLs** or create one distributable mod. |
| Repository layout changes | Resolve actual paths at runtime and audit references; never silently create an obsolete destination. |
| SDK unavailable | Detect/report missing or incompatible `dotnet` tooling before writes where possible. |
| Failed project registration | Report partial outcome; plan safe retry/cleanup behavior during technical design. |
| Cross-domain project selection | Allow intentional inclusion of Mods/Utilities projects while preserving their individual build configuration. |
| Code copied from older mods | Third-party licenses and technical compatibility are separate problems; no automatic copying/importing. |
| Future expansion pressure | Keep the WPF app small. Build/release automation is separate and optional. |
| Single-project misinterpretation | One WPF `.csproj` does not require one source file or one published physical executable. |
| New-app testing | Unit tests could eventually live in a separate **test** project without changing the decision against an extra **service** class library. |

---

## 12. References for future verification

- Microsoft, .NET solution CLI: <https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-sln>
- Microsoft, `.slnx` default in .NET 10: <https://learn.microsoft.com/en-us/dotnet/core/compatibility/sdk/10.0/dotnet-new-sln-slnx-default>
- Microsoft, `Directory.Build.props` / `.targets`: <https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory>
- Microsoft, WPF overview: <https://learn.microsoft.com/en-us/dotnet/desktop/wpf/overview>
- Microsoft, single-file deployment: <https://learn.microsoft.com/en-us/dotnet/core/deploying/single-file/overview>

The links are references for a later current-version verification; their inclusion here does not certify current runtime/package availability or an installed configuration on the user's machine.

---

## 13. Revision record

| Date | Change |
|---|---|
| 2026-11-19 — initial | Captured planned PowerShell replacement, repository/specialized solution workflow, console versus WPF options, and future new-mod scaffolding. |
| 2026-11-19 — regenerated | **Resolved the UI and project-boundary direction:** minimal WPF app, `net10.0-windows`, one WPF `.csproj`, internal logically separated MVVM/services/infrastructure, no initial separate core library. Retained the legacy script fallback and migration caveat. Added decision register, detailed acceptance criteria, phased roadmap, and future-chat initialization/decision-review/handoff instructions. |

**Final baseline statement:** Build **one small internal WPF project** for deliberately creating and managing `.slnx` development workspaces, with mod creation as a later capability. Keep solution membership manual and explicit, preserve the repository's domain separation, and complete the concept/technical design approval process **before implementation**.
