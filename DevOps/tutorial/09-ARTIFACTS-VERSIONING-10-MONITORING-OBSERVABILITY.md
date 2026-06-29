# Module 9: Artifacts & Versioning

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/09-ARTIFACTS-VERSIONING-10-MONITORING-OBSERVABILITY.md)


## Module Overview
- **Concept Core:** 60 min | **Hands-On Extension:** 60 min + Lab (90 min)
- **Real-World Scenarios:** FastCart Docker versioning; TenantHub NuGet feed management
- **Key Patterns:** Semantic versioning, artifact immutability, feed promotion, retention policies

---

## 🟢 Concept Core: Artifact Lifecycle

### What is an Artifact?

**Artifact** = Deployable output from a build (Docker image, .NET DLL, JAR, npm package)

```
CODE → BUILD → ARTIFACT
              ├── Docker image (1.2.3-abc1234)
              ├── .NET assembly (Checkout.dll v1.2.3)
              ├── npm package (checkout-service@1.2.3)
              └── JAR file (checkout-1.2.3.jar)
```

### Versioning Strategy: Semantic Versioning

**Format:** MAJOR.MINOR.PATCH

```
1.2.3
├── MAJOR (1): Breaking changes (old clients break)
│   Example: 1.0.0 → 2.0.0 (API endpoints removed)
│
├── MINOR (2): New features, backward compatible
│   Example: 1.1.0 → 1.2.0 (new endpoint added)
│
└── PATCH (3): Bug fixes
    Example: 1.2.2 → 1.2.3 (security patch)

Rules:
├── Increment MAJOR when making breaking changes
├── Increment MINOR when adding features (backward compatible)
├── Increment PATCH when fixing bugs
└── Reset MINOR and PATCH to 0 when MAJOR increments

Pre-release versions:
├── 1.2.3-alpha.1 (early development)
├── 1.2.3-beta.1 (feature complete, testing)
└── 1.2.3-rc.1 (release candidate, ready for prod)

Examples:
├── 1.0.0 → 1.0.1 (patch: typo fix)
├── 1.0.0 → 1.1.0 (minor: new checkout feature)
└── 1.1.0 → 2.0.0 (major: completely new checkout API)
```

---

## 🟢 Real-World Scenario 1: FastCart Docker Versioning

```
Build #100: Code commit abc1234
├── Build artifact: docker image checkout:1.2.3
├── Tag with version: ghcr.io/fastcart/checkout:1.2.3
├── Tag with commit: ghcr.io/fastcart/checkout:abc1234
├── Tag as latest: ghcr.io/fastcart/checkout:latest
└── Image immutable (can't change 1.2.3 once pushed)

Deploy to Staging:
├── Pull: checkout:1.2.3
├── QA tests
├── Approves ✅

Deploy to Production:
├── Pull: same checkout:1.2.3 (bit-for-bit identical)
├── Blue-green swap
└── Revenue tracking

If issue in production:
├── Rollback: Pull and deploy checkout:1.2.2
├── Quick recovery (same image was tested before)

Retention policy:
├── Keep last 10 tagged versions (1.2.3, 1.2.2, ...)
├── Keep latest for 30 days
├── Delete pre-release after 7 days (clutter)
└── Cost: Less storage, faster access to recent versions
```

---

## 🟢 Real-World Scenario 2: TenantHub NuGet Feeds

```
Artifact Management:
├── NuGet Feed (private Azure Artifacts)
│   ├── Dev feed (all builds, pre-release, 7-day retention)
│   ├── Staging feed (tested builds, 30-day retention)
│   └── Prod feed (production-approved, 1-year retention)
│
└── Package lifecycle:
    ├── Build 1: 2.1.0-build.001 → Dev feed
    ├── Tests pass → Copy to Staging feed
    ├── QA approves → Copy to Prod feed (immutable)
    └── 1 year retention in Prod feed (historical audit)

Dependency management:
├── Service A depends on SharedLib v2.1.0 (from Prod feed)
├── Service B depends on SharedLib v2.0.5 (from Prod feed)
├── If SharedLib v2.0.5 has bug: Only rebuild Service B (targeted fix)
└── No "shared dependency hell"
```

