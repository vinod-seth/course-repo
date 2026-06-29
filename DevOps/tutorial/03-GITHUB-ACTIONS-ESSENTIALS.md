# Module 3: GitHub Actions Essentials

## Module Overview
- **Concept Core:** 60 min | **Hands-On Extension:** 60 min + Lab (90 min)
- **Real-World Scenarios:** FastCart CI for checkout feature; TenantHub daily build automation
- **Key Tools:** GitHub Actions, Workflows, Jobs, Steps, Runners, GitHub-hosted vs. self-hosted

---

## 🟢 Concept Core: GitHub Actions Model

### What Are GitHub Actions?

GitHub Actions is a **continuous integration and continuous delivery (CI/CD) platform built directly into GitHub**.

When you push code to GitHub, Actions can automatically:
1. **Run tests** (does the code work?)
2. **Build** (compile, package the code)
3. **Deploy** (move artifact to staging or production)
4. **Notify** (send alerts to Slack, email, etc.)

**Key advantage:** Actions live in your repo (`.github/workflows/`) alongside your code. Non-developers can read and understand them.

### The Anatomy of a GitHub Actions Workflow

A **workflow** is a YAML file that defines:
- **When** it runs (trigger: push, PR, schedule, manual)
- **What** it runs (jobs, steps, scripts)
- **Where** it runs (runners: GitHub-hosted Ubuntu, Windows, macOS; or self-hosted)

```
TRIGGER (e.g., push to main)
    ↓
WORKFLOW STARTS
    ├── JOB 1: Build
    │   ├── Step 1: Checkout code
    │   ├── Step 2: Install dependencies
    │   └── Step 3: Build application
    │
    └── JOB 2: Test (runs in parallel)
        ├── Step 1: Checkout code
        ├── Step 2: Install dependencies
        └── Step 3: Run tests
    ↓
BOTH JOBS COMPLETE
    ↓
SUCCESS/FAILURE NOTIFICATION
```

### Core Concepts

| Concept | Meaning | Example |
|---------|---------|---------|
| **Workflow** | YAML file in `.github/workflows/` | `ci.yml`, `deploy.yml` |
| **Trigger** | When the workflow starts | `on: [push, pull_request]` |
| **Job** | A unit of work (runs on a runner) | `build`, `test`, `deploy` |
| **Step** | A command or action within a job | `run: npm test` or `uses: actions/checkout@v3` |
| **Action** | Reusable code from the marketplace | `actions/checkout`, `actions/setup-node` |
| **Runner** | The server where jobs run | `ubuntu-latest`, `windows-latest`, or self-hosted |
| **Artifact** | Output of a job (can be downloaded) | Docker image, compiled binary, test reports |
| **Environment** | Secret/config storage for a stage | `staging`, `production` |

### GitHub-Hosted vs. Self-Hosted Runners

#### GitHub-Hosted Runners (Free Tier)
- **Provided by GitHub**
- **Options:** Ubuntu, Windows, macOS
- **Cost:** Free tier includes 2,000 minutes/month for private repos
- **Performance:** Good for most CI/CD (tests, build, simple deployments)
- **When to use:** Small teams, standard workloads

```
GitHub manages:
├── Provisioning (spins up a machine)
├── OS + software (GitHub Actions pre-installed)
├── Networking
└── Cleanup (deletes machine after job)

You focus on:
└── Your workflow (just write YAML)
```

#### Self-Hosted Runners (Enterprise)
- **You provision and manage**
- **Can be:** On-prem server, AWS EC2, your laptop
- **Cost:** Free, but you pay for infrastructure
- **Performance:** Fast for large/frequent builds (your own hardware)
- **When to use:** Private dependencies, IP restrictions, GPU workloads, extreme scale

```
You manage:
├── Infrastructure (EC2, on-prem server)
├── OS + software installation
├── Networking & security
└── Cleanup

GitHub manages:
└── Orchestration (tells your runner what to do)
```

### Why GitHub-Hosted is Enough for Most Teams

