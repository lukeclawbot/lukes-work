# Detailed Mac Workstation Setup Guide for VS Code, GitHub, and Infrastructure as Code

## Purpose

This guide walks through a practical, opinionated setup for turning a Mac into a solid infrastructure-as-code workstation.

It is written for a workflow centered on:

- **VS Code** for editing
- **GitHub** for source control and pull requests
- **Terraform** for infrastructure provisioning
- **Bicep** for Azure-native deployments where it fits best
- **Azure CLI** for authentication and Azure operations
- **GitHub Actions** for CI/CD

This is not just a list of tools to install.
The real goal is to build a setup that is:

- easy to maintain
- secure enough to be responsible
- friendly to review and automation
- consistent across projects
- less likely to become a giant pile of laptop-specific mystery behavior

---

# Setup Philosophy

Before touching the machine, here is the mindset behind the setup.

## What we want

We want a workstation where:

1. tools are installed in a repeatable way
2. Git and GitHub auth are clean and reliable
3. VS Code helps catch mistakes early
4. infrastructure code is validated before it reaches production
5. day-to-day work flows naturally into PRs and CI/CD
6. future-you does not have to excavate weird shell hacks at 10:30 PM

## What we do not want

We do **not** want:

- random manual installs from assorted websites unless necessary
- duplicate tools fighting over PATH
- one-off fixes with no documentation
- long-lived secrets stuffed everywhere
- production changes that depend on remembering the exact right sequence from memory

That way lies the ancient and terrible art of “it worked once on Steve’s laptop.”

---

# High-Level Plan

We will do this in phases:

1. **Prepare the Mac**
2. **Install package management with Homebrew**
3. **Set up Git correctly**
4. **Set up GitHub authentication**
5. **Install VS Code and useful extensions**
6. **Install infrastructure tooling**
7. **Configure local quality and safety checks**
8. **Create a sane repo and workflow pattern**
9. **Prepare for GitHub Actions and Azure authentication**
10. **Validate the setup end to end**

---

# Phase 1: Prepare the Mac

## Step 1: Confirm macOS is updated enough to be worth trusting

### What to do
- Open **System Settings**
- Go to **General > Software Update**
- Install pending macOS updates if they are reasonable and you have time
- Reboot if required

### Why we are doing this
A lot of dev tooling problems on Macs are not glamorous engineering problems. They are boring system problems.

Updating first helps avoid:

- certificate issues
- broken command line tools
- weird package install failures
- security problems you did not need in your life

This is not about chasing every same-day release instantly. It is about not building your setup on top of a stale foundation.

---

## Step 2: Install Apple Command Line Tools

### What to do
Run:

```bash
xcode-select --install
```

If it says the tools are already installed, that is fine.

### Why we are doing this
Many developer tools on macOS expect Apple’s command line tooling to exist.

This provides basics needed by:

- Git
- compilers
- package builds
- certain Homebrew formulas

Without it, the rest of the setup tends to turn into a scavenger hunt.

---

# Phase 2: Install Homebrew

## Step 3: Install Homebrew

### What to do
Run the official Homebrew install command from the Homebrew website.

At the time of writing, it is commonly:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, follow any shell profile instructions Homebrew gives you.

Then verify:

```bash
brew --version
```

### Why we are doing this
Homebrew gives you a consistent way to install and update tools.

That matters because without a package manager, Mac setup tends to become:

- some tools installed by hand
- some tools installed by drag-and-drop app bundles
- some tools living in random directories
- PATH confusion
- no simple update story

Homebrew keeps your workstation from turning into a yard sale.

---

## Step 4: Confirm your shell is sane

### What to do
Check your shell:

```bash
echo $SHELL
```

Most likely you will use `zsh`, which is the normal modern macOS default.

Then inspect your shell config files if needed:

- `~/.zshrc`
- `~/.zprofile`

Do **not** start adding random aliases and exports yet.

### Why we are doing this
We want your shell to be predictable before layering tools into it.

A clean shell setup means:

- fewer PATH issues
- easier debugging
- less conflict between tool installers

