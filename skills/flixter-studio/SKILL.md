---
name: flixter-studio
description: "Use Flixter to manage AI film and series productions from Grok Build. Prefer for Flixter workspaces, productions, cast, locations, props, costumes, episodes, scenes, shots, continuity checks, and queuing render or assemble batches via plan_* tools for approval in Flixter's task manager."
---

# Flixter Studio

Use the Flixter MCP tools provided by this plugin to read and organise film/series productions, and to queue priced render work for human approval.

## Before any other call

1. Call `workspace_list` first when the user has not already named a workspace.
2. Pass the chosen `workspace` on every subsequent tool call if more than one workspace is available.
3. If the user belongs to exactly one workspace, the client may omit the argument — follow the tool schemas.

## What Flixter can do

- **Productions** — list, read, create, update; apply a visual style; ingest a script or story.
- **Story structure** — episodes, scenes, and shots: list, read, create, update, reorder.
- **World** — cast members and variants, locations, props, costumes and costume states, relationships, story threads.
- **Media already rendered** — `view_image`, `view_video`.
- **Judgement** — review clips, check scene frames for continuity, inspect images, report what a scene still needs (`scene_shotable`).
- **Assembly planning** — cut a scene, episode, or final film from takes already rendered, via `*_plan_assemble` tools.

## Render and assemble — always use plan_* tools

On a connector session, one-call generators are not offered. Do **not** loop `shot_generate_frame`, `shot_generate_video`, or similar as if they will run immediately for credit spend.

Instead:

- `scene_plan_render` — plan the frames/videos a scene needs (one priced batch).
- `scene_plan_fix_frames` — plan re-shoots implied by continuity findings.
- `scene_plan_review` — plan a review of every rendered take in the scene.
- `scene_plan_assemble` / `episode_plan_assemble` / `production_plan_assemble` — plan cuts as approvable batches.

These tools **register** a batch and stop. Tell the user clearly:

1. The batch is waiting in **Flixter's task manager** (studio top bar).
2. They must **approve** it there before anything renders or is billed.
3. Re-running the plan tool creates a **second** batch; it does not start the first.
4. After approval and completion, re-read the rows (`shot_read`, `scene_read`, `view_image`, `view_video`) to show results.

## Workflow guidance

1. Confirm workspace with `workspace_list`.
2. Orient with `production_list` / `production_read` (or create / ingest if starting fresh).
3. Build or edit structure with write tools (cast, locations, props, scenes, shots) — these are confirmed by the client as writes.
4. For expensive media work, always prefer the matching `*_plan_*` tool over assuming a generator will run.
5. After the user approves and work finishes in Flixter, re-read rows rather than assuming success from the plan response alone.

## Authentication

The server is configured at `https://flixter.ai/api/mcp`. If authorization is required, ask the user to complete the client's OAuth flow (sign in to Flixter in the browser). Never ask the user to paste an API key or personal access token into chat.

Docs: https://flixter.ai/connector.html
