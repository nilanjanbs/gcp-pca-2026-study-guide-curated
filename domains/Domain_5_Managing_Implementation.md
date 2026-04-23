# 🚀 Domain 5: Managing Implementation

> **GCP Professional Cloud Architect — 2026 Exam | Deep-Dive Study Guide (5 of 6)**

**Exam Weight:** ~11%

---

## 📖 Table of Contents
1. [What the Exam Actually Tests](#-what-the-exam-actually-tests)
2. [CI/CD Pipeline Architecture](#-cicd-pipeline-architecture)
3. [Cloud Build (Deep Dive)](#-cloud-build-deep-dive)
4. [Artifact Registry](#-artifact-registry)
5. [Cloud Deploy (Deep Dive)](#-cloud-deploy-deep-dive)
6. [Deployment Strategies](#-deployment-strategies)
7. [Supply Chain Security](#-supply-chain-security)
8. [API Management](#-api-management)
9. [Working with Development Teams](#-working-with-development-teams)
10. [Secrets & Configuration](#-secrets--configuration)
11. [Testing Strategies](#-testing-strategies)
12. [Exam Traps & Tricks](#-exam-traps--tricks)
13. [Decision Frameworks](#-decision-frameworks)
14. [Mnemonics & Memory Hacks](#-mnemonics--memory-hacks)
15. [Practice Checkpoint (15 scenarios)](#-practice-checkpoint)
16. [2025–2026 Changes](#-2025-2026-changes)

---

## 🎯 What the Exam Actually Tests

Domain 5 is about **operating the delivery machine** — how code moves from dev's laptop to production safely, quickly, and repeatably. Expect scenarios like:

- "Deploy to GKE with 5% canary initially"
- "Enforce only signed images in prod"
- "Replace manual deploys with automation"
- "Expose APIs to external partners with quota + monetization"
- "Store secrets for CI/CD pipelines without long-lived keys"

The exam tests these sub-skills:

1. **CI/CD design** — Cloud Build, Artifact Registry, Cloud Deploy pipelines
2. **Deployment strategies** — rolling, blue/green, canary, traffic splitting
3. **Supply chain security** — Binary Authorization, attestations, SLSA, Artifact Analysis
4. **API management** — Apigee vs API Gateway vs Cloud Endpoints
5. **Dev team collaboration** — IAM for CI/CD, self-service platforms
6. **Secrets in pipelines** — Secret Manager + WIF
7. **Progressive delivery** — canary + auto-rollback on error budget

---

## 🔧 CI/CD Pipeline Architecture

### 🧠 The Modern GCP CI/CD Stack (2026)

```
┌──────────────┐     ┌──────────────┐      ┌──────────────────┐
│   Source     │────→│     CI       │─────→│    Artifact      │
│  (GitHub,    │     │ (Cloud Build)│      │    (Artifact     │
│ GitLab, CSR) │     │              │      │     Registry)    │
└──────────────┘     └──────────────┘      └──────────────────┘
                            │                       │
                            ▼                       ▼
                     ┌──────────────┐      ┌──────────────────┐
                     │ Artifact     │      │     CD           │
                     │ Analysis     │─────→│  (Cloud Deploy)  │
                     │ (vuln scan)  │      │                  │
                     └──────────────┘      └──────────────────┘
                            │                       │
                            ▼                       ▼
                     ┌──────────────┐      ┌──────────────────┐
                     │   Binary     │─────→│  Prod (GKE /     │
                     │ Authorization│      │  Cloud Run / GCE)│
                     └──────────────┘      └──────────────────┘
```

### 🔑 Source Control Options

| Source | When to Use |
|---|---|
| **GitHub** | Most common; best ecosystem |
| **GitLab** | If already on GitLab |
| **Cloud Source Repositories** | Mirror or use natively within GCP |
| **Bitbucket** | If already using Atlassian |

**Exam rule:** Cloud Source Repositories is the "GCP-native" answer but exams often accept GitHub/GitLab integration.

### 🔑 The Full CI/CD Toolchain Comparison

| Role | GCP Native | Third-Party (also exam-valid) |
|---|---|---|
| **Source** | Cloud Source Repositories | GitHub, GitLab, Bitbucket |
| **CI** | Cloud Build | Jenkins, CircleCI, GitHub Actions |
| **Artifacts** | Artifact Registry | Docker Hub, JFrog Artifactory |
| **Vulnerability scanning** | Artifact Analysis (Container Analysis) | Snyk, Aqua |
| **Container signing** | Binary Authorization + KMS | Sigstore/Cosign |
| **CD** | Cloud Deploy | Spinnaker, ArgoCD, FluxCD |
| **Secrets** | Secret Manager | HashiCorp Vault |
| **IaC** | Infrastructure Manager / Terraform | — |

---

## 🏗️ Cloud Build (Deep Dive)

### 🧠 Cloud Build Fundamentals

**Purpose:** Managed CI — runs steps in containers to build/test/push/deploy code.

**Configuration:** `cloudbuild.yaml` (or JSON) defines steps.

**Example:**
```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'us-docker.pkg.dev/PROJECT/repo/app:$SHORT_SHA', '.']
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'us-docker.pkg.dev/PROJECT/repo/app:$SHORT_SHA']
  - name: 'gcr.io/cloud-builders/gke-deploy'
    args: ['run', '--filename=k8s.yaml', '--image=...', '--cluster=...']
images:
  - 'us-docker.pkg.dev/PROJECT/repo/app:$SHORT_SHA'
```

### 🔑 Cloud Build Concepts

| Concept | Description |
|---|---|
| **Build config** | `cloudbuild.yaml` with steps |
| **Triggers** | Start builds on push, PR, schedule, Pub/Sub |
| **Substitutions** | Variables (`$SHORT_SHA`, custom) |
| **Artifacts** | Output (container images, binaries) |
| **Logs** | Stored in Cloud Logging / GCS |
| **Builder images** | Pre-built containers for common tasks (docker, gcloud, gradle, etc.) |
| **Custom builders** | Your own containers for custom steps |

### 🔑 Cloud Build Pools

| Pool Type | Purpose |
|---|---|
| **Default pool** | Public; no VPC access; fast & free up to limit |
| **Private pool** | Runs in your VPC; access to private resources (Cloud SQL private IP, on-prem via VPN) |
| **Worker pool** | More CPU/memory machine types |

**Exam rule:** If build needs access to **private Cloud SQL, on-prem resources, internal APIs** → **Private Pool**.

### 🔑 Cloud Build Triggers

**Trigger types:**
- **Push to branch** (e.g., `main` → deploy to prod)
- **Pull request** (run tests)
- **Tag** (release builds)
- **Manual**
- **Pub/Sub** (event-driven)
- **Webhook**
- **Scheduled** (via Cloud Scheduler)

**Branching pattern:**
- `feature/*` → run tests
- `main` → build + deploy to staging
- `tags/v*` → deploy to prod via Cloud Deploy

### 🔑 Cloud Build Best Practices

- **Use build caching** (Docker layer cache, build cache with GCS)
- **Parallelize independent steps** (`waitFor: ['-']`)
- **Use Workload Identity Federation** for external triggers (no keys)
- **Scope service account permissions** — minimal IAM for the build SA
- **Store state artifacts** (test reports, SBOMs) in GCS
- **Stream logs to GCS** for retention > default

### 🔑 Cloud Build Security

| Concern | Solution |
|---|---|
| Default SA too powerful | Create custom SA with minimum perms |
| Keys in logs | Enable secret masking; use Secret Manager |
| Unauthorized triggers | Use branch protection rules |
| External IdP auth | WIF (GitHub OIDC → impersonate SA) |
| Need to sign artifacts | Use KMS signing in build step |

---

## 📦 Artifact Registry

### 🧠 Artifact Registry vs Container Registry

⚠️ **Container Registry (GCR) is DEPRECATED.** Always use **Artifact Registry**.

### 🔑 Supported Formats

| Format | Use |
|---|---|
| **Docker/OCI** | Container images |
| **Maven** | Java artifacts |
| **npm** | Node packages |
| **Python** | pip packages |
| **Apt/Yum** | OS packages |
| **Go** | Go modules (via proxy) |
| **Generic** | Any file type |

### 🔑 Artifact Registry Features

- **Regional, multi-regional, or standard locations**
- **Remote repos** (proxy public registries like Docker Hub, Maven Central)
- **Virtual repos** (aggregate multiple repos)
- **CMEK support**
- **VPC-SC support**
- **IAM per repository**
- **Vulnerability scanning** (Artifact Analysis)
- **Labels** for organization

### 🔑 Remote and Virtual Repos

**Remote repo:** Cache upstream public registry (Docker Hub, Maven Central)
- Pull-through cache
- Reduces external dependency
- Authenticates pulls via your repo

**Virtual repo:** Single endpoint combining multiple upstream + local repos
- Simplifies client config
- Useful when migrating or consolidating

### 🔑 Cleanup Policies

Auto-delete old artifacts based on:
- Age (e.g., older than 90 days)
- Tag patterns
- Keeping N most recent versions

**Why:** Old images cost storage; cleanup saves money.

---

## 🎯 Cloud Deploy (Deep Dive)

### 🧠 What Cloud Deploy Does

**Managed CD pipeline** for GKE, Cloud Run, and GCE with:
- Pipeline stages (dev → staging → prod)
- Approval gates
- Canary deployments
- Automated rollbacks
- Release history + audit

### 🔑 Cloud Deploy Concepts

| Concept | Description |
|---|---|
| **Delivery pipeline** | Series of target environments |
| **Target** | A deploy destination (cluster, Cloud Run, VM pool) |
| **Release** | Specific version of app to deploy |
| **Rollout** | Deployment of release to target |
| **Skaffold** | Underlying deployment config (K8s manifests, Cloud Run YAML) |
| **Phases** | Stages within a rollout (canary %, stable) |
| **Approvals** | Manual approval gates |
| **Promotion** | Move release from one target to next (dev → stage → prod) |

### 🔑 Example Pipeline

```yaml
apiVersion: deploy.cloud.google.com/v1
kind: DeliveryPipeline
metadata:
  name: my-pipeline
serialPipeline:
  stages:
    - targetId: dev
    - targetId: staging
      profiles: [staging]
    - targetId: prod
      profiles: [prod]
      strategy:
        canary:
          runtimeConfig:
            kubernetes:
              serviceNetworking:
                service: my-service
          canaryDeployment:
            percentages: [10, 50]
            verify: true
```

### 🔑 Supported Targets

| Target | 2026 Status |
|---|---|
| **GKE** | ✅ First-class |
| **GKE Enterprise (Fleet)** | ✅ Multi-cluster |
| **Cloud Run** | ✅ Supported |
| **Cloud Run Jobs** | ✅ Supported |
| **GCE MIG** | ✅ Supported (2024+) |

### 🔑 Cloud Deploy Features

| Feature | Purpose |
|---|---|
| **Canary deployments** | Gradual traffic shift with verification |
| **Parallel deployments** | Deploy to multiple clusters/regions |
| **Approval gates** | Manual approval before production |
| **Automated rollback** | If error budget exceeded |
| **Verify jobs** | Run tests between phases |
| **Post-deploy hooks** | Integration with monitoring |
| **Audit logging** | Full history of deploys |

### 🔑 Cloud Deploy vs Building Your Own

| Build-Your-Own (e.g., Cloud Build + Helm) | Cloud Deploy |
|---|---|
| Flexible | Opinionated |
| You maintain | Managed |
| Custom | Standard |
| Unknown compliance | Audit-ready |

**Exam rule:** "Managed progressive delivery" → **Cloud Deploy**. "Custom pipeline for complex orchestration" → Cloud Build + custom.

---

## 🎯 Deployment Strategies

### 🧠 The Full Strategy Matrix

| Strategy | Description | Risk | Cost | Rollback Speed |
|---|---|---|---|---|
| **Recreate** | Kill old → start new | ⚠️ Downtime | $ | Slow |
| **Rolling** | Gradually replace old with new | Low | $ | Gradual |
| **Blue/Green** | Full new env → swap | Very Low | $$ (2× infra) | Instant |
| **Canary** | Small % → observe → ramp | Very Low | $ | Quick |
| **Traffic splitting** | Gradual ramp (10% → 50% → 100%) | Low | $ | Quick |
| **Shadow / Dark Launch** | Mirror real traffic to new (no user impact) | Very Low | $$ | Instant |
| **Feature flags** | Toggle features without deploy | Low | $ | Instant |

### 🔑 Rolling Update (Default)

**How:** Replace instances gradually (e.g., 1 at a time).

**Key params:**
- `maxSurge`: Extra instances above desired count during update
- `maxUnavailable`: Instances allowed to be down during update

**Good for:** Stateless services, low-risk changes.

**Native support:** GKE Deployments, MIG rolling updates, App Engine Standard.

### 🔑 Blue/Green Deployment

**How:** Spin up full new environment (Green) → test → switch LB to Green → decommission Blue.

**Pros:**
- Zero downtime
- Instant rollback (switch back)

**Cons:**
- 2× infra cost during cutover
- State/DB migrations need care

**Native support:**
- Cloud Run: deploy new revision, tag, switch traffic
- GKE: two Deployments + Service label swap
- Compute Engine: Two MIGs behind LB

### 🔑 Canary Deployment

**How:** Deploy new version to a small subset (5%) of users → monitor errors/latency → ramp up (25% → 50% → 100%).

**Best for:** Risky changes, critical services.

**Native support:**
- **Cloud Run**: Revisions + traffic splits (`--tag=green --no-traffic`, then `gcloud run services update-traffic`)
- **GKE**: Multiple Deployments + Service with label selector
- **Cloud Deploy**: First-class canary support with automatic progression

### 🔑 Cloud Run Traffic Management

**Revisions:** Each deploy creates a new revision; multiple revisions can coexist.

**Traffic split commands:**
```bash
# Deploy but don't route traffic
gcloud run deploy my-service --tag=v2 --no-traffic

# Shift 10% to v2
gcloud run services update-traffic my-service \
  --to-tags=v2=10

# Shift 100% (promote)
gcloud run services update-traffic my-service \
  --to-latest
```

**Benefits:**
- Built-in canary without extra tooling
- Tagged URLs for testing (`https://v2---my-service.run.app`)
- Instant rollback to older revision

### 🔑 App Engine Traffic Migration

- **Split traffic** by IP, cookie, or random
- **Migrate gradually** with built-in tool
- Similar to Cloud Run revisions

### 🔑 GKE Deployment Strategies

**Rolling update (default):**
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

**Blue/Green:**
- Two separate Deployments (`app-blue`, `app-green`)
- Service selects one based on label
- Swap label to switch traffic

**Canary with Cloud Deploy:**
- Automatic phased rollout with `percentages: [10, 50]`
- Verification between phases

---

## 🛡️ Supply Chain Security

### 🧠 The Modern Supply Chain Problem

Attackers target:
1. **Upstream dependencies** (npm, PyPI, Docker Hub)
2. **CI/CD pipelines** (inject malicious builds)
3. **Artifacts** (swap images post-build)
4. **Runtime** (deploy unauthorized images)

**GCP's answer:** **Software Delivery Shield** — a suite combining Artifact Analysis, Binary Authorization, Cloud Build provenance, and more.

### 🔑 SLSA Framework (Supply Chain Levels for Software Artifacts)

| Level | Requirements | GCP Support |
|---|---|---|
| **SLSA 1** | Documented build process | Cloud Build |
| **SLSA 2** | Versioned source, hosted build | Cloud Build + Cloud Source |
| **SLSA 3** | Non-falsifiable provenance | Cloud Build (auto-attests) |
| **SLSA 4** | Two-party review, hermetic | Cloud Build w/ private pools + signed attestations |

### 🔑 Artifact Analysis (Container Analysis)

**What:** Scans container images for known vulnerabilities (CVEs) and stores metadata.

**Features:**
- **Continuous scanning** — re-scans when new CVEs published
- **On-push scanning** — scans when image pushed to Artifact Registry
- **Occurrences** — findings stored as metadata
- **Notes** — CVE info
- **Integration** with Binary Authorization

**Sources:** NVD, distro advisories (Debian, Ubuntu, RHEL, Alpine), language-specific (Python, npm, Java).

### 🔑 Binary Authorization

**What:** Deploy-time policy enforcement — only deploy images that meet criteria.

**Rules:**
- **Require attestation from N signers** (e.g., build attestation + security scan attestation)
- **Allow from specific repositories** (e.g., only `us-docker.pkg.dev/my-org/*`)
- **Dry-run mode** (log violations but don't block)
- **Breakglass** (emergency deploy with audit)

**Enforcement points:**
- **GKE** (admission controller)
- **GKE Autopilot**
- **Cloud Run**
- **Cloud Run Functions**
- **Anthos clusters**

**Typical attestations:**
1. **Build attestation** (signed by Cloud Build with KMS)
2. **Vulnerability scan attestation** (no CRITICAL CVEs)
3. **QA passed attestation** (signed after tests)

### 🔑 Example Binary Auth Policy

```yaml
defaultAdmissionRule:
  evaluationMode: REQUIRE_ATTESTATION
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
  requireAttestationsBy:
    - projects/my-proj/attestors/build-attestor
    - projects/my-proj/attestors/qa-attestor
clusterAdmissionRules:
  "us-central1.prod-cluster":
    requireAttestationsBy:
      - projects/my-proj/attestors/prod-attestor
```

### 🔑 Cloud Build Provenance

**What:** Automatically generated, signed metadata about **HOW** an image was built.

**Contents:**
- Source repo + commit SHA
- Build pipeline config
- Build environment
- Build ID, timestamp

**Use:** Verify that image came from expected pipeline before deploying.

### 🔑 Software Delivery Shield (combined)

Complete picture:
- Cloud Source / GitHub → 
- Cloud Build (with provenance) → 
- Artifact Registry (with vuln scanning) → 
- Binary Authorization (with attestations) → 
- GKE / Cloud Run (enforced deploy)

---

## 🌉 API Management

### 🧠 API Management Options

| Product | Positioning | Cost |
|---|---|---|
| **Apigee X** | Full enterprise API lifecycle + monetization | $$$$ |
| **API Gateway** | Lightweight for serverless backends | $ |
| **Cloud Endpoints** | Legacy OpenAPI proxy | $ (legacy) |
| **Cloud Service Mesh (Istio/ASM)** | Service-to-service + ingress | $$ |

### 🔑 Apigee X Deep Dive

**Capabilities:**
- **API proxies** with transformation, policies, mediation
- **Developer portal** for external consumers
- **API products** (bundle APIs for partners)
- **Monetization** (billing, plans, quotas)
- **Analytics** (usage, performance, errors)
- **Security** (OAuth, API keys, JWT, mTLS)
- **Rate limiting & quotas**
- **Multi-region HA**
- **Integration with SOAP/REST/GraphQL**

**When to use:**
- Exposing APIs to external partners/customers
- Need monetization or tiered plans
- Complex transformations (SOAP→REST, XML→JSON)
- Developer portal requirements

### 🔑 API Gateway

**Capabilities:**
- Simple API front for **Cloud Run, Cloud Functions, App Engine**
- **OpenAPI spec** based
- **API keys, OAuth2, JWT**
- **Basic quota + rate limiting**
- **Regional**

**When to use:**
- Protecting serverless backends
- No need for monetization or dev portal
- Quick setup

### 🔑 Cloud Endpoints (Legacy)

- **OpenAPI or gRPC** proxy
- ⚠️ **Still supported but exam rarely picks**
- Use API Gateway for new projects

### 🔑 Cloud Service Mesh (formerly Anthos Service Mesh)

- **Istio-based** mesh for microservices
- **mTLS between services**, traffic management, observability
- **Multi-cluster** and **hybrid**
- For **east-west traffic** (service-to-service), not primarily north-south APIs

### 🔑 API Management Decision

```
Public APIs to external partners with monetization?
  → Apigee X

Serverless backend needs simple auth + quota?
  → API Gateway

Internal services need mTLS + observability?
  → Cloud Service Mesh

Legacy OpenAPI proxy in place?
  → Cloud Endpoints (but migrate to API Gateway)
```

---

## 👥 Working with Development Teams

### 🧠 Team Enablement Patterns

| Pattern | Description |
|---|---|
| **Platform team + product teams** | Platform builds golden paths; products use them |
| **Self-service** | Templates, modules, docs for teams to provision themselves |
| **DevOps teams** | Product teams own end-to-end (build + run) |
| **SRE embed** | SRE works within product team, transfers practices |

### 🔑 Golden Paths / Paved Road

**Concept:** Pre-built, opinionated paths to production. Teams follow them to get consistent results.

**Examples:**
- **Terraform modules** for standard app (GKE + Cloud SQL + LB + monitoring)
- **Cloud Build templates** for common pipelines
- **Approved base images** in Artifact Registry
- **Cloud Deploy templates** for canary delivery

### 🔑 IAM for CI/CD

**Service account scopes:**
- Build SA: Only what's needed to build/test
- Deploy SA: Only what's needed to deploy to target
- Never share SAs across pipelines

**Per-environment SAs:**
- `deploy-dev-sa` with dev project perms
- `deploy-prod-sa` with prod project perms (stricter; approval required)

### 🔑 Environment Separation

| Env | Project | Key Controls |
|---|---|---|
| **Dev** | `my-app-dev` | Developers = Owner/Editor; fast iteration |
| **Staging** | `my-app-staging` | Only CI/CD deploys |
| **Prod** | `my-app-prod` | Tightly controlled; no direct user access; Binary Auth enforced |

### 🔑 Developer Experience (DX)

- **Cloud Code** in VS Code / IntelliJ — deploy from IDE
- **Cloud Shell** — always-ready CLI environment
- **Skaffold** for fast GKE dev loop
- **Cloud Workstations** — managed dev environments

---

## 🔐 Secrets & Configuration

### 🧠 Secrets in CI/CD

**DON'T:**
- ❌ Hardcode in `cloudbuild.yaml`
- ❌ Put in Git (even private)
- ❌ Use user tokens
- ❌ Use long-lived SA keys

**DO:**
- ✅ **Secret Manager** for build-time secrets
- ✅ **Workload Identity Federation** for external CI/CD → GCP
- ✅ **KMS-encrypted** in transit
- ✅ **Short-lived tokens** from metadata server / WIF

### 🔑 Secret Manager + Cloud Build

```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '--build-arg', 'API_KEY=$$API_KEY', '.']
    secretEnv: ['API_KEY']
availableSecrets:
  secretManager:
    - versionName: projects/PROJECT/secrets/api-key/versions/latest
      env: API_KEY
```

### 🔑 Configuration Management Tiers

| Tier | Tool |
|---|---|
| **Non-sensitive config** | Env vars, ConfigMaps (GKE), Cloud Run env vars |
| **Sensitive secrets** | Secret Manager (mount or runtime fetch) |
| **TLS certs** | Certificate Manager |
| **Feature flags** | Firebase Remote Config, LaunchDarkly, custom |

---

## 🧪 Testing Strategies

### 🧠 The Testing Pyramid in Cloud Build

```
       /\
      /E2E\        ← Few, slow, expensive (run in staging)
     /------\
    /Integration\  ← Some, medium speed (in Cloud Build)
   /------------\
  /    Unit      \ ← Many, fast, cheap (first in pipeline)
 /________________\
```

### 🔑 Testing Stages in Pipeline

| Stage | What | Where |
|---|---|---|
| **Lint / Format** | Style checks | Cloud Build early step |
| **Unit tests** | Fast, isolated | Cloud Build |
| **Security scan** | SAST, dependency scan | Cloud Build + Artifact Analysis |
| **Build image** | Docker build | Cloud Build |
| **Integration tests** | DB, external APIs | Cloud Build (with Private Pool if VPC needed) |
| **Deploy to staging** | Ephemeral env | Cloud Deploy |
| **E2E tests** | Full-stack | Against staging |
| **Deploy canary to prod** | 5% traffic | Cloud Deploy |
| **Verify canary** | Run smoke tests, monitor SLOs | Cloud Deploy verify job |
| **Promote to prod** | 100% traffic | Cloud Deploy |

### 🔑 Preview Environments

- Spin up **ephemeral env per PR**
- Use Cloud Run (scale-to-zero) + Firebase Hosting for cheap preview
- Cloud Deploy "verify" phase can test here

---

## 🚨 Exam Traps & Tricks

### ⚠️ Top 15 Traps in Domain 5

1. **Container Registry** → ⚠️ **DEPRECATED**, always use **Artifact Registry**
2. **"Progressive delivery on GKE"** → **Cloud Deploy**, NOT homegrown Cloud Build scripts
3. **"Enforce only signed images"** → **Binary Authorization with attestations**
4. **"Cloud Build needs private Cloud SQL access"** → **Private Pool**, not default
5. **"External API to partners with monetization"** → **Apigee X**, NOT API Gateway
6. **"Simple API key + quota on Cloud Run"** → **API Gateway**, NOT Apigee (overkill)
7. **"Secrets in cloudbuild.yaml"** → WRONG; use **Secret Manager**
8. **"GitHub Actions → GCP deploy"** → **Workload Identity Federation**, NOT SA keys
9. **"Blue/Green needs 2× infra"** → true; Canary is cheaper for same risk
10. **"Instant rollback"** → **Blue/Green** (switch back) or Cloud Run revision
11. **"5% canary"** → Cloud Run traffic splitting OR Cloud Deploy canary
12. **"Continuous CVE scanning"** → **Artifact Analysis** (continuous mode)
13. **"Supply chain security"** → **Software Delivery Shield** (Artifact Analysis + Binary Auth + provenance)
14. **"Cloud Endpoints for new API"** → use **API Gateway** (Endpoints is legacy)
15. **"Cloud Deploy for Cloud Run"** → ✅ supported; not GKE-only

### ⚠️ "Sounds Right but Isn't"

| Seems Right | Actually Right | Why |
|---|---|---|
| Cloud Source Repositories for modern source | GitHub/GitLab + CSR mirror | Teams prefer GitHub/GitLab ecosystem |
| Deployment Manager | Terraform via Infrastructure Manager | DM deprecated |
| Jenkins on GCE | Cloud Build | Managed beats self-hosted |
| GCR for containers | Artifact Registry | GCR deprecated |
| Custom canary scripts | Cloud Deploy | Managed CD with built-in canary |
| API Gateway for monetization | Apigee X | API GW doesn't do monetization |
| Cloud Endpoints for new | API Gateway | Endpoints legacy |
| Default Cloud Build SA | Custom minimum-perms SA | Default is too broad |

---

## 🔑 Decision Frameworks

### 🎯 Deployment Strategy Picker

```
How risky is the change?
  ├── Very risky / new feature → Canary (5% → 25% → 50% → 100%)
  ├── Medium → Rolling update (default)
  └── Infra change, zero downtime → Blue/Green

Need instant rollback?
  ├── Yes → Blue/Green OR Cloud Run revisions
  └── Gradual OK → Rolling or canary

Budget / infra cost tight?
  └── Canary or Rolling (not Blue/Green)

Already on Cloud Run?
  └── Traffic splitting (native, free)

Already on GKE?
  └── Cloud Deploy canary (best) or custom rolling
```

### 🎯 CI/CD Tooling Picker

```
Need managed CI on GCP?
  └── Cloud Build

Need managed CD with progressive delivery?
  └── Cloud Deploy

Need artifact storage?
  └── Artifact Registry (NOT Container Registry)

Need vulnerability scanning?
  └── Artifact Analysis (continuous)

Need to enforce deploy policies?
  └── Binary Authorization

External CI/CD (GitHub Actions) deploying to GCP?
  └── Workload Identity Federation

Need private resource access in build?
  └── Private Pool
```

### 🎯 API Management Picker

```
Expose APIs externally with monetization + dev portal?
  └── Apigee X

Fronting Cloud Run / Cloud Functions with simple auth + quota?
  └── API Gateway

Service-to-service traffic with mTLS?
  └── Cloud Service Mesh

Existing OpenAPI proxy (legacy)?
  └── Cloud Endpoints (but migrate to API Gateway)
```

---

## 📝 Mnemonics & Memory Hacks

### 🧠 "CABD" (CI/CD Stack)
**C**loud Build → **A**rtifact Registry → **B**inary Authorization → **D**eploy (Cloud Deploy)

### 🧠 "BROC" (Deployment Types)
**B**lue-green, **R**olling, **O**bservational (shadow), **C**anary

### 🧠 "Apigee vs Gateway vs Endpoints"
- **A**pigee = **A**ll the features (enterprise + monetization)
- **G**ateway = **G**o simple (serverless)
- **E**ndpoints = **E**xpiring (legacy)

### 🧠 "SLSA 1-4"
- Level 1: **S**ource documented
- Level 2: **L**abeled source, hosted build
- Level 3: **S**igned provenance (non-falsifiable)
- Level 4: **A**ctivity reviewed, hermetic

### 🧠 "WIF > KEY"
Workload Identity Federation for external CI/CD always beats downloaded keys.

### 🧠 "BAP" (Binary Auth basics)
**B**inary **A**uth requires **P**olicy + attestations + attestors

### 🧠 "Private Pool = VPC Access"
Remember: **Default pool = public**; if build needs private resources, use **Private Pool**.

---

## ✅ Practice Checkpoint

### Q1. Container registry for new project
**Scenario:** Starting a new microservices project. Need container image storage with vulnerability scanning.

**Options:**
- A) Container Registry (GCR)
- B) Artifact Registry with Artifact Analysis enabled
- C) Docker Hub
- D) GCS bucket

✅ **Answer: B** — **GCR is deprecated**; **Artifact Registry** is the successor. Artifact Analysis provides continuous vuln scanning.

---

### Q2. Canary on Cloud Run
**Scenario:** Deploy new version of Cloud Run service with 5% canary traffic, monitor, then promote.

**Options:**
- A) Deploy and switch DNS manually
- B) Deploy new revision with `--no-traffic`, use traffic splitting 5% → monitor → `--to-latest`
- C) Blue/Green with 2 services
- D) Cloud Deploy with custom integration

✅ **Answer: B** — **Cloud Run revisions + traffic splitting** is native, no additional tooling needed.

---

### Q3. Enforce signed images
**Scenario:** Security mandates: only images built by our Cloud Build pipeline AND passing security scan can deploy to prod GKE.

**Options:**
- A) IAM on GKE cluster
- B) Binary Authorization with 2 attestors (build + scan); enforce in prod cluster
- C) VPC Service Controls
- D) Pod Security Policies

✅ **Answer: B** — **Binary Authorization** + multiple attestors is the standard supply chain answer.

---

### Q4. CI/CD from GitHub
**Scenario:** GitHub Actions workflow deploys to GCP. Security bans long-lived credentials.

**Options:**
- A) Download SA JSON key, GitHub encrypted secret
- B) Create user account
- C) Workload Identity Federation with GitHub OIDC
- D) Store key in Secret Manager

✅ **Answer: C** — **WIF** uses GitHub's OIDC tokens; no keys ever created.

---

### Q5. Cloud Build accessing private Cloud SQL
**Scenario:** Cloud Build integration tests need to connect to Cloud SQL with private IP only.

**Options:**
- A) Default pool
- B) Cloud Build Private Pool in VPC
- C) Cloud SQL external IP with authorized networks
- D) SSH tunnel

