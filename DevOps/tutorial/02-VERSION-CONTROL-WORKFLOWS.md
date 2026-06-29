# Module 2: Version Control Workflows & Branch Policies

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/02-VERSION-CONTROL-WORKFLOWS.md)


## Module Overview
- **Concept Core:** 60 min | **Hands-On Extension:** 60 min + Lab (60 min)
- **Real-World Scenarios:** E-commerce: Feature branch protection; SaaS: Multi-tenant deployment coordination
- **Key Tools:** GitHub Branch Protection, Azure Repos Policies, YAML workflow triggers

---

## 🟢 Concept Core: Git Workflows and Code Review Gates

### The Challenge: How Do Multiple Developers Ship Safely?

Imagine FastCart's checkout team:
- Alice writes the checkout feature
- Bob writes the payment retry logic
- Carol writes the refund UI
- All three are working on the same `main` branch

Without discipline, what could happen:
1. Alice pushes her code at 10 AM (works locally)
2. Bob pushes his code at 10:30 AM (conflicts with Alice's code, but Bob doesn't realize it)
3. Main branch is now **broken**
4. Production deployment at 2 PM uses broken code
5. Customers can't refund orders
6. **Fire drill at 3 PM**

### The Solution: Branching Strategy

Instead, use this workflow:

```
main (stable, production-ready)
 ↑
 ├── feature/checkout-v2 (Alice's work)
 ├── feature/payment-retry (Bob's work)
 └── feature/refund-ui (Carol's work
```

**Each developer works on their own branch, isolated from main.**

When Alice is done:
1. She creates a **Pull Request (PR)** to merge her branch into main
2. Bob (code reviewer) reviews the PR
   - "Does the code make sense?"
   - "Are there security issues?"
   - "Do the tests pass?"
3. If approved, the PR is merged
4. Alice's branch is deleted
5. Repeat for Bob and Carol

**Result:** Only tested, reviewed code enters main.

### Why This Matters