| Scenario | GitHub-Hosted | Self-Hosted |
|----------|---------------|-------------|
| **Startup** (1–5 devs) | ✅ Yes | Overkill |
| **Small company** (5–50 devs) | ✅ Yes, usually | Only if build > 10 min |
| **Enterprise** (1,000+ devs) | ⚠️ Cost scales | ✅ Better ROI |
| **Private dependencies** | ✅ VPN possible | ✅ Easier |
| **GPU workloads** (ML) | ❌ No | ✅ Yes |
| **Build time > 30 min** | ⚠️ Gets expensive | ✅ Better |

**Rule of thumb:** If you're paying > $200/month for CI/CD, consider self-hosted.

---

## 🟢 Real-World Scenario 1: FastCart Checkout CI

**Feature:** Alice's one-click checkout

**Workflow Trigger:** When Alice pushes to `feature/checkout-v2`

**Workflow Steps:**
```
1. Checkout code
2. Set up Node.js runtime
3. Install npm dependencies
4. Run linting (style checks)
5. Run unit tests
6. Run E2E tests (with staging DB)
7. Build Docker image
8. Push image to GitHub Packages
9. Notify Slack: "Build passed! Version 1.5.0-rc.1 ready for staging"
```

**Timeline:**
- 10:00 AM: Alice pushes code
- 10:00:05 AM: GitHub notices the push, triggers workflow
- 10:02 AM: Tests complete (linting, unit, E2E)
- 10:03 AM: Docker image built and pushed to registry
- 10:03:30 AM: Slack notification: "Ready for staging"
- **Total: 3.5 minutes from push to ready-for-deploy**

Without CI:
- Alice has to manually run: npm install, npm test, npm build, docker build, docker push
- Takes 15 minutes
- Inconsistent (did she forget a step?)
- Error-prone (what if she's on a different OS than production?)

---

## 🟢 Real-World Scenario 2: TenantHub Scheduled Deploys

**Feature:** AI maintenance routing

**Workflow Trigger:** Daily at 2 AM UTC (to deploy to "Fast" tenant pool)

**Workflow Steps:**
```
1. Checkout code (main branch)
2. Run full test suite
3. Build artifact (version: 2.1.0-2024-01-15)
4. Deploy to "Fast" tenant pool (via Azure Pipelines)
5. Monitor health for 5 minutes
6. If metrics good: notify team "Deploy successful"
7. If metrics bad: automatic rollback
```

**Timeline:**
- 2:00 AM: Workflow starts
- 2:05 AM: Tests pass
- 2:06 AM: Artifact built
- 2:07 AM: Deploy to tenants
- 2:12 AM: Health checks pass
- 2:12:30 AM: Team wakes up to "All good, 4,800 tenants updated"

**Cost-benefit:**
- Manual deploy: 30 minutes + on-call person
- Automated deploy: 12 minutes + zero human intervention
- **Savings: 18 min/day × 365 days = 110 hours/year**

---

## 🔷 Hands-On Extension: Your First GitHub Actions Workflow

### Part A: Create a Simple CI Workflow

**Goal:** Build a workflow that runs tests every time code is pushed.

#### Tab: Python with pytest

**File: `.github/workflows/ci-python.yml`**

```yaml
# This workflow runs tests for a Python project
name: CI - Python

# Trigger: Run when code is pushed OR pull request is created
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    # Runner: Use GitHub's free Ubuntu machine
    runs-on: ubuntu-latest
    
    # Test against multiple Python versions
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11"]
    
    steps:
      # Step 1: Checkout the code
      - uses: actions/checkout@v3
      
      # Step 2: Set up Python
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      
      # Step 3: Install dependencies
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov
      
      # Step 4: Run linting
      - name: Run linting (flake8)
        run: |
          pip install flake8
          # Stop if any errors
          flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
          # Warn on style issues
          flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      
      # Step 5: Run tests
      - name: Run tests with pytest
        run: |
          pytest --cov=src --cov-report=xml --cov-report=html
      
      # Step 6: Upload coverage report
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
          fail_ci_if_error: true
```

**What this does:**
1. Runs on push to `main` or `develop`
2. Runs on every PR
3. Tests against Python 3.9, 3.10, 3.11 (in parallel)
4. Lints code (catches style issues)
5. Runs tests with coverage
6. Uploads coverage to Codecov.io (free service)

**Key parts:**
- `${{ matrix.python-version }}`: Runs the job 3 times (once per version)
- `uses:` — Reusable actions from GitHub Marketplace
- `run:` — Raw shell commands
- `with:` — Parameters passed to an action

---

#### Tab: JavaScript/Node.js with Jest

**File: `.github/workflows/ci-node.yml`**

```yaml
name: CI - Node.js

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: ["16.x", "18.x", "20.x"]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'  # Cache node_modules for speed
      
      - name: Install dependencies
        run: npm ci  # ci = clean install (reproducible)
      
      - name: Run linting (ESLint)
        run: npm run lint
      
      - name: Run tests with Jest
        run: npm test -- --coverage --passWithNoTests
      
      - name: Build application
        run: npm run build
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

---

#### Tab: C# / .NET with xUnit

**File: `.github/workflows/ci-dotnet.yml`**

```yaml
name: CI - .NET

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        dotnet-version: ["6.0.x", "7.0.x", "8.0.x"]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET ${{ matrix.dotnet-version }}
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ matrix.dotnet-version }}
      
      - name: Restore dependencies
        run: dotnet restore
      
      - name: Build
        run: dotnet build --no-restore --configuration Release
      
      - name: Run tests
        run: dotnet test --no-build --verbosity normal --configuration Release --collect:"XPlat Code Coverage"
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
```

---

### Part B: Add a Build & Publish Step

Once tests pass, build a Docker image and publish it.

**File: `.github/workflows/ci-with-publish.yml`**

```yaml
name: CI with Docker Build

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io  # GitHub Container Registry
  IMAGE_NAME: ${{ github.repository }}  # owner/repo

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: npm test
  
  build:
    needs: test  # Only run if test job succeeds
    runs-on: ubuntu-latest
    
    permissions:
      contents: read
      packages: write  # Allow pushing to ghcr.io
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}  # Your GitHub username
          password: ${{ secrets.GITHUB_TOKEN }}  # Automatic GitHub token
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=sha
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