✅ **Answer: B** — **Private Pool** runs builds in your VPC for private resource access.

---

### Q6. API monetization
**Scenario:** Exposing APIs to 500 external partners with tiered plans, usage-based billing, developer portal.

**Options:**
- A) API Gateway
- B) Cloud Endpoints
- C) Apigee X
- D) Cloud Load Balancer

✅ **Answer: C** — **Apigee** is the only option with monetization + dev portal.

---

### Q7. Lightweight API fronting
**Scenario:** Cloud Run backend serving JSON API. Need API keys, 1000 req/min quota per key.

**Options:**
- A) Apigee
- B) API Gateway
- C) Cloud Endpoints (legacy)
- D) Cloud Armor

✅ **Answer: B** — **API Gateway** is purpose-built for serverless backends; Apigee is overkill.

---

### Q8. Rollback speed
**Scenario:** Production incident: deploy 5 min ago is causing 500 errors. Need instant rollback.

**Options:**
- A) Rolling update back to old version (5 min)
- B) Cloud Run traffic shift back to previous revision (seconds)
- C) Manual patch file
- D) Wait for traffic to drain

✅ **Answer: B** — **Cloud Run revisions** allow instant traffic shift back to last-known-good.

---

### Q9. Continuous CVE monitoring
**Scenario:** Images pushed yesterday — today a new CRITICAL CVE in openssl was disclosed. Want to know if it affects our images.

