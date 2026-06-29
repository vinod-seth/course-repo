# Module 8: Secrets, Security & Compliance

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/08-SECRETS-SECURITY-COMPLIANCE.md)


## Module Overview
- **Concept Core:** 90 min | **Hands-On Extension:** 90 min + Lab (120 min)
- **Real-World Scenarios:** FastCart payment API secrets; TenantHub HIPAA compliance
- **Key Tools:** GitHub Secrets, Azure Key Vault, OIDC, Audit Logging, Branch Policies

---

## 🟢 Concept Core: Secrets & Credential Management

### The Problem: Hardcoded Secrets

**Dangerous (DON'T DO THIS!):**
```python
# app.py
DATABASE_PASSWORD = "super-secret-password-123"  # ❌ In code!
STRIPE_API_KEY = "sk_live_abc123..."  # ❌ Exposed!
AWS_ACCESS_KEY = "AKIAIOSFODNN7EXAMPLE"  # ❌ DISASTER!
```

**Why it's a disaster:**
- Secret ends up in Git history (forever)
- If repo is public, anyone has credentials
- Attacker uses credentials to access prod
- Hacker drains bank account, steals customer data
- **Cost:** Millions in fraud + HIPAA fines + reputation damage

### The Solution: Secrets Management

**Secrets never in code:**
```python
# app.py
import os
DATABASE_PASSWORD = os.getenv('DATABASE_PASSWORD')  # ✅ From environment
STRIPE_API_KEY = os.getenv('STRIPE_API_KEY')  # ✅ From environment
```

**Where do secrets come from?**

| Source | Use Case | Security | Speed |
|--------|----------|----------|-------|
| **GitHub Secrets** | GitHub Actions | ✅ Encrypted, masked in logs | ✅ Instant |
| **Azure Key Vault** | Azure apps, pipelines | ✅✅ Vault, HSM options | ✅ Fast |
| **.env file** (local dev) | Development only | ⚠️ Not for production | ✅ Fast |
| **AWS Secrets Manager** | AWS workloads | ✅✅ Vault, rotation | ✅ Fast |
| **HashiCorp Vault** | Enterprise, multi-cloud | ✅✅✅ Complex, powerful | ⚠️ Slower |

---

## 🟢 Secret Rotation & OIDC

### Secret Rotation

**Problem:** Database password stored in GitHub Secret for 2 years. Attacker found it. Now what?

**Solution:** Rotate secrets regularly (quarterly, or on compromise)

```
Month 1: Create secret v1 in GitHub Secrets + Key Vault
         └── Pipeline uses v1

Month 4: Create new secret v2
         ├── Add v2 to GitHub Secrets
         ├── Deploy code to use v2
         ├── Update database to invalidate v1
         └── Delete v1 from GitHub Secrets

Result: Old secret useless even if leaked
```

**Best practice:** Automate with:
- GitHub secret auto-rotation (in preview)
- Azure Key Vault auto-rotate
- HashiCorp Vault leases

---

### OIDC: Passwordless Authentication

**Traditional (risky):**
```
GitHub Actions stores: AWS_ACCESS_KEY_ID + AWS_SECRET_ACCESS_KEY
├── These are long-lived credentials
├── If leaked, attacker has permanent access
└── Must manually rotate
```

**OIDC (modern, secure):**
```
GitHub Issues short-lived token to runner:
├── Valid for only 5 minutes
├── Only valid for this specific GitHub repo + workflow
├── No credentials stored in GitHub
└── Much safer, zero manual rotation
```

**Setup (for Azure):**
```yaml
- name: Azure login (OIDC)
  uses: azure/login@v1
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
  # No password! OIDC handles authentication
```

**Benefits:**
- ✅ Short-lived tokens (5 min)
- ✅ Scoped to specific workflows
- ✅ No secrets stored long-term
- ✅ Audit trail (which workflow accessed what)

---

## 🟢 Real-World Scenario 1: FastCart Payment Secrets

**Secrets needed:**
- Stripe API key (production)
- Database password (RDS)
- Azure Container Registry credentials
- SendGrid API key (email)

**Management:**
```
GitHub Secrets (for GitHub Actions):
├── STRIPE_API_KEY_PROD
├── DATABASE_PASSWORD
└── ACR_PASSWORD

Azure Key Vault (for App Service):
├── stripe-api-key-prod
├── database-password
└── sendgrid-key

Process:
1. Developer pushes code
2. GitHub Actions (in CI) retrieves from GitHub Secrets
3. Builds and deploys artifact
4. At deployment, App Service retrieves from Key Vault (OIDC)
5. Each system has least-privilege access

Rotation (Quarterly):
├── Month 4: Create new password in RDS
├── Add new password to Azure Key Vault + GitHub Secrets (as v2)
├── Deploy code (uses v2)
├── Disable old password (v1) in RDS
└── Remove v1 from GitHub + Azure
```

---

## 🟢 Real-World Scenario 2: TenantHub HIPAA Compliance

**Regulatory requirement:** HIPAA (Healthcare data protection)
- All access to customer data must be logged
- Audit trail: "Who accessed what, when, why"
- Encryption in transit + at rest
- No default passwords, MFA required

**TenantHub Solution:**
```
Secrets Management:
├── All credentials in Azure Key Vault (HIPAA compliant)
├── No hardcoded secrets in code
├── Auto-rotate every 90 days
└── Access logged to Azure Monitor

Deployment Audit:
├── Every deploy requires approval (logged)
├── Approval record: "David approved v2.1.0 at 2024-01-15 10:30 UTC"
├── Sensitive data access logged
└── Monthly audit report to compliance team

Branch Policy Enforcement:
├── No direct pushes to main (all PRs)
├── Code review mandatory (both eyes on sensitive changes)
├── CODEOWNERS for healthcare-specific code (Alice must review)
└── All commits signed (git commit -S)
```

---

## 🔷 Hands-On Extension: GitHub Secrets & Azure Key Vault

### Part A: GitHub Secrets in Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy

on: [workflow_dispatch]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}  # From GitHub Secrets
          password: ${{ secrets.DOCKER_PASSWORD }}  # From GitHub Secrets (masked in logs)
      
      - name: Build and push Docker image
        run: |
          docker build -t myapp:latest .
          docker push myapp:latest
      
      - name: Deploy to App Service
        env:
          DB_PASSWORD: ${{ secrets.DATABASE_PASSWORD }}  # Injected as env var
          STRIPE_KEY: ${{ secrets.STRIPE_API_KEY }}
        run: |
          echo "Deploying with DB_PASSWORD=***" # Secret is masked!
          ./deploy.sh
