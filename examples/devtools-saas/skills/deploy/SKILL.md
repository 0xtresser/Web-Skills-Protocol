---
name: deploy
description: >
  Deploy web applications to DevTools Cloud from Git repositories.
  Use when the user wants to deploy, ship, publish, or launch a web app.
  Supports Node.js, Python, Go, and Rust with automatic build detection.
  Handles zero-downtime deployments with automatic rollback on failure.
  Do NOT use for domain management or log viewing.
version: 2.1.0
auth: oauth2
rate_limit:
  agent: 3/minute
  api: 10/minute
base_url: https://api.devtools.cloud/v2
---

# Deploy Application

## Authentication

Include an OAuth 2.0 access token in every request:

```
Authorization: Bearer ACCESS_TOKEN
```

Obtain tokens via the OAuth 2.0 flow at https://devtools.cloud/docs/oauth.

## Prerequisites

- An OAuth 2.0 access token
- A Git repository URL (GitHub, GitLab, or Bitbucket)

## Workflow

Complete the following steps in order.

### Step 1: Create Application (if new)

Skip this step if the app already exists. Use GET /apps to list existing apps.

POST /apps

```json
{
  "name": "my-app",
  "git_url": "https://github.com/user/repo",
  "branch": "main",
  "runtime": "nodejs"
}
```

| Field   | Type   | Required | Description                                |
|---------|--------|----------|--------------------------------------------|
| name    | string | yes      | App name (lowercase, hyphens OK)           |
| git_url | string | yes      | Git repository URL                         |
| branch  | string | no       | Branch to deploy (default: main)           |
| runtime | string | no       | `nodejs`, `python`, `go`, `rust` (auto-detected if omitted) |

Response (201 Created):

```json
{
  "id": "app_k8x7m",
  "name": "my-app",
  "status": "created",
  "git_url": "https://github.com/user/repo",
  "url": "https://my-app.devtools.cloud",
  "created_at": "2026-03-04T10:00:00Z"
}
```

### Step 2: Trigger Deployment

POST /apps/{app_id}/deployments

```json
{
  "branch": "main",
  "env_vars": {
    "NODE_ENV": "production",
    "DATABASE_URL": "postgres://..."
  }
}
```

| Field    | Type   | Required | Description                            |
|----------|--------|----------|----------------------------------------|
| branch   | string | no       | Branch to deploy (default: app's branch)|
| env_vars | object | no       | Environment variables (key-value pairs)|
| force    | bool   | no       | Force deploy even if no changes (default: false) |

Response (202 Accepted):

```json
{
  "id": "deploy_9f3n",
  "app_id": "app_k8x7m",
  "status": "building",
  "branch": "main",
  "commit": "a1b2c3d",
  "started_at": "2026-03-04T10:30:00Z"
}
```

### Step 3: Check Deployment Status

Poll until status is `live` or `failed`:

GET /apps/{app_id}/deployments/{deploy_id}

Response (200 OK):

```json
{
  "id": "deploy_9f3n",
  "app_id": "app_k8x7m",
  "status": "live",
  "branch": "main",
  "commit": "a1b2c3d",
  "url": "https://my-app.devtools.cloud",
  "build_time_seconds": 47,
  "started_at": "2026-03-04T10:30:00Z",
  "finished_at": "2026-03-04T10:30:47Z"
}
```

Status flow: `building` → `deploying` → `live` | `failed`

Poll every 5 seconds. Typical builds complete in 30-120 seconds.

If status is `failed`, check the `error` field:

```json
{
  "id": "deploy_9f3n",
  "status": "failed",
  "error": "Build failed: missing package.json",
  "build_log_url": "https://api.devtools.cloud/v2/apps/app_k8x7m/deployments/deploy_9f3n/logs"
}
```

### Step 4: View Logs (if needed)

GET /apps/{app_id}/logs?lines=50&level=error

| Parameter | Type   | Required | Description                              |
|-----------|--------|----------|------------------------------------------|
| lines     | number | no       | Number of lines to return (default: 100) |
| level     | string | no       | Filter: `error`, `warn`, `info`, `debug` |
| since     | string | no       | ISO 8601 timestamp to start from         |

Response (200 OK):

```json
{
  "logs": [
    {
      "timestamp": "2026-03-04T10:31:00Z",
      "level": "info",
      "message": "Server listening on port 3000"
    },
    {
      "timestamp": "2026-03-04T10:31:05Z",
      "level": "info",
      "message": "Connected to database"
    }
  ]
}
```

## Error Handling

| Code | Meaning                | Action                                   |
|------|------------------------|------------------------------------------|
| 400  | Invalid request        | Check required fields                    |
| 401  | Auth failed            | Refresh OAuth token                      |
| 404  | App not found          | Verify app_id                            |
| 409  | Deployment in progress | Wait for current deploy to finish        |
| 422  | Invalid Git URL        | Verify repository URL and access         |
| 429  | Rate limited           | Wait, retry after `Retry-After` header   |
| 500  | Server error           | Retry with exponential backoff           |

## Notes

- **Atomic deployments:** If the build fails, the previous version stays live. No downtime.
- **Environment variables** set in Step 2 persist across deployments. To remove one, set its value to `null`.
- **Build detection** is automatic — the platform reads `package.json`, `requirements.txt`, `go.mod`, or `Cargo.toml` to determine the build command.
- **Rollback:** Send POST /apps/{app_id}/deployments with `{"rollback_to": "deploy_id"}` to instantly revert.
- **Deploy hooks:** Configure webhooks at https://devtools.cloud/settings/hooks to trigger deploys on Git push.