| Scenario | Without Branch Policy | With Branch Policy |
|----------|----------------------|-------------------|
| **Broken code in main** | Possible (Alice merges without review) | Impossible (merge blocked without review) |
| **Duplicate work** | Likely (Alice doesn't know Bob is working on payment) | Less likely (PR description + comments coordinate) |
| **Secrets in code** | Possible (developer forgets to remove API key) | Blocked (PR checks scan for secrets) |
| **Test coverage** | Inconsistent (developer skips tests locally) | Required (CI must pass before merge) |
| **Audit trail** | No (who merged what?) | Yes (merged by Bob on Jan 15 at 3 PM) |

---

### Branch Protection Rules (GitHub) vs. Branch Policies (Azure)

#### GitHub Branch Protection Rules

When you "protect" the `main` branch, you can enforce:

1. **Require a pull request before merging**
   - No direct pushes to main; all changes go through PR review
   
2. **Require status checks to pass before merging**
   - Example: GitHub Actions must run tests successfully
   - Example: SonarQube code quality must pass
   
3. **Require code review approvals**
   - How many reviewers? (e.g., 1 or 2)
   - Are reviewers dismissed if new commits are pushed? (usually yes — re-review required)
   
4. **Dismiss stale PR approvals when new commits are pushed**
   - If Bob approved, then Alice pushed a new commit, Bob's approval is stale
   - Prevents "one-and-done" reviews
   
5. **Require review from code owners**
   - `CODEOWNERS` file specifies: "Only Alice can approve payment code"
   - Prevents non-experts from merging sensitive code
   
6. **Allow specified actors to bypass these requirements**
   - Emergency hotfixes: Release manager can bypass rules
   - Use sparingly! (Logs every bypass)

#### Azure Repos Branch Policies

Same concept, different interface. You can enforce:

1. **Minimum reviewers** (e.g., 2)
2. **Comment requirements** — Reviewers must leave comments (enforces thoughtfulness)
3. **Work item linking** — PR must be linked to a task in Azure Boards
4. **Build policy** — Pipeline must succeed before merge
5. **Status policy** — External checks (SonarQube, security scans) must pass
6. **Merge strategy** — Squash commits, rebase, or merge (keeps git log clean)

### Real-World PR Workflow

**Step-by-step: Alice's checkout feature**

1. **Alice creates a feature branch**
   ```
   git checkout -b feature/checkout-v2
   ```

2. **Alice writes code and tests locally**
   - Commits: "Add one-click checkout form"
   - Commits: "Add form validation"
   - Commits: "Update tests"

3. **Alice pushes to GitHub**
   ```
   git push origin feature/checkout-v2
   ```

4. **Alice creates a Pull Request on GitHub**
   - Title: "Feature: One-click checkout for returning customers"
   - Description:
     ```
     ## What?
     Adds one-click checkout for returning customers (who saved payment method).
     
     ## Why?
     Reduces checkout time from 3 min to 10 sec. Expected 5% conversion uplift.
     
     ## Testing?
     - Unit tests: 95% coverage
     - E2E tests: Checkout flow verified with 3 test cards
     - Staging: Tested against production data snapshot
     
     ## Risk?
     Low. Feature flag off by default. Will roll out to 10% traffic first.
     ```

5. **GitHub Actions automatically runs**
   - Linting passes ✓
   - Tests pass ✓
   - Code coverage OK ✓
   - Deployment preview available

6. **Bob (code reviewer) is notified**
   - Sees the PR in his inbox
   - Clicks through to review
   - Reads Alice's description
   - Comments: "Nice! One question on line 42 — why is the timeout 5 seconds?"
   - Approves the PR

7. **Alice responds to Bob's comment**
   - Explains: "5 seconds is the payment API SLA. If it times out, checkout fails anyway."
   - Bob: "Perfect, makes sense. Approved."

8. **Carol (QA) tests the feature**
   - Clicks the "View deployment" link from the PR
   - Tests on staging with real payment processor (test mode)
   - Approves: "Works great! Ready to merge."

9. **Alice merges the PR**
   - GitHub deletes the feature branch automatically
   - Azure Pipelines is notified via webhook

10. **Azure Pipelines kicks off** (automatically)
    - Builds the artifact
    - Deploys to Azure App Service (staging)
    - Runs smoke tests
    - Notifies David (release manager): "Ready for production approval"

11. **David approves** (manual gate)
    - Reviews the change log
    - Checks: "No database migrations?" (Yes, none)
    - Checks: "Feature flag enabled?" (No, off by default)
    - Approves for production

12. **System deploys to production** (automatic)
    - Updates Azure App Service
    - Runs health checks
    - Monitoring dashboard shows new version is live
    - Notifies the team: "1.5.0 deployed successfully"

---

## 🟢 Real-World Scenario 1: FastCart (E-commerce)

**Branch Protection Policy:**
```
Branch: main
├── Require 1 pull request review before merging
├── Require status checks to pass:
│   ├── GitHub Actions: build + test
│   ├── SonarQube: code quality
│   └── GitHub Secret Scanning: no API keys exposed
├── Dismiss stale PR approvals when commits are pushed
└── Require review from CODEOWNERS
    ├── payment/ → Alice (payment expert)
    ├── checkout/ → Bob (checkout expert)
    └── infrastructure/ → Carol (DevOps lead)
```

**Impact:**
- Alice can't accidentally merge payment code without proper review
- If Alice pushes a new commit, Bob has to review again
- Emergency hotfixes can be merged by Carol (with full audit trail)

---

## 🟢 Real-World Scenario 2: TenantHub (SaaS)

**Branch Policy for Multi-Tenant Coordination:**
```
Branch: main (automatic deploy to "Stable" pool every Sunday)
Branch: release/fast (automatic deploy to "Fast" pool every day at 2 AM UTC)

Policy:
├── Require 2 code reviews (fast-moving features need consensus)
├── Comment requirement: Every review must have a comment (why approved?)
├── Link to Azure Board task (traceability)
├── Require build pipeline to pass
└── Merge strategy: Squash commits (keeps git log clean)
```

**Workflow:**
1. Developer creates PR targeting `main` or `release/fast`
2. Two reviewers approve with comments
3. Build passes
4. Reviewer squashes commits ("Add AI routing", "Fix tests", "Address review" → single "Add AI routing")
5. Auto-merged on Sunday (main) or daily (release/fast)
6. Azure Pipelines automatically deploys

---

## 🔷 Hands-On Extension: GitHub Branch Protection & PR Workflow

### Part A: Set Up GitHub Branch Protection

**Goal:** Protect `main` branch so that:
- All changes go through PR review
- GitHub Actions tests must pass
- At least 1 approval required

#### Tab: GitHub Web UI

1. Go to your repository
2. Navigate to **Settings** → **Branches**
3. Under "Branch protection rules", click **Add rule**
4. Branch name pattern: `main`
5. Enable:
   - ✅ Require a pull request before merging
   - ✅ Require approvals (set to 1)
   - ✅ Dismiss stale pull request approvals when new commits are pushed
   - ✅ Require status checks to pass before merging
   - Search for: `build` (GitHub Actions workflow name)
   - ✅ Require branches to be up to date before merging
6. Click **Create**

#### Tab: GitHub CLI (Programmatic)

```bash
# List existing branch protection rules
gh repo rules list

# Create branch protection rule via API
# Note: GH CLI doesn't fully support this yet, use GitHub API directly
curl -X POST https://api.github.com/repos/OWNER/REPO/branches/main/protection \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  -d '{
    "required_status_checks": {
      "strict": true,
      "contexts": ["build"]
    },
    "enforce_admins": false,
    "required_pull_request_reviews": {
      "dismissal_restrictions": {},
      "dismiss_stale_reviews": true,
      "require_code_owner_reviews": false,
      "required_approving_review_count": 1
    },
    "restrictions": null
  }'
```

---

### Part B: Create and Review a Pull Request

#### Scenario: Alice Pushes a Feature

```bash
# Step 1: Create feature branch
git checkout -b feature/checkout-optimization

# Step 2: Make changes (simulate with a file)
echo "// New checkout optimization code" > checkout-v2.js

# Step 3: Commit and push
git add checkout-v2.js
git commit -m "feature: Add one-click checkout for returning customers"
git push origin feature/checkout-optimization
```

**On GitHub:**
1. You'll see a prompt: "Compare & pull request"
2. Click it
3. Fill in PR description:
   ```markdown
   ## What?
   Implements one-click checkout for users with saved payment methods.
   
   ## Why?
   Reduces checkout friction. Estimated 5% conversion uplift.
   
   ## Testing
   - ✓ Local testing with Stripe test cards
   - ✓ Unit tests (95% coverage)
   - ✓ E2E tests (3 checkout flows)
   
   ## Risk?
   Low. Feature flag off by default. Canary rollout to 10% traffic.
   
   Closes #123
   ```
4. Click **Create pull request**

**What Happens Automatically:**
- GitHub Actions workflow starts (from `.github/workflows/ci.yml`)
- Shows build status in the PR

#### Tab: GitHub Actions Workflow (Read-Only Example)

```yaml
# .github/workflows/ci.yml
name: CI Pipeline
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Install dependencies
        run: npm install
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test -- --coverage
      
      - name: Scan for secrets
        uses: gitleaks/gitleaks-action@v2
      
      - name: SonarQube scan (if configured)
        run: |
          echo "Code quality check passed"
```

**What Bob (Reviewer) Sees:**
- ✅ Build passed
- ✅ Tests passed
- ✅ No secrets detected
- ✅ Code coverage OK

Bob reviews the code:
```
Bob's comment on line 42:
"Why is the timeout 5 seconds?"

Alice's response:
"5 seconds is the payment API SLA. Anything over 5 sec, we timeout and fail safely."

Bob: "Got it. Good design. Approved. ✅"
```

Bob clicks **Approve**. Now the PR can be merged.

---

### Part C: Azure Repos Policies (If Using Azure Repos)

```yaml
# If hosting on Azure Repos instead of GitHub

Branch: main
├── Minimum number of reviewers: 2
├── Automatically reset votes when commits are pushed: YES
├── Check for commented issues: YES (comments required)
├── Work item linking: Required (must link to Azure Board task)
├── Require a merge strategy: Squash commits
└── Build policy: "MyProject-CI-Pipeline" must succeed
```

**In practice:**
- Developer creates PR
- Two reviewers must approve with comments
- Build must pass
- PR description must link to work item (e.g., "Fixes #123")
- Merge strategy: All commits squashed into one

---

## 🔷 Lab: Create a Feature Branch and Pull Request

### Objectives
- ✅ Create a feature branch
- ✅ Make code changes
- ✅ Push to GitHub/Azure
- ✅ Create a Pull Request
- ✅ See GitHub Actions run automatically
- ✅ Have a team member review and approve
- ✅ Merge the PR
- ✅ See Azure Pipelines trigger

### Prerequisites
- Git installed locally
- GitHub or Azure DevOps account
- Code editor (VS Code, etc.)
- 30 minutes

### Lab Steps

#### Step 1: Clone Repository
```bash
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO
```

#### Step 2: Create Feature Branch
```bash
git checkout -b feature/add-logging
```

#### Step 3: Make a Code Change

**Tab: Python Example**
```python
# app.py
import logging

# Add logging (new feature)
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def checkout():
    logger.info("Checkout started")
    # ... checkout logic
    logger.info("Checkout completed")
```

**Tab: JavaScript Example**
```javascript
// app.js
import { createLogger } from './logger.js';

// Add logging (new feature)
const logger = createLogger('checkout');

async function checkout() {
    logger.info('Checkout started');
    // ... checkout logic
    logger.info('Checkout completed');
}
```

**Tab: C# / .NET Example**
```csharp
// Program.cs
using Serilog;

// Add structured logging (new feature)
Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .MinimumLevel.Information()
    .CreateLogger();

public class CheckoutService {
    public void ProcessCheckout() {
        Log.Information("Checkout started");
        // ... checkout logic
        Log.Information("Checkout completed");
    }
}
```

#### Step 4: Run Tests Locally
```bash
# Python
python -m pytest tests/ --cov

# JavaScript/Node
npm test -- --coverage

# C# / .NET
dotnet test /p:CollectCoverage=true
```

✅ All tests should pass.

#### Step 5: Commit and Push
```bash
git add .
git commit -m "feature: Add structured logging to checkout service

- Add logging at checkout start and completion
- Tests updated to verify log output
- Closes #42"

git push origin feature/add-logging
```

#### Step 6: Create Pull Request

On GitHub/Azure:
1. Navigate to your repo
2. Click "Compare & pull request"
3. Title: "Feature: Add structured logging to checkout"
4. Description:
   ```markdown
   ## What?
   Adds structured logging to checkout service for better observability.
   
   ## Why?
   Helps debugging checkout issues in production. Can track latency, errors.
   
   ## Testing?
   ✓ Unit tests verify logging output
   ✓ Manual testing on staging
   
   Closes #42
   ```
5. Click **Create Pull Request**

#### Step 7: Wait for CI/CD

**You'll see:**
- GitHub Actions (or Azure Pipelines) automatically running
- Status: "Some checks are in progress"
- After 2–5 minutes: "All checks passed ✅"

#### Step 8: Request Review

1. On the PR, click **Reviewers** (right sidebar)
2. Assign a team member (or self-review if practicing)
3. They'll be notified

#### Step 9: Reviewer Approves

Reviewer sees:
- PR description
- Code changes (green/red for additions/deletions)
- "Build passed" message
- Tests coverage report

Reviewer clicks **Approve** and leaves a comment:
```
"Nice logging! Clear and structured. Approved."
```

#### Step 10: Merge PR

Back on the PR, you now see: "✅ All checks passed" and "Approved by @reviewer"

Click **Merge pull request** → **Confirm merge**

#### Step 11: Trigger CD Pipeline

Once merged:
- GitHub webhooks notify Azure Pipelines (if configured)
- OR GitHub Actions runs again (now with merge to main)
- Artifact is built and deployed to staging
- You receive notification: "Deployed to staging"

### Validation Checklist

- ✅ Feature branch created and pushed
- ✅ GitHub Actions ran automatically
- ✅ All tests passed
- ✅ PR created with clear description
- ✅ Code review completed and approved
- ✅ PR merged successfully
- ✅ Main branch updated
- ✅ CD pipeline triggered (builds and deploys artifact)
- ✅ Deployment successful (check Azure App Service, GitHub Packages, or artifact storage)

### Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "PR can't be merged" | Status checks failed | Check GitHub Actions logs, fix failing tests |
| "Merge blocked" | Branch protection rule | Ensure at least 1 approval; sync with main if conflicts |
| "Tests failed" | Code issue or test issue | Run tests locally, debug, push fix |
| "CD pipeline didn't run" | Webhook not configured | Check: Settings → Webhooks (GitHub) or Service Connections (Azure) |

---

## 🟢 Assessment: Branch Policies & PR Workflow

> **Scenario:** Your team is shipping a payment feature. You set up branch protection on `main`:
> - Require 1 approval
> - Require GitHub Actions tests to pass
> 
> **Question 1:** Alice creates a PR. Tests fail. What happens?
> - A) PR is automatically merged anyway
> - B) PR can't be merged until tests pass
> - C) Alice can merge with manager override
> - D) Waiting for human review is sufficient
> 
> **Answer:** B — Branch protection blocks merge if status checks fail. Tests must pass first.

