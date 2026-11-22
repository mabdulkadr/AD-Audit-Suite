<div align="center">

# 🏢 Active Directory Audit Suite

**Standalone AD health and security reports, one script and one HTML file per report**

A set of PowerShell scripts that read your live environment and write a portable, interactive report for each area of Active Directory health and security.

[![Mode](https://img.shields.io/badge/Mode-CLI-334155?style=for-the-badge)](#-usage)
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%2B-5391FE?style=for-the-badge&logo=powershell&logoColor=white)](https://learn.microsoft.com/en-us/powershell/)
[![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-0F172A?style=for-the-badge)](#%EF%B8%8F-requirements)
[![Docs](https://img.shields.io/badge/Docs-HTML%20Reports-10B981?style=for-the-badge)](#-modules)
[![License](https://img.shields.io/badge/License-MIT-F59E0B?style=for-the-badge)](#-license)
[![Version](https://img.shields.io/badge/Version-1.0.0-334155?style=for-the-badge)](#-overview)

[Overview](#-overview) • [Modules](#-modules) • [Structure](#-project-structure) • [Requirements](#-requirements) • [License](#-license)

</div>

---

# 📖 Overview

This suite is a set of standalone PowerShell scripts. Each one reads the live Active Directory environment and writes a single interactive HTML report about one area of AD health or security.

---

# Active Directory Audit Suite

> A growing collection of standalone PowerShell scripts that each produce a single, self-contained, interactive HTML report for a different area of Active Directory health and security.

Every module is one script. Run it on a domain-joined machine with the AD PowerShell module, and it reads the live environment and writes one HTML file you can open in any browser. It needs no agents or internet access and sends no telemetry. The report is a single file, so you can attach it to a ticket or email.

---

# ✨ Core Features

* One PowerShell script per AD assessment area.
* Each script writes one self-contained HTML report that works offline.
* Data comes from the local domain-joined machine. No agents or telemetry are involved.

---

# 📂 Project Structure

```text
AD-Audit-Suite/
│
├── scripts\       # One PowerShell script per assessment module
├── docs\          # Demos and roadmap
├── examples\      # Sample HTML reports built from fictional data
├── CHANGELOG.md
└── README.md
```

---

## Modules

| Module | Script | What it covers |
|---|---|---|
| **Overview** | \`AD-Overview.ps1\` | Forest & domain facts, object inventory, account posture, OS distribution, group breakdown |
| **Topology** | \`AD-Topology.ps1\` | Forest → domain → site → DC hierarchy with health, FSMO roles, services, replication links, dcdiag |
| **DC Inventory** | \`AD-DCInventory.ps1\` | Per-DC inventory plus hardware & performance specs (CPU, RAM, disks, NICs) |
| **Account Security** | \`AD-AccountSecurity.ps1\` | Password/lockout policy, fine-grained password policies, managed service accounts, LAPS coverage, risky user & computer accounts, krbtgt age |
| **Trust Relationships** | \`AD-TrustRelationships.ps1\` | Every AD trust with direction/type/transitivity, SID filtering & selective auth posture, connectivity probe, derived risk findings, interactive trust map |

Each module uses its own layout, so the reports are easy to tell apart.

---

## Overview Module

![AD Overview demo](docs/overview-demo.gif)

The forest-wide summary is the natural landing page for the whole audit. It shows a hero banner with forest facts (functional level, schema version, FSMO masters, tombstone lifetime), object-count KPI cards, donut charts for directory objects and account posture, bar charts for computer OS distribution and group breakdown, and a per-domain comparison table.

Stat cards that represent a list of objects (enabled / disabled / stale / never-logged-on / locked / password-never-expires accounts, computers, and security / distribution / scoped groups) include a download icon that exports that exact list to CSV (Name, SamAccountName, Enabled, LastLogon, and DN) entirely client-side, straight from the report.

\`\`\`powershell
.\scripts\AD-Overview.ps1
\`\`\`

---

## Topology

![AD Topology demo](docs/topology-demo.gif)

An interactive, pannable/zoomable map of the forest. Nested cards for forest → domain → site → DC, colour-coded by health. Click anything for a slide-in detail panel: forest info, domain FSMO holders, site subnets, or a rich DC panel with services, resources, replication status, and dcdiag results. Toggleable inter-site replication link overlay with cost and frequency labels.

The toolbar also opens two analysis panels:

- **Replication Health Matrix:** a directional source → destination grid of every DC partnership across every domain in the forest, built from the replication metadata AD already tracks. It checks all naming contexts (domain, Configuration, Schema, DomainDnsZones, ForestDnsZones), not only the default partition, and each cell shows the worst partition. A pair that is healthy on the domain NC but failing on Configuration still shows as failing. Click any cell for the full per-partition breakdown. It reads over ADWS (TCP 9389) and falls back to `repadmin`/RPC when ADWS is unreachable. "Delayed" thresholds are site-aware (`-ReplDelayIntraSiteHours`, default 1; `-ReplDelayInterSiteHours`, default 6), so healthy inter-site partners on the standard 180-minute schedule aren't flagged. Passive; no network probing.
- **Stale / Lingering Objects:** a read-only scan of the Configuration partition and Sites & Services for orphaned DC metadata, empty sites, sites without subnets, unlinked subnets, and lingering replication connections, each with the recommended remediation.

\`\`\`powershell
.\scripts\AD-Topology.ps1
\`\`\`

---

## DC Inventory

![AD DC Inventory demo](docs/dcinventory-demo.gif)

Per-DC inventory plus deep hardware and performance detail. KPI cards for online/offline/GC/RODC/legacy-OS counts, a full inventory table (name, domain, site, OS, build, IPv4, time zone, uptime, GC, RODC, status), and an expandable card per DC with System (manufacturer, model, BIOS, RAM), Performance (CPU/memory usage, AD database size), Processors, Logical Volumes with usage bars, and Network Adapters.

\`\`\`powershell
.\scripts\AD-DCInventory.ps1
\`\`\`

---

## Account Security

![AD Account Security demo](docs/AccountSecurity-demo.png)

Credential hygiene across the domain. It scores the default password & lockout policy against a recommended baseline, lists fine-grained password policies (PSOs) with precedence and who they apply to, inventories managed service accounts (gMSA/sMSA) and LAPS coverage (Windows and Legacy), and flags risky accounts (password-never-expires, password-not-required, unconstrained delegation, SID history, inactive-but-enabled users, stale service-account passwords, stale/legacy/EOL-OS computers), plus krbtgt password age. Long lists stay off the main page behind per-category **Open** (full-page view) and **CSV download** actions so a large tenant stays readable. Optional sources (LAPS schema, KDS root key, etc.) degrade gracefully to "Not available" instead of failing.

\`\`\`powershell
.\scripts\AD-AccountSecurity.ps1
\`\`\`

---

## Trust Relationships

![AD Trust Relationships demo](docs/AD-TrustRelationships-demo.gif)

Enumerates every AD trust and assesses its security posture: partner domain, direction (inbound/outbound/bidirectional), trust type (external/forest/realm/parent-child/tree-root), and transitivity, plus SID filtering (quarantine), selective authentication, TGT delegation, and forest-transitive flags. Each trust gets a live connectivity probe (degrading gracefully to "Unavailable / Not tested" in locked-down environments) and derived risk findings (SID filtering disabled on an external/forest trust, selective authentication off, stale trusts, and more). Includes KPI tiles, a filterable trust explorer with detail panel, and an interactive trust map.

\`\`\`powershell
.\scripts\AD-TrustRelationships.ps1
\`\`\`

---

## Common parameters

Every module accepts:

| Parameter | Default | Description |
|---|---|---|
| \`-OutputPath\` | current directory | Folder to write the HTML report to |
| \`-OpenReport\` | \`\$true\` | Open the report in the default browser when finished |

Some modules add their own (\`-StaleDays\` on Overview, \`-SkipHardware\` on DC Inventory, \`-InactiveDays\`/\`-StaleComputerDays\`/\`-ServiceAccountStalePasswordDays\` on Account Security, \`-TestConnectivity\` on Trust Relationships). Run \`Get-Help .\scripts\<script>.ps1 -Full\` for details.

> **Execution policy:** if a script is blocked, run it for the current process only:
> \`\`\`powershell
> powershell -ExecutionPolicy Bypass -File .\scripts\AD-Overview.ps1
> \`\`\`

---

## Requirements

| Requirement | Notes |
|---|---|
| Windows with **RSAT: Active Directory PowerShell** | \`Import-Module ActiveDirectory\` must work |
| **Domain-joined** machine (or one that can reach a DC) | Read-only AD queries |
| PowerShell **5.1+** | Windows PowerShell or PowerShell 7 |
| Account with **read access** to AD | No write operations are performed |

For the richest per-DC detail (Topology and DC Inventory), the running account should be able to reach the DCs via **WinRM** or **WMI/DCOM**. When a DC can't be reached, the report degrades gracefully and shows the fields it could not collect as *unavailable* rather than failing.

---

## Sample reports

The [\`examples/\`](examples/) folder contains a sample HTML report for each module, built from **fictional data** (\`corp.local\`). Open any of them in a browser to explore the interactive features without running anything.

---

## Read-only & safe to run

- Every module performs **read-only** AD and DC queries. Nothing is modified in Active Directory, Group Policy, or on any domain controller.
- The generated HTML is fully self-contained and offline, with no external scripts, trackers, or network calls when opened.
- Each script writes **only** its single HTML report to your chosen \`-OutputPath\`. Nothing else is created or changed.
- No credentials, passwords, or tokens are stored or transmitted.

The reports contain infrastructure and security-posture detail (domain/DC names, IPs, group memberships, password policy weaknesses, risky account lists, etc.), so treat them as confidential and store/share them accordingly.

---

## Part of a larger project

These five modules are part of an ongoing Active Directory Audit Suite. Additional modules covering other areas of AD health and security are in development and will be published here one by one. The long-term goal is a single orchestrator that runs all modules together and produces a comprehensive combined report.

Watch or ⭐ the repo to catch new modules as they land.

For more detail on the project direction, see [docs/ROADMAP.md](docs/ROADMAP.md).

---

## Author

**Mohamed ZEGHLACHE**, Hybrid Cloud Engineer.

If this is useful, a ⭐ on the repo is appreciated, and feedback / issues are welcome.

---

## 📜 License

This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).

---

## ⚠ Disclaimer

This skill and every script it generates are provided as-is with no warranty of any kind. Test generated tools in a staging environment before deploying to production. The authors assume no liability for any damage or data loss resulting from their use.

---
<div align="center">

⭐ **If this skill saves you time, star the repo — it helps others find it.**

[Report an Issue](../../issues) · [momar.tech](https://momar.tech)

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=for-the-badge&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/mabdulkadrx)

Built with [**PowerShell Enterprise Admin**](https://github.com/mabdulkadr/powershell-enterprise-admin)

</div>