---

## 🔷 Hands-On Extension: Docker Versioning & NuGet Feeds

### Part A: Docker Image Versioning

```yaml
# .github/workflows/build-and-tag.yml
name: Build and Tag Docker Image

on: [push]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: actions/checkout@v3
      
      # Determine version from git tags or default
      - name: Generate version
        id: version
        run: |
          if [[ $GITHUB_REF == refs/tags/v* ]]; then
            VERSION=${GITHUB_REF#refs/tags/}
          else
            VERSION=$(date +%Y%m%d-%H%M%S)-${GITHUB_SHA::7}
          fi
          echo "version=$VERSION" >> $GITHUB_OUTPUT
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.version.outputs.version }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          labels: |
            org.opencontainers.image.version=${{ steps.version.outputs.version }}
            org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
```

**Tags created:**
- `ghcr.io/owner/repo:1.2.3` (semantic version)
- `ghcr.io/owner/repo:latest` (always points to newest)
- `ghcr.io/owner/repo:abc123` (commit SHA for traceability)

---

### Part B: NuGet Feed Promotion

```yaml
# azure-pipelines.yml
trigger:
  - main

variables:
  buildConfiguration: 'Release'
  version: '2.1.0'  # Hardcoded or generate dynamically

stages:
  - stage: Build
    jobs:
      - job: BuildNuGet
        steps:
          - task: UseDotNet@2
            inputs:
              packageType: sdk
              version: 7.0.x
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'restore'
              projects: '**/*.csproj'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
              arguments: '--configuration $(buildConfiguration)'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'pack'
              packagesToPack: '**/SharedLib.csproj'
              versioningScheme: 'byEnvVar'
              versionEnvVar: 'version'
              outputDir: '$(Build.ArtifactStagingDirectory)'
          
          - task: NuGetCommand@2
            inputs:
              command: 'push'
              packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg'
              nuGetFeedType: 'internal'
              publishVstsFeed: 'DevFeeds/dev'  # Push to DEV feed
            displayName: 'Push to dev feed'

  - stage: PromoteToStaging
    dependsOn: Build
    condition: succeeded()
    jobs:
      - job: PromoteJob
        steps:
          - task: NuGetCommand@2
            inputs:
              command: 'push'
              nuGetFeedType: 'internal'
              publishVstsFeed: 'DevFeeds/staging'
            displayName: 'Promote to staging feed'

  - stage: PromoteToProduction
    dependsOn: PromoteToStaging
    condition: succeeded()
    jobs:
      - job: ManualPromotion
        pool: server
        steps:
          - task: ManualValidation@0
            inputs:
              notifyUsers: 'release-managers'
              instructions: 'Approve to promote to production'
          
          - task: NuGetCommand@2
            inputs:
              command: 'push'
              nuGetFeedType: 'internal'
              publishVstsFeed: 'DevFeeds/production'
            displayName: 'Promote to production feed'
```

**Feed promotion:**
```
Build → Dev Feed → Staging Feed → (Approval) → Production Feed
```

---

## 🔷 Lab: Version and Publish Artifacts

### Objectives
- ✅ Create semantic version
- ✅ Build Docker image with version tag
- ✅ Push to registry
- ✅ Create NuGet package
- ✅ Push to multiple feeds
- ✅ Verify immutability

### Prerequisites
- GitHub repo with Dockerfile
- Or .NET project with .csproj
- 90 minutes

### Lab Steps (Docker)

#### Step 1: Create Dockerfile

```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 8000
CMD ["python", "app.py"]
```

#### Step 2: Set Up GitHub Container Registry

