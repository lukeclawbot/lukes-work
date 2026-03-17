# Mac VS Code + GitHub Setup Plan for Infrastructure as Code

## Goal

Build a clean, repeatable, low-drama Mac setup for working on infrastructure as code with:

- VS Code
- GitHub
- Terraform
- Bicep
- Azure CLI
- GitHub Actions

The goal is not just to install tools. The goal is to create a workflow that is:

- easy to maintain
- safe by default
- friendly to PR-based work
- structured for validation before deployment
- scalable from personal projects to team repos

---

## Recommended Approach

My recommended baseline for your setup is:

- **Mac + Homebrew** for package management
- **Git + GitHub CLI** for source control and GitHub auth
- **VS Code** as the main editor
- **Terraform + Bicep + Azure CLI** for IaC work
- **GitHub Actions** for CI/CD
- **pre-commit + linters/security checks** for local quality gates
- **SSH-based GitHub auth** for repo access
- **Plan-before-apply workflow** for safer infrastructure changes

This gives you a setup that stays practical, reviewable, and much less likely to become a future-you cleanup project.

---

## 1. Base Mac Setup

### Install Homebrew
Use Homebrew as the default way to install and maintain your tooling.

Why:

- keeps installs consistent
- makes updates simpler
- avoids random manual package drift
- reduces mystery-tool chaos

### Suggested core packages
Install or verify these first:

- `git`
- `gh` (GitHub CLI)
- `azure-cli`
- `terraform`
- `pre-commit`
- `tflint`
- `terraform-docs`
- `checkov` or an equivalent IaC security scanner
- `jq`
- `yq`
- `wget` or `curl` if needed

Optional but useful:

- `tfenv` if you work across multiple Terraform versions
- `direnv` if you want per-folder environment loading
- `podman` or `docker` if you want devcontainers or local container tooling

### Shell sanity
Keep your shell setup boring and predictable.

Recommended:

- use `zsh` unless you have a strong reason not to
- keep PATH edits minimal
- prefer Homebrew-managed binaries over hand-installed duplicates
- avoid piling unrelated customizations into shell startup files

If the shell config starts looking like an archaeological dig, that is the moment to stop and clean it up.

---

## 2. Git and GitHub Foundation

### Git configuration
Set these basics:

- user name
- email
- default branch name
- sensible pull behavior
- credential/auth strategy

Recommended defaults:

- default branch: `main`
- pull strategy: rebase or fast-forward only, depending on preference
- line endings: keep them consistent and avoid accidental cross-platform churn

### GitHub authentication
Recommended approach:

- use **GitHub CLI** for general GitHub auth and workflow tasks
- use **SSH keys** for cloning and pushing repos

Why SSH:

- durable and reliable for day-to-day git use
- easier for repo operations once configured
- avoids repeated HTTPS credential friction

### Optional but worthwhile
Consider adding:

- **commit signing** if you want stronger identity assurance
- GitHub org SSO verification if your employer or org requires it

---

## 3. VS Code Setup

## Core editor goals
Your VS Code setup should help enforce good habits automatically:

- format on save
- visible linting problems
- easy terminal access
- extension support for Terraform, Bicep, YAML, JSON, and GitHub workflows
- minimal friction when switching repos

### Recommended extensions
Install a focused set, not every extension with a shiny icon.

Recommended baseline:

- **HashiCorp Terraform**
- **Azure Tools** or at minimum Bicep support
- **GitHub Pull Requests and Issues**
- **YAML**
- **EditorConfig**
- **Markdownlint**
- **Prettier** for non-IaC text/config formatting where appropriate
- **GitLens** if you like deeper Git visibility

Optional:

- **Dev Containers** if you want consistent repo-specific environments
- **Error Lens** if you want errors surfaced more aggressively

### Recommended VS Code settings
At a minimum:

- enable format on save
- enable trimming trailing whitespace
- insert final newline
- keep autosave behavior intentional, not chaotic
- use integrated terminal
- set Terraform formatting and validation support cleanly

The idea is simple: make the editor quietly prevent sloppiness.

---

## 4. Infrastructure as Code Toolchain

### Terraform
Recommended local capabilities:

- `terraform fmt`
- `terraform validate`
- version pinning per repo
- provider version constraints
- remote state strategy defined clearly

Strong recommendation:

- do not rely on local ad hoc applies as the main workflow
- use local validation for fast feedback
- use CI/CD for reviewable plan/apply flows where possible

### Bicep
Use Bicep where it makes sense in Azure-native scenarios, especially where ARM-native coverage or Azure-specific expressiveness is useful.

Recommended:

- ensure Bicep CLI support is installed via Azure CLI or separately as needed
- keep Bicep modules versioned and structured
- validate templates before deployment

### Azure CLI
Use Azure CLI for:

- identity/login verification
- subscription/context management
- Bicep support
- operational checks and automation helpers

Recommended habits:

- verify subscription context before making changes
- avoid relying on one sticky default subscription forever
- prefer named scripts over one-off terminal heroics

---

## 5. Local Quality and Safety Guardrails

This is the part that saves pain.

### Recommended local checks
Use `pre-commit` to run lightweight checks before code lands in Git.

Suggested checks:

- Terraform format
- Terraform validate
- `tflint`
- YAML/JSON formatting and validation
- trailing whitespace cleanup
- large file detection
- secret detection if possible
- Markdown linting where useful

### Security and policy scanning
Recommended:

- `checkov` or equivalent for IaC scanning
- optional tfsec/trivy depending on your preference and stack
- treat these as guardrails, not decorative stickers

### Philosophy
The local machine should make it easy to catch dumb mistakes early.

