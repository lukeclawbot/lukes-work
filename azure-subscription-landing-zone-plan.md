# Azure Subscription Landing Zone Plan for App Developers

## Goal

Design the best way to deploy a new Azure subscription into an existing tenant with a landing zone that lets an application developer start building quickly **without creating governance, security, networking, or operational problems later**.

This plan is aligned to the **Azure Well-Architected Framework** and is written with a bias toward:

- repeatable deployment
- safe automation
- clear ownership boundaries
- developer productivity through paved roads, not chaos
- avoiding future cleanup projects disguised as “quick setup”

---

## Executive Summary

The recommended approach is to:

1. **Create a dedicated Azure subscription** for the app team or workload domain
2. **Place it under the correct management group hierarchy**
3. **Apply a standard landing zone baseline** using infrastructure as code
4. **Use policy, RBAC, observability, and cost controls from day one**
5. **Give developers a paved deployment path** instead of unrestricted subscription-wide freedom
6. **Use GitHub Actions with OIDC to Azure** for secure CI/CD
7. **Align the design to all five Azure Well-Architected pillars**

The core principle is simple:

> Do not just hand over a subscription.
> Hand over a governed, observable, secure, developer-ready platform slice.

---

## Recommended Architecture Approach

The best starting model is:

**A lightweight but opinionated enterprise landing zone**

That means the subscription should be:

- governed by management groups
- protected by baseline Azure Policy assignments
- integrated into monitoring and security tooling
- structured with consistent RBAC and naming standards
- designed for CI/CD-first deployment workflows
- flexible enough for development, but not loose enough to become a cloud junk drawer

---

# 1. Management Group Placement

## Recommendation

Place the new subscription into a defined management group hierarchy such as:

- Tenant Root
  - Platform
    - Identity
    - Management
    - Connectivity
  - LandingZones
    - Corp
    - Online
    - AppDev
    - Sandbox
  - Decommissioned

For this app-developer subscription, a likely placement would be:

- `LandingZones/AppDev`

or

- `LandingZones/Online`

depending on whether the workload is internal, internet-facing, or part of a larger application platform.

## Why this matters

Management groups give you:

- inherited governance
- inherited Azure Policy
- cleaner role separation
- consistent cost and compliance patterns
- a scalable structure for future subscriptions

Without this, every new subscription becomes a custom governance project.

## Well-Architected alignment

- **Operational Excellence**: standardizes deployment and governance patterns
- **Security**: enables inherited policy and control boundaries
- **Cost Optimization**: supports consistent cost governance at scale

---

# 2. Treat the Landing Zone as a Baseline, Not a One-Off Build

## Recommendation

Deploy the landing zone using **infrastructure as code**, not manual portal work.

At minimum, the landing zone baseline should define:

- subscription placement
- naming conventions
- required tags
- RBAC assignments
- Azure Policy assignments
- budget and cost alerts
- monitoring and diagnostic settings
- Defender for Cloud baseline
- network model
- identity model
- logging destinations

## Why this matters

If the first subscription is built manually, the second usually will be too.
That leads to:

- drift
- inconsistent controls
- unclear ownership
- difficult troubleshooting
- higher audit and security risk

Using IaC makes the landing zone:

- reproducible
- reviewable
- version-controlled
- easier to improve over time

## Well-Architected alignment

- **Operational Excellence**: repeatable deployment and change control
- **Security**: consistent application of controls
- **Reliability**: fewer one-off configuration mistakes

---

# 3. Identity and Access Model

## Recommendation

Use **Microsoft Entra ID group-based RBAC**, not direct user assignments wherever possible.

Example group pattern:

- `az-sub-appdev-owners`
- `az-sub-appdev-contributors`
- `az-sub-appdev-readers`
- `az-sub-appdev-security-readers`
- `az-sub-appdev-platform-admins`

For application developers, avoid broad **Owner** role assignments at subscription scope unless there is a very specific reason.

Prefer:

- Contributor or more limited roles at **resource group scope**
- separate elevated admin access for platform/security operations
- Privileged Identity Management where available for just-in-time elevation

## Why this matters

Developers need enough access to ship.
They usually do **not** need subscription-wide authority to change:

- governance
- policy
- RBAC
- networking foundations
- security posture

Group-based RBAC also makes access:

- easier to audit
- easier to manage
- easier to automate
- easier to clean up when teams change

## Well-Architected alignment

- **Security**: least privilege and reduced blast radius
- **Operational Excellence**: cleaner access lifecycle management

---

# 4. Azure Policy Guardrails

## Recommendation

Apply a baseline set of Azure Policies at the management group or subscription scope.

### Recommended baseline policies

At minimum, consider policies for:

- allowed Azure regions
- required tags
- resource naming conventions where practical
- denied public IP creation unless approved
- denied public storage access unless approved
- diagnostic settings requirements where supported
- secure transfer requirements for storage
- Key Vault network and security controls
- allowed SKUs or service types if needed
- restrictions on deprecated or legacy resource types
- Defender for Cloud posture requirements

Use a mix of:

- **Deny** for truly disallowed patterns
- **Audit** where you want visibility before enforcement
- **DeployIfNotExists** where Azure can help remediate baseline config

## Why this matters

Policy guardrails prevent known-bad decisions from becoming “temporary exceptions” that live forever.

They help protect against:

- unapproved internet exposure
- missing tags and cost attribution
- insecure storage settings
- missing diagnostics
- accidental use of disallowed services or regions

Do not make the first iteration so restrictive that app teams cannot function.
Start strong, but not suffocating.

## Well-Architected alignment

- **Security**: enforces baseline security posture
- **Operational Excellence**: standardizes platform expectations
- **Cost Optimization**: helps constrain wasteful or unapproved resource usage

---

# 5. Networking Model

## Recommendation

Choose the networking model before developers begin deploying workloads.

The right model depends on whether the applications are:

- primarily PaaS-based
- private/internal only
- internet-facing
- hybrid-connected
- expected to scale into production-grade enterprise workloads

## Default recommendation for a new app-dev landing zone

### Option A: Lightweight app development baseline

Best when the team is building modern cloud apps and can stay mostly PaaS-first.

Characteristics:

- limited VNet usage where necessary
- private endpoints only where needed
- no ad hoc peering by developers
- centralized DNS and connectivity standards if a platform team exists
- approved ingress patterns only

### Option B: Enterprise-aligned application landing zone

Best when the environment is expected to evolve into serious production hosting.

Characteristics:

- hub-spoke or virtual WAN aligned design
- centrally managed connectivity and egress model
- private DNS strategy defined up front
- app teams consume network services rather than inventing them
- route and inspection strategy governed centrally

## Why this matters

Networking is one of the most expensive places to improvise badly.

A poor early network model leads to:

- security drift
- connectivity confusion
- private endpoint and DNS pain
- accidental exposure
- painful refactoring later

## Well-Architected alignment

- **Security**: controlled ingress/egress and reduced exposure
- **Reliability**: predictable connectivity patterns
- **Performance Efficiency**: cleaner network paths and service connectivity

---

# 6. Resource Organization Strategy

## Recommendation

Define a resource group strategy before workloads begin landing.

Example pattern:

- `rg-appdev-shared-dev`
- `rg-appdev-app1-dev`
- `rg-appdev-app1-test`
- `rg-appdev-monitoring`
- `rg-appdev-networking`

Separate:

- shared services
- per-application resources
- per-environment resources
- monitoring and operational services
- network foundations, if those exist in-subscription

## Why this matters

Clear resource group structure improves:

- RBAC scoping
- lifecycle management
- cost reporting
- blast radius control
- operational clarity

If everything lands in a few giant resource groups, access and ownership become blurry fast.

## Well-Architected alignment

- **Operational Excellence**: clearer ownership and manageability
- **Cost Optimization**: better tagging and spend analysis
- **Security**: easier scope control for roles and policies

---

# 7. Observability and Security Baseline

## Recommendation

Enable observability and security tooling **before the first real app deployment**.

Minimum baseline:

- Log Analytics workspace strategy defined
- Activity Log retention/export configured
- diagnostic settings enabled where supported
- Azure Monitor alerting baseline
- Application Insights for app workloads
- Defender for Cloud enabled appropriately
- security monitoring visibility for the subscription

## Why this matters

If you wait until the first outage or security scare to wire in logging and monitoring, you will be investigating with missing context.

A good observability baseline helps with:

- troubleshooting
- compliance
- incident response
- operational maturity
- performance tuning

