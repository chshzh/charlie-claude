---
name: chsh-pm-prd
description: Interactive PRD authoring for NCS projects — guide the user through creating, updating, or extending a versioned PRD-YYYY-MM-DD.md under docs/product/. Use when the user wants to create a new PRD, add or change a feature, or sync the PRD after code changes.
---

# chsh-pm-prd — Interactive PRD Workflow

This skill guides the user interactively through PRD authoring and maintenance.
All output is a versioned `PRD-YYYY-MM-DD.md` saved to `docs/product/` in the project.

---

## Step 0 — Detect Mode

Before doing anything else, check the project:

```bash
ls docs/product/PRD-*.md 2>/dev/null | sort | tail -1   # latest versioned PRD
git log --oneline -5                                       # recent commits
```

Present the user with the correct starting options:

| Condition | Offer |
|---|---|
| No `docs/product/PRD-*.md` exists | **New** only |
| PRD exists, no code changes since its date | **Add Feature**, **Change Feature** |
| PRD exists, commits exist after its date | **Update** (code changed), **Add Feature**, **Change Feature** |

Ask the user: *"Which mode would you like?"*

---

## Mode A — New PRD

Walk through these questions one section at a time. Wait for answers before moving on.

### A1. Project Identity
- Project name (also used as folder/repo name)?
- One-sentence description of what it does?
- NCS version (e.g. v3.2.4)?
- Target board(s)? (nRF7002DK / nRF54LM20DK+nRF7002EBII / other)

### A2. Problem & Users
- What problem does this project solve?
- Who are the primary users? Secondary?
- What is the main pain point today without this product?

### A3. Feature Selection

Present each category and ask which to include. Show flash/RAM cost next to each.

**Wi-Fi Mode** (pick one or more):
| # | Feature | Config flag | Flash | RAM |
|---|---|---|---|---|
| 1 | Wi-Fi STA | `CONFIG_WIFI_NM_WPA_SUPPLICANT=y` | ~60 KB | ~40 KB |
| 2 | Wi-Fi SoftAP | `CONFIG_NRF70_AP_MODE=y` | ~65 KB | ~50 KB |
| 3 | Wi-Fi P2P | snippet `wifi-p2p` | ~70 KB | ~45 KB |

**Network Protocols** (pick any):
| # | Feature | Config flag | Flash | RAM |
|---|---|---|---|---|
| 4 | HTTP Server | `CONFIG_HTTP_SERVER=y` | ~25 KB | ~20 KB |
| 5 | HTTPS Server | + TLS | ~45 KB | ~30 KB |
| 6 | HTTP Client | `CONFIG_NET_HTTP_CLIENT=y` | ~15 KB | ~8 KB |
| 7 | MQTT | `CONFIG_MQTT_LIB=y` | ~20 KB | ~10 KB |
| 8 | CoAP | `CONFIG_COAP=y` | ~10 KB | ~5 KB |
| 9 | UDP | `CONFIG_NET_UDP=y` | ~5 KB | ~2 KB |
| 10 | TCP | `CONFIG_NET_TCP=y` | ~20 KB | ~10 KB |
| 11 | mDNS | `CONFIG_MDNS_RESPONDER=y` | ~10 KB | ~5 KB |

**Storage:**
| # | Feature | Config flag | Flash | RAM |
|---|---|---|---|---|
| 12 | NVS | `CONFIG_NVS=y` | ~8 KB | ~3 KB |
| 13 | Settings | `CONFIG_SETTINGS=y` | ~10 KB | ~4 KB |
| 14 | Wi-Fi Credentials | `CONFIG_WIFI_CREDENTIALS=y` | ~5 KB | ~2 KB |

**Advanced:**
| # | Feature | Config flag | Flash | RAM |
|---|---|---|---|---|
| 15 | Memfault | overlay-memfault.conf | ~120 KB | ~50 KB |
| 16 | BLE Provisioning | overlay-ble-prov.conf | ~60 KB | ~20 KB |
| 17 | Heap Monitor | `CONFIG_HEAPS_MONITOR=y` | ~2 KB | ~1 KB |

**Architecture Pattern:**
| # | Pattern | When to use |
|---|---|---|
| A | SMF + Zbus | Recommended for multi-module systems; Nordic's preferred pattern |
| B | Multi-threaded | Simpler; good for small single-purpose apps |

### A4. Functional Requirements

For each selected feature, ask:
- User story: "As a [user], I want to [action] so that [benefit]"
- Acceptance criteria (2–4 bullet points each)
- Priority: P0 (must have) / P1 (should have) / P2 (nice to have)

Link each FR to the matching engineering spec:
`[spec-name.md](../engineering/specs/spec-name.md)`

### A5. Non-Functional Requirements
- Dashboard/API response time target?
- Connection time targets (STA, SoftAP, P2P)?
- 24-hour stability requirement?
- Memory headroom target (Flash/RAM free)?

### A6. Hardware Matrix

For each target board, confirm:
- Available buttons (count + DK indices)
- Available LEDs (count)
- Wi-Fi chip (onboard or shield)
- Any pin conflicts with shields?

### A7. Success Metrics

Ask for 3–5 measurable metrics with targets, e.g.:
- Build success rate
- Connection time
- Dashboard load time
- Memory headroom

