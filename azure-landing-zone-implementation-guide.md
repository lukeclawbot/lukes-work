# Azure Landing Zone Implementation Guide for a New App-Developer Subscription

## Purpose

This guide turns the landing zone strategy into a practical implementation plan for deploying a **new Azure subscription** into an existing tenant so an application developer can begin building with fewer blockers and fewer avoidable mistakes.

The goal is not simply to create a subscription.
The goal is to create a **governed, supportable, developer-ready landing zone** aligned to the **Azure Well-Architected Framework**.

This guide assumes you want a setup that is:

- safe by default
- repeatable through infrastructure as code
- friendly to GitHub-based CI/CD
- practical for real-world app development
- structured so platform governance and developer autonomy both exist without trying to murder each other

---

# 1. Desired Outcome

By the end of this implementation, you should have:

- a new Azure subscription placed in the correct management group
- baseline governance applied through policy and RBAC
- observability and security enabled from day one
- a cost-management baseline in place
- a network model appropriate to the app team’s needs
- a developer access model that supports delivery without handing over total subscription control
- a paved path for application deployment using GitHub Actions and OIDC
- a structure that can be reused for future subscriptions

---

# 2. Guiding Principles

Before implementation, agree on a few principles.

## Principle 1: Platform controls the foundation
The platform team or subscription owner should control:

- management group placement
- policy assignments
- baseline RBAC
- network standards
- monitoring standards
- cost governance
- CI/CD identity guardrails

## Principle 2: Developers get freedom inside approved boundaries
The app team should get enough access to:

- deploy application resources
- manage app configuration
- troubleshoot within their scope
- iterate quickly through GitHub-driven workflows

They should not need unrestricted control over:

- subscription-wide governance
- policy definitions
- core platform networking
- security baselines

## Principle 3: Repeatability beats heroics
If a step matters, it should be:

- documented
- scriptable
- reviewable
- reproducible

## Principle 4: Start with a practical baseline, then harden
Do not build a giant bureaucratic wall before the first workload lands.
Also do not launch a “temporary” loose environment that becomes permanent.

The sweet spot is a solid baseline with room to tighten safely as usage becomes clearer.

---

# 3. Roles and Responsibilities

## Platform / Cloud Owner responsibilities
Own these areas:

- subscription creation
- management group placement
- baseline policy assignment
- baseline RBAC assignments
- network design standards
- monitoring and Defender baseline
- cost governance baseline
- GitHub OIDC federation model
- reference architecture and deployment templates

## Application Developer / App Team responsibilities
Own these areas within approved scope:

- application-specific infrastructure
- application code and CI/CD config within approved patterns
- app-level monitoring and alert tuning
- environment-specific app configuration
- working within naming, tagging, and security expectations

## Shared responsibilities
These typically need collaboration:

- selecting service patterns
- deciding network exposure model
- production-readiness review
- well-architected reviews over time
- incident response playbooks

---

# 4. Prerequisites

Before building anything, confirm these prerequisites.

## Organizational prerequisites
You should know:

- who owns the Azure tenant/platform
- who will own the new subscription
- whether this is dev-only, dev/test, or expected to support production later
- whether network connectivity to on-prem or other VNets is required
- whether the workload is internal, external, regulated, or sensitive
- who approves access and budget

## Technical prerequisites
You should have:

- permissions to create or request a new subscription
- permissions to move the subscription into a management group
- permissions to assign policy and RBAC at the needed scope
- a decision on whether Terraform, Bicep, or both will be used
- a GitHub org/repo strategy for deployment automation
- an Entra ID group management strategy
- a target logging/monitoring destination strategy

## Architectural prerequisites
Decide early:

- landing management group placement
- allowed Azure regions
- tag standards
- network approach
- app hosting pattern preferences
- identity and secret management expectations
- whether environments will share one subscription or be split later

---

# 5. Recommended Implementation Phases

This rollout is best done in five phases:

1. Governance design
2. Subscription provisioning and baseline controls
3. Shared platform services and networking
4. Developer enablement and deployment path
5. Validation, hardening, and iteration

---

# Phase 1: Governance Design

This phase prevents the project from becoming “we already created it, now what should the rules be?”

## Step 1: Define the subscription purpose and classification

### What to do
Document:

- subscription name
- workload purpose
- business owner
- technical owner
- expected environments hosted there
- sensitivity/compliance level
- whether it is temporary, permanent, internal, or internet-facing

