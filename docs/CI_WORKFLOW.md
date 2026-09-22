# Lab 2 — Basic CI Workflow Triggered by Commits (GitHub Actions)

**Course:** DevOps Lab · **Repository:** [Arpan7125/Lab2_CI_Workflow](https://github.com/Arpan7125/Lab2_CI_Workflow) · **CI platform:** GitHub Actions

---

## 1. Objective

Design and implement a basic Continuous Integration pipeline that runs automatically on every
commit pushed to the repository. The pipeline must fetch the code, set up the runtime, install
dependencies, check code style, and run the automated test suite — failing loudly if any step
breaks.

## 2. Why GitHub Actions (and not GitLab CI)

| Criterion | GitHub Actions | GitLab CI |
|---|---|---|
| Hosting of this repo | Already on GitHub | Would need a mirror |
| Runners | Free hosted `ubuntu-latest` | Free shared runners, but extra setup |
| Config file | `.github/workflows/ci.yml` | `.gitlab-ci.yml` |
| Setup effort | Commit one YAML file — nothing to enable | Requires project/runner configuration |

Because the project already lives on GitHub, Actions gives a commit-triggered pipeline with a
single committed file and zero external configuration. An equivalent `.gitlab-ci.yml` is given in
Appendix A for comparison.

## 3. Project under test

A small CommonJS calculator module with a Jest test suite — deliberately simple, so the focus stays
on the pipeline rather than the application.

```
Lab2_CI_Workflow/
├── .github/workflows/ci.yml     # the CI pipeline
├── .eslintrc.json               # lint rules (eslint:recommended)
├── package.json                 # npm scripts: "lint", "test"
├── src/calculator.js            # add, subtract, multiply, divide
├── tests/calculator.test.js     # 5 Jest test cases
└── docs/CI_WORKFLOW.md          # this document
```

`divide()` throws on division by zero, and one test asserts that behaviour — so the suite covers a
happy path and an error path.

## 4. Pipeline design

### 4.1 Trigger

```yaml
on:
  push:
    branches: ["**"]        # every commit, on every branch
  pull_request:
    branches: ["main"]      # and every PR aimed at main
```

`push` with the `"**"` glob is what satisfies the "triggered by commits" requirement: any commit
pushed to any branch starts a run. The `pull_request` trigger adds a second safety net so that a
merge into `main` is verified before it lands.

### 4.2 Job and matrix

```yaml
jobs:
  build-and-test:
    name: Build & Test (Node ${{ matrix.node-version }})
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18.x, 20.x]
```

One job, executed twice in parallel — once on Node 18 and once on Node 20. This catches
runtime-version-specific breakage, which is the cheapest useful form of compatibility testing.

### 4.3 Steps

| # | Step | Action / command | Purpose |
|---|------|------------------|---------|
| 1 | Checkout repository | `actions/checkout@v4` | Clone the commit that triggered the run |
| 2 | Set up Node.js | `actions/setup-node@v4` (`cache: npm`) | Install the matrix Node version; cache `~/.npm` |
| 3 | Install dependencies | `npm install` | Install ESLint and Jest |
| 4 | Run linter | `npm run lint` | Static analysis — fails the job on any lint error |
| 5 | Run tests | `npm test` | Execute the Jest suite |

Steps run in order and any non-zero exit code aborts the job and marks the run **failed**, so a
broken commit is visible on GitHub within seconds.

### 4.4 Full workflow file

```yaml
name: CI

# Trigger the workflow on every commit pushed to any branch,
# and on every pull request targeting main.
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

## 5. Execution evidence (screenshots)

### 5.1 The workflow file committed to the repository

Committing `.github/workflows/ci.yml` is the entire "installation" — GitHub discovers the workflow
automatically from that path.

![Workflow file on GitHub](screenshots/04-workflow-file.png)

### 5.2 Runs triggered by commits

Each row is one run, labelled with the commit that produced it (`CI #2: Commit e205584 pushed by
Arpan7125`). No run was started manually — every one was triggered by a `git push`.

![Actions tab showing workflow runs](screenshots/01-actions-runs-list.png)

### 5.3 Run summary — both matrix jobs green

The two matrix jobs (Node 18.x and Node 20.x) ran in parallel and both succeeded.

![Run summary with matrix jobs](screenshots/02-run-summary-matrix-jobs.png)

### 5.4 Step-by-step job detail

Every step of the Node 18.x job — checkout, Node setup, install, lint, test — completed with a
green check.

![Job detail showing all steps](screenshots/03-job-steps-node18.png)

### 5.5 CI log output — lint and tests

The actual log of the `Run linter` and `Run tests` steps from the hosted runner: ESLint produced no
output (no violations) and Jest reported 5 of 5 tests passing.

![CI log for lint and test steps](screenshots/05-ci-log-node18.png)

### 5.6 Same commands run locally

Running the identical npm scripts on the development machine gives the same result, confirming the
pipeline reproduces local behaviour rather than depending on the CI environment.

![Local lint and test output](screenshots/06-local-verification.png)

## 6. Results

| Run | Trigger commit | Branch | Event | Node 18.x | Node 20.x | Duration |
|-----|----------------|--------|-------|-----------|-----------|----------|
| CI #1 | `394a450` — Lab 2: Add basic CI workflow with GitHub Actions | `main` | push | ✅ success | ✅ success | 22 s |
| CI #2 | `e205584` — Add REPORT.md documenting Lab 2 CI workflow execution | `main` | push | ✅ success | ✅ success | 15 s |

Tests executed per job: **5 passed / 5 total**. Lint violations: **0**.

Run #2 finished faster than run #1 because `actions/setup-node`'s npm cache was already warm.

## 7. Reproducing the lab

```bash
git clone https://github.com/Arpan7125/Lab2_CI_Workflow.git
cd Lab2_CI_Workflow
npm install
npm run lint && npm test      # same checks the pipeline runs
```

To see the pipeline fire, make any commit and push it:

```bash
git commit --allow-empty -m "Trigger CI"
git push
```

Then open the **Actions** tab of the repository.

To verify the pipeline actually catches breakage, change `add(a, b)` in `src/calculator.js` to
return `a - b`, commit, and push — the `Run tests` step fails and the run is marked red.

## 8. Observations

- **Feedback is fast.** A full run (two Node versions, install, lint, test) finished in 15–22
  seconds, so a broken commit is reported almost immediately.
- **Step ordering matters.** Linting before testing means cheap failures surface first.
- **Caching helps even at this scale.** `cache: "npm"` in `setup-node` shaved roughly 30 % off the
  second run.
- **`npm install` vs `npm ci`.** `npm install` is used here for simplicity; `npm ci` is stricter
  (it installs exactly what `package-lock.json` pins and fails if the lockfile is out of sync) and
  is the better choice for a real project.
- **Deprecation notices.** The runs report one warning/notice annotation from transitive npm
  dependencies of ESLint 8. It does not fail the build, but it is a reminder that pinned tool
  versions age.

## 9. Learning outcomes

1. A CI pipeline is just a versioned file in the repository — the pipeline evolves with the code.
2. Event triggers (`push`, `pull_request`) decide *when* automation runs; a matrix decides *how
   many ways* it runs.
3. Exit codes are the contract between a build step and the CI system.
4. Running the same commands locally and in CI is what makes the pipeline trustworthy.

---

## Appendix A — Equivalent GitLab CI configuration

For reference, the same pipeline expressed as `.gitlab-ci.yml`:

```yaml
stages:
  - test

.node-template:
  stage: test
  script:
    - npm install
    - npm run lint
    - npm test

test:node18:
  extends: .node-template
  image: node:18

test:node20:
  extends: .node-template
  image: node:20
```

GitLab CI runs on every pushed commit by default, so no explicit trigger block is needed; the
equivalent of the Actions matrix is written out as two jobs sharing a template.
