# CI Toolbox
A centralized repository of reusable GitHub Actions and Workflows for all projects at my personal projects.

---

### What is this project?
This repository's goal is to standardize and simplify our CI/CD processes. Instead of copying and pasting CI logic across dozens of repositories, we centralize it here.

This toolbox contains two types of automations:
- **Composite Actions (/actions)**: Small, reusable "building blocks" that perform a single task (e.g., setup Node.js, post a comment).
- **Reusable Workflows (/.github/workflows)**: Complete, end-to-end CI pipelines (e.g., Test & Coverage, Publish) that can be "called" by any project.

---

### Project Directory
This is the layout of the ci-toolbox. "Reusable Workflows" are in .github/workflows and "Composite Actions" are in actions.
```bash
ci-toolbox/
├── .github/
│   └── workflows/
│       ├── test.yaml
│       ├── versioning.yaml
│       ├── bundle-size.yaml
│       └── ...
│
├── actions/
│   ├── setup-node/
│   │   └── action.yaml
│   └── post-comment/
│       └── action.yaml
│
└── LICENSE
└── README.md
```

### How to Use in Other Repos
#### 1. Calling a Reusable Workflow (Recommended)
- This is the most common use case. Instead of a large `ci.yaml` file in your project, you'll have a small one that "calls" the workflow from this toolbox.

#### Example: Running the `Test & Coverage` Workflow
Create a file named `.github/workflows/ci.yaml` in your project:

```yaml
name: Main CI

on:
  push:
    branches:
      - main
      - 'feat/**'
      - 'fix/**'
      - 'hotfix/**'
  pull_request:
    branches:
      - main

jobs:
  run-tests:
    uses: itsferdiardiansa/ci-toolbox/.github/workflows/test.yaml@v1.0.0
    
    with:
      node-versions: '[22, 23]'
      
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}
```

**IMPORTANT:** Always use a specific Git tag (e.g., `@v1.0.0`) instead of `@main`. This "version pinning" ensures that updates to the toolbox don't unexpectedly break your project's CI.

### How to Update This Toolbox (Contributing)
Maintaining this toolbox is critical. Follow these best practices.

#### 1. Making Changes
- Create a new branch (e.g., `feat/update-setup-node`).
- Make your changes to the composite action or reusable workflow.
- **Best Practice:** If you edit a workflow (e.g., `test.yaml`) that calls a composite action (e.g., `setup-node`), you must use the `@${{ github.ref_name }}` variable to pin the version. This ensures the workflow is a "sealed" unit.

**Example:** Inside `ci-toolbox/.github/workflows/test.yaml`
```yaml
# GOOD
- name: Setup
  uses: itsferdiardiansa/ci-toolbox/actions/setup-node@${{ github.ref_name }}

# BAD
- name: Setup
  uses: itsferdiardiansa/ci-toolbox/actions/setup-node@main
```

#### 2. Publishing a New Version
- Open a Pull Request with your changes to the `main` branch.
- Get it reviewed and merge it.
- Create a new Git tag on `main` and push it. This is what "publishes" the new version for other projects to use.

```bash
git pull origin main
git tag v1.1.0
git push origin v1.1.0
```

#### 3. Updating Projects
Notify project owners that a new version of the `ci-toolbox` is available. They can then update the `uses:` line in their caller workflows:

Before:
```yaml
uses: itsferdiardiansa/ci-toolbox/.github/workflows/test.yaml@v1.0.0
```

After:
```yaml
uses: itsferdiardiansa/ci-toolbox/.github/workflows/test.yaml@v1.1.0
```

---

### Available Automations
#### Reusable Workflows (/.github/workflows)
| Workflow | Description |
| --- | --- |
| test.yaml | Runs lint, type-check, test (matrix), and uploads coverage to Codecov. [cite: workflows/test.yaml] |
| versioning.yaml | Checks for changesets and creates/updates a "Version" PR. [cite: workflows/versioning.yaml] |
| bundle-size.yaml | Calculates the bundle size impact of a PR and posts a comment. [cite: workflows/bundle-size.yaml] |

#### Composite Actions (/actions)
| Action | Description |
| --- | --- |
| setup-node | Installs Node.js & pnpm, configures the pnpm store, caches dependencies, and runs pnpm install. [cite: actions/ setup-node/action.yaml] |
| post-comment | Finds an existing comment by a unique ID and updates it, or posts a new one. [cite: actions/post-comment/action.yaml] |

---

© 2025 [itsferdiardiansa](https://github.com/itsferdiardiansa)