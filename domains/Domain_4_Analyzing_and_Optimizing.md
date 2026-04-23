# 📊 Domain 4: Analyzing and Optimizing Technical and Business Processes

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (4 of 6)**

**Exam Weight:** ~18%

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [Cost Optimization Fundamentals](#-cost-optimization-fundamentals)
3. [Committed Use Discounts Deep Dive](#-committed-use-discounts-deep-dive)
4. [Spot VMs & Preemption Strategies](#-spot-vms--preemption-strategies)
5. [Rightsizing & Recommender](#-rightsizing--recommender)
6. [Storage Cost Optimization](#-storage-cost-optimization)
7. [BigQuery Cost Optimization (Deep Dive)](#-bigquery-cost-optimization-deep-dive)
8. [Network Cost Optimization](#-network-cost-optimization)
9. [Billing Management & FinOps](#-billing-management--finops)
10. [Performance Optimization](#-performance-optimization)
11. [SDLC & DevOps Process Optimization](#-sdlc--devops-process-optimization)
12. [Organizational Alignment](#-organizational-alignment)
13. [Exam Traps & Tricks](#-exam-traps--tricks)
14. [Decision Frameworks](#-decision-frameworks)
15. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
16. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
17. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

This domain is about **making architectures efficient, affordable, and aligned with business goals**. It's less about "which service" and more about "how to make it cheaper / faster / better".

Common scenario patterns:
- "Reduce cloud bill by 40%"
- "BigQuery bill is exploding — fix it"
- "Team velocity is slow — how to speed up deployments"
- "FinOps needs cost visibility per team"
- "Workload is slow — diagnose and fix"

The exam tests these sub-skills:

1. **Cost optimization** — CUDs, SUDs, Spot VMs, rightsizing, storage tiers
2. **BigQuery tuning** — partitioning, clustering, materialized views, slot reservations
3. **Performance optimization** — caching, CDN, read replicas, connection pooling
4. **FinOps practices** — labels, billing exports, budget alerts, chargeback
5. **Process optimization** — CI/CD, SRE, managed services to reduce ops
6. **Team/org alignment** — mapping tech choices to team maturity
7. **Trade-off analysis** — cost vs. perf vs. ops vs. velocity

---

## 💰 Cost Optimization Fundamentals

### 🧠 The 8 Levers of GCP Cost Optimization

| Lever | Tool / Technique | Typical Savings |
|---|---|---|
| **1. Rightsize** | Recommender, custom machine types | 10–30% |
| **2. Shutdown idle** | Scheduled start/stop, Recommender | 20–40% on dev/test |
| **3. Spot / Preemptible** | Spot VMs | 60–91% |
| **4. Commitments (CUDs)** | 1-yr / 3-yr commits | 20–70% |
| **5. Sustained Use Discounts** | Automatic | Up to 30% |
| **6. Storage tiering** | GCS lifecycle rules | 50–80% on cold data |
| **7. Serverless (scale-to-zero)** | Cloud Run, Functions | Variable (huge for bursty) |
| **8. Network egress** | CDN, choose region, premium vs standard tier | 10–40% on egress-heavy |

### 🔑 The Cost Triage Order

When asked "reduce cost", apply in this order:

1. **Find waste** (idle, oversized, unused resources) via **Recommender**
2. **Apply Spot VMs** for fault-tolerant workloads
3. **Right-size** remaining on-demand VMs
4. **Commit** to CUDs for stable baseline
5. **Tier storage** via lifecycle rules
6. **Optimize serverless** (min instances, concurrency)
7. **Optimize BigQuery** (partition, cluster, reservations)
8. **Optimize network** (CDN, premium vs standard tier)

### 🔑 Free Tier Awareness

GCP Free Tier includes monthly limits on:
- 1 e2-micro GCE (us regions)
- 5 GB GCS
- 1 GB BigQuery queries + 10 GB storage
- 2M Cloud Functions invocations
- 2M Cloud Run requests
- 1 Pub/Sub (10 GB)

**Exam hint:** "Startup, minimize cost for MVP" often implies using free tier.

---

## 💳 Committed Use Discounts Deep Dive

### 🧠 The 3 Types of CUDs (2026)

| Type | Commit On | Flexibility | Best For |
|---|---|---|---|
| **Resource-based CUD** | Specific vCPUs + RAM in a region | Low | Stable, fixed workloads |
| **Spend-based (Flexible) CUD** | Dollar amount per hour | High — spans VM families, sizes, regions | Evolving workloads |
| **Service-specific CUDs** | Cloud SQL, Memorystore, Bigtable, Dataflow, etc. | Product-specific | Stable managed service usage |

### 🔑 CUD Terms & Discounts

| Commitment | VM Discount | Memorystore / Cloud SQL |
|---|---|---|
| **1 year** | ~20–40% | ~25% |
| **3 years** | ~40–70% | ~52% |

### 🔑 When to Use CUD vs Spot vs On-Demand

| Workload | Recommendation |
|---|---|
| Stable 24/7 production baseline | **3-yr CUD** |
| Predictable but may grow/change | **1-yr Flexible CUD** |
| Fault-tolerant batch / ML training | **Spot VMs** |
| Unpredictable spiky | **On-demand** (or serverless) |
| Bursty workload on stable base | **Combine:** CUD for base + on-demand for burst |

### 🔑 Flexible CUD Specifics (2023+ key answer)

- Commit to **$X/hour spend** in a region
- Covers **any machine family** (N1, N2, E2, etc.)
- Covers **any size**
- Great if you don't know exactly which family you'll use
- ⚠️ **Doesn't cover GPU/TPU/sole-tenant** — those need separate resource CUDs

### 🔑 Sustained Use Discounts (SUDs)

- **Automatic** — no commitment needed
- Applied after VM runs >25% of the month
- Maximum ~30% on full-month usage
- Applied **per region per machine family**
- ⚠️ **Don't combine** with CUDs (CUDs trump SUDs on committed usage)

### ⚠️ CUD Exam Traps

- CUDs are **regional + family-bound** (resource-based) — moving workload wastes commitment
- If workload moves → Flexible CUD is safer
- CUDs **don't cover** Spot VMs (already discounted), sole-tenant nodes
- You get **billed for commitment** even if unused — overcommit = wasted money

---

## ⚡ Spot VMs & Preemption Strategies

### 🧠 Spot VM Characteristics

| Aspect | Detail |
|---|---|
| **Discount** | 60–91% off on-demand |
| **Preemption** | Can happen any time, 30-second shutdown notice |
| **Max runtime** | No 24-hour cap (changed from old Preemptible VMs) |
| **Availability** | Subject to capacity; can be refused |
| **Restart** | MIG auto-recreates preempted Spot VMs |

### 🔑 Suitable Workloads

✅ **Good for Spot:**
- Batch processing (Dataflow, Dataproc)
- ML training (checkpoint regularly)
- CI/CD runners
- Stateless web tier (with MIG auto-restart)
- Containerized jobs (GKE Spot node pools)

❌ **Bad for Spot:**
- Stateful databases
- Long-running stateful workflows
- Workloads that can't handle restarts
- Anything where data loss costs > savings

### 🔑 Spot VM Best Practices

- **Checkpointing**: Save progress regularly (esp. ML training)
- **Graceful shutdown**: Handle SIGTERM, save state
- **Mix Spot + On-demand** via MIG for resilience
- **GKE Spot node pools** with taints/tolerations
- **Dataflow**: enable use of Spot by default for batch

### 🔑 Spot in Managed Services

| Service | Spot Support |
|---|---|
| **GCE / MIG** | ✅ Native |
| **GKE** | ✅ Spot node pools |
| **Dataflow** | ✅ Flex-resource scheduling |
| **Dataproc** | ✅ Secondary workers as preemptible |
| **Cloud Run** | ❌ N/A (serverless) |
| **Batch** | ✅ Native |

---

## 📐 Rightsizing & Recommender

### 🧠 Active Assist / Recommender Products

| Recommender | What It Does |
|---|---|
| **VM Rightsizing** | Detects oversized VMs, suggests smaller machine |
| **Idle VM** | Flags VMs with <5% CPU for weeks |
| **Idle Persistent Disk** | Unused disks |
| **Idle IP Address** | Reserved static IPs not attached |
| **Committed Use Discount** | Recommends CUD purchases based on usage |
| **IAM Recommender** | Over-privileged roles → suggests least-privilege |
| **Cloud SQL Idle / Overprovisioned** | Downsize or delete |
| **GKE Cost Optimization** | Pod right-sizing |
| **Firewall Insights** | Unused / shadowed firewall rules |
| **Error Reporting** | Rollup of error patterns |

### 🔑 Custom Machine Types

- Create VMs with **exact vCPU + RAM** (not bound to predefined sizes)
- Useful when predefined doesn't fit (e.g., need 2 vCPU + 16 GB — N2 has 2 vCPU / 8 GB or 4 vCPU / 16 GB)
- Available via **E2 and N2** families

### 🔑 Machine Family Selection

| Family | Strength | Use |
|---|---|---|
| **E2** | Lowest cost | General purpose, dev |
| **N2 / N2D** | Balanced perf | Web, databases |
| **C3 / C3D / C4** | Latest gen, high perf | Performance-sensitive |
| **T2D / T2A** | Scale-out (AMD / Arm) | Stateless web |
| **M-series (M1/M2/M3)** | High memory | SAP HANA, in-memory DBs |
| **C2 / C2D** | Compute-optimized | HPC, gaming |
| **A2 / A3 / G2** | GPU (A100, H100, L4) | ML training/inference |
| **H3** | HPC | Scientific computing |

**2025+ trend:** C4, C4A (Arm), C4D (AMD) are the newest, cheapest-per-performance options.

---

## 💾 Storage Cost Optimization

### 🧠 GCS Lifecycle Strategies

**Classic data aging pattern:**
```
Days 0-30:   Standard  ($0.020/GB)
Days 31-90:  Nearline  ($0.010/GB)
Days 91-365: Coldline  ($0.004/GB)
Days 366+:   Archive   ($0.0012/GB)
Delete at 7 years.
```

**Savings:** Archive is ~94% cheaper than Standard for storage (but retrieval costs more).

### 🔑 Storage Class Tradeoffs

| Class | Storage | Retrieval | Early Delete |
|---|---|---|---|
| **Standard** | $$$ | Free | N/A |
| **Nearline** | $$ | $ | 30-day min |
| **Coldline** | $ | $$ | 90-day min |
| **Archive** | ¢ | $$$ | 365-day min |

⚠️ **Trap:** Don't put hot data in Archive — retrieval + early delete fees destroy savings.

### 🔑 Other GCS Cost Tips

- **Object versioning**: Old versions cost money; use lifecycle to delete noncurrent versions
- **Multi-region** costs more than regional; only use if truly global
- **Autoclass**: Google auto-transitions objects between classes based on access
- **Turbo replication** (dual-region): Premium feature; only if RPO requires

### 🔑 Persistent Disk Optimization

- **Use PD Balanced** (not SSD) unless truly need IOPS
- **Shared disks** for multi-attach (not duplicate)
- **Snapshots** for backups vs keeping old disks
- **Hyperdisk Balanced** auto-tunes IOPS — pay for what you use

### 🔑 Snapshot Optimization

- **Incremental snapshots** — only changed blocks
- **Snapshot schedules** — automate, then delete old
- **Multi-regional snapshots** cost more than regional
- Use **lifecycle policy** on snapshots

---

## 🔍 BigQuery Cost Optimization (Deep Dive)

**BigQuery is the #1 cost-optimization topic on the PCA exam.** Master this.

### 🧠 BigQuery Pricing Models

| Model | How | Best For |
|---|---|---|
| **On-demand** | $6.25 per TB scanned | Ad-hoc, unpredictable usage |
| **Capacity-based (Editions: Standard / Enterprise / Enterprise Plus)** | Pay per slot-hour | Predictable heavy usage |
| **Flat-rate (legacy)** | Deprecated | → Migrated to Editions |

### 🔑 The 7 BigQuery Cost Killers

1. **`SELECT *`** — scans every column. Use **column projection** (select only needed cols)
2. **No partitioning** — full-table scans
3. **No clustering** — full-partition scans
4. **Materialized views not used** for repeat queries
5. **Streaming inserts** in high volume (more expensive than loads)
6. **Excessive external queries** (federated sources scanned every time)
7. **Storage of old data** at active prices (should be long-term)

### 🔑 Partitioning

**What:** Split table by a column (usually date/time).  
**Benefit:** Query only scans relevant partitions.  
**Types:**
- **Time-unit column** (hour/day/month/year)
- **Ingestion time** (`_PARTITIONTIME` pseudo-column)
- **Integer range**

**Best practice:** Partition by **date/time column most commonly filtered in WHERE**.

**Require partition filter:** Force queries to include partition filter (prevents full scans):
```sql
CREATE TABLE dataset.t (...) 
PARTITION BY DATE(event_time)
OPTIONS(require_partition_filter=TRUE);
```

### 🔑 Clustering

**What:** Sort data within partitions by 1–4 columns.  
**Benefit:** BigQuery skips blocks not matching filter.  
**Use with partitioning** for max benefit.

Example:
```sql
CREATE TABLE dataset.orders (...)
PARTITION BY DATE(order_date)
CLUSTER BY customer_id, region;
```

Now `WHERE customer_id = 'ABC' AND region = 'US'` is nearly free.

### 🔑 Materialized Views

- **Precomputed aggregations**, auto-refreshed
- Great for **dashboards** running same queries
- Query re-routing: query plans automatically use MVs when applicable
- **Incremental refresh** for low cost

**Example:**
```sql
CREATE MATERIALIZED VIEW mv_daily_sales AS
SELECT date, SUM(amount) AS daily_total
FROM raw.sales
GROUP BY date;
```

### 🔑 BI Engine

- **In-memory cache** for BQ data
- Sub-second queries for dashboards
- Separate capacity (GB of RAM)
- Integrates with Looker, Tableau, Data Studio

**Exam rule:** "Sub-second dashboards on BQ" → **BI Engine + Materialized Views**.

### 🔑 Storage Classes in BQ

| Class | Price | Trigger |
|---|---|---|
| **Active** | Full price | Modified in last 90 days |
| **Long-term** | 50% off | Not modified in 90 days |
| **Auto-applied** | No action needed | |

### 🔑 Reservations & Editions (capacity-based)

**Editions:**
- **Standard** — basic features
- **Enterprise** — RBAC, BI Engine, materialized views
- **Enterprise Plus** — cross-region DR, CMEK, reservations

**Reservations:**
- Buy **slots** (units of compute)
- **Baseline** (committed) + **Autoscaling** (flex)
- **Assignments**: Projects, folders, orgs use the slots
- **Idle slot sharing**: Unused slots flow to others

**When to switch:**
- Sustained heavy usage → reservations save money
- Ad-hoc → on-demand is fine
- Break-even around ~**$40–60K/month** scanned

### 🔑 Query Optimization Checklist

- [ ] Avoid `SELECT *`
- [ ] Filter on partition column
- [ ] Use clustering columns in filters
- [ ] Use `LIMIT` for exploration (but it still scans — use preview!)
- [ ] Use **preview** feature (free) for sampling
- [ ] Leverage **materialized views** for aggregates
- [ ] Use **approximate aggregations** (APPROX_COUNT_DISTINCT) when exact not needed
- [ ] **Denormalize** for analytics (nested + repeated fields)
- [ ] **Cache query results** (24-hour cache, free)
- [ ] **Flat-rate reservations** for predictable heavy

---

## 🌐 Network Cost Optimization

### 🧠 Egress Pricing Rules

| Traffic | Cost |
|---|---|
| **Ingress to GCP** | Free |
| **Within same zone** | Free |
| **Within same region, different zone** | $ |
| **Same continent, different region** | $$ |
| **Cross-continent** | $$$ |
| **Internet egress** | $$$$ (by destination continent + tier) |

### 🔑 Network Service Tiers

| Tier | Description | Cost |
|---|---|---|
| **Premium (default)** | Google backbone, anycast | Higher |
| **Standard** | Public internet routing to region | Lower |

**Exam rule:** "Reduce internet egress cost, latency OK" → **Standard tier**. "Global low latency" → **Premium**.

### 🔑 CDN to Reduce Egress

- **Cloud CDN** caches at edge → cached hits = **cheaper egress** (cache fill cost only)
- Critical for: images, video, static assets, API responses
- **Cache Hit Ratio** is the KPI

### 🔑 Cross-Region Data Transfer

- **Expensive** between continents
- **Dataflow / Dataproc** — keep compute in same region as data
- **BigQuery** — cross-region queries cost extra
- **Multi-region GCS** → single-region egress free if compute is in same region family

### 🔑 Other Egress Optimizations

- **Private Service Connect** for private API access (no internet egress)
- **VPC peering** traffic (in same region) is **free**
- **Interconnect egress** cheaper than internet egress
- **Shared VPC** to avoid inter-project egress charges

---

## 📊 Billing Management & FinOps

### 🧠 Billing Hierarchy

```
Billing Account (Cloud Billing)
    └── linked to Projects
           └── Resources generate usage → billed to account
```

### 🔑 FinOps Core Practices

1. **Labels on every resource** for attribution
2. **Billing export to BigQuery** for analytics
3. **Budgets with alerts** (50%, 75%, 90%, 100%)
4. **Project-per-team/product** for clean attribution
5. **Recommender reviews** monthly
6. **Chargeback or showback** reports
7. **Anomaly detection** on bills
8. **Tag governance** (enforce label requirements)

### 🔑 Budget Alerts

- Set **threshold triggers** (% of budget or $ amount)
- **Actions**:
  - Email (default)
  - Pub/Sub → Cloud Functions → automated response (e.g., stop VMs)
- **Scope**: Per billing account, per project, per label, per service

### 🔑 Billing Export Destinations

| Destination | Use |
|---|---|
| **BigQuery** | Cost analysis, SQL queries, dashboards |
| **Cloud Storage** (CSV) | Archive / simple reporting |
| **Detailed vs Standard** | Detailed has per-resource usage |

**Exam rule:** "FinOps team needs detailed cost by team/service" → **Billing export to BigQuery + Looker dashboards**.

### 🔑 Labels for Chargeback

**Required labels** (typical corporate):
- `team` = `payments`
- `environment` = `prod`
- `cost-center` = `CC12345`
- `app` = `checkout`

**Use Org Policy** to require labels:
```
constraints/compute.requireComputeResourceLabels
```

### 🔑 Cloud Billing Reports

- **Cost Table**: Per project, service, SKU
- **Cost Breakdown**: Visualizations
- **Reports**: Trends, comparisons
- **Commitment Analysis**: Show CUD utilization

---

## 🚀 Performance Optimization

### 🧠 The Performance Bottleneck Map

| Symptom | Likely Bottleneck | Fix |
|---|---|---|
| "Slow DB queries" | DB CPU/memory, no indexes | Add indexes; use read replicas; cache |
| "Slow BQ queries" | No partition/cluster, SELECT * | Partition + cluster; MV; BI Engine |
| "Slow web app globally" | Single-region origin | Multi-region + Global LB + CDN |
| "Cold start on serverless" | Infrequent invocations | Min instances > 0 |
| "Cross-region latency" | Chatty API calls | Co-locate services |
| "GCE CPU throttling" | E2 shared-core on heavy load | Move to N2 / C3 |
| "Disk bottleneck" | PD Standard for DB | Upgrade to SSD / Hyperdisk |
| "Pub/Sub lag" | Consumer too slow | Scale consumers; use ordering only if needed |

### 🔑 Caching Strategies

| Layer | Tool |
|---|---|
| **CDN (edge)** | Cloud CDN, Media CDN |
| **App layer** | Memorystore (Redis / Valkey / Memcached) |
| **Database** | Read replicas, Cloud SQL query cache |
| **BigQuery** | Automatic query cache (24 hr), BI Engine, MV |
| **DNS** | Cloud DNS TTLs |

### 🔑 Database Performance

**Cloud SQL:**
- **Read replicas** for read-heavy workloads
- **Connection pooling** (use PgBouncer or Cloud SQL Proxy)
- **Right-size** instance
- **SSD** disks for IOPS
- **HA configuration** for failover

**AlloyDB:**
- **4× faster** than Cloud SQL PostgreSQL for transactional
- **100× faster** for analytical queries (columnar engine)
- **Read pools** for horizontal read scaling

**Spanner:**
- **Interleaved tables** for locality
- **Processing unit sizing** (100 PUs = 1/10 of a node)
- **Avoid hotspots** in primary keys

**Bigtable:**
- **Random/hashed row keys** (not sequential!)
- **App profiles** for multi-cluster routing
- **Bigtable Autoscaling** for node count

### 🔑 Load Balancer + CDN Tuning

- Enable **HTTP/2, HTTP/3 (QUIC)**
- Enable **gzip / Brotli** compression (via backend)
- **Cache CDN-friendly assets** (hash in URL for versioning)
- **Signed URLs** for private-but-cacheable
- **Origin shield** to reduce origin load

### 🔑 Cloud Run Performance

- **Min instances > 0** to eliminate cold starts
- **Concurrency tuning** (default 80 per instance)
- **CPU allocation**: "Always" for background work
- **Smaller container images** → faster cold start
- **Use gen 2 execution environment** for faster I/O

### 🔑 Observability Tools for Performance

| Tool | Purpose |
|---|---|
| **Cloud Monitoring** | Metrics, alerts |
| **Cloud Trace** | Distributed tracing |
| **Cloud Profiler** | Continuous CPU/memory profiling |
| **Cloud Logging** | Log aggregation |
| **Error Reporting** | Error aggregation |
| **Application Performance Monitoring (APM)** | Full-stack (via Trace + Profiler + Logs) |

---

## 🔄 SDLC & DevOps Process Optimization

### 🧠 DevOps Maturity Ladder

| Level | Characteristics | GCP Stack |
|---|---|---|
| **Manual** | Clicks in Console | Avoid |
| **Scripted** | Shell scripts + CLI | Minimum for dev |
| **IaC** | Terraform + Git | Baseline |
| **CI/CD** | Automated test + deploy | Cloud Build + Cloud Deploy |
| **Progressive Delivery** | Canary, blue/green, feature flags | Cloud Deploy + traffic splitting |
| **GitOps** | Git as source of truth | Config Connector + Config Sync |
| **Platform Engineering** | Self-service platforms | GKE Enterprise + Backstage |

### 🔑 Velocity Improvements

| Problem | Solution |
|---|---|
| "Deploys take 2 hours" | Parallelize builds; caching; smaller artifacts |
| "Manual testing" | Automated tests in Cloud Build |
| "Manual deploys" | Cloud Deploy pipelines |
| "Rollbacks are scary" | Progressive delivery; canary; blue/green |
| "Infra drift" | Terraform + Infrastructure Manager (GitOps) |
| "Secrets sprawl" | Secret Manager + WIF |

### 🔑 The DORA Metrics

| Metric | What |
|---|---|
| **Deployment frequency** | How often you deploy |
| **Lead time for changes** | Commit → prod time |
| **Change failure rate** | % of deployments causing incident |
| **Time to restore service** | MTTR |

**Exam rule:** Improve DORA metrics = managed CI/CD + automated tests + progressive delivery + observability.

### 🔑 SRE Principles (exam favorite)

- **SLI** (what you measure)
- **SLO** (internal target)
- **SLA** (customer contract)
- **Error Budget** = 100% − SLO
- **Toil reduction** — automate repetitive ops
- **Blameless postmortems**
- **Gradual rollouts** based on error budget

---

## 🏢 Organizational Alignment

### 🧠 Team Topology → Tech Choice

| Team Shape | Best GCP Stack |
|---|---|
| **Small startup, few engineers** | Cloud Run + Firestore + Firebase — minimal ops |
| **Central IT + app teams** | Shared VPC + per-team projects |
| **SRE-led ops** | GKE + SLOs + Cloud Ops suite |
| **Data team separate** | BigQuery + Dataform + Looker |
| **Enterprise with security team** | Assured Workloads + VPC-SC + Hierarchical Firewall |
| **Hybrid / multi-cloud** | GKE Enterprise + Cloud WAN + Cross-Cloud Interconnect |

### 🔑 Conway's Law on GCP

**"Organizations design systems that mirror their communication structure."**

- **Monolithic org** → monolithic architecture
- **Small autonomous teams** → microservices (Cloud Run per team)
- **Platform team** → enables other teams (GKE platform with Config Sync)

### 🔑 Skills Gap Handling

| Gap | Solution |
|---|---|
| No Kubernetes expertise | → Cloud Run, not GKE |
| No DevOps | → Serverless everything |
| No security specialist | → Assured Workloads + defaults |
| Heavy Python/DS team | → BigQuery + Vertex AI Workbench |
| Legacy SysAdmin team | → GCE + Migrate to VMs as stepping stone |

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 15 Traps in Domain 4

1. **"Most cost-effective"** + predictable workload → **CUD** (not Spot if non-fault-tolerant)
2. **"Most cost-effective"** + fault-tolerant batch → **Spot VMs**
3. **"Most cost-effective"** + unpredictable spiky → **Serverless (Cloud Run, Functions)**
4. **SUD doesn't need contract** — CUD does
5. **"BigQuery bill too high"** → partition + cluster + MV (NOT "switch to Cloud SQL")
6. **"Optimize BQ dashboards"** → **BI Engine + MV**, NOT reservations alone
7. **"Cost attribution per team"** → **Labels + billing export to BigQuery**
8. **"Reduce egress"** → **CDN + Standard tier + co-locate compute with data**
9. **"Sub-second dashboards"** → **BI Engine**, NOT "more slots"
10. **Spot VM with stateful DB** → wrong; data loss on preemption
11. **3-yr CUD for workload that might move** → use **Flexible CUD** instead
12. **"Improve deploy velocity"** → **Cloud Build + Cloud Deploy + progressive delivery**
13. **"No ops team"** → push toward **serverless + managed**
14. **"On-demand BQ" + huge monthly scans** → switch to **capacity (slots)**
15. **"Streaming inserts to BQ" for high volume** → consider **loads or Pub/Sub BQ subscription**

### ⚠️ "Sounds Right but Isn't"

| Seems Right | Actually Right | Why |
|---|---|---|
| Bigger VM for speed | Right-size + caching | Vertical scaling often wastes money |
| On-demand BQ always cheaper | Capacity for heavy predictable use | Break-even exists |
| Spot VMs for everything | Only for fault-tolerant | Preemption kills stateful |
| CUD for everything | Only for stable workloads | Wasted commit if workload moves |
| Multi-region GCS for cost | Regional if only used in one region | Multi-region costs more |
| Add more BQ slots | Partition + cluster first | Fix queries before throwing compute |
| CDN for dynamic content | Only static/cacheable | Dynamic won't cache hit |

---

## 🔑 Decision Frameworks

### 🎯 Cost Reduction Framework (step-by-step)

```
Step 1: VISIBILITY
  → Enable billing export to BigQuery
  → Label all resources
  → Dashboards per team/service

Step 2: QUICK WINS
  → Recommender: idle VMs, disks, IPs
  → Delete unused resources
  → Shutdown dev/test off-hours

Step 3: RIGHT-SIZE
  → VM Rightsizing Recommender
  → GKE pod right-sizing
  → Cloud SQL rightsizing

Step 4: DISCOUNT
  → Spot VMs for fault-tolerant
  → 1-yr Flex CUD for stable baseline
  → 3-yr CUD if confident stable

Step 5: ARCHITECT
  → Serverless for bursty
  → Storage lifecycle
  → BigQuery partitioning

Step 6: OPTIMIZE EGRESS
  → CDN for static
  → Standard tier if OK
  → Co-locate compute + data
```

### 🎯 BigQuery Cost Reduction

```
Query too expensive?
  ├── Is there SELECT *? → Column-project
  ├── Is there a WHERE on date? → Partition
  ├── Is there a WHERE on other col? → Cluster
  ├── Same query hourly by 100 users? → Materialized View
  ├── Dashboards slow? → BI Engine
  └── Hot data > 90 days old? → Let it auto-tier to long-term storage

Too many on-demand queries?
  ├── Spend > $30K/month? → Consider slots (Enterprise Edition)
  ├── Unpredictable? → Autoscaling slots
  └── Steady? → Committed baseline slots
```

### 🎯 Performance Framework

```
What's slow?
  ├── Global latency → Cloud CDN + Global LB + multi-region
  ├── DB reads → Read replicas / Memorystore cache
  ├── BQ queries → Partition + cluster + MV + BI Engine
  ├── Cold start → Cloud Run min-instances > 0
  ├── Chatty → Co-locate, batch requests
  └── Under-provisioned → Right-size UP (not down!)
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "RSLBN" (Cost ladder)
**R**ight-size → **S**pot VMs → **L**ifecycle storage → **B**udgets + labels → **N**etwork tier/CDN

### 🧠 "PCS" (BQ optimization)
**P**artition → **C**luster → **S**lot reservations

### 🧠 "SUD = Auto, CUD = Contract"
Sustained Use = automatic; Committed Use = contract.

### 🧠 "FLEX" (New CUD type)
**F**lexible **C**ommitted use = $ per hour, spans families/regions — **F**orgiving if workload changes.

### 🧠 "DORA"
**D**eploy frequency, **O**perations lead time, **R**estore time, **A**ccident (failure) rate — the DevOps KPIs.

### 🧠 "8 Levers"
**R**ightsize, **I**dle shutdown, **S**pot, **C**UD, **S**UD, **T**ier storage, **S**erverless, **N**etwork → "RISCSSNT" (or just "8 levers")

### 🧠 "LTES" (Performance golden signals)
**L**atency, **T**raffic, **E**rrors, **S**aturation

### 🧠 "MV + BI = ⚡ Dashboards"
**M**aterialized **V**iews + **BI** Engine = fastest BQ dashboards

---

## ✅ Practice Checkpoint

### Q1. BQ cost reduction priority
**Scenario:** Company's BQ bill is $80K/month. 80% from `SELECT * FROM events WHERE date = '...'`. No partitioning.

**Options:**
- A) Migrate to Cloud SQL
- B) Partition on `date`, avoid `SELECT *`
- C) Enable BI Engine
- D) Buy 3-yr slot reservation

✅ **Answer: B** — **Root cause fix**: partition + column projection. Reservations won't help if queries are inefficient.

---

### Q2. Fault-tolerant batch
**Scenario:** Nightly ML training job, 6-hour runtime, can restart from checkpoint. Reduce cost.

**Options:**
- A) On-demand GCE
- B) 3-yr CUD
- C) Spot VMs with checkpointing + MIG to auto-recreate
- D) Cloud Run

✅ **Answer: C** — **Spot VMs** = up to 91% off; checkpointing handles preemption. Cloud Run has time limits.

---

### Q3. Unpredictable traffic, minimize cost
**Scenario:** SaaS with 95% idle, 5% heavy bursts. Stateless HTTP.

**Options:**
- A) GKE with 5 always-on nodes
- B) Cloud Run with min=0, max=500
- C) GCE with CUDs
- D) App Engine Flexible

✅ **Answer: B** — **Cloud Run scale-to-zero** + pay-per-request = lowest cost for idle-heavy workloads.

---

### Q4. Dashboards slow
**Scenario:** Looker dashboards on BQ, 200 execs querying same aggregates hourly. Query latency 15+ sec.

**Options:**
- A) Upgrade BQ slots
- B) Materialized Views + BI Engine
- C) Move to Cloud SQL
- D) Replicate to Bigtable

✅ **Answer: B** — Repeat aggregates = **Materialized Views**; sub-second = **BI Engine**.

---

### Q5. Cost attribution
**Scenario:** CFO needs monthly GCP spend broken down by business unit for chargeback.

**Options:**
- A) Manual inspection of invoices
- B) Label resources + billing export to BQ + Looker dashboard
- C) Separate billing accounts per BU
- D) Cloud Monitoring

✅ **Answer: B** — **Labels + billing export to BigQuery** is the FinOps standard.

---

### Q6. Egress reduction
**Scenario:** Global video streaming, 500 TB/month egress, most content is popular (10 videos = 60% traffic).

**Options:**
- A) Standard network tier
- B) Cloud CDN + Media CDN on cacheable content
- C) Move origin closer to users
- D) Reduce video bitrate

✅ **Answer: B** — **CDN** cuts egress costs dramatically by serving from edge cache. 60% of traffic hitting cache = massive savings.

---

### Q7. Stable production baseline
**Scenario:** Stable web tier: 40 always-on n2-standard-4 VMs for 3+ years, expected to stay constant.

**Options:**
- A) Spot VMs
- B) On-demand
- C) 3-year Resource-based CUD
- D) Serverless

✅ **Answer: C** — 3-yr CUD = ~55% off. Workload is stable and long-running.

---

### Q8. Workload might change families
**Scenario:** Web tier currently N2, but planning to test T2D (Arm). Expected stable $/hr spend.

**Options:**
- A) 3-yr N2 Resource CUD
- B) 3-yr Flexible (spend-based) CUD
- C) Spot
- D) On-demand

✅ **Answer: B** — **Flexible CUD** spans families; committing to N2 now would waste money when switching to T2D.

---

### Q9. GCS cold data
**Scenario:** 200 TB of compliance records accessed once/year, must retain 10 yr.

**Options:**
- A) Standard class
- B) Nearline
- C) Archive + locked retention policy
- D) Coldline

✅ **Answer: C** — Once/year → **Archive** ($0.0012/GB). Retention policy for immutability.

---

### Q10. Deploy velocity
**Scenario:** Team deploys manually once every 2 weeks, rollbacks are painful, want to deploy daily.

**Options:**
- A) More engineers
- B) Cloud Build CI + Cloud Deploy with canary + observability
- C) Bigger VMs
- D) Skip tests

✅ **Answer: B** — **Cloud Build + Cloud Deploy + canary** → daily safe deploys.

---

### Q11. Streaming BQ ingestion cost
**Scenario:** Ingesting 1M events/sec into BQ via streaming inserts. Bill is huge.

**Options:**
- A) Continue streaming inserts
- B) Switch to Pub/Sub → BQ subscription (native; cheaper than Dataflow + streaming inserts)
- C) Batch loads via Dataflow + avoid streaming if minute-level latency OK
- D) Move to Bigtable

✅ **Answer: B or C** — **Pub/Sub BQ subscription** (2022+) is cheapest direct path for streaming; if real-time not critical, batch loads are even cheaper.

---

### Q12. Idle resources
**Scenario:** 100+ dev VMs run 24/7 but dev team only works 9-5 weekdays.

**Options:**
- A) Delete them
- B) Instance schedulers to stop nights/weekends + Spot VMs
- C) Just leave them
- D) CUDs

✅ **Answer: B** — **Instance scheduling** for off-hours (~70% savings) + **Spot** for further discount.

---

### Q13. Switch BQ pricing model
**Scenario:** Current on-demand BQ spend: $50K/month, mostly predictable dashboards + reports.

**Options:**
- A) Keep on-demand
- B) Move to BQ Editions with baseline slots (Enterprise or Plus)
- C) Move to Cloud SQL
- D) Use BI Engine only

✅ **Answer: B** — Predictable heavy use → **slots (Editions)** often 30-50% cheaper than on-demand at this scale.

---

### Q14. Serverless cold starts
**Scenario:** Cloud Run API has 5% of requests with 3-second cold starts during low traffic.

**Options:**
- A) Accept it
- B) Min instances > 0 (e.g., 2) + CPU always allocated
- C) Move to GKE
- D) Larger container

✅ **Answer: B** — **Min instances > 0** eliminates cold starts; balance cost with warm-instance fees.

---

### Q15. Team maturity alignment
**Scenario:** 3-engineer startup, no DevOps, need to ship MVP fast.

**Options:**
- A) GKE Standard + Istio
- B) Cloud Run + Firestore + Firebase Hosting
- C) GCE + self-managed Postgres
- D) Anthos

✅ **Answer: B** — **Zero-ops serverless stack** matches team maturity. GKE/Anthos are overkill for 3 people.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **Spot VMs** | Replaces Preemptible (no 24-hr cap, deeper discounts) |
| **Flexible CUDs** | Spend-based, span VM families |
| **BigQuery Editions** | Standard / Enterprise / Enterprise Plus replace flat-rate |
| **Autoscaling BQ slots** | Baseline + autoscale flex |
| **Active Assist / Recommender** | Expanded: Cloud SQL, GKE, IAM |
| **Gemini Cloud Assist** | AI-powered optimization recommendations |
| **Hyperdisk Autotune** | Pay for actual IOPS used |
| **Pub/Sub BQ subscription** | Cheaper than Dataflow for BQ streaming |
| **GCS Autoclass** | Auto-tier without lifecycle rules |
| **Cost Anomaly Detection** | ML-based anomaly alerts |
| **Budget with programmatic actions** | Pub/Sub → automated response |
| **Cloud Billing API** | Pull billing data for FinOps tools |
| **BigQuery column-level pricing breakdown** | Understand query cost drivers |

---

## 🎯 Final Domain 4 Checklist

Before exam day, you should be able to:

- [ ] Recite the 8 cost optimization levers
- [ ] Know when to pick Spot vs CUD vs On-demand vs Serverless
- [ ] Know Flexible CUD vs Resource CUD
- [ ] Know SUD is automatic, CUD is contract
- [ ] Know BigQuery partition + cluster + MV + BI Engine stack
- [ ] Know on-demand vs capacity (slots) pricing break-even
- [ ] Know GCS class price/access tradeoffs + lifecycle rules
- [ ] Know network tiers (Premium vs Standard)
- [ ] Know CDN + egress reduction patterns
- [ ] Know FinOps stack: labels + billing export + budgets
- [ ] Know DORA metrics
- [ ] Know SRE SLI/SLO/SLA/error budget relationship
- [ ] Know Recommender products and what each does
- [ ] Know when to switch BQ pricing models

> **Domain 4 is where smart architects separate from good architects. Cost + performance + velocity are ALWAYS tradeoffs — know how to balance.** 💰
