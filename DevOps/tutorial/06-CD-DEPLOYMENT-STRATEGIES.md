# Module 6: CD — Deployment Strategies

## Module Overview
- **Concept Core:** 75 min | **Hands-On Extension:** 75 min + Lab (120 min)
- **Real-World Scenarios:** FastCart blue-green deployments; TenantHub canary rollouts
- **Key Patterns:** Artifact promotion, versioning, blue-green, canary, rolling updates, health checks

---

## 🟢 Concept Core: Deployment Strategies & Artifact Promotion

### The Problem: Risky Deployments

**Traditional deployment (pre-DevOps):**
```
10:00 AM: Stop app server
10:00:30 AM: Copy new code to server
10:01 AM: Start app server
10:01 – 10:05 AM: App is DOWN (5 minutes of lost revenue)
10:05 AM: App is UP
10:05 – 11:00 AM: Monitor for issues
11:00 AM: If issues, manual rollback (another 5 min downtime)
```

**Result:** Every deployment = customer downtime risk + manual rollback pain

### The Solution: Modern Deployment Strategies

Modern strategies enable:
- ✅ **Zero downtime** — Customers don't notice deployment
- ✅ **Quick rollback** — Issues found? Revert in 30 seconds
- ✅ **Gradual rollout** — Test with 10% of traffic first
- ✅ **Automatic health checks** — If metrics degrade, auto-rollback

---

## 🟢 Deployment Strategies

### 1. Blue-Green Deployment (Recommended for Web Apps)

**Concept:** Two identical production environments (Blue = old, Green = new). Swap instantly.

```
BEFORE DEPLOYMENT
└── Load Balancer
    └── All traffic → BLUE (v1.0)

DEPLOY TO GREEN (parallel)
├── Load Balancer
│   └── All traffic → BLUE (v1.0)
└── GREEN starting up (v1.1)
    ├── Containers pulling new image
    ├── Dependencies initialized
    └── Health checks passing

SWITCH (instant)
└── Load Balancer
    └── All traffic → GREEN (v1.1)
    
KEEP BLUE AS ROLLBACK
├── Load Balancer
│   └── All traffic → GREEN (v1.1)
└── BLUE (v1.0) kept running for 24 hours
    (If issues with v1.1, switch back instantly)
```

**Pros:**
- ✅ Zero downtime
- ✅ Instant rollback (flip switch back)
- ✅ Full test before traffic switches

