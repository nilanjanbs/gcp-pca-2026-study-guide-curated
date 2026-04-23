# 🏗️ GCP Professional Cloud Architect — 2026 Exam Study Guide

<div align="center">

![GCP PCA Badge](https://img.shields.io/badge/Google_Cloud-Professional_Cloud_Architect-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)
![Exam Year](https://img.shields.io/badge/Exam_Year-2026-orange?style=for-the-badge)
![Domains](https://img.shields.io/badge/Domains-6-green?style=for-the-badge)
![Questions](https://img.shields.io/badge/Practice_Questions-90%2B-red?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

**A complete, exam-focused, deep-dive study guide for the Google Professional Cloud Architect certification — updated for 2026.**

[📚 Start Studying](#-exam-domains) · [⚡ Quick Reference](./quick-reference/GCP_PCA_2026_MasterGuide.md) · [🤝 Contribute](#-contributing) · [🐛 Report Issue](https://github.com/YOUR_USERNAME/gcp-pca-2026-study-guide/issues)

</div>

---

## 📋 Table of Contents

- [About This Guide](#-about-this-guide)
- [Exam Overview](#-exam-overview)
- [Exam Domains](#-exam-domains)
- [How to Use This Guide](#-how-to-use-this-guide)
- [2025–2026 Key Changes](#-2025-2026-key-changes)
- [Study Plan](#-study-plan)
- [Master Quick-Reference Card](#-master-quick-reference-card)
- [Additional Resources](#-additional-resources)
- [License](#-license)

---

## 📖 About This Guide

This repository contains a **complete, exam-focused study guide** for the **Google Professional Cloud Architect (PCA) certification — 2026 edition**. Every guide is written like a **senior engineer coaching you the night before the exam** — opinionated, concise, and built around real exam patterns.

### ✨ What Makes This Different

| Feature | This Guide |
|---|---|
| **Exam-focused** | Built around what the exam *actually* tests, not documentation |
| **Decision frameworks** | "If X → then Y" trees for every major choice |
| **Exam traps** | Common wrong answers and WHY they're wrong |
| **2026 updated** | Flags deprecated services, new services, changed patterns |
| **Practice questions** | 90+ scenario-style questions with detailed explanations |
| **Mnemonics** | Custom memory hacks for every domain |
| **Cheat sheets** | Tables, not walls of text |

---

## 🎯 Exam Overview

| Attribute | Detail |
|---|---|
| **Exam name** | Professional Cloud Architect |
| **Exam code** | PCA |
| **Format** | 50–60 multiple choice / multiple select |
| **Duration** | 2 hours |
| **Passing score** | ~70% (Google does not publish exact score) |
| **Cost** | $200 USD |
| **Validity** | 2 years |
| **Case studies** | TerramEarth, Mountkirk Games, Helicopter Racing League, EHR Healthcare |
| **Languages** | English, Japanese |
| **Delivery** | Online proctored or test center |

### ⚠️ 2025–2026 Notable Changes vs Older Guides

> These services/patterns have changed — old study materials may mislead you.

| ❌ Old (Avoid) | ✅ New (Use) |
|---|---|
| Deployment Manager | **Terraform / Infrastructure Manager** |
| Container Registry (GCR) | **Artifact Registry** |
| Classic VPN | **HA VPN** |
| Preemptible VMs | **Spot VMs** (no 24-hr cap) |
| Anthos | **GKE Enterprise** |
| AI Platform | **Vertex AI** |
| Cloud DLP | **Sensitive Data Protection** |
| Data Catalog | **Dataplex Catalog** |
| Flat-rate BigQuery | **BigQuery Editions (Standard/Enterprise/Plus)** |
| Downloaded SA Keys | **Workload Identity Federation** |

---

## 📚 Exam Domains

The exam is divided into 6 official domains. Each has a dedicated deep-dive guide in this repository.

| # | Domain | Weight | Guide | Questions |
|---|---|---|---|---|
| 1 | [Designing and Planning a Cloud Solution Architecture](./domains/Domain_1_Designing_and_Planning.md) | **~24%** | 15 sections | 15 scenarios |
| 2 | [Managing and Provisioning a Cloud Solution Infrastructure](./domains/Domain_2_Managing_and_Provisioning.md) | **~15%** | 15 sections | 15 scenarios |
| 3 | [Designing for Security and Compliance](./domains/Domain_3_Security_and_Compliance.md) | **~18%** | 17 sections | 15 scenarios |
| 4 | [Analyzing and Optimizing Technical and Business Processes](./domains/Domain_4_Analyzing_and_Optimizing.md) | **~18%** | 15 sections | 15 scenarios |
| 5 | [Managing Implementation](./domains/Domain_5_Managing_Implementation.md) | **~11%** | 16 sections | 15 scenarios |
| 6 | [Ensuring Solution and Operations Reliability](./domains/Domain_6_Reliability.md) | **~14%** | 12 sections | 15 scenarios |

### Domain 1 — Designing and Planning (~24%)

> **Heaviest domain. Master this first.**

Covers: Business & technical requirements, resource hierarchy, region/zone strategy, compute selection, storage & database selection, data architecture, network topology, migration planning (6 R's), hybrid and multi-cloud.

**Key topics:**
- Compute ladder: Cloud Run Functions → Cloud Run → GKE Autopilot → GKE → GCE
- Database selection: Spanner vs AlloyDB vs Cloud SQL vs Bigtable vs Firestore
- Migration strategies: 6 R's (Rehost, Replatform, Refactor, Repurchase, Retire, Retain)
- Network topology: Shared VPC vs VPC Peering vs Network Connectivity Center
- Hybrid connectivity: HA VPN vs Dedicated Interconnect vs Partner Interconnect

[📖 Open Domain 1 Guide →](./domains/Domain_1_Designing_and_Planning.md)

---

### Domain 2 — Managing and Provisioning (~15%)

Covers: IaC tooling (Terraform, Infrastructure Manager), MIG autoscaling, GKE scaling (HPA/VPA/CA), Cloud Run scaling, load balancer types, networking provisioning, hybrid connectivity setup.

**Key topics:**
- IaC: Terraform + Infrastructure Manager (Deployment Manager is deprecated)
- 9 load balancer types and when to pick each
- MIG autoscaling signals (CPU, LB, Pub/Sub, custom metrics, schedule)
- HA VPN (2 tunnels + BGP = 99.99%), Dedicated IC (4× 10G across 2 metros = 99.99%)
- Labels vs Tags (cost attribution vs policy enforcement)

[📖 Open Domain 2 Guide →](./domains/Domain_2_Managing_and_Provisioning.md)

---

### Domain 3 — Security and Compliance (~18%)

Covers: IAM design, service accounts, Workload Identity Federation, encryption (CMEK/EKM/HSM), VPC Service Controls, Cloud Armor, IAP, Secret Manager, Assured Workloads, audit logging.

**Key topics:**
- WIF always beats downloaded SA keys
- VPC Service Controls = data exfiltration prevention (not a firewall)
- CMEK vs EKM (Google can use CMEK; EKM = you control the key)
- Assured Workloads for HIPAA/FedRAMP/ITAR/CJIS
- Data Access audit logs are OFF by default

[📖 Open Domain 3 Guide →](./domains/Domain_3_Security_and_Compliance.md)

---

### Domain 4 — Analyzing and Optimizing (~18%)

Covers: Cost optimization (CUDs, Spot VMs, rightsizing, lifecycle), BigQuery tuning (partitioning, clustering, MV, BI Engine), network egress, FinOps (labels, billing export, budgets), SDLC/DevOps optimization.

**Key topics:**
- 8 levers of GCP cost optimization
- BigQuery: Partition → Cluster → Materialized Views → BI Engine
- CUD vs SUD (CUD = contract; SUD = automatic)
- Flexible CUDs for evolving workloads
- DORA metrics for DevOps velocity

[📖 Open Domain 4 Guide →](./domains/Domain_4_Analyzing_and_Optimizing.md)

---

### Domain 5 — Managing Implementation (~11%)

Covers: CI/CD pipelines, Cloud Build (including Private Pools), Artifact Registry, Cloud Deploy, deployment strategies (rolling/blue-green/canary), supply chain security (Binary Authorization, SLSA), API management (Apigee vs API Gateway).

**Key topics:**
- Container Registry is deprecated → Artifact Registry
- Cloud Deploy for managed progressive delivery (GKE, Cloud Run, GCE)
- Binary Authorization for supply chain security
- SLSA levels 1–4
- Apigee (enterprise+monetization) vs API Gateway (simple+serverless)

[📖 Open Domain 5 Guide →](./domains/Domain_5_Managing_Implementation.md)

---

### Domain 6 — Ensuring Solution and Operations Reliability (~14%)

Covers: SRE fundamentals (SLI/SLO/SLA/error budgets), HA architecture (MIGs, probes, LBs), DR strategies (RPO/RTO), Cloud SQL failover, Spanner multi-region, observability (Monitoring, Logging, Trace), hotspot prevention.

**Key topics:**
- HA ≠ DR (HA = within-region; DR = cross-region catastrophic failure)
- Liveness vs Readiness vs Startup probes (critical distinction)
- RPO/RTO → DR strategy (Backup → Pilot → Warm → Hot)
- Spanner multi-region for zero-RPO (Cloud SQL is async cross-region)
- Bigtable/GCS hotspot prevention via key design

[📖 Open Domain 6 Guide →](./domains/Domain_6_Reliability.md)

---


## 🚀 How to Use This Guide

### Option 1: Start from scratch (recommended for beginners)

```
Week 1 → Domain 1 (Designing & Planning) — largest domain
Week 2 → Domain 3 (Security & Compliance)
Week 3 → Domain 4 (Optimizing) + Domain 2 (Provisioning)
Week 4 → Domain 5 (Implementation) + Domain 6 (Reliability)
Week 5 → Quick Reference Card + Practice tests + Case studies
```

---

## 🔄 2025–2026 Key Changes

These are the most impactful changes for the 2026 exam that older study guides miss:

### ❌ Deprecated / Avoid in Answers

| Service | Status | Replacement |
|---|---|---|
| **Deployment Manager** | Deprecated | Terraform + Infrastructure Manager |
| **Container Registry (GCR)** | Deprecated | Artifact Registry |
| **Cloud VPN Classic** | Deprecated | HA VPN |
| **Preemptible VMs** | Renamed | Spot VMs (no 24-hr cap) |
| **AI Platform** | Renamed | Vertex AI |
| **Cloud DLP API** | Renamed | Sensitive Data Protection |
| **Data Catalog** | Merged | Dataplex Catalog |
| **Flat-rate BQ** | Deprecated | BigQuery Editions |
| **Cloud Debugger** | Deprecated | Cloud Trace + Error Reporting |
| **Cloud Functions 1st gen** | Legacy | Cloud Run Functions (2nd gen) |

### ✅ New Services to Know

| Service | What It Is | Exam Relevance |
|---|---|---|
| **Infrastructure Manager** | Managed Terraform runner | IaC questions |
| **AlloyDB** | PostgreSQL HTAP | DB selection questions |
| **Hyperdisk** | Premium block storage | Storage questions |
| **Parallelstore** | Lustre-based HPC FS | AI/HPC storage |
| **Network Connectivity Center** | Transitive VPC mesh | Networking topology |
| **Private Service Connect (PSC)** | Private API consumption | Connectivity |
| **Cross-Cloud Interconnect** | GCP ↔ AWS/Azure/OCI | Multi-cloud |
| **GKE Autopilot** | Fully managed GKE | Compute selection |
| **Cloud Deploy** | Managed CD pipelines | CI/CD |
| **BigQuery Editions** | Standard/Enterprise/Plus | BQ pricing |
| **Spanner Graph** | Graph queries on Spanner | Data |
| **Backup and DR Service** | Managed backup (ex-Actifio) | DR |
| **GCS Soft Delete** | 7-day object recovery | Storage |
| **Workload Identity Federation** | External → GCP creds | Security |
| **IAM Deny Policies** | Explicit deny > allow | Security |

---

## 📅 Study Plan

### 4-Week Study Plan

```
╔══════════════════════════════════════════════════════════════╗
║                    WEEK 1 — Foundation                       ║
╠══════════════════════════════════════════════════════════════╣
║  Mon  → Domain 1: Requirements + Resource Hierarchy           ║
║  Tue  → Domain 1: Compute + Storage Selection                 ║
║  Wed  → Domain 1: Networking + Migration                      ║
║  Thu  → Domain 1: Practice Checkpoint (all 15 Qs)             ║
║  Fri  → Domain 2: IaC + Autoscaling                          ║
║  Sat  → Domain 2: Networking + Load Balancers                 ║
║  Sun  → Review + flashcards for Weeks 1 decisions            ║
╠══════════════════════════════════════════════════════════════╣
║                    WEEK 2 — Security                         ║
╠══════════════════════════════════════════════════════════════╣
║  Mon  → Domain 3: IAM + Service Accounts + WIF               ║
║  Tue  → Domain 3: Encryption (CMEK/EKM/HSM)                  ║
║  Wed  → Domain 3: VPC-SC + Cloud Armor + IAP                  ║
║  Thu  → Domain 3: Compliance + Audit Logs                     ║
║  Fri  → Domain 3: Practice Checkpoint (all 15 Qs)             ║
║  Sat  → Take 1 full practice exam (see resources below)       ║
║  Sun  → Review wrong answers against domain guides            ║
╠══════════════════════════════════════════════════════════════╣
║                    WEEK 3 — Optimization                     ║
╠══════════════════════════════════════════════════════════════╣
║  Mon  → Domain 4: Cost Optimization (CUD, Spot, Rightsize)    ║
║  Tue  → Domain 4: BigQuery Deep Dive                         ║
║  Wed  → Domain 4: FinOps + DevOps + SDLC                     ║
║  Thu  → Domain 5: CI/CD + Cloud Build + Artifact Registry     ║
║  Fri  → Domain 5: Cloud Deploy + Strategies + BinAuth        ║
║  Sat  → Domain 5: API Management + Practice Qs               ║
║  Sun  → Review + update flashcards                           ║
╠══════════════════════════════════════════════════════════════╣
║                    WEEK 4 — Reliability + Final Prep         ║
╠══════════════════════════════════════════════════════════════╣
║  Mon  → Domain 6: SRE + HA Architecture                      ║
║  Tue  → Domain 6: DR Strategies (RPO/RTO)                    ║
║  Wed  → Domain 6: Scalability + Observability                ║
║  Thu  → All 4 Case Studies (TerramEarth, Mountkirk, etc.)    ║
║  Fri  → Full practice exam #2                                 ║
║  Sat  → Master Quick-Reference Card cover-to-cover           ║
║  Sun  → Rest. Light review of traps. Early to bed.            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## 🗺️ Master Quick-Reference Card

> Open the [Master Guide](./quick-reference/GCP_PCA_2026_MasterGuide.md) for the full 50+ row scenario → solution table.

**High-frequency scenario signals (memorize these):**

| Scenario Signal | Answer |
|---|---|
| "Global strongly consistent SQL" | **Spanner multi-region** |
| "IoT / time-series / telemetry" | **Bigtable** |
| "Mobile offline sync + real-time" | **Firestore Native** |
| "PostgreSQL, HTAP, 4× faster" | **AlloyDB** |
| "Hadoop / Spark existing" | **Dataproc** |
| "Greenfield streaming ETL" | **Dataflow** |
| "Lift-and-shift VMs" | **GCE + Migrate to Virtual Machines** |
| "Containers, scale-to-zero, serverless" | **Cloud Run** |
| "No long-lived keys, external CI/CD" | **Workload Identity Federation** |
| "HIPAA / FedRAMP / ITAR" | **Assured Workloads** |
| "Data exfiltration prevention" | **VPC Service Controls** |
| "WAF / DDoS / OWASP" | **Cloud Armor** |
| "Zero-trust web app, replace VPN" | **IAP** |
| "99.99% SLA" | **Multi-zone/multi-region + Global LB** |
| "99.999% SLA database" | **Spanner multi-region** |
| "IaC in 2026" | **Terraform + Infrastructure Manager** |
| "Progressive delivery GKE/Cloud Run" | **Cloud Deploy** |
| "Enforce signed images" | **Binary Authorization** |
| "Batch fault-tolerant, cheapest" | **Spot VMs** |
| "Compliance logs long-term" | **Log sink → GCS + locked retention** |

---

## 📚 Additional Resources

### Official Google Resources

| Resource | Link |
|---|---|
| Official Exam Guide | [cloud.google.com/certification/guides/professional-cloud-architect](https://cloud.google.com/certification/guides/professional-cloud-architect) |
| Official Practice Exam | [cloud.google.com/certification/practice-exam](https://cloud.google.com/certification) |
| Case Studies | [TerramEarth](https://services.google.com/fh/files/blogs/master_case_study_terramearth.pdf) · [Mountkirk](https://services.google.com/fh/files/blogs/master_case_study_mountkirk_games.pdf) · [HRL](https://services.google.com/fh/files/blogs/master_case_study_helicopter_racing_league.pdf) · [EHR](https://services.google.com/fh/files/blogs/master_case_study_ehr_healthcare.pdf) |
| GCP Documentation | [cloud.google.com/docs](https://cloud.google.com/docs) |
| Architecture Center | [cloud.google.com/architecture](https://cloud.google.com/architecture) |

### Practice Exams (Third-Party)

| Platform | Notes |
|---|---|
| **Whizlabs** | Large question bank, scenario-heavy |
| **TutorialsDojo** | High quality, well-explained |
| **ExamTopics** | Community-driven, debate answers carefully |
| **Udemy (various)** | Course + practice bundle |

### Hands-On Labs

| Platform | Notes |
|---|---|
| **Google Cloud Skills Boost** | Official labs, free tier available |
| **Qwiklabs** | Same as Skills Boost |
| **A Cloud Guru** | Good GCP PCA learning path |
| **Cloud Academy** | Structured paths |

---

---


## 📊 Coverage Status

| Domain | Guide | Practice Qs | Mnemonics | Updated 2026 |
|---|---|---|---|---|
| Domain 1 | ✅ Complete | ✅ 15 | ✅ 6 | ✅ |
| Domain 2 | ✅ Complete | ✅ 15 | ✅ 7 | ✅ |
| Domain 3 | ✅ Complete | ✅ 15 | ✅ 8 | ✅ |
| Domain 4 | ✅ Complete | ✅ 15 | ✅ 8 | ✅ |
| Domain 5 | ✅ Complete | ✅ 15 | ✅ 7 | ✅ |
| Domain 6 | ✅ Complete | ✅ 15 | ✅ 10 | ✅ |
| Case Studies | 🚧 In Progress | — | — | — |
| Extra Practice Tests | 🚧 In Progress | — | — | — |

---

## ⚖️ License

This project is licensed under the **MIT License** — see the [LICENSE](./LICENSE) file for details.

You are free to:
- Use this material for personal study
- Share with colleagues and study groups
- Fork and adapt for your own use
- Include in courses with attribution

---

<div align="center">

**Made with ❤️ for the GCP community**

If this guide helps you pass the exam, consider ⭐ starring the repo and sharing with others!

[![GitHub Stars](https://img.shields.io/github/stars/YOUR_USERNAME/gcp-pca-2026-study-guide?style=social)](https://github.com/YOUR_USERNAME/gcp-pca-2026-study-guide)
[![GitHub Forks](https://img.shields.io/github/forks/YOUR_USERNAME/gcp-pca-2026-study-guide?style=social)](https://github.com/YOUR_USERNAME/gcp-pca-2026-study-guide)

*Good luck on the exam! 🚀*

</div>
