---
name: chsh-dev-project
description: Complete Nordic nRF Connect SDK project workflow — read the PRD, generate technical specs under docs/engineering/, and implement code according to those specs. Use when creating a new NCS project, updating specs after a PRD change, or implementing features from existing specs.
---

# chsh-dev-project — Specs-First Development Workflow

This skill drives NCS project development from PRD to running code via technical specs.
Specs live in `docs/engineering/specs/` and are the single source of truth for implementation.

---

## Step 0 — Detect Context

Before anything else, check what exists:

```bash
ls docs/product/PRD-*.md 2>/dev/null | sort | tail -1        # latest PRD
ls docs/engineering/specs/*.md 2>/dev/null                    # existing specs
ls src/modules/ 2>/dev/null                                   # existing modules
git log --oneline -5                                          # recent commits
```

Choose the mode that fits:

| Condition | Mode |
|---|---|
| No specs exist yet | **A — Generate Specs + Implement** |
| Specs exist, PRD is newer (date changed) | **B — Update Specs**, then optionally implement |
| Specs exist and are current, code is missing/incomplete | **C — Implement from Specs** |
| User explicitly asks to only update specs | **B — Update Specs** |
| User explicitly asks to only implement | **C — Implement from Specs** |

Ask the user to confirm the mode before proceeding.

---

## Mode A — Generate Specs and Implement (New Project)

### A1. Read the PRD

Load `docs/product/PRD-*.md` (latest version). Extract:
- Project name, NCS version, target board(s)
- Selected features (FR list)
- Architecture pattern (SMF+Zbus or multi-threaded)
- Hardware matrix (buttons, LEDs per board)
- Non-functional requirements (memory, latency)

If no PRD exists, stop and say:
> "No PRD found in `docs/product/`. Please run **chsh-pm-prd** first to create one."

### A2. Plan the spec set

Based on PRD features, decide which spec files are needed:

| Always required | Per feature |
|---|---|
| `docs/engineering/specs/architecture.md` | `wifi-module.md` |
| `docs/engineering/config.yaml` | `webserver-module.md` |
| | `button-module.md` |
| | `mode-selector.md` |
| | `mqtt-client.md` |
| | *(one per significant module)* |

Present the plan to the user and confirm.

### A3. Generate `docs/engineering/config.yaml`

```yaml
schema: spec-driven
context: |
  Project: <name>
  Tech stack: Zephyr RTOS, NCS <version>
  Architecture: SMF+Zbus  # or Multi-threaded
  Target boards: <boards>

  Configuration strategy:
  - Base configs in prj.conf
  - Module configs co-located with module code (module.conf.template)
  - Merge module configs into prj.conf when module is enabled
  - Use overlays only for credentials and environment differences
```

### A4. Generate `docs/engineering/specs/architecture.md`

Include:
- Module map (table of all modules, their Zbus channels, SYS_INIT priority)
- Zbus channel definitions (struct types for each channel)
- SYS_INIT priority ordering (list all modules in startup order)
- System architecture diagram (ASCII or Mermaid)
- Boot sequence diagram
- Memory budget (Flash + RAM per module, total vs available)
- Board capability matrix (buttons, LEDs, Wi-Fi per board)

### A5. Generate per-module specs

For each module in the PRD, write `docs/engineering/specs/<module>.md` with:

```markdown
# <Module> Specification

## Overview
What this module does and why.

## Location
src/modules/<name>/  — files: .c, .h, Kconfig.<name>, CMakeLists.txt

## Zbus Integration
Publishes: <CHAN_NAME> (struct definition)
Subscribes: <CHAN_NAME>

## State Machine (if SMF)
Mermaid stateDiagram-v2 with all states and transitions

## Key Functions / API
Public functions with signatures and brief descriptions

## Kconfig Options
config APP_<MODULE>_MODULE
  bool / int / string ...

## Memory Footprint
| Component | Flash | RAM |

## Log Output Examples
[timestamp] <inf> module: example log line

## Testing (acceptance criteria from PRD)
TC-XXX-001: test case title
  Steps + expected outcome

## Related Specs
Links to other spec files this module depends on
```

### A6. Create project scaffold

```bash
# Core structure
mkdir -p src/modules boards docs/product docs/engineering/specs docs/engineering/proposals

# Copy base templates
cp ~/.claude/skills/chsh-dev-project/templates/LICENSE .
cp ~/.claude/skills/chsh-dev-project/templates/.gitignore .
cp ~/.claude/skills/chsh-dev-project/templates/README_TEMPLATE.md README.md

# Copy Wi-Fi config (pick mode from PRD)
cp ~/.claude/skills/chsh-dev-project/wifi/configs/wifi-sta.conf .      # STA
# cp ~/.claude/skills/chsh-dev-project/wifi/configs/wifi-softap.conf .  # SoftAP
# cp ~/.claude/skills/chsh-dev-project/wifi/configs/wifi-p2p.conf .     # P2P
```

