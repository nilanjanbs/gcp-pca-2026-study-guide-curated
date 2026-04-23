# 🛡️ Domain 6: Ensuring Solution and Operations Reliability

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (6 of 6)**

**Exam Weight:** ~14%

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [SRE Fundamentals (SLI/SLO/SLA/Error Budgets)](#-sre-fundamentals-slislosla-error-budgets)
3. [6.1 High Availability (HA) Architecture](#-61-high-availability-ha-architecture)
   - [Compute Resilience (MIGs, Autohealing, Active-Standby)](#-compute-resilience)
   - [Network Availability (LB, Health Checks, VPN/IC HA)](#-network-availability)
   - [Container Health (GKE Probes, Cloud Run HA)](#-container-health)
4. [6.2 Disaster Recovery (DR) Strategies](#-62-disaster-recovery-dr-strategies)
   - [Recovery Planning (RPO/RTO, Backup & DR)](#-recovery-planning-rporto-backup--dr)
   - [Data Protection & Failover](#-data-protection--failover)
   - [State & Session Management](#-state--session-management)
5. [6.3 Scalability & Resource Reliability](#-63-scalability--resource-reliability)
   - [Dynamic Scaling](#-dynamic-scaling)
   - [Data Integrity under Load (Hotspots, Watermarks)](#-data-integrity-under-load)
6. [6.4 Operational Continuity](#-64-operational-continuity)
   - [Maintenance & Patching](#-maintenance--patching)
   - [Observability for Reliability](#-observability-for-reliability)
7. [Incident Management & Postmortems](#-incident-management--postmortems)
8. [Exam Traps & Tricks](#-exam-traps--tricks)
9. [Decision Frameworks](#-decision-frameworks)
10. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
11. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
12. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

Domain 6 is about **keeping things running when bad stuff happens**. It's heavy on SRE practices, HA topology, DR planning, and observability. Expect scenarios like:

- "Need 99.99% SLA globally"
- "RTO = 5 min, RPO = 1 min" — design the DR plan
- "Regional outage hits — how do we fail over?"
- "GKE pod keeps restarting — what's wrong with probes?"
- "Error budget exhausted — should we ship the feature?"

The exam tests these sub-skills:

1. **SRE math** — SLI vs SLO vs SLA, error budgets, burn rate alerts
2. **HA vs DR distinction** — and when to use each
3. **RPO/RTO planning** — mapping requirements to DR strategy
4. **Health checks & probes** — Liveness vs Readiness vs Startup
5. **Failover mechanisms** — Cloud SQL HA, Spanner, regional vs multi-region
6. **Autoscaling reliability** — cold starts, over-provisioning
7. **Hotspotting prevention** — Bigtable, GCS key design
8. **Observability** — Monitoring, Logging, Trace, Profiler

### 🧠 Modulator's Deep-Dive Key Concepts

> ⚠️ **Three things to tattoo on your brain for Domain 6:**
>
> 1. **HA ≠ DR.** HA = failover *within* a region/across zones seamlessly. DR = plan for when an *entire region or provider* fails.
> 2. **Liveness Probe = "is it crashed? (restart it)"**. **Readiness Probe = "is it ready for traffic? (route to it)"**. They are NOT interchangeable.
> 3. **For global resiliency, prioritize Spanner (synchronous cross-region replication) over Cloud SQL (asynchronous cross-region)** to minimize RPO.

---

## 🎯 SRE Fundamentals (SLI/SLO/SLA/Error Budgets)

### 🧠 The Big 3 (+ 1)

| Term | Full Name | Description | Example |
|---|---|---|---|
| **SLI** | Service Level Indicator | Measurable metric | "% of HTTP requests returning 2xx under 200ms" |
| **SLO** | Service Level Objective | Internal target for SLI | "99.95% of requests succeed" |
| **SLA** | Service Level Agreement | External contract with penalties | "99.9% or we credit customers" |
| **Error Budget** | Allowed failure budget | 100% − SLO | 99.95% SLO = 0.05% = ~22 min/month |

### 🔑 Relationships

- **SLA < SLO < SLI ceiling** — SLA is the loosest (external), SLO is stricter (internal buffer), SLI is the measurement itself
- Example: SLA = 99.9%, SLO = 99.95%, SLI = actual measurement
- **Error budget = 100% − SLO** (NOT SLA)

### 🔑 Error Budget Policy

When **within budget:**
- ✅ Ship features
- ✅ Take calculated risks
- ✅ Move fast

When **over budget (burned through):**
- ❌ Freeze risky releases
- ❌ Halt feature development
- ✅ Focus on reliability work
- ✅ Incident retro deep-dive

**Exam rule:** "Error budget exhausted" → **freeze features, invest in reliability**.

### 🔑 The 4 Golden Signals (SRE Book)

| Signal | What |
|---|---|
| **Latency** | Time to serve a request |
| **Traffic** | Volume of requests |
| **Errors** | Rate of failed requests |
| **Saturation** | How full the system is (CPU, memory, queue depth) |

Design dashboards and alerts around these four. **Mnemonic: LTES**.

### 🔑 Burn Rate Alerts (modern SRE)

Instead of alerting on raw thresholds, alert on **how fast you're consuming error budget**:

- **Fast burn** (high burn rate, short window) → page immediately (e.g., 2% in 1 hour)
- **Slow burn** (low burn rate, long window) → ticket (e.g., 10% in 3 days)

**Exam rule:** Modern alerting = **burn-rate alerts on SLO**, not "CPU > 80%".

---

## 🏗️ 6.1 High Availability (HA) Architecture

### 🧠 HA Fundamentals

**HA = failing over WITHIN a region** (across zones) OR **between regions** seamlessly, to survive zone/region failures **without data loss or significant downtime**.

**HA is about:**
- Redundancy (more than one of everything)
- Health checks (detect failures)
- Automatic failover (no human intervention)
- No single point of failure

### 🧱 Compute Resilience

#### 🔑 Managed Instance Groups (MIGs)

**A MIG is a fleet of identical VMs** managed together — auto-healing, auto-scaling, auto-updating.

| Component | Role |
|---|---|
| **Instance template** | Blueprint for VMs (image, machine type, startup script) |
| **Autoscaler** | Scales based on CPU, LB, custom metrics |
| **Autohealer (health check)** | Recreates unhealthy VMs |
| **Update policy** | How rolling updates happen |
| **Stateful config** | Per-instance disks/metadata (stateful MIGs) |

#### 🔑 MIG Autohealing and Automatic Restart

**Two layers of resilience:**

| Layer | What It Does | When |
|---|---|---|
| **Automatic restart** | VM-level: if host fails, VM restarts on new host | Infrastructure failure |
| **Autohealing** | Group-level: if app-layer health check fails, MIG recreates the VM | Application failure (app crashed, stuck) |

**Autohealing requires:**
- **Application-layer health check** (not just LB health check — those only affect traffic routing)
- **Initial delay** (grace period before checks start — give apps time to boot)
- **Health check interval & threshold**

**⚠️ Key exam distinction:**
- **LB health check** → routes traffic away from unhealthy VM (keeps it alive)
- **Autohealing health check** → recreates the VM entirely (replaces it)

#### 🔑 MIG Location: Regional vs. Zonal

| Type | Zones | HA? | When to Use |
|---|---|---|---|
| **Zonal MIG** | 1 zone | ❌ No | Dev, test, batch in 1 zone |
| **Regional MIG** | 3+ zones | ✅ Yes | ⭐ **Always for production** |

**Regional MIG benefits:**
- Spreads VMs across zones automatically
- Survives single-zone outage
- Default **target shape: EVEN** (equal VMs per zone)
- Can also use **BALANCED** (optimized for availability)

**Exam rule:** "Production HA" → **Regional MIG**, never zonal.

#### 🔑 Active-Standby Architectural Models

**Active-Standby** (also called Active-Passive):
- One primary active instance serving traffic
- One or more standbys ready to take over on failure
- Failover can be: automatic (via LB health check) or manual

**Active-Active:**
- Multiple instances serve traffic simultaneously
- Load balancer distributes across them
- Survives node failure with no cutover

**When to pick which:**

| Model | Pros | Cons | Typical Use |
|---|---|---|---|
| **Active-Standby** | Simpler state management, avoids split-brain | Standby resources idle | Legacy DBs, stateful licensed apps |
| **Active-Active** | Full resource utilization, zero-failover downtime | Complex state sync, conflict resolution | Web tier, modern microservices |

**GCP-native HA per service:**

| Service | HA Model | How |
|---|---|---|
| **Cloud SQL HA** | Active-Standby (same region, different zone) | Auto failover via regional PD + LB |
| **Spanner** | Active-Active (multi-region) | Synchronous replication |
| **Bigtable replication** | Active-Active (multi-cluster) | App profile routing |
| **Firestore / Memorystore HA** | Active-Standby | Managed failover |
| **MIGs + LB** | Active-Active (multi-zone) | LB distributes |

### 🌐 Network Availability

#### 🔑 Global vs. Regional Load Balancers

| Feature | Global LB | Regional LB |
|---|---|---|
| **Scope** | Worldwide | Single region |
| **Anycast IP** | ✅ 1 IP globally | ❌ 1 IP per region |
| **User routing** | Nearest POP | Single endpoint |
| **Failover** | Cross-region automatic | Within region only |
| **Cost** | Higher | Lower |
| **Use** | Global apps, 99.99%+ SLA | Single-region apps |

**Global LB types:**
- Global External Application LB (HTTP/S)
- Global External Proxy Network LB (TCP/SSL)
- Cross-region Internal Application LB

**Exam rule:** "Global low-latency, 99.99%+" → **Global External Application LB** + multi-region backends.

#### 🔑 Health Checks

**Types:**
| Type | Protocol | Use |
|---|---|---|
| **HTTP(S) health check** | HTTP(S) | Web apps |
| **TCP health check** | TCP | Any TCP service |
| **SSL health check** | SSL/TLS | TLS services |
| **gRPC health check** | gRPC | gRPC services |

**Key parameters:**
- **Check interval** (how often to probe) — typical: 10s
- **Timeout** (max wait for response) — typical: 5s
- **Healthy threshold** (consecutive successes to mark healthy) — typical: 2
- **Unhealthy threshold** (consecutive failures to mark unhealthy) — typical: 3

**⚠️ Tuning gotchas:**
- Too aggressive → flapping (healthy → unhealthy → healthy)
- Too lax → slow to detect real failures
- Rule of thumb: unhealthy detection within 30–60 seconds

#### 🔑 Session Affinity

Makes LB send **same user's requests to the same backend**.

| Mode | Basis |
|---|---|
| **None** (default) | Round-robin |
| **Client IP** | Hash of client IP |
| **Generated cookie** | LB creates cookie |
| **Header field** | Custom header value |
| **HTTP cookie** | Application-provided cookie |
| **Stateful: Client IP + Proto** | Consistent hash |

**When to use:**
- ✅ Stateful session data (shopping cart in memory) — but better: externalize state
- ✅ Expensive per-user setup (warmed caches)
- ❌ True stateless apps (doesn't help)

**⚠️ Exam rule:** Session affinity **doesn't guarantee stickiness on failover** — if backend dies, user is reassigned.

#### 🔑 Multi-region Cloud VPN & HA Interconnect

**HA VPN for multi-region:**
- Deploy HA VPN gateways in **multiple regions** on-prem → cloud
- Each gateway: 2 tunnels, BGP, 99.99% SLA
- Multi-region: additional redundancy for cross-region failure

**HA Interconnect (Dedicated):**
- Deploy across **2 edge availability domains** OR **2 metros**
- **99.99% SLA** requires redundant topology (4 × 10G across 2 metros)
- Each metro has its own Google-redundant edge

**Use case:** Mission-critical enterprise hybrid workloads.

### 📦 Container Health

#### 🔑 GKE Liveness and Readiness Probes (⚠️ exam favorite)

**Three probe types:**

| Probe | Purpose | On Failure | When Used |
|---|---|---|---|
| **Liveness** | "Is it crashed/stuck?" | **Restart container** | Detect deadlocks, hangs |
| **Readiness** | "Is it ready for traffic?" | **Remove from Service endpoints** (no restart) | During startup, overload, graceful drain |
| **Startup** | "Has it finished starting?" | Delay liveness/readiness until app is up | Slow-starting apps |

**Key distinction:**
- **Liveness probe fails** → container is **restarted** (kubelet kills + recreates)
- **Readiness probe fails** → pod **stops receiving traffic** but stays running (might recover)

**Probe types:**
- **HTTP GET** (most common) — respond 200 = healthy
- **TCP Socket** — open port = healthy
- **Exec** — command returns 0 = healthy
- **gRPC** — health protocol

**Example:**
```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

**⚠️ Common misconfigurations:**
- Liveness that fails during slow startup → pod stuck in restart loop → use **Startup Probe**
- Readiness too aggressive → flapping pods removed/added to service
- Same endpoint for liveness + readiness → misleading; often separate `/healthz` (live) vs `/ready` (ready to serve)

#### 🔑 Cloud Run High Availability Architecture

**Cloud Run is HA by default:**
- **Multi-zone** within a region (automatic)
- Load-balanced across instances
- Regional resource (survives zonal outage)

**For multi-region HA:**
- Deploy Cloud Run service in **multiple regions**
- Front with **Global External Application LB + Serverless NEGs**
- User hits nearest region via anycast

**Features for HA:**
- **Min instances** — keep warm, reduce cold starts
- **CPU always allocated** — for background work
- **Max instances** — prevent runaway
- **Concurrency tuning** — requests per instance
- **Revisions** — instant rollback to previous version
- **Health checks** (startup + liveness) — built-in for containers

**⚠️ Cloud Run failures:**
- Container crashes → auto-replaced (built-in)
- Regional outage → you need **multi-region + Global LB**

---

## 🌋 6.2 Disaster Recovery (DR) Strategies

### 🧠 HA vs DR Distinction

> ⚠️ **Critical exam concept:** 
> - **HA = survive zone / regional component failure** (seamless within same region)
> - **DR = survive entire-region / provider / catastrophic failure** (failover plan to different region)

### 🧠 Recovery Planning (RPO/RTO, Backup & DR)

#### 🔑 RPO vs RTO (memorize)

| Metric | Full Name | Meaning | Smaller = |
|---|---|---|---|
| **RPO** | **R**ecovery **P**oint **O**bjective | How much data loss is acceptable | More frequent backups / replication |
| **RTO** | **R**ecovery **T**ime **O**bjective | How long to restore service | Faster failover / hotter standby |

**Example:**
- RPO = 1 min → replication every minute (e.g., Spanner, Cloud SQL cross-region async lag)
- RPO = 1 hr → hourly backups acceptable
- RTO = 0 → active-active multi-region
- RTO = 4 hr → backup-restore from GCS

#### 🔑 DR Strategies (4 common tiers)

| Strategy | RTO | RPO | Cost | Description |
|---|---|---|---|---|
| **Backup & Restore** | Hours–days | Hours | $ | Backups in GCS, restore on-demand |
| **Pilot Light** | Minutes–hours | Minutes | $$ | Minimal infra in DR region, scale up |
| **Warm Standby** | Minutes | Seconds–minutes | $$$ | Scaled-down full replica always running |
| **Hot Standby / Active-Active** | Near zero | Near zero | $$$$ | Full multi-region, LB auto-routes |

#### 🔑 Backup and DR Service (managed service)

**Backup and DR Service (formerly Actifio):**
- **Managed backup** for GCE VMs, VMware Engine, Oracle, SAP HANA, SQL Server, file systems
- **Application-consistent** backups (quiesce the app)
- **Incremental-forever** model (save space)
- **Cross-region** backup vault
- **Automated DR orchestration** — scripted failover
- **Immutable backups** (ransomware protection)

**Exam rule:** "Centralized backup and DR for GCE + VMware + DBs" → **Backup and DR Service**.

#### 🔑 Compute Engine Disaster Recovery Strategies

**Options in order of RTO/RPO:**

1. **Persistent Disk Snapshots** (hourly/daily)
   - Scheduled via **resource policy**
   - Incremental, cross-region
   - RTO: minutes–hours (restore snapshot to new disk)
   - RPO: matches snapshot frequency

2. **Machine Images**
   - Full image: instance template + disks + metadata
   - Restore in another region/zone
   - Good for periodic "golden image" DR

3. **Regional Persistent Disks**
   - Synchronously replicated across **2 zones in same region**
   - Zone failover: instant
   - ❌ NOT cross-region

4. **MIGs in multiple regions behind Global LB**
   - Active-Active
   - Near-zero RTO/RPO (stateless tier)

5. **Backup and DR Service**
   - Managed end-to-end

#### 🔑 Snapshot Scheduling

```yaml
# Resource policy example
snapshotSchedulePolicy:
  schedule:
    dailySchedule:
      startTime: "03:00"
  retentionPolicy:
    maxRetentionDays: 30
    onSourceDiskDelete: KEEP_AUTO_SNAPSHOTS
  snapshotProperties:
    labels: {env: prod}
    storageLocations: [us-central1, us-east1]  # Cross-region
```

### 🗃️ Data Protection & Failover

#### 🔑 Cloud SQL Availability and Failover Mechanisms

**Cloud SQL HA configuration:**
- **Primary** instance in one zone
- **Standby** instance in another zone (same region)
- **Synchronous replication** via regional persistent disk
- **Automatic failover** on primary failure (~60 seconds)
- **99.95% SLA**

**How it works:**
1. Regional PD synchronously replicates data across 2 zones
2. Health checks monitor primary
3. On failure, standby takes over; DNS/private IP stays the same
4. Failed primary is rebuilt as new standby

**⚠️ Key exam trap:** Cloud SQL HA is **intra-regional** (same region, different zone). For cross-region DR, you need **cross-region read replicas**.

#### 🔑 Cloud SQL Cross-Region DR

**Cross-region read replicas:**
- **Asynchronous** replication
- Lives in different region
- Can be **promoted** to primary if region outage
- ⚠️ **Manual promotion** (not automatic)
- **Replication lag** varies (seconds to minutes)

**Alternative: Cloud SQL Enterprise Plus + advanced DR features**
- Faster recovery
- Near-zero downtime maintenance
- Better DR tooling

**Exam rule:**
- "Zone failover, same region" → **Cloud SQL HA** (primary + standby)
- "Region failover" → **Cross-region read replica** (manual promote)
- "Multi-region zero-RPO SQL" → **Spanner** (not Cloud SQL)

#### 🔑 Cloud SQL Backup Strategies

**Automated backups:**
- **Daily full backup** (you set time)
- **Binary logging** for point-in-time recovery (PITR)
- **Retention**: up to 365 days

**On-demand backups:**
- Manual, retained until deleted
- Useful before risky changes

**Point-in-Time Recovery (PITR):**
- Restore to any moment (up to retention window)
- Requires binary logs enabled
- Recovers to **new instance**

**Managing replication lag:**
- Monitor via **`seconds_behind_master`** metric
- If lag grows:
  - Upgrade replica machine size
  - Check network bandwidth
  - Reduce write load on primary
  - Check for long-running transactions
- **Alert on replication lag > threshold** (e.g., 30 seconds)

#### 🔑 Database DR Comparison

| DB | HA (zone) | DR (region) | Global Strong Consistency | RPO |
|---|---|---|---|---|
| **Cloud SQL** | Auto failover (~60s) | Cross-region async replica (manual promote) | ❌ | Seconds–minutes (async) |
| **AlloyDB** | Auto failover | Cross-region replica | ❌ | Seconds–minutes |
| **Spanner** | Automatic | Multi-region config: automatic | ✅ (synchronous) | **~0** |
| **Bigtable** | Multi-cluster replication | Multi-cluster across regions | Eventually consistent (or strong w/ single-cluster routing) | Seconds |
| **Firestore** | Multi-region by default | Built-in | ✅ | ~0 |
| **BigQuery** | Multi-region/regional | Cross-region replication (Editions Plus) | N/A (OLAP) | Hours–minutes |
| **GCS** | Multi-region/dual-region | Turbo replication (dual-region) | Strong after write | Seconds–minutes |

**⚠️ Modulator's Deep-Dive:** For **global resiliency**, prioritize **Spanner (synchronous replication across regions)** over **Cloud SQL (asynchronous cross-region)** when RPO must be minimized.

#### 🔑 Cloud Storage Data Management Features

**Versioning:**
- Keep old object versions on overwrite/delete
- **Noncurrent versions** can be queried / restored
- Use **lifecycle rules** to delete old versions
- Protection against accidental delete or ransomware

```bash
gsutil versioning set on gs://my-bucket
```

**Retention Policy:**
- **Minimum retention** before object can be deleted
- **Locked retention** — permanent, immutable (even admins can't reduce)
- Great for regulatory compliance (SEC, HIPAA, FINRA)

**Object Holds:**
- **Event-based hold** — locks object until event (e.g., legal case ends)
- **Temporary hold** — locks until removed
- Prevents deletion regardless of retention policy expiration

**Bucket Lock:**
- Lock **retention policy** permanently
- Once locked, retention **cannot be reduced** (only increased)
- WORM (Write Once Read Many) compliance

**Soft Delete (2024+):**
- Auto-enabled on new buckets
- Retains deleted objects for **7 days** (default)
- Recover accidentally deleted objects
- Can disable or configure retention (1–90 days)

**Turbo Replication (dual-region buckets):**
- RPO **< 15 minutes** (target)
- For mission-critical cross-region bucket replication
- Optional premium feature

### 🪑 State & Session Management

#### 🔑 App Engine State Management and Memcache

**App Engine Standard supports:**
- **Memcache** — distributed in-memory cache (shared or dedicated)
  - **Shared memcache** (free, best-effort)
  - **Dedicated memcache** (paid, guaranteed capacity)
- **Task Queue / Cloud Tasks** — async work
- **Datastore / Firestore** — for persistent state

**Session management patterns:**

| Pattern | How |
|---|---|
| **Externalized sessions** | Store in **Memorystore (Redis)** or **Firestore** — any instance can serve |
| **Sticky sessions** | LB session affinity — same instance serves same user |
| **Stateless (JWT)** | User state in token — no server session storage |

**Exam rule:** For **modern HA apps**, **externalize session state** (Memorystore/Redis or Firestore) so any instance can serve any user. Avoid sticky sessions where possible.

#### 🔑 Memorystore for Session Store

**Memorystore for Redis (or Valkey):**
- Managed Redis/Valkey
- **Basic tier** (single node) vs **Standard tier** (HA with replica + auto-failover)
- **Memorystore for Redis Cluster** (horizontal scale, 2024+)
- Use for: sessions, leaderboards, rate limiting, caching

---

## 📈 6.3 Scalability & Resource Reliability

### 🧠 Dynamic Scaling

#### 🔑 GKE Autoscaling and Overprovisioning

**Three autoscaling layers:**

| Layer | Tool | Scales |
|---|---|---|
| **Pod horizontal** | HPA | Pod replicas (based on CPU/mem/custom metrics) |
| **Pod vertical** | VPA | Pod CPU/mem requests (based on history) |
| **Cluster horizontal** | Cluster Autoscaler (CA) | Node count |
| **Pool creation** | Node Auto-Provisioning (NAP) | New node pools |

**Autopilot:** all of this is auto-enabled; no config needed.

**Overprovisioning for faster scale-up:**
- Problem: CA takes time (1–2 min) to add nodes — pods wait
- Solution: **Balloon pods** or **Pause pods**
  - Deploy low-priority placeholder pods that consume resources
  - When real workload needs space, they get evicted, and CA scales up **preemptively**
  - Real workload fits immediately into "pre-warmed" capacity

**When to use:**
- Latency-sensitive apps that can't wait for node provisioning
- Spiky workloads with known burst patterns

#### 🔑 Cloud Run Autoscaling and Cold Starts

**How Cloud Run scales:**
- Scales **per-instance based on concurrency** (default 80 requests/instance)
- Scales **0 to max** instances
- **Cold start** = first request to new instance (container pull + boot)

**Managing cold starts:**

| Technique | Effect |
|---|---|
| **Min instances > 0** | Keep warm instances always ready |
| **CPU always allocated** | Faster response; enables background work |
| **Smaller container image** | Faster pull |
| **Gen 2 execution environment** | Faster startup, faster I/O |
| **Startup CPU boost** | Automatic more CPU during boot |
| **Concurrency tuning** | Fewer instances = fewer cold starts per request |

**Exam trap:** "5% of requests slow from cold starts" → **min-instances > 0** (accept small always-on cost).

#### 🔑 Compute Engine MIG Autoscaling

**MIG autoscaling signals:**
- **CPU utilization** — general-purpose
- **HTTP LB utilization** — LB-fronted web apps
- **Cloud Monitoring metric** — any custom signal
- **Pub/Sub queue depth** — for workers
- **Schedule-based** — predictable patterns

**Key settings:**
- **Min replicas** (floor)
- **Max replicas** (ceiling)
- **Target utilization** (e.g., 60% CPU)
- **Cool-down period** (time between scaling actions — prevent thrashing)
- **Scale-in controls** (max % to remove at once — prevent over-aggressive scale-down)

**⚠️ Anti-pattern:** Aggressive scale-in during traffic trough → not enough capacity for spike that follows → use **scale-in controls**.

### 🔥 Data Integrity under Load

#### 🔑 Handling Out-of-Order Data in Dataflow (Watermarks + Triggers)

**Problem:** In streaming pipelines, data arrives out of order (late events, lagging sources).

**Dataflow solution:**

| Concept | What It Does |
|---|---|
| **Event time** | When event happened (in data) |
| **Processing time** | When Dataflow processes it |
| **Windows** | Group events by time (fixed, sliding, session) |
| **Watermark** | Dataflow's estimate of "event time progress" — when to close a window |
| **Triggers** | Determine when to emit window results (early, on-time, late) |
| **Allowed lateness** | Tolerate late events up to this duration |

**Typical pattern:**
```
Fixed window of 1 hour
- Watermark fires at end of hour → emit on-time results
- Allowed lateness: 10 min → late events still trigger updates
- After 10 min lateness → window is discarded
```

**Triggers:**
- **AfterWatermark()** — default; emit once window closes
- **AfterProcessingTime()** — emit at intervals
- **AfterCount()** — emit after N elements
- Composite — early triggers + late triggers

**Exam rule:** "Streaming pipeline with late-arriving data" → **watermarks + triggers + allowed lateness** in Dataflow.

#### 🔑 Preventing Hotspotting in Bigtable (Row Key Design)

**Hotspotting:** When a few row keys/tablets receive disproportionately high traffic → overloaded nodes, scale issues.

**Good row key design:**

| Bad Pattern | Why Bad | Good Alternative |
|---|---|---|
| **Timestamp as prefix** (`20260423-...`) | All writes hit same tablet (latest time) | **Reverse timestamp** or **hash prefix** |
| **Sequential IDs** (1, 2, 3, ...) | Recent IDs cluster | **Hash-prefix** or **random salt** |
| **User ID as prefix** for popular user | That user's tablet overloaded | **Hash or distribute** user IDs |
| **Country code only** (US, GB, ...) | Popular countries hotspot | Add variability (e.g., `US#hash#...`) |

**Good patterns:**
- **Field promotion** — put high-cardinality, queryable fields first
- **Salted keys** — prefix with hash to spread load
- **Reversed timestamps** — `(MAX_LONG - timestamp)` so latest is first but spread across tablets when combined with other dimensions
- **Tall narrow tables** — many rows, few columns

**Key design mantra:** Keys should **distribute writes/reads evenly across tablets**.

**Tools:**
- **Key Visualizer** — heatmap tool to detect hotspots
- **Bigtable Autoscaling** — adjust node count based on CPU

#### 🔑 Preventing Hotspotting in Cloud Storage

**Hotspotting pattern:** Sequential object names → same shard → rate limits (~5000 writes/sec per prefix).

**Bad:**
```
gs://my-bucket/2026-04-23-000001.log
gs://my-bucket/2026-04-23-000002.log
gs://my-bucket/2026-04-23-000003.log
...
```
All writes hit the same prefix, overwhelming shard.

**Good (random/hash prefix):**
```
gs://my-bucket/a7f9/2026-04-23-000001.log
gs://my-bucket/c2e4/2026-04-23-000002.log
gs://my-bucket/9d31/2026-04-23-000003.log
```
Writes distribute across shards.

**Rules:**
- **High write rates (>1000/sec)** need hash-prefixed object names
- **Sequential reads** can stay sequential (GCS auto-scales reads differently)
- **Use date/time as suffix, not prefix**

**Exam rule:** "High-volume streaming writes to GCS hitting rate limits" → **randomize/hash prefix** of object names.

---

## 🔧 6.4 Operational Continuity

### 🛠️ Maintenance & Patching

#### 🔑 Compute Engine On-Host Maintenance

When Google performs host maintenance:

| Policy | Behavior |
|---|---|
| **Migrate (default)** | Live-migrate VM to another host — **no downtime** |
| **Terminate** | Stop VM, restart after maintenance |

**When Terminate is required:**
- VMs with **GPUs** (some types)
- **Preemptible/Spot** VMs
- **Sole-tenant nodes** (optional)
- **Confidential VMs** (depending on config)

**Maintenance notifications:**
- Logged in **System Event audit logs**
- **Notifications API** for advance notice
- **60-second notice** before impact (for migrate)

**Best practices:**
- Use **Migrate** unless specific need for Terminate
- **Graceful shutdown handling** for Terminate VMs
- **Health checks** and retries in app code

#### 🔑 OS Patch Management

**VM Manager** (previously OS Config):
- **Patch jobs** — schedule and execute OS patches
- **Patch management** — track patch state across fleet
- **OS inventory** — installed packages, versions
- **OS policies** — enforce configuration

**How it works:**
1. Install **OS Config agent** (usually pre-installed on GCP images)
2. Create patch jobs — specify:
   - Targets (instances by filter/labels)
   - Patch config (OS-specific settings)
   - Pre/post patch scripts
   - Rollout strategy (e.g., 10% at a time)
   - Maintenance window
3. Monitor via patch reports

**Best practices:**
- Patch **non-prod** first
- Use **canary rollouts** (10% → 50% → 100%)
- **Maintenance windows** during low traffic
- **Automated patching** for security CVEs

#### 🔑 Managed Instance Group Update Policies

**Rolling Update** options:

| Param | Purpose |
|---|---|
| **maxSurge** | Max extra instances during update (e.g., 1 or 25%) |
| **maxUnavailable** | Max unavailable during update (e.g., 0 or 25%) |
| **minReadySec** | Wait after instance healthy before continuing |
| **replacementMethod** | `SUBSTITUTE` (replace) or `RECREATE` (delete + create) |
| **type** | `PROACTIVE` (execute on template change) or `OPPORTUNISTIC` (only when recreating for other reason) |

**Update strategies:**

| Strategy | Config |
|---|---|
| **Rolling update** | `maxSurge=1, maxUnavailable=0` for zero-downtime |
| **Canary** | Versioned instance templates — small portion on new, monitor, promote |
| **Pause** | Pause mid-update; resume later |
| **Cancel** | Stop mid-update; keep current state |

**Canary via Versioned Instance Templates:**
```yaml
instanceTemplate: template-v1
versions:
  - instanceTemplate: template-v2
    targetSize: 
      fixed: 5   # 5 of 50 on canary
```

**Exam rule:** "Zero-downtime MIG upgrade" → rolling update with `maxUnavailable=0, maxSurge>=1`.

### 🔭 Observability for Reliability

#### 🔑 Cloud Operations Suite

| Product | Purpose |
|---|---|
| **Cloud Monitoring** | Metrics, dashboards, alerts, uptime checks |
| **Cloud Logging** | Log aggregation, search, export |
| **Cloud Trace** | Distributed tracing |
| **Cloud Profiler** | Continuous CPU/memory profiling |
| **Error Reporting** | Aggregates app errors |
| **Cloud Debugger** | (Deprecated in 2023) |

#### 🔑 Cloud Monitoring Uptime Checks

**Proactive failure detection:**
- **HTTP(S)** — web URLs
- **TCP** — service ports
- **From multiple regions** (up to 6 POPs) — detect regional issues
- **Interval**: 1, 5, 10, 15 min
- **Alerts** on failure (via notification channel)

**Use cases:**
- External API availability monitoring
- Multi-region health verification
- SSL cert expiry monitoring (via response code check)
- Synthetic user flow (Synthetic Monitoring)

**Key metrics from uptime check:**
- **Check passed** — success rate
- **Request latency** — perf measure
- **Availability SLI** — feed into SLOs

**Exam rule:** "Monitor external URL from multiple regions" → **Cloud Monitoring Uptime Check**.

#### 🔑 Cloud Logging Exports and Sinks

**Default log retention:**
- **_Default bucket**: 30 days
- **_Required bucket** (Admin Activity, System Event, Policy Denied audit logs): **400 days** free

**Log sinks — route logs elsewhere:**

| Destination | Use Case |
|---|---|
| **Cloud Storage** | Long-term archive, compliance (locked retention) |
| **BigQuery** | Analytics on logs, SQL queries |
| **Pub/Sub** | Real-time forwarding to SIEM (Splunk, Chronicle) |
| **Other Cloud Logging project** | Aggregate across projects |

**Aggregated sinks (Org/Folder level):**
- Capture logs from all downstream projects
- Send to **centralized security project** or **audit project**
- Exam-common for compliance scenarios

**Log-based metrics:**
- **Counter metrics** — count log entries matching filter (e.g., 5xx errors per minute)
- **Distribution metrics** — extract numeric value from log entry

**Log-based alerts:**
- Alert when log pattern matches (e.g., "IAM policy changed")
- Useful for security + audit

**Exam rule:**
- "Long-term audit logs beyond default" → **Log sink to GCS** (with locked retention)
- "Real-time log forwarding to SIEM" → **Log sink to Pub/Sub**
- "SQL queries on logs" → **Log sink to BigQuery**

#### 🔑 Cloud Trace Distributed Tracing

- **OpenTelemetry** compatible
- **Auto-instrument** supported languages (Go, Java, Node, Python, etc.)
- **Trace waterfall** across microservices
- **Identifies latency bottlenecks**
- Integrated with Cloud Run, GKE, App Engine, GCE

**Use when:** Microservices debugging, performance analysis.

#### 🔑 Cloud Profiler Continuous Profiling

- **CPU + Heap** profiling
- **Production-safe** — low overhead
- Supports Go, Java, Python, Node, .NET
- Identifies hot code paths, memory leaks

---

## 🚨 Incident Management & Postmortems

### 🧠 Incident Response Lifecycle

1. **Detect** — alerts fire, uptime check fails
2. **Respond** — on-call engages, acknowledges
3. **Diagnose** — check dashboards, logs, traces
4. **Mitigate** — rollback, failover, scale up
5. **Resolve** — root cause addressed
6. **Postmortem** — learn, improve

### 🔑 Postmortem Best Practices (SRE)

- **Blameless** — focus on systems, not people
- **Timeline** — detailed sequence of events
- **Root cause analysis** — 5 whys
- **Action items** — specific, owned, time-bound
- **Shared broadly** — spread learning

### 🔑 Runbooks

- **Written procedures** for common incidents
- Linked to alerts
- Updated post-incident
- Living documents

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 20 Traps in Domain 6

1. **SLA vs SLO vs SLI confusion** — SLA = external, SLO = internal target, SLI = measurement
2. **99.99% SLA with single-zone** → impossible
3. **Cloud SQL HA is regional** (zone failover only); cross-region needs **replica + manual promote**
4. **Spanner multi-region for zero-RPO** (Cloud SQL is async across regions)
5. **Regional Persistent Disk = 2 zones, same region** (not cross-region DR)
6. **Autohealing ≠ LB health check** — autohealing recreates VMs; LB reroutes
7. **Liveness Probe failure = container restart**; **Readiness Probe failure = removed from service** (NO restart)
8. **Same endpoint for liveness + readiness** — common anti-pattern
9. **Uptime Check lives in Cloud Monitoring**, not Logging
10. **Data Access audit logs are OFF by default**
11. **Cloud Logging default retention = 30 days** (need sinks for longer)
12. **Multi-region GCS ≠ dual-region** — multi-region Google-chosen, dual-region you-chosen
13. **Turbo replication = dual-region feature** (RPO < 15 min)
14. **Bigtable timestamp-prefix row keys hotspot** — use reverse-timestamp or hash
15. **GCS sequential object names hotspot at >1000 writes/sec** — hash-prefix
16. **Zonal MIG in HA scenario** → wrong, use Regional MIG
17. **Error budget exhausted** → **freeze features**, not "add more alerts"
18. **Cloud SQL PITR requires binary logging enabled**
19. **Cloud Run is regional by default** — multi-region needs Global LB + multi-region deploy
20. **Dataflow out-of-order data** → watermarks + triggers + allowed lateness

### ⚠️ "Sounds Right but Isn't"

| Seems Right | Actually Right | Why |
|---|---|---|
| Cloud SQL for global strong consistency | Spanner | Cloud SQL is regional HA + async cross-region |
| Regional PD for cross-region DR | Snapshots to another region OR multi-region topology | Regional PD = 2 zones in 1 region |
| Liveness probe on startup endpoint | Startup Probe | Slow starts cause liveness fail loops |
| Session affinity solves HA | Externalize sessions | Affinity breaks on backend loss |
| SLA determines error budget | SLO determines error budget | SLO is the internal (tighter) target |
| Cloud Monitoring retention 30 days | Admin Activity logs = 400 days free; metrics = 6 weeks | Different retention by type |
| Single Cloud SQL replica = HA | HA requires primary + standby + auto failover | Replica ≠ standby |
| Bigtable strong consistency always | Strong consistency with single-cluster routing profile only | Multi-cluster = eventual |
| GCS multi-region = faster | Regional near compute = faster | Multi-region balances availability |
| Adding more replicas = better DR | Topology + regions matter more than replica count | DR needs regional failover plan |

---

## 🔑 Decision Frameworks

### 🎯 HA vs DR Picker

```
Failing zone only?
  → HA within region (multi-zone MIG, regional PD, Cloud SQL HA)

Failing entire region?
  → DR to another region (multi-region topology, cross-region replicas, backup-restore)

Both seamless?
  → Multi-region active-active (Spanner, multi-region GCS, Global LB)
```

### 🎯 DR Strategy Picker (from RPO/RTO)

```
What's your RTO / RPO?

RTO = hours, RPO = hours, cost-sensitive:
  → Backup & Restore (GCS backups, manual restore)

RTO = minutes–hours, RPO = minutes:
  → Pilot Light (minimal DR infra, scale up on failure)

RTO = minutes, RPO = seconds–minutes:
  → Warm Standby (scaled-down replica running)

RTO ≈ 0, RPO ≈ 0:
  → Hot Standby / Active-Active (Spanner, multi-region GCS, Global LB)
```

### 🎯 Database DR Picker

```
SQL, global, zero-RPO?
  → Spanner multi-region

SQL, regional HA + cross-region DR?
  → Cloud SQL HA + cross-region read replica (manual promote)

PostgreSQL high-perf, HTAP?
  → AlloyDB

Time-series, multi-region HA?
  → Bigtable multi-cluster replication

Mobile/web document DB?
  → Firestore Native (multi-region by default)

Analytics?
  → BigQuery Editions Plus with cross-region replication
```

### 🎯 Probe Picker

```
"Is it crashed/deadlocked?" → Liveness Probe (restart on fail)
"Is it ready for traffic?" → Readiness Probe (remove from service on fail)
"Has it finished starting up?" → Startup Probe (delay other probes)
```

### 🎯 LB Type for HA

```
Global low-latency, 99.99%+ SLA:
  → Global External Application LB + multi-region backends

Single region, HA across zones:
  → Regional External Application LB + Regional MIG

Internal app, cross-region:
  → Cross-region Internal Application LB (2024+)

TCP/UDP with source IP preservation:
  → External Passthrough Network LB
```

### 🎯 Logging/Monitoring Picker

```
Metrics, dashboards, alerts?
  → Cloud Monitoring

Log aggregation, search?
  → Cloud Logging

Long-term log archive (compliance)?
  → Log sink → GCS with locked retention

Real-time log forward to SIEM?
  → Log sink → Pub/Sub

SQL on logs?
  → Log sink → BigQuery

Distributed tracing?
  → Cloud Trace

Continuous CPU/memory profiling?
  → Cloud Profiler

Error aggregation?
  → Error Reporting

External URL availability?
  → Uptime Check (Cloud Monitoring)
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "SLI → SLO → SLA" order
- **SLI** = measurement (what you observe)
- **SLO** = internal target (tighter)
- **SLA** = external contract (loosest — room for error)

### 🧠 "RTO = Time, RPO = Pain"
- **RTO** = Recovery **T**ime (how long until back up)
- **RPO** = Recovery **P**ain (how much data you're willing to lose)
- **Smaller = expensive**

### 🧠 "LTES" (Golden Signals)
**L**atency, **T**raffic, **E**rrors, **S**aturation

### 🧠 "LiveRead StartUp"
- **Live**ness = alive? (restart if dead)
- **Read**iness = ready for traffic? (remove from service)
- **StartUp** = booted yet? (delay other probes)

### 🧠 "BPHW" (DR Ladder)
**B**ackup-restore → **P**ilot light → **W**arm standby → **H**ot standby
(From cheap + slow → expensive + instant)

### 🧠 "4-2-99"
Dedicated Interconnect: **4** connections across **2** metros = **99.99%**

### 🧠 "HAPS" (HA VPN 99.99%)
**HA** = **P**air of tunnels + BGP (**S**LA requirement)

### 🧠 "99s Downtime"
- 99% = 3.65 days/yr
- 99.9% = 8.76 hr/yr  
- 99.99% = 52 min/yr
- 99.999% = 5 min/yr

### 🧠 "ADSP" (Audit Logs)
**A**dmin Activity (on), **D**ata Access (OFF!), **S**ystem Event (on), **P**olicy Denied (on)

### 🧠 "Spanner wins Global"
For global RPO ≈ 0, Spanner's **synchronous** replication beats Cloud SQL's **asynchronous** cross-region replicas every time.

---

## ✅ Practice Checkpoint

### Q1. RTO/RPO near-zero global
**Scenario:** Global e-commerce. RTO < 5 min, RPO < 1 min. Must survive full regional outage with zero data loss.

**Options:**
- A) Cloud SQL regional HA + nightly backups
- B) Cloud SQL HA + cross-region read replica (manual promote)
- C) Spanner multi-region configuration (e.g., `nam-eur-asia1`)
- D) Firestore regional

✅ **Answer: C** — **Spanner multi-region** provides **synchronous replication** across regions, near-zero RPO, automatic failover. Cloud SQL cross-region replicas are asynchronous (data loss on failover).

---

### Q2. Error budget exhausted
**Scenario:** SLO = 99.9%. This month: 45 min downtime (budget = 43.2 min). Engineering wants to ship big feature.

**Options:**
- A) Ship feature
- B) Freeze risky releases, invest in reliability work per error budget policy
- C) Lower SLO to 99%
- D) Just add more alerts

✅ **Answer: B** — **Error budget policy**: when budget exhausted, prioritize reliability over features.

---

### Q3. Liveness vs Readiness
**Scenario:** GKE pod takes 60 seconds to initialize (load large model). After deploying, pods restart repeatedly and never become ready.

**Options:**
- A) Lower liveness `initialDelaySeconds`
- B) Add a Startup Probe to give the app time to boot; set liveness + readiness to start after startup completes
- C) Remove probes
- D) Bigger pod

✅ **Answer: B** — **Startup Probe** delays liveness/readiness until app is ready. Without it, liveness fires during startup → kills container → restart loop.

---

### Q4. Cloud SQL cross-region failover
**Scenario:** Production Cloud SQL in `us-central1`. Need DR in `us-east1`. RTO = 30 min acceptable, RPO = minutes.

**Options:**
- A) Another Cloud SQL HA in us-east1 with Dataflow sync
- B) Cloud SQL cross-region read replica in us-east1; manual promote on disaster
- C) Cloud SQL HA only in us-central1
- D) Spanner

✅ **Answer: B** — **Cross-region read replica** = async replication, manual promote for DR. Matches RTO/RPO.

---

### Q5. Uptime monitoring
**Scenario:** Need to alert if public API is unreachable from US, Europe, and Asia.

**Options:**
- A) Cloud Logging filter
- B) Cron job with curl on GCE
- C) Cloud Monitoring Uptime Check from multiple regions + alert policy
- D) Cloud Trace

✅ **Answer: C** — **Uptime Checks** support multi-region probes + alerting natively.

---

### Q6. Bigtable hotspotting
**Scenario:** IoT telemetry to Bigtable. Row key = `{timestamp}#{device_id}`. Hot tablets during data ingest; some writes throttled.

**Options:**
- A) Add more nodes
- B) Redesign row key as `{hash(device_id)}#{timestamp}#{device_id}` to distribute writes
- C) Use timestamp as suffix
- D) Switch to Firestore

✅ **Answer: B** — **Timestamp prefix = hotspot** (all writes hit latest tablet). Prefix with hash distributes writes.

---

### Q7. Cloud Run multi-region
**Scenario:** Cloud Run service; need 99.99% SLA and low latency for users in US, EU, APAC.

**Options:**
- A) Single-region Cloud Run
- B) Deploy Cloud Run in 3 regions; Global External Application LB with Serverless NEGs
- C) Regional LB per region
- D) GKE Autopilot

✅ **Answer: B** — **Multi-region Cloud Run + Global LB** gives low-latency + regional failover. Anycast routes users to nearest region.

---

### Q8. Dataflow out-of-order data
**Scenario:** Streaming pipeline aggregates hourly sales. Sales events sometimes arrive 15 min late due to network delays.

**Options:**
- A) Process time windows
- B) Event-time windows + watermarks + allowed lateness = 15 min + late-firing trigger
- C) Just drop late data
- D) Restart pipeline hourly

✅ **Answer: B** — **Event-time windows + watermarks + allowed lateness** handle late events correctly. Process-time windows give incorrect aggregates.

---

### Q9. Compute Engine DR
**Scenario:** 50 GCE VMs in `us-central1` running enterprise app. Need RTO = 2 hr, RPO = 6 hr, cross-region.

**Options:**
- A) Regional persistent disks
- B) Scheduled snapshots (every 6 hr) replicated to `us-east1`; playbook to restore
- C) Migrate to Cloud Run
- D) Multi-region Global LB with active-active

✅ **Answer: B** — **Snapshots every 6 hr to another region** match RPO. Restore = playbook within 2 hr. Regional PD = same region only.

---

### Q10. Externalize sessions
**Scenario:** App Engine app loses user sessions on instance restart. Growing user base.

**Options:**
- A) Sticky sessions via LB
- B) Store sessions in Memorystore for Redis or Firestore; stateless app tier
- C) Store sessions on instance disk
- D) Increase instance memory

