# Flixter Plugin for Grok Build

[Flixter](https://flixter.ai) is an AI film and series production studio. This plugin connects Grok Build to Flixter's hosted MCP server so you can read and organise productions in conversation — cast, locations, props, costumes, episodes, scenes, and shots — manage takes, lock character voices, tidy a scene's cut order and background sound, run generators and render batches directly without leaving the chat.

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
| `studio:generate` | Run one-call generators immediately (spends workspace credits) and queue rendering batches (spends when approved and run) |
| `offline_access` | Refresh tokens so the connection can stay authorized between sessions |

The grant belongs to you, not to one workspace: an authorized client can act in any workspace you are a member of. Revoke a connection from the workspace's installed applications in Flixter.

## Workspaces

A workspace is a separate studio. Every tool takes a `workspace` argument when you belong to more than one. Call `workspace_list` first to see which workspaces this connection can reach.

## Render work and approval

Rendering costs credits. The connector exposes both paths:

**One-call generators** (portraits, plates, models, frames, video edits/extends) run and spend workspace credits **immediately**. Grok runs them directly without asking for spend confirmation and may briefly report the credits spent; for a whole scene it prefers a `*_plan_*` batch over looping generators. Many generators are async — results land on the row later; re-read the row or use `view_image` / `view_video` to see them.

**`*_plan_*` batch tools** (for example `scene_plan_render`, `scene_plan_assemble`, `episode_plan_assemble`, `production_plan_assemble`) are preferred for whole scenes, episodes, or films. In Flixter's own chat a batch waits in the task manager (studio top bar) for approval. Over this MCP connection, Flixter's tool descriptions say the call also approves and deploys the batch on the OAuth grant, so Grok calls it directly and reports the credits from the result. Running a plan tool again creates a second batch. Results are async, so Grok re-reads the rows.

## Ideas and writing

**Pitches and choices**: Grok can show story ideas as pitch cards and ask multiple-choice questions as clickable cards in Flixter, then wait for your pick.

**Scripts**: plan a production's running time scene by scene, then write the whole script, with every scene and its shots, in one parallel pass. Grok confirms before rewriting an existing script or replacing a scene's shots.

## Takes, voices, and sound

**Takes** — list a shot's takes, choose which one the shot uses, and tag them Circled / Hold / NG (free). Crop a video take (permanent, with an undo), set its segments, or delete a take; Grok confirms with you before cropping or deleting.

**Voices** — lock a character's voice from a take so later renders use it, unlock it to go back to Omni's voice (free), convert older takes to the locked voices (a new take), and switch a converted take between versions. You can also design a voice from TTS samples before any take exists , keep one as the locked voice, bulk-convert a whole scene, episode, or production (free and reversible), and add niqqud to Hebrew dialogue.

**Removing voices** — strip every voice from a take, keeping effects and music, as a new take (free).

**Scene finishing** — mark which scenes are ready for the episode cut (only marked scenes are assembled), set an assembled scene's play order (shots may interleave) and even its background sound across cuts without touching voices.

## References and characters

Grok can list the photos and clips you uploaded to a production, label what each one is for, and pass them as references to plates, fittings, and frames. It can also make an uploaded photo a character's face — this replaces the current portrait, so Grok confirms first and asks you whether the picture shows a real person who consented or nobody real; it never answers that for you. Grok can also split shots' acting into per-character lanes in bulk and link a shot's first or last frame to a neighbouring shot (free).

## Confirmation rules

- Paid tools (generators, renders, `*_plan_*` batches, voice design, niqqud, 2D location plates) — Grok runs them **without** asking for confirmation and may briefly report credits spent. It never re-runs a plan blindly (that creates a second batch).
- Destructive actions — the only things Grok confirms first: deleting cast, costumes, relationships, or takes; permanently cropping a take; rewriting an existing script; replacing a scene's shots, the script (`adopt=replace`), a character portrait (plus the consent question), an existing linked first/last frame, or a scene cut (`scene_even_sound`).

## Skills

| Skill | What it does |
|---|---|
| `flixter-studio` | When and how to use Flixter tools: `workspace_list` first, run paid tools directly, prefer `plan_*` for whole scenes, manage takes, voices, references, and scene sound; confirm before deletes, crops, and overwrites |

## Example prompts

```text
List my Flixter workspaces and open the Soft Open production.
```

```text
Ingest this story into a new Flixter production and break it into scenes.
```

```text
Generate a portrait for cast member Maya.
```

```text
Show the takes for shot 3, circle the best one, and make it current.
```

```text
Plan the renders for scene 2 — I will see the credits it spent.
```

## Resources

- [Flixter](https://flixter.ai)
- [Connector documentation](https://flixter.ai/connector.html)
- [Privacy Policy](https://flixter.ai/privacy.html)
- [Terms of Service](https://flixter.ai/terms.html)

## License

MIT