That means:

- fast feedback locally
- stricter enforcement in CI
- fewer opportunities for “I meant to check that” moments

---

## 6. Repository Structure

A good repo structure matters more than people like to admit.

### Recommended principles
Prefer:

- clear separation of modules and live environments
- predictable folder names
- environment-specific configuration kept obvious
- reusable modules separated from deployed stacks
- documentation near the code

### Common patterns
A practical structure often looks like:

- `modules/` for reusable Terraform or Bicep building blocks
- `environments/` or `live/` for deployed environment definitions
- `.github/workflows/` for CI/CD
- `docs/` for design notes and runbooks
- `scripts/` for repeatable helper automation

### Avoid
Try not to build a repo that:

- mixes reusable modules and production config randomly
- hides environment differences in mystery variables
- relies on tribal knowledge to deploy safely

If a repo requires a small oral tradition to operate, it is already slightly cursed.

---

## 7. GitHub Workflow Design

### Recommended flow
For infrastructure work, prefer:

1. branch from `main`
2. make changes
3. run local validation
4. open PR
5. CI runs formatting/lint/validate/security checks
6. generate plan output where appropriate
7. review
8. merge
9. apply through controlled workflow

### Why this works
This supports:

- auditability
- safer collaboration
- easier rollback understanding
- better visibility into what changed and why

### GitHub Actions pipeline basics
At minimum, your pipeline should handle:

- formatting checks
- linting
- validation
- security scanning
- Terraform plan for changed areas where practical

For mature repos, add:

- environment-aware workflows
- required reviewers
- protected branches
- manual approval gates for production
- reusable workflows across repos

---

## 8. Azure Authentication for CI/CD

This is an area where “works on my laptop” can become “security incident with branding.”

### Recommended approach
Prefer:

- GitHub Actions with **federated identity / OIDC** into Azure
- avoid long-lived client secrets where possible
- separate identities by environment or blast radius
- least-privilege roles

### Avoid when possible
- stuffing long-lived Azure secrets into GitHub
- using overly broad contributor permissions everywhere
- sharing one giant service principal across all environments

### Good pattern
A strong setup usually looks like:

- one repo or workflow identity model that is easy to understand
- scoped permissions per subscription/resource group/environment
- approvals for production changes
- logging and traceability preserved

---

## 9. Environment Strategy

For infrastructure as code, environment design matters as much as tooling.

### Recommended baseline
Use clearly separated environments such as:

- dev
- test
- prod

Each should have:

- distinct state
- distinct auth boundaries where possible
- clearly defined promotion path
- minimal ambiguity about what is safe to change

### Strong recommendation
Do not blur environment boundaries just because it feels faster in the moment.

That almost always becomes slower later, usually at the worst possible time.

---

## 10. Best-Practice Mac Setup Summary

If I were optimizing for your likely preferences, I would recommend this as the default stack:

### Tooling
- Homebrew
- Git
- GitHub CLI
- VS Code
- Azure CLI
- Terraform
- Bicep
- pre-commit
- tflint
- terraform-docs
- checkov
- jq/yq

### VS Code extensions
- HashiCorp Terraform
- GitHub Pull Requests and Issues
- YAML
- EditorConfig
- Markdownlint
- Prettier
- optional Azure/Bicep support extensions

### Workflow
- SSH auth for git operations
- GitHub CLI for GitHub interactions
- format/lint/validate locally
- PR-driven infrastructure changes
- GitHub Actions for CI/CD
- OIDC to Azure for deployment auth
- plan before apply
- branch protection and approvals for production

### Design principles
- modules separated from environments
- reproducible tooling
- minimal manual click-ops
- safer automation over cowboy automation
- clear documentation for future-you

---

## 11. Suggested Phased Rollout

### Phase 1: Foundation
- install Homebrew-managed tools
- configure Git
- configure GitHub auth
- install VS Code and core extensions

### Phase 2: IaC readiness
- install Terraform, Azure CLI, Bicep support, linters
- test `terraform fmt`, `validate`, and Azure login flows
- create a starter repo structure

### Phase 3: Guardrails
- add `pre-commit`
- add lint/security/documentation tooling
- standardize VS Code settings

### Phase 4: CI/CD
- add GitHub Actions for validate/lint/plan
- wire Azure auth with OIDC
- add branch protections and approval flow

### Phase 5: Maturity
- reusable workflows
- environment promotion discipline
- shared modules
- documentation and operating runbooks

---

## 12. Practical Outcome

If this is done well, your Mac becomes a clean control center for infrastructure work:

- easy to open a repo and get going
- low friction for writing and reviewing IaC
- fewer configuration surprises
- safer deployments
- cleaner GitHub workflow
- less dependence on manual portal click-ops

That is the real win.
Not “look at all my extensions.”
Not “I have seven half-configured CLIs and a dream.”

A setup that helps you do solid work repeatedly is the goal.

---

## 13. Next Step Options

From here, the best next documents to create would be one of these:

1. **Exact install checklist for your Mac**
   - concrete Homebrew commands
   - Git config commands
   - VS Code extension list
   - validation commands

2. **Opinionated VS Code settings file**
   - a ready-to-use `settings.json`
   - recommended extension list
   - terminal/editor defaults

3. **Starter GitHub repo template for Azure IaC**
   - folder structure
   - `pre-commit` config
   - sample GitHub Actions workflow
   - Terraform/Bicep starter layout

4. **End-to-end best-practice blueprint**
   - machine setup
   - repo setup
   - CI/CD setup
   - Azure auth strategy
   - branch protection and production guardrails

If you want, I can turn this into the next level down and make it fully actionable with exact commands and file examples.