```bash
# Authenticate (local testing)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Build locally
docker build -t ghcr.io/username/repo:1.2.3 .

# Push
docker push ghcr.io/username/repo:1.2.3
```

#### Step 3: Create CI Workflow

Add `.github/workflows/publish.yml`:

```yaml
name: Publish

on:
  push:
    tags:
      - 'v*'

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      packages: write
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Extract version from tag
        id: version
        run: |
          VERSION=${GITHUB_REF#refs/tags/v}
          echo "version=$VERSION" >> $GITHUB_OUTPUT
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.version.outputs.version }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```

#### Step 4: Create Git Tag and Push

```bash
git tag -a v1.2.3 -m "Release version 1.2.3"
git push origin v1.2.3
```

Workflow runs and publishes image.

#### Step 5: Verify in Registry

```bash
# List tags
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/username/repo/packages/container/repo/versions
```

### Validation Checklist

- ✅ Dockerfile created
- ✅ Semantic version assigned (1.2.3)
- ✅ Docker image built
- ✅ Image pushed to registry
- ✅ Multiple tags (version, latest, commit SHA)
- ✅ Image pulled and ran successfully

---

## 🟢 Assessment: Versioning & Artifacts

> **Question 1:** FastCart releases 1.2.0. Next release removes a payment API endpoint. What version?
> - A) 1.2.1
> - B) 1.3.0
> - C) 2.0.0
> - D) 1.0.0
> 
> **Answer:** C — Removing an API endpoint is a **breaking change**. Old clients break. Increment MAJOR.

> **Question 2:** Artifact immutability means...
> - A) You can overwrite version 1.2.3 if you need to fix a bug
> - B) Version 1.2.3 can never change; if bug found, release 1.2.4
> - C) Artifacts expire automatically
> - D) You don't need versioning
> 
> **Answer:** B — Once 1.2.3 is deployed to production, it must never change. Consistency is critical. Bug fix = new version 1.2.4.

---

## Next Steps

✅ **You now understand:**
- Semantic versioning
- Artifact lifecycle and promotion
- Docker image tagging
- Feed management

🔜 **Next module:** Monitoring & Observability (dashboards, alerts, DORA metrics)

---

## References

