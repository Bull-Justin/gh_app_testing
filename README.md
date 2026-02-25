# gh-app-testing

A dummy Python Flask app with GitHub Actions workflows demonstrating environment-gated deployments approved via a GitHub App.

## App

A minimal Flask app (`app/app.py`) with two endpoints:

| Endpoint | Response |
|----------|----------|
| `GET /` | `{"status": "ok", "app": "gh-app-testing"}` |
| `GET /health` | `{"healthy": true}` |

## Workflows

### `deploy.yml` — CI/CD Pipeline

Triggered on push to `main` or manually via `workflow_dispatch`.

1. **test** — installs dependencies and runs pytest
2. **deploy** — runs after tests pass, targets the `production` environment

The `deploy` job will pause for manual approval if the `production` environment has required reviewers configured (see setup below).

### `approve-deployment.yml` — GitHub App Approval Bot

Triggered manually via `workflow_dispatch`. Uses a GitHub App token to programmatically approve a pending deployment.

**Inputs:**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `run_id` | Yes | — | Run ID of the deploy workflow waiting for approval |
| `environment` | No | `production` | Environment to approve |
| `comment` | No | `Approved by GitHub App bot` | Approval comment |

---

## Setup

### 1. Create a GitHub App

1. Go to **Settings → Developer settings → GitHub Apps → New GitHub App**
2. Set a name (e.g. `my-deploy-approver`)
3. Uncheck **Webhook active** (not needed)
4. Under **Repository permissions**, set **Actions** to **Read and write**
5. Click **Create GitHub App**
6. Note the **App ID** shown on the app's settings page
7. Scroll to **Private keys** and click **Generate a private key** — save the `.pem` file

### 2. Install the App on the Repository

1. On your GitHub App's settings page, click **Install App**
2. Select your account and choose the repository

### 3. Add Repository Secrets

In your repo: **Settings → Secrets and variables → Actions → New repository secret**

| Secret name | Value |
|-------------|-------|
| `GH_APP_ID` | The numeric App ID from step 1 |
| `GH_APP_PRIVATE_KEY` | The full contents of the `.pem` file from step 1 |

### 4. Create the `production` Environment with Required Reviewers

1. Go to **Settings → Environments → New environment**
2. Name it `production`
3. Enable **Required reviewers** and add yourself (or the GitHub App) as a reviewer
4. Save

> **Note:** Adding the GitHub App as a reviewer is what allows `approve-deployment.yml` to approve on its behalf. You can also add yourself as a reviewer and use the workflow as an alternative approval path.

---

## Usage

### Triggering a deployment

Push to `main` or manually run the **Deploy** workflow. The `deploy` job will pause at the `production` environment gate.

### Approving via GitHub App

1. Find the **Run ID** of the paused deploy workflow (visible in the URL on the Actions run page)
2. Go to **Actions → Approve Deployment → Run workflow**
3. Enter the Run ID and click **Run workflow**

The bot will generate a short-lived GitHub App token, find the pending deployment, and approve it — unblocking the deploy job.
