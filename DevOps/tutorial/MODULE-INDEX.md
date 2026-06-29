# Module Index & Progression Map

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/vinod-seth/course-repo/blob/main/tutorial/MODULE-INDEX.md)


## Quick Navigation

```
COURSE STRUCTURE
├── 📚 Foundations (Modules 1–2)
│   ├── Module 1: DevOps Fundamentals [Both]
│   └── Module 2: Version Control Workflows [Both]
│
├── ⚙️  CI/CD Essentials (Modules 3–6)
│   ├── Module 3: GitHub Actions Essentials [Both]
│   ├── Module 4: Azure Pipelines Essentials [Both]
│   ├── Module 5: CI: Build & Test [Both]
│   └── Module 6: CD: Deployment [Both]
│
├── 🔒 Governance & Security (Modules 7–9)
│   ├── Module 7: Environments & Approval Gates [Both]
│   ├── Module 8: Secrets, Security & Compliance [Both]
│   └── Module 9: Artifacts & Versioning [Both]
│
├── 📊 Operations & Patterns (Modules 10–11)
│   ├── Module 10: Monitoring, Logging & Observability [Both]
│   └── Module 11: Scaling & Real-World Patterns [Both]
│
└── 🎯 Capstone (Module 12)
    └── Module 12: End-to-End DevOps Delivery [Both + Technical]
```

---

## Module Details at a Glance

### **Module 1: DevOps Fundamentals** [Both]
- **Concept:** What is DevOps? CI, CD, automation, collaboration
- **Why GitHub + Azure DevOps?** Ecosystem fit, GitHub as repo + Actions, Azure DevOps for enterprise
- **The workflow:** Commit → Test → Build → Stage → Approve → Deploy
- **Roles:** Developer, reviewer, ops, approver, stakeholder
- **Business impact:** Speed, quality, risk reduction, cost
- **Track A Time:** 45 min | **Track B Time:** 45 min (+ setup context)

### **Module 2: Version Control Workflows** [Both]
- **Concept:** Git, branches, pull requests, code review, branch policies
- **GitHub native:** Branch protection rules, PR templates, CODEOWNERS
- **Azure Repos:** Branch policies, PR reviewers, merge strategies
- **Triggers into CI/CD:** Webhook model, what kicks off a pipeline
- **Business decision:** How strict should code review be? What's the approval policy?
- **Track A Time:** 60 min | **Track B Time:** 60 min + Lab (60 min)

### **Module 3: GitHub Actions Essentials** [Both]
- **Concept:** Actions model (workflows, jobs, steps, runners)
- **Trigger events:** push, pull_request, schedule, manual (workflow_dispatch)
- **GitHub-hosted runners:** ubuntu-latest, windows-latest, macos-latest (cost & time trade-off)
- **Self-hosted runners:** on-premises option (enterprise scenario)
- **Actions Marketplace:** Community actions, verified partners, risk
- **Secrets & contexts:** How GitHub injects secure data
- **GitHub vs. Azure Pipelines:** Architecture differences explained plainly
- **Track A Time:** 60 min | **Track B Time:** 60 min + Lab (90 min)

### **Module 4: Azure Pipelines Essentials** [Both]
- **Concept:** Stages, jobs, tasks; agent pools; system vs. hosted agents
- **YAML vs. Classic UI:** Why YAML wins (versioned, auditable, repeatable)
- **Multi-repo support:** Trigger pipelines across GitHub, Azure Repos, Bitbucket, generic Git
- **Service connections:** How to authenticate Azure, Docker, npm, custom services (JSON payloads shown)
- **Variable groups & secure files:** Central secret/config management
- **Azure Pipelines vs. GitHub Actions:** Feature parity, when to choose each
- **Track A Time:** 60 min | **Track B Time:** 60 min + Lab (90 min)

### **Module 5: CI — Build & Test** [Both]
- **Concept:** Automated build (compile, package), test execution (unit, integration, code quality), fail-fast
- **GitHub example:** Node.js app → npm install → npm test → npm run build
- **Azure example:** .NET app → dotnet build → dotnet test → publish artifacts
- **Quality gates:** Test coverage % thresholds, linting, SAST scanning
- **Cache strategy:** Speeding up builds with dependency caching (Track B)
- **Parallel jobs:** Running tests across multiple agents (saves time)
- **Business view:** Why failing fast matters (feedback to developer in minutes, not hours)
- **Track A Time:** 75 min | **Track B Time:** 75 min + Lab (120 min)