Example:

- Name: `sub-appdev-online-dev-001`
- Purpose: application development and lower environment hosting
- Owner: App Platform Team
- Technical owner: Cloud Engineering
- Environments: dev and test
- Exposure: internet-facing app services possible

### Why
If you do not define the purpose early, governance becomes vague and inconsistent.

This also informs:

- management group placement
- policy strictness
- network requirements
- RBAC design
- cost expectations

### WAF alignment
- **Operational Excellence**
- **Security**
- **Cost Optimization**

---

## Step 2: Select management group placement

### What to do
Choose the landing management group.

Example options:

- `LandingZones/AppDev`
- `LandingZones/Online`
- `LandingZones/Corp`

### Why
Management group placement determines inherited governance and often reflects intended workload behavior.

This is one of the highest-leverage decisions because it affects:

- policy inheritance
- compliance inheritance
- budget and cost reporting structures
- platform operating model

### WAF alignment
- **Operational Excellence**
- **Security**

---

## Step 3: Define naming and tagging standards

### What to do
Define required tags and name patterns before deployment.

Recommended required tags:

- `Environment`
- `Application`
- `Owner`
- `CostCenter`
- `BusinessUnit`
- `Criticality`
- `DataClassification`

Document naming rules for:

- subscription
- resource groups
- storage accounts
- key vaults
- app services
- managed identities

### Why
Naming and tagging are boring until you try to operate without them.

They support:

- cost visibility
- searchability
- ownership tracing
- automation filtering
- incident response clarity

### WAF alignment
- **Operational Excellence**
- **Cost Optimization**

---

## Step 4: Define the RBAC model

### What to do
Create Entra groups for role-based access.

Recommended baseline groups:

- `az-sub-appdev-platform-admins`
- `az-sub-appdev-contributors`
- `az-sub-appdev-readers`
- `az-sub-appdev-security-readers`
- `az-sub-appdev-cost-readers`
- `az-rg-app1-dev-contributors` for app-scoped access if needed

Define which roles each group gets and at what scope.

Recommended starting pattern:

- Platform admins: elevated admin role at subscription scope
- Developers: Contributor at resource group scope where possible
- Readers/security readers: Reader or Security Reader as appropriate
- Cost readers: Cost Management Reader or equivalent reporting visibility

### Why
An access model that is not defined up front will be defined by urgency later.
Urgency produces terrible architecture.

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 5: Define baseline policy controls

### What to do
Decide which policies are:

- required immediately
- audit-only initially
- future hardening candidates

Recommended day-one policies:

- allowed locations
- required tags
- deny public storage access unless approved
- require secure transfer for storage
- deny disallowed resource types if relevant
- require diagnostic settings where feasible
- restrict public IP creation unless explicitly allowed
- Defender for Cloud baseline requirements

### Why
If you wait to define guardrails until after deployments begin, the first app becomes the standard by accident.

### WAF alignment
- **Security**
- **Operational Excellence**
- **Cost Optimization**

---

# Phase 2: Subscription Provisioning and Baseline Controls

## Step 6: Create the Azure subscription

### What to do
Create the new subscription through your approved enterprise process.
This may be via:

- Azure portal enterprise billing / MCA flow
- platform automation
- subscription vending pipeline

Capture:

- subscription ID
- subscription name
- billing scope
- owner assignment

### Why
This is the formal creation step, but it should happen only after governance decisions are made.

Creating subscriptions before deciding placement and controls is how environments begin life as orphans.

### WAF alignment
- **Operational Excellence**

---

## Step 7: Move the subscription into the correct management group

### What to do
Immediately associate the new subscription with the correct management group.

Validate that inherited policies and assignments are as expected.

### Why
If this is skipped or delayed, the subscription can run outside intended governance boundaries.

That creates risk around:

- policy enforcement
- visibility
- cost control
- compliance posture

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 8: Assign baseline RBAC groups

### What to do
Assign the agreed Entra groups at the proper scopes.

Initial example:

- `az-sub-appdev-platform-admins` → subscription admin role
- `az-sub-appdev-readers` → Reader on subscription
- `az-sub-appdev-security-readers` → Security Reader on subscription
- app developer groups → resource group scope later, not necessarily full subscription

### Why
This makes ownership explicit and avoids the classic startup phase where one or two humans have unexplained god-mode access because it was “faster.”

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 9: Apply baseline policy assignments

