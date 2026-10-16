# Kinesis Network for Cursor

Run Docker images, GitHub repositories and AI models on Kinesis without leaving the editor: serverless, or on a Grid of your own servers.

[Kinesis](https://kinesis.network) runs containers on two kinds of capacity:

- **Serverless**: Kinesis-managed CPU machines with nothing to set up, billed per millisecond of use.
- **Grids**: pools of servers your organization owns, for GPUs and dedicated capacity. One Grid can mix dedicated GPU servers, AWS and Azure instances, machines of your own and spot machines from the Kinesis marketplace. Kinesis places your apps across them, and can hibernate a server when nothing runs on it and wake it when an app needs it.

This plugin connects Cursor's agent to Kinesis, so it can ship what you build, set up the capacity it runs on and keep an eye on the bill.

## What you can ask

- "Deploy this repository to Kinesis."
- "Run Qwen3 30B with vLLM on our GPU Grid."
- "Order an H100 server for the `inference` Grid. Show me the cost first."
- "Give me the command to add my own Linux server to our Grid."
- "Roll `api` back to its previous build."
- "What did our organization spend on Kinesis this month?"

The agent can:

- Deploy a Docker image (public or from a private registry), a GitHub repository (built from its Dockerfile and rebuilt on every push), or a ready-made template: Ollama and vLLM model servers, JupyterLab, n8n, Grafana and more.
- Give apps GPUs, persistent volumes, environment variables and secrets, HTTPS and custom domains.
- Build Grids: order dedicated servers with a cost estimate first, attach your own machines, and choose which servers apps prefer.
- Run apps day to day: start, stop and scale them, roll back a build, and read their status and logs when something goes wrong.
- Show your organization's costs, usage and invoices, and manage members, roles and sharing.

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
