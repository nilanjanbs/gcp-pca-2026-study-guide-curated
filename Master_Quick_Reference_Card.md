# 🗺️ Master Quick-Reference Card — GCP PCA 2026

> **The exam-night cheat sheet. 120+ scenario → solution rows, organized by domain, with traps called out and wrong answers flagged.**
>
> **How to use:** Read the scenario signal column → match it to your answer → check the trap column to make sure you're not being fooled.

---

## 🔑 Legend

| Symbol | Meaning |
|---|---|
| ✅ | Correct pattern / right answer |
| ❌ | Anti-pattern / wrong answer |
| ⚠️ | Trap — common exam trick |
| 🔥 | High-frequency exam topic |
| 🆕 | New in 2025–2026 exam version |
| 💀 | Deprecated — never pick |

---

## 📋 Table of Contents
- [Domain 1 — Architecture Design & Service Selection](#-domain-1--architecture-design--service-selection)
- [Domain 2 — Infrastructure Provisioning & Networking](#-domain-2--infrastructure-provisioning--networking)
- [Domain 3 — Security & Compliance](#-domain-3--security--compliance)
- [Domain 4 — Cost Optimization & Performance](#-domain-4--cost-optimization--performance)
- [Domain 5 — CI/CD & Implementation](#-domain-5--cicd--implementation)
- [Domain 6 — Reliability, HA & DR](#-domain-6--reliability-ha--dr)
- [Cross-Domain: "Minimum Ops" Ladder](#-cross-domain-minimum-ops-ladder)
- [Cross-Domain: Deprecated Services Trap Table](#-cross-domain-deprecated-services-trap-table)
- [Availability Math Cheat Sheet](#-availability-math-cheat-sheet)
- [The 30-Second Decision Trees](#-the-30-second-decision-trees)

---

## 🏗️ Domain 1 — Architecture Design & Service Selection

### Compute Selection

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Stateless HTTP containers, scale-to-zero, minimal infra management" | **Cloud Run** | GKE Standard | GKE requires node management + has a minimum node cost | 🔥 |
| "Event-driven function triggered by Pub/Sub / GCS / HTTP, < 9 min" | **Cloud Run Functions (gen 2)** | Cloud Functions gen 1 | Gen 1 is legacy; gen 2 = Cloud Run under the hood | 🔥 |
| "Long-running containers, no idle traffic, cost-sensitive" | **Cloud Run (min-instances=0)** | App Engine Flexible | App Engine Flex has a minimum 1 instance always running | 🔥 |
| "Need full K8s features: StatefulSets, DaemonSets, CRDs, operators" | **GKE Autopilot** (or Standard if custom nodes needed) | Cloud Run | Cloud Run has no StatefulSets or DaemonSets | 🔥 |
| "GKE with minimum node management, per-pod billing" | **GKE Autopilot** | GKE Standard | Standard requires node pool management; Autopilot = fully managed | 🆕🔥 |
| "Custom node pools, specific GPU types, advanced K8s tuning" | **GKE Standard** | GKE Autopilot | Autopilot limits customization; Standard gives full control | |
| "Multi-cluster K8s across GCP + on-prem + AWS" | **GKE Enterprise (Anthos)** | GKE Standard alone | Standard only manages GCP; Enterprise = fleet management across environments | 🆕 |
| "Lift-and-shift 200 VMs from VMware, 60-day deadline, no code changes" | **GCE + Migrate to Virtual Machines** | GKE / Cloud Run | Re-platforming or refactoring takes months; Rehost = fastest to cloud | 🔥 |
| "Auto-scaling fleet of VMs with auto-healing and rolling updates" | **Regional MIG (Managed Instance Group)** | Zonal MIG | Zonal MIG = single zone failure = downtime; always Regional for production | 🔥 |
| "WebSocket-based real-time app, persistent connections" | **App Engine Flexible** or **GKE** or **Cloud Run (always-on CPU)** | App Engine Standard | Standard doesn't support WebSockets; Flex/GKE/Run do | |
| "Windows Server app, per-core licensing compliance, dedicated hardware" | **GCE Sole-tenant Nodes** | Standard GCE | Sole-tenant = dedicated hardware = satisfies per-core Windows license requirements | |
| "SAP HANA, Oracle RAC, mainframe-level hardware requirements" | **Bare Metal Solution** | GCE | BMS = physical servers on GCP network; required for Oracle RAC / SAP HANA | |
| "HPC batch jobs (ML training, genomics) — scale to 0, GPU support" | **Cloud Batch** or **GKE Spot node pools** | Cloud Run | Batch = purpose-built HPC; handles GPU/TPU, long jobs, preemption | |
| "Python/Java/Go/Node web app, no containers, scale-to-zero PaaS" | **App Engine Standard** | App Engine Flexible | Flex has no scale-to-zero; Standard is the lean PaaS option | 🔥 |

### Database & Storage Selection

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Global SQL, strong consistency, financial transactions, 99.999% SLA" | **Spanner (multi-region)** | Cloud SQL with cross-region replicas | Cloud SQL replicas are async → data loss on failover; not globally consistent | 🔥 |
| "High-performance PostgreSQL, HTAP (transactional + analytical), 4× faster" | **AlloyDB** | Cloud SQL PostgreSQL | Cloud SQL is slower for analytics; AlloyDB has a columnar engine | 🆕🔥 |
| "Existing MySQL/PostgreSQL app, regional, managed, minimal re-arch" | **Cloud SQL** | Spanner | Spanner is overkill + expensive for single-region relational needs | 🔥 |
| "Mobile/web app, offline persistence, real-time listeners, doc model" | **Firestore Native mode** | Firestore Datastore mode | Datastore mode = no offline sync, no real-time listeners | 🔥 |
| "Legacy App Engine app using Datastore API, can't change code" | **Firestore in Datastore mode** | Firestore Native | Native uses a different API; legacy App Engine code only works with Datastore mode | |
| "IoT telemetry, 2M devices, time-series, >1B rows, sub-10ms reads" | **Bigtable** | BigQuery | BigQuery is OLAP (high latency writes); Bigtable = low-latency wide-column | 🔥 |
| "Sub-10ms row key lookups + need time-series aggregations via SQL" | **Bigtable (hot reads) + BigQuery (analytics via external table)** | Bigtable alone | Bigtable has no SQL; BigQuery provides the analytical layer | |
| "Petabyte analytics, serverless SQL, BI dashboards, no infra" | **BigQuery** | Bigtable | Bigtable is for OLTP-like patterns; BQ is the OLAP warehouse | 🔥 |
| "In-memory caching, sessions, leaderboards, sub-ms reads" | **Memorystore for Redis (Standard tier)** | Cloud SQL | Cloud SQL is disk-based; Memorystore = RAM = sub-ms | 🔥 |
| "Object storage, images, videos, backups, data lake files" | **Cloud Storage (appropriate class)** | Persistent Disk | PD = block storage for VMs; GCS = object storage accessible from anywhere | 🔥 |
| "Compliance archive: access once/year, retain 10 years, immutable" | **GCS Archive + Locked Retention Policy** | GCS Coldline | Coldline = quarterly; Archive = yearly+. Lock policy = immutable WORM | 🔥 |
| "NFS file share for GCE VMs and GKE pods" | **Filestore** | Cloud Storage | GCS has no POSIX/NFS; Filestore = managed NFS | |
| "Parallel FS for AI training / HPC workloads (Lustre-based)" | **Parallelstore** | Filestore | Filestore max throughput too low for ML training; Parallelstore = high-perf parallel FS | 🆕 |
| "Block storage for GCE VM, premium IOPS, configurable throughput" | **Hyperdisk (Balanced/Extreme/ML)** | Persistent Disk SSD | Hyperdisk = newer, configurable, higher perf ceiling | 🆕🔥 |
| "Multi-attach read-only block for multiple AI training VMs simultaneously" | **Hyperdisk ML** | Persistent Disk | PD doesn't support multi-attach for read; Hyperdisk ML is purpose-built | 🆕 |

### Data & Analytics Architecture

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Existing Hadoop / Spark workloads, migrate with minimal code changes" | **Dataproc** | Dataflow | Dataflow = Apache Beam; Dataproc = Hadoop/Spark — completely different engines | 🔥 |
| "Greenfield streaming + batch ETL pipeline, Apache Beam preferred" | **Dataflow** | Dataproc | Dataproc runs Spark/Hadoop; Dataflow is managed Beam for new pipelines | 🔥 |
| "Stream Pub/Sub messages directly into BigQuery, minimal pipeline code" | **Pub/Sub BigQuery subscription** | Dataflow | BQ subscription (2022+) writes directly to BQ without Dataflow overhead | 🆕🔥 |
| "Async message queue to decouple microservices, global fan-out" | **Pub/Sub** | Cloud Tasks | Pub/Sub = at-least-once delivery, fan-out, push/pull; Tasks = exactly-once, targeted | 🔥 |
| "Orchestrate complex multi-step DAG workflows (cron, retries, dependencies)" | **Cloud Composer (Apache Airflow)** | Cloud Scheduler + Cloud Functions | Composer handles DAGs natively; Functions + Scheduler = too much custom code | |
| "Simple sequential step workflow (API calls, conditional logic)" | **Workflows** | Cloud Composer | Composer is overkill for simple steps; Workflows = serverless, cheaper, faster | 🆕 |
| "Scheduled trigger for periodic jobs (cron)" | **Cloud Scheduler** | Cloud Composer | Composer is for DAGs; Scheduler = managed cron, simple triggers | |
| "CDC from PostgreSQL/MySQL database into BigQuery" | **Datastream** | Dataflow custom job | Datastream is purpose-built CDC; custom Dataflow requires maintenance | 🆕 |
| "Data quality profiling + catalog for all data assets (BQ, GCS, Pub/Sub)" | **Dataplex** | Data Catalog (standalone) | Data Catalog merged into Dataplex Catalog 2023; Dataplex = unified data management | 🆕🔥 |

### Network Topology Design

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Central network team, 10 app teams each in their own project, share VPC" | **Shared VPC** | VPC Peering | VPC Peering is between 2 VPCs only; Shared VPC = 1 host + many service projects | 🔥 |
| "5 VPCs need any-to-any transitive connectivity including on-prem" | **Network Connectivity Center (NCC)** | VPC Peering mesh | VPC Peering is non-transitive; NCC hub-and-spoke handles transitive routing natively | 🆕🔥 |
| "2 VPCs in different projects need private connectivity, same org" | **VPC Peering** | Shared VPC | Shared VPC requires same org and host/service project setup; Peering = simpler for 2 VPCs | 🔥 |
| "Private VMs need outbound internet access (e.g., OS updates), no external IP" | **Cloud NAT** | Bastion host | Bastion is for inbound SSH; NAT is for outbound internet from private VMs | 🔥 |
| "Private VMs (no external IP) must reach Cloud Storage / BigQuery APIs" | **Private Google Access (enabled on subnet)** | Cloud NAT | Cloud NAT routes to internet; PGA routes Google API traffic internally | 🔥 |
| "Consume Cloud SQL / Memorystore from VPC with private IP only" | **Private Service Connect (PSC)** | Private Google Access | PGA is for Google APIs only; PSC is for managed services with internal endpoint | 🆕🔥 |
| "App in VPC needs to privately consume 3rd-party SaaS (e.g., Snowflake)" | **Private Service Connect (PSC)** | VPN | VPN adds latency + complexity; PSC creates private internal endpoint | 🆕 |

---

## ⚙️ Domain 2 — Infrastructure Provisioning & Networking

### IaC & Provisioning

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "IaC for GCP infrastructure, multi-cloud, industry standard" | **Terraform** | Deployment Manager | DM deprecated 2023; Terraform = industry standard, multi-cloud | 💀🔥 |
| "Managed Terraform runner, GitOps, state in GCP, no pipeline setup" | **Infrastructure Manager** | Deployment Manager | IM = managed Terraform runner; DM is deprecated | 🆕🔥 |
| "Manage GCP resources via Kubernetes YAML (CRDs), GKE-centric org" | **Config Connector** | Terraform | Config Connector is K8s-native; Terraform is broader; for GKE orgs CC is natural | 🆕 |
| "Enforce org-wide Kubernetes policies (OPA Gatekeeper)" | **Policy Controller (GKE Enterprise)** | VPC-SC | VPC-SC is for API-level exfiltration; Policy Controller enforces K8s admission | |
| "Prevent any service account key creation across entire org" | **Org Policy: `iam.disableServiceAccountKeyCreation`** | IAM role removal | IAM is additive; only Org Policy enforces a blanket prevent | 🔥 |
| "Restrict all VMs to only deploy in specific regions, org-wide" | **Org Policy: `gcp.resourceLocations`** | IAM conditions | IAM conditions = complex per-resource; Org Policy = single enforcement point | 🔥 |
| "Enforce CMEK for all Cloud SQL and GCS in production folder" | **Org Policy: `gcp.restrictCmekCryptoKeyProjects` at folder** | Manual CMEK setup per resource | Manual = error-prone; Org Policy = automatic guardrail | 🔥 |

### Load Balancing

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Global HTTPS traffic, CDN, URL routing, WAF, single anycast IP" | **Global External Application LB** | Regional Application LB | Regional LB = 1 region only; Global = anycast across all regions | 🔥 |
| "Single-region HTTPS app, cost optimization, no global routing needed" | **Regional External Application LB** | Global Application LB | Global LB costs more; if no global users, Regional is cheaper and sufficient | |
| "TCP/UDP, preserve source IP, ultra-low latency, no TLS termination at LB" | **External Passthrough Network LB** | Global Application LB | App LB terminates TLS = rewrites source IP; Passthrough preserves source IP | 🔥 |
| "Internal HTTP(S) traffic, service-to-service in VPC" | **Internal Application LB (Regional)** | Global Application LB | Internal = private; Global App LB is external only | 🔥 |
| "Internal traffic across multiple regions inside GCP" | **Cross-region Internal Application LB** | Multiple Regional Internal LBs | Cross-region ILB handles multi-region internal routing natively | 🆕 |
| "SSL/TLS termination + TCP proxy, global, not HTTP(S)" | **Global External Proxy Network LB** | Global Application LB | App LB only does HTTP; Proxy Network LB handles TCP/SSL proxy | |

### Autoscaling & Hybrid Connectivity

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "MIG scales based on Pub/Sub queue backlog, not CPU" | **MIG autoscaler with Cloud Monitoring custom metric (Pub/Sub backlog)** | CPU autoscaling | CPU won't reflect queue depth; need custom metric on subscription backlog | 🔥 |
| "GKE scales pods based on custom queue depth metric" | **HPA with custom/external metric (Pub/Sub, Stackdriver)** | VPA on CPU | VPA changes resource requests; HPA changes replica count — queue = HPA territory | 🔥 |
| "GKE: HPA on CPU + VPA on CPU at same time → pods restart constantly" | **Remove VPA on CPU; keep HPA on CPU OR use VPA only on memory** | Keep both | HPA + VPA on same metric conflict; VPA resizes pods → HPA can't stabilize count | 🔥 |
| "Hybrid: < 3 Gbps, fast to set up, 99.99% SLA required" | **HA VPN (2 tunnels, 2 interfaces, BGP)** | Classic VPN | Classic VPN = 99.9%; HA VPN with 2 tunnels + BGP = 99.99% | 🔥 |
| "Hybrid: 10–100 Gbps, physical colo, lowest latency, private" | **Dedicated Interconnect** | HA VPN | HA VPN caps at ~3 Gbps per tunnel; Dedicated IC = 10/100 Gbps with SLA | 🔥 |
| "Dedicated Interconnect 99.99% SLA requirement" | **4 × 10G connections across 2 metro locations (edge availability domains)** | 1 or 2 Dedicated IC connections | 1 connection = 99.9%; Need 4+ across 2 metros for 99.99% | 🔥 |
| "Hybrid: 1–50 Gbps, no colo facility, 99.99% SLA needed" | **Partner Interconnect (redundant topology via 2 providers)** | HA VPN | HA VPN < 3 Gbps; Partner IC supports up to 50 Gbps without colo requirement | |
| "Direct private connectivity GCP ↔ AWS / Azure / OCI, 10+ Gbps" | **Cross-Cloud Interconnect** | VPN between clouds | VPN = internet routing; Cross-Cloud IC = direct physical link between clouds | 🆕🔥 |

---

## 🔒 Domain 3 — Security & Compliance

### IAM & Identity

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "GitHub Actions CI/CD deploys to GCP, no long-lived credentials" | **Workload Identity Federation (OIDC from GitHub)** | SA JSON key stored in GitHub Secrets | JSON key = long-lived, leaked in logs/repos; WIF = short-lived, keyless | 🔥 |
| "GKE pod needs access to Cloud Storage bucket, no keys" | **Workload Identity for GKE (KSA → GSA binding)** | Mounting SA JSON key as K8s secret | K8s secrets are base64; keys compromise entire node if one pod is exploited | 🔥 |
| "On-premises workload (not GCE) needs to call GCP APIs, no keys" | **Workload Identity Federation (self-signed JWT or OIDC)** | Downloaded SA key | Downloaded key = long-lived risk; WIF = short-lived, revocable | 🔥 |
| "Grant temporary access to a service account for 1 hour" | **SA Impersonation (`iam.serviceAccountTokenCreator` role)** | Download SA key, delete after | Downloaded key exists until explicitly deleted; impersonation is genuinely ephemeral | 🔥 |
| "IAM policy must block EVERYONE (including Org Admins) from deleting a bucket" | **IAM Deny Policy** | IAM Allow Policy with no delete | Allow policies are additive; only Deny Policy explicitly blocks even admin | 🆕🔥 |
| "Users can only access GCP from corp IPs during business hours" | **IAM Conditions (IP range + time-based conditions)** | Firewall rules | Firewall rules are network-level; IAM Conditions control *API access* | 🔥 |
| "Grant Dev team access to only `dev-*` prefix buckets, not prod" | **IAM Conditions (`resource.name.startsWith(...)`)** | Separate projects | Separate projects work but conditions allow finer control within a project | |
| "Minimum ops IAM for prod — avoid Owner/Editor basic roles" | **Predefined roles** (or **Custom roles** for least-priv) | Owner / Editor roles | Basic roles are project-wide; predefined roles scope to specific services | 🔥 |

### Encryption

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Compliance requires customer-controlled encryption key rotation and audit" | **CMEK (Cloud KMS)** | Google-managed default encryption | Default = Google rotates; CMEK = you control rotation and audit key use | 🔥 |
| "FIPS 140-2 Level 3 hardware-backed keys, still managed in GCP" | **Cloud HSM (CMEK with HSM backend)** | Cloud KMS Software key | Software KMS = not hardware-validated; Cloud HSM = FIPS 140-2 L3 compliant | 🔥 |
| "Key must NEVER enter Google infrastructure — true HYOK" | **External Key Manager (EKM)** | CMEK with Cloud KMS | CMEK key lives inside GCP KMS infra; EKM key lives in YOUR external HSM | 🔥 |
| "Encrypt data WHILE being processed / in RAM (in-use encryption)" | **Confidential VMs / Confidential GKE Nodes** | CMEK | CMEK = at-rest only; Confidential Computing = in-use memory encryption via AMD SEV | 🆕🔥 |
| "Encrypt specific GCS objects/GCE disks with a raw customer-provided key" | **CSEK (Customer-Supplied Encryption Key)** | CMEK | CSEK = you provide raw key per API call; limited scope; mostly legacy | |

### Network Security & Compliance

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Prevent authorized users with stolen creds from exfiltrating BQ data to personal GCS" | **VPC Service Controls (perimeter around BQ + GCS)** | IAM tightening | IAM controls WHO; VPC-SC controls WHERE data can flow — stops exfil even with valid creds | 🔥 |
| "Protect web app from SQLi, XSS, L7 DDoS, geo-based blocking" | **Cloud Armor (Managed Protection Plus with OWASP rules)** | VPC firewall rules | VPC Firewall = L3/L4 only; Cloud Armor = L7 WAF with OWASP preconfigured rules | 🔥 |
| "Replace VPN for employee access to internal web apps and SSH to VMs" | **Identity-Aware Proxy (IAP)** | Larger VPN | VPN = network-level; IAP = identity + context-aware, zero-trust, no network tunnel | 🔥 |
| "HIPAA workload: PHI data, need compliance guardrails with minimal manual setup" | **Assured Workloads (HIPAA)** | Manually configure CMEK + VPC-SC | Manual = error-prone gaps; Assured Workloads pre-configures all HIPAA controls | 🔥 |
| "FedRAMP High / DoD IL4 / IL5 — US federal government workload" | **Assured Workloads (FedRAMP High / IL4 / IL5)** | Standard GCP deployment | Standard GCP doesn't enforce government-specific personnel/access controls | 🔥 |
| "EU data sovereignty — data must never leave EU, EU staff only" | **Assured Workloads (EU Sovereign) or EU Sovereign Cloud (T-Systems/Thales)** | `europe-west1` region alone | Just picking EU region doesn't restrict Google staff to EU or enforce sovereignty policies | 🆕🔥 |
| "Know when Google SREs access your data for support" | **Access Transparency** | Cloud Audit Logs | Admin Activity logs track YOUR actions; Access Transparency logs GOOGLE's access to your data | 🔥 |
| "Require YOUR approval before any Google employee accesses your data" | **Access Approval** | Access Transparency | Transparency = passive logging; Approval = active gate requiring you to approve access | 🔥 |
| "Who READ the `patients` table in BigQuery last month?" | **Enable Data Access audit logs (DATA_READ) for BigQuery** | Admin Activity audit logs | Admin Activity = config changes only; Data Access (OFF by default!) = data reads/writes | 🔥 |
| "Store API keys and DB passwords, rotate quarterly, audit every access" | **Secret Manager** | Environment variables / Git | Env vars = visible in container inspect; Git = catastrophic if leaked; SM = purpose-built | 🔥 |
| "Block access to specific columns in BigQuery (e.g., SSN) for analysts" | **Policy Tags (Dataplex Catalog column-level security)** | Row-level security | Row-level = filter rows; Policy Tags = restrict specific COLUMN access | 🔥 |

---

## 💰 Domain 4 — Cost Optimization & Performance

### Compute Cost

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Batch job, fault-tolerant, can restart from checkpoint, run overnight" | **Spot VMs** | On-demand GCE | On-demand = paying full price for interruptible workload; Spot = 60-91% off | 🔥 |
| "Stable 24/7 web tier, will run for 3+ years, predictable" | **3-year Resource-based CUD** | Spot VMs | Spot = can be preempted = wrong for stable 24/7; 3-yr CUD = deepest stable discount | 🔥 |
| "Stable workload but might change VM family/size as tech evolves" | **Flexible (Spend-based) CUD** | Resource-based CUD | Resource CUD = locked to family/size; Flexible CUD spans families/sizes automatically | 🆕🔥 |
| "Dev/test VMs idle 16 hrs/day and all weekend" | **Instance schedule (auto stop/start) + Spot VMs** | CUDs | CUDs = pay for commitment even when off; Schedule = only pay while running | 🔥 |
| "Unpredictable traffic — 95% idle, 5% heavy spike" | **Cloud Run (scale-to-zero)** | GCE MIG with CUDs | CUD + always-on VMs = paying for idle; Cloud Run = pay only during requests | 🔥 |
| "Discount applied automatically without any commitment or config" | **Sustained Use Discount (SUD)** | CUD | CUD requires a purchase; SUD is fully automatic after 25% of month usage | 🔥 |

### BigQuery Optimization

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "BQ queries always filter by `WHERE event_date = '...'`, full table scan" | **Partition table on `event_date`** | Add more slots | More slots = faster scan; partitioning = only scan relevant partition = cheaper | 🔥 |
| "Queries filter by `date` AND `customer_id` AND `region`" | **Partition on date + Cluster by `customer_id`, `region`** | Partition alone | Clustering skips blocks within a partition; both together = maximum cost reduction | 🔥 |
| "100 analysts run the SAME hourly revenue aggregation query" | **Materialized Views** (auto-refreshed aggregate) | Scheduled query to separate table | Materialized views auto-refresh + query planner rewrites to use them automatically | 🔥 |
| "Looker dashboards hitting BQ, 200 users, need sub-second results" | **BI Engine + Materialized Views** | Buy more slots | BI Engine = in-memory acceleration for dashboards; MVs = precomputed aggregates | 🔥 |
| "BQ bill $60K/month, predictable heavy queries all month long" | **Capacity pricing — BQ Editions (Enterprise) with slot reservations** | On-demand pricing | On-demand billed per TB scanned; at $60K+ capacity is usually cheaper | 🔥 |
| "Reduce BQ scanned bytes: analyst runs `SELECT *` on 10 TB table" | **Column projection: `SELECT col1, col2, col3 FROM ...`** | Caching | Query cache = free repeat query; column selection = reduce bytes scanned = lower bill | 🔥 |
| "BQ table hasn't been modified in 100 days, still billed at active storage rate" | **Long-term storage pricing auto-applies after 90 days — no action needed** | Move to GCS | BQ auto-applies 50% discount on inactive tables; no action required | |
| "Slow BQ queries on 500 GB table with no WHERE clause optimization possible" | **Require partition filter (set `require_partition_filter=TRUE`)** | More slots | Forcing partition filter prevents accidental full-table scans from analysts | 🔥 |

### Storage & Network Cost

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Logs accessed daily for 0–30 days, then maybe once monthly after" | **Standard → Nearline after 30 days (lifecycle rule)** | Keep all in Standard | Standard = expensive for cold data; lifecycle = automatic cost reduction | 🔥 |
| "Compliance archive: medical records, accessed < once/year, 20-year retention" | **GCS Archive + locked retention policy** | GCS Nearline | Nearline = monthly; Archive = yearly+ at $0.001/GB. Lock policy = immutable | 🔥 |
| "GCS object lifecycle: auto-delete 7-year-old objects" | **Lifecycle rule: `condition.age = 2555 (days)`, `action.type = Delete`** | Manual deletion script | Manual = ops overhead; lifecycle rules = set-and-forget automation | 🔥 |
| "High-volume internet egress from GCE, latency not critical, reduce cost" | **Standard Network Tier (instead of Premium)** | Premium Tier | Premium routes via Google backbone = lower latency but higher cost; Standard = cheaper | |
| "Reduce CDN/GCS egress cost for popular static content globally" | **Cloud CDN in front of Global LB + GCS backend** | Serve directly from GCS | CDN caches at edge POPs = fewer origin fetches = lower egress cost + lower latency | 🔥 |

### FinOps & Performance

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Attribute monthly cloud costs to 5 different engineering teams for chargeback" | **Labels on all resources + billing export to BigQuery + Looker dashboard** | Separate billing accounts per team | Separate accounts = org fragmentation; labels = lightweight attribution on shared account | 🔥 |
| "Alert engineering Slack channel when spending exceeds 90% of monthly budget" | **Cloud Billing budget + Pub/Sub notification → Cloud Functions → Slack** | Cloud Monitoring metric alert | Billing budgets + Pub/Sub is the native pattern for programmatic spend alerts | 🔥 |
| "Find and delete idle VMs, oversized VMs, unused persistent disks automatically" | **Active Assist / Recommender (Idle VM + Rightsizing recommenders)** | Manual review | Recommender uses ML to identify waste across the fleet; faster + more complete | 🆕🔥 |
| "Cloud Run API response P99 latency spikes on first requests after idle" | **Set `min-instances = 1+` (minimum warm instances)** | Increase max instances | Max instances doesn't help cold starts; min instances = always-warm containers | 🔥 |
| "DB read-heavy app, queries same dataset repeatedly, reduce DB load" | **Memorystore (Redis/Valkey) as cache layer** | Add read replicas only | Replicas help but still hit disk; Memorystore = RAM cache = much faster, zero DB load | 🔥 |

---

## 🚀 Domain 5 — CI/CD & Implementation

### CI/CD Toolchain

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Managed CI pipeline on GCP: build, test, push images" | **Cloud Build** | Jenkins on GCE | Jenkins = self-managed, ops overhead; Cloud Build = serverless CI, zero infra | 🔥 |
| "Cloud Build needs to access private Cloud SQL (private IP, no external IP)" | **Cloud Build Private Pool (runs inside your VPC)** | Default Cloud Build pool | Default pool = public internet; Private Pool = runs inside your VPC | 🔥 |
| "Store Docker images, npm packages, Maven artifacts in one managed registry" | **Artifact Registry** | Container Registry (GCR) | Container Registry is DEPRECATED; Artifact Registry = multi-format successor | 💀🔥 |
| "Continuously scan container images for new CVEs after initial push" | **Artifact Analysis (continuous scanning)** | One-time `trivy` scan in CI | One-time scan misses new CVEs; Artifact Analysis rescans when new vulnerabilities published | 🔥 |
| "Managed CD pipeline: deploy to GKE → staging, then prod, with canary phases" | **Cloud Deploy** | Custom Cloud Build deploy steps | Cloud Build = CI; Cloud Deploy = managed CD with stages, approvals, canary built-in | 🔥🆕 |
| "Enforce: only images built by our Cloud Build pipeline can deploy to prod GKE" | **Binary Authorization + Cloud Build attestations** | IAM on GKE | IAM = who can deploy; BinAuth = validates WHAT image can deploy | 🔥 |
| "SLSA Level 3: non-falsifiable, signed build provenance for all images" | **Cloud Build (auto-generates signed SLSA provenance) + BinAuth** | Manual image signing | Cloud Build automatically creates signed SLSA 3 provenance; BinAuth enforces it | 🆕🔥 |

### Deployment Strategies

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Deploy new version with 5% traffic initially, monitor errors, ramp up" | **Canary deployment** (Cloud Deploy or Cloud Run traffic split) | Blue/Green | Blue/Green switches 100% at once; canary = gradual with monitoring | 🔥 |
| "Zero downtime deploy, instant rollback if needed, stateless HTTP containers" | **Blue/Green deployment** | Rolling update | Rolling update takes time to roll back; Blue/Green = instant switch back | 🔥 |
| "Zero downtime VM fleet update, replace old VMs gradually, cost-sensitive" | **MIG rolling update (maxSurge=1, maxUnavailable=0)** | Recreate all at once | Recreate = downtime; Rolling = no downtime, no 2× infra cost | 🔥 |
| "Deploy new Cloud Run service version, route 10% traffic, test, then 100%" | **Cloud Run revision traffic splitting** | Deploy new Cloud Run service entirely | New service = new URL; revision split = same service, same URL, gradual traffic | 🔥 |
| "Test new backend with real production traffic but users don't see responses" | **Shadow deployment (dark launch)** | Canary | Canary = real users see new responses; Shadow = mirrored traffic, users unaffected | |
| "Most cost-effective zero-downtime strategy (not 2× infra)" | **Canary or Rolling** | Blue/Green | Blue/Green requires 2× infra during cutover window | 🔥 |

### API Management

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Expose APIs to external partners, developer portal, usage plans, monetization" | **Apigee X** | API Gateway | API Gateway has no monetization or developer portal; Apigee = full lifecycle | 🔥 |
| "Simple API key + quota enforcement for Cloud Run / Cloud Functions backend" | **API Gateway** | Apigee | Apigee = massive overhead/cost for simple use case; API Gateway = lightweight serverless proxy | 🔥 |
| "Existing OpenAPI proxy needs migration to modern API management" | **API Gateway** (migrate from) | Cloud Endpoints | Cloud Endpoints = legacy; migrate to API Gateway for new capability | |
| "mTLS + observability + traffic management between microservices in GKE" | **Cloud Service Mesh (Istio-based)** | Apigee | Apigee is north-south (external APIs); Service Mesh is east-west (service-to-service) | 🔥 |

---

## 🛡️ Domain 6 — Reliability, HA & DR

### SRE Fundamentals

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Service has used 43 min of its 99.9% SLO monthly budget, team wants to ship" | **Freeze risky releases; invest in reliability work** | Ship the feature | Error budget = 43.2 min/month at 99.9%; at ~43 min it's exhausted — SRE policy = freeze | 🔥 |
| "Alert fires too often on CPU > 80%, team ignores it (alert fatigue)" | **Switch to burn-rate-based SLO alerts** | Lower CPU threshold | Raw metric thresholds = noisy; burn-rate alerts = fire only when error budget is at risk | 🆕🔥 |
| "Customer SLA promises 99.9% availability. What should internal SLO be?" | **99.95% or higher** (internal SLO must be tighter than SLA) | 99.9% (same as SLA) | Same SLO = SLA = no buffer for engineering; internal SLO must beat the SLA | 🔥 |
| "Measure: % requests served < 200ms. What type of metric is this?" | **SLI (Service Level Indicator)** | SLO | SLI = the measurement itself; SLO = the target number for that measurement | 🔥 |

### High Availability Architecture

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Production web app needs HA across zones in us-central1" | **Regional MIG across 3 zones + Regional LB** | Zonal MIG | Zonal MIG = single point of failure; Regional MIG = distributes across zones | 🔥 |
| "Web app must survive full regional outage, 99.99% SLA" | **Multi-region deployment + Global External Application LB** | Single-region MIG | Single region = one regional outage = downtime; need multi-region for 99.99% | 🔥 |
| "GKE pod keeps restarting in a loop immediately after deploy" | **Check Liveness Probe — likely firing during startup; add Startup Probe** | Increase resource limits | Loop = Liveness killing pod on startup; Startup Probe delays Liveness until app is ready | 🔥 |
| "GKE pod is running but never receives traffic from Service" | **Check Readiness Probe — failing causes pod to be removed from Endpoints** | Check Liveness Probe | Liveness = restart; Readiness = remove from service traffic. Pod running but no traffic = Readiness | 🔥 |
| "GKE app takes 60 sec to initialize (load model), Liveness keeps killing it" | **Add Startup Probe (delays Liveness until startup completes)** | Increase `initialDelaySeconds` on Liveness | initialDelay is fixed; Startup Probe waits adaptively; safer for variable startup times | 🔥 |
| "Cloud Run service needs global HA, survives regional outage" | **Deploy Cloud Run in 2+ regions + Global External Application LB + Serverless NEGs** | Single-region Cloud Run only | Single-region Cloud Run is zone-redundant but NOT region-redundant | 🔥 |
| "MIG autohealing vs LB health check — what's the difference?" | **LB health check = routes traffic away. MIG autohealing = RECREATES the VM** | They are the same | LB health check removes bad VM from LB pool (VM stays alive); Autohealing kills + recreates it | 🔥 |

### Disaster Recovery (DR)

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "RTO = near-zero, RPO = near-zero, global DB, survive regional outage" | **Spanner multi-region configuration** | Cloud SQL cross-region replica | Cloud SQL replica = async (data loss); manual promote (time loss); Spanner = sync + auto | 🔥 |
| "Cloud SQL: need to survive zone failure (same region, auto failover)" | **Cloud SQL regional HA (Primary + Standby in different zone, same region)** | Cross-region read replica | Cross-region replica = DR (manual promote, not auto); HA config = automatic 60s zone failover | 🔥 |
| "Cloud SQL: need to survive full regional outage, RTO = 30 min" | **Cloud SQL cross-region read replica (promote manually on disaster)** | Cloud SQL HA | HA = same region only; cross-region replica survives regional outage (manual promote) | 🔥 |
| "RTO = hours, RPO = hours, cheapest possible DR for GCE VMs" | **Scheduled PD snapshots to another region + documented restore runbook** | Regional Persistent Disk | Regional PD = 2 zones in SAME region, not cross-region; doesn't help with regional disaster | 🔥 |
| "RTO = minutes, always-on infrastructure in DR region, but scaled down" | **Warm Standby** | Backup & Restore | Backup & Restore = hours-scale RTO; Warm Standby = scaled-down infra already running | 🔥 |
| "RTO ≈ 0, RPO ≈ 0, active production traffic in both regions simultaneously" | **Hot Standby / Active-Active** (Global LB + multi-region + Spanner) | Blue/Green | Blue/Green = staging; Hot Standby = real production traffic in both regions | 🔥 |

### Scalability & Data Integrity

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Bigtable row key: `{timestamp}#{device_id}` → hot tablets, throttled writes" | **Redesign row key with hash prefix: `{hash(device_id)}#{timestamp}#{device_id}`** | Add more Bigtable nodes | More nodes help temporarily but don't fix the hotspot root cause in key design | 🔥 |
| "GCS: 5,000+ object writes/sec with sequential names → 429 errors" | **Randomize/hash-prefix object names to distribute across shards** | Increase bucket capacity | GCS rate limit is per-prefix; hashing distributes writes across multiple prefix shards | 🔥 |
| "Dataflow: IoT events arrive 15 min late, aggregations incorrect" | **Event-time windows + watermarks + `allowedLateness(Duration.standardMinutes(15))`** | Process-time windows | Process-time windows are ordered by arrival, not event time → wrong aggregations for late data | 🔥 |
| "GKE scales slowly during sudden traffic spike — pods wait 2 min for nodes" | **Overprovisioning: run low-priority balloon pods to pre-warm node capacity** | Just set higher max node count | Max count = ceiling; balloon pods = pre-warmed capacity available instantly | |

### Observability

| 🔑 Scenario Signal | ✅ Correct Answer | ❌ Common Wrong Answer | ⚠️ Why the Wrong Answer Fails | 🔥 |
|---|---|---|---|---|
| "Monitor external API URL from US, EU, Asia — alert if any region fails" | **Cloud Monitoring Uptime Check (multi-region)** | Custom cron job on GCE | Cron = ops overhead, single location; Uptime Check = managed, multi-region, integrated alerts | 🔥 |
| "Export Cloud Logging to SIEM (Splunk/Chronicle) in real time" | **Log sink → Pub/Sub → SIEM** | Direct SIEM API push | Direct push = custom code + maintenance; Pub/Sub sink = managed, decoupled, reliable | 🔥 |
| "Keep audit logs for 7 years for financial compliance, immutable" | **Log sink → GCS (locked retention policy, 7 years)** | Cloud Logging default retention | Default logging = 30 days for Data Access, 400 days for Admin Activity; needs long-term sink | 🔥 |
| "Analyze logs with SQL: 'How many 5xx errors per service last 30 days?'" | **Log sink → BigQuery + SQL queries** | Cloud Logging log-based metrics | Metrics = counts/distributions; BigQuery = full SQL analysis on structured log data | 🔥 |
| "Find slow code paths in production without sampling or code changes" | **Cloud Profiler** | Cloud Trace | Cloud Trace = request tracing; Cloud Profiler = continuous CPU/heap profiling in production | |
| "Trace a slow request across 5 microservices, find the bottleneck service" | **Cloud Trace** | Cloud Profiler | Cloud Profiler = code-level profiling; Trace = distributed request tracing across services | 🔥 |

---

## 📊 Cross-Domain: "Minimum Ops" Ladder

> When the exam says **"minimize operational overhead"** or **"least management"**, always pick the most managed option that meets the requirements.

| If You Need | Pick (left = less ops → right = more ops) |
|---|---|
| **Stateless HTTP app** | Cloud Run Functions → **Cloud Run** → App Engine Standard → App Engine Flex → GKE Autopilot → GKE Standard → GCE MIG → GCE |
| **Container orchestration** | **Cloud Run** → GKE Autopilot → GKE Standard |
| **Database** | **Firestore** → **Spanner** → **Cloud SQL** → self-managed MySQL on GCE |
| **Analytics** | **BigQuery** → Dataproc Serverless → Dataproc → self-managed Spark |
| **Streaming pipeline** | Pub/Sub BQ subscription → **Dataflow** → Dataproc Streaming → custom |
| **IaC** | **Infrastructure Manager** → Terraform + CI/CD → manual gcloud |
| **CD pipelines** | **Cloud Deploy** → custom Cloud Build CD → manual kubectl |

---

## 💀 Cross-Domain: Deprecated Services Trap Table

> These services appear in exam distractors. NEVER pick them as the correct answer in 2026.

| 💀 Deprecated Service | ✅ Correct 2026 Replacement | Why Deprecated |
|---|---|---|
| **Deployment Manager** | **Terraform + Infrastructure Manager** | Google sunset it; Terraform is the standard |
| **Container Registry (GCR)** | **Artifact Registry** | GCR deprecated 2024; AR supports all formats |
| **Cloud VPN Classic** | **HA VPN** | Classic = 99.9% only; HA VPN = 99.99% with BGP |
| **Preemptible VMs** | **Spot VMs** | Renamed; Spot has no 24-hr limit |
| **AI Platform** | **Vertex AI** | Rebranded and expanded |
| **Cloud DLP API** | **Sensitive Data Protection** | Rebranded 2023 |
| **Data Catalog** | **Dataplex Catalog** | Merged into Dataplex |
| **Flat-rate BigQuery** | **BigQuery Editions (Standard/Enterprise/Plus)** | Flat-rate pricing model discontinued |
| **Anthos** | **GKE Enterprise** | Rebranded 2023 |
| **Cloud Functions gen 1** | **Cloud Run Functions (gen 2)** | Gen 1 = legacy; gen 2 = built on Cloud Run |
| **Cloud Debugger** | **Cloud Trace + Error Reporting** | Deprecated 2023 |
| **Datastore (standalone)** | **Firestore in Datastore mode** | Merged into Firestore; same API |
| **App Engine (for new workloads)** | **Cloud Run** | Still valid for legacy; Cloud Run for new work |
| **Cloud Source Repositories** | **GitHub / GitLab + CSR mirroring** | Not deprecated but rarely the primary choice |

---

## 📐 Availability Math Cheat Sheet

| SLA | Downtime/Year | Downtime/Month | Architecture Needed |
|---|---|---|---|
| **99%** | 3.65 days | 7.2 hrs | Single zone acceptable |
| **99.5%** | 1.83 days | 3.6 hrs | Single zone with recovery plan |
| **99.9%** | 8.76 hrs | 43.2 min | Multi-zone (Regional MIG) |
| **99.95%** | 4.38 hrs | 21.6 min | Regional with HA (Cloud SQL HA) |
| **99.99%** | 52.6 min | 4.3 min | Multi-zone mandatory + Global LB |
| **99.999%** | 5.26 min | 26 sec | Multi-region active-active (Spanner) |

> **Rule:** 99.99% = **cannot** be achieved with a single-zone resource. Always multi-zone minimum.  
> **Rule:** 99.999% = **cannot** be achieved with a single-region database. Always Spanner multi-region.

---

## 🌲 The 30-Second Decision Trees

### "What database do I need?"
```
Is it SQL?
├── Global strong consistency + 99.999%? → SPANNER
├── High-perf PostgreSQL + analytics (HTAP)? → ALLOYDB
└── Regional, existing app, managed? → CLOUD SQL

Is it NoSQL?
├── Time-series / IoT / billions of rows / low-latency? → BIGTABLE
├── Mobile/web, offline sync, real-time listeners, doc model? → FIRESTORE NATIVE
└── Analytics only (petabyte SQL)? → BIGQUERY

Is it cache?
└── Sub-ms reads, session store, leaderboards? → MEMORYSTORE (Redis)

Is it object/file?
├── Blobs, backups, data lake? → CLOUD STORAGE (pick right class)
├── NFS file share? → FILESTORE
└── HPC/AI parallel FS? → PARALLELSTORE
```

### "How do I authenticate this workload to GCP?"
```
Is the workload running on GCP?
├── GCE / GKE node / Cloud Run / Cloud Functions? → ATTACHED SERVICE ACCOUNT (metadata server)
├── GKE pod (need pod-level identity)? → WORKLOAD IDENTITY (GKE)
└── Need to impersonate another SA temporarily? → SA IMPERSONATION (serviceAccountTokenCreator)

Is the workload external to GCP?
├── GitHub Actions / GitLab CI / Bitbucket? → WORKLOAD IDENTITY FEDERATION (OIDC)
├── AWS Lambda / EC2? → WORKLOAD IDENTITY FEDERATION (AWS provider)
├── Azure workload? → WORKLOAD IDENTITY FEDERATION (OIDC/Azure AD)
└── Any OIDC-compatible system? → WORKLOAD IDENTITY FEDERATION

NEVER? → Downloaded SA JSON Key (always wrong in 2026)
```

### "What's my DR strategy?"
```
What are your RTO/RPO requirements?
├── RTO = hours, RPO = hours, tight budget? → BACKUP & RESTORE
├── RTO = 30–60 min, RPO = minutes? → PILOT LIGHT (minimal standby infra)
├── RTO = minutes, RPO = seconds? → WARM STANDBY (scaled-down replica always running)
└── RTO ≈ 0, RPO ≈ 0? → HOT STANDBY / ACTIVE-ACTIVE (Spanner + Global LB)

For databases specifically:
├── Zone failover (same region, auto, ~60s)? → CLOUD SQL HA CONFIG
├── Region failover, SQL, minutes RPO? → CLOUD SQL CROSS-REGION READ REPLICA (manual promote)
└── Region failover, zero RPO, auto? → SPANNER MULTI-REGION
```

### "How do I secure my deployment pipeline?"
```
Step 1: Source → Use branch protection, code review
Step 2: CI → Cloud Build (no downloaded keys → use WIF)
Step 3: Artifacts → Artifact Registry (NOT GCR!)
Step 4: Scanning → Artifact Analysis (continuous CVE scanning)
Step 5: Signing → Cloud Build generates SLSA provenance
Step 6: Gate → Binary Authorization (enforce attestations at deploy)
Step 7: Deploy → Cloud Deploy (canary + approvals + audit log)
Step 8: Runtime → Secrets via Secret Manager (NOT env vars)
```

---

## 🎯 Top 25 "Sounds Right But Isn't" Traps

| ❌ Sounds Right | ✅ Actually Right | 🔑 Why |
|---|---|---|
| Cloud SQL cross-region replicas for zero-RPO global | **Spanner multi-region** | Cloud SQL replicas are async; Spanner is synchronous |
| VPC peering mesh for 5+ VPCs | **Network Connectivity Center** | VPC peering is non-transitive |
| App Engine Flexible for scale-to-zero | **App Engine Standard or Cloud Run** | Flex has minimum 1 instance always running |
| GCR (Container Registry) for images | **Artifact Registry** | GCR is deprecated |
| Deployment Manager for IaC | **Terraform + Infrastructure Manager** | DM is deprecated |
| GKE Standard for minimum ops | **GKE Autopilot** | Standard requires node management |
| Dataflow for existing Spark/Hadoop | **Dataproc** | Dataflow = Apache Beam; completely different engine |
| Dataproc for greenfield streaming | **Dataflow** | Dataproc = Hadoop/Spark; Dataflow = Beam for new pipelines |
| Single Dedicated IC for 99.99% | **4× connections across 2 metros** | 1 IC = 99.9%; need redundant topology for 99.99% |
| HA VPN single tunnel for 99.99% | **2 tunnels across 2 interfaces + BGP** | 1 tunnel = 99.9%; 2 tunnels = 99.99% |
| IAM deny → remove the allow | **IAM Deny Policy** | Removing allow doesn't help if it's inherited; Deny explicitly trumps all |
| CMEK = Google can't see my data | **EKM (External Key Manager)** | CMEK key is inside GCP infra; EKM key never leaves your HSM |
| Data Access logs are on by default | **Enable Data Access logs explicitly** | They are OFF by default for most services |
| BigQuery Flat-Rate pricing model | **BigQuery Editions (Standard/Enterprise/Plus)** | Flat-rate pricing model is discontinued |
| `SELECT *` is fine for BQ exploration | **Use Preview feature (free) or column projection** | SELECT * scans all columns = full cost regardless of columns needed |
| Firestore Datastore mode for new apps | **Firestore Native mode** | Native = real-time, offline; Datastore = legacy App Engine API |
| Cloud Armor on Internal LB | **Cloud Armor on Global/Regional External Application LB** | Cloud Armor only attaches to external-facing LBs |
| Uptime Check lives in Cloud Logging | **Cloud Monitoring** | Uptime checks are a Cloud Monitoring feature, not Logging |
| SA JSON key in K8s Secret is secure | **Workload Identity for GKE** | K8s secrets are base64 only; Workload Identity is truly keyless |
| Add more BQ slots before optimizing queries | **Partition + Cluster first, then slots** | Slots make bad queries faster but cost more; fix queries first |
| Cloud SQL HA = cross-region DR | **Cloud SQL HA = zone HA only** | HA = Primary + Standby in same region; cross-region needs separate read replica |
| Zonal MIG for HA | **Regional MIG (always for production)** | Zonal = single zone = single failure point |
| Cloud NAT for Private Google Access | **Private Google Access (enable on subnet)** | NAT = internet egress; PGA = Google API access without internet |
| Apigee for simple Cloud Run API protection | **API Gateway** | Apigee is full enterprise lifecycle; overkill + expensive for simple auth/quota |
| Bigtable sequential timestamp row keys | **Hash prefix + reverse timestamp** | Sequential keys → all writes to latest tablet = hotspot |

---

> 📌 **Save this card.** Come back to it the night before the exam.  
> 🎯 **If you can answer "why is the wrong answer wrong" for every row**, you're ready.  
> 🚀 **Good luck — you've got this!**
