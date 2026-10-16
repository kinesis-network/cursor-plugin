---
name: deploy-to-kinesis
description: Deploy the repository open in Cursor to Kinesis, or redeploy it after changes. Use when the user asks to deploy, ship, host or put this project online on Kinesis.
---

# Deploy this repository to Kinesis

Kinesis builds a repository app from GitHub with its Dockerfile, so what gets deployed is what is pushed to the branch, not the files on disk.

The names below are Kinesis actions. With the plugin's default connection, call each through its area's tool (`account`, `projects`, `projects_write`, `apps`, `apps_write`) with `action` and `params`; `browse_github`, `read_docs` and `describe_action` are tools of their own. With `?mode=all`, every action is a tool. `read_docs` with topic `deploy` has every config field.

## 1. Know what to deploy

- `git remote get-url origin` gives `owner/name`, and the current branch is the one to deploy unless the user says otherwise. Only repositories on GitHub can be built.
- `git status -sb` shows uncommitted changes and commits not yet pushed. Tell the user about them: they won't be in the build.

## 2. Dockerfile, port and environment

- Use the repository's Dockerfile if it has one; its `EXPOSE` lines give the ports.
- Without one, write a Dockerfile for the stack (package.json, requirements.txt, go.mod, ...). Show it to the user, then either commit and push it with their go-ahead or pass it as `dockerfileContent`.
- The server must listen on `0.0.0.0`, not `localhost` or `127.0.0.1`, on the configured port. Otherwise the container runs but never answers.

### Ports

Each entry in `ports` is `{ port, protocol }`. `port` is where the server listens inside the container. `protocol` says how the internet reaches it. Choose it by what clients speak, not by the port number:

| The port serves | `protocol` |
| --- | --- |
| A website or HTTP API: almost every app | `https443_load_balancer` |
| Another HTTP port on the same app | `https_load_balancer` |
| gRPC | `grpcs_load_balancer` |
| Other TCP, such as a database | `tcp_load_balancer` |
| UDP | `udp` |

`https443_load_balancer` gives the app `https://<its domain>`. Kinesis handles the certificate, so the app itself serves plain HTTP. Use it for the main HTTP port even when the server listens on port 80 or has no TLS of its own. `http80_load_balancer` is unencrypted HTTP on port 80: use it only when the user asks for that. An app has at most one `https443_load_balancer` port.
- Look for environment variables the app needs at startup (`.env.example`, config code) and ask the user for their values. Mark keys, tokens and passwords `secret: true`. Don't copy values from a local `.env` without asking.
- If Docker is installed, `docker build .` catches most build errors before the remote build does.

## 3. Where it goes

- `list_orgs`, then `list_projects`. Ask which project to use, or `create_project`.
- `browse_github` with `kind: accounts`: take the `installationId` of the connection that covers the repository's owner. If none does, give the user the `connectUrl` and wait for them to connect it.

## 4. Create the app

Running an app spends the organization's Kinesis credits, so confirm the name, project, size and ports with the user first. Then `create_app`:

```json
{
  "projectId": "<from list_projects>",
  "name": "my-api",
  "source": { "kind": "github", "installationId": 12345678, "repo": "owner/name", "branch": "main" },
  "config": { "ports": [{ "port": 3000, "protocol": "https443_load_balancer" }] }
}
```

Add `env`, `instances` or `hardware` only as needed; the defaults are 1 GiB RAM, 1 to 3 instances and no GPU. Pass an `idempotencyKey`, and reuse it if the call times out, so a retry can't create a second app.

## 5. Follow the build

- The result carries `buildId`. Poll `get_build` every `pollAfterSec` (a build takes one to three minutes) until it has `succeeded` or `failed`.
- On `failed`: read `get_build_logs`, fix the cause in the repository and, with the user's go-ahead, commit and push. `autoDeploy` (on by default) rebuilds on every push to the branch; `list_builds` shows the new build.
- On `succeeded`: poll `get_app_status` until the app is `running`, then give the user the address from `endpoints`.

## 6. If it doesn't run

`get_app_status` shows recent errors, `get_app_logs` the container's own output, and `check_placement` whether any server can run the app. `read_docs` with topic `troubleshoot` covers the rest.

## Redeploying

With `autoDeploy` on, pushing to the branch redeploys. Otherwise `trigger_build` and follow the build as above. `rollback_build` returns to an earlier build without rebuilding.
