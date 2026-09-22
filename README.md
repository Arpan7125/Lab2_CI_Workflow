# Lab Exercise 2 — Basic CI Workflow with GitHub Actions

**Course:** DevOps | **Instructor:** Cynthia T | **Points:** 10

## Objective

Design and implement a basic Continuous Integration (CI) workflow that is
automatically triggered by commits, using GitHub Actions.

## Project Overview

A small Node.js "calculator" library (`src/calculator.js`) with a Jest test
suite (`tests/calculator.test.js`) is used as the sample application. The CI
pipeline lints and tests the code automatically every time a commit is
pushed, catching regressions before they reach `main`.

```
Lab2_CI_Workflow/
├── .github/
│   └── workflows/
│       └── ci.yml          # GitHub Actions CI workflow definition
├── src/
│   └── calculator.js        # Sample application code
├── tests/
│   └── calculator.test.js   # Jest unit tests
├── docs/
│   ├── CI_WORKFLOW.md       # Lab write-up with screenshots
│   ├── CI_WORKFLOW.docx     # Same write-up as a Word document
│   └── screenshots/         # Captured evidence of the CI runs
├── .eslintrc.json           # Lint rules
├── package.json
└── README.md
```

## Documentation

Full write-up with pipeline design and screenshots of the executed runs:
[`docs/CI_WORKFLOW.md`](docs/CI_WORKFLOW.md) — also available as a Word
document for submission: [`docs/CI_WORKFLOW.docx`](docs/CI_WORKFLOW.docx)

## CI Workflow Design

File: [`.github/workflows/ci.yml`](.github/workflows/ci.yml)

| Aspect            | Detail                                                              |
|-------------------|-----------------------------------------------------------------------|
| **Trigger**       | `push` to any branch, and `pull_request` targeting `main`             |
| **Runner**        | `ubuntu-latest`                                                       |
| **Matrix**        | Runs the pipeline against Node.js `18.x` and `20.x`                   |
| **Steps**         | 1) Checkout code → 2) Set up Node.js → 3) `npm install` → 4) `npm run lint` → 5) `npm test` |
| **Caching**       | npm dependency cache enabled via `actions/setup-node`                 |

Every commit therefore automatically:
1. Checks out the latest code.
2. Installs dependencies.
3. Lints the code with ESLint.
4. Runs the Jest test suite.

If any step fails, the workflow run is marked failed and shows up as a red ❌
next to the commit / PR on GitHub, giving immediate feedback to the
developer.

## How to Run Locally

```bash
npm install
npm run lint
npm test
```

## How to Reproduce the CI Trigger on GitHub

1. Create a new GitHub repository and push this folder to it:
   ```bash
   git init
   git add .
   git commit -m "Lab 2: Add basic CI workflow with GitHub Actions"
   git branch -M main
   git remote add origin <your-repo-url>
   git push -u origin main
   ```
2. Open the **Actions** tab on GitHub — the `CI` workflow run starts
   automatically because of the `push` trigger in `ci.yml`.
3. Make any change (e.g., edit `src/calculator.js`), commit, and push again —
   a new workflow run is triggered automatically, demonstrating CI on every
   commit.
4. Open a pull request into `main` to see the `pull_request` trigger fire as
   well, with results shown as required/optional checks on the PR.

## Result

- ✅ CI workflow triggers automatically on every commit / push.
- ✅ Lint and test stages run in an isolated, reproducible environment.
- ✅ Build status is visible directly on GitHub (Actions tab / commit status
  checks), giving fast feedback on code health.

## Conclusion

This exercise demonstrates a minimal but complete CI pipeline: source
control triggers automated build/lint/test steps on every commit via GitHub
Actions, forming the foundation for larger CI/CD pipelines (adding stages
like build artifacts, deployment, and notifications).