---

## Mode B — Add Feature

1. Read the latest `docs/product/PRD-*.md`.
2. Show the current feature list.
3. Ask: *"Which feature would you like to add?"* (offer the table from A3 minus already-selected features)
4. Ask user story + acceptance criteria + priority for the new feature.
5. Ask: *"Does this feature require any changes to NFRs or the hardware matrix?"*
6. Proceed to **Generate Output**.

---

## Mode C — Change Feature

1. Read the latest `docs/product/PRD-*.md`.
2. List the current functional requirements (FR-xxx) with their titles.
3. Ask: *"Which requirement would you like to change?"*
4. Show the current text and ask: *"What should change — the user story, acceptance criteria, priority, or all?"*
5. Collect the changes interactively.
6. Proceed to **Generate Output**.

---

## Mode D — Update (Code Changed, PRD Stale)

Use this when the codebase has moved ahead of the PRD.

### D1. Find the gap

```bash
# Get date of latest PRD
LATEST_PRD=$(ls docs/product/PRD-*.md | sort | tail -1)
PRD_DATE=$(echo "$LATEST_PRD" | grep -oE '[0-9]{4}-[0-9]{2}-[0-9]{2}')

# Show commits after that date
git log --oneline --since="$PRD_DATE" -- src/ prj.conf CMakeLists.txt Kconfig boards/
```

### D2. Analyze changes

For each changed file/module, summarize:
- What was added, removed, or changed
- Whether it affects any FR acceptance criteria
- Whether any new features are now present in code but missing from PRD

Present a summary table to the user:

| File / Area | Change summary | PRD impact |
|---|---|---|
| `src/modules/wifi/` | Added P2P support | FR-003 needs new acceptance criteria |
| `prj.conf` | Added `CONFIG_MDNS_RESPONDER=y` | New feature not in PRD |

### D3. Confirm updates

For each gap found, ask: *"Should this be reflected in the PRD?"*

Collect user's answers before writing.

---

## Generate Output

After any mode is complete:

1. Determine today's date: `date +%Y-%m-%d`
2. Write `docs/product/PRD-YYYY-MM-DD.md` using the full PRD structure below.
3. If a previous PRD exists, add a **Changelog** section at the top listing what changed from the prior version.
4. Print: *"PRD saved to `docs/product/PRD-YYYY-MM-DD.md`."*

### PRD File Structure

```markdown
# Product Requirements Document — [Project Name]

## Document Information
- Product Name: ...
- Version: YYYY-MM-DD
- Previous Version: YYYY-MM-DD (link)
- Status: Draft / Review / Approved
- NCS Version: ...

## Changelog
| Version | Change summary |
|---|---|
| YYYY-MM-DD | Initial / Added X / Changed Y |

## 1. Executive Summary
### 1.1 Product Overview
### 1.2 Problem Statement
### 1.3 Target Users
### 1.4 Success Metrics

## 2. Product Requirements
### 2.1 Feature Selection (table with selected features, config flags, Flash, RAM)
### 2.2 Functional Requirements (P0/P1/P2, with spec links)
### 2.3 Non-Functional Requirements
### 2.4 Hardware Requirements (per-board matrix)
### 2.5 User Experience Requirements (boot UX, mode selection UX, etc.)

## 3. Architecture Overview
(Pattern selected, high-level module map — NOT implementation detail)

## 4. Release Criteria
(List of P0 FRs that must all pass before release)

## 5. Open Questions
(Anything still unresolved)

## 6. Engineering Spec References
- [architecture.md](../engineering/specs/architecture.md)
- [module-name.md](../engineering/specs/module-name.md) (one per module)
```

---

## End-of-Workflow Prompt

After saving the PRD, always ask:

> "The PRD `docs/product/PRD-YYYY-MM-DD.md` is ready.
>
> Would you like to run **chsh-dev-project** now to:
> - Update engineering specs in `docs/engineering/specs/` to match this PRD, and/or
> - Implement or update the corresponding code?
>
> Reply **yes** to continue, or **no** to stop here."

If the user says yes, hand off to `chsh-dev-project` with context:
*"PRD is at `docs/product/PRD-YYYY-MM-DD.md`. Previous specs are in `docs/engineering/specs/`. Please update specs and implement according to the new PRD."*

---

## Reference: Feature Overlays

Ready-to-use Kconfig overlays in `~/.claude/skills/chsh-pm-prd/overlays/`:

| File | Feature |
|---|---|
| `overlay-wifi-shell.conf` | Wi-Fi shell commands |
| `overlay-mqtt.conf` | MQTT over TLS |
| `overlay-http-client.conf` | HTTP/HTTPS client |
| `overlay-https-server.conf` | HTTPS server |
| `overlay-coap.conf` | CoAP |
| `overlay-tcp.conf` | TCP |
| `overlay-udp.conf` | UDP |
| `overlay-memfault.conf` | Memfault monitoring + OTA |
| `overlay-ble-prov.conf` | BLE credential provisioning |
| `overlay-smf-zbus.conf` | SMF + Zbus architecture |
| `overlay-multithreaded.conf` | Multi-threaded architecture |

PRD template: `~/.claude/skills/chsh-pm-prd/prd/PRD_TEMPLATE.md`
