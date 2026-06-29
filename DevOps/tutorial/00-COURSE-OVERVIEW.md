# DevOps Mastery: GitHub + Azure DevOps — Complete Course

## Course At A Glance

This course teaches modern DevOps practices through **GitHub** and **Azure DevOps** — the tools driving enterprise CI/CD, release automation, and operational excellence. The course is **intentionally dual-track** so that business stakeholders can understand the *why* and *what*, while technical practitioners can master the *how*.

---

## Two-Track Model Explained

### **Track A — Non-Technical (Business/Leadership)**
- **Who:** Delivery managers, product owners, business analysts, pre-sales engineers, governance leads
- **Goal:** Understand DevOps workflows at a strategic and operational level; read pipelines; make or advise on governance decisions; speak confidently with technical teams
- **Key Outcome:** Able to interpret a pipeline log, explain release gates, assess risk, and understand cost implications
- **Code Requirement:** None. You read YAML/JSON examples in plain English; you never write or debug them
- **Format:** Concept explanations, real-world analogies, narrative walkthroughs, scenario-based assessment

### **Track B — Technical (Developers/Ops Engineers)**
- **Who:** Software engineers, DevOps engineers, platform engineers, infrastructure specialists
- **Goal:** Master hands-on pipeline design, YAML/JSON authoring, troubleshooting, security hardening, and full end-to-end automation
- **Key Outcome:** Can design, implement, and troubleshoot CI/CD pipelines; integrate GitHub and Azure DevOps; deploy to multi-stage production environments
- **Code Requirement:** Write and debug YAML, JSON, and scripts; complete hands-on labs and the capstone build
- **Format:** Concept core (shared with Track A) + hands-on labs, runnable code snippets, troubleshooting guides, capstone project

---

## Prerequisites

### **Track A Prerequisites**
- Familiarity with software delivery lifecycle (SDLC) concepts
- Basic understanding of testing, staging, and production environments
- No coding experience required

### **Track B Prerequisites**
- Intermediate software development experience (shipped at least one feature)
- Comfort with command-line tools and basic scripting (bash/PowerShell helpful but not required)
- Understanding of HTTP/REST APIs at a conceptual level
- Familiarity with Git basics (clone, push, pull, branch)

---

## Duration Estimate

| Track | Format | Weeks | Hours/Week | Total |
|-------|--------|-------|-----------|-------|
| **A** | Self-paced (concept + scenario assessment) | 4–5 | 5–7 | 20–35 hours |
| **B** | Self-paced (concept + hands-on labs + capstone) | 8–10 | 8–12 | 64–120 hours |
| **Both** | Instructor-led (condensed, joint sessions) | 3–5 | 10–15 | 30–75 hours |

*Note:* Track A can be completed independently and faster. Track B learners can attend all sessions; Track A learners skip lab-only sections.

---

## Learning Outcomes

### **Track A — Non-Technical Learning Outcomes**
By the end of this course, you will be able to:

1. **Explain** the purpose of CI/CD and why it reduces risk and accelerates delivery
2. **Interpret** a GitHub Actions workflow or Azure Pipelines YAML at a high level (triggers, stages, approvals)
3. **Identify** the key roles in a DevOps workflow (developer, reviewer, deployer, approver) and their responsibilities
4. **Evaluate** release gates and approval policies for governance, compliance, and risk
5. **Decide** when and how to use GitHub Environments vs. Azure DevOps Environments for multi-stage releases
6. **Assess** cost implications of CI/CD infrastructure and optimize for free-tier vs. paid tiers
7. **Discuss** security, secrets management, and compliance requirements with technical teams
8. **Follow** a real feature from code commit through staging to production using provided dashboards and logs

### **Track B — Technical Learning Outcomes**
By the end of this course, you will be able to:

1. **Implement** a fully functional GitHub Actions workflow for build, test, and deploy
2. **Design** an Azure DevOps multi-stage YAML pipeline with gates, approvals, and artifact management
3. **Configure** service connections, variable groups, and Azure credentials securely in both platforms
4. **Build** and version artifacts (Docker images, NuGet packages, binaries) with semantic versioning
5. **Manage** secrets, environment variables, and runtime context across GitHub and Azure environments
6. **Deploy** applications to multiple environments (dev, staging, prod) with proper approvals and validation
7. **Troubleshoot** failed builds, deployments, and pipeline hangs using logs, diagnostics, and common patterns
8. **Integrate** GitHub repositories with Azure DevOps Pipelines using personal access tokens and service connections
9. **Implement** security hardening: OIDC authentication, branch policies, required reviews, audit logging
10. **Architect** and execute a complete end-to-end DevOps delivery as a capstone project

---

## Stack & Technologies Covered

| Component | GitHub | Azure DevOps | Format |
|-----------|--------|--------------|--------|
| **Version Control** | ✅ Repos, branches, PRs | ✅ Git Repos, policy templates | YAML (workflow triggers) |
| **CI/CD Orchestration** | ✅ GitHub Actions | ✅ Pipelines (YAML/Classic) | YAML primary, JSON for config |
| **Artifact Storage** | ✅ Packages, Releases | ✅ Artifacts feeds, NuGet | JSON (REST API payloads) |
| **Environments & Gates** | ✅ Environments + manual approvals | ✅ Environments + deployment gates | YAML (environment names), JSON (gate logic) |
| **Secrets Management** | ✅ Repository secrets, org secrets | ✅ Variable groups, Key Vault | JSON (service connection payloads) |
| **Approvals & Governance** | ✅ Branch protection + manual gates | ✅ Multi-stage gates + RBAC | YAML (stage gates), JSON (policies) |

