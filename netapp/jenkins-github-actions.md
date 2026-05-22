# Jenkins + GitHub Actions

---

## Jenkins

CI/CD automation server. A **pipeline** is code that defines the build → test → deploy workflow.

### Jenkinsfile

- No file extension
- Checked into source repo at the root — Jenkins auto-detects it
- **Declarative** (choose this) vs Scripted pipeline
  - Declarative: structured, readable, easier error tracking, standard CI/CD
  - Use `script {}` blocks when you need Groovy logic inside Declarative

### Full Structure

```groovy
pipeline {
    agent any                          // where to run (any available agent)
    // agent { label 'linux' }         // or specific agent

    environment {                      // variables available to all stages
        APP_ENV = 'test'
        API_TOKEN = credentials('my-api-token')  // from Jenkins credentials store
    }

    parameters {                       // inputs — can be passed when triggering
        string(name: 'VERSION', defaultValue: '1.0')
    }

    triggers {                         // when to run
        cron('H 2 * * *')              // nightly at 2 AM
        // pollSCM('H/5 * * * *')      // poll SCM every 5 min
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        retry(3)
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build') {
            steps { sh 'make build' }
        }
        stage('Test') {
            steps {
                sh 'pytest tests/ --junitxml=results.xml'
                junit 'results.xml'    // records failures as UNSTABLE, not FAILURE
            }
        }
        stage('Deploy') {
            steps { sh 'terraform apply -auto-approve' }
        }
    }

    post {
        always  { sh 'terraform destroy -auto-approve' }  // cleanup even on failure
        success { echo 'Pipeline passed' }
        failure { emailext(to: 'team@example.com', subject: 'Pipeline failed') }
        unstable {}   // tests failed but pipeline completed
        changed  {}   // result changed from last run
    }
}
```

### Parallel Stages

Run independent test suites at the same time — key for reducing CI runtime:

```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests')        { steps { sh 'pytest tests/unit/' } }
        stage('Integration Tests') { steps { sh 'pytest tests/integration/' } }
        stage('Lint')              { steps { sh 'flake8 .' } }
    }
}
```

### Variables

**Built-in:** `JOB_NAME`, `WORKSPACE`, `BUILD_NUMBER`, `BUILD_URL`  
**Custom:** defined in `environment {}` block or assigned in `script {}`.

### Credentials

Never store secrets directly in the Jenkinsfile. Use the Jenkins credentials store — Jenkins auto-masks them in logs:

```groovy
environment {
    API_TOKEN = credentials('my-api-token')  // referenced by ID
}
```

### Error Handling

```groovy
// Option 1 — let stage fail, handle in post{}
// Option 2 — try/catch inside script block
script {
    try {
        sh 'run-tests.sh'
    } catch (e) {
        currentBuild.result = 'UNSTABLE'
    }
}
// Option 3 — || true so shell always exits 0, junit records failures without stopping pipeline
sh 'pytest tests/ --junitxml=results.xml || true'
junit 'results.xml'
```

### script {} Block

When you need Groovy logic (loops, conditionals, variables) inside Declarative:

```groovy
stage('Dynamic') {
    steps {
        script {
            def envs = ['dev', 'staging', 'prod']
            for (env in envs) {
                sh "deploy.sh ${env}"
            }
        }
    }
}
```

### Tools Block

Manage specific tool versions per pipeline — useful when different services need different Python/Java/Maven versions:

```groovy
tools {
    python 'Python-3.11'
    maven 'Maven-3.9'
}
```

### External Groovy Scripts

As Jenkinsfiles grow, pull logic into separate `.groovy` files and load dynamically:

```groovy
script {
    def helper = load 'scripts/helper.groovy'
    helper.buildApp()
}
```

`helper.groovy` must end with `return this` — when Jenkins executes the file, `load` returns whatever the file returns. Without `return this`, load returns `null` and you can't call any methods.

```groovy
// helper.groovy
def buildApp() {
    sh 'make build'
}
return this   // required
```

### Key Interview Takeaways

- **Jenkinsfile in SCM** — always. Never define pipelines in the UI. Version control = audit trail + code review + reproducibility.
- **Declarative over Scripted** — cleaner, more maintainable. Use `script {}` for Groovy logic when needed.
- **`post { always }` for teardown** — put `terraform destroy` in `post { always }` so infra is torn down even if the pipeline fails.
- **`junit` for test results** — records failures as UNSTABLE, not FAILURE. Pipeline continues so you can collect results and teardown cleanly.
- **Parallel stages** — independent test suites should run in parallel, not sequentially.
- **Never hardcode credentials** — always use Jenkins credentials store, referenced by ID.

---

## GitHub Actions

CI/CD platform built directly into GitHub. No separate server to install. Triggered by events in your repo (push, PR, schedule, etc.). Everything is YAML files committed to your repo under `.github/workflows/`.

**Jenkins vs GitHub Actions:** Jenkins for on-prem/self-hosted pipelines. GitHub Actions for anything git-native.

### Architecture

```
GitHub Event (push / PR / schedule)
    └── triggers Workflow (.github/workflows/my-workflow.yml)
              └── contains one or more Jobs
                        └── each Job runs on a Runner (ubuntu-latest, windows-latest, etc.)
                                  └── each Job contains Steps
                                            ├── run: <shell command>
                                            └── uses: <reusable Action>
```

**Jobs run in parallel by default.** Use `needs:` to enforce ordering.

### Basic Workflow

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 2 * * *'    # nightly at 2 AM

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install -r requirements.txt
      - run: pytest tests/

  deploy:
    needs: test              # runs only after test job passes
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```

### Job Matrix — Run Across Many Configurations

Creates a job for every combination:

```yaml
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    python: ['3.9', '3.10', '3.11']
    # 3 × 3 = 9 jobs running in parallel
```

### Passing Data Between Jobs

```yaml
jobs:
  build:
    outputs:
      image_tag: ${{ steps.tag.outputs.tag }}
    steps:
      - id: tag
        run: echo "tag=v1.0.$(date +%s)" >> $GITHUB_OUTPUT   # modern way

  deploy:
    needs: build
    steps:
      - run: echo "Deploying ${{ needs.build.outputs.image_tag }}"
```

> Modern way to set outputs: `echo "key=value" >> $GITHUB_OUTPUT`

### Key Interview Takeaways

- **No server to manage** — Actions is fully managed by GitHub.
- **Parallel by default** — use `needs:` to serialize jobs.
- **Matrix builds** — run the same job across OS/version combinations with minimal config.
- **Reusable Actions** — `uses:` steps call community or internal actions (checkout, setup-python, configure-aws-credentials, etc.).
- **Secrets** — stored in GitHub repo/org settings, referenced as `${{ secrets.MY_SECRET }}`, masked in logs.
