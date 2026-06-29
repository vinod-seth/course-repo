# Module 7: Environments & Approval Gates

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/07-ENVIRONMENTS-AND-APPROVAL-GATES.md)


## Module Overview
- **Concept Core:** 75 min | **Hands-On Extension:** 75 min + Lab (120 min)
- **Real-World Scenarios:** FastCart release approval workflow; TenantHub multi-tenant gates
- **Key Tools:** GitHub Environments, Azure Environments, Deployment Gates, Policies

---

## 🟢 Concept Core: Who Approves Production Deployments?

### The Problem Without Gates

**Scenario:** Alice is excited about her feature and deploys it directly to production without approval.

```
10:00 AM: Alice merges PR
10:00:05 AM: Artifact built
10:00:10 AM: Alice deploys to production (no approval needed)
10:05 AM: Customer notices checkout is broken (Alice introduced a bug)
10:06 AM: Revenue impact: $50k/hour
10:30 AM: Someone notices the outage, starts investigating
10:45 AM: Alice's bug found and fixed
11:00 AM: Rolled back to v1.0
11:30 AM: Re-deployed v1.1 with fix
Total damage: $70k lost revenue + 1.5 hours on-call stress
```

### The Solution: Approval Gates

**Gate = Human checkpoint** before deployment to sensitive environment

```
CODE → APPROVAL GATE → PRODUCTION

Who approves?
├── Production: Release manager (understands risk)
├── Staging: QA lead (validates functionality)
├── Dev: Developer (can deploy freely, it's dev)
```

### Types of Approval Gates

| Gate Type | Where | Who | Time | Use Case |
|-----------|-------|-----|------|----------|
| **Manual Approval** | Production, Staging | Release manager, ops | 5–30 min | Business hours, standard deployments |
| **Timer Gate** | Production | (automated) | 2–24 hours | Wait period for monitoring/rollback window |
| **Auto Gate (Branch Policy)** | Before merge | (system) | Instant | Must pass tests before merging to main |
| **Deployment Gate** | Production | (system + human) | Varies | Condition-based (e.g., "if metrics good") |
| **Business Hours Gate** | Production | (scheduled) | Varies | Only deploy Mon–Fri, 9–5 |

### Real-World Gate Policy

**FastCart Production Gate Policy:**

```
Deployment to Production requires:
├── 1. Build must pass tests + linting ✓ (automatic)
├── 2. Code review approved by 1 expert ✓ (automatic via PR)
├── 3. QA approval on staging ✓ (manual, 30 min window)
├── 4. Release manager approval ✓ (manual, notified via Slack)
├── 5. No other deployments in progress (prevent conflicts)
├── 6. Deploy only Mon–Fri 9 AM–5 PM UTC (avoid middle-of-night)
└── 7. Must have rollback plan (documented in PR)
```

**Approval chain:**
```
Alice (dev) → Bob (code reviewer) → Carol (QA) → David (release mgr) → Production
```

### Why This Matters

| Without Gates | With Gates |
|--------------|-----------|
| ❌ Anyone can deploy anything | ✅ Only approved changes go live |
| ❌ No audit trail | ✅ Full audit: who, what, when, why |
| ❌ Compliance violations | ✅ SoX/HIPAA compliant |
| ❌ Risky deployments | ✅ Controlled, low-risk |

**Cost:** Approval takes 30 min. **Benefit:** One outage prevented saves $50k+.

---

## 🟢 Real-World Scenario 1: FastCart Approval Workflow

