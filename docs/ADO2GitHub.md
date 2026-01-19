Published Date: 2026-19-01 19th January 2026

Title: Migrating from Azure DevOps to GitHub: An In-Depth Guide for Teams

Hero image: ado2github.png

# Migrating from Azure DevOps to GitHub: An In-Depth Guide for Teams

*Inspired by insights from April Yoho (GitHub) and real-world migration stories.*

---

## Introduction

Many teams are looking to migrate from **Azure DevOps (ADO)** to **GitHub** to take advantage of modern developer workflows, integrated automation, and enhanced collaboration. If you’re considering or planning this migration, this post covers *everything you need*: technical steps, pitfalls, code examples, process comparisons, and tips for making the switch seamless.

> **Fact:** GitHub and Azure DevOps are both owned by Microsoft—but the platforms evolved with different focuses. Migrations are not just possible, they are well-supported and increasingly common.

---

## Table of Contents

1. [Why Migrate? Key Benefits](#why-migrate-key-benefits)
2. [Mapping Features: ADO vs. GitHub](#mapping-features-ado-vs-github)
3. [Step-by-Step Migration Process](#step-by-step-migration-process)
    - [1. Assessment & Planning](#1-assessment--planning)
    - [2. Preparing GitHub](#2-preparing-github)
    - [3. Migrating Repositories (with Code)](#3-migrating-repositories-with-code)
    - [4. Work Item and Boards Migration](#4-work-item-and-boards-migration)
    - [5. Pipeline Migration (ADO Pipelines to GitHub Actions)](#5-pipeline-migration-ado-pipelines-to-github-actions)
    - [6. Artifacts & Packages Migration](#6-artifacts--packages-migration)
    - [7. Wiki & Documentation Migration](#7-wiki--documentation-migration)
    - [8. Training & Change Management](#8-training--change-management)
4. [Technical Tips, Tools, and Gotchas](#technical-tips-tools-and-gotchas)
5. [Conclusion](#conclusion)
6. [References and Resources](#references-and-resources)

---

## Why Migrate? Key Benefits

- **Unified Developer Experience**: One place for code, issues, planning, and automation.
- **Modern Automation**: GitHub Actions offers an event-driven, open-source-powered CI/CD engine.
- **Collaboration**: Easy cross-org, cross-repo collaboration; open source-friendly.
- **Security**: Code scanning, Dependabot, and advanced permissions.
- **Community**: Vast marketplace and user base.

---

## Mapping Features: ADO vs. GitHub

| ADO Feature        | GitHub Equivalent            | Notes                                          |
|--------------------|-----------------------------|------------------------------------------------|
| Repos (Git)        | Repos (Git)                 | Seamless migration; TFVC requires conversion   |
| Work Items         | Issues, Projects            | Custom fields, automation; less reporting      |
| Boards             | Projects (Next Gen)         | Kanban, Table views                            |
| Pipelines (YAML)   | Actions (YAML)              | Requires migration; syntax differs             |
| Pipelines (Classic)| -                           | No direct equivalent; refactor to YAML         |
| Artifacts          | GitHub Packages             | Test compatibility with package types          |
| Wikis              | Repo Wikis, Markdown files  | Export to Markdown recommended                 |
| Extensions         | GitHub Marketplace, Actions | Larger ecosystem in GitHub                     |

---

## Step-by-Step Migration Process

### 1. Assessment & Planning

- **Inventory all assets**: Repos, Boards, Pipelines, Packages, Permissions.
- **Map dependencies**: Know what is ADO-only and must be replaced/refactored.
- **Choose a pilot**: Use a non-critical or isolated project for trial migration.

> **Tip**: Document every process your team uses in ADO so nothing gets lost in translation.

---

### 2. Preparing GitHub

- Create a new GitHub Organization (if needed).
- Set up teams, permissions, security policies, branch protection rules, CODEOWNERS, etc.
- Enable SSO or role mapping if integrating with enterprise identity solutions.

```yaml
# Example: enforce branch protection via .github/workflows/policy.yml
name: Branch Policy
on:
  push:
    branches: [main]
jobs:
  enforce:
    runs-on: ubuntu-latest
    steps:
      - name: Require PR reviews
        uses: actions/github-script@v7
        with:
          script: |
            github.repos.updateBranchProtection({
              owner: context.repo.owner,
              repo: context.repo.repo,
              branch: 'main',
              required_pull_request_reviews: { required_approving_review_count: 2 }
            })
```

---

### 3. Migrating Repositories (with Code)

**For Git-Based Repos:**  
Here’s how you can migrate a Git repo from ADO to GitHub.

```bash
# Clone from Azure DevOps (mirror to capture all branches/tags)
git clone --mirror https://dev.azure.com/ADO_ORG/ADO_PROJECT/_git/REPO_NAME
cd REPO_NAME.git

# Push to GitHub (replace placeholder values)
git remote add github https://github.com/GITHUB_ORG/REPO_NAME.git
git push --mirror github
```

- Mirror cloning grabs every branch, tag, and ref.
- After migration, encourage devs to update local remotes.

**For TFVC:**  
You'll need to first export/convert to Git (many organizations use [git-tfs](https://github.com/git-tfs/git-tfs)).

---

### 4. Work Item and Boards Migration

- Each ADO Work Item (user story, bug, etc.) can become a GitHub Issue.
- Use [Azure DevOps Migration Tools](https://github.com/solidify/jira-azuredevops-migrator/) or [ado2gh](https://github.com/github/ado-migration) for bulk migration.
- Custom fields may need mapping or creative use of GitHub labels and custom fields.

**Converting a Work Item:**

```json
{
  "title": "Add user authentication",
  "body": "Migrated from ADO: User Story #1234",
  "labels": ["user story", "backlog"]
}
```
*Use the [GitHub REST API](https://docs.github.com/en/rest/issues/issues#create-an-issue) to bulk-create issues if needed.*

---

### 5. Pipeline Migration (ADO Pipelines to GitHub Actions)

- Migrate YAML-based ADO pipelines to GitHub Actions workflows.
- Refactor (or redesign) multi-stage pipelines, deployment environments, and secrets.

**ADO Example (azure-pipelines.yml):**
```yaml
trigger:
- main

jobs:
- job: build
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - script: npm install
  - script: npm test
```

**Equivalent GitHub Actions (.github/workflows/ci.yml):**
```yaml
name: CI
on:
  push:
    branches: [main]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test
```

*Notes:*
- Triggers and job environments are similar, but GitHub Actions syntax is unique.
- Use the [actions/checkout](https://github.com/actions/checkout) and [actions/setup-node](https://github.com/actions/setup-node) for Node.js workflows.
- For secrets, use GitHub’s encrypted secrets at organization, repository, or environment scope.

---

### 6. Artifacts & Packages Migration

- Push packages from ADO to GitHub using standard tools (npm, NuGet, Maven, etc.)
- Update `.npmrc`, `nuget.config`, etc., to point to GitHub Packages.

**Example: Publishing npm package to GitHub Packages**  
`.npmrc` for GitHub:
```ini
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
@your-org:registry=https://npm.pkg.github.com
```
**Publish Command:**
```bash
npm publish --access public
```

*Note:* Test that your build/deploy processes resolve dependencies from the new registry.

---

### 7. Wiki & Documentation Migration

- Export ADO Wiki as Markdown (`.md`) files.
- Add docs to `/docs` directory, `README.md`, or use GitHub’s built-in wiki.
- Re-link images (ADO stores them differently!).
- Use a docs generator like [MkDocs](https://www.mkdocs.org/) or [Docusaurus](https://docusaurus.io/) for beautiful documentation portals if desired.

---

### 8. Training & Change Management

- Run hands-on sessions for devs and Product Owners on GitHub Issues, Projects, Actions, and Security features.
- Share internal guides and cheat sheets.
- Appoint “GitHub Champions” on each team to help with transition.
- Monitor feedback closely for the first sprint after going live.

---

## Technical Tips, Tools, and Gotchas

- **Test everything in a pilot project before running organization-wide.**
- **Plan board/issue mapping:** Some ADO customizations don’t translate 1:1; simplify your workflow where possible.
- **CI environments:** GitHub Actions runners and ADO agents have slightly different permissions/networks.
- **Reliance on ADO Extensions:** Check whether there’s an equivalent GitHub App or Action.
- **Artifacts retention, permissions, and clean-up**: Review how GitHub handles these compared to ADO.
- **Incremental migration:** It’s fine to run both systems in parallel for a transition period; synchronize key pipelines and issues as needed.

---

## Conclusion

Migrating from Azure DevOps to GitHub lets your team modernize, simplify, and collaborate more effectively. While migrating can be complex, breaking it down into logical phases and combining technical migration with cultural change makes the process approachable—even enjoyable.

> **Pro Tip:** Treat migration as a chance to clean house: drop unused pipelines, clarify your documentation, and adopt best practices that weren’t possible (or obvious) in your old setup.

---

## References and Resources

- [GitHub Docs: Moving from Azure DevOps](https://docs.github.com/en/migrations/)
- [Azure DevOps Migration Tools](https://github.com/github/azure-devops-migration-tools)
- [ADO to GitHub Issues Migration Tool](https://github.com/github/ado-to-gh-issues)
- [Migrating Work Items from Azure Boards to GitHub Issues](https://learn.microsoft.com/en-us/azure/devops/migrate/migrate-from-azure-boards-to-github-issues)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Sample Repos for Migration Practice](https://github.com/solidify/JiraAzureDevOpsMigration)
- [MkDocs: Project Documentation Generator](https://www.mkdocs.org/)

---

If you have questions, want migration scripts or YAML templates for your specific use case, or want to know how to get help from GitHub’s engineering team—let’s connect!

#AzureDevOps #GitHub #Migration #DevOps #ProjectManagement #GitHubActions #Engineering #AprilYoho