### A7. Implement each module

For each spec in `docs/engineering/specs/`:

1. Create `src/modules/<name>/` with:
   - `<name>.c` — implementation following the state machine/API in the spec
   - `<name>.h` — public API and types
   - `Kconfig.<name>` — module Kconfig
   - `CMakeLists.txt` — `zephyr_library_sources` guarded by `CONFIG_APP_<MODULE>_MODULE`

2. Rsource the module Kconfig from the top-level `Kconfig`

3. Add module config to `prj.conf`:
   ```
   # <Module name>
   CONFIG_APP_<MODULE>_MODULE=y
   <paste relevant lines from overlay>
   ```

4. Wire the module into `src/main.c` (Zbus subscriptions or thread starts)

### A8. Build and verify

```bash
west build -p -b <board>
# For P2P: west build -p -b nrf54lm20dk/nrf54lm20a/cpuapp -S wifi-p2p --shield nrf7002eb2
```

Fix any build errors, then verify against spec acceptance criteria.

---

## Mode B — Update Specs (PRD Changed)

### B1. Identify what changed in the PRD

```bash
# Compare latest two PRD versions
PREV=$(ls docs/product/PRD-*.md | sort | tail -2 | head -1)
CURR=$(ls docs/product/PRD-*.md | sort | tail -1)
diff "$PREV" "$CURR"
```

### B2. Map PRD changes to spec impact

For each PRD change, identify which spec file(s) are affected:

| PRD change type | Spec impact |
|---|---|
| New feature added | Create new `docs/engineering/specs/<feature>.md` |
| Feature removed | Mark spec as deprecated or delete |
| Acceptance criteria changed | Update testing section in affected spec |
| Architecture pattern changed | Rewrite `architecture.md` + all module specs |
| New board added | Update hardware sections in all specs |
| NFR changed | Update memory budget in `architecture.md` |

### B3. Update affected specs

Update only the changed sections, preserving existing content that is still valid.
Add a version note at the top of each changed spec:
```markdown
> **Updated YYYY-MM-DD**: [brief description of what changed]
```

### B4. Prompt about implementation

After updating specs, ask:
> "Specs updated. Would you like to proceed to **Mode C** to implement the changes in code?"

---

## Mode C — Implement from Specs

### C1. Audit what is implemented vs what specs require

For each spec file in `docs/engineering/specs/`:
- Check if the corresponding module exists in `src/modules/`
- Check if Kconfig option is enabled in `prj.conf`
- Check if the module is wired into `CMakeLists.txt` and `Kconfig`

Present a gap table:

| Spec | Module exists? | Kconfig enabled? | Wired? |
|---|---|---|---|
| wifi-module.md | ✅ | ✅ | ✅ |
| mqtt-client.md | ❌ | ❌ | ❌ |

### C2. Implement missing modules

Follow the same steps as A7 for each gap.

### C3. Update existing modules

For modules that exist but whose spec has changed:
1. Read the spec diff (what changed)
2. Update the implementation to match
3. Ensure log output matches the spec's example log lines
4. Verify Kconfig options match the spec

### C4. Build and verify

```bash
west build -p -b <board>
```

Check that all spec acceptance criteria (TC-XXX-YYY) pass.

---

## Coding Standards

Always follow these rules when generating NCS/Zephyr code:

### Architecture
- SMF+Zbus: each module is a `SYS_INIT`-registered state machine; communicates only via Zbus channels
- Multi-threaded: each module has its own k_thread; communicates via message queues or semaphores
- Never use global variables to share state between modules — use Zbus channels

### File structure (one module)
```
src/modules/<name>/
├── <name>.c           # state machine + Zbus subscriber/publisher
├── <name>.h           # public API, structs, channel declarations
├── Kconfig.<name>     # config APP_<NAME>_MODULE + sub-options
└── CMakeLists.txt     # zephyr_library_sources if CONFIG_APP_<NAME>_MODULE
```

### Kconfig pattern
```kconfig
config APP_<NAME>_MODULE
    bool "Enable <Name> module"
    default y
    select <DEPENDENCY>

config APP_<NAME>_LOG_LEVEL
    int "<Name> module log level"
    default 3
    depends on APP_<NAME>_MODULE
```