```
TIMELINE

2:00 PM: Alice creates PR with checkout feature
         └── GitHub Actions runs tests ✅

2:15 PM: Bob (code reviewer) reviews and approves ✅
         └── Comment: "Nice, tests are strong"

2:30 PM: PR merged to main
         └── Webhook triggers build + deploy to staging

2:35 PM: Carol (QA) sees "staging ready" notification
         └── Tests on staging for 20 minutes
         └── Approves: "Feature looks good, no bugs" ✅

2:55 PM: Alice clicks "Request Production Approval" in Azure DevOps
         └── David (release manager) gets notified via Slack

3:00 PM: David reviews:
         ├── Checks: Deployment checklist filled? ✅
         ├── Checks: Rollback plan documented? ✅
         ├── Checks: Is it Mon–Fri, 9–5 UTC? ✅
         ├── Checks: No other deployments in progress? ✅
         └── Approves ✅

3:05 PM: Production deployment starts automatically
         ├── Blue-green deploy (0 downtime)
         └── Health checks pass ✅

3:10 PM: Feature live to customers
         └── Alice receives: "v1.5.0 deployed to production"

3:10–4:00 PM: Monitor dashboard
             ├── Checkout success rate: +5% ✅
             └── Revenue impact: +$8,300/hour ✅
```

---

## 🟢 Real-World Scenario 2: TenantHub Multi-Tenant Gates

**Challenge:** Different tenant groups, different approval flows

```
DEPLOYMENT: AI Routing Feature

├── STABLE POOL (200 conservative tenants)
│   └── Gate policy: Release manager + compliance officer approval
│       └── (Stricter: want extra review for risk-averse customers)
│
└── FAST POOL (4,800 early-adopter tenants)
    └── Gate policy: Release manager approval only
        └── (These tenants want fast updates)

TIMELINE
10:00 AM: Alice requests prod approval
10:15 AM: David (release manager) approves for BOTH pools
10:20 AM: Eve (compliance officer) approves for STABLE pool
10:25 AM: Deploy to FAST pool (only needs David's approval) ✓
10:30 AM: Deploy to STABLE pool (needed both approvals) ✓
10:30–11:00 AM: Monitor both pools for issues
```

---

## 🔷 Hands-On Extension: GitHub Environments & Azure DevOps Gates

### Part A: GitHub Environments

**File structure:**
```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  workflow_dispatch:  # Manual trigger

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: 'staging'  # Uses staging environment
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to staging
        run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: 'production'  # Uses production environment (with approval gate)
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to production
        run: ./deploy.sh production
```

**Configure environment in GitHub Settings:**
1. Go to repo **Settings** → **Environments**
2. Click **New environment**: `production`
3. Add settings:
   - ✅ Required reviewers: Select release managers
   - ✅ Deployment branches: `main` only
   - ✅ Deployment gates (optional): Custom check scripts

**When workflow runs:**
1. Deploy to staging runs automatically
2. Deploy to production waits for approval
3. GitHub notifies reviewers
4. Reviewer approves or rejects
5. Approval/rejection logged in audit trail

---

### Part B: Azure DevOps Deployment Gates

```yaml
# azure-pipelines.yml
stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - script: dotnet build

  - stage: DeployStaging
    dependsOn: Build
    jobs:
      - deployment: DeployStaging
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure'
                    appName: 'myapp-staging'
                    package: '$(Agent.BuildDirectory)/**/*.zip'

  - stage: DeployProduction
    dependsOn: DeployStaging
    jobs:
      - deployment: DeployProduction
        environment: 'production'  # This environment has approval gates
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure'
                    appName: 'myapp-prod'
                    package: '$(Agent.BuildDirectory)/**/*.zip'
```

**Configure in Azure DevOps:**
1. **Pipelines** → **Environments**
2. Create: `production`
3. Add approval policy:
   - Required approvers: david@company.com, release-managers group
   - Auto-approve time limit: None (manual only)
   - Branch filters: main only
   - Business hours: 9 AM – 5 PM UTC, Mon–Fri

**Deployment gates (automated checks):**

```yaml
# Azure DevOps allows "gates" (automated approvals)
# Example: Only deploy if test coverage > 80%

# In environment settings, add custom "Deployment gates"
# Gate: Query Azure Monitor metrics
# Condition: "If error rate < 1%, allow deployment"
# Condition: "If latency < baseline + 10%, allow deployment"
```

---

## 🔷 Lab: Set Up Approval Gates

### Objectives
- ✅ Create production environment
- ✅ Set up approval requirements
- ✅ Trigger deployment workflow
- ✅ Request approval
- ✅ Approve and deploy
- ✅ View audit trail

### Prerequisites
- GitHub repo (for GitHub Actions) OR Azure DevOps project
- 120 minutes

