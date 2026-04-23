# 🏗️ Domain 2: Managing and Provisioning a Cloud Solution Infrastructure

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (2 of 6)**

**Exam Weight:** ~15%

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [Resource Hierarchy Management](#-resource-hierarchy-management)
3. [Infrastructure as Code (IaC)](#-infrastructure-as-code-iac)
4. [Compute Provisioning & Autoscaling](#-compute-provisioning--autoscaling)
5. [Storage Provisioning & Lifecycle](#-storage-provisioning--lifecycle)
6. [Networking Provisioning (Deep Dive)](#-networking-provisioning-deep-dive)
7. [Load Balancing (Complete Matrix)](#-load-balancing-complete-matrix)
8. [Hybrid Connectivity Setup](#-hybrid-connectivity-setup)
9. [DNS Configuration](#-dns-configuration)
10. [Quotas, Limits & Capacity Planning](#-quotas-limits--capacity-planning)
11. [Exam Traps & Tricks](#-exam-traps--tricks)
12. [Decision Frameworks](#-decision-frameworks)
13. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
14. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
15. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

This domain is about **how to implement** the architecture you designed in Domain 1. Expect questions on:

1. **IaC tooling** — which tool for what, and why Deployment Manager is dead
2. **Autoscaling configuration** — MIGs, HPA/VPA, Cluster Autoscaler, Cloud Run concurrency
3. **Network implementation** — subnets, firewall rules, routes, NAT, DNS
4. **Load balancer selection** — 8+ LB types, pick the right one
5. **Storage provisioning** — right class, lifecycle, retention, versioning
6. **Hybrid connectivity setup** — VPN vs Interconnect, SLA math
7. **Quota management** — when to request, what's adjustable
8. **Capacity planning** — committed use, reservations, sizing

This domain rewards **hands-on knowledge**. If you've never built it, study the CLI/Terraform patterns.

---

## 🌳 Resource Hierarchy Management

### 🧠 Operations at Each Level

| Level | What You Do Here | Tools |
|---|---|---|
| **Organization** | Org policies, org-wide IAM, central billing | gcloud, Console, Terraform |
| **Folder** | Group projects, apply env policies | gcloud resource-manager folders |
| **Project** | Enable APIs, set quotas, billing account link | gcloud projects |
| **Resource** | Create/destroy VMs, buckets, etc. | All tools |

### 🔑 Project Lifecycle

**Create:**
```bash
gcloud projects create my-project-123 --folder=FOLDER_ID --name="My Project"
gcloud beta billing projects link my-project-123 --billing-account=BILLING_ID
gcloud services enable compute.googleapis.com --project=my-project-123
```

**Shutdown:**
- Projects in **30-day grace period** after delete (can be undeleted)
- After 30 days → permanent deletion
- ⚠️ **Project IDs are never reusable** once deleted

### 🔑 Labels vs Tags (⚠️ commonly confused)

| | **Labels** | **Tags** |
|---|---|---|
| Purpose | Cost allocation, organization | Policy enforcement (firewall, IAM) |
| Format | key-value (string-string) | key-value with namespace |
| Scope | Resource | Inherited via hierarchy |
| Used by | Billing, monitoring, filtering | Firewall rules, IAM conditions |
| Max per resource | 64 | 50 |
| Example | `env=prod, team=payments` | `prod/env`, `allow-ssh` |

**Exam rule:** "Cost attribution" → **Labels**. "Firewall targeting" → **Tags** (or Network Tags for VMs).

### 🔑 Org Policy Constraints (high-frequency exam)

| Constraint | What It Does |
|---|---|
| `compute.disableSerialPortAccess` | Block SSH via serial console |
| `compute.requireShieldedVm` | Force Shielded VMs only |
| `compute.requireOsLogin` | Mandate OS Login (no SSH keys on metadata) |
| `iam.disableServiceAccountKeyCreation` | Block new SA keys |
| `iam.disableServiceAccountKeyUpload` | Block importing keys |
| `iam.allowedPolicyMemberDomains` | Restrict IAM to certain domains (prevents external sharing) |
| `gcp.resourceLocations` | Restrict regions for resource creation |
| `gcp.restrictCmekCryptoKeyProjects` | Force CMEK from approved projects |
| `storage.uniformBucketLevelAccess` | Enforce uniform bucket-level IAM (no ACLs) |
| `compute.vmExternalIpAccess` | Restrict external IPs on VMs |
| `sql.restrictPublicIp` | Block public IPs on Cloud SQL |

**Exam tip:** "Prevent any X across the whole org" → **Org Policy constraint at Organization node**.

---

## 🤖 Infrastructure as Code (IaC)

### 🧠 The IaC Landscape in 2026

| Tool | Type | When to Use |
|---|---|---|
| **Terraform** | Multi-cloud DSL | ⭐ Default answer for new IaC |
| **Infrastructure Manager** | Managed Terraform runner | GitOps, managed state, audit |
| **Config Connector** | K8s CRDs for GCP resources | GKE-centric orgs |
| **Config Controller** | Hosted Config Connector + Policy Controller | Managed GKE fleet mgmt |
| **Policy Controller** | OPA Gatekeeper on GKE | Enforce K8s policies |
| **Cloud Deploy** | CD pipelines | Progressive delivery |
| **gcloud CLI / Console** | Manual | Dev/learning |
| ~~Deployment Manager~~ | YAML/Jinja | ⚠️ **DEPRECATED** — never pick |

### 🔑 Terraform on GCP Essentials

**State management:**
- Store state in **GCS backend** with versioning + object lock
- Enable **state locking** via GCS native locking
- Use **separate state per environment**

**Service Account best practice:**
- Use **Workload Identity Federation** for CI/CD (no keys)
- Grant minimum IAM to the SA (only what's needed)

**Common patterns:**
- **Modules** for reusable components (networks, projects)
- **Workspaces** for environment separation
- **Variables** with validation
- **Remote state** data source for cross-stack references

### 🔑 Infrastructure Manager (new 2024+)

- **Managed Terraform runner** on GCP
- Pulls code from Git (Cloud Source, GitHub, GitLab)
- Runs `terraform apply` for you
- State stored on GCP
- Integrated with **Cloud Build** for CI
- Replaces Deployment Manager

**Exam trigger:** "Managed IaC with Google support" → **Infrastructure Manager**

### 🔑 Config Connector (GKE)

- Manage GCP resources via **Kubernetes YAML**
- Useful when your team already lives in K8s
- CRDs like `PubSubTopic`, `SQLInstance`, `StorageBucket`
- Part of **GKE Enterprise**

```yaml
apiVersion: storage.cnrm.cloud.google.com/v1beta1
kind: StorageBucket
metadata:
  name: my-bucket
spec:
  location: US
```

### 🔑 Cloud Foundation Toolkit (CFT)

- **Blueprints** and **modules** for common patterns
- Available for Terraform
- Good for **landing zones**, enterprise onboarding
- Not a separate product — just a library of modules

---

## 💻 Compute Provisioning & Autoscaling

### 🧠 Managed Instance Groups (MIGs) Deep Dive

| Feature | Zonal MIG | Regional MIG |
|---|---|---|
| Failure domain | Single zone | Multiple zones (3+) |
| HA | ❌ | ✅ |
| Use | Dev/test | Production |

**Always use Regional MIGs for production.** Exam trap: zonal MIG for "HA" scenario is wrong.

### 🔑 MIG Autoscaling Signals

| Signal | Best For |
|---|---|
| **CPU utilization** | General-purpose compute |
| **Load balancer utilization** | Web traffic |
| **HTTP load balancing** | LB-fronted apps |
| **Cloud Monitoring custom metric** | Custom logic (e.g., queue depth) |
| **Pub/Sub queue backlog** | Worker pools |
| **Schedule-based** | Predictable daily patterns |

**Multiple signals:** MIG picks the highest recommendation among signals.

### 🔑 MIG Rolling Updates

| Strategy | How |
|---|---|
| **Proactive** | Replace all instances when template changes |
| **Opportunistic** | Only replace when instance is recreated for other reasons |
| **Rolling** | Update N at a time (maxSurge, maxUnavailable) |
| **Canary** | Update small subset first; observe; promote |

### 🔑 Stateful MIGs

- Preserve per-instance disks and metadata across updates
- Use for: DBs, apps that need identity (though usually K8s StatefulSets are better)
- ⚠️ **Prefer GKE StatefulSets** for new workloads — exam rarely picks stateful MIGs

### 🔑 GKE Autoscaling (HPA / VPA / CA / NAP)

| Tool | Scales | Signal |
|---|---|---|
| **HPA (Horizontal Pod Autoscaler)** | Pod count | CPU, memory, custom metrics |
| **VPA (Vertical Pod Autoscaler)** | Pod CPU/mem requests | Historical usage |
| **Cluster Autoscaler (CA)** | Node count | Unschedulable pods |
| **Node Auto-Provisioning (NAP)** | Node pool creation | Workload requirements |

**Exam trap:** ⚠️ Don't enable **HPA + VPA on the same metric** (both scale CPU = conflict). OK to use HPA on CPU + VPA on memory, or HPA on custom metrics + VPA on everything.

**GKE Autopilot:** All of this is built-in; you don't configure CA/NAP.

### 🔑 Cloud Run Scaling

| Setting | Purpose |
|---|---|
| **Min instances** | Keep warm instances (no cold start) |
| **Max instances** | Cap cost, prevent runaway |
| **Concurrency** | Requests per instance (1–1000) |
| **CPU allocation** | "During requests only" (cheaper) vs "Always allocated" (background work) |

**Signals Cloud Run uses to scale:**
- Request concurrency
- CPU utilization (if always-allocated)

### 🔑 Spot VMs (new name for preemptible)

- **Up to 91% discount**
- **No 24-hour maximum lifetime** (old preemptible cap)
- Can be preempted any time (30-sec notice)
- Use with: **MIG Spot**, **GKE Spot node pools**, **Batch**, **Dataflow**

**Exam rule:** "Fault-tolerant + cheap" → **Spot VMs**.

---

## 💾 Storage Provisioning & Lifecycle

### 🧠 Cloud Storage Provisioning

**Bucket creation decisions:**
1. **Location type:** Region / Dual-region / Multi-region
2. **Storage class:** Standard / Nearline / Coldline / Archive
3. **Access control:** Uniform bucket-level access (recommended) vs fine-grained ACLs
4. **Encryption:** Google-managed / CMEK / CSEK
5. **Versioning:** On/off (keeps old object versions)
6. **Retention policy:** Object lock for compliance
7. **Public access prevention:** Enforced at bucket

### 🔑 Lifecycle Rules (exam favorite)

**Typical rules:**
- After 30 days → move to Nearline
- After 90 days → move to Coldline
- After 365 days → move to Archive
- After 7 years → delete

**Other actions:**
- Delete noncurrent versions (cleanup old versions)
- Abort incomplete multipart uploads

**Example JSON:**
```json
{
  "rule": [
    {"action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
     "condition": {"age": 30}},
    {"action": {"type": "Delete"},
     "condition": {"age": 365}}
  ]
}
```

### 🔑 Retention Policy vs Object Hold

| Feature | Retention Policy | Temporary/Event-based Hold |
|---|---|---|
| Scope | Bucket-wide | Per object |
| Duration | Time-based (N days) | Until hold removed |
| Use case | "Keep all invoices 7 yrs" | "Keep evidence until case closes" |
| Locking | Can lock retention policy permanently | Hold itself |

**Exam tip:** "WORM / immutable for compliance" = **locked retention policy** (cannot be reduced even by admin).

### 🔑 Persistent Disk Provisioning

| Disk Type | IOPS per GB | Throughput per GB | Best For |
|---|---|---|---|
| **PD Standard** | 0.75 | Low | Boot, cold data |
| **PD Balanced** | 6 | Medium | General |
| **PD SSD** | 30 | High | DBs |
| **PD Extreme** | Provisioned | High | Demanding DBs |
| **Hyperdisk Balanced** | Configurable | Configurable | Modern general |
| **Hyperdisk Throughput** | — | Very high | Hadoop, Kafka |
| **Hyperdisk Extreme** | Highest | Highest | Top-tier DBs |
| **Hyperdisk ML** | Multi-attach RO | High | AI training |

**Regional PD:** Synchronously replicated across **2 zones** in a region. For zonal HA of stateful workloads.

### 🔑 Filestore Tiers

| Tier | Performance | Use |
|---|---|---|
| **Basic HDD** | Low | General file share |
| **Basic SSD** | Better | DB backups |
| **Zonal** | High | GKE workloads (zonal) |
| **Enterprise** | Highest + regional HA | Mission-critical |

---

## 🌐 Networking Provisioning (Deep Dive)

### 🧠 VPC Design

| Decision | Options | Best Practice |
|---|---|---|
| **VPC mode** | Auto / Custom | **Custom** (you control CIDR) |
| **Subnet CIDR** | /29 to /8 | Plan for growth; use /20 or /22 |
| **Primary vs Secondary ranges** | Secondary = for GKE pods/services | Alias IPs |
| **Subnet regional scope** | Regional | Remember: VPC = global, subnet = regional |

### 🔑 Firewall Rules

**Structure:**
- **Direction:** Ingress or Egress
- **Priority:** 0 (highest) to 65535 (lowest)
- **Action:** Allow or Deny
- **Source/Target:** IP ranges, network tags, service accounts
- **Protocol/Port:** TCP/UDP/ICMP etc.

**Implicit rules (always present):**
- ✅ **Ingress: Deny all** (priority 65535)
- ✅ **Egress: Allow all** (priority 65535)

**Default network rules (created with default VPC):**
- Allow SSH (22), RDP (3389), ICMP from anywhere
- ⚠️ **Delete default network in production**

### 🔑 Firewall Policies vs VPC Firewall Rules

| | VPC Firewall Rules | Hierarchical Firewall Policies |
|---|---|---|
| Scope | One VPC | Org or Folder |
| Inheritance | No | Yes (downward) |
| Evaluation order | By priority | Org → Folder → VPC |
| Use | Application-specific | Enterprise-wide guardrails |

**Exam rule:** "Enforce deny rule across all VPCs in company" → **Hierarchical Firewall Policy at Org**.

### 🔑 Network Tags vs Service Accounts as Targets

| Target Type | Pros | Cons |
|---|---|---|
| **Network Tags** | Simple, flexible | Anyone with instance.create permission can apply tags |
| **Service Accounts** | Harder to spoof, ties to IAM | Slightly less flexible |

**Exam rule:** For secure firewall targeting → **Service Accounts**.

### 🔑 Cloud NAT

- **Outbound internet access for private VMs** (no external IP)
- Regional resource
- Automatic port allocation (or manual)
- Works with Private Google Access for Google APIs

**Exam tip:** "Private VMs need internet access" → **Cloud NAT**. (Not a proxy, not a NAT gateway VM.)

### 🔑 Private Service Access vs Private Service Connect (PSC)

| | Private Service Access (legacy) | Private Service Connect (PSC) |
|---|---|---|
| Purpose | Reach managed services (Cloud SQL, Memorystore) | Consume any service (Google, SaaS, own) |
| Setup | VPC peering to Google-managed producer | PSC endpoint in your VPC |
| Flexibility | Limited | Much more |
| Use for Google APIs | No (use PGA) | ✅ Yes (PSC for Google APIs) |

**2025+ exam default:** PSC over private service access where possible.

### 🔑 Cloud DNS

| Zone Type | Purpose |
|---|---|
| **Public zone** | Internet-facing DNS |
| **Private zone** | VPC-internal DNS |
| **Forwarding zone** | Forward to on-prem DNS |
| **Peering zone** | Cross-project/VPC DNS |
| **Service directory zone** | Managed service discovery |

- **DNSSEC:** Enable for public zones
- **Cloud DNS SLA:** **100%** (unique!)

---

## ⚖️ Load Balancing (Complete Matrix)

### 🧠 The Complete LB Picker

| Load Balancer | Scope | Protocol | Proxy/Passthrough | Public/Internal |
|---|---|---|---|---|
| **Global External Application LB** | Global | HTTP(S) | Proxy | External |
| **Regional External Application LB** | Regional | HTTP(S) | Proxy | External |
| **Cross-region Internal Application LB** | Global | HTTP(S) | Proxy | Internal |
| **Regional Internal Application LB** | Regional | HTTP(S) | Proxy | Internal |
| **Global External Proxy Network LB** | Global | TCP/SSL | Proxy | External |
| **Regional External Proxy Network LB** | Regional | TCP | Proxy | External |
| **Regional Internal Proxy Network LB** | Regional | TCP | Proxy | Internal |
| **External Passthrough Network LB** | Regional | TCP/UDP | Passthrough | External |
| **Internal Passthrough Network LB** | Regional | TCP/UDP | Passthrough | Internal |

### 🔑 Key Differences

| Feature | Proxy LB | Passthrough LB |
|---|---|---|
| Source IP preserved? | ❌ No | ✅ Yes |
| SSL termination | ✅ Yes | ❌ No |
| Layer | L7 (HTTP) or L4 (TCP/SSL) | L4 (TCP/UDP) |
| URL maps, host rules | ✅ Yes (L7) | ❌ No |
| WAF (Cloud Armor) | ✅ Yes | ⚠️ Limited |
| Performance | Slightly more latency | Near-wire speed |

### 🔑 LB Decision Framework

**Step 1: Internal or external?**
- External → internet-facing
- Internal → within VPC

**Step 2: L7 or L4?**
- HTTP(S), URL routing, WAF → **Application LB**
- TCP/UDP, low-latency, preserve IP → **Network LB (passthrough)**
- TCP/SSL with proxy features → **Proxy Network LB**

**Step 3: Global or regional?**
- Global → **Global Application/Proxy LB** (single anycast IP)
- Regional → **Regional Application LB** (simpler, cheaper)

### 🔑 Backend Types

| Backend | Works With |
|---|---|
| **Instance Group (MIG/UG)** | All LBs |
| **NEG (Network Endpoint Group)** | Zonal, Serverless, Internet, Hybrid |
| **Serverless NEG** | Cloud Run, App Engine, Cloud Functions |
| **Internet NEG** | External backends (on-prem, other cloud) |
| **Hybrid NEG** | On-prem via Interconnect/VPN |
| **PSC NEG** | Private Service Connect backends |
| **GKE Services** | Via standalone NEGs |

### 🔑 Global External Application LB Features

- **Single anycast IP** globally
- **Cloud CDN** integration
- **Cloud Armor** (WAF, DDoS)
- **Identity-Aware Proxy (IAP)**
- **URL maps, host rules, path rules**
- **SSL/TLS termination** with managed certs
- **HTTP/2, HTTP/3 (QUIC)**
- **Global & regional backend services**
- **Backend buckets** (GCS)

### 🔑 Cloud CDN Integration

**Cache modes:**
- **Cache all static** — default for images/JS/CSS
- **Use origin headers** — respect Cache-Control
- **Force cache all** — cache everything (even dynamic)

**Features:**
- Edge POPs globally
- Custom cache keys
- Signed URLs / signed cookies for private content
- Cache invalidation API

---

## 🔗 Hybrid Connectivity Setup

### 🧠 Complete Hybrid Matrix

| Option | Throughput | SLA | Latency | Cost | When |
|---|---|---|---|---|---|
| **HA VPN (1 tunnel)** | ~3 Gbps | 99.9% | Internet | $ | Dev, small |
| **HA VPN (2 tunnels, redundant)** | ~3 Gbps each | **99.99%** | Internet | $ | Small prod |
| **Partner Interconnect (50M–10G)** | 50M–10G | 99.9% | Low | $$ | No colo, medium BW |
| **Partner Interconnect (redundant)** | 50M–10G | **99.99%** | Low | $$ | No colo, prod |
| **Dedicated Interconnect (1 × 10G)** | 10G | 99.9% | Lowest | $$$ | Colo, high BW |
| **Dedicated Interconnect (4 × 10G, 2 metros)** | 40G+ | **99.99%** | Lowest | $$$$ | Enterprise prod |
| **Cross-Cloud Interconnect** | 10G/100G | 99.9–99.99% | Low | $$$ | Multi-cloud |

### 🔑 HA VPN Setup

Requirements for **99.99%**:
- **2 HA VPN gateways** OR
- **1 HA VPN gateway with 2 interfaces + 2 tunnels**
- **BGP dynamic routing** (not static)
- Peer VPN on on-prem side must support it

**IKE/IPsec:**
- IKEv2 preferred
- MTU consideration: 1460 typical for VPN

### 🔑 Dedicated Interconnect Setup

- **10 Gbps or 100 Gbps** circuits
- **Physical cross-connect** in a Google **colocation facility**
- **VLAN attachments** to connect to VPC
- **99.9% SLA**: 1 connection
- **99.99% SLA**: 4 connections across **2 metros** or **2 edge availability domains**

### 🔑 Partner Interconnect Setup

- For customers **not in colo**
- Use a **service provider** (Equinix, Megaport, etc.)
- **50 Mbps to 50 Gbps**
- **99.9% SLA**: single connection
- **99.99% SLA**: redundant topology via 2 providers

### 🔑 Cross-Cloud Interconnect

- **Direct private link** GCP ↔ AWS / Azure / OCI
- 10 Gbps or 100 Gbps
- **Replaces VPN for multi-cloud** data transfers
- Launched 2023; key 2025+ answer

---

## 🌍 DNS Configuration

### 🧠 Cloud DNS Architecture

```
Internet → Public Zone (example.com) → Public IPs / GCLB
VPC → Private Zone (internal.example.com) → Internal IPs / ILB
VPC → Forwarding Zone (corp.example.com) → On-prem DNS server
```

### 🔑 DNS Best Practices

- **Split-horizon DNS:** Same name resolves differently from public vs private
- **DNS Peering:** Multiple VPCs share private zones without duplication
- **Cloud DNS FQDN lookups** work with **Shared VPC**
- **Response Policies:** Override DNS answers (firewall/blocking)

### 🔑 On-prem DNS Integration

**Pattern:** On-prem DNS ↔ Cloud DNS via **forwarding zones** over Interconnect/VPN.

Requirements:
- Enable **inbound DNS policy** to allow on-prem → Cloud DNS
- Enable **outbound forwarding zone** for Cloud → on-prem

---

## 📊 Quotas, Limits & Capacity Planning

### 🧠 Quotas vs Limits

| | Quota | Limit |
|---|---|---|
| Adjustable? | ✅ Request increase | ❌ Hard cap |
| Example | "24 CPUs per region per project" | "Max 5 firewall rules per VM" |
| Scope | Per project per region | Product-level |

**Exam tip:** "Running out of CPUs in a region" → **request quota increase**.

### 🔑 Committed Use Discounts (CUDs)

| Type | How |
|---|---|
| **Resource-based** | Commit to specific vCPUs / RAM in a region for 1 or 3 yr |
| **Spend-based (Flexible)** | Commit to $X/hour in a region for 1 or 3 yr across VM families |

**Savings:**
- 1-year: ~20–40%
- 3-year: ~40–70%

**Flexible CUDs (2023+):** Span VM families and size changes — better if workload evolves.

### 🔑 Reservations

| | Use |
|---|---|
| **Specific reservation** | Guarantee capacity for named workload |
| **Any reservation** | Pool any matching VM can use |
| **Shared reservation** | Across projects |

**Pair CUDs + Reservations** for guaranteed capacity at discounted price.

### 🔑 Capacity Planning Principles

- **Forecast growth** from monitoring trends
- **Alert before hitting quota** (set at 80%)
- **Multi-region** for burst capacity
- **Right-size** via Recommender

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 15 Traps in Domain 2

1. **Deployment Manager** → ⚠️ deprecated, never pick
2. **Zonal MIG for HA** → wrong; use Regional MIG
3. **Preemptible VMs** → terminology gone; say **Spot VMs**
4. **Default VPC in production** → wrong; use Custom mode + delete default
5. **Auto-mode VPC** → not best practice; use custom
6. **Firewall rules via Network Tags in secure env** → use **Service Account targets** instead
7. **HPA + VPA on same CPU metric** → conflict
8. **HA VPN single tunnel for 99.99%** → wrong; need 2 tunnels + BGP
9. **Single Dedicated IC for 99.99%** → wrong; need 4 × 10G across 2 metros
10. **VPC peering for mesh topology** → non-transitive; use NCC
11. **Passthrough LB for L7 routing** → impossible; passthrough is L4 only
12. **External IP on Cloud SQL** → use **Private Service Access / PSC**
13. **Regional LB for global latency** → wrong; use Global LB
14. **Public zone for internal DNS** → use Private Zone
15. **Persistent Disk regional ≠ multi-region** → it's 2-zone sync

### ⚠️ "Sounds Right but Isn't"

| Seems Right | Actually Right | Why |
|---|---|---|
| Deployment Manager (appears in old materials) | Terraform / Infrastructure Manager | DM deprecated |
| Container Registry | Artifact Registry | CR deprecated |
| Cloud VPN Classic | HA VPN | Classic deprecated |
| Auto-mode VPC (easier) | Custom-mode VPC | Auto creates unneeded subnets everywhere |
| VPN single tunnel (working) | HA VPN 2 tunnels (99.99%) | Single tunnel only 99.9% |
| Preemptible VMs | Spot VMs | Preemptible = old name |
| Regional MIG in "one zone for cost" | Regional MIG multi-zone | Defeats purpose |
| Instance Template not versioned | Versioned Instance Templates | For rolling updates |

---

## 🔑 Decision Frameworks

### 🎯 LB Picker (from scenario keywords)

```
Is it HTTP(S)?
  ├── Yes:
  │     ├── Global users? → Global External Application LB
  │     ├── Single region users? → Regional External Application LB
  │     └── Internal (within VPC)? → Internal Application LB (regional or cross-region)
  └── No (TCP/UDP):
        ├── Need to preserve source IP? → Passthrough Network LB
        ├── Need TLS termination at LB? → Proxy Network LB
        └── Internal TCP only? → Internal Passthrough Network LB
```

### 🎯 Autoscaling Picker

```
Is it containers?
  ├── Cloud Run? → Concurrency + min/max instances
  ├── GKE? → HPA (default) + CA (Autopilot bakes in NAP)
  └── GKE with VM-level needs? → Cluster Autoscaler

Is it VMs?
  └── MIG autoscaler:
        ├── Web traffic? → LB utilization
        ├── Workers? → Pub/Sub queue depth (custom metric)
        ├── Known pattern? → Schedule-based
        └── General? → CPU utilization
```

### 🎯 IaC Picker

```
Using Kubernetes heavily?
  ├── Yes → Config Connector (plus Terraform for infra)
  └── No:
        ├── Want managed + GCP-native? → Infrastructure Manager
        └── Multi-cloud / industry-standard? → Terraform
```

### 🎯 Hybrid Connectivity Picker

```
Bandwidth needs?
  ├── ≤ 3 Gbps, want quick? → HA VPN (2 tunnels for 99.99%)
  ├── 1–10 Gbps, no colo? → Partner Interconnect
  ├── 10–100 Gbps, have colo? → Dedicated Interconnect
  └── Multi-cloud direct link? → Cross-Cloud Interconnect
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "TIP" (IaC order of preference)
**T**erraform → **I**nfrastructure Manager → Config Connector (**P**orts K8s)

### 🧠 "HAPS" (HA VPN setup for 99.99%)
**H**igh **A**vailability = **P**air of tunnels + BGP (**S**LA hit)

### 🧠 "4-2-99" (Dedicated Interconnect 99.99%)
**4** connections across **2** metros = **99.99%**

### 🧠 "SCHeMA" (MIG autoscaling signals)
**S**chedule, **C**PU, **H**TTP LB, **M**onitoring custom metric, **A**dvanced (Pub/Sub)

### 🧠 "PAPPI" (LB type order by scope)
**P**assthrough, **A**pplication, **P**roxy... **P**riority by **I**nternal/external

### 🧠 "LIFT" (Lifecycle rule pattern)
**L**ifecycle → **I**nitial: Standard → **F**ollow: Nearline → Coldline → **T**erminate: Archive/Delete

### 🧠 "DELTA" (Org Policy vs IAM)
**D**eny **E**verything? Use Org Policy. **L**imit access to someone? Use IAM. **T**argeted **A**ction? Pick the right layer.

---

## ✅ Practice Checkpoint

### Q1. IaC choice
**Scenario:** Enterprise is standardizing IaC across AWS and GCP. 50 engineers. Need state locking, modules, CI/CD integration.

**Options:**
- A) Deployment Manager for GCP, CloudFormation for AWS
- B) Terraform for both, stored in Git with remote state on GCS
- C) gcloud CLI scripts
- D) Config Connector

✅ **Answer: B** — **Terraform** is the multi-cloud standard. DM is deprecated. Config Connector is K8s-only. Scripts don't scale.

---

### Q2. Firewall targeting securely
**Scenario:** Multiple teams share a VPC. Team A's VMs should accept SSH only from Team A's bastion. Minimize risk of misconfiguration.

**Options:**
- A) Network tag `team-a` on both VMs and bastion
- B) Service account as target + source (Team A's SA)
- C) Deny all SSH
- D) IAM role on the VM

✅ **Answer: B** — **Service account targets** are harder to spoof; anyone can add a network tag. Best practice for secure firewall rules.

---

### Q3. Autoscaling misconfiguration
**Scenario:** GKE cluster has HPA on CPU 70% and VPA on CPU+memory. Pods are constantly restarting.

**Options:**
- A) Increase HPA threshold
- B) Disable VPA on CPU (keep on memory) or switch HPA to custom metric
- C) Use Cluster Autoscaler
- D) Move to Autopilot

✅ **Answer: B** — **HPA + VPA on same CPU metric conflict** (both try to scale on CPU). Decouple them.

---

### Q4. 99.99% hybrid SLA
**Scenario:** Must deliver 99.99% SLA hybrid connectivity at 5 Gbps.

**Options:**
- A) Single HA VPN tunnel
- B) Two HA VPN tunnels across 2 gateways with BGP
- C) Single Dedicated Interconnect 10G
- D) Partner Interconnect 10G single circuit

✅ **Answer: B** — 5 Gbps is within HA VPN range (2 tunnels × 3 Gbps). **99.99% requires 2 tunnels with BGP**. Single Dedicated IC = 99.9% only.

---

### Q5. LB for preserving source IP
**Scenario:** Gaming UDP server needs to see client source IPs for anti-cheat geo-IP logic. Regional.

**Options:**
- A) Regional External Application LB
- B) External Passthrough Network LB
- C) Global External Proxy Network LB
- D) Internal Passthrough Network LB

✅ **Answer: B** — Passthrough preserves source IP; Application/Proxy LBs don't. External because internet-facing.

---

### Q6. Cold data archival
**Scenario:** 100 TB of medical records accessed once every 2-3 years but must be kept 20 years.

**Options:**
- A) Standard + lifecycle delete after 20 yrs
- B) Nearline
- C) Archive + retention policy of 20 yrs (locked)
- D) Coldline

✅ **Answer: C** — **Archive** for yearly+ access; **locked retention policy** for immutability. Standard costs too much.

---

### Q7. Private VM needs Google API access
**Scenario:** VMs without external IPs need to read/write to BigQuery and Cloud Storage.

**Options:**
- A) Create external IP
- B) Cloud NAT
- C) Private Google Access (on subnet)
- D) VPN

✅ **Answer: C** — **Private Google Access** lets private VMs reach Google APIs. Cloud NAT would also work but adds egress cost; PGA is free and specifically for Google APIs.

---

### Q8. Preventing SA key creation
**Scenario:** Security policy requires blocking all new service account key creation across the entire org.

**Options:**
- A) IAM role removal per user
- B) Org Policy constraint `iam.disableServiceAccountKeyCreation` at Org node
- C) Audit logs + detect-and-revoke automation
- D) VPC Service Controls

✅ **Answer: B** — **Org Policy at Org** level enforces and inherits everywhere.

---

### Q9. Global LB with path routing
**Scenario:** Global API gateway. `/api/*` → service A, `/images/*` → GCS bucket, `/stream/*` → Cloud Run. Need Cloud CDN for images.

**Options:**
- A) Regional External Application LB
- B) Global External Application LB with URL map + backend bucket for images + serverless NEG for Cloud Run
- C) Passthrough Network LB
- D) Internal Application LB

✅ **Answer: B** — Global Application LB does URL maps + backend buckets + serverless NEGs + CDN. Perfect fit.

---

### Q10. MIG updating
**Scenario:** Deploy new version of web app to MIG with 100 instances. Zero downtime, gradual rollout, ability to rollback fast.

**Options:**
- A) Recreate all instances at once
- B) Rolling update with maxSurge=10, maxUnavailable=0
- C) Delete old MIG, create new MIG
- D) In-place update

✅ **Answer: B** — **Rolling update** with maxSurge (extra capacity) and maxUnavailable=0 keeps all traffic served. Canary → Rolling is the typical pattern.

---

### Q11. Multi-region storage for images
**Scenario:** Image CDN backend. Objects accessed globally; latency critical; survive regional outage.

**Options:**
- A) Regional bucket in us-central1
- B) Multi-region bucket (`us`) + Cloud CDN
- C) Archive class
- D) Regional bucket + cross-region replication script

✅ **Answer: B** — **Multi-region bucket** (auto-replicated) + **Cloud CDN** (edge cache) = optimal for global image delivery.

---

### Q12. Quota hit
**Scenario:** Batch job needs 1000 vCPUs in us-east1 but default quota is 24. Cluster launch fails.

**Options:**
- A) Switch to another region
- B) File a quota increase request via Console or gcloud with business justification
- C) Create more projects
- D) Use Spot VMs (doesn't change quota)

✅ **Answer: B** — **Request quota increase**. Google typically approves if usage justified.

---

### Q13. Hierarchical firewall
**Scenario:** Policy: "No VM in the entire org can receive SSH from the internet." Want to enforce even if project admin creates allow rules.

**Options:**
- A) VPC firewall rules in each project
- B) Org Policy
- C) Hierarchical Firewall Policy at Org with Deny rule (higher priority than VPC rules)
- D) Cloud Armor

✅ **Answer: C** — **Hierarchical Firewall Policies** evaluated before VPC rules; Org-level Deny beats project Allow.

---

### Q14. Cloud SQL private connection
**Scenario:** Cloud SQL must not have a public IP. Apps in VPC need to connect.

**Options:**
- A) VPN
- B) Private Service Access (VPC peering with Google-managed producer) or Private Service Connect
- C) External IP with authorized networks
- D) Cloud Interconnect

✅ **Answer: B** — **Private Service Access** (older) or **PSC** (2024+ preferred) for private connectivity to managed services like Cloud SQL.

---

### Q15. Labels vs tags
**Scenario:** Need to attribute GCS bucket costs to different business units for the finance team's chargeback report.

**Options:**
- A) Firewall rules
- B) Network Tags
- C) Labels (e.g., `bu=finance`, `app=billing`)
- D) Resource Manager Tags (policy tags)

✅ **Answer: C** — **Labels** are for cost attribution; show up in billing exports. Tags are for policy.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **Deployment Manager deprecated** | Use Terraform / Infrastructure Manager |
| **Classic VPN deprecated** | Use HA VPN |
| **Preemptible VMs renamed Spot VMs** | No 24hr cap, deeper discount |
| **Hyperdisk** is preferred block storage | New premium tier above PD |
| **Cross-region Internal Application LB** | 2024+ cross-region internal traffic |
| **Flexible Committed Use Discounts** | Span VM families, size changes |
| **Cloud DNS routing policies** (geo, WRR) | New routing answers |
| **Network Connectivity Center** with VPC spokes | Replaces complex VPC mesh |
| **Private Service Connect** | Default answer for private consumption |
| **Cloud NGFW Enterprise** | New advanced firewall with IDS/IPS |
| **Infrastructure Manager** | Managed Terraform |
| **Container Registry deprecated** | Artifact Registry only |
| **Cloud WAN** (enterprise) | Global private backbone |

---

## 🎯 Final Domain 2 Checklist

- [ ] Know the 9 LB types and when to pick each
- [ ] Know why Deployment Manager is dead
- [ ] Know HA VPN requirements for 99.99%
- [ ] Know Dedicated Interconnect requirements for 99.99%
- [ ] Know all 4 GCS storage classes + min durations + lifecycle
- [ ] Know MIG autoscaling signals
- [ ] Know HPA vs VPA vs CA vs NAP in GKE
- [ ] Know firewall rules: priorities, implicit rules, hierarchical
- [ ] Know labels vs tags
- [ ] Know Terraform patterns (remote state, WIF for CI/CD)
- [ ] Know quota increase process
- [ ] Know org policy constraints

> **Domain 2 is all about execution. If you've never built it, drill the CLI / Terraform.** ⚡
