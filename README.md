# Charlie Skills Repository

Personal collection of specialized skills for development workflows with Claude.

## 📦 Skills Overview

### Developer Skills

#### `chsh-dev-commit` — Git Commit Workflow
Plan, group, and execute git commits following Conventional Commits style.
- Entry: [SKILL.md](chsh-dev-commit/SKILL.md)

#### `chsh-dev-mem-opt` — NCS Memory Optimization
Strategies for footprint reduction and heap profiling in NCS/Zephyr projects.
- Entry: [SKILL.md](chsh-dev-mem-opt/SKILL.md)

#### `chsh-dev-project` — NCS Project Workflow
Complete lifecycle management: generate → develop → review → improve.
Includes sub-skills for debug, env-setup, architecture patterns, protocols, and Wi-Fi.
- Entry: [SKILL.md](chsh-dev-project/SKILL.md)
- Quick start:
	- `cp ~/.claude/skills/chsh-dev-project/templates/.gitignore ./`
	- `cp ~/.claude/skills/chsh-dev-project/templates/LICENSE ./`
	- `cp ~/.claude/skills/chsh-dev-project/templates/README_TEMPLATE.md README.md`
	- `cp ~/.claude/skills/chsh-dev-project/wifi/configs/wifi-sta.conf overlay-wifi.conf`
- Sub-skills:
	- [debug/SKILL.md](chsh-dev-project/debug/SKILL.md) — debugging, RTT, GDB
	- [env-setup/SKILL.md](chsh-dev-project/env-setup/SKILL.md) — toolchain & west setup
	- [architecture/SKILL.md](chsh-dev-project/architecture/SKILL.md) — SMF+zbus vs multi-threaded patterns
	- [protocols/SKILL.md](chsh-dev-project/protocols/SKILL.md) — MQTT, CoAP, HTTP, TCP/UDP
	- [protocols/webserver/SKILL.md](chsh-dev-project/protocols/webserver/SKILL.md) — static HTTP server
	- [wifi/SKILL.md](chsh-dev-project/wifi/SKILL.md) — STA, SoftAP, P2P modes

### Product Manager Skills

#### `chsh-pm-prd` — PRD & Feature Planning
PRD template, feature selection matrices, and configuration overlays for NCS projects.
- Entry: [SKILL.md](chsh-pm-prd/SKILL.md)
- Key assets:
	- [prd/PRD_TEMPLATE.md](chsh-pm-prd/prd/PRD_TEMPLATE.md)
	- [FEATURE_SELECTION.md](chsh-pm-prd/FEATURE_SELECTION.md)
	- [overlays/](chsh-pm-prd/overlays/) — ready-to-use Kconfig overlays

#### `chsh-pm-review` — NCS Project Review & QA
Checklists, report templates, and automation scripts for validating builds against the PRD.
- Entry: [SKILL.md](chsh-pm-review/SKILL.md)
- Key assets:
	- [CHECKLIST.md](chsh-pm-review/CHECKLIST.md)
	- [QA_TEMPLATE.md](chsh-pm-review/QA_TEMPLATE.md)
	- [check_project.sh](chsh-pm-review/check_project.sh)

### Writing Skills

#### `chsh-txt-review` — Text / Message Reviewer
Polish replies to customers or colleagues: friendly, professional, clear.
- Entry: [SKILL.md](chsh-txt-review/SKILL.md)

---

## 🚀 Usage

Skills are loaded during development conversations. Each skill provides templates,
guides, automation scripts, and best practices.

### Invocation Pattern

Mention the skill name in your request, for example:
- *"Use `chsh-dev-project` to generate a new Wi-Fi STA project."*
- *"Run `chsh-pm-review` on the nordic-wifi-webdash project."*
- *"Commit with `chsh-dev-commit`."*

## 📁 Structure

```
skills/
├── chsh-dev-commit/       Git commit workflow
├── chsh-dev-mem-opt/      Memory optimization
├── chsh-dev-project/      NCS project lifecycle
│   ├── debug/
│   ├── env-setup/
│   ├── architecture/
│   ├── protocols/
│   │   └── webserver/
│   ├── wifi/
│   ├── templates/
│   ├── guides/
│   └── examples/
├── chsh-pm-prd/           PRD & feature planning
│   ├── prd/
│   └── overlays/
├── chsh-pm-review/        Project review & QA
├── chsh-txt-review/       Text / message review
├── README.md
└── .gitignore
```

## 🔄 Workflow Integration

```
PRD (chsh-pm-prd) → Generate (chsh-dev-project) → Develop → Review (chsh-pm-review) → QA Report → Fix → Improve
```

## 📊 Token Efficiency

- Core `SKILL.md` files: ~2,000 tokens (auto-loaded)
- Detailed guides: 5,000+ tokens (loaded on-demand)
- Templates and configs: accessed as needed

## 📝 License

Skills and templates include appropriate license headers for target projects.

Individual skills may reference or include:
- Nordic 5-Clause License (LicenseRef-Nordic-5-Clause)
- Apache License 2.0
- MIT License


## 📅 Last Updated

April 9, 2026