A messy shell config is like plumbing assembled from leftover Lego bricks. Sometimes water still comes out, but nobody feels good about it.

---

# Phase 3: Install Core Tooling

## Step 5: Install core packages with Homebrew

### What to do
Install the baseline toolchain:

```bash
brew install git gh azure-cli terraform pre-commit tflint terraform-docs jq yq
```

If you want an IaC security scanner too, also install one of these:

```bash
brew install checkov
```

If `checkov` is not available or you prefer a Python install path, that can be handled separately.

Optional but useful tools:

```bash
brew install tfenv direnv
```

### Why we are doing this
This gives you the core parts of the workstation:

- `git` for version control
- `gh` for GitHub integration
- `azure-cli` for Azure authentication and management
- `terraform` for infrastructure as code
- `pre-commit` for local quality checks
- `tflint` for Terraform linting
- `terraform-docs` for documentation generation
- `jq` and `yq` for working with JSON and YAML cleanly
- `tfenv` if you need multiple Terraform versions across repos
- `direnv` if you want controlled per-directory environment loading

The reason to install these early is simple: this is the functional skeleton of the workflow.

---

## Step 6: Verify the installed tools

### What to do
Run:

```bash
git --version
gh --version
az version
terraform version
pre-commit --version
tflint --version
terraform-docs --version
jq --version
yq --version
```

### Why we are doing this
Install success is not the same thing as working success.

This step confirms:

- the commands are on your PATH
- the installs actually completed
- your shell can find the binaries

Catching problems here is much cheaper than discovering them halfway through setting up a repo.

---

# Phase 4: Configure Git Properly

## Step 7: Set your Git identity

### What to do
Configure your Git name and email:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

Replace those with your real values.

### Why we are doing this
Every commit needs a clear identity attached to it.

Without this, Git will either complain or create inconsistent commit metadata.

That matters because:

- commit history should be attributable
- GitHub identity matching works better
- teams generally prefer clean authorship

---

## Step 8: Set useful Git defaults

### What to do
Set some practical defaults:

```bash
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global fetch.prune true
git config --global core.editor "code --wait"
```

Optional line ending protection:

```bash
git config --global core.autocrlf input
```

### Why we are doing this
These defaults reduce friction.

- `init.defaultBranch main` keeps new repos aligned with modern GitHub defaults
- `pull.rebase true` often keeps history cleaner during day-to-day branch syncing
- `fetch.prune true` cleans up stale remote-tracking branches
- `core.editor "code --wait"` lets VS Code act as your Git editor when needed
- `core.autocrlf input` helps avoid line-ending nonsense on macOS

These are small settings, but they help Git behave more like a well-trained coworker and less like a mischievous raccoon.

---

## Step 9: Review your Git config

### What to do
Run:

```bash
git config --global --list
```

### Why we are doing this
You want to confirm what is actually set, not what you think you set.

This is especially useful if the machine already had old Git configuration.

---

# Phase 5: Set Up GitHub Access

## Step 10: Authenticate GitHub CLI

### What to do
Run:

```bash
gh auth login
```

Choose:

- **GitHub.com** or your enterprise host as appropriate
- **SSH** for Git operations
- **Login with a web browser** unless you specifically want token flow

Then verify:

```bash
gh auth status
```

### Why we are doing this
GitHub CLI is useful far beyond login.

It helps with:

- repo cloning
- PR creation
- PR review
- issue management
- workflow monitoring

Authenticating it early gives you a smoother GitHub-native workflow from the terminal.

---

## Step 11: Create or verify your SSH key for GitHub

### What to do
First, check whether you already have SSH keys:

```bash
ls -la ~/.ssh
```

If you do not have a suitable key, create one:

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Accept the default path unless you have a reason not to.

Start the SSH agent and add the key if needed.

Then copy the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Add it to GitHub under **SSH and GPG keys**.

Test it:

```bash
ssh -T git@github.com
```

### Why we are doing this
SSH is a clean and reliable way to authenticate Git operations.

Why it is worth using:

- avoids repeated HTTPS credential prompts
- works smoothly with `git clone`, `push`, and `pull`
- plays nicely with GitHub CLI choices