✅ **Answer: B** — **Externalize sessions** so any instance serves any user. Sticky sessions break on failover/scale.

---

### Q11. GKE probe misconfiguration
**Scenario:** GKE app: liveness probe fails during heavy load (app slow to respond) → pod restarts → worse slowness → cascading failure.

**Options:**
- A) Remove liveness probe
- B) Increase liveness `timeoutSeconds` and `failureThreshold`; consider separating readiness (remove from traffic during load spikes) from liveness (only restart on real deadlock)
- C) Lower health check interval
- D) Add more replicas only

✅ **Answer: B** — **Tune liveness to only catch real deadlocks**; use readiness to shed traffic during load.

---

### Q12. GCS hotspotting
**Scenario:** Streaming log writes to GCS at 5,000 objects/sec. Names like `/logs/20260423-HHMMSS-seq.log`. Getting 429 errors.

**Options:**
- A) More buckets
- B) Hash-prefix object names: `/logs/a7f9/20260423-HHMMSS-seq.log`
- C) Larger objects
- D) Switch to BigQuery

✅ **Answer: B** — **GCS rate limit per prefix ~5K/sec**; hash-prefix spreads load across shards.

---

### Q13. Log retention for audit
**Scenario:** Compliance: keep all admin activity logs for 7 years, immutable.