```

**Security features:**
- Secrets encrypted at rest in GitHub
- Only decrypted in workflow step
- Masked in logs (shows `***` instead of actual value)
- Audit log: who accessed which secret

---

### Part B: Azure Key Vault Integration

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: 'KeyVaultSecrets'  # Link to Key Vault

stages:
  - stage: Build
    jobs:
      - job: BuildJob
        steps:
          - checkout: self
          
          # Get secrets from Key Vault
          - task: AzureKeyVault@2
            inputs:
              azureSubscription: 'Azure'
              KeyVaultName: 'mykeyvault'
              SecretsFilter: '*'  # Fetch all secrets
              RunAsPreJob: false
          
          - script: |
              # Secrets are now available as variables
              # But NEVER echo them!
              echo "Deploying..."
              ./deploy.sh
            env:
              DB_PASSWORD: $(database-password)  # From Key Vault
              STRIPE_API_KEY: $(stripe-api-key)
            displayName: 'Deploy with secrets'
```

**Key Vault benefits:**
- ✅ Centralized secret storage
- ✅ RBAC: Only specific users/apps can read specific secrets
- ✅ Audit: All access logged
- ✅ Rotation: Azure can auto-rotate (for Azure resources)

---

### Part C: OIDC Setup (Azure + GitHub)

**Prerequisites:**
- Azure tenant
- GitHub repo

**Step 1: Create Azure Service Principal**

```bash
az ad sp create-for-rbac \
  --name github-oidc \
  --role contributor \
  --scopes /subscriptions/SUBSCRIPTION_ID

# Output:
# {
#   "clientId": "abc123...",
#   "tenantId": "xyz789...",
#   "subscriptionId": "sub123..."
# }
```

**Step 2: Trust GitHub in Azure**

```bash
# Allow GitHub to authenticate as this service principal
az identity federated-credentials create \
  --name github-oidc \
  --identity-name my-identity \
  --issuer https://token.actions.githubusercontent.com \
  --subject repo:owner/repo:ref:refs/heads/main \
  --audiences api://AzureADTokenExchange
```

**Step 3: Store IDs as GitHub Secrets**

1. Go to **Settings** → **Secrets**
2. Add:
   - `AZURE_CLIENT_ID` = abc123...
   - `AZURE_TENANT_ID` = xyz789...
   - `AZURE_SUBSCRIPTION_ID` = sub123...

**Step 4: Use in Workflow**

```yaml
- name: Azure Login (OIDC)
  uses: azure/login@v1
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

- name: Deploy to Azure
  run: az webapp up -n myapp -g mygroup
```

**Result:** No credentials stored! OIDC handles it.

---

## 🔷 Lab: Set Up Secrets & OIDC

### Objectives
- ✅ Create GitHub Secrets
- ✅ Use secrets in workflow
- ✅ Verify secrets are masked in logs
- ✅ Set up Azure Key Vault
- ✅ Access Key Vault from pipeline
- ✅ Configure OIDC

### Prerequisites
- GitHub repo
- Azure subscription
- 120 minutes

### Lab Steps

#### Step 1: Create GitHub Secrets

1. Go to repo **Settings** → **Secrets and variables** → **Actions**
2. Add secrets:
   - Name: `DB_PASSWORD` | Value: `temp-test-password-123`
   - Name: `API_KEY` | Value: `test-api-key-xyz`

#### Step 2: Create Workflow Using Secrets

