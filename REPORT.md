# Lab 2 Report — Basic CI Workflow with GitHub Actions

**Course:** DevOps Lab (5th Trimester)
**Institution:** Department of Computer Science, CHRIST (Deemed to be University)
**Instructor:** Cynthia T
**Points:** 10
**GitHub Repository:** [`Arpan7125/Lab2_CI_Workflow`](https://github.com/Arpan7125/Lab2_CI_Workflow)
**Default Branch:** `main`

---

## 1. Objective

Design and implement a basic Continuous Integration (CI) workflow that is
automatically **triggered by commits**, using **GitHub Actions**.

The exercise required demonstrating:
1. A sample application with an automated test suite.
2. A CI pipeline definition (`ci.yml`) that installs dependencies, lints, and
   tests the code.
3. Proof that the pipeline runs automatically the moment new commits are
   pushed to GitHub — with no manual intervention.

---

## 2. Project Structure

A minimal Node.js "calculator" library was used as the sample application so
the CI pipeline has real build/lint/test steps to execute.

```
Lab2_CI_Workflow/
├── .github/
│   └── workflows/
│       └── ci.yml            # GitHub Actions CI workflow definition
├── src/
│   └── calculator.js         # Sample application (add/subtract/multiply/divide)
├── tests/
│   └── calculator.test.js    # Jest unit tests (5 test cases)
├── .eslintrc.json            # ESLint configuration
├── .gitignore
├── package.json
├── package-lock.json
├── README.md
└── REPORT.md                 # This report
```

---

## 3. CI Workflow Design

File: [`.github/workflows/ci.yml`](.github/workflows/ci.yml)

```yaml
name: CI

on:
  push:
    branches: ["**"]
  pull_request:
    branches: ["main"]

jobs:
  build-and-test:
    name: Build & Test (Node ${{ matrix.node-version }})
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Set up Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - name: Install dependencies
        run: npm install

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test
```

| Design Decision   | Rationale                                                                 |
|--------------------|---------------------------------------------------------------------------|
| `on.push: ["**"]`  | Guarantees the workflow is **triggered by every commit**, on any branch, satisfying the core lab requirement. |
| `on.pull_request`  | Adds a second trigger so PRs into `main` are also validated before merge. |
| Matrix build (18.x/20.x) | Verifies the code works across two LTS Node versions, catching version-specific regressions. |
| `cache: "npm"`     | Speeds up repeated runs by caching `node_modules` between commits.        |
| Separate lint + test steps | Isolates failures — a lint failure is reported distinctly from a test failure. |

---

## 4. Setup Sequence Performed

1. **Local project scaffolding** — created `src/calculator.js`, Jest tests in
   `tests/calculator.test.js`, ESLint config, `package.json`, and the
   workflow file under `.github/workflows/ci.yml`.

2. **Local verification before pushing:**
   ```bash
   npm install
   npm run lint
   npm test
   ```
   Result: **5/5 tests passed**, lint clean.

3. **Git initialization and first commit:**
   ```bash
   git init
   git add -A
   git commit -m "Lab 2: Add basic CI workflow with GitHub Actions"
   ```

4. **Remote repository creation** (via GitHub CLI, account `Arpan7125`):
   ```bash
   gh repo create Lab2_CI_Workflow --public --source=. --remote=origin \
     --description "Lab Exercise 2: Basic CI workflow triggered by commits using GitHub Actions"
   ```
   → Created: `https://github.com/Arpan7125/Lab2_CI_Workflow`

5. **Push to `main`:**
   ```bash
   git push -u origin master:main
   ```
   (Initial push was rejected once because the default GitHub CLI token
   lacked the `workflow` OAuth scope required to create/update files under
   `.github/workflows/`. Fixed with `gh auth refresh -s workflow`, then the
   push succeeded.)

---

## 5. Proof of Automatic Trigger on Commit

Immediately after `git push`, GitHub Actions picked up the `push` event on
`main` with **no manual trigger** — confirmed via `gh run list`:

```
STATUS      NAME                                                WORKFLOW  BRANCH  EVENT  ID           ELAPSED  AGE
in_progress Lab 2: Add basic CI workflow with GitHub Actions     CI        main    push   35686529676  10s      2026-09-22T04:20:45Z
```

### Full run log (`gh run watch 35686529676 --exit-status`)

```
✓ main CI · 35686529676
Triggered via push less than a minute ago

JOBS
✓ Build & Test (Node 20.x) in 16s (ID 106614439854)
  ✓ Set up job
  ✓ Checkout repository
  ✓ Set up Node.js 20.x
  ✓ Install dependencies
  ✓ Run linter
  ✓ Run tests
  ✓ Post Set up Node.js 20.x
  ✓ Post Checkout repository
  ✓ Complete job
✓ Build & Test (Node 18.x) in 17s (ID 106614439944)
  ✓ Set up job
  ✓ Checkout repository
  ✓ Set up Node.js 18.x
  ✓ Install dependencies
  ✓ Run linter
  ✓ Run tests
  ✓ Post Set up Node.js 18.x
  ✓ Post Checkout repository
  ✓ Complete job
```

Final status: **`gh run list`**

```
completed  success  Lab 2: Add basic CI workflow with GitHub Actions  CI  main  push  35686529676  22s  2026-09-22T04:20:45Z
```

Both matrix jobs (Node 18.x and Node 20.x) completed successfully in ~22
seconds total, confirming the pipeline builds, lints, and tests the code
automatically on every commit.

> **Screenshot note:** A live browser screenshot of the GitHub Actions run
> page (`https://github.com/Arpan7125/Lab2_CI_Workflow/actions/runs/35686529676`)
> was intended for this report, but the Claude-in-Chrome browser extension
> was not connected during this session, so it could not be captured
> automatically. The `gh run watch`/`gh run list` output above is the
> command-line equivalent of that same run, pulled directly from the GitHub
> Actions API. You can view the green ✓ check live at the run URL above, or
> at **Repo → Actions tab**, and paste a screenshot into this report if a
> visual is still needed.

---

## 6. Local Verification Output

```
npm run lint && npm test

> lab2-ci-workflow@1.0.0 lint
> eslint src tests

> lab2-ci-workflow@1.0.0 test
> jest

PASS tests/calculator.test.js
  √ adds 2 + 3 to equal 5 (2 ms)
  √ subtracts 5 - 2 to equal 3 (1 ms)
  √ multiplies 4 * 3 to equal 12
  √ divides 10 / 2 to equal 5
  √ throws when dividing by zero (9 ms)

Test Suites: 1 passed, 1 total
Tests:       5 passed, 5 total
Snapshots:   0 total
Time:        2.038 s
```

---

## 7. Result

- ✅ CI workflow file (`ci.yml`) authored and committed to the repository.
- ✅ Repository created and pushed to GitHub (`Arpan7125/Lab2_CI_Workflow`).
- ✅ Workflow **triggered automatically** on `git push` — no manual dispatch
  used.
- ✅ Both lint and test stages passed across two Node.js versions (18.x,
  20.x).
- ✅ Total run time: 22 seconds, confirming a fast feedback loop.

---

## 8. Conclusion

This lab demonstrates a complete, minimal CI pipeline: a commit pushed to
GitHub automatically triggers a GitHub Actions workflow that installs
dependencies, lints the codebase, and runs the automated test suite across
multiple Node.js versions — all without any manual step after `git push`.
This is the foundational pattern used in larger CI/CD systems, which extend
this same trigger-on-commit model with additional stages such as build
artifact generation, containerization, staging deployment, and production
release gating.
