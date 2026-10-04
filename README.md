# Flixter Plugin for Grok Build

[Flixter](https://flixter.ai) is an AI film and series production studio. This plugin connects Grok Build to Flixter's hosted MCP server so you can read and organise productions in conversation — cast, locations, props, costumes, episodes, scenes, and shots — and queue render work for human approval without leaving the chat.

## Installation

In Grok Build, open `/plugin`, search for **Flixter**, and install the plugin.

On first connection, Grok opens Flixter's OAuth 2.1 (PKCE) authorization flow in the browser. Sign in with a Flixter account that belongs to at least one workspace. Create an account at [flixter.ai](https://flixter.ai) before connecting if you do not have one yet.

## MCP endpoint

The plugin connects only to Flixter's hosted MCP endpoint:

`https://flixter.ai/api/mcp`

Protocol: streamable HTTP. The client discovers how to authorize on its own — the URL is the only thing configured. OAuth issuer: `https://api.flixter.ai`.

Full connector documentation: [flixter.ai/connector.html](https://flixter.ai/connector.html).

## Authorization and scopes

Authorization is OAuth 2.1 with PKCE. Clients that have never seen Flixter register themselves (RFC 7591), send you to Flixter to sign in, and show a consent screen. Scopes:

| Scope | What it allows |
|---|---|
| `studio:read` | See productions: scripts, cast, locations, props, scenes, shots, and media already rendered |
| `studio:write` | Create and change productions: scenes, shots, cast, locations, props, and related objects |
| `studio:generate` | Queue rendering work (spends workspace credits when approved and run) |
| `offline_access` | Refresh tokens so the connection can stay authorized between sessions |

The grant belongs to you, not to one workspace: an authorized client can act in any workspace you are a member of. Revoke a connection from the workspace's installed applications in Flixter.

## Workspaces

A workspace is a separate studio. Every tool takes a `workspace` argument when you belong to more than one. Call `workspace_list` first to see which workspaces this connection can reach.

## Render work and approval

Rendering costs credits, so on a connector the decision to spend stays with a person. The connector does **not** expose one-call generators. Instead, use the `*_plan_*` tools (for example, `scene_plan_render`, `scene_plan_assemble`, `episode_plan_assemble`, `production_plan_assemble`). They register a priced batch and stop there.

The batch appears in Flixter's task manager (studio top bar). You approve it there; then the work begins and results land on the rows. Nothing renders and nothing is billed until a person approves the batch. Asking the connector to run a plan tool again registers a second batch rather than starting the first.

## Skills

| Skill | What it does |
|---|---|
| `flixter-studio` | When and how to use Flixter tools: `workspace_list` first, prefer `plan_*` for renders, approve batches in Flixter's task manager |

## Example prompts

```text
List my Flixter workspaces and open the Soft Open production.
```

```text
Ingest this story into a new Flixter production and break it into scenes.
```

```text
Plan the renders for scene 2 — I will approve the batch in Flixter's task manager.
```

## Resources

- [Flixter](https://flixter.ai)
- [Connector documentation](https://flixter.ai/connector.html)
- [Privacy Policy](https://flixter.ai/privacy.html)
- [Terms of Service](https://flixter.ai/terms.html)

## License

MIT
