# Module 4: Azure Pipelines Essentials

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/04-AZURE-PIPELINES-ESSENTIALS.md)


## Module Overview
- **Concept Core:** 60 min | **Hands-On Extension:** 60 min + Lab (90 min)
- **Real-World Scenarios:** FastCart enterprise deployment; TenantHub multi-stage rollout
- **Key Tools:** Azure Pipelines, YAML, Stages, Deployment Jobs, Environments, Approval Gates

---

## 🟢 Concept Core: Azure Pipelines Architecture

### GitHub Actions vs. Azure Pipelines: When to Choose Each

| Aspect | GitHub Actions | Azure Pipelines |
|--------|----------------|-----------------|
| **Best for** | Simple CI, GitHub-native projects | Complex multi-stage CD, Azure deployments |
| **Stages** | Limited (basic with environments) | ✅ Built for multi-stage pipelines |
| **Approval gates** | Manual (simple) | ✅ Advanced (gates, policies, branches) |
| **Artifact management** | GitHub Packages | ✅ Azure Artifacts (feeds, symbols) |
| **Source repos** | GitHub only | ✅ GitHub, Azure Repos, Bitbucket, any Git |
| **Cost** | Free tier: 2,000 min/month | Free tier: 1 job, unlimited minutes |
| **Enterprise features** | Limited | ✅ RBAC, audit, compliance |

**Rule of thumb:**
- **GitHub Actions:** Simple workflows, GitHub-native, speed-to-market
- **Azure Pipelines:** Complex multi-stage, enterprise governance, cost-conscious teams

---

### Azure Pipelines Structure

```
AZURE PIPELINES YAML
├── Trigger (when does it run?)
├── Variables (reusable config)
├── Pool (what runner?)
├── Stages (sequence of deployment phases)
│   ├── Stage 1: Build
│   │   └── Jobs
│   │       └── Steps
│   ├── Stage 2: Deploy to Staging (requires approval from Build)
│   ├── Stage 3: Tests in Staging
│   └── Stage 4: Deploy to Production (requires approval from Tests)
└── Resources (external repos, containers, packages)
```

### Key Concepts

| Concept | Meaning | Example |
|---------|---------|---------|
| **Pipeline** | YAML file in repo (`azure-pipelines.yml`) | Orchestrates CI/CD |
| **Trigger** | When pipeline runs | `on: [push, pull_request]` |
| **Stage** | Logical phase (Build, Test, Deploy) | `- stage: DeployToProduction` |
| **Job** | Unit of work within a stage | `- job: PublishArtifact` |
| **Deployment** | Special job type for environments | Deploy to App Service, Kubernetes |
| **Step** | Individual command or action | `script:`, `task:`, `checkout:` |
| **Pool** | Runner agent pool | `vmImage: 'ubuntu-latest'` |
| **Environment** | Deployment target + approval gate | `staging`, `production` |
| **Artifact** | Build output | Docker image, .NET DLL, tarball |
| **Service Connection** | Credentials for external services | Azure subscription, Docker Hub, GitHub |

---

## 🟢 Real-World Scenario 1: FastCart Multi-Stage Pipeline

**Feature:** One-click checkout
**Workflow:**

```
1. Developer pushes code to main
   ↓
2. STAGE: Build
   - Compile .NET code
   - Run unit tests
   - Build Docker image
   - Push to Azure Container Registry
   
3. STAGE: Deploy to Staging (Automatic)
   - Pull image from ACR
   - Deploy to Azure App Service (staging slot)
   - Run smoke tests
   
4. STAGE: Approval Gate
   - QA approves (manual) — "Staging looks good"
   
5. STAGE: Deploy to Production (Conditional)
   - If approved, pull image from ACR
   - Deploy to Azure App Service (production slot)
   - Blue-green swap (zero downtime)
   - Run health checks
   
6. SUCCESS → Notify team Slack
   OR
   FAILURE → Automatic rollback + alert
```

**Timeline:** 10 min (auto) + 5 min (QA review) + 2 min (deploy) = 17 min total

---

## 🟢 Real-World Scenario 2: TenantHub Scheduled Deploys

```
Daily 2 AM UTC Schedule:
├── Checkout main branch (latest code)
├── Run comprehensive test suite
├── Build artifact (version stamped: 2.1.0-2024-01-15)
├── Create "Stable" artifact (for 200 conservative tenants)
├── Create "Fast" artifact (for 4,800 early-adopter tenants)
├── Deploy "Stable" with approval gate (release manager only)
├── Deploy "Fast" (auto, feature flag gated)
├── Monitor for 10 min
└── If good: announce "All systems updated"
    OR
    If bad: automatic rollback to previous version
```