**Cons:**
- ❌ Costs 2x resources (need two full environments)
- ❌ Database migrations are hard (two apps can't share schema changes mid-deploy)
- ❌ Complex for stateful apps

**Cost estimate:** FastCart blue-green setup:
- Green environment: 2× App Service instances, 2× databases
- Extra: $500–1000/month
- **But worth it:** Zero downtime saves customer trust, avoids SLA breaches

---

### 2. Canary Deployment (Recommended for New Features)

**Concept:** Roll out to 10% of users, monitor metrics, gradually increase if good.

```
DEPLOY v1.1
└── Track these metrics (in real-time):
    ├── Request latency (is it slow?)
    ├── Error rate (5XX errors?)
    └── Business metrics (checkout rate, revenue)

ROLLOUT STRATEGY
├── 10% of users on v1.1 (5 min) — all metrics good? Proceed
├── 25% of users on v1.1 (5 min) — still good? Proceed
├── 50% of users on v1.1 (5 min) — metrics stable? Proceed
├── 75% of users on v1.1 (5 min) — looking good? Proceed
└── 100% of users on v1.1 (complete)

OR IF ISSUE
├── 10% on v1.1, latency spikes 50%
├── INSTANT: Roll back to 0% on v1.1 (automatic)
└── Alert: "Canary detected issue, rolled back"
```

**Pros:**
- ✅ Limits blast radius (bug only affects 10% of users)
- ✅ Real user metrics (not synthetic tests)
- ✅ Automatic rollback if metrics degrade
- ✅ Same cost as single environment

**Cons:**
- ❌ Slower rollout (could take 20 min for 100%)
- ❌ Requires feature flags (code must support old + new behavior)
- ❌ Metrics must be very reliable

**Tools:** Service mesh (Istio, Linkerd), app gateway routing, feature flags

---

### 3. Rolling Deployment (Common for Kubernetes)

**Concept:** Replace instances one by one (no downtime, but brief capacity loss).

```
DEPLOYMENT (Kubernetes)
├── 10 pod replicas running v1.0
└── Rolling update:
    ├── Drain pod 1 → Start new pod with v1.1 (9 replicas running)
    ├── Drain pod 2 → Start new pod with v1.1 (8 replicas running)
    ├── ... repeat for all pods
    └── Done: All 10 pods running v1.1

During update:
├── Capacity reduced during transition
├── Slow requests go to older instances (no breaking version mismatches)
├── If issue: Pause and revert (old pods still running)
```

**Pros:**
- ✅ No extra infrastructure cost
- ✅ Gradual, low-risk
- ✅ Works well with Kubernetes

**Cons:**
- ❌ Brief capacity loss (if 10 pods, only 8–9 handling traffic)
- ❌ Slower than blue-green
- ❌ Database migrations still tricky

---

### 4. Shadow Deployment (Advanced, Rarely Used)

**Concept:** Route % of traffic to new version, but discard its response. Users don't see it.

```
REQUEST FROM USER
├── Split: Send to both v1.0 (primary) and v1.1 (shadow)
├── v1.0 response → Return to user (user sees this)
└── v1.1 response → Discard (but log metrics)

Use: Validate v1.1 on real traffic without risk
Cost: 2x backend load (expensive)
```

**When to use:** Validating risky architectural changes before flipping traffic.

---

## 🟢 Real-World Scenario 1: FastCart Blue-Green

**Feature:** One-click checkout
**Deployment strategy:** Blue-green

```
TIMELINE:

10:00 AM: Build and test artifact (v1.5.0) ✅

10:05 AM: Deploy to GREEN environment
        ├── Pull image from Azure Container Registry
        ├── Spin up 5 App Service instances (GREEN pool)
        ├── Run health checks ✅
        └── GREEN ready, still no traffic

10:10 AM: Run smoke tests against GREEN
        ├── Checkout flow test ✅
        ├── Payment test ✅
        └── Confirmed working with production-like data

10:15 AM: SWITCH traffic (blue → green)
        ├── Load balancer: ALL traffic now → GREEN
        └── 0 seconds downtime ✅

10:15 – 10:45 AM: Monitor GREEN
        ├── Latency: Normal ✅
        ├── Error rate: <0.1% ✅
        ├── Checkout success rate: Up 5% ✅
        └── Revenue: +$8,300/hour expected

10:45 AM: BLUE deployment complete (keep for 24 hours as rollback)

11:00 AM: Issue detected in GREEN (hypothetical)
        ├── Latency spike observed
        ├── INSTANT: Switch traffic back to BLUE
        └── 30 second rollback ✅ (disaster averted)
```

**Cost:** $500–1000/month for extra infrastructure, but justified by zero downtime + confidence.

---

## 🟢 Real-World Scenario 2: TenantHub Canary Rollout

**Feature:** AI maintenance routing
**Deployment strategy:** Canary (gradual)

```
TIMELINE:

2:00 AM: Build artifact (v2.1.0) and run tests ✅

2:10 AM: Deploy to production with feature flag OFF
        └── Code is live, but disabled (no user impact)

2:15 AM: CANARY: Route 10% of requests to new code
        ├── 4,800 tenants × 10% = 480 tenants on v2.1
        └── Monitor metrics for 5 minutes

2:20 AM: Check metrics
        ├── Request latency: +2% (acceptable)
        ├── Error rate: 0.05% (normal)
        ├── AI routing accuracy: 87% (good)
        └── Decision: Continue

2:21 AM: CANARY: Route 25% to new code (1,200 tenants)
        └── Monitor 5 minutes

2:26 AM: Metrics still good → Roll out to 50% (2,400 tenants)
        └── Monitor 5 minutes

2:31 AM: Metrics good → Roll out to 100% (all 4,800 tenants)
        └── Full rollout complete in 30 minutes ✅

2:31 – 3:00 AM: Monitor production
        ├── All metrics stable
        ├── Support team: "No tickets on new feature"
        └── Success: Feature live to all tenants

HYPOTHETICAL ROLLBACK:
2:25 AM: Latency spike at 50% rollout
        ├── System detects: latency > baseline + 30%
        ├── AUTOMATIC: Revert to 0% (all traffic to v2.0)
        └── Alert: "Canary detected issue, rolled back"
        ├── Team investigates offline
        └── Retry tomorrow with fix
```

**Advantages:**
- Real-time monitoring (not lab tests)
- Limited user impact (50 tenants affected before rollback)
- Automatic rollback safety net

---

## 🟢 Artifact Promotion & Versioning

### Versioning Strategy

**Semantic Versioning (semver):** MAJOR.MINOR.PATCH

```
1.5.0-rc.1 (Release Candidate, pre-release)
  ↑ MAJOR:  Big breaking changes (e.g., API redesign)
    MINOR:  Backward-compatible features (new endpoint)
    PATCH:  Bug fixes (typo, performance)
    -rc.1:  Release candidate (pre-release, not production yet)

Examples:
├── 1.0.0 (first production release)
├── 1.1.0 (new feature, backward compatible)
├── 1.1.1 (bug fix)
├── 2.0.0 (breaking change, old clients need update)
└── 2.0.0-alpha, 2.0.0-beta, 2.0.0-rc.1 (pre-releases)
```

### Artifact Promotion

**Concept:** Build once, test multiple times in different environments.

```
SINGLE BUILD ARTIFACT (Docker image, .NET DLL, JAR)
└── Versioned: myapp:1.5.0

PROMOTE THROUGH ENVIRONMENTS
├── dev environment: Pull myapp:1.5.0
│   ├── Basic smoke tests
│   └── Deployment: 5 minutes
│
├── staging environment: Pull same myapp:1.5.0
│   ├── Full test suite
│   ├── QA validation
│   └── Deployment: 5 minutes
│
└── production environment: Pull same myapp:1.5.0
    ├── Blue-green deploy
    ├── Gradual traffic shift (canary)
    └── Deployment: 10 minutes

RESULT: Same binary tested in 3 environments
Cost: 1 build, 3 deployments (reuse = efficiency)
```

### Artifact Storage

**Options:**

| Registry | Best For | Cost | Example |
|----------|----------|------|---------|
| **GitHub Packages** | GitHub-native, public projects | Free (public), $5/mo (private) | `ghcr.io/owner/app:1.5.0` |
| **Docker Hub** | Community projects, large user base | Free public, $5/mo private | `docker.io/owner/app:1.5.0` |
| **Azure Container Registry** | Enterprise, Azure workloads | $5–50/mo depending on storage | `myregistry.azurecr.io/app:1.5.0` |
| **AWS ECR** | AWS-native | $0.10/GB stored/month | `12345.dkr.ecr.us-east-1.amazonaws.com/app:1.5.0` |
| **NuGet** | .NET packages | Free (NuGet.org) or private feed | `nuget.org/packages/MyLib/1.5.0` |
| **npm** | JavaScript packages | Free (npm.org) or private registry | `npm.org/package/mylib@1.5.0` |

**FastCart uses:** Azure Container Registry (tied to Azure deployments)
**TenantHub uses:** Azure Container Registry + Azure Artifacts (for metadata, policies)

---

## 🔷 Hands-On Extension: Blue-Green & Canary Deployments

### Part A: Blue-Green Deployment with Azure App Service

**Setup:** Two App Service slots (Blue = production, Green = staging slot)

#### Step 1: Create Dual Slot Setup

```bash
# Already has "production" slot
# Create "staging" slot for green environment
az webapp deployment slot create \
  -n checkout-app \
  -g checkout-rg \
  --slot staging
```

**Result:**
```
checkout-app (load balancer)
├── production slot (blue) — live traffic
└── staging slot (green) — no traffic, for testing
```

#### Step 2: Deploy to Green (Staging Slot)

```yaml
# azure-pipelines.yml
stages:
  - stage: BuildImage
    jobs:
      - job: BuildAndPush
        steps:
          - task: Docker@2
            inputs:
              command: buildAndPush
              repository: '$(ACR_REGISTRY)/checkout:$(Build.BuildId)'
              Dockerfile: Dockerfile

  - stage: DeployToGreen
    displayName: Deploy to Green Slot
    jobs:
      - deployment: DeployGreen
        environment: 'green'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebAppContainer@1
                  inputs:
                    azureSubscription: 'Azure'
                    appName: 'checkout-app'
                    imageName: '$(ACR_REGISTRY)/checkout:$(Build.BuildId)'
                    slotName: 'staging'  # Deploy to GREEN (staging slot)
                  displayName: 'Deploy image to green slot'
                
                - script: |
                    # Health check on green
                    sleep 30  # Wait for startup
                    curl -f https://checkout-app-staging.azurewebsites.net/health || exit 1
                  displayName: 'Health check green'

  - stage: BlueGreenSwap
    displayName: Swap Blue-Green
    dependsOn: DeployToGreen
    jobs:
      - job: SwapSlots
        pool: server
        steps:
          - task: ManualValidation@0
            inputs:
              notifyUsers: 'david@fastcart.com'
              instructions: 'Approve to swap blue-green'
          
          - task: AzureAppServiceManage@0
            inputs:
              azureSubscription: 'Azure'
              appName: 'checkout-app'
              action: 'Swap Slots'
              sourceSlot: 'staging'
              targetSlot: 'production'
            displayName: 'Swap staging → production'
```

**What happens:**
1. Deploy to staging slot (GREEN) while production (BLUE) serves traffic
2. Test on GREEN without affecting users
3. On approval: Swap traffic from BLUE → GREEN (instant)
4. If issues: Swap back to BLUE (instant rollback)

---

### Part B: Canary Deployment with Feature Flags

```yaml
# Feature flag canary approach
stages:
  - stage: DeployCanary
    jobs:
      - job: CanaryRollout
        steps:
          - script: |
              # Deploy to all servers, but with feature flag OFF
              kubectl set image deployment/checkout \
                app=checkout:v2.1 \
                --record
              
              # Confirm all pods running new image
              kubectl rollout status deployment/checkout
            displayName: 'Deploy new image to all pods'
          
          - script: |
              # Enable feature for 10% of traffic (via feature flag)
              # Using a feature flag system (e.g., LaunchDarkly, Firebase)
              curl -X POST https://featureflags-api.com/rollout \
                -H "Authorization: Bearer ${{ secrets.FEATURE_FLAG_TOKEN }}" \
                -d '{
                  "feature": "ai_routing",
                  "rollout_percent": 10
                }'
              echo "Canary: 10% on AI routing"
            displayName: 'Enable feature for 10% (canary)'
          
          - script: |
              # Monitor for 5 minutes
              for i in {1..5}; do
                sleep 60
                METRICS=$(curl -s https://metrics-api/get?metric=latency)
                echo "Minute $i: $METRICS"
                
                # If latency > 30% baseline, rollback
                LATENCY=$(echo $METRICS | jq '.latency')
                if (( $(echo "$LATENCY > 130" | bc -l) )); then
                  echo "Latency spike detected! Rolling back..."
                  curl -X POST https://featureflags-api.com/rollout \
                    -d '{"feature": "ai_routing", "rollout_percent": 0}'
                  exit 1
                fi
              done
            displayName: 'Monitor canary metrics (5 min)'
          
          - script: |
              # If we get here, metrics are good
              # Increase to 100%
              curl -X POST https://featureflags-api.com/rollout \
                -H "Authorization: Bearer ${{ secrets.FEATURE_FLAG_TOKEN }}" \
                -d '{
                  "feature": "ai_routing",
                  "rollout_percent": 100
                }'
              echo "Canary successful! Feature at 100%"
            displayName: 'Roll out to 100%'
```

---

## 🔷 Lab: Implement Blue-Green Deployment

### Objectives
- ✅ Set up dual App Service slots
- ✅ Deploy to green (staging) slot
- ✅ Run health checks on green
- ✅ Perform blue-green swap
- ✅ Test traffic routing to green
- ✅ Perform rollback

### Prerequisites
- Azure subscription (free tier works)
- Existing App Service
- 120 minutes

### Lab Steps

#### Step 1: Set Up Slots

```bash
# Create staging slot
az webapp deployment slot create \
  -n myapp \
  -g mygroup \
  --slot staging

# Verify slots
az webapp deployment slot list \
  -n myapp \
  -g mygroup
```

#### Step 2: Deploy Different Versions

**Deploy v1.0 to production:**
```bash
az webapp up \
  -n myapp \
  --resource-group mygroup \
  --zip myapp-v1.0.zip
```

**Deploy v1.1 to staging:**
```bash
az webapp deployment slot swap \
  --resource-group mygroup \
  --name myapp \
  --slot staging
  
# Wait a moment, then swap back
az webapp deployment slot swap \
  --resource-group mygroup \
  --name myapp \
  --slot staging
```

#### Step 3: Run Health Checks

```bash
#!/bin/bash
PROD_URL="https://myapp.azurewebsites.net"
STAGING_URL="https://myapp-staging.azurewebsites.net"

echo "Testing production (blue)..."
curl -f $PROD_URL/health
curl -f $PROD_URL/api/checkout

echo "Testing staging (green)..."
curl -f $STAGING_URL/health
curl -f $STAGING_URL/api/checkout
```

#### Step 4: Perform Swap

```bash
az webapp deployment slot swap \
  --resource-group mygroup \
  --name myapp \
  --slot staging
```

**Before swap:**
- Production: v1.0
- Staging: v1.1

**After swap:**
- Production: v1.1 (now live)
- Staging: v1.0 (rollback ready)

#### Step 5: Monitor & Verify

```bash
# Check which version is live
curl https://myapp.azurewebsites.net/version
# Should show: {"version": "1.1.0"}

# Check metrics for issues
az monitor metrics list \
  -n myapp \
  --resource-group mygroup \
  --metric ResponseTime
```

#### Step 6: Rollback

```bash
# If issues detected, swap back
az webapp deployment slot swap \
  --resource-group mygroup \
  --name myapp \
  --slot staging
```

Now production is back to v1.0 (instant rollback).

### Validation Checklist

- ✅ Two slots created (production + staging)
- ✅ Different versions deployed to each slot
- ✅ Swapped slots successfully
- ✅ Verified traffic on new version
- ✅ Rolled back to previous version
- ✅ No downtime during swap

---

## 🟢 Assessment: Deployment Strategies

> **Question 1:** FastCart uses blue-green deployments. Cost is $1000/month extra. Why is it worth it?
> - A) Because it's cooler than other strategies
> - B) Zero downtime + instant rollback >> cost (if even one incident costs $10k, it pays for itself)
> - C) They have no choice
> - D) It's required by law
> 
> **Answer:** B — One incident with 1-hour downtime could cost $10k+ in lost revenue. $1000/month extra for zero-downtime deployments is ROI-positive.

> **Question 2:** TenantHub uses canary deployments. They roll out 10% → 25% → 50% → 100%. What's the business benefit?
> - A) Slower deployments are better
> - B) Limits blast radius; real metrics guide safety; can auto-rollback
> - C) No benefits, just slow
> - D) It's for developers only
> 
> **Answer:** B — If 10% sees an issue, you've limited damage. Real production metrics beat lab tests. Auto-rollback if metrics degrade keeps customers safe.

---

## Next Steps

✅ **You now understand:**
- Blue-green deployment (zero downtime)
- Canary deployment (limited risk rollout)
- Artifact promotion (build once, test many)
- Versioning strategies

🔜 **Next module:** Environments & Approval Gates (who approves deployments?)

---

## References

- [Azure Deployment Slots](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)
- [Blue-Green Deployment Pattern](https://martinfowler.com/bliki/BlueGreenDeployment.html)
- [Canary Releases](https://martinfowler.com/bliki/CanaryRelease.html)
- [Semantic Versioning](https://semver.org/)