- [Semantic Versioning](https://semver.org/)
- [Container Registry Best Practices](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-best-practices)
- [Azure Artifacts Feeds](https://learn.microsoft.com/en-us/azure/devops/artifacts/concepts/feeds)
---

# Module 10: Monitoring, Logging & Observability

## Module Overview
- **Concept Core:** 60 min | **Hands-On Extension:** 60 min + Lab (90 min)
- **Real-World Scenarios:** FastCart deployment health checks; TenantHub DORA metrics
- **Key Patterns:** Health checks, dashboards, alerts, DORA metrics, post-deployment validation

---

## 🟢 Concept Core: DevOps Metrics & Observability

### The Problem Without Monitoring

```
10:15 AM: Deploy v1.5.0 to production
10:16 AM: Deployment completes, "Success!" message
10:16 – 11:00 AM: Nobody checks anything
11:00 AM: Customer complains: "Checkout is broken"
11:05 AM: Support team starts investigating
11:30 AM: Issue found in production logs
11:45 AM: Rollback initiated
12:00 PM: Problem fixed
Revenue lost: $25k (1 hour downtime)
```

### The Solution: Automated Monitoring

```
10:15 AM: Deploy v1.5.0
10:16 AM: Deployment completes
10:17 AM: Monitoring system automatically checks:
       ├── App is responding (HTTP 200)?
       ├── Database queries < 500ms?
       ├── Error rate < 1%?
       ├── Checkout success rate normal?
       └── All checks pass ✅

10:17 – 11:00 AM: Continuous monitoring
11:00 AM: If any metric degrades, alert fires immediately
11:00:30 AM: On-call engineer gets notified via Slack/PagerDuty
11:01 AM: Automatic rollback triggered
11:02 AM: Issue resolved (customer sees v1.4.0 again)
Total downtime: 1 minute
Cost: Near-zero (automatic recovery)
```

---

## 🟢 DORA Metrics: Measure DevOps Success

**DORA** = DevOps Research and Assessment (Google Cloud)

Four key metrics that correlate with business success:

| Metric | What | Good | Fast DevOps |
|--------|------|------|-------------|
| **Deployment Frequency** | How often deploy to prod? | 1/month | 1/day (or more) |
| **Lead Time for Change** | Commit → prod | 6 months | < 1 hour |
| **Change Failure Rate** | % of deployments that need hotfix | 30% | < 15% |
| **Mean Time to Recovery** | How long to fix a production issue | 8 hours | < 1 hour |

**Why they matter:**
- Teams with high deployment frequency, short lead time, low failure rate are more successful
- These metrics aren't competitive; they're internal benchmarks
- Improvement = more customer value faster

---

## 🟢 Real-World Scenario 1: FastCart Health Checks

```
Post-Deployment Validation:
├── 10:17 AM: Deployment complete
├── Run health check suite:
│   ├── [200 OK] GET /health (API responding)
│   ├── [OK] SELECT 1 FROM test_table (DB connected)
│   ├── [OK] Stripe test transaction (payment processor connected)
│   ├── [OK] Checkout /api/v1/checkout exists (endpoint available)
│   └── [LATENCY 45ms] Average response time (within baseline)
│
├── If all pass: "Deployment healthy" notification
└── If any fail: "Deployment failed health check" alert + auto-rollback

Monitoring Dashboard (real-time):
├── Requests/sec (should be stable)
├── Errors/sec (spike = problem)
├── Latency percentiles (p50, p95, p99)
├── Checkout success rate (should be 95%+)
└── Revenue/hour trending (manual feature = revenue increase visible)

Alert rules:
├── Error rate > 5% → Slack alert
├── Latency p99 > 1 sec → Slack alert + page engineer
├── Checkout success rate < 90% → Critical alert + auto-rollback
```

---

## 🟢 Real-World Scenario 2: TenantHub DORA Tracking

```
Monthly DORA Metrics Dashboard:
├── Deployment frequency: 12 deployments (was 4 last month) ↑
├── Lead time for change: 4 hours (was 8 hours) ↑
├── Change failure rate: 10% (was 20%) ↓
├── MTTR: 30 min (was 2 hours) ↓

Team wins:
├── Shipping features faster (12 deployments/month)
├── Shorter code-to-production (4 hours from commit)
├── Fewer bugs making it to production (10% fix rate)
├── Quicker recovery when issues happen (30 min)

Business impact:
├── More features shipped = customers happier
├── Faster feedback loops = better quality
├── Less downtime = SLA compliance
└── Faster recovery = trust maintained
```

---

## 🔷 Hands-On Extension: Health Checks & Monitoring

### Part A: Post-Deployment Health Checks

```yaml
# .github/workflows/deploy-and-validate.yml
name: Deploy with Health Checks

on: [workflow_dispatch]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: 'production'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to App Service
        run: |
          az webapp up -n myapp -g mygroup --runtime "PYTHON|3.11"
      
      - name: Wait for app to start
        run: sleep 30
      
      - name: Health check
        run: |
          set -e
          ENDPOINT="https://myapp.azurewebsites.net"
          
          # Test 1: HTTP 200
          echo "Checking app responds..."
          curl -f $ENDPOINT/health || exit 1
          echo "✅ App is responding"
          
          # Test 2: Database
          echo "Checking database..."
          curl -f $ENDPOINT/api/db-check || exit 1
          echo "✅ Database is connected"
          
          # Test 3: Performance
          echo "Checking latency..."
          START=$(date +%s%N)
          curl -f $ENDPOINT/api/checkout > /dev/null
          END=$(date +%s%N)
          LATENCY=$(( (END - START) / 1000000 ))
          echo "Latency: ${LATENCY}ms"
          if [ $LATENCY -lt 1000 ]; then
            echo "✅ Latency acceptable"
          else
            echo "❌ Latency too high!"
            exit 1
          fi
      
      - name: If health check fails, rollback
        if: failure()
        run: |
          echo "Deployment health check failed! Rolling back..."
          az webapp deployment slot swap \
            -g mygroup \
            -n myapp \
            --slot staging
          exit 1
```

---

### Part B: Azure Monitor Alerts

```yaml
# azure-pipelines.yml (configure alerts after deploy)
steps:
  - task: AzureMonitorAlerts@0
    inputs:
      azureSubscription: 'Azure'
      resourceGroup: 'mygroup'
      resourceName: 'myapp'
      alertName: 'HighErrorRate'
      metricName: 'ErrorRate'
      condition: 'GreaterThan'
      threshold: 5
      actionGroupId: '/subscriptions/.../actionGroups/AlertOps'
    displayName: 'Configure error rate alert'
```

---

## 🔷 Lab: Build Monitoring Dashboard

### Objectives
- ✅ Create health check endpoint
- ✅ Deploy and validate
- ✅ Create monitoring dashboard
- ✅ Set up alerts
- ✅ Test alert triggering

### Prerequisites
- App Service
- Azure Monitor (free tier)
- 90 minutes

### Lab Steps

#### Step 1: Add Health Endpoint to App

**Python:**
```python
# app.py
from flask import Flask, jsonify
import psutil
import time

app = Flask(__name__)

@app.route('/health')
def health():
    return jsonify({
        'status': 'ok',
        'timestamp': time.time(),
        'memory_percent': psutil.virtual_memory().percent
    })

@app.route('/api/checkout', methods=['POST'])
def checkout():
    # ... checkout logic
    return jsonify({'status': 'success'})
```

#### Step 2: Deploy

```bash
az webapp up -n healthapp -g mygroup --runtime "PYTHON|3.11"
```

#### Step 3: Test Health Endpoint

```bash
curl https://healthapp.azurewebsites.net/health
# Response: {"status": "ok", "timestamp": 1234567890, "memory_percent": 45}
```

#### Step 4: Create Dashboard

1. Azure Portal → **Monitor** → **Dashboards**
2. Click **New dashboard**
3. Add tiles:
   - App Service "Requests"
   - "Server errors"
   - "Response time"
4. Set time range: Last 1 hour

#### Step 5: Add Alert Rule

1. **Monitor** → **Alerts** → **New alert rule**
2. Scope: Select App Service
3. Condition: "Requests" > 100/min
4. Action: Send email notification
5. Create

#### Step 6: Trigger Alert (Optional)

```bash
# Simulate high traffic (generates alert)
for i in {1..200}; do
  curl https://healthapp.azurewebsites.net/health &
done
wait
```

### Validation Checklist

- ✅ Health endpoint returns status
- ✅ Dashboard displays metrics
- ✅ Alert rule configured
- ✅ Can view historical data
- ✅ Alert triggered when threshold exceeded

---

## 🟢 Assessment: Monitoring

> **Question 1:** FastCart deploys at 2 PM. Health check fails at 2:01 PM. What happens?
> - A) Nothing, deployment completes
> - B) Automatic rollback to previous version (zero customer impact)
> - C) Waiting for manual investigation (1 hour+ downtime)
> - D) Customer gets error message
> 
> **Answer:** B — Health check failure → automatic rollback. Deployment is "undone" instantly, keeping customers safe.