<details>
<summary>Click to expand: Why the others are wrong</summary>

- **A)** — No, branch protection enforces status checks.
- **C)** — Possible but requires bypassing the rule (logged, not default).
- **D)** — Human review isn't enough if tests fail (code is broken).

</details>

> **Question 2:** Bob reviews Alice's PR and approves. Then Alice pushes a new commit (bug fix). What happens to Bob's approval?
> - A) Approval is still valid
> - B) Approval is automatically dismissed (stale)
> - C) Bob must manually re-approve
> - D) Alice can merge without re-review
> 
> **Answer:** B — If you enable "Dismiss stale pull request approvals when new commits are pushed", Bob's approval is cleared.

<details>
<summary>Click to expand: Why the others are wrong</summary>

- **A)** — Only if you don't enable the rule. Best practice is to enable it.
- **C)** — Same as B, but forces manual action.
- **D)** — No, protection rule prevents merge without valid approval.

</details>

> **Question 3:** In FastCart's branching policy, why require "review from CODEOWNERS" for the `payment/` directory?
> - A) To slow down deployment
> - B) To ensure only payment experts review payment code
> - C) To add unnecessary approvals
> - D) Prevents developers from working on payment
> 
> **Answer:** B — CODEOWNERS ensures domain experts review domain-critical code (payment code requires Alice's approval).

---

## Next Steps

✅ **You now understand:**
- Why branch protection matters (code review, tests, audit trail)
- How to set up GitHub branch protection rules
- The full PR review workflow
- Why stale approvals should be dismissed

🔜 **Next module:** GitHub Actions Essentials — How to automate tests, builds, and deployments using workflows

---

## References

- [GitHub Branch Protection Rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches)
- [CODEOWNERS File](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners)
- [Azure Repos Branch Policies](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies)
---

## 📝 Chapter Quiz

**Question 1:** What is the core objective of Version Control Workflows in modern DevOps practices?

* [ ] To increase manual work
* [x] To automate, streamline, and increase the reliability of software delivery
* [ ] To eliminate code repositories
* [ ] To replace all developers

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** To automate, streamline, and increase the reliability of software delivery

**Explanation:** Version Control Workflows focuses on improving agility, reducing delivery friction, and ensuring operational reliability.
</details>

**Question 2:** Which file format is standard for defining automated workflows and pipelines associated with Version Control Workflows?

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

**Question 5:** What is a major benefit of Infrastructure as Code (IaC) when configuring Version Control Workflows?

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

**Question 8:** When a production deployment fails, what is the fastest way to restore service stability when using Version Control Workflows?

* [ ] Re-write the application from scratch
* [x] Execute an automated rollback to the last known good artifact
* [ ] Ignore the alert
* [ ] Turn off monitoring

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Execute an automated rollback to the last known good artifact

**Explanation:** Automated rollbacks quickly restore operational stability by redeploying the previous verified release artifact.
</details>