### prj.conf strategy
- Base config (logging, shell, DK library) in `prj.conf`
- One section per module, merged from `module.conf.template`
- Credentials in git-ignored `overlay-credentials.conf`
- No feature overlays needed for modules (config is in prj.conf directly)

### Memory requirements (Wi-Fi projects)
- `CONFIG_HEAP_MEM_POOL_SIZE` ≥ 80000 (critical)
- `CONFIG_MAIN_STACK_SIZE` ≥ 4096
- Always verify with heap_monitor module

### Security
- Never hardcode credentials in source or `prj.conf`
- Credentials go in `overlay-credentials.conf` (git-ignored)
- Track `overlay-credentials.conf.template` in git instead
- Copyright year: use **current year** (2026) in all new files

---

## Reference: Reusable Modules

Copy these from the `ncs-project-logo` reference project:

| Module | Enable flag | Purpose |
|---|---|---|
| `src/modules/heap_monitor/` | `CONFIG_HEAPS_MONITOR=y` | System + mbedTLS heap tracking |
| `src/modules/app_memfault/` | `CONFIG_APP_MEMFAULT_MODULE=y` | Memfault OTA + metrics |
| `src/modules/network/` | `CONFIG_WIFI_MODULE=y` | Wi-Fi STA connection manager |
| `src/modules/wifi_prov_over_ble/` | `CONFIG_WIFI_STA_PROV_OVER_BLE_ENABLED=y` | BLE credential provisioning |
| `src/modules/app_https_client/` | `CONFIG_APP_HTTPS_CLIENT_MODULE=y` | Periodic HTTPS GET |
| `src/modules/app_mqtt_client/` | `CONFIG_APP_MQTT_CLIENT_MODULE=y` | Persistent MQTT over TLS |
| `src/modules/button/` | `CONFIG_BUTTON_MODULE=y` | SMF button with short/long press |

## Reference: Wi-Fi Config Files

```bash
~/.claude/skills/chsh-dev-project/wifi/configs/
├── wifi-sta.conf       # Station mode
├── wifi-softap.conf    # SoftAP mode
├── wifi-p2p.conf       # P2P / Wi-Fi Direct
└── wifi-raw.conf       # Monitor / raw packets
```

## Reference: Architecture Templates

```bash
~/.claude/skills/chsh-dev-project/architecture/smf-zbus/
├── templates/          # module_template_smf.c/h, Kconfig.module_template, messages.h
└── modules/            # button_example/, sensor_example/ (ready to copy)

~/.claude/skills/chsh-dev-project/architecture/simple-multithreaded/
└── templates/          # module_template_simple.c/h, Kconfig.module_template_simple
```

## Reference: Guides

- [ARCHITECTURE_PATTERNS.md](architecture/guides/ARCHITECTURE_PATTERNS.md) — SMF+Zbus vs multi-threaded deep dive
- [WIFI_GUIDE.md](wifi/guides/WIFI_GUIDE.md) — Wi-Fi modes, reconnection, event handling
- [RECONNECTION_PATTERNS.md](wifi/guides/RECONNECTION_PATTERNS.md) — STA retry and back-off
- [CONFIG_GUIDE.md](guides/CONFIG_GUIDE.md) — prj.conf strategy
- [PROJECT_STRUCTURE.md](guides/PROJECT_STRUCTURE.md) — recommended file layout

## Reference: Sub-Skills

Load these for deeper domain knowledge:

| Sub-skill | When to load |
|---|---|
| `chsh-dev-project/debug` | Debugging crashes, RTT logging, GDB |
| `chsh-dev-project/env-setup` | Toolchain setup, west init/update |
| `chsh-dev-project/architecture` | Architecture pattern selection and templates |
| `chsh-dev-project/protocols` | MQTT, CoAP, HTTP, TCP/UDP details |
| `chsh-dev-project/protocols/webserver` | Static HTTP server, REST API patterns |
| `chsh-dev-project/wifi` | Wi-Fi STA/SoftAP/P2P implementation details |

## Critical Requirements (NCS Wi-Fi projects)

- `CONFIG_WIFI=y` + `CONFIG_WIFI_NRF70=y`
- `CONFIG_WIFI_READY_LIB=y` (safe initialization — never skip)
- `CONFIG_HEAP_MEM_POOL_SIZE` ≥ 80000
- Network stack (IPv4, TCP, DHCP) configured
- All net events handled (CONNECTED, DISCONNECTED, IPV4_ADDR_ADD)
- NO hardcoded credentials anywhere