**What this does:**
1. Runs tests first
2. If tests pass, builds a Docker image
3. Tags image with git branch/version/commit SHA
4. Pushes to GitHub Container Registry (GHCR)
5. Uses `${{ secrets.GITHUB_TOKEN }}` for authentication (auto-provided by GitHub)

---

### Part C: GitHub Secrets (Secure Variables)

**Scenario:** Your CI needs to push to Docker Hub or deploy to Azure, but you need to keep API keys secret.

#### How to Use GitHub Secrets

1. **Go to Repository Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `DOCKER_USERNAME`
4. Value: `your-dockerhub-username` (or any API key)
5. Click **Add secret**

**In your workflow:**
```yaml
- name: Log in to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

**Important:** Secrets are:
- ✅ Encrypted at rest
- ✅ Never logged to workflow output
- ✅ Only decrypted inside your job
- ✅ Masked in logs (shows `***`)

**Bad practice (don't do this!):**
```yaml
# NEVER hardcode secrets!
- name: Deploy
  run: docker login -u myusername -p mypassword  # ❌ EXPOSED!
```

---

## 🔷 Lab: Build a Real CI Workflow

### Objectives
- ✅ Create a GitHub Actions workflow file
- ✅ Trigger workflow on push
- ✅ Run tests automatically
- ✅ Build and push Docker image
- ✅ View workflow runs in GitHub UI
- ✅ Fix a failing test and re-run

### Prerequisites
- GitHub repo (public or private)
- Code with tests (Python/Node.js/C#)
- Dockerfile in repo root
- 90 minutes

### Lab Steps

#### Step 1: Create Workflow File

Create `.github/workflows/ci.yml` in your repo:

```bash
mkdir -p .github/workflows
```

**Tab: Copy this for Python projects:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest
```

**Tab: Copy this for Node.js projects:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: "18"
          cache: "npm"
      - run: npm ci
      - run: npm test
      - run: npm run build
```

**Tab: Copy this for .NET projects:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-dotnet@v3
        with:
          dotnet-version: "7.0.x"
      - run: dotnet restore
      - run: dotnet build
      - run: dotnet test
```

#### Step 2: Commit and Push

```bash
git add .github/workflows/ci.yml
git commit -m "chore: Add GitHub Actions CI workflow"
git push origin main
```

