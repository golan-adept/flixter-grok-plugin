---
name: flixter-studio
description: "Use Flixter to manage AI film and series productions from Grok Build. Prefer for Flixter workspaces, productions, cast, locations, props, costumes, episodes, scenes, shots, continuity checks, takes (choose, tag, crop, delete, video edits), character voice locking, scene sound and cut order, single-asset generation (with user confirmation before spend), and queuing render or assemble batches via plan_* tools for approval in Flixter's task manager."
---

# Flixter Studio

Use the Flixter MCP tools provided by this plugin to read and organise film/series productions, generate individual assets after user confirmation, and queue priced render work for human approval.

## Before any other call

1. Call `workspace_list` first when the user has not already named a workspace.
2. Pass the chosen `workspace` on every subsequent tool call if more than one workspace is available.
3. If the user belongs to exactly one workspace, the client may omit the argument — follow the tool schemas.

## What Flixter can do

- **Productions** — list, read, create, update; apply a visual style; ingest a script or story.
- **Story structure** — episodes, scenes, and shots: list, read, create, update, reorder.
- **World** — cast members and variants, locations, props, costumes and costume states, relationships, story threads, eras.
- **Media already rendered** — `view_image`, `view_video`.
- **Judgement** — review clips, check scene frames for continuity, inspect images, report what a scene still needs (`scene_shotable`), apply review clips.
- **Single-asset generation** — one-call generators for portraits, plates, models, frames, and video (see below; they bill immediately).
- **Takes and voices** — list, choose, tag, crop, and delete takes; set segments; edit a rendered take; lock character voices and revoice older takes (see below).
- **Assembly** — cut a scene, episode, or final film from takes already rendered, via `*_plan_assemble` tools; set a scene's play order and even its background sound.

## Single-asset generators — confirm before every call

These tools run and spend workspace credits **immediately** when called:

- Cast / costume / world: `cast_member_generate_portrait`, `costume_generate_merge`, `location_generate_plate`, `prop_generate_plate`, `prop_generate_model`
- Shots: `shot_generate_frame`, `shot_generate_last_frame`, `shot_generate_end_frame`, `shot_generate_video`, `shot_edit_frame`, `shot_edit_last_frame`, `shot_extend_video`, `shot_extend_to_next`, `shot_edit_video`
- Voices: `shot_revoice_take` (a render that may spend credits)

Before calling any of them:

1. Confirm with the user what will run and roughly how many calls.
2. Do **not** loop generators silently across many shots or assets.
3. Many generators are async — results land on the row later; re-read with the matching `*_read` tool (or `view_image` / `view_video`) rather than assuming the first response contains the media.

## Takes and editing

- `shot_takes` — list a shot's takes, newest first (kind, number, status, duration, tag, report).
- `shot_use_take` — make a take the shot's current one (free). `shot_tag_take` — label it Circled / Hold / NG or any short word (free).
- `shot_crop_take` — **permanently** crops a video take to from…to; `undo=1` restores the uncut take. Confirm with the user first.
- `shot_delete_take` — deletes a take (not the current one or one still rendering) along with its scene clips and voice versions. Confirm with the user first.
- `shot_take_segments` — sets a video take's segment list; it **replaces** the old list, so send the whole list.
- `shot_edit_video` — changes a rendered take with one instruction and lands as a new take. It is a generator: confirm before calling.

## Voices

- `shot_lock_voice` — lock a character's voice from a take (their separated voice, up to 25 s); later renders use it.
- `cast_member_unlock_voice` — drop a locked voice; renders go back to Omni's own voice (free).
- `shot_revoice_take` — convert an older take to the locked voices; lands as a **new** take linked to the original. Treat it as a render: confirm before calling.
- `shot_take_voice` — switch a converted take between `use=omni` and `use=locked`.

## Whole scenes, episodes, and films — prefer plan_* tools

For a full scene, episode, or film, prefer the batch `*_plan_*` tools over looping one-call generators:

- `scene_plan_render` — plan the frames/videos a scene needs (one priced batch).
- `scene_plan_fix_frames` — plan re-shoots implied by continuity findings.
- `scene_plan_review` — plan a review of every rendered take in the scene (free).
- `scene_plan_assemble` / `episode_plan_assemble` / `production_plan_assemble` — plan cuts as approvable batches.

These tools **register** a batch and stop. Tell the user clearly:

1. The batch is waiting in **Flixter's task manager** (studio top bar).
2. They must **approve** it there before anything renders or is billed.
3. Re-running the plan tool creates a **second** batch; it does not start the first.
4. After approval and completion, re-read the rows (`shot_read`, `scene_read`, `view_image`, `view_video`) to show results.

Finishing an assembled scene:

- `scene_footage_order` — set the scene cut's play order of pieces (may interleave shots).
- `scene_even_sound` — even the background sound across the assembled scene's cuts; voices are untouched.

## Workflow guidance

1. Confirm workspace with `workspace_list`.
2. Orient with `production_list` / `production_read` (or create / ingest if starting fresh).
3. Build or edit structure with write tools (cast, locations, props, scenes, shots) — these are confirmed by the client as writes.
4. For a single portrait, plate, frame, or clip the user asked for: confirm spend, then call the matching one-call generator; re-read when async work finishes.
5. For expensive whole-scene or whole-film media work: prefer the matching `*_plan_*` tool so the batch is approved in Flixter's task manager.
6. After work finishes, re-read rows rather than assuming success from the first response alone.

## Authentication

The server is configured at `https://flixter.ai/api/mcp`. If authorization is required, ask the user to complete the client's OAuth flow (sign in to Flixter in the browser). Never ask the user to paste an API key or personal access token into chat.

Docs: https://flixter.ai/connector.html