**Options:**
- A) Default Cloud Logging (400 days)
- B) Log sink to GCS bucket with 7-year locked retention policy
- C) Log sink to BigQuery
- D) Manual export

✅ **Answer: B** — **GCS sink with locked retention** provides immutable long-term retention. BigQuery is for analysis, not compliance archive.

---

### Q14. Cloud Run cold starts
**Scenario:** Cloud Run API has P99 latency spikes from cold starts. Cost-sensitive but need consistent latency.

**Options:**
- A) Min-instances = 0 (current)
- B) Set min-instances = 1–2; CPU always allocated if background work needed
- C) Move to GKE
- D) Larger memory

✅ **Answer: B** — **Min-instances > 0** keeps warm instances, eliminates cold-start P99 spikes. Small cost for always-on baseline.

---

### Q15. Cloud SQL HA mechanism
**Scenario:** Cloud SQL with HA enabled. Primary in zone-a fails. What happens?

**Options:**
- A) Manual intervention required
- B) Standby in zone-b takes over automatically in ~60s; clients reconnect (same private IP); failed instance rebuilt as new standby
- C) Cross-region replica promotes
- D) New instance created from snapshot

✅ **Answer: B** — **Automatic zonal failover** via regional PD + health checks. ~60s downtime.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **Backup and DR Service GA** | Managed backup (formerly Actifio) |
| **Soft Delete for GCS (2024)** | 7-day default, protects against accidental deletion |
| **Turbo Replication** | Dual-region GCS RPO < 15 min |
| **Cloud SQL Enterprise Plus** | Faster DR, near-zero downtime maintenance |
| **AlloyDB DR improvements** | Cross-region replicas, better failover |
| **GKE Autopilot** | HA by default; managed probes |
| **Cloud Run Health Checks** | Built-in startup + liveness |
| **Cross-region Internal Application LB** | New answer for internal HA |
| **Spot VMs** replace Preemptible | No 24-hr cap |
| **Cloud Monitoring Synthetic Monitoring** | Multi-step uptime checks |
| **Active Assist / Recommender for Reliability** | Suggests HA/DR improvements |
| **Burn-rate alerts** | Modern alerting standard |
| **Personalized Service Health** | Per-customer outage notifications |
| **BigQuery cross-region DR (Editions Plus)** | Automated cross-region replication |
| **Cloud Logging Log Buckets** | More flexible retention/storage classes |