#### Step 3: View Workflow Run

1. Go to your GitHub repo
2. Click **Actions** tab
3. You should see "CI" workflow running
4. Click on it to see steps
5. Wait for it to complete (green checkmark = success, red X = failed)

#### Step 4: Intentionally Break a Test

Make the test fail to see what GitHub Actions reports:

**Tab: Python:**
```python
# tests/test_checkout.py
def test_checkout():
    result = checkout()
    assert result == "SUCCESS"  # Temporarily change to check the workflow catches it
```

**Tab: JavaScript:**
```javascript
// tests/checkout.test.js
test('checkout returns success', () => {
    const result = checkout();
    expect(result).toBe('WRONG');  // Intentionally wrong
});
```

#### Step 5: Commit the Broken Test

```bash
git add tests/
git commit -m "test: Intentionally break test to see CI failure"
git push origin main
```

#### Step 6: Watch CI Fail

1. Go to **Actions** tab
2. You'll see the workflow running
3. Wait for it to complete
4. You'll see the test step showing red X and error output
5. **Important:** The workflow log shows exactly which assertion failed and why

#### Step 7: Fix the Test

Revert your test break:

```bash
git revert HEAD  # Or manually fix the test
git push origin main
```

1. Workflow runs again automatically
2. Tests pass (green checkmark)
3. You've now seen the full cycle: code → push → CI → pass/fail

#### Step 8: Add Docker Build (Optional)

Create a `Dockerfile`:

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY src/ .

CMD ["python", "app.py"]
```

Add to `.github/workflows/ci.yml`:

```yaml
name: CI with Docker

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: pip install -r requirements.txt
      - run: pytest
  
  build:
    needs: test  # Only run if test succeeds
    runs-on: ubuntu-latest
    permissions:
      packages: write
    steps:
      - uses: actions/checkout@v3
      - uses: docker/setup-buildx-action@v2
      - uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:latest
```

Push and watch Docker image get built and pushed to GHCR.

### Validation Checklist

- ✅ Workflow file created in `.github/workflows/`
- ✅ Workflow triggered on push
- ✅ Tests ran automatically
- ✅ Viewed workflow run in GitHub Actions tab
- ✅ Intentionally broke and fixed a test
- ✅ Saw CI failure message
- ✅ Saw CI success message
- ✅ (Optional) Docker image built and pushed

---

## 🟢 Assessment: GitHub Actions Concepts

> **Question 1:** What is the difference between a job and a step in GitHub Actions?
> - A) Job runs on runner, step is a command in a job
> - B) They are the same thing
> - C) Step runs on runner, job is a collection of steps
> - D) Jobs are for testing, steps are for building
> 
> **Answer:** A — A job is a unit of work that runs on a runner (e.g., "test"), and a step is a command or action within that job (e.g., "run pytest").

<details>
<summary>Click to expand</summary>

Hierarchy:
```
Workflow
└── Job 1 (runs on runner)
    ├── Step 1 (run: command)
    ├── Step 2 (uses: action)
    └── Step 3 (run: another command)
└── Job 2 (runs on another runner, in parallel)
    ├── Step 1
    └── Step 2
```

</details>

> **Question 2:** You have 100 developers pushing to your repo. GitHub-hosted runners cost $0.008 per minute. Estimate daily CI cost.
> - A) Free (first 2,000 minutes/month)
> - B) ~$1/day (~3 hours build time across all devs)
> - C) ~$50/day (100 devs × 30 min builds)
> - D) Incalculable without knowing build times
> 
> **Answer:** D — Depends on how long each build takes. Rule: 100 devs × (1–5 pushes/day) × (2–10 min/build) = 200–5,000 minutes/day. Free tier covers first 2,000 minutes/month, so you'd stay free or pay $0.16–$2.40/day depending on build speed.

---

## Next Steps

✅ **You now understand:**
- How GitHub Actions workflows are structured
- When to use GitHub-hosted vs. self-hosted runners
- How to write a CI workflow (test, build, publish)
- How to use GitHub Secrets securely

🔜 **Next module:** Azure Pipelines Essentials — For multi-stage, enterprise deployments

---

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [Pricing: GitHub-Hosted Runners](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)
- [Codecov Integration](https://codecov.io/)