This gives you a stable everyday Git transport method.

---

## Step 12: Test a GitHub clone workflow

### What to do
Clone a test repo or one of your real repos:

```bash
git clone git@github.com:OWNER/REPO.git
```

Or use GitHub CLI:

```bash
gh repo clone OWNER/REPO
```

### Why we are doing this
It is better to prove the auth path works now than discover later that the key was added wrong, the repo URL is wrong, or SSH is not being used.

A quick test now prevents future grumbling.

---

# Phase 6: Install and Configure VS Code

## Step 13: Install VS Code

### What to do
Install VS Code from Microsoft or with Homebrew cask:

```bash
brew install --cask visual-studio-code
```

Open it once after install.

### Why we are doing this
VS Code will be your working surface for:

- editing Terraform and Bicep
- viewing PR-related diffs
- using integrated terminals
- managing settings and extensions

Installing with Homebrew keeps it aligned with the rest of your package-managed setup.

---

## Step 14: Enable the `code` command in your shell

### What to do
In VS Code, open the Command Palette and run:

- **Shell Command: Install 'code' command in PATH**

Then test:

```bash
code --version
```

### Why we are doing this
This lets you:

- open repos from terminal with `code .`
- use VS Code as your Git editor
- bridge terminal and editor workflows cleanly

It is one of those tiny quality-of-life steps that saves a surprising amount of annoyance.

---

## Step 15: Install core VS Code extensions

### What to do
Install these extensions:

- **HashiCorp Terraform**
- **GitHub Pull Requests and Issues**
- **YAML**
- **EditorConfig**
- **Markdownlint**
- **Prettier**

For Azure work, also consider:

- **Bicep** support extension if needed
- Azure extensions if they fit your workflow, but do not install the entire circus unless you will use it

Optional but useful:

- **GitLens**
- **Dev Containers**
- **Error Lens**

### Why we are doing this
These extensions help make VS Code actively useful rather than just aesthetically employed.

They give you:

- Terraform syntax support and formatting awareness
- GitHub integration
- YAML and Markdown validation
- cleaner standards enforcement
- less manual cleanup work

The goal is not maximum extension count. The goal is maximum signal with minimum nonsense.

---

## Step 16: Configure VS Code settings intentionally

### What to do
Add or review these kinds of settings in your VS Code `settings.json`:

```json
{
  "editor.formatOnSave": true,
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "editor.tabSize": 2,
  "editor.detectIndentation": false,
  "terminal.integrated.defaultProfile.osx": "zsh"
}
```

You may want language-specific overrides later for Terraform or JSON.

### Why we are doing this
A good editor setup quietly prevents bad habits.

These settings help enforce:

- cleaner diffs
- consistent formatting behavior
- less accidental whitespace junk
- predictable terminal usage

Basically, this is you asking your tools to help instead of just standing there.

---

# Phase 7: Prepare Azure Tooling

## Step 17: Log in to Azure CLI

### What to do
Run:

```bash
az login
```

If you work with multiple tenants or subscriptions, also inspect them:

```bash
az account list --output table
```

Set the correct subscription if needed:

```bash
az account set --subscription "SUBSCRIPTION_NAME_OR_ID"
```

Confirm current context:

```bash
az account show --output table
```

### Why we are doing this
Azure CLI will be part of your workflow for:

- checking authentication
- managing Bicep features
- validating you are pointed at the right environment
- performing automation tasks when needed

The key reason for this step is to avoid accidental work in the wrong subscription.

Azure happily accepts your mistakes with all the emotional warmth of a vending machine.

---

## Step 18: Verify Bicep is available

### What to do
Run:

```bash
az bicep version
```

If needed, install or upgrade Bicep through Azure CLI:

```bash
az bicep install
az bicep upgrade
```

### Why we are doing this
If you plan to use Bicep at all, you want to verify it now.

This ensures:

- local authoring support exists
- version issues are discovered early
- Azure-native deployment paths are available when you need them

---

# Phase 8: Prepare Terraform for Real Use

## Step 19: Confirm Terraform works cleanly