**File: `.github/workflows/test-secrets.yml`**

```yaml
name: Test Secrets

on: [push]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Use secrets safely
        env:
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          echo "Testing with secrets..."
          # Never echo secrets!
          # echo $DB_PASSWORD  # ❌ DON'T DO THIS
          # Instead, use them:
          python -c "import os; print('DB connected' if os.getenv('DB_PASSWORD') else 'Failed')"
```

#### Step 3: Commit and Watch

```bash
git add .github/
git commit -m "Add secrets test workflow"
git push
```

View workflow run:
- Should show "Testing with secrets..."
- No actual password in logs (masked)

#### Step 4: Create Azure Key Vault

```bash
az keyvault create \
  -n mykeyvault-$(date +%s) \
  -g mygroup \
  --sku standard

# Add secrets
az keyvault secret set \
  --vault-name mykeyvault-xxxxx \
  --name database-password \
  --value "prod-db-password-secure"

az keyvault secret set \
  --vault-name mykeyvault-xxxxx \
  --name stripe-api-key \
  --value "sk_test_abc123"
```

#### Step 5: Create Pipeline Using Key Vault

**File: `azure-pipelines.yml`**

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
  - checkout: self
  
  - task: AzureKeyVault@2
    inputs:
      azureSubscription: 'Azure'
      KeyVaultName: 'mykeyvault-xxxxx'
      SecretsFilter: '*'
  
  - script: |
      echo "Accessing Key Vault secrets..."
      echo "Database password set: $(database-password)"  # This is safe in Azure context
    displayName: 'Use secrets from Key Vault'
    env:
      DB_PASSWORD: $(database-password)
```

#### Step 6: Set Up OIDC (Optional)

```bash
# Create Azure identity
az identity create -n github-oidc -g mygroup

# Trust GitHub
az identity federated-credentials create \
  --name github-oidc \
  --identity-name github-oidc \
  --issuer https://token.actions.githubusercontent.com \
  --subject repo:your-org/your-repo:ref:refs/heads/main \
  --audience api://AzureADTokenExchange
```

Add GitHub Secrets:
- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`

Use in workflow:
```yaml
- uses: azure/login@v1
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

### Validation Checklist

- ✅ GitHub Secrets created
- ✅ Secrets used in workflow without logging
- ✅ Secrets masked in logs (show ***)
- ✅ Azure Key Vault created with secrets
- ✅ Pipeline accesses Key Vault
- ✅ OIDC configured (no hardcoded credentials)
- ✅ Audit log shows access

---

## 🟢 Assessment: Secrets & Security

> **Question 1:** You see this in someone's code: `DATABASE_PASSWORD = "super-secret-123"`. What's wrong?
> - A) Nothing, it's secure
> - B) Password visible in Git history forever, anyone with repo access can see it, disaster if public repo
> - C) It's encrypted
> - D) Only security team can see it
> 
> **Answer:** B — Hardcoded secrets in code are a nightmare. Even in private repos, 100+ developers have access. If someone leaves company, they have credentials. Git history is permanent.

> **Question 2:** OIDC vs. storing AWS credentials in GitHub Secrets. Which is more secure?
> - A) They're the same
> - B) OIDC: short-lived tokens, scoped, no long-term secrets. Credentials: stored forever, full access
> - C) Credentials are more secure (you control them)
> - D) OIDC is only for Azure
> 
> **Answer:** B — OIDC tokens expire in minutes. Even if leaked, they're useless after 5 minutes. Credentials stored in GitHub Secrets are valid forever (until rotated, which many teams forget).

---

## Next Steps

✅ **You now understand:**
- Secret management best practices
- GitHub Secrets and Azure Key Vault
- OIDC for passwordless authentication
- Audit logging and compliance

🔜 **Next modules:** 9–12 (Artifacts, Monitoring, Patterns, Capstone)

---

## References

- [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/overview)
- [OIDC in GitHub Actions](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
---

## 📝 Chapter Quiz

**Question 1:** What is the core objective of Secrets Security Compliance in modern DevOps practices?

* [ ] To increase manual work
* [x] To automate, streamline, and increase the reliability of software delivery
* [ ] To eliminate code repositories
* [ ] To replace all developers

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** To automate, streamline, and increase the reliability of software delivery

**Explanation:** Secrets Security Compliance focuses on improving agility, reducing delivery friction, and ensuring operational reliability.
</details>

**Question 2:** Which file format is standard for defining automated workflows and pipelines associated with Secrets Security Compliance?

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

**Question 5:** What is a major benefit of Infrastructure as Code (IaC) when configuring Secrets Security Compliance?

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

**Question 8:** When a production deployment fails, what is the fastest way to restore service stability when using Secrets Security Compliance?

* [ ] Re-write the application from scratch
* [x] Execute an automated rollback to the last known good artifact
* [ ] Ignore the alert
* [ ] Turn off monitoring

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Execute an automated rollback to the last known good artifact

**Explanation:** Automated rollbacks quickly restore operational stability by redeploying the previous verified release artifact.
</details>
