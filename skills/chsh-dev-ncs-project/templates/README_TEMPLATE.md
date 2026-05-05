# <Project Name>

<!--
README QUALITY TARGETS
  Reading time:  3–5 min for Evaluator Quick Start path | 8–12 min for full document
  Character count:  3,000–7,000 characters (excluding code blocks)
  Principle: cognitive funnel — broad → narrow. Evaluator should reach
  a working device by the end of Evaluator Quick Start without scrolling past it.
-->

[![Build](https://github.com/<org>/<repo>/actions/workflows/build.yml/badge.svg)](https://github.com/<org>/<repo>/actions/workflows/build.yml)
[![License](https://img.shields.io/badge/License-LicenseRef--Nordic--5--Clause-blue.svg)](LICENSE)
[![NCS](https://img.shields.io/badge/NCS-v3.x.x-skyblue)](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/index.html)

<One sentence describing what the device does and for whom.>

![Screenshot or demo of the device in action](docs/images/screenshot.png)

---

## Project Overview

### Introduction

<Two to three sentences: what problem this solves, who the target user is, and what makes it distinctive.>

### Supported hardware

| Board | Build target |
|-------|--------------|
| &lt;Board A&gt; | `<target/cpuapp>` |
| &lt;Board B&gt; + &lt;Shield&gt; | `<target/cpuapp>` + `-DSHIELD=<shield>` |

### Features

- **&lt;Feature 1&gt;** — brief description
- **&lt;Feature 2&gt;** — brief description
- **&lt;Feature 3&gt;** — brief description
- **&lt;Feature 4&gt;** — brief description

### Project Structure

```
<project-name>/
├── CMakeLists.txt
├── Kconfig
├── prj.conf
├── west.yml
├── docs/
│   ├── PRD.md                   ← product requirements and acceptance criteria
│   └── specs/
│       ├── overview.md          ← spec index and architecture summary
│       ├── architecture.md      ← module map, Zbus channels, boot sequence
│       └── <module>-module.md   ← per-module specs
├── src/
│   ├── main.c
│   └── modules/
│       ├── <module-a>/
│       ├── <module-b>/
│       └── messages.h
└── boards/
    └── <board>.overlay
```

### Target Users

- **Evaluator** — grab a pre-built `.hex` from the [Releases](<releases-url>) page, follow [Evaluator Quick Start](#evaluator-quick-start), and reach a working device in under 5 minutes.
- **Developer** — clone the workspace, build from source, and customise the firmware; see [Developer Info](#developer-info).

---

## Evaluator Quick Start

> Evaluator path — no build environment needed. ~5 minutes.

### Step 1 — Flash the firmware

Download the pre-built `.hex` for your board from the [Releases](<releases-url>) page.

Open **nRF Connect for Desktop → Programmer**, select your board, add the `.hex` file, and click **Erase & Write**.

### Step 2 — Connect

<Describe the shortest path to a working device: join Wi-Fi network, open browser URL, etc.>

| Action | Value |
|--------|-------|
| &lt;e.g. Wi-Fi SSID&gt; | `<SSID>` |
| &lt;e.g. Browser URL&gt; | `http://<hostname>.local` |

### Step 3 — Verify

&lt;What the user should see: LED state, browser dashboard URL, UART output. Include the screenshot here.&gt;

![&lt;Project name&gt; dashboard](docs/images/screenshot.png)

## Buttons & LEDs

<!--
Include this section when buttons/LEDs have user-visible behaviour or controls.
For projects where buttons are read-only inputs and LEDs are API-controlled,
replace with two tables (one per component).
-->

### Buttons

| Board | Buttons | Function |
|-------|---------|----------|
| &lt;Board A&gt; | &lt;SW1, SW2&gt; | &lt;description, e.g. state and count shown in dashboard&gt; |
| &lt;Board B&gt; + &lt;Shield&gt; | &lt;BUTTON0–BUTTON2&gt; | &lt;Same; BUTTON3 unavailable — shield pin conflict&gt; |

### LEDs

| Board | LEDs | Control |
|-------|------|---------|
| &lt;Board A&gt; | &lt;LED1, LED2&gt; | &lt;description, e.g. controlled via dashboard or `/api/led`&gt; |
| &lt;Board B&gt; | &lt;LED0–LED3&gt; | &lt;Same&gt; |

---

## Developer Info

### Workspace Setup

#### Method 1 (Preferred) — Add to an existing NCS installation

If you already have a matching NCS version installed, reuse it directly — no re-downloading required.

Under a terminal with the toolchain:

```sh
cd /opt/nordic/ncs/<ncs-version>   # your existing NCS workspace root

git clone https://github.com/<org>/<repo>.git

# Switch the workspace manifest to <repo> (one-time change)
west config manifest.path <repo>

# Sync — NCS repos already present, only new project repos are cloned
west update
```

#### Method 2 — Fresh installation as a Workspace Application

##### Option A: nRF Connect for VS Code

Follow the [custom repository guide](https://docs.nordicsemi.com/bundle/nrf-connect-vscode/page/guides/extension_custom_repo.html).

##### Option B: CLI

See the Nordic guide on [Workspace Application Setup](https://docs.nordicsemi.com/bundle/ncs-latest/page/nrf/dev_model_and_contributions/adding_code.html#workflow_4_workspace_application_repository_recommended).

```sh
west init -m https://github.com/<org>/<repo> --mr main <workspace-dir>
cd <workspace-dir>
west update
```

For product context and implementation details, start at [docs/specs/overview.md](docs/specs/overview.md) — it maps every PRD requirement to the spec file that implements it.

### Build

```bash
nrfutil sdk-manager toolchain launch --ncs-version=v3.x.x -- \
  west build -p -b <board-target> -d build <app-dir>
```

For &lt;Board B&gt; with shield:

```bash
nrfutil sdk-manager toolchain launch --ncs-version=v3.x.x -- \
  west build -p -b <board-target> -d build <app-dir> -- -DSHIELD=<shield>
```

### Flash

```bash
# <Board A>
west flash --erase

# <Board B>
west flash --recover
```

### Serial Monitor

Connect at **115200 baud**. The device prints its IP address and connection status at boot.

---

## Documentation

| Document | Description |
|----------|-------------|
| [docs/PRD.md](docs/PRD.md) | Product requirements, features, acceptance criteria |
| [docs/specs/overview.md](docs/specs/overview.md) | Spec index, PRD-to-spec mapping, architecture summary |
| [docs/specs/architecture.md](docs/specs/architecture.md) | Module map, Zbus channels, SYS_INIT boot order |

---

## Methodology

This project was developed using the [chsh-ncs-workflow](https://github.com/chshzh/charlie-skills) — a four-phase lifecycle for NCS/Zephyr IoT projects where each phase has a dedicated AI skill:

| Phase | Focus | Skill | Output |
|-------|-------|-------|--------|
| 1 — Product Definition | What the device should do, for whom, and why | `chsh-pm-prd` | `docs/PRD.md` |
| 2 — Technical Design | Translate PRD into engineering specs | `chsh-dev-spec` | `docs/specs/*.md` |
| 3 — Implementation | Implement code from approved specs | `chsh-dev-project` | `src/`, passing build |
| 4 — QA & Test | Validate the build against PRD criteria | `chsh-qa-test` | `TEST-*.md`, `QA-*.md` |

Each phase feeds the next: requirements drive specs, specs drive code, code drives tests. Issues loop back to the right phase — code bugs to Phase 3, spec gaps to Phase 2, new requirements to Phase 1.

Supporting skills: `chsh-dev-commit` (logical git history), `chsh-dev-mem-opt` (flash/RAM analysis).

---

## License

Copyright (c) &lt;year&gt; Nordic Semiconductor ASA

[SPDX-License-Identifier: LicenseRef-Nordic-5-Clause](LICENSE)

---

<!--
## Optional Sections

Add any of the following when appropriate for your project.
Delete this comment block before publishing.

### API Reference

Use when the device exposes an HTTP REST API or other machine-readable interface.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/resource` | GET | Returns ... |
| `/api/resource` | POST | Controls ... |

### Troubleshooting

Use when common failure modes are known and documented.

- &lt;Symptom&gt;: &lt;cause and resolution&gt;
- &lt;Symptom&gt;: &lt;cause and resolution&gt;

### References

Use for key upstream documentation links.

- [nRF Connect SDK Documentation](https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/index.html)
- [Zephyr RTOS Documentation](https://docs.zephyrproject.org/latest/)

### Contributing

This project follows Nordic Semiconductor coding standards and Zephyr contribution guidelines.
Contributions are welcome — open an issue or pull request.
-->
