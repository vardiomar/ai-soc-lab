# Watchstander — AI-Augmented Home SOC Lab

> After building an enterprise Active Directory lab, I added an AI analyst to the
> SOC and tested whether it can be trusted. This repo documents the build, the
> AI tooling, and an attack against the AI itself.

**Status:** Phase 0 of 10 — Planning
**Author:** Omar Vardi — U.S. Navy veteran, cybersecurity student

---

## What this project does

A small isolated lab (Active Directory, a Wazuh SIEM, and packet capture) where:

1. **Build** — 5 VMs on an isolated network.
2. **Use** — a Python AI assistant triages Wazuh alerts; a second script summarizes packet captures.
3. **Measure** — AI verdicts are scored against my own analysis.
4. **Break** — prompt-injection payloads are planted in logs to fool the AI.
5. **Defend** — guardrails are added and the attacks re-run.

## Lab topology

![Lab topology](docs/images/topology.png)

| VM | Role | IP |
|----|------|----|
| DC01 | Windows Server 2022 — AD, DNS | 10.10.10.10 |
| WS01 | Windows 11 — victim workstation | 10.10.10.20 |
| SIEM01 | Ubuntu — Wazuh SIEM | 10.10.10.30 |
| AI01 | Ubuntu — AI scripts + packet capture | 10.10.10.40 |
| KALI01 | Kali — attacker | 10.10.10.50 |

## Documentation

| Phase | Doc |
|-------|-----|
| 0 — Planning | [docs/00-planning.md](docs/00-planning.md) |
| 1 — Network & Domain Controller | _coming soon_ |
| 2 — Endpoint & SIEM | _coming soon_ |
| ... | ... |
| AI collaboration log | [docs/ai-collaboration-log.md](docs/ai-collaboration-log.md) |

## Tools used

Windows Server 2022 · Wazuh · Sysmon · tcpdump · Wireshark · tshark ·
Kali Linux · Python · Gemini API · Atomic Red Team

## A note on AI use

This project is partly *about* using AI in security. I used an AI assistant to
help plan and build it, and I kept a log of where it helped, where it was wrong,
and what I decided myself. See the [AI collaboration log](docs/ai-collaboration-log.md).

---
*Built on Linux Mint with VirtualBox. All testing done in an isolated lab.*