### What to do
Apply the initial policy initiative(s) or assignments at management group or subscription scope.

Recommended approach:

- inherit broad enterprise policy from management group
- add subscription-specific policy only where justified
- document all explicit exceptions

### Why
You want policy to be understandable and supportable.
Too many ad hoc assignments become impossible to reason about.

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 10: Enable cost governance baseline

### What to do
Create:

- subscription budget
- alert thresholds, such as 50%, 75%, 90%, and 100%
- cost views by tag and resource group
- ownership for budget alerts

### Why
The best time to create cost visibility is before anyone can spend money creatively.

### WAF alignment
- **Cost Optimization**
- **Operational Excellence**

---

# Phase 3: Shared Platform Services and Networking

## Step 11: Create the initial resource group structure

### What to do
Create baseline resource groups.

Example:

- `rg-appdev-shared-dev`
- `rg-appdev-monitoring`
- `rg-appdev-networking`
- `rg-app1-dev`
- `rg-app1-test`

Adjust based on whether the subscription hosts one app or multiple teams.

### Why
This gives you clean scoping for:

- access
- cost reporting
- lifecycle management
- monitoring
- platform shared services

### WAF alignment
- **Operational Excellence**
- **Cost Optimization**
- **Security**

---

## Step 12: Establish the monitoring baseline

### What to do
Implement the monitoring model.

This usually includes:

- Log Analytics workspace decision
- diagnostic settings routing
- Activity Log retention/export
- baseline Azure Monitor alerts
- Application Insights pattern for apps

Decide whether the subscription uses:

- centralized shared workspace
- dedicated workspace
- hybrid model

### Why
Monitoring is part of the platform, not an optional add-on after the first incident.

### WAF alignment
- **Reliability**
- **Operational Excellence**
- **Security**

---

## Step 13: Enable Defender for Cloud and baseline security visibility

### What to do
Enable the appropriate Defender plans and ensure the subscription is visible in security monitoring processes.

Decide:

- which plans are required
- who reviews recommendations
- how remediation is tracked

### Why
Security posture without visibility is just optimism wearing a badge.

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 14: Implement the network foundation

### What to do
Choose one of the following patterns and build accordingly.

### Pattern A: Lightweight app-dev subscription
Use when:

- app team is mostly PaaS-first
- minimal hybrid dependency exists
- lower environments need speed with guardrails

Build:

- only required VNets/subnets
- private endpoints where justified
- NSG patterns if IaaS exists
- DNS integration plan
- approved ingress pattern

### Pattern B: Enterprise-integrated application subscription
Use when:

- shared connectivity services already exist
- hybrid/private access matters
- this may mature toward production

Build:

- hub-spoke or vWAN-aligned network placement
- DNS integration with platform standards
- route and inspection alignment
- app team consumption model for networking

### Why
Network improvisation scales badly and refactoring it later is the cloud equivalent of moving plumbing after the tile is down.

### WAF alignment
- **Security**
- **Reliability**
- **Performance Efficiency**

---

## Step 15: Define secret and identity handling patterns

### What to do
Standardize:

- Key Vault usage
- managed identity expectations
- where secrets may and may not live
- rotation ownership
- how CI/CD accesses cloud resources

Recommended default:

- managed identities for workloads where possible
- Key Vault for secrets
- no secrets embedded in code or pipeline YAML

### Why
If secret handling is not designed, developers will invent one.
And they are often very creative in the worst possible direction.

### WAF alignment
- **Security**
- **Operational Excellence**

---

# Phase 4: Developer Enablement and Deployment Path

## Step 16: Create developer-facing access scopes

### What to do
Assign app teams access primarily at resource group scope unless there is a justified reason to go broader.

Examples:

- `az-rg-app1-dev-contributors` → Contributor on `rg-app1-dev`
- `az-rg-app1-test-contributors` → Contributor on `rg-app1-test`
- app team reader groups for shared resources as needed

### Why
This gives the team room to build without letting every app deployment become an accidental platform engineering exercise.

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 17: Publish a starter repository template

### What to do
Create a standard GitHub template repository that includes:

- folder structure
- Terraform and/or Bicep starter layout
- README
- tagging/naming guidance
- pre-commit config
- sample GitHub Actions workflows
- monitoring and diagnostics expectations

Suggested layout:

```text
repo/
  .github/workflows/
  modules/
  environments/
    dev/
    test/
    prod/
  scripts/
  docs/
```

### Why
A starter repo is how you turn “please follow standards” into something humans might actually do.

### WAF alignment
- **Operational Excellence**
- **Reliability**
- **Security**

---

## Step 18: Implement GitHub Actions with OIDC

### What to do
For each app or deployment repo:

- create federated identity in Entra/Azure
- scope it to the right subscription/resource group
- map GitHub repo/environment claims appropriately
- create workflow environments for approval if needed
- validate that plan/deploy jobs authenticate without client secrets

Typical workflow stages:

1. format/lint
2. validate
3. plan
4. manual approval for protected envs
5. deploy/apply

### Why
This creates a secure, reviewable, repeatable deployment path and keeps long-lived secrets out of the pipeline wherever possible.

### WAF alignment
- **Security**
- **Operational Excellence**
- **Reliability**

---

## Step 19: Document the approved service patterns

### What to do
Write a short design guide covering:

- preferred compute patterns
- preferred database patterns
- network exposure rules
- logging expectations
- secret storage patterns
- identity patterns
- cost-sensitive service choices

Example recommendations:

- prefer PaaS over IaaS where possible
- use managed identity before storing service credentials
- use App Insights and diagnostics by default
- avoid public exposure unless explicitly designed and reviewed

### Why
Teams move faster when the paved road is visible.
Without it, each project burns time re-deciding solved problems.

### WAF alignment
- **Performance Efficiency**
- **Security**
- **Operational Excellence**
- **Cost Optimization**

---

## Step 20: Deploy a first pilot workload

### What to do
Choose a small but realistic first application deployment.
Use it to validate:

- RBAC
- policy behavior
- CI/CD authentication
- naming/tagging rules
- diagnostics
- app monitoring
- network connectivity
- cost visibility

### Why
The first workload is the best reality check.
It will show where your landing zone is:

- too loose
- too strict
- too manual
- missing critical documentation

### WAF alignment
- **Operational Excellence**
- **Reliability**
- **Security**

---

# Phase 5: Validation, Hardening, and Iteration

## Step 21: Run a post-deployment review after the first workload

### What to do
Review:

- what policies blocked good work unnecessarily
- what risky behavior was still possible
- what RBAC assignments were too broad or too narrow
- whether diagnostics and alerts were sufficient
- whether costs were attributable and understandable
- whether CI/CD identity scoping was correct

### Why
No first version is perfect.
A review turns the first deployment into a learning loop instead of a folk tale.

### WAF alignment
- **Operational Excellence**
- **Security**
- **Cost Optimization**

---

## Step 22: Tighten controls where justified

### What to do
After learning from real usage, consider tightening:

- audit policies to deny
- service allowlists
- network restrictions
- production approval workflows
- more granular RBAC
- mandatory diagnostics coverage

### Why
It is usually smarter to start with a solid but workable baseline and then harden intentionally than to build a perfect prison that nobody can use.

### WAF alignment
- **Security**
- **Operational Excellence**

---

## Step 23: Create runbooks and ownership notes

### What to do
Document:

- how access is requested
- how budget alerts are handled
- how incidents are triaged
- who owns policy exceptions
- how new apps are onboarded
- how production promotion works

### Why
A landing zone becomes operationally real only when people know how to live in it.

### WAF alignment
- **Operational Excellence**
- **Reliability**

---

## Step 24: Reuse the pattern for the next subscription

### What to do
Turn the final result into a reusable vending model or baseline module set.

That may include:

- Terraform modules
- Bicep modules
- policy initiative definitions
- RBAC assignment code
- GitHub repo templates
- workflow templates

### Why
The second and third subscriptions should get easier, not more artisanal.

### WAF alignment
- **Operational Excellence**
- **Reliability**
- **Security**

---

# 6. Suggested Technical Baseline

Below is an opinionated technical baseline for a new app-dev subscription.

## Governance baseline
- correct management group placement
- tag standard enforced
- location restrictions
- policy for public exposure control
- Defender for Cloud enabled
- budget alerts configured

## RBAC baseline
- platform admin group at subscription level
- security reader group at subscription level
- app developers scoped mostly to resource groups
- cost/reporting visibility assigned intentionally