**Options:**
- A) Re-run scans manually
- B) Artifact Analysis continuous scanning
- C) Cloud Armor
- D) Security Command Center only

✅ **Answer: B** — **Artifact Analysis** continuously re-scans as new CVEs are published.

---

### Q10. Multi-env progressive delivery
**Scenario:** Need pipeline: dev (auto) → staging (auto after tests) → prod (canary 10% → 50% → 100% with approval).

**Options:**
- A) Cloud Build scripts
- B) Cloud Deploy delivery pipeline with stages + canary strategy + approvals
- C) Jenkins
- D) Manual gcloud deploys

✅ **Answer: B** — **Cloud Deploy** is purpose-built for multi-stage pipelines with canary + approvals.

---

### Q11. Secrets in build
**Scenario:** Cloud Build step needs API key to call external service.

**Options:**
- A) Hardcode in cloudbuild.yaml
- B) Store in Secret Manager, reference via `availableSecrets`
- C) Env var in Cloud Build trigger
- D) GCS file

✅ **Answer: B** — **Secret Manager integration** in Cloud Build via `availableSecrets` is the secure pattern.

---

### Q12. Supply chain compliance
**Scenario:** Enterprise wants SLSA Level 3 compliance: non-falsifiable build provenance, signed attestations.

**Options:**
- A) Cloud Build with build provenance + Binary Authorization with attestations + Artifact Registry
- B) Jenkins on GCE
- C) Docker Hub
- D) Manual signing

