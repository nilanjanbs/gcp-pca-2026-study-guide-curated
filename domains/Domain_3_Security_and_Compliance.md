# 🔒 Domain 3: Designing for Security and Compliance

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (3 of 6)**

**Exam Weight:** ~18%

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [Identity & Access Management (IAM)](#-identity--access-management-iam)
3. [Service Accounts & Workload Identity](#-service-accounts--workload-identity)
4. [Encryption: KMS, CMEK, CSEK, EKM, HSM](#-encryption-kms-cmek-csek-ekm-hsm)
5. [Data Protection (Sensitive Data Protection)](#-data-protection-sensitive-data-protection)
6. [Network Security (Defense in Depth)](#-network-security-defense-in-depth)
7. [VPC Service Controls (Deep Dive)](#-vpc-service-controls-deep-dive)
8. [Cloud Armor & DDoS Protection](#-cloud-armor--ddos-protection)
9. [Identity-Aware Proxy (IAP)](#-identity-aware-proxy-iap)
10. [Secrets Management](#-secrets-management)
11. [Compliance Frameworks & Assured Workloads](#-compliance-frameworks--assured-workloads)
12. [Audit Logging & Access Transparency](#-audit-logging--access-transparency)
13. [Exam Traps & Tricks](#-exam-traps--tricks)
14. [Decision Frameworks](#-decision-frameworks)
15. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
16. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
17. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

Security is an **18% chunk** — and the PCA is unforgiving here. Wrong security answers are rarely "close" — they're outright wrong. Expect scenarios like:

- "Encrypt data so Google can never access it" → **EKM**, not CMEK
- "Prevent data exfiltration even with stolen creds" → **VPC Service Controls**, not IAM
- "External CI/CD authenticate without keys" → **Workload Identity Federation**
- "Replace VPN for internal web apps" → **IAP**
- "Meet HIPAA/FedRAMP with guardrails" → **Assured Workloads**

The exam tests these sub-skills:

1. **IAM design** — least privilege, custom roles, conditions, deny policies
2. **Service account hygiene** — impersonation, WIF, avoiding keys
3. **Encryption strategy** — picking between Google-managed, CMEK, CSEK, EKM, HSM
4. **Network security layering** — firewall + VPC-SC + Cloud Armor + IAP
5. **Data classification** — DLP/Sensitive Data Protection, Data Catalog tagging
6. **Compliance** — matching frameworks to GCP products (Assured Workloads, Access Transparency)
7. **Audit & forensics** — Cloud Audit Logs, log exports
8. **Secrets handling** — Secret Manager vs env vars vs config files

---

## 🔐 Identity & Access Management (IAM)

### 🧠 IAM Model Fundamentals

**Principal → Role → Resource**

| Component | Examples |
|---|---|
| **Principal** | User, Group, Domain, Service Account, allAuthenticatedUsers, allUsers |
| **Role** | Basic (Owner/Editor/Viewer), Predefined, Custom |
| **Resource** | Project, Folder, Org, Bucket, Specific VM, etc. |

### 🔑 The 3 Role Types

| Type | Description | Exam Rule |
|---|---|---|
| **Basic** (legacy "Primitive") | Owner, Editor, Viewer — broad, project-wide | ⚠️ **Never for prod** — too broad |
| **Predefined** | Curated by Google (e.g., `storage.objectViewer`) | ✅ Default choice |
| **Custom** | You create with specific permissions | For least-privilege when no predefined fits |

### 🔑 IAM Policy Inheritance

```
Organization  ← IAM policy A
    └── Folder  ← IAM policy B
           └── Project  ← IAM policy C
                  └── Resource  ← IAM policy D
```

**Effective policy = UNION** of all policies from Org → Folder → Project → Resource.

⚠️ **You cannot "override to less" via IAM allow policies** (allow is additive). Use **IAM Deny Policies** (2023+) for explicit blocks.

### 🔑 IAM Deny Policies (new)

- Evaluated **BEFORE** allow policies
- Explicit deny **trumps** any allow
- Attach at Org, Folder, Project level
- Use for: "NO ONE can delete this bucket, period"

**Example use:** Deny `storage.buckets.delete` on audit-log bucket — even Org Admins can't delete.

### 🔑 IAM Conditions

Conditional role grants based on:
- **Time** (e.g., access only M-F 9-5)
- **Resource name/prefix** (e.g., only buckets starting with `dev-`)
- **Request attributes** (e.g., only from certain IPs)

**Example condition:**
```
resource.name.startsWith("projects/_/buckets/dev-") && 
request.time < timestamp("2026-12-31T23:59:59Z")
```

### 🔑 IAM Best Practices

| Practice | Why |
|---|---|
| **Use Groups, not individuals** | Easier to manage joiners/leavers |
| **Predefined > Custom > Basic** | Least privilege |
| **Service Accounts for workloads** | Not user identities |
| **IAM Recommender** | Auto-detect overly broad grants |
| **Conditional IAM** | Time, IP, resource-based restrictions |
| **IAM Deny for absolute rules** | Can't be overridden by project admin |
| **Policy Intelligence / Troubleshooter** | Diagnose access issues |

### ⚠️ IAM Anti-Patterns (Exam Traps)

| Anti-Pattern | Why Wrong |
|---|---|
| Granting `Owner` to a user in prod | Way too broad |
| Adding users one-by-one | Use Groups |
| No service accounts (use user creds) | User leaves = workload breaks |
| Downloaded service account keys | Long-lived, easy to leak |
| IAM only at resource level | Maintenance nightmare; set higher |
| "Allow all users" on public bucket for PII | Catastrophic |

---

## 🤖 Service Accounts & Workload Identity

### 🧠 Service Account Types

| Type | Description | Use |
|---|---|---|
| **User-managed SA** | You create and manage | Application workloads |
| **Default SAs** | Auto-created (Compute Engine default, App Engine default) | ⚠️ **Avoid for prod** — too broad |
| **Google-managed SAs** | Google-owned, runs Google services | Never interact directly |
| **Service Agents** | Google services need to act on your behalf | Auto-granted; don't mess with |

### 🔑 Authenticating Workloads — The Options

| Method | Security | When to Use |
|---|---|---|
| **Attached SA (metadata server)** | ✅ Best (no keys) | GCE, GKE, Cloud Run, Cloud Functions |
| **Service Account Impersonation** | ✅ Great | User → SA chains, cross-project |
| **Workload Identity (GKE)** | ✅ Best for GKE | Pods need GCP API access |
| **Workload Identity Federation (WIF)** | ✅ Best for external | AWS, Azure, GitHub Actions, on-prem |
| **SA JSON Keys (downloaded)** | ❌ Worst | ⚠️ Almost never the right answer |

### 🔑 Workload Identity (GKE) Deep Dive

**Concept:** Map a Kubernetes Service Account (KSA) → Google Service Account (GSA).

**Setup:**
```bash
# Bind KSA to GSA
gcloud iam service-accounts add-iam-policy-binding my-gsa@proj.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:proj.svc.id.goog[namespace/my-ksa]"

# Annotate KSA
kubectl annotate serviceaccount my-ksa \
  iam.gke.io/gcp-service-account=my-gsa@proj.iam.gserviceaccount.com
```

**Result:** Pods using `my-ksa` can call GCP APIs as `my-gsa` — no keys, no node-wide creds.

### 🔑 Workload Identity Federation (WIF) Deep Dive

**Concept:** Let external identities (AWS IAM, Azure AD, OIDC providers like GitHub) impersonate GCP service accounts **without a downloaded key**.

**How it works:**
1. External workload gets token from its IdP (e.g., GitHub OIDC token)
2. Workload trades the token with Google STS (Security Token Service)
3. Google STS issues short-lived GCP credentials

**Setup components:**
- **Workload Identity Pool** — logical container
- **Workload Identity Provider** — trust configuration for external IdP
- **Attribute mapping** — map external claims to GCP subject
- **Service account binding** — grant `iam.workloadIdentityUser` role

**Supported external IdPs:**
- AWS (accounts, roles)
- Azure AD
- OIDC providers (GitHub, GitLab, Okta, Auth0)
- SAML providers

**Exam rule:** "External CI/CD, avoid long-lived keys" → **Workload Identity Federation**. Always.

### 🔑 Service Account Impersonation

**Concept:** User or SA A temporarily "acts as" SA B.

**Why:** 
- User doesn't need SA keys
- Audit logs show both identities
- Short-lived tokens (1 hour default)

**Required role:** `roles/iam.serviceAccountTokenCreator` on target SA.

**Usage:**
```bash
gcloud auth print-access-token --impersonate-service-account=my-sa@proj.iam.gserviceaccount.com
```

### ⚠️ Service Account Key Dangers

- **Never expire** (unless manually rotated)
- **Leaked keys = disaster** (often found in GitHub)
- Hard to audit & rotate
- **Block key creation at Org level** via `iam.disableServiceAccountKeyCreation`

**If you MUST use keys:** Rotate every 90 days, store in Secret Manager, monitor usage.

---

## 🔐 Encryption: KMS, CMEK, CSEK, EKM, HSM

### 🧠 The Encryption Stack

**By default, ALL data in GCP is encrypted at rest and in transit with Google-managed keys.** You don't have to do anything. The question is: **do you want more control?**

### 📊 Encryption Options Matrix

| Option | Who Holds Key | Where Key Lives | You Control Rotation? | Google Can Access? |
|---|---|---|---|---|
| **Google-managed (default)** | Google | Google KMS | ❌ No | ✅ Yes |
| **CMEK (Cloud KMS)** | You | Google KMS | ✅ Yes | ✅ With your permission |
| **CMEK (Cloud HSM)** | You | Google's FIPS 140-2 L3 HSM | ✅ Yes | ✅ With your permission |
| **CSEK (Customer-Supplied)** | You | You provide raw key per request | ✅ Yes | ⚠️ Only during API call |
| **EKM (External Key Manager)** | You (external) | Your HSM (Fortanix, Thales, etc.) | ✅ Yes | ❌ Cannot without your IdP approving |

### 🔑 When to Pick Each

**Google-managed (default):**
- Most workloads
- No compliance requirement to control keys
- Simple, no cost for KMS

**CMEK (Cloud KMS):**
- Regulated industries (HIPAA, PCI)
- Need to rotate/revoke on your schedule
- Need to audit key usage
- **Most common correct answer on exam** for compliance

**CMEK with Cloud HSM:**
- Need FIPS 140-2 Level 3 hardware
- Still Google infra but hardware-backed

**CSEK (Customer-Supplied):**
- ⚠️ Limited to **GCE persistent disks, GCE images, GCS objects**
- You provide key in API request (raw)
- Google doesn't store the key
- Complex to manage; lose key = lose data

**EKM (External Key Manager):**
- HYOK (Hold Your Own Key)
- Key never leaves YOUR HSM
- Google calls out to your HSM over VPC (internal) or internet
- **Highest control**, lower availability (depends on your HSM)
- Partners: Fortanix, Thales, Equinix SmartKey, Virtru, Atos

### 🔑 Cloud KMS Hierarchy

```
Key Ring (regional/global/multi-region)
   └── Crypto Key (symmetric or asymmetric)
          └── Crypto Key Version (the actual key material)
```

**Key types:**
- **Symmetric** (AES-256): Most common
- **Asymmetric signing** (RSA, EC): For signing/verification
- **Asymmetric decryption** (RSA): For encrypt/decrypt
- **HMAC**: For MACs

**Key rotation:**
- Auto-rotate every N days (90 default)
- Old versions kept for decryption of old data
- Newest version used for new encryption

### 🔑 Envelope Encryption

**Concept:** Encrypt data with a **Data Encryption Key (DEK)**, then encrypt the DEK with a **Key Encryption Key (KEK)** stored in KMS.

- DEKs are fast, used to encrypt large data
- KEKs stay in KMS; only small amount crosses network

**GCS, BigQuery, Compute Engine all use envelope encryption under the hood.**

### 🔑 Confidential Computing (encryption IN USE)

| Product | Tech | Use |
|---|---|---|
| **Confidential VMs** | AMD SEV / Intel TDX | Memory encrypted even from Google |
| **Confidential GKE Nodes** | Same | Confidential K8s workloads |
| **Confidential Space** | TEE | Multi-party computation |

**Exam rule:** "Encrypt data even in memory / in use" → **Confidential VMs or Confidential GKE Nodes**.

---

## 🛡️ Data Protection (Sensitive Data Protection)

### 🧠 Sensitive Data Protection (formerly DLP API)

**Capabilities:**
- **Inspect** data for sensitive types (SSN, credit card, health records, PII)
- **De-identify** (redact, mask, tokenize, crypto-hash)
- **Re-identify** (reverse de-identification with key)
- **Risk analysis** (re-identification risk)

**Built-in detectors (infoTypes):**
- PERSON_NAME, EMAIL_ADDRESS, PHONE_NUMBER
- US_SOCIAL_SECURITY_NUMBER, CREDIT_CARD_NUMBER
- MEDICAL_RECORD_NUMBER, US_PASSPORT
- 150+ country-specific types

### 🔑 De-identification Techniques

| Technique | Example | Reversible? |
|---|---|---|
| **Redaction** | `John Smith` → `[NAME]` | ❌ No |
| **Masking** | `123-45-6789` → `***-**-6789` | ❌ No |
| **Replacement** | `John Smith` → `Jane Doe` | ❌ No |
| **Tokenization** | `1234-5678` → `TOK_a8f9c` | ✅ With crypto key |
| **Cryptographic hashing** | `john@x.com` → `sha256(...)` | ❌ No (one-way) |
| **Format-preserving encryption (FPE)** | `4532-1111` → `8123-9823` (valid format) | ✅ |
| **Date shifting** | `2024-05-10` → `2024-05-17` (consistent shift) | ✅ |
| **Generalization** | Age 37 → `30-40` | ❌ |

### 🔑 Use Cases

- Scan **GCS buckets** for PII
- Inspect **BigQuery tables** for SSN/PHI
- De-identify data **before loading into analytics**
- **Streaming pipelines** via Dataflow (DLP templates)

### 🔑 Data Catalog / Dataplex Catalog

**Purpose:** Metadata catalog for all data assets (BQ tables, GCS files, Pub/Sub topics).

**Features:**
- **Tag templates** for classifying data (e.g., "contains PII", "confidentiality=HIGH")
- **Policy Tags** for column-level ACLs in BigQuery
- Integration with **Sensitive Data Protection** for auto-tagging

**Exam rule:** "Discover and classify all data assets" → **Dataplex Catalog + Sensitive Data Protection**.

### 🔑 BigQuery Column-Level Security

Use **Policy Tags** to restrict column access:
1. Create taxonomy in **Dataplex Catalog / Data Catalog**
2. Create policy tags (e.g., `HIGH_SENSITIVITY`)
3. Apply tag to BigQuery column (e.g., `ssn`)
4. Grant `Fine-Grained Reader` role on policy tag
5. Users without role see `NULL` or get error

---

## 🌐 Network Security (Defense in Depth)

### 🧠 The Layered Security Model

```
┌───────────────────────────────────────┐
│ Internet                              │
├───────────────────────────────────────┤
│ 🛡 Cloud Armor (DDoS, WAF)            │  ← L7 edge
├───────────────────────────────────────┤
│ 🔐 IAP (identity-aware access)        │  ← L7 app access
├───────────────────────────────────────┤
│ 🌐 Load Balancer (TLS term)           │
├───────────────────────────────────────┤
│ 🔥 VPC Firewall / Hierarchical Policy │  ← L3/L4
├───────────────────────────────────────┤
│ 🏰 VPC Service Controls               │  ← API data plane
├───────────────────────────────────────┤
│ 🖥 VM / Container / Service           │
├───────────────────────────────────────┤
│ 🗝 IAM (authentication)               │
├───────────────────────────────────────┤
│ 📦 Data (CMEK, VPC-SC, DLP)           │
└───────────────────────────────────────┘
```

### 🔑 Firewall Best Practices

- **Default-deny** mindset: Delete default network, start from scratch
- **Tag/SA-based rules** (not IP-based where possible)
- **Hierarchical policies** for org-wide rules
- **Priority 0** reserved for critical denies; work down

### 🔑 Cloud NGFW Enterprise (new 2024+)

- **L7 deep packet inspection**
- **IDS/IPS** signatures (Palo Alto)
- **TLS inspection**
- Applied via VPC firewall policies
- For enterprise/high-security VPCs

---

## 🏰 VPC Service Controls (Deep Dive)

### 🧠 What VPC-SC Actually Does

**Plain English:** Builds a **data-exfiltration perimeter** around GCP APIs. Even if an attacker has valid IAM creds, they can't copy data **out of the perimeter** to their personal bucket.

### 🔑 Key Concepts

| Concept | Description |
|---|---|
| **Perimeter** | A set of projects whose specified services are "inside" |
| **Restricted services** | APIs protected by perimeter (BQ, GCS, Spanner, etc.) |
| **Access level** | Condition that allows traffic in (e.g., from corp network) |
| **Ingress rule** | Allow data IN (specific identity from specific source) |
| **Egress rule** | Allow data OUT (specific identity to specific destination) |
| **Bridge perimeter** | Connect perimeters with controlled sharing |
| **Dry-run mode** | Test perimeter without enforcing |

### 🔑 What VPC-SC Protects

✅ Data-plane APIs:
- Cloud Storage
- BigQuery
- Bigtable
- Spanner
- Pub/Sub
- KMS
- 60+ services

❌ NOT protected (different control plane):
- Compute Engine (VM metadata, firewall rules)
- IAM itself
- Monitoring

### 🔑 Common VPC-SC Patterns

**Pattern 1: PII Perimeter**
- Put project with sensitive data + KMS + Storage + BigQuery inside perimeter
- Block any egress to outside projects/buckets

**Pattern 2: On-prem Access**
- Access level for on-prem IP ranges or VPN endpoints
- Allow corp → perimeter only through Private Google Access

**Pattern 3: Shared Services**
- Separate perimeter for shared logging/monitoring projects
- Use perimeter bridges to cross boundaries

### ⚠️ VPC-SC Gotchas

- ⚠️ **Not a firewall** — doesn't block network traffic, only API calls
- ⚠️ **Doesn't protect against authorized access** — just unauthorized cross-perimeter data movement
- ⚠️ **Setting up is hard** — use **dry-run mode first**
- ⚠️ **Takes effect in minutes**, not instant
- ⚠️ **Service account cross-project access** is a common source of unexpected denials

---

## 🛡️ Cloud Armor & DDoS Protection

### 🧠 Cloud Armor Tiers

| Tier | What You Get |
|---|---|
| **Standard** | Free DDoS protection (network + volumetric) |
| **Managed Protection Plus** (paid) | Advanced features below |

### 🔑 Managed Protection Plus Features

| Feature | Purpose |
|---|---|
| **Adaptive Protection** | ML-based L7 DDoS detection |
| **Preconfigured WAF rules** | OWASP Top 10 (SQLi, XSS, LFI, RFI) |
| **Custom rules** | Your own logic (CEL expressions) |
| **Rate limiting** | Per IP, per header, per cookie |
| **Bot management** | reCAPTCHA Enterprise integration |
| **Named IP lists** | Fastly, Cloudflare, Tor lists |
| **Geo-based rules** | Block by country |
| **Edge security policies** | Attached to global LB |
| **Network edge security** | For Network LB (2024+) |

### 🔑 Where Cloud Armor Attaches

| Attachment | Coverage |
|---|---|
| **Global External Application LB** | ✅ Full WAF + DDoS |
| **Global External Proxy Network LB** | ✅ Some rules |
| **Regional External Application LB** | ✅ Full WAF + DDoS |
| **External Passthrough Network LB** | ⚠️ Limited (2024+ network edge policies) |
| **Internal LB** | ❌ No (not internet-facing) |

### 🔑 Cloud Armor Rule Structure

```yaml
Priority: 1000
Action: allow | deny | throttle | rate_based_ban | redirect
Match: 
  - IP range: "1.2.3.0/24"
  - Expression: "origin.region_code == 'CN'"
  - Preconfigured: "sqli-v33-stable"
```

### ⚠️ Exam Trap

"Protect web app from DDoS and OWASP attacks" — **the answer is Cloud Armor**, not Cloud CDN, not firewall rules, not IAP alone.

---

## 🔐 Identity-Aware Proxy (IAP)

### 🧠 What IAP Does

**Replaces VPN for accessing internal apps.** Users authenticate to Google, IAP checks their identity + context, then forwards to your app.

### 🔑 IAP Use Cases

| Use Case | How |
|---|---|
| **Protect internal web app** | IAP in front of Application LB |
| **Protect GCE VM SSH** | IAP TCP forwarding (`gcloud compute ssh --tunnel-through-iap`) |
| **Protect GKE app** | IAP as ingress authorization |
| **Protect on-prem app** | IAP with hybrid connectivity |

### 🔑 IAP + Context-Aware Access

Combine IAP with **Access Context Manager** for:
- Device posture (managed Chromebook only)
- User location (corp IP only)
- Time of day
- BeyondCorp Enterprise

### ⚠️ IAP Gotchas

- Apps must be behind **HTTPS LB** (or use TCP forwarding)
- Need **OAuth consent screen** configured
- Users need `IAP-secured Web App User` role

**Exam rule:** "Zero-trust web access without VPN" → **IAP**.

---

## 🗝️ Secrets Management

### 🧠 Secret Manager Basics

**Purpose:** Store API keys, passwords, certs, tokens.

**Features:**
- **Versioning** (secrets are immutable; new values = new versions)
- **IAM per secret** (who can read which secret)
- **Automatic rotation** hooks (via Cloud Functions/Scheduler)
- **Regional replication** or auto (multi-region)
- **CMEK support**
- **Audit logs** per access

### 🔑 Secret Manager vs Alternatives

| Storage | Should You Use? |
|---|---|
| **Secret Manager** | ✅ Yes — default |
| **Environment variables** | ⚠️ For non-sensitive config only |
| **Config files in image** | ❌ Never |
| **Git repository** | ❌ Never (even private) |
| **KMS directly** | ⚠️ KMS encrypts keys, not stores arbitrary secrets |
| **Cloud Storage** | ⚠️ Can work with strict IAM + CMEK, but Secret Manager is purpose-built |

### 🔑 Certificate Management

**Certificate Manager:**
- Manages TLS certs for LBs
- **Google-managed certs**: auto-issued, auto-renewed (free)
- **Self-managed certs**: you upload

**Exam rule:** "HTTPS cert for LB, auto-renew" → **Certificate Manager with Google-managed cert**.

---

## 📋 Compliance Frameworks & Assured Workloads

### 🧠 Compliance Frameworks Supported

| Framework | Domain |
|---|---|
| **HIPAA** | US healthcare |
| **PCI-DSS** | Payment cards |
| **GDPR** | EU privacy |
| **FedRAMP Moderate / High** | US federal |
| **IL4 / IL5 / IL6** | US DoD |
| **ITAR** | US export control (defense) |
| **CJIS** | US criminal justice |
| **SOC 1 / 2 / 3** | Service orgs |
| **ISO 27001 / 27017 / 27018** | Intl |
| **HITRUST** | Healthcare (extended HIPAA) |
| **FIPS 140-2** | Cryptographic modules |

### 🔑 Assured Workloads

**What it is:** Pre-configured compliance environments that enforce data residency, personnel controls, and encryption requirements.

**Offerings:**
- **HIPAA**
- **FedRAMP Moderate / High**
- **IL4 / IL5**
- **CJIS**
- **ITAR**
- **EU Sovereignty**
- **Canada**
- **Korea**
- **Australia**
- **Israel**
- **Saudi Arabia**
- **UK**

**What it enforces:**
- Resource location (data stays in jurisdiction)
- Personnel access controls (e.g., US persons only for ITAR)
- CMEK requirements
- Product allow-list (only compliant services)

**Exam rule:** "Meet HIPAA / FedRAMP / etc. with guardrails" → **Assured Workloads**, not manual config.

### 🔑 Sovereign Cloud / EU Sovereign Cloud

- **Trusted Partners** run GCP in-country (T-Systems Germany, Thales France, Telecom Italia)
- Full data + operational sovereignty
- For EU public sector / highly regulated

### 🔑 Access Transparency vs Access Approval

| | Access Transparency | Access Approval |
|---|---|---|
| What | **Logs** when Google staff access your data | **Requires your approval** before access |
| Purpose | Audit trail | Active control |
| Required for | Compliance | Sensitive workloads |
| Cost | Free (for supported services) | Paid add-on |

**Exam rule:** "Know when Google staff access our data" → **Access Transparency**. "Require our approval before Google accesses" → **Access Approval**.

---

## 📜 Audit Logging & Access Transparency

### 🧠 The 4 Types of Cloud Audit Logs

| Type | What It Logs | On by Default? | Retention |
|---|---|---|---|
| **Admin Activity** | API calls that modify config/resources | ✅ Always on | 400 days free |
| **Data Access** | API calls that read/write user data | ❌ **OFF by default** (for most services) | 30 days free |
| **System Event** | Google-initiated actions (e.g., live migration) | ✅ Always on | 400 days free |
| **Policy Denied** | Requests denied by Org Policy / VPC-SC | ✅ Always on | 30 days free |

### ⚠️ Huge Exam Trap

**"Need to know who read sensitive data"** → Enable **Data Access audit logs** (they're OFF by default for most services). Just Admin Activity isn't enough.

### 🔑 Audit Log Configuration

Enable Data Access logs:
```bash
# In IAM Audit Config
{
  "auditLogConfigs": [
    {"logType": "DATA_READ"},
    {"logType": "DATA_WRITE"},
    {"logType": "ADMIN_READ"}
  ]
}
```

### 🔑 Log Retention & Export

| Destination | Use |
|---|---|
| **Cloud Logging** | Default; short-term |
| **BigQuery** (via sink) | Long-term analytics on logs |
| **Cloud Storage** (via sink) | Archival compliance (locked retention) |
| **Pub/Sub** (via sink) | Real-time forward to SIEM |
| **Another project** | Aggregated logging project |

**Aggregated sinks:** At Org/Folder level, collect all logs from downstream projects.

### 🔑 Log-Based Alerts & Metrics

- **Log-based metrics:** Count matching log entries (e.g., 4xx errors per service)
- **Log-based alerts:** Alert when specific log pattern occurs (e.g., "IAM policy changed")

### 🔑 Security Command Center (SCC)

**Centralized security management:**
- **Standard tier (free):** Security Health Analytics, Web Security Scanner
- **Premium tier (paid):** Event Threat Detection, Container Threat Detection, Virtual Machine Threat Detection, Mandiant integrations, VPC-SC violations, Rapid Vulnerability Detection
- **Enterprise tier (2024+):** Full SIEM/SOAR with Mandiant

**Exam rule:** "Centralized security posture, threat detection" → **Security Command Center**.

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 20 Traps in Domain 3

1. **"Prevent data exfiltration"** → **VPC Service Controls**, NOT firewall/IAM
2. **"External CI/CD authenticate"** → **Workload Identity Federation**, NOT SA keys
3. **"GKE pod accesses GCP API"** → **Workload Identity (GKE)**, NOT SA key secret
4. **"HIPAA/FedRAMP compliance"** → **Assured Workloads**, NOT manual CMEK+VPC-SC
5. **"Google can never access data"** → **EKM**, NOT CMEK (Google has your CMEK key with permission)
6. **"Encrypt data in memory"** → **Confidential VMs**, NOT CMEK (CMEK is at rest)
7. **"WAF / OWASP / DDoS"** → **Cloud Armor**, NOT Cloud CDN/Firewall alone
8. **"Zero-trust internal web app"** → **IAP**, NOT VPN
9. **"Who read this data?"** → **Enable Data Access audit logs** (off by default!)
10. **"Know when Google accesses data"** → **Access Transparency**
11. **"Require approval before Google access"** → **Access Approval**
12. **"Store API keys"** → **Secret Manager**, NOT env vars/GCS/git
13. **"Basic role in prod (Owner/Editor)"** → Wrong; use **predefined or custom**
14. **"Individual user grants"** → Use **Groups**
15. **"Deny despite grants"** → **IAM Deny Policies**, NOT removing allow
16. **"Cloud SQL private connection"** → **Private Service Access / PSC**, NOT VPN
17. **"Column-level restriction in BQ"** → **Policy Tags**, NOT row-level or views
18. **"Block SA key creation org-wide"** → **Org Policy `iam.disableServiceAccountKeyCreation`**
19. **"FIPS 140-2 L3 hardware keys"** → **Cloud HSM** or **EKM to your HSM**
20. **"Classify PII across GCS/BQ"** → **Sensitive Data Protection + Dataplex Catalog**

### ⚠️ "Sounds Right but Isn't"

| Seems Right | Actually Right | Why |
|---|---|---|
| Firewall rules to prevent exfil | VPC-SC | Firewall is network L3/L4; exfil is over APIs |
| Download SA key, store in Vault/GitHub Secrets | Workload Identity Federation | Still a long-lived secret |
| CMEK alone for HIPAA | Assured Workloads for HIPAA + CMEK | HIPAA needs more controls |
| IAM only to limit access | IAM + Deny Policies | Deny > allow |
| Admin Activity logs alone | Enable Data Access logs too | Admin doesn't show reads |
| Cloud Armor for internal | IAP + firewall for internal | Armor is edge/LB |
| Cloud Endpoints for security | API Gateway + OAuth or Apigee | Endpoints is for API management, not defense |
| CSEK for all storage | CMEK (CSEK is limited, error-prone) | CSEK = edge case only |

---

## 🔑 Decision Frameworks

### 🎯 Encryption Picker

```
Any compliance requirement for key control?
  ├── No → Google-managed (default, free)
  ├── Yes, rotate/audit → CMEK (Cloud KMS)
  ├── Yes, FIPS 140-2 L3 hardware → Cloud HSM
  ├── Yes, key never leaves our infra → EKM
  └── Encrypt even in use (memory) → Confidential VMs / GKE Nodes
```

### 🎯 Workload Auth Picker

```
Where does the workload run?
  ├── GCE / Cloud Run / Cloud Functions → Attached SA (metadata)
  ├── GKE pod → Workload Identity (GKE)
  ├── External (AWS, Azure, GitHub, on-prem) → Workload Identity Federation
  ├── User → SA impersonation (short-lived)
  └── SA key → ❌ ALMOST NEVER
```

### 🎯 Network Security Picker

```
What's the threat?
  ├── L3/L4 network access → Firewall rules (ideally SA targets)
  ├── L7 DDoS / WAF → Cloud Armor on global LB
  ├── Identity-based app access → IAP
  ├── API data exfiltration → VPC Service Controls
  ├── Sensitive data leak → Sensitive Data Protection + DLP
  └── In-flight Google staff access → Access Transparency + Access Approval
```

### 🎯 Compliance Picker

```
Regulated workload?
  ├── HIPAA / PCI-DSS (US) → Assured Workloads (HIPAA) + BAA signed
  ├── FedRAMP / DoD → Assured Workloads (FedRAMP / IL4 / IL5)
  ├── EU data sovereignty → Assured Workloads EU or Sovereign Cloud partner
  ├── ITAR (defense exports) → Assured Workloads ITAR (US persons only)
  └── CJIS (police data) → Assured Workloads CJIS
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "WIF > KEY"
**Workload Identity Federation** always beats downloaded **KEYs**.

### 🧠 "4 A's of IAM"
**A**ccounts (user/group/SA) → **A**uthentication (who) → **A**uthorization (roles) → **A**uditing (logs)

### 🧠 "ADSP" (Audit Log Types)
**A**dmin Activity (on), **D**ata Access (OFF!), **S**ystem Event (on), **P**olicy Denied (on)

### 🧠 "Moat vs Lock"
VPC Service Controls = **moat** (stops data leaving).  
IAM = **lock** (stops people entering).

### 🧠 "CEK Pyramid"
Encryption control ladder (low → high):
Google-managed → **C**MEK → HSM → **E**KM → Confidential Computing (in use)

### 🧠 "CHIEF"
Compliance answer picker:
**C**JIS, **H**IPAA, **I**L4/5, **E**U sovereignty, **F**edRAMP → all → **Assured Workloads**

### 🧠 "SCIPIO" (Cloud Armor features)
**S**QLi detection, **C**ustom rules, **I**P/geo blocking, **P**reconfigured OWASP, **I**AP-compatible, **O**f course rate limiting

### 🧠 "IAP = VPN-killer"
For web apps + SSH, **IAP** replaces VPN with zero-trust identity.

---

## ✅ Practice Checkpoint

### Q1. External CI/CD without keys
**Scenario:** GitHub Actions deploys to GCP. Security team bans long-lived credentials.

**Options:**
- A) Download SA JSON key, store in GitHub encrypted secret
- B) Configure Workload Identity Federation with GitHub's OIDC
- C) Create a user account for GitHub
- D) Store key in Secret Manager and fetch via GitHub Action

✅ **Answer: B** — **WIF** uses GitHub's OIDC token to get short-lived GCP creds. No key ever created.

---

### Q2. Healthcare compliance stack
**Scenario:** Healthcare SaaS. PHI, HIPAA, 50 states + EU patients. Customer-managed keys, audit trail, exfiltration prevention.

**Options:**
- A) Default encryption + IAM
- B) CMEK + VPC-SC + Data Access audit logs
- C) Assured Workloads (HIPAA) + CMEK + VPC Service Controls + Access Transparency + Data Access audit logs
- D) EKM only

✅ **Answer: C** — Defense-in-depth: Assured Workloads sets compliance floor; CMEK for key control; VPC-SC for exfiltration; Access Transparency for Google access logs; Data Access logs for user access.

---

### Q3. Prevent exfiltration with stolen creds
**Scenario:** Risk: an authorized user's creds get phished. Attacker tries to copy BigQuery data to their personal GCS bucket.

**Options:**
- A) Require MFA for all users
- B) VPC Service Controls perimeter around BQ + GCS projects
- C) IAM Deny Policy on BQ
- D) Cloud Armor

✅ **Answer: B** — MFA helps but doesn't stop authorized-creds use. **VPC-SC** blocks cross-perimeter API calls even with valid IAM.

---

### Q4. Google can't access data
**Scenario:** Government client requires that Google have no cryptographic access to data, period. Keys must reside outside GCP.

**Options:**
- A) CMEK in Cloud KMS
- B) CMEK with Cloud HSM
- C) EKM (External Key Manager) with Fortanix HSM
- D) CSEK

✅ **Answer: C** — **EKM** keeps key outside GCP; Google calls YOUR HSM. CMEK/HSM still run inside GCP infra.

---

### Q5. GKE pod accessing GCS
**Scenario:** Pod needs to read a GCS bucket. Security: no node-wide creds, no SA keys.

**Options:**
- A) Mount SA key as secret
- B) Use GCE metadata server (attached SA to nodes)
- C) Workload Identity (GKE): bind KSA to GSA
- D) Use user token

✅ **Answer: C** — **Workload Identity (GKE)** maps KSA → GSA. Per-pod identity, no keys, no node-wide compromise.

---

### Q6. DDoS + OWASP Top 10
**Scenario:** Public-facing web app. Need defense against SQLi, XSS, L7 DDoS, and geo-blocking from certain countries.

**Options:**
- A) Cloud CDN + firewall rules
- B) VPC Service Controls
- C) Cloud Armor with managed WAF rules + custom geo rules + adaptive protection
- D) IAP

✅ **Answer: C** — **Cloud Armor Managed Protection Plus** is the full WAF + DDoS + geo-blocking answer.

---

### Q7. Replace VPN for internal apps
**Scenario:** 500 employees currently VPN to access internal Jenkins and Grafana. Security wants zero-trust; work-from-anywhere.

**Options:**
- A) Larger VPN
- B) Public expose with basic auth
- C) IAP in front of LB, users authenticate to Google
- D) Cloud Armor

✅ **Answer: C** — **IAP** replaces VPN with identity-aware zero-trust access.

---

### Q8. Audit who read PII table
**Scenario:** Need report of every user who SELECTed from `patients` table in BigQuery last month.

**Options:**
- A) Admin Activity audit logs
- B) Enable Data Access (DATA_READ) audit logs for BigQuery
- C) Query the data ourselves
- D) VPC-SC logs

✅ **Answer: B** — **Data Access logs** are OFF by default and must be enabled for DATA_READ to capture SELECTs.

---

### Q9. Column-level security in BigQuery
**Scenario:** Marketing analysts can query customer table, but `ssn` column must be invisible to them. Data engineers can see all.

**Options:**
- A) Different table per role
- B) Row-level security
- C) Policy Tags on `ssn` column + role-based grant on tag
- D) Custom view

✅ **Answer: C** — **Policy Tags** (via Dataplex Catalog) provide column-level access control; most elegant.

---

### Q10. Secret storage
**Scenario:** Cloud Run service needs a third-party API key. Rotate quarterly, audit access.

**Options:**
- A) Env variable in the service
- B) Mount a Docker image with key baked in
- C) Secret Manager, granted via SA IAM; rotate version quarterly
- D) Git private repo

✅ **Answer: C** — **Secret Manager** with IAM + versioning + audit logs + CMEK option.

---

### Q11. Immutable storage for compliance
**Scenario:** Financial records must be retained 7 years and cannot be deleted even by admins.

**Options:**
- A) GCS Coldline + lifecycle delete after 7yr
- B) GCS bucket with locked retention policy (7yr) + bucket-level IAM + Object Hold
- C) BigQuery time-travel
- D) Cloud SQL backups

✅ **Answer: B** — **Locked retention policy** is immutable — even Project Owner can't delete. Perfect for compliance.

---

### Q12. Block SA key creation
**Scenario:** Security team wants to enforce: no one can create SA keys anywhere in the org.

**Options:**
- A) Revoke all user IAM roles
- B) Org Policy constraint `iam.disableServiceAccountKeyCreation` at Org
- C) Audit logs + automation to delete keys
- D) VPC-SC

✅ **Answer: B** — **Org Policy constraint** at Org node enforces downward. Automation detection is weak (creates, then detects).

---

### Q13. Hybrid zero-trust
**Scenario:** Company uses BeyondCorp principles. Apps accessible only from managed Chromebooks in corp IP range during business hours.

**Options:**
- A) VPN with time restrictions
- B) IAP + Access Context Manager (access levels: device + IP + time)
- C) Firewall rules
- D) Cloud Armor

✅ **Answer: B** — **IAP + Access Context Manager** delivers full BeyondCorp zero-trust policy.

---

### Q14. Detect malware on VM
**Scenario:** Need to detect cryptomining, suspicious processes on production GCE fleet.

**Options:**
- A) Cloud Armor
- B) Security Command Center Premium — Virtual Machine Threat Detection
- C) VPC-SC
- D) Antivirus agent install

✅ **Answer: B** — **SCC Premium VM Threat Detection** monitors runtime for malware, cryptomining, etc., agentless.

---

### Q15. ITAR compliance
**Scenario:** US defense contractor must process ITAR-regulated designs. Only US persons can access; data must not leave US.

**Options:**
- A) us-central1 region with IAM restrictions
- B) Assured Workloads ITAR with CMEK
- C) VPC-SC only
- D) On-prem only

✅ **Answer: B** — **Assured Workloads ITAR** enforces US-persons-only support + US region + CMEK — out of the box.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **Cloud DLP renamed Sensitive Data Protection** | Same product, new name |
| **Data Catalog merged into Dataplex Catalog** | Unified catalog |
| **Cloud Armor Network Edge Security** | Protect Network LBs (2024+) |
| **Confidential Computing expanded** | Intel TDX added; Confidential GKE Nodes |
| **IAM Deny Policies GA** | Explicit denies org-wide |
| **Workload Identity Federation expanded** | More OIDC providers, simpler setup |
| **EU Sovereign Cloud partners GA** | T-Systems, Thales, TIM |
| **Security Command Center Enterprise** | Full SIEM/SOAR with Mandiant |
| **Assured Workloads added regions** | Korea, Israel, Saudi, UK, etc. |
| **Cloud KMS Autokey** | Auto-create CMEK keys for resources |
| **Certificate Manager (Google-managed)** | Preferred over per-LB certs |
| **Cloud NGFW Enterprise** | L7 DPI with Palo Alto IDS/IPS |

---

## 🎯 Final Domain 3 Checklist

Before exam day, you should be able to:

- [ ] Pick encryption tier from scenario (Google-mgmt → CMEK → HSM → EKM → Confidential)
- [ ] Choose auth method (attached SA / WI / WIF / impersonation / never keys)
- [ ] Know VPC-SC use cases and limitations (not a firewall)
- [ ] Know Cloud Armor features and when to attach
- [ ] Know IAP use cases (web apps, SSH via TCP forwarding)
- [ ] Know when Assured Workloads is the answer
- [ ] Know audit log types — **Data Access is OFF by default**
- [ ] Know Access Transparency vs Access Approval
- [ ] Know Secret Manager as default secrets store
- [ ] Know Sensitive Data Protection (DLP) infoTypes + de-identification
- [ ] Know Policy Tags for BigQuery column-level security
- [ ] Know Security Command Center tiers
- [ ] Know IAM Deny Policies vs allow

> **Domain 3 is the unforgiving one. Wrong answers here scream "doesn't understand security". Master the decision trees above.** 🔒