## Well-Architected alignment

- **Reliability**: earlier detection of failures and service degradation
- **Operational Excellence**: improved operability and troubleshooting
- **Security**: better auditability and threat visibility

---

# 8. Cost Controls from Day One

## Recommendation

Apply cost governance immediately.

Baseline controls:

- subscription budget
- alert thresholds
- required cost allocation tags
- environment tags
- application tags
- owner tags
- optional SKU restrictions or policy controls for expensive services
- review process for high-cost services

If appropriate, add non-production scheduling and shutdown automation later.

## Why this matters

Cost surprises are much easier to prevent than explain.

A strong early cost model helps:

- allocate spend clearly
- identify waste quickly
- keep dev/test costs from quietly ballooning
- support chargeback/showback if needed

## Well-Architected alignment

- **Cost Optimization**: direct alignment with budgeting and usage control
- **Operational Excellence**: better financial accountability and visibility

---

# 9. Developer Paved Road

## Recommendation

Do not stop at deploying the subscription.
Provide the developer with a **standard way to build and deploy**.

That paved road should include:

- a starter repository template
- approved Terraform/Bicep module patterns
- GitHub Actions workflow templates
- deployment identity guidance
- naming and tagging standards
- environment separation guidance
- secret management guidance
- approved services and reference architecture patterns

## CI/CD recommendation

Use:

- GitHub repositories for source control
- PR-based change management
- Terraform and/or Bicep modules for standard patterns
- GitHub Actions for validation, planning, and deployment
- OIDC / federated identity to Azure
- environment-based approvals where needed

## Why this matters

Developers move faster when the platform gives them a path that is already:

- documented
- secure
- supported
- automatable

If every team invents its own deployment and auth model, your tenant becomes a museum of inconsistent cloud folklore.

## Well-Architected alignment

- **Operational Excellence**: standard delivery and deployment patterns
- **Security**: safer defaults through approved patterns
- **Reliability**: more consistent delivery practices
- **Performance Efficiency**: reusable, tested architecture building blocks
- **Cost Optimization**: approved service patterns reduce accidental overspend

---

# 10. Azure Authentication for GitHub Actions

## Recommendation

Use **GitHub Actions OIDC / federated identity** to authenticate to Azure.

Avoid long-lived client secrets whenever possible.

Recommended model:

- separate workload identities by environment or blast radius
- least-privilege role assignments
- approval gates for production workflows
- traceable CI/CD identity usage

## Why this matters

OIDC is preferred because it:

- reduces secret sprawl
- avoids storing long-lived credentials in GitHub
- supports modern cloud-native trust patterns
- improves auditability and operational hygiene

## Well-Architected alignment

- **Security**: eliminates or reduces static secret usage
- **Operational Excellence**: cleaner CI/CD identity model
- **Reliability**: fewer failures due to expired or rotated credentials

---

# 11. Environment Strategy

## Recommendation

Separate environments clearly, even if the initial scope is small.

Baseline example:

- dev
- test
- prod

Each environment should have:

- distinct deployment boundaries
- distinct state handling
- clear promotion flow
- clearly defined ownership

Depending on scale, this may mean:

- multiple resource groups
- separate subscriptions
- separate deployment identities

## Why this matters

Environment blurring is one of the easiest ways to create risk and confusion.

Clear boundaries improve:

- deployment safety
- testing confidence
- rollback clarity
- cost visibility
- access control

## Well-Architected alignment

- **Reliability**: safer testing and promotion paths
- **Security**: clearer identity and access boundaries
- **Operational Excellence**: more disciplined change control

---

# 12. Recommended Deployment Sequence

## Phase 1: Governance foundation

1. Define management group placement
2. Define naming standards
3. Define tagging standards
4. Define Entra ID group model
5. Define RBAC model
6. Define Azure Policy baseline
7. Define budget and cost management approach

## Phase 2: Subscription deployment

8. Create the subscription
9. Move it into the correct management group
10. Apply baseline tags and policies
11. Assign baseline RBAC groups
12. Enable Defender and monitoring integration
13. Configure budget alerts

## Phase 3: Platform baseline inside the subscription

14. Create shared resource groups
15. Configure monitoring and diagnostic routing
16. Establish network baseline
17. Establish secret and identity patterns
18. Document approved service usage

