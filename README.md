# Olympus — Self-Hosted AI Agent on Azure

A personal project deploying **Hermes** (by Nous Research) onto Microsoft Azure infrastructure — a self-improving, always-on AI agent accessible from anywhere via Telegram.

Built as a hands-on Azure learning project aligned with **AZ-104: Microsoft Azure Administrator** certification.

---

## What Is Olympus?

Olympus is a cloud-deployed AI agent that runs 24/7 on an Azure Virtual Machine. Unlike AI assistants that reset after every session, Hermes uses an **Agentic Reinforcement Learning (RL) pipeline** — it learns, improves, and builds persistent memory over time.

Key capabilities:
- 75+ built-in skills across research, software development, productivity, and automation
- Persistent memory across sessions — learns your preferences and context
- Always online — runs independently of my local machine
- Accessible via Telegram from anywhere in the world
- Powered by Trinity Large (free, open-weight model via OpenRouter)

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Microsoft Azure                    │
│                                                     │
│  ┌─────────────────────────────────────────────┐   │
│  │           Resource Group: hermes-rg          │   │
│  │                                             │   │
│  │  ┌─────────────────────────────────────┐   │   │
│  │  │     Virtual Network: hermes-vnet     │   │   │
│  │  │         (10.0.0.0/16)               │   │   │
│  │  │                                     │   │   │
│  │  │  ┌───────────────────────────────┐  │   │   │
│  │  │  │   Subnet: hermes-subnet       │  │   │   │
│  │  │  │       (10.0.1.0/24)          │  │   │   │
│  │  │  │                               │  │   │   │
│  │  │  │  ┌─────────────────────────┐  │  │   │   │
│  │  │  │  │  VM: hermes-vm          │  │  │   │   │
│  │  │  │  │  Ubuntu 24.04 LTS       │  │  │   │   │
│  │  │  │  │  Standard D2s v3        │  │  │   │   │
│  │  │  │  │  2 vCPU / 8GB RAM       │  │  │   │   │
│  │  │  │  │                         │  │  │   │   │
│  │  │  │  │  ┌───────────────────┐  │  │  │   │   │
│  │  │  │  │  │  Docker Container  │  │  │  │   │   │
│  │  │  │  │  │  Hermes Agent      │  │  │  │   │   │
│  │  │  │  │  │  (nousresearch/    │  │  │  │   │   │
│  │  │  │  │  │   hermes-agent)    │  │  │  │   │   │
│  │  │  │  │  └───────────────────┘  │  │  │   │   │
│  │  │  │  └─────────────────────────┘  │  │   │   │
│  │  │  └───────────────────────────────┘  │   │   │
│  │  └─────────────────────────────────────┘   │   │
│  │                                             │   │
│  │  NSG: hermes-nsg                            │   │
│  │  ├── Allow SSH (port 22) — restricted IP    │   │
│  │  ├── Allow HTTP (port 80)                   │   │
│  │  └── Allow HTTPS (port 443)                 │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
         │
         │ OpenRouter API
         ▼
┌─────────────────┐
│  Trinity Large   │
│  (free, open     │
│   weights)       │
│  via OpenRouter  │
└─────────────────┘
         │
         │ Telegram Gateway
         ▼
┌─────────────────┐
│  Telegram Bot   │
│  (anywhere,     │
│   any device)   │
└─────────────────┘
```

---

## Azure Infrastructure

| Resource | Name | Details |
|---|---|---|
| Resource Group | hermes-rg | Central US |
| Virtual Network | hermes-vnet | 10.0.0.0/16 |
| Subnet | hermes-subnet | 10.0.1.0/24 |
| Network Security Group | hermes-nsg | Custom inbound rules |
| Virtual Machine | hermes-vm | Ubuntu 24.04 LTS, D2s v3 |
| Public IP | hermes-vm-ip | Static |
| OS Disk | Premium SSD | 30GB |

### NSG Rules

| Priority | Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| 200 | Allow-SSH | 22 | TCP | Admin IP only | Allow |
| 210 | Allow-HTTP | 80 | TCP | Any | Allow |
| 220 | Allow-HTTPS | 443 | TCP | Any | Allow |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Cloud Provider | Microsoft Azure |
| Compute | Azure Virtual Machine (Ubuntu 24.04 LTS) |
| Containerization | Docker + Docker Compose |
| AI Agent | Hermes by Nous Research |
| LLM | Trinity Large Preview (free) via OpenRouter |
| Messaging | Telegram Bot API |
| Authentication | SSH key pair (RSA) |
| Monitoring | Azure Boot Diagnostics |

---

## AZ-104 Concepts Covered

This project was built as a practical application of AZ-104 exam objectives:

**Manage Azure identities and governance**
- Resource Groups — organizing related resources
- Azure subscription management

**Implement and manage storage**
- OS disk management (Premium SSD LRS)
- Azure Boot Diagnostics with managed storage

**Deploy and manage Azure compute resources**
- Virtual Machine deployment (IaaS)
- VM sizing and configuration
- Ubuntu Linux on Azure
- SSH key pair authentication

**Implement and manage virtual networking**
- Virtual Network creation and configuration
- Subnet design and CIDR notation (10.0.0.0/16, 10.0.1.0/24)
- Network Security Groups
- Inbound security rules (priority, protocol, port, source)
- Static Public IP addresses
- NIC configuration and attachment

**Monitor and maintain Azure resources**
- Azure Boot Diagnostics
- Azure Monitor configuration
- VM restart and maintenance

---

## Why This Project

I'm pursuing the **AZ-104 Azure Administrator** certification while transitioning into remote IT consulting. Rather than study purely through labs and textbooks, I built real infrastructure I actually use daily.

The result: a cloud-deployed AI agent that runs 24/7, costs near-zero to operate (free-tier VM + free LLM), and gave me hands-on experience with every major networking and compute concept on the AZ-104 exam.

This is part of a larger personal infrastructure project — **PAIOS (Personal AI Operating System)** — combining local and cloud AI agents to support remote work, research, and automation.

---

## Related Projects

- **[ClaudeClaw / PAIOS](#)** — Local multi-agent AI command center with War Room voice interface

---

## Status

- [x] Azure infrastructure deployed
- [x] Docker + Hermes running on VM
- [x] OpenRouter + Trinity model configured
- [x] SSH access and security hardened
- [ ] Telegram gateway connected (in progress)
- [ ] Persistent gateway service (systemd)
- [ ] Azure Monitor alerts configured
- [ ] Resize to B2s when available in region

---

## Author

**Ade Cobbs** — IT Professional | Azure enthusiast | Building in public

Washington DC | [GitHub](https://github.com/Pade89) | [LinkedIn](#) | [Upwork](#)
