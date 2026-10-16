# Kinesis Network for Cursor

Deploy an app, find out why it isn't running, and see what it costs, without leaving the editor.

[Kinesis](https://kinesis.network) runs containerized apps and AI models on serverless CPU capacity, on your organization's own grids of dedicated servers (including GPUs), or on spot capacity from the Kinesis marketplace. This plugin connects Cursor's agent to the Kinesis MCP server, so the agent can do the work you would otherwise click through the dashboard for.

## What you can ask

- "Deploy this repository to Kinesis as `my-api`, serving HTTP on port 8080."
- "Why isn't `api-prod` running? Check its status and logs."
- "Roll `worker` back to its previous build."
- "What did our organization spend on Kinesis this month?"
- "Estimate a dedicated server with the cheapest GPU and a 100 GB disk. Don't order it."

The agent can:

- Deploy from a container image (public or in a private registry), from a GitHub repository (Kinesis builds it with your Dockerfile), or from a template such as an Ollama chat or a vLLM server.
- Troubleshoot: read an app's status, build output, container logs and server logs, and check whether any server can run it.
- Start, stop and restart apps, change their settings and environment variables, rebuild from another branch or roll back, and connect a custom domain.
- Add dedicated servers to a grid, with a cost estimate first, or attach a machine of your own.
- Show your organization's costs, usage and invoices.
- Manage members, roles and sharing, and read the audit log.

## Getting started

1. Install the plugin from the Cursor Marketplace.
2. The first time the agent uses Kinesis, Cursor asks you to sign in. Sign in to your Kinesis account and approve the connection. You can limit it to some of your organizations.
3. Ask away.

You need a [Kinesis account](https://portal.kinesis.network/signup). Deploying apps and adding servers use your organization's Kinesis credits. To deploy from GitHub, connect your GitHub account to Kinesis first; the agent gives you the link when it's needed.

## Skills

- **deploy-to-kinesis**: deploys the repository you have open, or redeploys it. The agent checks what is pushed, the Dockerfile, the port and the environment variables, connects the app to your GitHub repository, follows the build and gives you the address.

## How it behaves

- Every action runs with your own Kinesis role and permissions.
- Each area has a tool that only reads and, where the area has changes to make, a separate `_write` tool for them, so you can let the agent run the read tools freely and approve the others.
- Permanent actions, such as deleting an app or removing a server, need an explicit confirmation.
- Secrets are write-only: secret environment variables and registry passwords can be set but are never read back.
- App logs are your app's own output, as it wrote them, so they can hold anything your app prints.

## Options

The plugin connects to `https://mcp.kinesis.network/mcp`, which lists 18 tools: per area (apps, servers, billing, ...) a read tool and a write tool that take an action and its parameters. Query parameters change the listing:

| Parameter | Effect |
| --- | --- |
| `?readonly=true` | Only the tools that read |
| `?groups=apps,projects` | Only some areas: `account`, `projects`, `apps`, `github`, `sharing`, `grids`, `servers`, `catalog`, `billing` |
| `?mode=all` | One tool per action, about 95 in all. Cursor works best with fewer than 40 tools across every server, so pair it with `groups` |

To use one, add the server yourself in Cursor's MCP settings with the changed URL instead of installing the plugin.

### API key instead of signing in

Create a key under **Account settings** in the [Kinesis dashboard](https://portal.kinesis.network/dashboard/account) and add the server with a header:

```json
{
  "mcpServers": {
    "kinesis": {
      "url": "https://mcp.kinesis.network/mcp",
      "headers": { "Authorization": "Bearer ${env:KINESIS_API_KEY}" }
    }
  }
}
```

## Help

- Documentation: https://docs.kinesis.network/mcp
- Support: https://docs.kinesis.network/support

## License

[MIT](LICENSE)