### **Module 6: CD — Deployment** [Both]
- **Concept:** Artifact promotion (build once, deploy many), deployment strategies
- **Blue-green, canary, rolling:** When to use each; trade-offs in complexity vs. downtime
- **GitHub example:** Release workflow → artifact to GH Packages → deploy to staging/prod
- **Azure example:** Multi-stage pipeline (build → deploy to dev → deploy to staging → deploy to prod)
- **Artifact versioning:** Semantic versioning, build number tagging, rollback ability
- **Health checks & validation:** Smoke tests post-deploy
- **Business decision:** Downtime tolerance? Rollback SLA? Blue-green costs extra infrastructure.
- **Track A Time:** 75 min | **Track B Time:** 75 min + Lab (120 min)

### **Module 7: Environments & Approval Gates** [Both]
- **Concept:** Multi-stage deployments, manual gates, human decision points
- **GitHub Environments:** Environment secrets, protection rules (approval, timer)
- **Azure Environments:** Deployment gates (manual, deployment gate functions, service checks)
- **Approval workflows:** Who approves? Sequential vs. parallel approvals?
- **Decision governance:** Business decision: Is every prod deploy manual or only high-risk features?
- **Cost of gates:** Slow releases? Or essential for compliance?
- **Track A Time:** 75 min | **Track B Time:** 75 min + Lab (120 min)

### **Module 8: Secrets, Security & Compliance** [Both]
- **Concept:** Secret rotation, OIDC vs. PAT, audit trails, compliance requirements
- **GitHub:** Repository secrets, org secrets, GitHub Secret Scanning (free)
- **Azure:** Key Vault integration, variable group secrets, user-managed identities
- **OIDC / Workload Identity:** Passwordless GitHub → Azure authentication (preferred over PATs)
- **Audit logging:** Who deployed what, when, and why (compliance requirement)
- **Policy enforcement:** Branch policies, required status checks, enforced PR reviews
- **Business decision:** Compliance requirement? SoX, HIPAA? How strict is secret rotation?
- **Track A Time:** 90 min | **Track B Time:** 90 min + Lab (120 min)

### **Module 9: Artifacts & Versioning** [Both]
- **Concept:** Build once, version it, promote it through environments, retain / delete old builds
- **Versioning strategies:** Semantic versioning (1.2.3), build numbers, commit SHAs
- **GitHub Packages:** Docker, npm, NuGet, Maven container registry
- **Azure Artifacts:** NuGet feeds, npm registry, Maven, universal packages
- **Feed promotion:** Dev feed → staging feed → production feed (immutable)
- **Retention policies:** How long to keep old artifacts? (storage cost)
- **Rollback:** Can we deploy version 1.2.1 again if 1.2.2 fails?
- **Track A Time:** 60 min | **Track B Time:** 60 min + Lab (90 min)

### **Module 10: Monitoring, Logging & Observability** [Both]
- **Concept:** Pipeline health dashboards, deployment frequency, failure rate, MTTR
- **GitHub:** Actions dashboard, logs, artifact cleanup
- **Azure:** Pipeline analytics, deployment frequency reports, failure trends
- **Business metrics:** DORA metrics (deployment freq., lead time, change failure rate, MTTR)
- **Post-deployment monitoring:** Health checks, alerting on prod errors
- **Troubleshooting:** Reading logs, identifying common failure patterns
- **Track A Time:** 60 min | **Track B Time:** 60 min + Lab (90 min)

### **Module 11: Scaling & Real-World Patterns** [Both]
- **Concept:** Monorepo vs. polyrepo, dependency management, cost optimization, platform migrations
- **Monorepo pattern:** Single repo, multiple services, coordinated pipeline
- **Polyrepo pattern:** Service owns its repo, orchestration complexity
- **Dependency management:** Breaking changes, versioning across services
- **Cost optimization:** GitHub Actions runners (free tier), Azure Pipeline concurrency (cost per agent)
- **Platform migrations:** GitHub to Azure DevOps (or vice versa) without breaking CI/CD
- **Enterprise at scale:** Multiple teams, shared pipelines, governance at scale
- **Track A Time:** 75 min | **Track B Time:** 75 min + Optional deep-dives