### What to do
Run:

```bash
terraform version
```

If you expect to work across multiple repos with different versions, decide now whether to use `tfenv`.

If yes, install desired versions through `tfenv` and pin versions per repo.

### Why we are doing this
Terraform version drift causes a ridiculous amount of avoidable pain.

Pinning versions matters because:

- provider compatibility changes
- CI and local runs should agree
- teams should not rely on “whatever version Luke had installed that week”

Predictability beats vibes.

---

## Step 20: Decide on remote state strategy before writing real IaC

### What to do
Do **not** just start with local state for anything meaningful.

Choose and document a remote state pattern, such as:

- Azure Storage for Terraform state
- state separation by environment
- locking and access control considerations

### Why we are doing this
State is the memory of Terraform.

If state is poorly handled, you get:

- drift
- broken applies
- collaboration conflicts
- terrifying uncertainty about what Terraform believes is real

Remote state is not glamorous, but it is table stakes for grown-up infrastructure work.

---

# Phase 9: Add Local Quality Guardrails

## Step 21: Install and initialize pre-commit in repos

### What to do
In each IaC repo, create a `.pre-commit-config.yaml` and install hooks:

```bash
pre-commit install
```

Typical hooks to include later:

- whitespace cleanup
- final newline check
- YAML validation
- Terraform formatting
- `tflint`
- secret detection
- Markdown linting if useful

### Why we are doing this
This is one of the highest-value steps in the whole setup.

`pre-commit` catches mistakes before they become commits.

That means:

- faster feedback
- cleaner PRs
- fewer embarrassing “oops, formatting only” commits
- fewer easy-to-catch mistakes escaping to CI

This is the laptop politely tackling you before you run onto the freeway.

---

## Step 22: Add IaC security scanning

### What to do
Choose a scanning tool such as `checkov` and run it in a repo as part of local checks and CI.

For example, later you might run something like:

```bash
checkov -d .
```

### Why we are doing this
Security scanning is not magic, but it is useful.

It helps catch:

- insecure defaults
- obvious misconfigurations
- risky public exposure patterns
- missing best-practice controls

It should be treated as a guardrail, not as a substitute for judgment.

---

# Phase 10: Create a Good Repo Structure

## Step 23: Use a repo layout that separates reuse from deployment

### What to do
A practical layout often looks like this:

```text
repo/
  .github/
    workflows/
  modules/
  environments/
    dev/
    test/
    prod/
  scripts/
  docs/
```

If you are using Bicep heavily, adapt the names as needed, but keep the separation clear.

### Why we are doing this
This structure helps make the repo understandable.

- `modules/` holds reusable building blocks
- `environments/` holds deployed configurations
- `.github/workflows/` holds automation
- `scripts/` holds repeatable helper commands
- `docs/` holds the “why” and operating knowledge

The real benefit is cognitive clarity.
A good structure reduces the number of times you have to squint and ask, “Now what on earth is this folder for?”

---

## Step 24: Add README and operational notes early

### What to do
Every real IaC repo should get at least:

- a `README.md`
- setup instructions
- validation commands
- deployment flow notes
- environment descriptions

### Why we are doing this
Because documentation written early is useful.
Documentation written after a fire is usually autobiography.

Good repo docs reduce:

- onboarding friction
- dependency on memory
- mis-deployments
- future confusion

---

# Phase 11: Align with GitHub Actions and Safer CI/CD

## Step 25: Design for PR-first changes

### What to do
Adopt a default workflow like this:

1. create branch from `main`
2. make change
3. run local checks
4. open PR
5. let CI run formatting, lint, validation, and security checks
6. generate plan output if relevant
7. review before merge
8. apply through controlled process

### Why we are doing this
This makes infrastructure changes:

- reviewable
- auditable
- easier to discuss
- easier to reverse-engineer later

It also fits your bias toward safer automation and version-controlled operations.

---

## Step 26: Plan Azure authentication for GitHub Actions with OIDC

### What to do
When you build CI/CD, prefer:

- GitHub Actions OIDC / federated identity into Azure
- environment-specific roles
- least privilege
- approval gates for production