---

## 🔷 Hands-On Extension: Build Your First Azure Pipeline

### Part A: Set Up Azure DevOps Project

1. Go to [dev.azure.com](https://dev.azure.com)
2. Create new project (free tier available)
3. Set up Git repo
4. Create `azure-pipelines.yml` in repo root

---

### Part B: Write a Multi-Stage Pipeline

#### Tab: Python with Staging & Production

```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  pythonVersion: '3.11'
  registry: 'YOUR_REGISTRY.azurecr.io'  # Azure Container Registry
  repository: 'checkout-service'

stages:
  # STAGE 1: Build
  - stage: Build
    displayName: 'Build & Test'
    jobs:
      - job: BuildJob
        displayName: 'Build and Push Image'
        steps:
          - checkout: self
          
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '$(pythonVersion)'
            displayName: 'Use Python $(pythonVersion)'
          
          - script: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt
              pip install pytest pytest-cov
            displayName: 'Install dependencies'
          
          - script: |
              pytest --cov=src --cov-report=xml --junitxml=junit/test-results.xml
            displayName: 'Run tests'
          
          - task: PublishTestResults@2
            condition: succeededOrFailed()
            inputs:
              testResultsFiles: '**/test-*.xml'
              testRunTitle: 'Python Tests'
          
          - task: PublishCodeCoverageResults@1
            inputs:
              codeCoverageTool: Cobertura
              summaryFileLocation: '$(System.DefaultWorkingDirectory)/**/coverage.xml'
          
          - task: Docker@2
            inputs:
              command: buildAndPush
              repository: '$(registry)/$(repository)'
              dockerfile: 'Dockerfile'
              containerRegistry: 'AzureContainerRegistry'  # Service connection
              tags: |
                $(Build.BuildId)
                latest
            displayName: 'Build and push Docker image'
          
          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(Pipeline.Workspace)'
              artifact: 'PipelineArtifact'

  # STAGE 2: Deploy to Staging
  - stage: DeployToStaging
    displayName: 'Deploy to Staging'
    dependsOn: Build
    condition: succeeded()
    jobs:
      - deployment: DeployStaging
        displayName: 'Deploy to App Service (Staging)'
        environment: 'staging'  # Approval gate
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'AzureSubscription'
                    appName: 'checkout-service-staging'
                    package: '$(Agent.BuildDirectory)/**/*.zip'
                    deploymentMethod: 'zipDeploy'
                  displayName: 'Deploy to Azure App Service'
                
                - script: |
                    curl -X GET https://checkout-service-staging.azurewebsites.net/health
                  displayName: 'Run health check'

  # STAGE 3: Approval Gate (Manual)
  - stage: ApprovalForProduction
    displayName: 'Approval Gate'
    dependsOn: DeployToStaging
    condition: succeeded()
    jobs:
      - job: waitForValidation
        displayName: 'Wait for Release Manager Approval'
        pool: server
        timeoutInMinutes: 1440  # 24 hours
        steps:
          - task: ManualValidation@0
            inputs:
              notifyUsers: 'david@fastcart.com'  # Release manager
              instructions: 'Approve this deployment to production'

  # STAGE 4: Deploy to Production
  - stage: DeployToProduction
    displayName: 'Deploy to Production'
    dependsOn: ApprovalForProduction
    condition: succeeded()
    jobs:
      - deployment: DeployProduction
        displayName: 'Deploy to App Service (Production)'
        environment: 'production'
        strategy:
          runOnce:
            preDeploy:
              steps:
                - script: echo "Pre-deployment health check"
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'AzureSubscription'
                    appName: 'checkout-service-prod'
                    package: '$(Agent.BuildDirectory)/**/*.zip'
                    deploymentMethod: 'slotDeploy'  # Blue-green swap
                  displayName: 'Deploy to prod slot'
            postDeploy:
              steps:
                - script: |
                    # Run post-deploy validation
                    curl -X GET https://checkout-service.azurewebsites.net/health
                  displayName: 'Post-deployment health check'
```

**Key parts:**
- `stages:` — List of deployment phases
- `dependsOn:` — Enforces order (can't deploy to prod before staging succeeds)
- `condition: succeeded()` — Only run if previous stage passed
- `environment: 'staging'` — Creates approval gate
- `strategy: runOnce` — Deploy once (not rolling/canary)
- `slotDeploy:` — Blue-green swap (zero downtime)

---

#### Tab: .NET with Multi-Environment

```yaml
trigger:
  - main

pool:
  vmImage: 'windows-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '7.0.x'

stages:
  - stage: Build
    displayName: 'Build'
    jobs:
      - job: BuildJob
        steps:
          - checkout: self
          
          - task: UseDotNet@2
            inputs:
              packageType: 'sdk'
              version: '$(dotnetVersion)'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'restore'
              projects: '**/*.csproj'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'build'
              projects: '**/*.csproj'
              arguments: '--configuration $(buildConfiguration)'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'test'
              projects: '**/*Tests.csproj'
              arguments: '--configuration $(buildConfiguration) /p:CollectCoverage=true'
          
          - task: DotNetCoreCLI@2
            inputs:
              command: 'publish'
              publishWebProjects: true
              arguments: '--configuration $(buildConfiguration) --output $(Build.ArtifactStagingDirectory)'
          
          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: '$(Build.ArtifactStagingDirectory)'
              artifactName: 'drop'

  - stage: DeployDev
    displayName: 'Dev Environment'
    dependsOn: Build
    jobs:
      - deployment: DeployDev
        environment: 'dev'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure'
                    appName: 'checkout-dev-app'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'

  - stage: DeployStaging
    displayName: 'Staging Environment'
    dependsOn: DeployDev
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
                    appName: 'checkout-staging-app'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'

  - stage: DeployProduction
    displayName: 'Production Environment'
    dependsOn: DeployStaging
    jobs:
      - deployment: DeployProduction
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'Azure'
                    appName: 'checkout-prod-app'
                    package: '$(Pipeline.Workspace)/drop/**/*.zip'
                    deploymentMethod: 'slotDeploy'
                    slotName: 'staging'
```

---

### Part C: Set Up Service Connection to Azure

**Service Connection** = Credentials for Azure Pipeline to deploy to your Azure subscription

1. **In Azure DevOps:**
   - Go to **Project Settings** → **Service connections**
   - Click **New service connection**
   - Choose **Azure Resource Manager**
   - Select **Service Principal (Automatic)**
   - Select your Azure subscription
   - Name: `AzureSubscription`
   - Save

2. **In your pipeline:**
```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'AzureSubscription'  # Matches name above
    appName: 'my-app-service'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
```

**Security note:** Azure DevOps encrypts and safely stores the service principal credentials. Never hardcode credentials in YAML.

---

## 🔷 Lab: Build a Multi-Stage Azure Pipeline

### Objectives
- ✅ Create `azure-pipelines.yml`
- ✅ Set up Build stage with tests
- ✅ Deploy to staging environment
- ✅ Create approval gate
- ✅ Deploy to production (with manual approval)
- ✅ View pipeline run and approvals
- ✅ Approve and trigger production deploy

### Prerequisites
- Azure DevOps project
- Azure subscription (free tier works)
- Two App Service instances (staging & production)
- 90 minutes

### Lab Steps

#### Step 1: Create App Services

```bash
# Create resource group
az group create -n checkout-rg -l eastus

# Create App Service Plans
az appservice plan create -n checkout-staging-plan -g checkout-rg --sku B1 --is-linux
az appservice plan create -n checkout-prod-plan -g checkout-rg --sku B1 --is-linux

# Create Web Apps
az webapp create -n checkout-staging -g checkout-rg -p checkout-staging-plan --runtime "PYTHON|3.11"
az webapp create -n checkout-prod -g checkout-rg -p checkout-prod-plan --runtime "PYTHON|3.11"
```

#### Step 2: Create Service Connection

In Azure DevOps:
1. **Project Settings** → **Service connections** → **New**
2. Type: Azure Resource Manager
3. Select subscription
4. Name: `AzureSubscription`

#### Step 3: Create Pipeline YAML

**File: `azure-pipelines.yml`**

```yaml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: Build
    displayName: Build
    jobs:
      - job: BuildJob
        steps:
          - checkout: self
          
          - task: UsePythonVersion@0
            inputs:
              versionSpec: '3.11'
          
          - script: |
              pip install -r requirements.txt
              pytest
            displayName: 'Test'
          
          - script: |
              mkdir -p $(Build.ArtifactStagingDirectory)
              cp -r . $(Build.ArtifactStagingDirectory)/
            displayName: 'Prepare artifact'
          
          - task: PublishBuildArtifacts@1
            inputs:
              pathToPublish: $(Build.ArtifactStagingDirectory)
              artifactName: drop

  - stage: DeployStaging
    displayName: Deploy Staging
    dependsOn: Build
    jobs:
      - deployment: DeployStaging
        displayName: Deploy to Staging
        environment: 'staging'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'AzureSubscription'
                    appName: 'checkout-staging'
                    package: '$(Pipeline.Workspace)/drop'

  - stage: ProdApproval
    displayName: Production Approval
    dependsOn: DeployStaging
    jobs:
      - job: waitForValidation
        pool: server
        steps:
          - task: ManualValidation@0
            inputs:
              notifyUsers: 'you@example.com'
              instructions: 'Approve prod deployment'

  - stage: DeployProduction
    displayName: Deploy Production
    dependsOn: ProdApproval
    jobs:
      - deployment: DeployProduction
        displayName: Deploy to Production
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - task: AzureWebApp@1
                  inputs:
                    azureSubscription: 'AzureSubscription'
                    appName: 'checkout-prod'
                    package: '$(Pipeline.Workspace)/drop'
```

#### Step 4: Commit and Run

```bash
git add azure-pipelines.yml
git commit -m "Add multi-stage pipeline"
git push
```

1. Azure DevOps detects `azure-pipelines.yml`
2. Pipeline runs automatically
3. View in **Pipelines** tab

#### Step 5: Approve and Deploy to Production

1. In pipeline run, go to **Stages**
2. See "Production Approval" stage waiting
3. Click **Review** → **Approve**
4. Production deployment starts
5. Confirm on Azure: App Service updated

### Validation Checklist

- ✅ Pipeline file created
- ✅ Build stage runs tests
- ✅ Artifact published
- ✅ Deploy to staging succeeds
- ✅ Approval gate appears
- ✅ You can approve/reject
- ✅ Production deployment runs on approval
- ✅ App Service updated in Azure

---

## 🟢 Assessment: Azure Pipelines

> **Question 1:** What's the key difference between Azure Pipelines stages and GitHub Actions jobs?
> - A) Stages can't run in parallel; jobs can
> - B) Stages support approval gates and complex dependencies
> - C) They're identical
> - D) Stages only work in Azure
> 
> **Answer:** B — Azure Pipelines stages are designed for multi-stage CD with approval gates, dependencies, and conditions. GitHub Actions jobs are simpler (runs in parallel by default).