### **Module 12: Capstone — End-to-End DevOps Delivery** [Both + Technical]

**[Both] Scenario Walkthrough (90 min):**
- Company: E-commerce SaaS platform
- Feature: "One-click checkout" for mobile users
- Journey: Developer commit → code review → build → staging validation → approval → blue-green production deploy
- Roles: Developer (Alice), Code reviewer (Bob), QA validator (Carol), Release approver (David), Customer (Eve experiences faster checkout)
- Dashboards & logs: Screenshots/descriptions of GitHub + Azure status, approval flows, post-deploy health checks
- Narrative: "Here's what you'd see and what each stakeholder does"
- **Non-technical learners:** Can read dashboards, understand decision points, explain the flow to peers

**[Technical] Full Implementation Build (4–6 hours):**
- **Scenario:** Same "checkout" feature, real Azure subscription or free tier sandbox
- **Deliverables:**
  1. GitHub repo with feature branch → main merge policy
  2. GitHub Actions workflow for build + test
  3. Azure DevOps pipeline (YAML): multi-stage (build → deploy-dev → deploy-staging → deploy-prod)
  4. Service connection (GitHub → Azure), secrets, OIDC setup
  5. Approval gates on staging & prod
  6. Artifact versioning (1.0.0-feature-abc, 1.0.0-rc.1, 1.0.0)
  7. Blue-green deployment script (pseudo-Kubernetes or app service swap)
  8. Post-deploy health check, rollback procedure
  9. Audit trail validation (who, what, when)
  10. Cost breakdown (free tier vs. paid)
- **Rubric:** Code quality, pipeline reliability, security hardening, documentation

---

## Prerequisites Checklist

### **Track A**
- ✅ Understand basic software delivery lifecycle
- ✅ Know what staging and production environments are
- ✅ No coding required

### **Track B**
- ✅ Shipped at least one software feature (ideally in a team)
- ✅ Comfortable with command line (bash/PowerShell)
- ✅ Understand Git basics (clone, push, pull, branch)
- ✅ Familiar with HTTP/REST (conceptually)
- ✅ *Optional:* Docker basics (will be explained, not required)

---

## Recommended Learning Sequence

### **If taking both tracks (8–10 weeks):**
1. **Week 1:** Module 1–2 (Fundamentals, VCS) — both tracks together
2. **Week 2:** Module 3–4 (Actions, Pipelines) — both tracks together, then Track B labs
3. **Week 3:** Module 5–6 (CI, CD) — both tracks together, then Track B labs
4. **Week 4:** Module 7–8 (Environments, Security) — both tracks together, then Track B labs
5. **Weeks 5–6:** Module 9–11 (Artifacts, Monitoring, Patterns) — both tracks, Track B optional deep-dives
6. **Weeks 7–8:** Capstone walkthrough [Both] + Build [Technical]

### **If taking only Track A (4–5 weeks):**
1. **Week 1:** Module 1–2
2. **Week 2:** Module 3–4
3. **Week 3:** Module 5–6
4. **Week 4:** Module 7–11 (condensed, focus on governance)
5. **Week 5:** Capstone walkthrough

### **If taking only Track B (8–10 weeks):**
- Follow same sequence but spend more time on hands-on labs and troubleshooting
- Capstone is required (full build)

---

## Assessment & Certifications

### **Track A Completion Criteria**
- ✅ Score 75%+ on scenario-based assessment (interpret workflows, make governance decisions)
- ✅ Attend capstone walkthrough
- ✅ **Certificate:** "DevOps Stakeholder" (shows you understand the landscape)

### **Track B Completion Criteria**
- ✅ Complete all hands-on labs with passing validation
- ✅ Build & deploy the capstone project successfully (working pipeline, approval gates, versioning)
- ✅ Troubleshooting exercise: Diagnose 2–3 pipeline failures and propose fixes
- ✅ **Certificate:** "DevOps Engineer" (practical CI/CD competency)

---

## Next Steps

**Awaiting your confirmation on:**
1. Module structure — any additions or changes?
2. Which modules to expand first (suggest 1–3)?
3. Real-world scenario for capstone (e.g., microservices, monolith, specific industry)?
4. Any constraints (e.g., no Kubernetes, free tier only)?

Once confirmed, I'll proceed with full module content! 🚀