## Phase 4: Developer enablement

19. Create developer access groups
20. Scope access to resource groups where possible
21. provide repo template and workflow examples
22. configure GitHub Actions OIDC to Azure
23. validate first app deployment path

## Phase 5: Hardening and iteration

24. Review policy gaps after first workload deployment
25. tighten controls where safe
26. refine alerting and runbooks
27. review cost and service usage after 30 days
28. update landing zone IaC for future reuse

---

# 13. Azure Well-Architected Framework Mapping

## Security

Key recommendations:

- Entra group-based RBAC
- least privilege access
- PIM/JIT elevation where possible
- policy guardrails
- Defender for Cloud baseline
- managed identities over secrets
- Key Vault for secrets
- controlled network exposure
- centralized logging and audit trail

### Security goal
Make the secure path the default path.

---

## Operational Excellence

Key recommendations:

- landing zone deployed with IaC
- reusable modules/templates
- standardized naming and tags
- policy as code
- PR-driven change management
- GitHub Actions workflows
- documented operating model
- environment promotion discipline

### Operational Excellence goal
Make change repeatable, reviewable, and supportable.

---

## Reliability

Key recommendations:

- environment separation
- baseline monitoring and alerting
- tested deployment paths
- recovery expectations defined early
- zone or regional resilience decisions where needed
- resource protection for critical shared services

### Reliability goal
Avoid fragile environments that accidentally become production.

---

## Performance Efficiency

Key recommendations:

- prefer PaaS where possible
- standardize app hosting patterns
- right-size SKUs
- use autoscaling where needed
- avoid overengineering dev workloads
- collect performance telemetry early

### Performance Efficiency goal
Match architecture to real workload needs, not to theoretical maximum complexity.

---

## Cost Optimization

Key recommendations:

- budgets and alerts from day one
- mandatory cost allocation tags
- SKU guardrails
- visibility into shared vs app-specific spend
- shutdown or scheduling patterns for non-prod where appropriate
- periodic cost review after initial rollout

### Cost Optimization goal
Prevent non-production environments from spending like they are trying to launch a moon program.

---

# 14. What to Avoid

Avoid these common mistakes:

- giving developers **Owner** at subscription scope by default
- building the landing zone manually in the portal
- starting with no policy because “we will add governance later”
- letting each app team invent its own networking and identity model
- using long-lived client secrets in GitHub where OIDC is available
- delaying logging and diagnostics until after the first incident
- blurring dev, test, and prod boundaries for convenience

These usually begin life as “temporary shortcuts” and mature into permanent operational regret.

---

# 15. Recommended End State

The target end state is:

- **Platform ownership** of the landing zone baseline
- **Developer freedom** within approved and governed patterns
- **Infrastructure as code** for repeatable subscription baseline deployment
- **GitHub Actions + OIDC** for secure CI/CD
- **Policy + RBAC + observability** active from the beginning
- **Well-Architected review mindset** as the environment matures

In short:

- developers can move quickly
- governance is not an afterthought
- security is not improvised
- operations are not dependent on tribal knowledge
- the foundation can scale to additional subscriptions and teams

---

# 16. Opinionated Recommendation

If the objective is:

> Give an app developer a new Azure subscription where they can start building without issues

then the best practical answer is:

**Deploy a lightweight enterprise landing zone with governance, RBAC, policy, monitoring, budget controls, and a paved GitHub-based CI/CD path — while keeping developer autonomy mostly at the resource-group and application layer, not the full subscription governance layer.**

That gives the developer room to work while protecting the tenant from drift, misconfiguration, and avoidable chaos.

---

# 17. Suggested Next Deliverables

The next useful documents to create from this plan would be:

1. **Detailed implementation plan**
   - exact rollout steps
   - platform vs developer responsibilities
   - decision points and prerequisites

2. **Terraform/Bicep landing zone starter structure**
   - management group assignments
   - policy assignments
   - RBAC model
   - monitoring baseline

3. **GitHub Actions OIDC deployment pattern**
   - sample workflow
   - Azure federated identity setup
   - environment approvals and role scoping

4. **Azure landing zone review checklist**
   - security
   - governance
   - observability
   - cost
   - developer readiness

If needed, this document can be expanded into a fully detailed, step-by-step implementation guide next.