> **Question 2:** You need to deploy to Azure App Service without storing Azure credentials in your YAML. What should you use?
> - A) Hardcode credentials
> - B) Environment variables (in repo)
> - C) Service Connection (encrypts credentials in Azure DevOps)
> - D) Store in `.env` file
> 
> **Answer:** C — Service Connections securely store and inject credentials. Never hardcode or use repo environment variables for secrets.

---

## Next Steps

✅ **You now understand:**
- Azure Pipelines multi-stage structure
- How to write YAML pipelines with stages and jobs
- Approval gates and manual validations
- Service connections for secure deployment

🔜 **Next module:** CI — Build & Test (detailed patterns for both platforms)

---

## References

- [Azure Pipelines Documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines)
- [Service Connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/service-endpoints)
- [Deployment Jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs)
---

## 📝 Chapter Quiz

**Question 1:** What is the core objective of Azure Pipelines Essentials in modern DevOps practices?

* [ ] To increase manual work
* [x] To automate, streamline, and increase the reliability of software delivery
* [ ] To eliminate code repositories
* [ ] To replace all developers

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** To automate, streamline, and increase the reliability of software delivery

**Explanation:** Azure Pipelines Essentials focuses on improving agility, reducing delivery friction, and ensuring operational reliability.
</details>

**Question 2:** Which file format is standard for defining automated workflows and pipelines associated with Azure Pipelines Essentials?

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

**Question 5:** What is a major benefit of Infrastructure as Code (IaC) when configuring Azure Pipelines Essentials?

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

**Question 8:** When a production deployment fails, what is the fastest way to restore service stability when using Azure Pipelines Essentials?

* [ ] Re-write the application from scratch
* [x] Execute an automated rollback to the last known good artifact
* [ ] Ignore the alert
* [ ] Turn off monitoring

<details>
<summary>🔑 Click to Reveal Answer & Explanation</summary>

**Correct Answer:** Execute an automated rollback to the last known good artifact

**Explanation:** Automated rollbacks quickly restore operational stability by redeploying the previous verified release artifact.
</details>
