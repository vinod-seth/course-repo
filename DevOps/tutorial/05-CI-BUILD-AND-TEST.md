# Module 5: CI — Build & Test

## Module Overview
- **Concept Core:** 75 min | **Hands-On Extension:** 75 min + Lab (120 min)
- **Real-World Scenarios:** FastCart build optimization; TenantHub test parallelization
- **Key Patterns:** Linting, unit tests, E2E tests, code coverage, quality gates, caching

---

## 🟢 Concept Core: What is CI and Why Quality Gates Matter

### Continuous Integration: The Foundation

**CI = Automated testing every time code changes**

Traditional flow (before CI):
```
Monday: Alice writes code
Tuesday: Bob does code review
Wednesday: Dev integrates code
Thursday: QA tests in staging
Friday: Bug found, Alice fixes
Monday: Deploy (hoping for no more bugs)
```

With CI:
```
10:00 AM: Alice pushes code
10:00:05 AM: Tests run automatically (2 min)
10:02 AM: Results show: "Tests passed, code coverage OK"
10:02:30 AM: Alice sees feedback and fixes minor issue
10:03 AM: Second test run, all green
10:03:30 AM: Code is production-ready (if approved)
```

**Key insight:** Feedback loop went from days to minutes.

### The Three Levels of Testing

| Level | What | Speed | Cost | Catch | Example |
|-------|------|-------|------|-------|---------|
| **Unit** | Single function/method | Seconds | Low | Logic bugs | `test_checkout_discount_calculation()` |
| **Integration** | Components working together | Minutes | Medium | API/DB issues | `test_user_registration_and_login()` |
| **E2E** | Full workflow (user perspective) | Minutes | High | Real user issues | `test_checkout_flow_with_payment()` |

**Rule of thumb (pyramid):**
```
            /\
           /E2E\         (Few, slow, expensive but catch real problems)
          /      \
         /Integration\  (More, medium speed)
        /             \
       /   Unit Tests   \ (Many, fast, cheap)
      /_______________\
```