Avoid long-lived client secrets unless you have no better option.

### Why we are doing this
Long-lived secrets are annoying to manage and easy to mishandle.

OIDC is better because it:

- reduces secret sprawl
- improves security posture
- fits modern cloud CI/CD patterns
- gives you cleaner trust relationships

This is one of those choices that is slightly more effort up front and much less regret later.

---

# Phase 12: Validate the Whole Workstation

## Step 27: Test the full daily workflow

### What to do
Run through a realistic loop:

1. clone a repo
2. open it with `code .`
3. run `terraform fmt` and `terraform validate`
4. run linting tools
5. run `pre-commit run --all-files`
6. verify Azure CLI login and context
7. create a branch
8. make a small change
9. commit and push
10. open a PR with `gh`

Example commands:

```bash
code .
terraform fmt -recursive
terraform validate
pre-commit run --all-files
git checkout -b test/setup-validation
git add .
git commit -m "Test workstation setup"
git push -u origin test/setup-validation
gh pr create
```

### Why we are doing this
A workstation is not done when the tools are installed.
It is done when the real workflow works.

This final pass proves:

- Git works
- GitHub auth works
- VS Code integration works
- Terraform works
- local quality checks work
- the machine supports the way you actually want to work

That is the difference between “configured” and “ready.”

---

# Recommended Default Choices for You

Based on the kind of workflow you prefer, these are the default choices I would recommend.

## Tool choices
- Homebrew for installation and updates
- `zsh` as the shell
- Git + GitHub CLI
- VS Code as the primary editor
- Terraform + Bicep + Azure CLI
- `pre-commit`, `tflint`, `terraform-docs`, and `checkov`

## Workflow choices
- SSH for GitHub Git transport
- PR-first infrastructure changes
- local validate/lint before commit when practical
- GitHub Actions for CI/CD
- OIDC to Azure instead of long-lived secrets
- environment separation between dev, test, and prod

## Design choices
- reusable modules separated from deployed environments
- branch protection for important repos
- plan-before-apply discipline
- documentation that explains both setup and operation

---

# Common Mistakes to Avoid

## Mistake 1: Installing everything manually from random websites
Why it is bad:
- hard to maintain
- hard to update
- creates PATH confusion

## Mistake 2: Skipping Git and SSH cleanup
Why it is bad:
- repo access breaks at the worst time
- commit identity gets messy
- you lose confidence in the setup

## Mistake 3: Treating local laptop auth as the CI/CD strategy
Why it is bad:
- not repeatable
- not auditable
- encourages click-ops and secret sprawl

## Mistake 4: Using no local checks
Why it is bad:
- simple mistakes keep reaching PRs
- CI becomes the first line of defense instead of the second

## Mistake 5: Blurring environment boundaries
Why it is bad:
- raises deployment risk
- makes changes harder to reason about
- turns “quick shortcuts” into recurring pain

---

# What “Good” Looks Like at the End

By the end of this setup, you should be able to:

- install and update tools cleanly with Homebrew
- use Git and GitHub without auth friction
- open any IaC repo in VS Code and get productive quickly
- validate and lint Terraform locally
- use Azure CLI confidently without guessing subscription context
- structure repos cleanly for modules and environments
- move naturally into GitHub Actions-based CI/CD
- avoid depending on portal click-ops for repeatable work

That is the target.
Not “maximum number of dev tools.”
Not “most customized terminal on earth.”
Just a clean, dependable workstation that helps you do solid infrastructure work.

---

# Best Next Documents to Create

The next practical documents I recommend are:

1. **Exact install checklist**
   - copy/paste commands only
   - minimal commentary

2. **Recommended VS Code settings.json**
   - ready to paste into your Mac

3. **Starter `.pre-commit-config.yaml` for Terraform/Bicep repos**
   - real local guardrails

4. **Starter GitHub Actions workflow**
   - format, lint, validate, and plan

5. **Starter repo structure template**
   - folders, placeholders, README, scripts

If you want, I can build those next and make this go from “good guide” to “ready-to-use kit.”