✅ **Answer: A** — **Cloud Build generates signed provenance**; **Binary Auth** enforces policy. This is the Software Delivery Shield stack.

---

### Q13. Zero-downtime deploys for stateful
**Scenario:** Upgrading stateful GKE app with rolling schema migration. Need zero downtime.

**Options:**
- A) Recreate strategy
- B) Blue/Green with DB migration run before green goes live
- C) Canary
- D) Shadow deployment

✅ **Answer: B** — **Blue/Green** gives controlled schema migrations between envs; but schema changes need backward-compatibility ("expand-contract" pattern).

---

### Q14. Artifact Registry with private network
**Scenario:** GKE in private cluster must pull images from Artifact Registry; no internet egress allowed.

**Options:**
- A) Public Artifact Registry
- B) Artifact Registry with Private Google Access enabled on GKE subnet
- C) Push to GCS instead
- D) Allow internet egress

✅ **Answer: B** — **Private Google Access** lets private resources reach Google APIs (including Artifact Registry) without internet.

---

### Q15. Test isolation
**Scenario:** Integration tests in Cloud Build mutate a shared test DB, causing flakiness.

**Options:**
- A) Accept flakiness
- B) Spin up ephemeral Cloud SQL per build (expensive)
- C) Use emulators (Firestore emulator, Spanner emulator) in build steps
- D) Run tests in production

