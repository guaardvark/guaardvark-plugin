---
name: music-video
description: >-
  Turn a song into a beat-synced music video with Guaardvark's Director: song analysis,
  per-cut shot prompts, storyboard keyframes, image-to-video renders, cuts on the beat.
  Use when the user says "music video", "video for this track", or wants visuals timed to
  a song. Has an approval gate before GPU spend.
---

# Music video with Guaardvark

Read `setup` first. Needs the `comfyui` plugin and an installed i2v model
(`wan22-5b` by default; `minimax-h3-int8` for frame-matched continuity).

## Start: MCP `generate_music_video`

- `song`: document id or a filesystem path to mp3/wav/flac/ogg. A path is uploaded for the user.
- `style_prompt`: mood, palette, movement, era ("grainy 16 mm, neon rain, slow dolly, 1984").
- `name` (defaults to the song file name), `i2v_model` (defaults to the active i2v model).
- The tool analyses the song (tempo, beats, sections), writes one unique prompt per cut and
  **stops at the approval gate**. It does not spend GPU rendering clips. Tell the user that.

## The gate and the rest of the pipeline (REST)

`B=${GUAARDVARK_URL:-http://localhost:5000}`, `ID` = the project id the tool returned.

| step | call | what it does |
|---|---|---|
| read the plan | `GET $B/api/music-video/$ID` | cuts, prompts, status, storyboard state |
| edit prompts | `POST $B/api/music-video/$ID/plan` with the edited cut prompts (and optional global style) | operator edits before render |
| regenerate the plan | `POST $B/api/music-video/$ID/regenerate-plan` or `/replan` | new prompts from the Director |
| storyboards first | `POST $B/api/music-video/$ID/generate-storyboards` | one keyframe still per cut, thumbnails-first review |
| redo one keyframe | `POST $B/api/music-video/$ID/regen-storyboard/<idx>` | keeps the others |
| **approve** | `POST $B/api/music-video/$ID/approve` | releases per-clip generation. Only on the user's explicit yes |
| re-analyse | `POST $B/api/music-video/$ID/analyze` | restart song analysis |
| cancel | `POST $B/api/music-video/$ID/cancel` | |
| list all | `GET $B/api/music-video` (`?project_id=<id>` or `?project_id=none` to filter) | |
| progress | `GET $B/api/music-video/$ID/progress` | stages, the clip in flight (sampler step/steps), ETA. Report this instead of guessing |
| rename / file under a project | `PATCH $B/api/music-video/$ID` with `{name?, project_id?, notes?, tags?}` | the finished video joins that project's Documents |
| edit one cut | `POST $B/api/music-video/$ID/cut/<idx>` with `{prompt?, seed?, locked?}` | at the approval gate it only changes the plan; **while generating it re-renders that cut**: ask first |
| reopen a finished video | `POST $B/api/music-video/$ID/revise` | back to the approval gate, rendered clips kept; on approve only changed cuts render. `/replan` re-renders all |
| copy | `POST $B/api/music-video/$ID/duplicate` with `{name?, project_id?, keep_clips?}` | new video at the approval gate |
| versions | `GET/POST $B/api/music-video/$ID/revisions`, `POST .../revisions/<rid>/restore` | saved on approve, reassemble, revise, restore; restore lands at the approval gate |
| a project's videos | `GET $B/api/projects/<id>/videos` | music videos and Film Crew filed under it |

Creating a project without the MCP tool: `POST $B/api/music-video` with
`{"name", "song_document_id", "style_prompt", "settings": {"i2v_model": "..."}, "user_treatment": "optional short story", "project_id": optional}`.
The MCP tool takes `project` (a Project id or exact name).
`user_treatment` becomes the creative source the Director writes cuts from.

## How to run it well

1. Start the project, show the cut list and the beat count.
2. Offer storyboards before approval: they are cheap and catch a wrong style early.
3. Ask for approval in plain words with the cost: number of cuts times the clip time for the
   chosen model. Approve only after a clear yes, and only once `GET $B/api/music-video/$ID`
   shows `current_stage` `awaiting_approval` with cuts: before analysis finishes the approve
   route answers 409.
4. Character in the video: create a Cast subject and train a LoRA first (the cast skill) so the
   Director can lock identity across cuts.
5. The finished cut is assembled to the detected beat grid. The Video Editor page can trim or
   overlay text afterwards.