**Out of scope (unless flagged as optional):**
- Jenkins, GitLab CI, CircleCI, other platforms
- Kubernetes deployment details (reference only)
- Terraform, CloudFormation, IaC beyond pipeline-as-code
- Dedicated monitoring tools (reference Azure Monitor and GitHub Advanced Security)

---

## How to Use This Course

1. **Choose your track(s):**
   - Taking it as an **organization?** Both tracks run in parallel; attendees self-select.
   - Taking it solo? **Start with Track A** to understand the landscape, then deepen with Track B.

2. **Follow the module progression:**
   - Each module is **self-contained but sequential** (don't skip ahead).
   - Every section is **clearly labeled** [Both], [Non-technical], or [Technical].

3. **For Track A:**
   - Read the **Concept Core** section.
   - Study the **scenario walkthrough** (no lab required).
   - Complete the **scenario-based assessment**.

4. **For Track B:**
   - Read the **Concept Core** first.
   - Work through the **Hands-On Extension** and **lab exercises**.
   - Validate your work using the **troubleshooting guide**.
   - Build the **capstone project**.

5. **Assessment:**
   - **Track A:** Scenario questions, interpretation tasks.
   - **Track B:** Build/debug tasks, capstone rubric, real-world validation.

---

## Course Roadmap: 11 Modules + Capstone

### **Foundation Modules**
1. **[Both] DevOps Fundamentals** — What is DevOps? Why GitHub + Azure? The workflow and roles.
2. **[Both] Version Control Workflows** — Git, branches, PR reviews, branch policies. GitHub Actions trigger model intro.

### **Pipeline & Automation Modules**
3. **[Both] GitHub Actions Essentials** — Actions model, workflows, jobs, steps. Compare to Azure Pipelines.
4. **[Both] Azure Pipelines Essentials** — Stages, jobs, multi-repo support. Compare to GitHub Actions.
5. **[Both] CI Fundamentals: Build & Test** — Automated builds, test execution, fail-fast. Examples in both platforms.
6. **[Both] CD Fundamentals: Deployment** — Artifact promotion, deployment strategies (blue-green, canary, rolling). Both platforms.

### **Governance & Security Modules**
7. **[Both] Environments & Approval Gates** — Multi-stage deployments, manual gates, approvals, decision-makers.
8. **[Both] Secrets, Security & Compliance** — Secret management, OIDC, audit trails, compliance flags.

### **Advanced & Operational Modules**
9. **[Both] Artifacts & Versioning** — Semantic versioning, artifact feeds, package management, retention.
10. **[Both] Monitoring, Logging & Observability** — Pipeline health, release dashboards, troubleshooting metrics.
11. **[Both] Scaling & Real-World Patterns** — Monorepo vs. polyrepo, dependency management, cost optimization, migrations.

### **Capstone**
12. **[Both + Technical] Capstone: End-to-End DevOps Delivery** — Real scenario walkthrough. Track A narrative, Track B build.

---

## Module Overview Table

| # | Module Title | Concept Core | Hands-On Labs | Track A Est. Time | Track B Est. Time |
|---|--------------|--------------|----------------|-------------------|-------------------|
| 1 | DevOps Fundamentals | 45 min | 0 min | 45 min | 45 min + reading |
| 2 | Version Control Workflows | 60 min | Lab: PR & merge | 60 min | 60 min + 60 min lab |
| 3 | GitHub Actions Essentials | 60 min | Lab: Build workflow | 60 min | 60 min + 90 min lab |
| 4 | Azure Pipelines Essentials | 60 min | Lab: Multi-stage pipeline | 60 min | 60 min + 90 min lab |
| 5 | CI: Build & Test | 75 min | Lab: CI setup (both) | 75 min | 75 min + 120 min lab |
| 6 | CD: Deployment | 75 min | Lab: Artifact promotion | 75 min | 75 min + 120 min lab |
| 7 | Environments & Approvals | 75 min | Lab: Approval gates | 75 min | 75 min + 120 min lab |
| 8 | Secrets & Security | 90 min | Lab: Secret rotation | 90 min | 90 min + 120 min lab |
| 9 | Artifacts & Versioning | 60 min | Lab: Versioning strategy | 60 min | 60 min + 90 min lab |
| 10 | Monitoring & Observability | 60 min | Lab: Dashboard setup | 60 min | 60 min + 90 min lab |
| 11 | Scaling & Real-World Patterns | 75 min | Optional: Cost analysis | 75 min | 75 min + Optional labs |
| 12 | **Capstone** | 90 min narrative | Full build + test | 90 min | 90 min + 4–6 hours build |
| | **TOTAL** | | | **20–25 hours** | **60–80 hours** |

---

## What's Next?

I will now expand each module with:
- **Concept Core [Both]:** Plain-language explanation, analogies, diagrams (text descriptions), roles, and business impact
- **Hands-On Extension [Technical]:** Runnable YAML, JSON, practical labs, and troubleshooting
- **Labs [Technical]:** Step-by-step guided exercises with validation checklists
- **Capstone [Both + Technical]:** Scenario narrative and the full implementation

---

## Questions Before I Proceed?

Please confirm:
1. ✅ Is this module structure and learning outcome direction aligned with your vision?
2. ✅ Should I include any additional modules (e.g., advanced security, cost analysis, migration patterns)?
3. ✅ Do you want me to expand all 12 modules in full detail, or start with the first 3–4 for feedback?
4. ✅ Any specific real-world scenarios or company archetypes you'd like the capstone to reflect (e.g., microservices, monolith, SaaS, enterprise)?

Awaiting your confirmation to proceed! 🚀