**Goal:** Maximize unit tests (fast feedback), some integration, few E2E (they're slow).

### Code Quality Gates

Beyond tests, CI should enforce:
1. **Linting** — Code style consistency (indentation, naming, complexity)
2. **Code Coverage** — % of code exercised by tests (aim for 80%+)
3. **Security scanning** — Dependencies with vulnerabilities, hardcoded secrets
4. **Complexity analysis** — Functions shouldn't be too large (avoid "spaghetti code")

If any gate fails, build fails → code blocked from production.

### Cache Strategy for Speed

Building from scratch every time is slow. CI should cache:
- **Dependencies** (npm packages, pip venv, NuGet cache) — Biggest time saver
- **Build artifacts** (compiled objects) — Helps in monorepos
- **Docker base images** — Pulled from cache if unchanged

**Example:** With cache, build time: 15 min → 2 min (87% faster!)

---

## 🟢 Real-World Scenario 1: FastCart Build Pipeline

**Feature:** Checkout v2

**Build Steps:**
```
1. Checkout code (0.5 min)
2. Cache dependencies from npm cache (1 min saved vs. 3 min fresh)
3. Run linting (ESLint) — 1 min
   - Checks: indentation, naming, unused variables
   - Fails if: cyclomatic complexity > 10
4. Run unit tests — 2 min
   - Jest with coverage
   - Requires: 80% code coverage
5. Run integration tests (with staging DB) — 3 min
   - Test checkout API with real payment processor (test mode)
6. Build application — 1 min
   - Bundle JavaScript, optimize
7. Build Docker image — 2 min
   - Use cache from previous layer
8. Push to registry — 1 min

Total: 11 min (with cache), 18 min without
```

**Quality gates (if any fails, pipeline stops):**
- ✅ Linting must pass
- ✅ Tests must pass
- ✅ Coverage must be ≥80%
- ✅ No high-severity security vulnerabilities
- ✅ Docker image must be <500 MB

**Cost:** ~11 min × $0.008/min = $0.088 per build. 50 builds/day = $4.40/day = ~$132/month (free tier covers this).

---

## 🟢 Real-World Scenario 2: TenantHub Parallel Testing

**Challenge:** TenantHub's test suite has 5,000 tests taking 45 minutes to run serially. Too slow for CI!

**Solution:** Parallelization

```
Without parallelization:
Sequential test run: 45 min
(Tests run one after another)

With parallelization (10 agents):
Parallel test run: 6 min
(10 tests running simultaneously)
Cost: 1 agent × 45 min = 45 agent-min
```

**Implementation:**
```yaml
# Azure Pipelines: Parallelize tests
jobs:
  - job: Test
    strategy:
      parallel: 10  # Split tests across 10 agents
      maxParallel: 10
    steps:
      - script: |
          # Each agent runs a subset of tests
          pytest -k "test_${{ job.Index }}"
```

**Cost-benefit:**
- Test time: 45 min → 6 min (86% faster!)
- Developers get feedback quicker
- CI/CD pipeline faster overall
- Still same cost (~45 agent-minutes total, just distributed)

---

## 🔷 Hands-On Extension: Build a Real CI Workflow

### Part A: Multi-Language Test Setup

#### Tab: Python with pytest & coverage

```yaml
# .github/workflows/ci-python.yml
name: CI - Python Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11"]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
          cache: 'pip'  # Cache pip dependencies
      
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov flake8 black
      
      # Linting
      - name: Lint with flake8
        run: |
          # Critical errors
          flake8 src/ --count --select=E9,F63,F7,F82 --show-source --statistics
          # Warnings
          flake8 src/ --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
      
      - name: Format check with black
        run: black --check src/ tests/
      
      # Unit tests
      - name: Run unit tests with pytest
        run: |
          pytest \
            --cov=src \
            --cov-report=xml \
            --cov-report=html \
            --cov-report=term \
            --junitxml=junit.xml \
            --strict-markers
      
      # Check coverage
      - name: Check coverage threshold
        run: |
          coverage report --fail-under=80
      
      # Publish results
      - name: Publish test results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: junit.xml
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage.xml
          fail_ci_if_error: true
```

**What this does:**
- Caches pip packages (fast installs)
- Runs linting (flake8) and format checking (black)
- Runs tests with coverage
- Fails if coverage < 80%
- Uploads results to Codecov

---

#### Tab: JavaScript/Node.js with Jest

```yaml
# .github/workflows/ci-node.yml
name: CI - Node.js Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: ["16.x", "18.x", "20.x"]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci  # Clean install (reproducible)
      
      # Linting
      - name: Run ESLint
        run: npm run lint
      
      # Format check
      - name: Check formatting with Prettier
        run: npx prettier --check .
      
      # Unit tests
      - name: Run tests with Jest
        run: |
          npm test -- \
            --coverage \
            --maxWorkers=4 \
            --reporters=default \
            --reporters=jest-junit
        env:
          JEST_JUNIT_OUTPUT_DIR: ./junit
          JEST_JUNIT_OUTPUT_NAME: results.xml
      
      # Coverage check
      - name: Check coverage
        run: |
          npm test -- --coverage --coverageReporters=text-summary
      
      # Publish results
      - name: Publish test results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: ./junit/results.xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

---

#### Tab: C# / .NET with xUnit

```yaml
# .github/workflows/ci-dotnet.yml
name: CI - .NET Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        dotnet-version: ["6.0.x", "7.0.x", "8.0.x"]
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v3
        with:
          dotnet-version: ${{ matrix.dotnet-version }}
      
      - name: Restore dependencies
        run: dotnet restore
      
      - name: Build
        run: dotnet build --no-restore --configuration Release
      
      # Run tests with coverage
      - name: Run tests
        run: |
          dotnet test \
            --no-build \
            --configuration Release \
            --verbosity normal \
            --collect:"XPlat Code Coverage" \
            --logger "trx" \
            --results-directory "test-results"
      
      # Generate coverage report
      - name: Generate coverage report
        run: |
          dotnet tool install -g reportgenerator
          reportgenerator \
            -reports:"**/coverage.cobertura.xml" \
            -targetdir:"coverage-report" \
            -reporttypes:"HtmlInline;Badges"
      
      - name: Check coverage threshold
        run: |
          # Parse coverage and fail if < 80%
          COVERAGE=$(grep -oP 'CoveragePercent>\K[^<]+' coverage.cobertura.xml | head -1)
          if (( $(echo "$COVERAGE < 80" | bc -l) )); then
            echo "Coverage $COVERAGE% is below 80% threshold"
            exit 1
          fi
      
      - name: Publish test results
        uses: EnricoMi/publish-unit-test-result-action@v2
        if: always()
        with:
          files: test-results/**/*.trx
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage-report/SummaryGithub.txt
```

---

### Part B: Code Quality Gates

```yaml
# Add to your workflow
- name: SonarQube scan
  uses: SonarSource/sonarcloud-github-action@master
  with:
    args: >
      -Dsonar.projectKey=MY_PROJECT
      -Dsonar.organization=MY_ORG
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    SONARCLOUD_TOKEN: ${{ secrets.SONARCLOUD_TOKEN }}

- name: Security scan with Trivy
  uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'fs'
    scan-ref: '.'
    format: 'sarif'
    output: 'trivy-results.sarif'

- name: Upload SARIF
  uses: github/codeql-action/upload-sarif@v2
  with:
    sarif_file: 'trivy-results.sarif'
```

---

## 🔷 Lab: Build a Complete CI Workflow with Quality Gates

### Objectives
- ✅ Write tests with coverage target (80%)
- ✅ Add linting and format checks
- ✅ Set up CI workflow
- ✅ See tests run automatically
- ✅ Fail a test, watch pipeline fail
- ✅ Fix test, watch pipeline pass
- ✅ Verify coverage report

### Prerequisites
- GitHub repo with code + tests
- 120 minutes

### Lab Steps

#### Step 1: Write Code with Tests

**Tab: Python Example**

Create `src/checkout.py`:
```python
def calculate_discount(subtotal, is_member):
    """Calculate discount based on membership."""
    if not isinstance(subtotal, (int, float)):
        raise ValueError("Subtotal must be numeric")
    if subtotal < 0:
        raise ValueError("Subtotal cannot be negative")
    
    if is_member and subtotal >= 100:
        return subtotal * 0.1  # 10% discount
    elif subtotal >= 50:
        return subtotal * 0.05  # 5% discount
    return 0

def process_checkout(items, is_member):
    """Process checkout and return total."""
    subtotal = sum(item['price'] for item in items)
    discount = calculate_discount(subtotal, is_member)
    total = subtotal - discount
    return {'subtotal': subtotal, 'discount': discount, 'total': total}
```

Create `tests/test_checkout.py`:
```python
import pytest
from src.checkout import calculate_discount, process_checkout

class TestDiscount:
    def test_member_discount_high_subtotal(self):
        # Member with $150 → 10% discount = $15
        assert calculate_discount(150, True) == 15.0
    
    def test_non_member_low_subtotal(self):
        # Non-member with $30 → no discount
        assert calculate_discount(30, False) == 0
    
    def test_non_member_medium_subtotal(self):
        # Non-member with $75 → 5% discount = $3.75
        assert calculate_discount(75, False) == 3.75
    
    def test_invalid_subtotal_negative(self):
        with pytest.raises(ValueError):
            calculate_discount(-10, True)
    
    def test_invalid_subtotal_string(self):
        with pytest.raises(ValueError):
            calculate_discount("100", True)

class TestCheckout:
    def test_checkout_flow_member(self):
        items = [{'price': 50}, {'price': 60}]
        result = process_checkout(items, True)
        assert result['subtotal'] == 110
        assert result['discount'] == 11.0
        assert result['total'] == 99.0
```

#### Step 2: Create CI Workflow

Copy the Python workflow from Part A above to `.github/workflows/ci.yml`

#### Step 3: Add Requirements and Setup

Create `requirements.txt`:
```
pytest==7.4.0
pytest-cov==4.1.0
flake8==6.0.0
black==23.7.0
```

#### Step 4: Commit and Push

```bash
git add src/ tests/ requirements.txt .github/workflows/
git commit -m "Add checkout service with tests"
git push
```

#### Step 5: Watch CI Run

1. Go to **Actions** tab
2. See "CI - Python Tests" running
3. Wait for completion
4. View results:
   - Linting: ✅ Pass
   - Tests: ✅ 5/5 pass
   - Coverage: ✅ 95%
   - Summary: "All checks passed"

#### Step 6: Break a Test

Edit `tests/test_checkout.py`:
```python
def test_member_discount_high_subtotal(self):
    # Intentionally wrong expected value
    assert calculate_discount(150, True) == 20.0  # Should be 15.0!
```

Commit and push:
```bash
git commit -am "Break test intentionally"
git push
```

Watch CI fail:
- Tests: ❌ FAILED (1 failure out of 5)
- Coverage: Still OK
- Overall: ❌ Pipeline FAILED
- GitHub shows red X on PR/commit

#### Step 7: Fix the Test

Fix back to correct value:
```python
assert calculate_discount(150, True) == 15.0
```

Push:
```bash
git commit -am "Fix test"
git push
```

Watch CI pass again:
- All steps green
- Coverage 95%
- "All checks passed" ✅

#### Step 8: View Coverage Report

1. In GitHub Actions run, scroll to "Upload coverage"
2. Click Codecov link (if configured)
3. See coverage breakdown by file
4. Verify: checkout.py = 100% covered

### Validation Checklist

- ✅ Workflow file created
- ✅ Tests run automatically on push
- ✅ Coverage checked (threshold enforced)
- ✅ Linting enforced
- ✅ Failed test showed in CI
- ✅ Fixed test showed pipeline pass
- ✅ Coverage report viewable

---

## 🟢 Assessment: CI Patterns

> **Question 1:** Why is code coverage important?
> - A) Because 100% coverage means no bugs
> - B) To ensure tested code paths, not a guarantee of correctness
> - C) It's required by law
> - D) To slow down development
> 
> **Answer:** B — Coverage measures which lines are executed by tests. High coverage (80%+) reduces blind spots, but doesn't guarantee correctness (test quality matters too).

> **Question 2:** You have 5,000 tests taking 30 minutes serially. What's the fastest way to reduce this?
> - A) Delete some tests
> - B) Make tests run in parallel
> - C) Skip testing in CI
> - D) Use a faster computer
> 
> **Answer:** B — Parallelization. With 10 agents: 30 min / 10 ≈ 3 min. Cost is same, time is much faster.

---

## Next Steps

✅ **You now understand:**
- The importance of automated testing in CI
- How to set up multi-language CI pipelines
- Code coverage and quality gates
- Parallelization for speed

🔜 **Next module:** CD — Deployment (artifact promotion, deployment strategies)

---

## References

- [GitHub Actions Caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [Codecov Integration](https://codecov.io/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
