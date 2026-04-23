# 🏗️ Domain 1: Designing and Planning a Cloud Solution Architecture

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (1 of 6)**

**Exam Weight:** ~24% (the HEAVIEST domain — master this one)

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [Business & Technical Requirements Gathering](#-business--technical-requirements-gathering)
3. [Resource Hierarchy & Organization Design](#-resource-hierarchy--organization-design)
4. [Region & Zone Strategy](#-region--zone-strategy)
5. [Compute Service Selection (Deep Dive)](#-compute-service-selection-deep-dive)
6. [Storage & Database Selection (Deep Dive)](#-storage--database-selection-deep-dive)
7. [Data & Analytics Architecture](#-data--analytics-architecture)
8. [Network Topology Design](#-network-topology-design)
9. [Migration Planning (6 R's)](#-migration-planning-6-rs)
10. [Hybrid & Multi-Cloud Architecture](#️-hybrid--multi-cloud-architecture)
11. [Exam Traps & Tricks](#-exam-traps--tricks)
12. [Decision Frameworks](#-decision-frameworks)
13. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
14. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
15. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

The PCA exam wants to see you **translate fuzzy business language into concrete GCP architecture**. You will face scenarios like:

- "A startup with 5 engineers needs to launch a global e-commerce platform in 3 months"
- "A bank with strict EU data residency needs to modernize a 15-year-old Java monolith"
- "A gaming company expects a 100× traffic spike during a launch event"

The exam tests these sub-skills:

1. **Requirements interpretation** — turning "low latency" / "high availability" / "compliance" into specific SLAs, RTOs, and service choices
2. **Trade-off analysis** — balancing cost vs. performance vs. operational overhead vs. time-to-market
3. **Service selection** — picking the **minimum-viable, most-managed** service that meets the requirement
4. **Topology design** — VPCs, subnets, projects, folders, regions
5. **Migration strategy** — the 6 R's and matching each to timeline + skill level
6. **Future-proofing** — choosing services that can scale 10× without re-architecting
7. **Stakeholder alignment** — finance (cost), security (compliance), dev (velocity), ops (reliability)

---

## 🏢 Business & Technical Requirements Gathering

### 🧠 The 4 Requirement Categories

| Category | Examples | GCP Answer Trigger |
|---|---|---|
| **Business** | Time-to-market, budget, regulatory, revenue model | → Drives managed services, compliance products |
| **Functional** | Features, user flows, integrations | → Drives compute + data choices |
| **Non-Functional (NFR)** | Latency, throughput, availability, scale | → Drives SLA tier, region topology |
| **Compliance** | HIPAA, PCI-DSS, GDPR, FedRAMP, data residency | → Drives Assured Workloads, CMEK, VPC-SC |

### 📝 Reading Between the Lines — Keyword → Signal

| Scenario Phrase | What It Really Means | Typical Answer |
|---|---|---|
| "Global customers" | Multi-region + low latency | Global LB, Spanner, Cloud CDN |
| "Financial transactions" | Strong consistency + audit | Spanner + Audit Logs + CMEK |
| "Tight deadline / 30 days" | No time to refactor | Lift-and-shift (GCE) |
| "Startup / MVP" | Minimize up-front cost + ops | Serverless (Cloud Run, Firestore) |
| "Regulated / audited" | Compliance-first architecture | Assured Workloads + CMEK + VPC-SC |
| "Existing Oracle / mainframe" | Keep or migrate carefully | Bare Metal Solution / Oracle on GCE |
| "Existing Kubernetes" | Container-first | GKE (Standard or Autopilot) |
| "Unpredictable spikes" | Scale-to-zero or burst | Cloud Run, App Engine, Functions |
| "IoT / sensors / millions of devices" | High-throughput ingestion | Pub/Sub + Dataflow + Bigtable |
| "Mobile / offline sync" | Document DB + real-time | Firestore Native |
| "Analytics / dashboards" | OLAP | BigQuery |
| "Microservices" | Loosely coupled | Cloud Run / GKE + Pub/Sub |
| "Data residency in EU" | Sovereignty | `europe-*` region + Assured Workloads EU |
| "99.99% uptime" | Multi-zone minimum | Regional MIG + Global LB |
| "99.999% uptime" | Multi-region required | Spanner multi-region |
| "Team has no DevOps" | Managed everything | Cloud Run, Firestore, BigQuery |
| "Big data analytics" | Petabyte scale | BigQuery + Dataflow |
| "Move from AWS" | Multi-cloud migration | Migrate to VMs, BigQuery Omni, Cross-Cloud Interconnect |

### 🔑 The Priority Pyramid (always in this order)

```
        ┌──────────────────┐
        │   Compliance     │  ← NEVER compromise
        ├──────────────────┤
        │   Reliability    │  ← Meet the SLA
        ├──────────────────┤
        │   Security       │  ← Table stakes
        ├──────────────────┤
        │   Performance    │  ← Meet NFR
        ├──────────────────┤
        │   Cost           │  ← Optimize after
        ├──────────────────┤
        │   Ops Simplicity │  ← Managed > self-managed
        └──────────────────┘
```

**Exam rule:** If you must trade off, compliance > reliability > security > performance > cost > ops simplicity. A "cheap but non-compliant" answer is always wrong.

---

## 🌳 Resource Hierarchy & Organization Design

### 🧠 The Hierarchy

```
Organization (1 per company / domain)
    │
    ├── Folders (optional, up to 10 deep)
    │       │
    │       ├── Sub-folders (e.g., BU → Environment)
    │       │       │
    │       │       ├── Projects (billing + quota boundary)
    │       │       │       │
    │       │       │       └── Resources (VMs, buckets, etc.)
```

### 📊 Hierarchy Design Patterns

| Pattern | Structure | When to Use |
|---|---|---|
| **Environment-based** | Org → Folders(Prod/Stage/Dev) → Projects | Small/medium org, few teams |
| **BU-based** | Org → Folders(Marketing/Finance/Eng) → Sub(Env) → Projects | Enterprise, cost allocation by BU |
| **Hybrid (BU × Env)** | Org → Folders(BU) → Folders(Env) → Projects | Large enterprise |
| **Landing Zone (CFT)** | Prebuilt: Shared Services + Workloads + Security | Regulated / enterprise onboarding |

### 🔑 Project Strategy

**Always use separate projects for:**
- ✅ Different environments (prod, staging, dev)
- ✅ Different applications
- ✅ Different teams with different permissions
- ✅ Different billing attribution needs
- ✅ Isolation for blast radius

**1 project = 1 billing boundary = 1 quota boundary.**

### ⚠️ Common Org Design Mistakes (Exam Traps)

| Mistake | Why Wrong | Right Answer |
|---|---|---|
| All resources in 1 project | No isolation, quota collisions, mixed IAM | Project per app + env |
| No folder structure | Can't apply environment-specific org policies | Use folders per environment |
| IAM at resource level | Unmanageable, inconsistent | IAM at highest consistent level |
| Shared billing account with no labels | No cost visibility | Labels + billing exports to BigQuery |

### ✅ Policy Placement Rule

**Set policy at the highest level where it's uniformly true.**

- **Org level:** "No service account key creation anywhere" → Org Policy constraint
- **Folder level:** "All prod must use CMEK" → Folder policy on Prod folder
- **Project level:** "Only us-central1 allowed" → Project-scope policy
- **Resource level:** "Only user X can read bucket Y" → IAM on bucket

---

## 🌍 Region & Zone Strategy

### 🧠 Geographic Concepts

| Scope | Description | Failure Domain |
|---|---|---|
| **Multi-region** | 2+ regions (e.g., `us`) | Survives regional outage |
| **Dual-region** | 2 specific regions (e.g., `nam4` = Iowa + S. Carolina) | Survives regional outage with chosen regions |
| **Region** | ~3+ zones in a geographic area | Survives zonal outage |
| **Zone** | Isolated datacenter within region | Single point of failure if alone |

### 🔑 Region Selection Framework

**Step 1: Compliance / Data residency?**
- EU-only → `europe-*` region
- US-only → `us-*` region
- India → `asia-south1` / `asia-south2`
- Canada → `northamerica-northeast1/2`

**Step 2: Latency to users?**
- Pick region closest to user population
- For global apps → use Global LB + multi-region backend

**Step 3: Service availability?**
- Not all services in all regions (especially new ones: AlloyDB, Parallelstore, TPU)

**Step 4: Cost?**
- `us-central1` typically cheapest
- European regions slightly pricier
- Tier-1 vs Tier-2 network pricing differs

### 📊 Availability Tiers

| SLA Target | Architecture Required |
|---|---|
| **99.5%** | Single-zone OK (but rare on exam) |
| **99.9%** | Regional (multi-zone) |
| **99.95%** | Regional with HA (e.g., Cloud SQL HA) |
| **99.99%** | Multi-region or regional with LB + MIG across zones |
| **99.999%** | Multi-region active-active (Spanner, multi-region GCS) |

### 🔑 Availability Math (memorize)

| SLA | Downtime/year | Downtime/month |
|---|---|---|
| 99% | 3.65 days | 7.2 hrs |
| 99.9% | 8.76 hrs | 43.2 min |
| 99.95% | 4.38 hrs | 21.6 min |
| 99.99% | 52.6 min | 4.32 min |
| 99.999% | 5.26 min | 26 sec |

**Exam gotcha:** 99.99% is impossible with single-zone resources. Always need multi-zone minimum.

### 🌐 Multi-region GCS vs Dual-region

| Type | You Choose Regions? | Latency | Cost |
|---|---|---|---|
| **Multi-region** (`us`, `eu`, `asia`) | No — Google chooses | Variable | Middle |
| **Dual-region** (e.g., `nam4`) | Yes — pick 2 | Lower (predictable) | Higher |
| **Regional** | Yes — pick 1 | Lowest | Lowest |

---

## 💻 Compute Service Selection (Deep Dive)

### 🧠 The Full Compute Ladder

| Service | Abstraction | Scale-to-Zero | Use Case |
|---|---|---|---|
| **Cloud Run Functions** (gen 2) | Function | ✅ | Event-driven, single-purpose |
| **Cloud Run** | Container | ✅ | HTTP/event containers, stateless |
| **Cloud Run Jobs** | Container (batch) | ✅ (per invocation) | Short batch jobs ≤24 hr |
| **Batch** | Compute workload | ✅ | HPC, long batch, GPU/TPU |
| **App Engine Standard** | Language runtime | ✅ | Web apps in supported runtimes |
| **App Engine Flexible** | Container on GCE | ❌ (min 1) | Custom runtime, WebSockets |
| **GKE Autopilot** | K8s (managed) | ❌ (pod-level scale) | Containers w/ K8s features |
| **GKE Standard** | K8s (self-managed nodes) | ❌ | Full K8s control |
| **GKE Enterprise** (Anthos) | Multi-cluster K8s | ❌ | Hybrid, multi-cloud |
| **GCE MIG** | VMs (fleet) | ❌ (min 0 possible) | VM-based apps |
| **GCE** | VM (single) | ❌ | Full control, legacy |
| **GCE Sole-Tenant** | Dedicated host | ❌ | Licensing, compliance |
| **Bare Metal Solution** | Physical server | ❌ | Oracle RAC, specialized |

### 📊 Compute Decision Matrix (memorize)

| Requirement | Best Service | Second Choice |
|---|---|---|
| Simple HTTP web app, scale-to-zero | **Cloud Run** | App Engine Standard |
| Event handler (GCS, Pub/Sub trigger) | **Cloud Run Functions** | Cloud Run |
| Container-based microservices | **Cloud Run** | GKE Autopilot |
| K8s CRDs, operators, StatefulSets | **GKE Autopilot** | GKE Standard |
| Custom node pools, GPUs, specific k8s versions | **GKE Standard** | GKE Autopilot |
| Multi-cluster / hybrid K8s | **GKE Enterprise** | — |
| Lift-and-shift VMs | **GCE + Migrate to VMs** | MIG |
| Auto-healing, auto-scaling VM fleet | **Regional MIG** | GKE |
| Windows Server | **GCE (Windows images)** | — |
| Licensed software (Oracle, SAP) | **GCE Sole-tenant / Bare Metal** | — |
| Short batch (< 24 hr) | **Cloud Run Jobs** | Batch |
| HPC / GPU / TPU training | **Batch / GKE with Accelerators** | GCE |
| Long-running WebSocket/gRPC | **Cloud Run (with always-on CPU)** | GKE |

### 🔑 Cloud Run vs GKE — The Big Decision

| Factor | Cloud Run | GKE |
|---|---|---|
| **Ops overhead** | ✅ Minimal | ❌ Higher (even Autopilot) |
| **Scale to zero** | ✅ Yes | ❌ No |
| **Pay model** | Per-request | Per-node-hour |
| **Request duration** | 60 min max (jobs: 24hr) | Unlimited |
| **Max memory** | 32 GB | Whatever node size |
| **GPU support** | ✅ Yes (2024+) | ✅ Yes |
| **StatefulSet / PVC** | ❌ No | ✅ Yes |
| **Service mesh** | Limited | ✅ Cloud Service Mesh |
| **Multi-container** | Sidecars supported | ✅ Full pod model |
| **Portability** | OCI containers | K8s YAML (portable) |
| **Best for** | Microservices, APIs, event-driven | Stateful, complex orchestration |

**Exam rule:** Cloud Run first unless scenario explicitly needs K8s features (StatefulSets, DaemonSets, CRDs, operators, service mesh at scale).

### 🔑 GKE Autopilot vs Standard

| Factor | Autopilot | Standard |
|---|---|---|
| **Node management** | Google-managed | You manage |
| **Pricing** | Per-pod | Per-node |
| **Security** | Hardened by default | You harden |
| **Customization** | Limited | Full |
| **Best for** | New workloads, less ops | Advanced use cases |

### 🔑 App Engine Standard vs Flexible

| Factor | Standard | Flexible |
|---|---|---|
| **Runtime** | Supported langs only (Python, Java, Node, Go, PHP, Ruby) | Any Docker image |
| **Scale to zero** | ✅ Yes | ❌ No (min 1) |
| **Cold start** | Fast (~ms) | Slow (~min) |
| **SSH access** | No | Yes |
| **Background processes** | No | Yes |
| **WebSockets** | ❌ No | ✅ Yes |
| **Cost** | Cheaper for idle | More for idle |

**2026 note:** Most new work goes to **Cloud Run** instead of App Engine. But App Engine Standard is still common on exams.

---

## 💾 Storage & Database Selection (Deep Dive)

### 🧠 The Full Storage Landscape

| Category | Services |
|---|---|
| **Object** | Cloud Storage (Standard/Nearline/Coldline/Archive) |
| **Block** | Persistent Disk (Standard/Balanced/SSD/Extreme), Hyperdisk (Balanced/Throughput/Extreme/ML) |
| **File** | Filestore (Basic HDD/SSD, Zonal, Enterprise), Parallelstore |
| **Relational** | Cloud SQL (MySQL/PostgreSQL/SQL Server), AlloyDB (PostgreSQL), Spanner |
| **NoSQL Document** | Firestore (Native / Datastore mode) |
| **NoSQL Wide-column** | Bigtable |
| **Warehouse** | BigQuery, BigLake |
| **In-memory** | Memorystore (Redis, Valkey, Memcached) |

### 📊 Database Decision Matrix (memorize)

| Requirement | Pick | Why Not Others |
|---|---|---|
| Global SQL, strong consistency, 99.999% | **Spanner** | Only option that fits all 3 |
| Regional SQL, MySQL/PostgreSQL-compatible | **Cloud SQL** | Spanner = overkill cost |
| PostgreSQL, high performance, HTAP | **AlloyDB** | Cloud SQL slower for analytics |
| Time-series, IoT, billions of rows | **Bigtable** | Firestore doesn't scale that way |
| Mobile app w/ offline sync + real-time | **Firestore Native** | Datastore mode = no real-time |
| Legacy App Engine app | **Firestore Datastore mode** | Native doesn't support old API |
| Cache layer | **Memorystore** | Not a primary DB |
| Analytics on petabytes | **BigQuery** | OLTP DBs are too slow |
| Graph queries | **Spanner Graph** (2024+) | BigQuery/Neo4j alternatives |
| Vector search (AI/RAG) | **AlloyDB + pgvector** or **Vertex AI Vector Search** | New 2024 patterns |

### 🔑 Spanner vs Cloud SQL vs AlloyDB (the confusion trio)

| Factor | Cloud SQL | AlloyDB | Spanner |
|---|---|---|---|
| **Interface** | MySQL/PG/SQL Server | PostgreSQL-compatible | Custom SQL (Google dialect + PG dialect) |
| **Scale** | Regional, vertical scaling | Regional, better scaling | Global, horizontal |
| **Consistency** | Strong (single primary) | Strong (single primary) | Global strong |
| **Availability SLA** | 99.95% (HA) | 99.99% | 99.999% (multi-region) |
| **Max storage** | ~64 TB | ~128 TB+ | Unlimited |
| **Max QPS** | ~Tens of thousands | Higher than Cloud SQL | Millions |
| **HA failover** | Zone → zone | Zone → zone | Automatic, global |
| **Cost** | $ | $$ | $$$$ |
| **Best for** | Existing workloads, simple | High-perf PG, HTAP | Global critical systems |

### 🔑 Firestore: Native vs Datastore Mode

| Factor | Native | Datastore Mode |
|---|---|---|
| **API** | New SDK | Legacy Datastore API |
| **Real-time listeners** | ✅ Yes | ❌ No |
| **Offline mobile/web** | ✅ Yes | ❌ No |
| **Strong consistency** | Yes (single-doc + queries) | Yes (queries) |
| **Use case** | New mobile/web apps | Legacy App Engine apps |

**Exam rule:** **New project → always Native**. Datastore Mode is legacy-only.

### 🔑 Bigtable Specifics

- **Wide-column NoSQL** (HBase-compatible)
- **Massive scale:** Petabytes, millions of ops/sec
- **Single-digit ms latency**
- **Use cases:** Time-series, IoT, finance, user profiles, recommendations
- **Key design matters!** Random/hashed keys prevent hotspots
- ⚠️ **Not for ad-hoc queries** — only efficient with row key or prefix scan
- ⚠️ **No SQL** (until BigQuery external table integration)

### 🔑 BigQuery — Not Just a Database

| Feature | What It Enables |
|---|---|
| **Serverless SQL** | No infra management |
| **Columnar storage** | Fast analytics |
| **Partition + cluster** | Cost + perf optimization |
| **BigQuery ML** | Train ML models with SQL |
| **BigQuery Omni** | Query data in AWS S3, Azure Blob |
| **BigLake** | Unified table across GCS + BQ |
| **BI Engine** | Sub-second dashboards |
| **Materialized views** | Auto-refresh aggregates |
| **Streaming inserts** | Real-time data ingestion |
| **External tables** | Query GCS, Bigtable, Sheets |
| **Data Transfer Service** | Scheduled loads from SaaS |

### 💾 Cloud Storage Classes (memorize cost/access pattern)

| Class | Min Storage | Retrieval Cost | Use Case |
|---|---|---|---|
| **Standard** | None | Free | Hot data, frequent access |
| **Nearline** | 30 days | $$ | ~Monthly access |
| **Coldline** | 90 days | $$$ | ~Quarterly access |
| **Archive** | 365 days | $$$$ | ~Yearly / compliance only |

**Early deletion fee:** If you delete before min storage duration, charged as if stored for full duration.

**Lifecycle rules:** Automatically transition between classes or delete after N days.

### 💾 Persistent Disk vs Hyperdisk

| Type | IOPS | Throughput | Best For |
|---|---|---|---|
| **PD Standard (HDD)** | Low | Low | Boot, cold data |
| **PD Balanced** | Medium | Medium | General VM workloads |
| **PD SSD** | High | High | DBs, latency-sensitive |
| **PD Extreme** | Very High | High | Very demanding DBs |
| **Hyperdisk Balanced** | Configurable | Configurable | Latest-gen, flexible |
| **Hyperdisk Throughput** | — | Very High | Hadoop, Kafka |
| **Hyperdisk Extreme** | Highest | Highest | High-perf DBs |
| **Hyperdisk ML** | Optimized for AI | Optimized | AI/ML training, multi-attach |

**2025+ exam:** Hyperdisk is the preferred answer for new workloads; PD is legacy path.

### 💾 Filestore vs Parallelstore

| Filestore | Parallelstore |
|---|---|
| NFS for VMs, GKE | Lustre-based parallel FS |
| General file share | HPC, AI training |
| Basic/Zonal/Enterprise | High throughput |

---

## 📊 Data & Analytics Architecture

### 🧠 The Data Pipeline Patterns

**Pattern 1: Streaming Ingestion**
```
Devices/Apps → Pub/Sub → Dataflow → BigQuery / Bigtable
```

**Pattern 2: Batch ETL**
```
Sources → Cloud Storage → Dataflow/Dataproc → BigQuery
```

**Pattern 3: Real-time Analytics**
```
Pub/Sub → Dataflow → BigQuery (streaming insert) → Looker / BI Engine
```

**Pattern 4: Hadoop Migration**
```
On-prem Hadoop → Dataproc (GCS-backed) → BigQuery for analytics
```

**Pattern 5: Data Lakehouse**
```
Sources → GCS (raw) → BigLake → BigQuery (via external tables)
```

### 📊 Data Service Decision Matrix

| Need | Service |
|---|---|
| Async message queue, decouple services | **Pub/Sub** |
| Streaming ETL, Apache Beam | **Dataflow** |
| Batch ETL, Apache Beam | **Dataflow** |
| Existing Hadoop/Spark workload | **Dataproc** |
| Ephemeral Spark/Hadoop clusters | **Dataproc Serverless** |
| Orchestrate complex workflows | **Cloud Composer (Airflow)** |
| Simple step-based workflow | **Workflows** |
| Scheduled triggers | **Cloud Scheduler** |
| CDC from DBs | **Datastream** |
| Massive parallel data loads | **Dataflow** or **BQ Load** |
| Data quality & profiling | **Dataplex** |
| Metadata catalog | **Data Catalog** (now Dataplex Catalog) |
| Data classification (PII) | **Sensitive Data Protection** (formerly DLP) |

### 🔑 Dataflow vs Dataproc vs Dataform

| | Dataflow | Dataproc | Dataform |
|---|---|---|---|
| **Engine** | Apache Beam | Hadoop/Spark | SQL transforms in BQ |
| **Use case** | New streaming/batch pipelines | Lift-and-shift Hadoop | ELT in BigQuery |
| **Management** | Fully managed | Managed clusters | Fully managed |
| **Best for** | Greenfield | Existing Hadoop | BigQuery-native modeling |

### 🔑 Pub/Sub Concepts

| Feature | Description |
|---|---|
| **Topic** | Publishes messages |
| **Subscription** | Receives messages (pull or push) |
| **Ordering key** | Guarantee order within key |
| **Dead-letter topic** | Handle repeatedly failed messages |
| **Message retention** | Up to 31 days |
| **Exactly-once delivery** | ✅ Supported (2023+) |
| **Pub/Sub Lite** | Cheaper, zonal, lower SLA |
| **BigQuery subscription** | Direct write to BQ, no Dataflow needed (2022+) |
| **Cloud Storage subscription** | Direct write to GCS (2024+) |

**Exam tip:** If the scenario says "stream Pub/Sub directly to BigQuery with minimal pipeline code," the answer is **BigQuery subscription**, NOT Dataflow.

---

## 🌐 Network Topology Design

### 🧠 VPC Fundamentals

| Concept | Description |
|---|---|
| **VPC** | Global resource, spans all regions |
| **Subnet** | Regional, has CIDR range |
| **Auto mode** | Auto-creates subnets in every region |
| **Custom mode** | You define subnets ⭐ (recommended) |
| **Firewall rules** | VPC-level; target by tag/SA |
| **Routes** | Default route to internet; custom routes for hybrid |
| **Cloud NAT** | Outbound internet for private VMs |

### 🔑 VPC Connectivity Options

| Option | Use Case | Transitive? |
|---|---|---|
| **Shared VPC** | Central network, multiple service projects | N/A (single VPC) |
| **VPC Peering** | Two VPCs need private connectivity | ❌ No |
| **Network Connectivity Center (NCC)** | Many VPCs, hub-and-spoke | ✅ Yes |
| **Cloud VPN** | VPC ↔ on-prem/other cloud over IPsec | N/A |
| **Interconnect** | VPC ↔ on-prem (physical) | N/A |
| **Private Service Connect (PSC)** | Consume managed service privately | N/A |

### 🔑 Shared VPC Deep Dive

- **Host project**: owns the VPC and subnets
- **Service projects**: use the subnets
- **Permissions**: Network admin at host; app admins in service projects
- **Use case**: Central network team, decentralized app teams
- ⚠️ **Only within 1 organization**

### 🔑 VPC Peering Deep Dive

- **Non-transitive**: A↔B and B↔C does NOT give A↔C
- No bandwidth charge for peered traffic in same region
- Must have **non-overlapping CIDR**
- Supports cross-project and cross-org peering
- **Exam trap:** "Mesh topology for 5 VPCs" = NCC, not peering mesh

### 🔑 Network Connectivity Center (NCC)

- **Hub**: central networking construct
- **Spokes**: VPCs, VPN tunnels, Interconnect VLAN attachments, third-party SD-WAN
- **Transitive routing** between all spokes
- **2025+ preferred answer** for complex multi-VPC topology

### 🔑 Private Google Access vs Private Service Connect

| Feature | Private Google Access (PGA) | Private Service Connect (PSC) |
|---|---|---|
| **Purpose** | VMs without public IP reach Google APIs | Consume Google or 3P services privately |
| **Setup** | Enable on subnet | Create PSC endpoint in your VPC |
| **Traffic flow** | Via google.internal | Via your VPC internal IP |
| **Use case** | Private VMs call GCS/BQ | SaaS integration, Google APIs w/ fixed IPs |

### 🔑 Hybrid Connectivity Matrix

| Option | Bandwidth | SLA | Latency | Cost | When |
|---|---|---|---|---|---|
| **Cloud VPN (Classic)** | ~1.5–3 Gbps | 99.9% | Internet | $ | ⚠️ Deprecated |
| **HA VPN** | 1.5–3 Gbps per tunnel | 99.99% (2 tunnels) | Internet | $ | Quick, low BW |
| **Partner Interconnect** | 50 Mbps – 50 Gbps | 99.9–99.99% | Low | $$ | No colo |
| **Dedicated Interconnect** | 10/100 Gbps | 99.9–99.99% | Lowest | $$$ | High BW, colo |
| **Cross-Cloud Interconnect** | 10/100 Gbps | 99.9–99.99% | Low | $$$ | GCP ↔ AWS/Azure/OCI |
| **Carrier Peering** | Variable | — | Low | $ | Direct to Google edge (no private RFC1918) |
| **Direct Peering** | Variable | — | Low | $ | BGP peering at Google edge |

### 🔑 SLA for Hybrid

- **HA VPN**: 99.99% needs **2 tunnels across 2 interfaces**
- **Dedicated Interconnect**: 99.9% with 1 connection; **99.99% needs 4 connections across 2 metros or 2 edge availability domains**
- **Partner Interconnect**: Depends on redundancy; 99.99% requires full redundancy topology

---

## 🔄 Migration Planning (6 R's)

### 🧠 The 6 R's — Deep Dive

| Strategy | Description | GCP Tooling | Exam Signal |
|---|---|---|---|
| **Rehost (Lift-and-Shift)** | Move VMs as-is | Migrate to VMs | "Tight deadline", "No refactor" |
| **Replatform (Lift-and-Tinker)** | Move with minor mods | Migrate + swap DB, AE Flex, Cloud SQL | "Managed DB but keep app" |
| **Refactor / Re-architect** | Rewrite for cloud-native | Cloud Run, GKE, serverless | "Scale 10×", "Microservices" |
| **Repurchase** | Move to SaaS | Workspace, Looker | "Replace with off-the-shelf" |
| **Retire** | Turn off | — | "No longer needed" |
| **Retain** | Keep in place | — | "Mainframe", "Compliance locks it in" |

### 🔑 Migration Service Selection

| What You're Migrating | Service |
|---|---|
| VMware VMs → GCE | **Migrate to Virtual Machines** (formerly Migrate for Compute Engine) |
| VMware → VMware Engine (GCVE) | **Google Cloud VMware Engine** |
| Containers → GKE | **Migrate to Containers** |
| MySQL/PostgreSQL/SQL Server → Cloud SQL | **Database Migration Service (DMS)** |
| Oracle → PostgreSQL/Spanner | **DMS** (heterogeneous, limited) or Striim |
| Data from on-prem → GCS | **Transfer Appliance** (petabyte-scale), **Storage Transfer Service** |
| Data from AWS S3 / Azure | **Storage Transfer Service** |
| Data from HDFS | **Dataproc** or **Transfer Service for HDFS** |
| Mainframe | **Google Cloud Mainframe Connector** or 3P |

### 🔑 Transfer Service Decision

| Data Size | Link Speed | Service |
|---|---|---|
| < 1 TB | Fast link | **gsutil / gcloud storage cp** |
| TB to PB | Decent link | **Storage Transfer Service** |
| Tens of TB+ | Slow link | **Transfer Appliance** (ship physical device) |
| Petabytes+ | Any | **Transfer Appliance** |

### 🔑 Migration Phases (exam loves this)

1. **Assess** — catalog apps, dependencies, capacity
2. **Plan** — choose strategy per app (6 R's)
3. **Migrate** — execute; iterative pilots first
4. **Optimize** — right-size, refactor hot spots, enable CUDs
5. **Operate** — SRE practices, monitoring, cost governance

---

## ☁️ Hybrid & Multi-Cloud Architecture

### 🧠 Multi-Cloud Building Blocks

| Tool | Purpose |
|---|---|
| **GKE Enterprise (Anthos)** | Manage K8s clusters on GCP, AWS, Azure, on-prem |
| **Cloud Service Mesh** | Istio-based mesh for multi-cluster |
| **BigQuery Omni** | Query S3/Azure Blob from BQ |
| **Cross-Cloud Interconnect** | Private link GCP ↔ AWS/Azure/OCI |
| **Storage Transfer Service** | Data movement across clouds |
| **Looker** | BI across any data source |
| **Cloud WAN** (2025+) | Global private backbone for enterprises |

### 🔑 When to Go Multi-Cloud

- ✅ Regulatory requirement (avoid vendor lock-in)
- ✅ Best-of-breed services
- ✅ Geographic coverage (one cloud weak in a region)
- ✅ Acquired company on another cloud
- ❌ Just for the sake of it — multi-cloud adds complexity

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 15 Traps in Domain 1

1. **"Global" + "strong consistency" + "SQL"** → **Spanner** (never Cloud SQL)
2. **"Lift-and-shift"** → **GCE + Migrate to VMs** (never GKE/Cloud Run)
3. **"Hadoop / Spark on-prem"** → **Dataproc** (never Dataflow)
4. **"Mesh / transitive connectivity"** → **NCC** (never VPC peering mesh)
5. **"Data residency EU"** → regional `europe-*` + possibly Assured Workloads
6. **"Scale to zero"** → Cloud Run / App Engine Standard / Functions (never GKE Standard)
7. **"99.99%"** → must be multi-zone minimum (regional MIG + Global LB)
8. **"99.999%"** → must be multi-region (Spanner multi-region)
9. **"Real-time mobile sync"** → Firestore Native (never Datastore mode for new)
10. **"Container Registry"** → ⚠️ **deprecated**, use **Artifact Registry**
11. **"Deployment Manager"** → ⚠️ **deprecated**, use **Terraform**
12. **"Firestore Datastore mode"** → only for legacy App Engine apps
13. **"Mainframe"** → cannot migrate easily → **Retain** or use Mainframe Connector
14. **"Oracle RAC"** → **Bare Metal Solution**
15. **"100 Gbps, low latency, private"** → **Dedicated Interconnect** (never VPN)

### ⚠️ "Sounds Right but Isn't" Options

| Seems Right | Actually Right | Why |
|---|---|---|
| Cloud SQL multi-region | Spanner multi-region | Cloud SQL has no true multi-region active-active |
| VPC Peering mesh for 5 VPCs | Network Connectivity Center | Peering is non-transitive |
| App Engine for containers | Cloud Run | App Engine Flex still runs containers but heavier |
| GKE for a single microservice | Cloud Run | GKE ops overhead too high |
| Cloud Functions for 10-min job | Cloud Run Jobs | Functions timeout at 9/60 min (gen 2) |
| Storage Transfer for 5 PB | Transfer Appliance | Over-wire too slow |
| Dataflow for existing Spark | Dataproc | Spark = Dataproc |
| VPN for 20 Gbps | Dedicated Interconnect | VPN caps ~3 Gbps |
| Bigtable for simple CRUD | Firestore | Bigtable wants careful key design |

---

## 🔑 Decision Frameworks

### 🎯 The Ultimate Service Picker

**Start with "what's the workload?"**

```
Is it compute?
  ├── Stateless HTTP?
  │     ├── Scale-to-zero OK? → Cloud Run
  │     └── Always-on? → GKE / App Engine Flex
  ├── Event-driven function?
  │     └── → Cloud Run Functions (gen 2)
  ├── Batch job?
  │     ├── < 24 hr? → Cloud Run Jobs
  │     └── HPC / long? → Batch
  ├── K8s features (StatefulSet, CRD)?
  │     └── → GKE Autopilot (default) / Standard (advanced)
  ├── Full VM control / licensed software?
  │     └── → GCE (MIG for scale)
  └── Hybrid / multi-cloud K8s?
        └── → GKE Enterprise

Is it data?
  ├── Relational?
  │     ├── Global? → Spanner
  │     ├── PG high-perf? → AlloyDB
  │     └── Regional standard? → Cloud SQL
  ├── NoSQL document?
  │     └── → Firestore Native
  ├── NoSQL wide-column / time-series?
  │     └── → Bigtable
  ├── Analytics / warehouse?
  │     └── → BigQuery
  ├── Object blob?
  │     └── → Cloud Storage (class by access pattern)
  ├── Cache?
  │     └── → Memorystore
  ├── Block for VM?
  │     └── → Persistent Disk / Hyperdisk
  └── File share?
        └── → Filestore / Parallelstore (HPC)

Is it networking?
  ├── 2 VPCs? → VPC Peering
  ├── 5+ VPCs transitive? → Network Connectivity Center
  ├── Central team, many projects? → Shared VPC
  ├── On-prem ≤ 3 Gbps? → HA VPN
  ├── On-prem 10+ Gbps? → Dedicated Interconnect
  └── Multi-cloud direct? → Cross-Cloud Interconnect
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "SPAGHETTI" (Database Picker)
- **S**panner → Global SQL
- **P**ostgreSQL → Cloud SQL or **A**lloyDB
- **G**iant time-series → Bi**g**table
- **H**ierarchical docs → Firestore
- **E**nterprise analytics → BigQuery
- **T**erabytes streaming → Dataflow → BQ
- **T**ransient cache → Memorystore
- **I**mages/blobs → Cloud Storage

### 🧠 "6 R's" (Migration)
**R**ehost, **R**eplatform, **R**efactor, **R**epurchase, **R**etire, **R**etain

### 🧠 "CRAG"
Compute ops-burden ladder (low → high):
**C**loud Run → **R**unnables (App Engine) → **A**utopilot (GKE) → **G**CE

### 🧠 "PASS"
Storage duration by access pattern:
- **P**ersistent (Standard) = daily
- **A**lmost daily (Nearline) = monthly
- **S**omewhat (Coldline) = quarterly
- **S**eldom (Archive) = yearly+

### 🧠 "HANDS"
HA VPN tunnel SLA:
**HA** ≥ **2** tunnels = **99.99%**

### 🧠 "OFPR" (Hierarchy)
**O**rganization → **F**older → **P**roject → **R**esource

---

## ✅ Practice Checkpoint

### Q1. Global banking app — financial transactions
**Scenario:** A bank needs a database for global transactions. Strong consistency, 99.999% SLA, horizontal scale, SQL interface.

**Options:**
- A) Cloud SQL MySQL with cross-region read replicas
- B) Spanner multi-region configuration (e.g., `nam-eur-asia1`)
- C) Bigtable with multi-cluster replication
- D) AlloyDB with cross-region replication

✅ **Answer: B** — Only **Spanner** provides global strong consistency + 99.999% SLA + horizontal scale + SQL. AlloyDB is regional PG. Cloud SQL replicas aren't globally consistent. Bigtable isn't SQL.

---

### Q2. Startup MVP
**Scenario:** 3-person startup, MVP in 6 weeks, web app + mobile app, expect 100 users initially, must scale if viral, zero ops team.

**Options:**
- A) GKE Standard + Cloud SQL + React frontend on GCS + Cloud CDN
- B) Cloud Run (backend) + Firestore (DB) + Firebase Hosting (frontend)
- C) GCE VMs + self-managed Postgres + nginx
- D) App Engine Flexible + Cloud SQL

✅ **Answer: B** — Zero ops, scale-to-zero, real-time mobile sync from Firestore, Firebase hosting for front. Cheapest + fastest to build. Answer A has too much ops for 3 people.

---

### Q3. Lift-and-shift 200 VMs
**Scenario:** Enterprise must migrate 200 Linux VMs running Java monoliths from on-prem VMware to GCP in 60 days. No refactoring budget.

**Options:**
- A) Refactor all apps to Cloud Run
- B) GKE with Migrate to Containers
- C) Migrate to Virtual Machines → GCE
- D) Rewrite on App Engine Standard

✅ **Answer: C** — "No refactoring budget" + "60 days" = rehost. **Migrate to VMs** is purpose-built.

---

### Q4. IoT telemetry
**Scenario:** Fleet of 2M IoT sensors sending 1KB/sec per device. Queries: "show me last 24 hrs of sensor X," "avg temperature region Y last month." 

**Options:**
- A) BigQuery
- B) Cloud SQL
- C) Firestore
- D) Bigtable for hot data + BigQuery for analytics

✅ **Answer: D** — **Hybrid pattern**. Bigtable handles high-velocity writes + time-series lookups; BigQuery handles ad-hoc analytics. Using only BQ would mean high latency for per-device lookups.

---

### Q5. EU data residency
**Scenario:** SaaS serving EU customers; GDPR says all PII must never leave EU; company is US-headquartered.

**Options:**
- A) Deploy in `us-central1` with VPC Service Controls
- B) Deploy in `europe-west1` with Assured Workloads for EU + regional services
- C) Multi-region `us` + encryption
- D) `europe-west1` + IAM only

✅ **Answer: B** — Data residency requires **regional EU deployment**; **Assured Workloads** for EU adds guardrails (Google personnel from EU only, etc.). VPC-SC alone doesn't enforce data location.

---

### Q6. Existing Hadoop cluster on-prem
**Scenario:** Company has 500-node on-prem Hadoop cluster running Hive and Spark jobs. Wants to migrate with minimal code changes.

**Options:**
- A) Dataflow with Apache Beam rewrites
- B) Dataproc + GCS for storage
- C) BigQuery with federated queries
- D) GKE with custom Spark operators

✅ **Answer: B** — **Dataproc = managed Hadoop/Spark**. Pair with GCS (not HDFS) for cost and decoupling. Dataflow is Beam, not Spark.

---

### Q7. 5 VPCs need full-mesh connectivity
**Scenario:** 5 VPCs across 3 projects need any-to-any private connectivity including on-prem via HA VPN.

**Options:**
- A) VPC Peering mesh (10 peerings)
- B) Shared VPC
- C) Network Connectivity Center with VPC + VPN spokes
- D) Cloud VPN between every VPC pair

✅ **Answer: C** — **NCC** gives transitive routing + unified management. VPC peering mesh is non-transitive and has quadratic complexity.

---

### Q8. Spiky traffic web app
**Scenario:** E-commerce backend. 95% of the time handles 10 RPS; flash sales spike to 10,000 RPS for 2 hours. Stateless.

**Options:**
- A) GKE Standard with 10 always-on nodes
- B) Cloud Run with min-instances=0, max=1000
- C) GCE n2-standard-32 VMs with CUDs
- D) App Engine Standard with manual scaling

✅ **Answer: B** — **Cloud Run** scales to zero during idle, up to max during spike, pay per request. Most cost-effective.

---

### Q9. 99.99% SLA stack
**Scenario:** Global web app needs 99.99% availability with global low-latency.

**Options:**
- A) Single region GKE with regional LB
- B) Multi-region GKE clusters + Global External Application LB + Spanner
- C) App Engine Standard in us-central1
- D) GCE MIG in a single zone

✅ **Answer: B** — 99.99% + global = multi-region, GCLB for global LB, Spanner for global DB.

---

### Q10. Licensed Windows software
**Scenario:** Must run a licensed Windows Server 2019 app that requires the server to be on dedicated hardware for license compliance.

**Options:**
- A) GCE standard VM with Windows image
- B) GCE Sole-tenant Node with BYOL Windows
- C) GKE Windows node pool
- D) App Engine Flexible

✅ **Answer: B** — **Sole-tenant nodes** provide dedicated hardware required for many per-core licenses. Standard GCE is multi-tenant.

---

### Q11. Mobile app offline-first
**Scenario:** A mobile app for field technicians. Works offline, syncs data when online. Real-time updates when multiple techs view same job.

**Options:**
- A) Cloud SQL + custom sync logic
- B) Firestore Native (offline + real-time)
- C) Bigtable
- D) Cloud Storage

✅ **Answer: B** — **Firestore Native** has built-in offline persistence + real-time listeners + multi-user sync.

---

### Q12. Compliance + encryption
**Scenario:** Healthcare SaaS. PHI storage. Must meet HIPAA, log all access, use customer-managed keys, block cross-region exfiltration.

**Options:**
- A) CMEK only
- B) Assured Workloads for HIPAA + CMEK + VPC Service Controls + Audit Logs (Data Access enabled)
- C) Default encryption + IAM
- D) CSEK on GCS

✅ **Answer: B** — Defense-in-depth: Assured Workloads pre-configures HIPAA controls, CMEK for key control, VPC-SC for exfiltration, Data Access logs for audit.

---

### Q13. Petabyte data transfer
**Scenario:** Need to move 2 PB of video from on-prem to GCS. WAN is 1 Gbps.

**Options:**
- A) gsutil cp
- B) Storage Transfer Service over WAN
- C) Transfer Appliance (physical device)
- D) Cloud VPN + BigQuery Data Transfer

✅ **Answer: C** — Over 1 Gbps WAN, 2 PB takes ~185 days. **Transfer Appliance** ships physical storage; typically <1 week. Scale math matters.

---

### Q14. SQL database migration
**Scenario:** Migrate 4 TB Oracle DB from on-prem to GCP with minimal effort. Want managed PostgreSQL-compatible destination.

**Options:**
- A) Database Migration Service → AlloyDB
- B) Manual dump/restore → Cloud SQL MySQL
- C) Dataflow job
- D) Storage Transfer Service

✅ **Answer: A** — **DMS** supports Oracle → PostgreSQL migration (heterogeneous). **AlloyDB** is PostgreSQL-compatible with high performance — ideal for Oracle workloads.

---

### Q15. Retain mainframe, integrate with GCP
**Scenario:** Company has a 40-year-old COBOL mainframe that can't move. They want to modernize front-end on GCP and integrate.

**Options:**
- A) Rewrite mainframe on GCE (impossible in scope)
- B) Retain mainframe; use Mainframe Connector + Pub/Sub / Dataflow for integration; build new front-end on Cloud Run
- C) Migrate mainframe to GKE
- D) Bare Metal Solution

✅ **Answer: B** — **Retain** mainframe (6 R's); use integration layer. Mainframe Connector is Google's tool for mainframe → GCP data flow.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **AlloyDB** matured | New answer for high-perf PostgreSQL, HTAP |
| **Hyperdisk** replaces PD as premium | New answer for demanding block storage |
| **Cloud Run Jobs + GPU support + Sidecars** | Cloud Run replaces many GKE scenarios |
| **GKE Autopilot** preferred | Less ops; default GKE answer |
| **Infrastructure Manager** | Replaces Deployment Manager (deprecated) |
| **Network Connectivity Center** with VPC spokes | Replaces complex VPC peering mesh |
| **Private Service Connect (PSC)** | Replaces old "private services access" patterns |
| **Spanner Graph / Spanner for PostgreSQL** | Expanded answers |
| **BigQuery subscription for Pub/Sub** | Direct BQ writes, no Dataflow |
| **Cross-Cloud Interconnect** | Direct link to AWS/Azure/OCI |
| **Vertex AI replaces AI Platform** | ML workflow questions |
| **Duet AI / Gemini Cloud Assist** | Ops automation |
| **Cloud WAN** | Enterprise global private backbone |
| **Parallelstore** | HPC/AI-training file system |
| **Spot VMs** replace Preemptible VMs | No 24hr cap, deeper discounts |

---

## 🎯 Final Domain 1 Checklist

Before exam day, you should be able to:

- [ ] Match any scenario signal to a compute service in <10 seconds
- [ ] Match any data signal to a database in <10 seconds
- [ ] Know the 6 R's and when each applies
- [ ] Draw VPC peering vs Shared VPC vs NCC topology from memory
- [ ] Recite availability math (99.99% = 52 min/year)
- [ ] Know all 4 GCS storage classes + min durations
- [ ] Know HA VPN 99.99% requirements (2 tunnels + BGP)
- [ ] Know Dedicated Interconnect 99.99% requirements (redundant topology)
- [ ] Know Cloud Run vs GKE Autopilot vs GKE Standard differences
- [ ] Know Spanner vs AlloyDB vs Cloud SQL positioning
- [ ] Know Bigtable key design avoids hotspots
- [ ] Know when to pick Assured Workloads

> **You've got this. Domain 1 is a big one — master it and you've knocked out nearly a quarter of the exam.** 🚀