## Monitoring baseline
- Log Analytics pattern defined
- activity logs retained/exported
- diagnostic settings enabled where supported
- Application Insights used for apps
- baseline alerting for critical failures and cost

## Networking baseline
- approved ingress pattern
- DNS strategy defined
- private endpoints when justified
- no unmanaged peering by app teams
- route governance where enterprise networking exists

## CI/CD baseline
- GitHub repo templates
- GitHub Actions with OIDC
- plan-before-apply flow
- protected production environments
- no long-lived secrets unless unavoidable

## App architecture baseline
- PaaS-first bias where practical
- managed identities preferred
- Key Vault for secrets
- documented service standards
- environment separation from day one

---

# 7. Example Delivery Timeline

Here is a realistic rollout sequence.

## Week 1: Design and governance
- confirm purpose and owners
- define management group placement
- define naming/tags/RBAC/policy baseline
- choose network pattern

## Week 2: Subscription and baseline deployment
- create subscription
- place into management group
- assign RBAC
- apply policy
- configure budget and monitoring baseline

## Week 3: Developer enablement
- create repo template
- configure GitHub OIDC
- assign app team access
- document approved patterns

## Week 4: Pilot deployment and hardening
- deploy first sample workload
- review findings
- tighten controls
- document runbooks and exceptions

That timeline can compress or expand, but the order matters more than the exact duration.

---

# 8. Common Failure Modes to Avoid

## Failure Mode 1: Subscription first, governance later
Why it fails:
- controls arrive after bad habits
- cleanup becomes political and technical debt

## Failure Mode 2: Developers get Owner by default
Why it fails:
- broad blast radius
- policy/RBAC drift
- governance bypasses become normal

## Failure Mode 3: No paved CI/CD path
Why it fails:
- manual deployments multiply
- secrets spread around
- every team invents a slightly different bad idea

## Failure Mode 4: Networking is undefined
Why it fails:
- private endpoint pain
- DNS confusion
- accidental exposure
- expensive future rework

## Failure Mode 5: No monitoring until later
Why it fails:
- poor incident response
- weak audit trail
- operational blindness from day one

## Failure Mode 6: Cost tagging is optional
Why it fails:
- spend becomes hard to explain
- budgets lose meaning
- shared costs become muddy fast

---

# 9. Well-Architected Framework Crosswalk

## Security
Implementation choices supporting Security:

- group-based RBAC
- least privilege
- PIM/JIT where possible
- policy guardrails
- Defender baseline
- managed identities
- Key Vault
- controlled public exposure
- centralized logging

## Operational Excellence
Implementation choices supporting Operational Excellence:

- subscription baseline through IaC
- repo templates
- policy as code
- GitHub-driven workflows
- naming and tag standards
- documented runbooks
- review and iteration loop

## Reliability
Implementation choices supporting Reliability:

- environment separation
- monitoring and alerts from day one
- tested pilot deployment
- approved network model
- shared-service protection

## Performance Efficiency
Implementation choices supporting Performance Efficiency:

- PaaS-first bias
- right-sized service selection
- approved architecture patterns
- reduced unnecessary network complexity
- telemetry for tuning

## Cost Optimization
Implementation choices supporting Cost Optimization:

- budgets and alerts
- mandatory tags
- clear resource organization
- approved service patterns
- post-deployment spend review

---

# 10. Final Recommended Implementation Pattern

If you want the practical summary in one paragraph:

**Create a new subscription, place it immediately under the correct management group, apply a reusable IaC-driven governance baseline, enable policy/RBAC/monitoring/security/cost controls up front, provide developers resource-group-scoped autonomy and a GitHub Actions OIDC deployment path, then validate the design with a pilot workload and harden the pattern for reuse.**

That is the best balance of:

- speed
- safety
- supportability
- Azure alignment
- developer productivity

---

# 11. Suggested Next Artifacts

The most useful next items to build from this guide are:

1. **Terraform landing zone starter repo**
   - management group assignment
   - RBAC
   - policy assignment
   - monitoring baseline

2. **GitHub Actions OIDC sample workflow**
   - validation
   - plan
   - deploy
   - protected environment pattern

3. **Azure landing zone checklist**
   - pre-flight checks
   - day-one controls
   - production-readiness review

4. **Reference architecture for the app team**
   - network pattern
   - compute pattern
   - data pattern
   - observability pattern

If you want, I can build those next so this becomes not just a guide, but a ready-to-implement kit.
