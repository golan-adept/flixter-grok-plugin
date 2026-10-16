---
name: flixter-studio
description: "Use Flixter to manage AI film and series productions from Grok Build. Prefer for Flixter workspaces, productions, cast, locations, props, costumes, episodes, scenes, shots, continuity checks, pitching ideas and asking multiple-choice questions as cards, runtime planning and whole-script writing, takes (choose, tag, crop, delete, video edits), character voice design, locking and bulk conversion, Hebrew niqqud, scene sound and cut order, references and character photo import (with consent), cut readiness and background-voice removal, single-asset generation (with user confirmation before spend), confirmation before deletes, crops and overwrites, and render or assemble batches via plan_* tools (confirm first: over MCP they deploy and bill on call)."
---

# Flixter Studio

Use the Flixter MCP tools provided by this plugin to read and organise film/series productions, generate individual assets after user confirmation, and queue priced render work for human approval.

## Before any other call

1. Call `workspace_list` first when the user has not already named a workspace.
2. Pass the chosen `workspace` on every subsequent tool call if more than one workspace is available.
3. If the user belongs to exactly one workspace, the client may omit the argument — follow the tool schemas.

## What Flixter can do

- **Productions** — list, read, create, update; apply a visual style; ingest a script or story.
- **Talking with the user** — `production_pitch` shows 2–4 ideas as pitch cards; `production_ask` asks a multiple-choice question as clickable cards. After either, stop and wait for the pick (it arrives as the next message); do not also write the options out as text.
- **Writing** — `production_runtime_plan` sets the scene/second budget for a target running time (call before writing scenes; an off-target plan is refused). `production_write_script` writes every scene of the plan and all their shots in parallel, creating or re-writing scene rows — confirm before running it over an existing script, then read the result and fix with `shot_update` / `scene_write_shots`. `scene_write_shots` writes all of one scene's shots in one call; `replace=true` deletes the scene's current shots first, so confirm before using it.
- **Story structure** — episodes, scenes, and shots: list, read, create, update, reorder. `production_split_acting` splits every shot whose acting is only the scene row into per-character acting lanes (one model call per scene, async batch) — confirm scope first, since it rewrites many shots and may spend.
- **Continuity links (free)** — `shot_link_frame` makes a shot open on another shot's final frame; `shot_set_last_frame` chooses the image a clip ends on (another shot's first frame, any file id, or `clear`). Both overwrite the shot's current first/last frame, so confirm when one is already set.
- **World** — cast members and variants, locations, props, costumes and costume states, relationships, story threads, eras.
- **Destructive actions — confirm first** — `cast_member_delete` (shots lose the identity lock), `costume_delete` and `relationship_delete` (cannot be undone), `shot_delete_take`, `shot_crop_take` (permanent), `scene_write_shots` with `replace=true`, `production_ingest_story` with `adopt=replace` (replaces the script), `production_write_script` over an existing script, and `cast_member_import_image` (replaces the portrait).
- **Paid side effect of a write** — `location_create` with `kind=2d` renders its establishing plate automatically on create; confirm spend before creating a 2d location, and do not call `location_generate_plate` straight after it (only to re-render after rewriting the prompt).
- **Media already rendered** — `view_image`, `view_video`, `production_inspect_image` (ask a focused question about any image file id).
- **References** — `reference_list` lists the photos/clips the user uploaded (newest first, with `note` and `subject`); `reference_update` labels one (what it shows, which object it belongs to). Check references before inventing a look, and pass their file ids as `refs` to `costume_generate_merge` / `location_generate_plate` / `prop_generate_plate` or per-medium `frame_refs` / `video_refs` on `shot_update`.
- **Character photo import** — `cast_member_import_image` makes an uploaded photo the character's face, **replacing** the current portrait and filling empty sheet fields. Use it only when the user says the picture IS the character, confirm the overwrite, and ask the user whether it shows a real person who consented (`consent=real`) or nobody real (`consent=fictional`) — never answer that yourself. Then `cast_member_generate_portrait` with `from_face=true` (a paid render — confirm) builds the 4K sheet around that face.
- **Judgement** — review clips, check scene frames for continuity, inspect images, report what a scene still needs (`scene_shotable`), apply review clips.
- **Single-asset generation** — one-call generators for portraits, plates, models, frames, and video (see below; they bill immediately).
- **Takes and voices** — list, choose, tag, crop, and delete takes; set segments; edit a rendered take; design, lock, and bulk-convert character voices; add Hebrew niqqud (see below).
- **Assembly** — cut a scene, episode, or final film from takes already rendered, via `*_plan_assemble` tools; set a scene's play order and even its background sound.

## Single-asset generators — confirm before every call

These tools run and spend workspace credits **immediately** when called:

- Cast / costume / world: `cast_member_generate_portrait`, `costume_generate_merge`, `location_generate_plate`, `prop_generate_plate`, `prop_generate_model`
- Shots: `shot_generate_frame`, `shot_generate_last_frame`, `shot_generate_end_frame`, `shot_generate_video`, `shot_edit_frame`, `shot_edit_last_frame`, `shot_extend_video`, `shot_extend_to_next`, `shot_edit_video`
- Voices and text: `cast_member_design_voice` (renders TTS voice samples), `shot_revoice_take` (a render; its description now says free, but keep confirming), `production_niqqud` (one model call per ~120 lines; edits dialogue)

Before calling any of them:

1. Confirm with the user what will run and roughly how many calls.
2. Do **not** loop generators silently across many shots or assets.
3. Many generators are async — results land on the row later. Dispatch and move on: the live descriptions (e.g. `cast_member_generate_portrait`) say not to poll or immediately re-read. When the user wants to see the result, re-read with the matching `*_read` tool (or `view_image` / `view_video`) rather than assuming the first response contains the media.

## Takes and editing

- `shot_takes` — list a shot's takes, newest first (kind, number, status, duration, tag, report).
- `shot_use_take` — make a take the shot's current one (free). `shot_tag_take` — label it Circled / Hold / NG or any short word (free).
- `shot_crop_take` — **permanently** crops a video take to from…to; `undo=1` restores the uncut take. Confirm with the user first.
- `shot_delete_take` — deletes a take (not the current one or one still rendering) along with its scene clips and voice versions. Confirm with the user first.
- `shot_take_segments` — sets a video take's segment list; it **replaces** the old list, so send the whole list.
- `shot_edit_video` — changes a rendered take with one instruction and lands as a new take. It is a generator: confirm before calling.

## Voices

- `cast_member_design_voice` — before any take exists, renders ~12 s samples in up to three voices matching the character's sheet; nothing is saved. Confirm before calling; let the user audition.
- `cast_member_use_voice` — keep a designed sample as the character's locked voice (free).
- `shot_lock_voice` — lock a character's voice from a take (their separated voice, up to 25 s). Existing takes are not converted automatically; use `shot_revoice_take` or the bulk converters. Check with the user which take to lock from.
- `cast_member_unlock_voice` — drop a locked voice; renders go back to Omni's own voice (free).
- `shot_revoice_take` — convert an older take to the locked voices; lands as a **new** take linked to the original. Treat it as a render: confirm before calling.
- `shot_remove_voices` — takes every voice out of a take (keeps effects, ambience, music); lands as a **new** take, which becomes current when the original was. Free (~10 s); bulk via `scene_/episode_/production_convert_voices use=novoices`. Confirm, because it changes what the shot plays.
- `shot_take_voice` — switch a converted take between `use=omni` and `use=locked`.
- `scene_convert_voices` / `episode_convert_voices` / `production_convert_voices` — bulk-convert every take in the scene, episode, or production to the locked voices in one parallel batch (`use=omni` switches back; both versions are kept). Free and async: re-read takes afterwards. Confirm the scope with the user, since it touches many takes.
- `production_niqqud` — adds full niqqud to Hebrew dialogue lines that have none (letters unchanged); synchronous. Confirm first (see the list above).

## Whole scenes, episodes, and films — prefer plan_* tools

For a full scene, episode, or film, prefer the batch `*_plan_*` tools over looping one-call generators:

- `scene_plan_render` — plan the frames/videos a scene needs (one priced batch).
- `scene_check_frames` — continuity review of a scene's first frames together (needs two or more frames); runs in the background. Pass its `issues` verbatim to `scene_plan_fix_frames`.
- `scene_plan_fix_frames` — plan re-shoots implied by continuity findings (its description says it spends nothing, but over MCP it deploys and bills — confirm first).
- `scene_plan_review` — plan a review of every rendered take in the scene (free).
- `scene_plan_assemble` / `episode_plan_assemble` / `production_plan_assemble` — plan cuts as approvable batches.

In Flixter's own chat these batches wait in **Flixter's task manager** (studio top bar) for approval. **Over this MCP connection the live tool descriptions say the call also approves and deploys the batch before it returns** (result: `deployed`, `dispatched`, `credits`, or a `note`). So:

1. Confirm with the user before calling any `*_plan_*` tool (except the free `scene_plan_review`): say what it covers and that it will spend on call.
2. Report `deployed` / `credits` / `note` from the result; if not deployed, the batch may be waiting in the task manager for approval.
3. Re-running the plan tool creates a **second** batch; never re-run to "check" or "start" one.
4. Results are async: re-read the rows (`shot_read`, `scene_read`, `view_image`, `view_video`) to show results.

Finishing an assembled scene:

- `scene_assemble_ready` — mark a scene ready (`on=1`) or not (`on=0`) for the episode cut; episode and final cuts assemble **only** marked scenes. Free and instant; mark only the scenes the user approved.

- `scene_footage_order` — set the scene cut's play order of pieces (may interleave shots).
- `scene_even_sound` — even the background sound across the assembled scene's cuts; voices are untouched. Free and async; the result becomes the scene's cut (a new version, `undo=1` restores the previous), so confirm before replacing the cut.

## Workflow guidance

1. Confirm workspace with `workspace_list`.
2. Orient with `production_list` / `production_read` (or create / ingest if starting fresh).
3. Build or edit structure with write tools (cast, locations, props, scenes, shots) — these are confirmed by the client as writes.
4. For a single portrait, plate, frame, or clip the user asked for: confirm spend, then call the matching one-call generator; re-read when async work finishes.
1a. Pitching or choosing a direction: `production_pitch` / `production_ask`, then wait. Writing: `production_runtime_plan` → `production_write_script` → review and fix shots.
5. For expensive whole-scene or whole-film media work: prefer the matching `*_plan_*` tool, after confirming with the user (over MCP it deploys and bills on call).
6. After work finishes, re-read rows rather than assuming success from the first response alone.

## Authentication

The server is configured at `https://flixter.ai/api/mcp`. If authorization is required, ask the user to complete the client's OAuth flow (sign in to Flixter in the browser). Never ask the user to paste an API key or personal access token into chat.

Docs: https://flixter.ai/connector.html