> **Question 2:** Which DORA metric measures code quality?
> - A) Deployment frequency
> - B) Lead time for change
> - C) Change failure rate (% requiring hotfix)
> - D) All of the above
> 
> **Answer:** C — Change failure rate (% of deployments needing hotfix) directly measures code quality/testing effectiveness.

---

## Next Steps

✅ **You now understand:**
- Health checks and automated validation
- Post-deployment monitoring
- DORA metrics and their business impact
- Alert configuration and response

🔜 **Next modules:** 11–12 (Real-World Patterns & Capstone)

---

## References

- [DORA Metrics](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-devops-performance)
- [Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/)
- [SRE Book: Monitoring Distributed Systems](https://sre.google/resources/books/)
---

## 📝 Chapter Quiz

**Question 1:** What is the core objective of Artifacts Versioning & Monitoring Observability in modern DevOps practices?

* [ ] To increase manual work
* [x] To automate, streamline, and increase the reliability of software delivery
* [ ] To eliminate code repositories
* [ ] To replace all developers

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** To automate, streamline, and increase the reliability of software delivery

**Explanation:** Artifacts Versioning & Monitoring Observability focuses on improving agility, reducing delivery friction, and ensuring operational reliability.
</details>

**Question 2:** Which file format is standard for defining automated workflows and pipelines associated with Artifacts Versioning & Monitoring Observability?

* [ ] XML
* [x] YAML (.yml or .yaml)
* [ ] INI
* [ ] CSV

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** YAML (.yml or .yaml)

**Explanation:** YAML is the industry standard for defining pipeline-as-code configurations in GitHub Actions and Azure DevOps.
</details>

**Question 3:** What is Continuous Integration (CI)?

* [ ] Deploying directly to customers without testing
* [x] Automatically building and testing code changes upon commit
* [ ] Writing documentation manually
* [ ] Managing database backups

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Automatically building and testing code changes upon commit

**Explanation:** CI ensures that code pushed to repository branches is automatically validated through automated builds and tests.
</details>

**Question 4:** How should sensitive credentials and API keys be handled in deployment pipelines?

* [ ] Committed directly in source code
* [x] Stored in encrypted secrets management services and injected dynamically
* [ ] Printed to console logs
* [ ] Sent via unencrypted email

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Stored in encrypted secrets management services and injected dynamically

**Explanation:** Secrets must be managed using specialized vault services (e.g. GitHub Secrets or Azure Key Vault) to prevent credential leaks.
</details>

**Question 5:** What is a major benefit of Infrastructure as Code (IaC) when configuring Artifacts Versioning & Monitoring Observability?

* [ ] Manual server setup
* [x] Version-controlled, reproducible, and automated infrastructure provisioning
* [ ] Disabling firewalls
* [ ] Slower deployments

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Version-controlled, reproducible, and automated infrastructure provisioning

**Explanation:** IaC allows infrastructure configurations to be versioned, reviewed, and automatically applied without drift.
</details>

**Question 6:** What does a Deployment Approval Gate accomplish in enterprise release pipelines?

* [ ] Deletes old branches
* [x] Requires explicit authorization or automated checks before promoting changes to production
* [ ] Restarts the build server
* [ ] Compresses build files

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Requires explicit authorization or automated checks before promoting changes to production

**Explanation:** Approval gates introduce compliance checkpoints ensuring quality, security, and authorization before production releases.
</details>

**Question 7:** Which DORA metric measures how long it takes for committed code to reach production?

* [ ] Change Failure Rate
* [x] Lead Time for Changes
* [ ] Mean Time to Recovery
* [ ] Deployment Frequency

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Lead Time for Changes

**Explanation:** Lead Time for Changes quantifies the efficiency of the delivery pipeline from commit to deployment.
</details>

**Question 8:** When a production deployment fails, what is the fastest way to restore service stability when using Artifacts Versioning & Monitoring Observability?

* [ ] Re-write the application from scratch
* [x] Execute an automated rollback to the last known good artifact
* [ ] Ignore the alert
* [ ] Turn off monitoring

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Execute an automated rollback to the last known good artifact

**Explanation:** Automated rollbacks quickly restore operational stability by redeploying the previous verified release artifact.
</details>