### Lab Steps (GitHub Actions)

#### Step 1: Create Workflow

**File: `.github/workflows/deploy.yml`**

```yaml
name: Deploy with Approvals

on:
  workflow_dispatch:  # Manual trigger

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: npm run build
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: app-build
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: 'staging'
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: app-build
      - name: Deploy to staging
        run: echo "Deploying to staging..."

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: 'production'
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: app-build
      - name: Deploy to production
        run: echo "Deploying to production..."
```

#### Step 2: Create Environments

1. Go to repo **Settings** → **Environments**
2. Create `staging`:
   - No required reviewers (auto-deploy)
3. Create `production`:
   - Add required reviewers: Your GitHub user + team member
   - Deployment branches: `main` only

#### Step 3: Trigger Workflow

1. Go to **Actions** tab
2. Select "Deploy with Approvals"
3. Click **Run workflow** → **Run workflow**
4. Workflow starts

#### Step 4: See Approval Prompt

1. Workflow runs `build` and `deploy-staging` (auto)
2. When reaching `deploy-production`, workflow pauses
3. Shows: "Waiting for review"

#### Step 5: Approve Deployment

1. Click the workflow run
2. You'll see: "Review pending"
3. Click **Review deployments**
4. Select `production`
5. Write comment: "Looks good, ready for production"
6. Click **Approve and deploy**

#### Step 6: Watch Deployment Complete

Production job runs and completes.

#### Step 7: View Audit Trail

1. Go to **Settings** → **Environments** → **production**
2. Scroll to "Deployment history"
3. See: "Approved by [your name] at [time]"

### Lab Steps (Azure DevOps)

#### Step 1: Create Pipeline

Create `azure-pipelines.yml` with the YAML from Part B above.

#### Step 2: Create Environments

1. **Pipelines** → **Environments**
2. New: `production`

#### Step 3: Add Approval Policy

1. Click `production` environment
2. **More options** → **Approvals and checks**
3. Add: **Approval**
4. Required approvers: Select your user
5. Save

#### Step 4: Run Pipeline

1. Click **Run** on pipeline
2. Observe it progressing through stages
3. At `DeployProduction`, pipeline pauses: "Approval pending"

#### Step 5: Approve

1. Click the paused stage
2. Click **Approve** or **Reject**
3. Add comment: "Approved for release"
4. Deployment proceeds

#### Step 6: Check Audit

1. **Environments** → **production**
2. See deployment history with approvals

### Validation Checklist

- ✅ Environment created with approval policy
- ✅ Workflow/pipeline triggered manually
- ✅ Deployment paused at approval gate
- ✅ Approval requested (notification sent)
- ✅ Approved and deployment proceeded
- ✅ Audit trail shows who approved and when
- ✅ Can reject to block deployment

---

## 🟢 Assessment: Approval Gates

> **Question 1:** FastCart requires David (release manager) to approve all production deployments. Why?
> - A) David wants to slow things down
> - B) Concentrates responsibility, ensures releases are planned, manages risk/SLA
> - C) It's a legal requirement
> - D) Developers can't be trusted
> 
> **Answer:** B — Release managers own operational risk. They coordinate with ops, check rollback plans, manage deployment windows (no 2 AM on Saturday), ensure SLA readiness.

> **Question 2:** Can approval gates delay deployments too much?
> - A) No, always fast
> - B) Yes, if gate takes hours; mitigate by: auto-approvals for low-risk changes, business hours windows, clear criteria
> - C) It's always a bottleneck
> - D) Never use approval gates
> 
> **Answer:** B — Gate speed matters. If it takes 4 hours to approve a typo fix, that's a problem. Solution: Automate low-risk approvals, have clear escalation path.

---

## Next Steps

✅ **You now understand:**
- Why approval gates matter (risk management)
- How to set up GitHub Environments
- How to configure Azure Deployment gates
- Audit trails and who-approves-what policies

🔜 **Next module:** Secrets, Security & Compliance (keeping credentials safe)

---

## References

- [GitHub Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments)
- [Azure DevOps Environments](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments)
- [Deployment Gates in Azure Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/approvals/gates)