✅ **Answer: C** — **Emulators** give isolated, fast, free integration tests in Cloud Build.

---

## 🔄 2025–2026 Changes

| Change | Impact |
|---|---|
| **Container Registry deprecated** | Always Artifact Registry |
| **Cloud Deploy** | Now supports Cloud Run, Cloud Run Jobs, GCE MIG (not just GKE) |
| **Cloud Deploy parallel deploys** | Multi-target multi-region support |
| **Software Delivery Shield** | Unified supply chain security branding |
| **SLSA Level 3 achievable on GCP** | Cloud Build generates signed provenance automatically |
| **Workload Identity Federation expanded** | Default for external CI/CD |
| **Binary Authorization for Cloud Run** | Expanded beyond GKE |
| **Cloud Endpoints legacy** | Use API Gateway |
| **Apigee Hybrid** | Run Apigee runtime on GKE/on-prem |
| **Gemini Code Assist / Duet AI** | AI-assisted CI/CD config authoring |
| **Cloud Workstations** | Managed dev environments |
| **Artifact Registry remote and virtual repos** | Proxy Docker Hub etc. |
| **Deployment Manager deprecated** | Terraform / Infrastructure Manager only |

---

## 🎯 Final Domain 5 Checklist

Before exam day, you should be able to:

- [ ] Name the modern CI/CD stack: Cloud Build → Artifact Registry → Artifact Analysis → Binary Auth → Cloud Deploy
- [ ] Know Container Registry is deprecated
- [ ] Know Cloud Build Private Pool for VPC access
- [ ] Know WIF for external CI/CD (no keys!)
- [ ] Know Secret Manager integration in Cloud Build
- [ ] Know Cloud Deploy targets: GKE, Cloud Run, Cloud Run Jobs, GCE MIG
- [ ] Know canary strategies: Cloud Run revisions, Cloud Deploy canary, GKE multi-deployment
- [ ] Know Binary Authorization policy: attestations + enforcement mode
- [ ] Know Artifact Analysis continuous scanning
- [ ] Know SLSA 1-4 progression
- [ ] Know Apigee X (enterprise) vs API Gateway (simple) vs Endpoints (legacy)
- [ ] Know when to pick Blue/Green vs Rolling vs Canary
- [ ] Know Cloud Source Repos but GitHub/GitLab are also valid
- [ ] Know IAM separation per environment (dev/staging/prod SAs)

> **Domain 5 is the most modern domain — if you keep up with DevOps trends, you'll feel at home here.** 🚀