---

## 🎯 Final Domain 6 Checklist

Before exam day, you should be able to:

- [ ] Explain SLI/SLO/SLA in one sentence each
- [ ] Calculate error budget (100% − SLO)
- [ ] Describe HA vs DR without mixing them up
- [ ] Match RPO/RTO requirements to DR strategy (Backup/Pilot/Warm/Hot)
- [ ] Draw Cloud SQL HA architecture (primary + standby + regional PD)
- [ ] Know Spanner multi-region vs Cloud SQL cross-region replica trade-off
- [ ] Differentiate Liveness vs Readiness vs Startup probes
- [ ] Know when to use Regional MIG (always for prod!)
- [ ] Know MIG autohealing vs LB health check distinction
- [ ] Know hotspot prevention in Bigtable (row key design)
- [ ] Know hotspot prevention in GCS (object name prefix)
- [ ] Know Dataflow watermarks + triggers + allowed lateness
- [ ] Know 4 Golden Signals (LTES)
- [ ] Know burn-rate alerts > threshold alerts
- [ ] Know availability math (99.99% = 52 min/year)
- [ ] Know Cloud SQL PITR requires binary logging
- [ ] Know GCS retention policy + object holds + soft delete
- [ ] Know Log sinks (GCS, BQ, Pub/Sub, other project)
- [ ] Know Admin Activity = always on, Data Access = OFF by default
- [ ] Know compute maintenance policies (Migrate vs Terminate)
- [ ] Know MIG rolling update params (maxSurge, maxUnavailable)

> **Domain 6 is where architects prove they understand reality. The "happy path" is easy — designing for failure is where skill shows. Master HA ≠ DR, master probes, master RPO/RTO. You've got this.** 🛡️

---
