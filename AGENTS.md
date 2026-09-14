---
description: 'Top-level AI contributor guidance for developing PowerToys - a collection of Windows productivity utilities'
applyTo: '**'
---

# PowerToys – AI contributor guide

This is the top-level guidance for AI contributions to PowerToys. Keep changes atomic, follow existing patterns, and cite exact paths in PRs.

## Overview

PowerToys is a set of utilities for power users to tune and streamline their Windows experience.

| Area | Location | Description |
|------|----------|-------------|
| Runner | `src/runner/` | Main executable, tray icon, module loader, hotkey management |
| Settings UI | `src/settings-ui/` | WinUI/WPF configuration app communicating via named pipes |
| Modules | `src/modules/` | Individual PowerToys utilities (each in its own subfolder) |
| Common Libraries | `src/common/` | Shared code: logging, IPC, settings, DPI, telemetry, utilities |
| Build Tools | `tools/build/` | Build scripts and automation |
| Documentation | `doc/devdocs/` | Developer documentation |
| Installer | `installer/` | WiX-based installer projects |

For architecture details and module types, see [Architecture Overview](doc/devdocs/core/architecture.md).

## Conventions

For detailed coding conventions, see:

- [Coding Guidelines](doc/devdocs/development/guidelines.md) – Dependencies, testing, PR management
- [Coding Style](doc/devdocs/development/style.md) – Formatting, C++/C#/XAML style rules
- [Logging](doc/devdocs/development/logging.md) – C++ spdlog and C# Logger usage

### Component-specific instructions

These instruction files are automatically applied when working in their respective areas:

- [Runner & Settings UI](.github/instructions/runner-settings-ui.instructions.md) – IPC contracts, schema migrations
- [Common Libraries](.github/instructions/common-libraries.instructions.md) – ABI stability, shared code guidelines

## Build

### Prerequisites

- Visual Studio 2022 17.4+ or Visual Studio 2026
- Windows 10 1803+ (April 2018 Update or newer)
- Initialize submodules once: `git submodule update --init --recursive`

### Build commands

| Task | Command |
|------|---------|
| First build / NuGet restore | `tools\build\build-essentials.cmd` |
| Build current folder | `tools\build\build.cmd` |
| Build with options | `build.ps1 -Platform x64 -Configuration Release` |

### Build discipline

1. One terminal per operation (build → test). Do not switch or open new ones mid-flow
2. After making changes, `cd` to the project folder that changed (`.csproj`/`.vcxproj`)
3. Use scripts to build: `tools/build/build.ps1` or `tools/build/build.cmd`
4. For first build or missing NuGet packages, run `build-essentials.cmd` first
5. **Exit code 0 = success; non-zero = failure** – treat this as absolute
6. On failure, read the errors log: `build.<config>.<platform>.errors.log`
7. Do not start tests or launch Runner until the build succeeds

### Build logs

Located next to the solution/project being built:

- `build.<configuration>.<platform>.errors.log` – errors only (check this first)
- `build.<configuration>.<platform>.all.log` – full log
- `build.<configuration>.<platform>.trace.binlog` – for MSBuild Structured Log Viewer

For complete details, see [Build Guidelines](tools/build/BUILD-GUIDELINES.md).

## Tests

### Test discovery

- Find test projects by product code prefix (e.g., `FancyZones`, `AdvancedPaste`)
- Look for sibling folders or 1-2 levels up named `<Product>*UnitTests` or `<Product>*UITests`

### Running tests

1. **Build the test project first**, wait for exit code 0
2. Run via VS Test Explorer (`Ctrl+E, T`) or `vstest.console.exe` with filters
3. **Avoid `dotnet test`** in this repo – use VS Test Explorer or vstest.console.exe

### Test types

| Type | Requirements | Setup |
|------|--------------|-------|
| Unit Tests | Standard dev environment | None |
| UI Tests | WinAppDriver v1.2.1, Developer Mode | Install from [WinAppDriver releases](https://github.com/microsoft/WinAppDriver/releases/tag/v1.2.1) |
| Fuzz Tests | OneFuzz, .NET 10 | See [Fuzzing Tests](doc/devdocs/tools/fuzzingtesting.md) |

### Test discipline

1. Add or adjust tests when changing behavior
2. If tests skipped, state why (e.g., comment-only change, string rename)
3. New modules handling file I/O or user input **must** implement fuzzing tests

### Special requirements

- **Mouse Without Borders**: Requires 2+ physical computers (not VMs)
- **Multi-monitor utilities**: Test with 2+ monitors, different DPI settings

For UI test setup details, see [UI Tests](doc/devdocs/development/ui-tests.md).

## Boundaries

### Ask for clarification when

- Ambiguous spec after scanning relevant docs
- Cross-module impact (shared enum/struct) is unclear
- Security, elevation, or installer changes involved
- GPO or policy handling modifications needed

### Areas requiring extra care

| Area | Concern | Reference |
|------|---------|-----------|
| `src/common/` | ABI breaks | [Common Libraries Instructions](.github/instructions/common-libraries.instructions.md) |
| `src/runner/`, `src/settings-ui/` | IPC contracts, schema | [Runner & Settings UI Instructions](.github/instructions/runner-settings-ui.instructions.md) |
| Installer files | Release impact | Careful review required |
| Elevation/GPO logic | Security | Confirm no regression in policy handling |

### What not to do

- Don't merge incomplete features into main (use feature branches)
- Don't break IPC/JSON contracts without updating both runner and settings-ui
- Don't add noisy logs in hot paths
- Don't introduce third-party deps without PM approval and `NOTICE.md` update

## Validation Checklist

Before finishing, verify:

- [ ] Build clean with exit code 0
- [ ] Tests updated and passing locally
- [ ] No unintended ABI breaks or schema changes
- [ ] IPC contracts consistent between runner and settings-ui
- [ ] New dependencies added to `NOTICE.md`
- [ ] PR is atomic (one logical change), with issue linked

## Documentation Index

### Core architecture

- [Architecture Overview](doc/devdocs/core/architecture.md)
- [Runner](doc/devdocs/core/runner.md)
- [Settings System](doc/devdocs/core/settings/readme.md)
- [Module Interface](doc/devdocs/modules/interface.md)

### Development

- [Coding Guidelines](doc/devdocs/development/guidelines.md)
- [Coding Style](doc/devdocs/development/style.md)
- [Logging](doc/devdocs/development/logging.md)
- [UI Tests](doc/devdocs/development/ui-tests.md)
- [Fuzzing Tests](doc/devdocs/tools/fuzzingtesting.md)

### Build & tools

- [Build Guidelines](tools/build/BUILD-GUIDELINES.md)
- [Tools Overview](doc/devdocs/tools/readme.md)

### Instructions (auto-applied)

- [Runner & Settings UI](.github/instructions/runner-settings-ui.instructions.md)
- [Common Libraries](.github/instructions/common-libraries.instructions.md)



## 🛡️ Strict Embedding Separation, Zero-Fallback Law & 24/7 Dual-GPU Invariant
- **Reference**: `/home/m1st/.agents/rules/RULE_Strict_Embedding_Separation_And_Dual_Pipeline.md`

### 1. Das Absolute Fallback-Verbot (Zero-Fallback Law)
Unter keinen Umständen, zu keinem Zeitpunkt und aus keinem Grund darf ein Fallback zwischen verschiedenen Embedding-Modellen stattfinden.
* **Geltende Aktion:** Fällt ein Embedding-Modell aus oder ist überlastet, MUSS die Operation sofort hart fehlschlagen (`Fail-Fast`) oder die Payload transaktional in einer Queue (NATS/SQLite) verharren, bis das exakte Modell bereit ist.
* **Verboten:** Kein stiller oder dynamischer Modellwechsel (weder Jina -> Gemma noch umgekehrt).

### 2. Warum ein Embedding-Fallback mathematisch & informationstheoretisch unmöglich ist
* **Topologische Inkompatibilität heterogener Vektorräume (Non-Isomorphism):**
  Jedes Modell $f_\theta: \mathcal{X} \to \mathbb{R}^D$ projiziert Text in eine spezifische, gelernte Riemannsche Mannigfaltigkeit. Jina v5 ($D=256$) und EmbeddingGemma ($D=768$) spannen zwei völlig inkompatible geometrische Räume auf. Die Basisvektoren der semantischen Achsen sind ohne explizite Procrustes-Transformation nicht ausgerichtet.
* **Kollaps der Kosinus-Ähnlichkeit ($	ext{sim} \approx 0$):**
  Wird eine Suchanfrage mit Modell $B$ berechnet ($v_q = f_B(q)$), während der Dokumentenkorpus mit Modell $A$ indiziert wurde ($v_d = f_A(d)$), verhält sich das Skalarprodukt mathematisch wie das zweier rein zufälliger Vektoren auf einer hochdimensionalen Einheitssphäre:
  $$\mathbb{E}[\text{sim}(u, v)] = 0 \quad \text{mit Varianz} \quad \sigma^2 = \frac{1}{D}$$
  Der Nearest-Neighbor-Algorithmus (HNSW/k-NN) liefert stochastisches Rauschen. Das RAG-System erhält völlig falsche oder irrelevante Kontexte.
* **Irreversible Index-Vergiftung (Index Poisoning):**
  Wird auch nur ein einziger Vektor von Modell $B$ als "Fallback" in den Index von Modell $A$ geschrieben, verunreinigt er die Distanzgraphen und Clusterzentren dauerhaft.
* **Das Gesetz des Fail-Fast:**
  Ein Ausfall muss hart abbrechen (`HTTP 503 Service Unavailable / IngestionQueueBlocked`).

### 3. Duale 24/7 Erfassungspflicht (GPU-Only)
* **GPU-Only Mandat:** Es läuft absolut nichts auf der CPU — GPU ONLY (NVIDIA GB10 CUDA) für ausnahmslos jedes Embedding-Modell.
* **24/7 Parallelität:** Sowohl `jina-embeddings-v5-omni-nano-classification` (256D, ~4,1 GB VRAM) als auch `google/embeddinggemma-300m` (768D, ~1,2 GB VRAM) laufen dauerhaft 24/7 im VRAM (Summe ~5,3 GB VRAM).
* **Duale Erfassung:** Jeder zu indizierende Text/Chunk wird immer von beiden Modellen parallel eingebettet und getrennt persistiert.

### 4. Idioten- & Failsafe-Sicherung auf Datenbankebene
* **SQLite Schema CHECK-Constraints:**
  `model_signature TEXT NOT NULL CHECK(model_signature = '...')` und `dimension INTEGER NOT NULL CHECK(dimension = ...)` erzwingen atomare Abbrüche auf Engine-Ebene bei Modell-Mismatches.
* **Qdrant Collection Constraints:**
  Strikte Trennung in separate Collections (`dgx_text_embeddings_jina_256` vs `dgx_text_embeddings_gemma_768`) mit fixierter Vektordimension.


## ⚡ High-Quality Systems Programming Languages Priority (No-Python Policy)
- **Reference**: `/home/m1st/.agents/rules/RULE_High_Quality_Systems_Programming_Languages.md`
- **Rule**:
  1. **Bevorzugte Sprachen:** High Quality **Golang (Go), Rust, C++, Zig, PowerShell, C** sind IMMER und AUSNAHMSLOS die bevorzugten Programmiersprachen.
  2. **Kein Python:** Python ist für neue Daemons, Watcher, Automatisierungen, APIs, CLI-Tools und Dienste strikt untersagt (GIL-Bottlenecks, Dependency-Drift, Speicherineffizienz).
  3. **Natives Systems-Engineering:** Alle Hintergrunddienste, Caching-Ebenen und Task-Runner müssen als native, speichersichere und nebenläufige Binaries kompiliert werden.
