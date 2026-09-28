---
name: avatar-production-system
description: The complete craft bible of the Avatar Production by MIB tool: how the avatar/presenter system works inside the video (appearance counts, segment planner, visual grammar), every script style start-to-end (rules engine, LLM competitor-level, Amish storyteller, Behind The Hug retention skeleton), the per-video output spec, and the decoded competitor-level patterns the tool implements. Use when building, changing, reviewing, or explaining anything the tool produces.
---

# Avatar Production System — the tool's craft bible

Companion to the `ai-avatar-faceless` skill (channel strategy). This file is
about the TOOL: Avatar Production by MIB (repo:
https://github.com/in43am-beep/avatar-production-mib, v1.0.14). Everything
below is implemented in `mib/` and verified by tests + selftest.

## 1. The avatar in the video — how it works

Two render paths, one plan:

- **Free static presenter (default).** A presenter PNG with a slow Ken Burns
  push-in (ffmpeg zoompan), tight face close-up framing (forehead-to-beard,
  face centred on the upper third), speaking the voiceover slice at that
  point. Fast, deterministic, renders in seconds.
- **HeyGen talking-head (opt-in, paid).** The whole narration is rendered ONCE
  as a talking-head video (avatar3 ~1 credit/min, avatar4 ~4/min); presenter
  clips are cut from it at the same plan positions, lip-synced. Any failure
  (no key, no avatar id, API error) falls back silently to the static path —
  the pipeline never stalls. Cost-trap rule: **finalize audio before any
  HeyGen render** — script edits force full re-renders.

The segment planner (`mib/stages/avatar.py::plan_segments`, pure and
unit-tested) decides every appearance:

| Spot kind | What it is | Length |
|---|---|---|
| `intro` | Cold open at 0.0s, full-screen close-up | `seconds` (2–15s, default 6) |
| `chapter` | At rejoin beats; opens full-screen close-up, then hard cut to 50/50 split-screen | up to 45s (close-up = 12s or 40% of the segment) |
| `interlude` | Short split-screen check-in dropped into narration gaps > 110s | 7s |

Beats shorter than 8s never get a chapter segment. The presenter returns
roughly every 1–3 minutes. Every appearance speaks NEW, continuing script —
the spoken slice is cut OUT of the beat audio, so no line is ever repeated.

## 2. Appearance count — the exact rules

The user picks on the wizard Presenter step: **Intro only / Intro + 1 /
Intro + 2 / Intro + 3 / Intro + 5** (1, 2, 3, 4, or 6 total appearances).
Plain-language line under the buttons shows real timestamps for the
channel's video length, e.g. "about at 0:00, 7:30, 15:00, 22:30 of a
30-min video".

Capping logic: mid-video spots are capped at `appearances − 1`; the intro is
always kept. Chapters (real content boundaries) win over gap-fill
interludes. Chapter segments never overlap the cold open.

Worked example — 30-min video, "Intro + 3" (4 appearances): cold open at
0:00 (~6s close-up); chapter segments at three evenly spaced rejoin beats
(~7:30, ~15:00, ~22:30, each ≤45s: 12s close-up → split-screen); plus 7s
interludes wherever narration runs >110s without the presenter.

## 3. Visual grammar (decoded frame-by-frame from the reference)

- **Cold open, no title card.** First line lands within seconds on a
  full-screen tight face close-up.
- **Split-screen:** B-roll LEFT, presenter RIGHT, straight vertical divide,
  hard edges. The presenter is ALWAYS on the right — the only split layout.
- **All hard cuts.** No fades, dissolves, slides, or animated wipes between
  presenter states.
- **Captions:** bold white text on a solid black box, centred lower-third,
  progressive word-by-word reveal that clears per sentence (cumulative, not
  scrolling).
- **Persistent watermark:** small gold bell, bottom-right corner, entire video.
- **Soft promo is audio + caption only.** Product/affiliate mentions land in
  the narration (captioned like everything else), inserted once early-mid —
  never a presenter interruption or visual takeover. Off by default; channels
  opt in deliberately.
- **Outro:** full-screen presenter sign-off, then the YouTube end screen
  (not baked into the video).
- **Masking rule:** never leave the avatar full-screen longer than ~20s —
  full-screen close-ups stay short; the bulk of each segment is split-screen,
  which hides artifacts AND holds attention.

## 4. Script styles, start → end

### A. Local rules engine (free default, `mib/stages/script.py`)

Title + target minutes → `{hook, beats[], cta}`. The formula comes from the
channel's Pattern profile (`mib/patterns.py`); no custom pattern → the
built-in "Classic Story".

- **Hook:** curiosity + emotion, ≤ ~18s (~40 words).
- **Beats:** problem → complication → setback → twist → payoff → landing
  (or the channel's own beat map). ~140 wpm.
- **Rejoins:** mid-video avatar re-appearances each open with a transition
  line baked into that beat's narration — evenly spaced, never beat 0.
- **CTA:** subscribe + comment keyword.
- **Sanitised TTS input:** no headers, stage directions, markdown, or
  bracketed cues are ever spoken (this fixed the "extra spoken part at the
  start" bug).
- **Word budget:** hook ~12% of total, CTA ~8%, remainder split across beats.
- **Image prompts:** photorealistic, 16:9, no text, no watermarks.

### B. LLM competitor-level script (Gemini / AI33 Pro)

The channel's Pattern profile is fed into the prompt; the LLM writes
original, competitor-grade narration (never copied). Then **Claude review**:
strict mistake-check pass, fixes auto-applied, every issue logged; any
review failure keeps the original script. Saved per-provider in config and
honoured by the pipeline.

### C. Amish storyteller mode (persona: Elias Yoder)

Selected in Settings → Script → "Script style". The LLM writes the full
narration in the voice of Amish farmer Elias Yoder: shocking pain-point
hook, ancestral knowledge (grandfather/mother/ancestors), a hidden secret
modern people don't know, the industries-profit-from-ignorance angle, ≥3
personal stories, ≥2 neighbour/community examples, curiosity loops
("But that's not the most important part…", "What happened next shocked
me…", "Nobody talks about this anymore…"), never bullet points, continuous
voice-over narration, next-video teaser ending. Tone: warm, confident,
old-fashioned, practical, slightly controversial.

8 sections, word counts scaled proportionally to **target minutes × 130 wpm
(±10%)**:

| # | Section | Base words |
|---|---|---|
| 1 | HOOK | 300–500 |
| 2 | AUTHORITY + STORY | 500–1000 |
| 3 | THE HIDDEN SECRET | 1000–2000 |
| 4 | WHY MODERN PEOPLE GOT IT WRONG | 1000–1500 |
| 5 | STEP-BY-STEP METHOD | 1000–2000 |
| 6 | COMMON MISTAKES | 500–1000 |
| 7 | FINAL LESSON | 500–1000 |
| 8 | NEXT VIDEO TEASER | 200–300 |

Curiosity beat every 30–60 seconds. Needs an LLM key; the local rules engine
keeps competitor mechanics as the free fallback.

### D. Behind The Hug retention skeleton (emotional dog stories)

- **Hook 0:00–0:18:** strange behaviour + unanswered question, no intro first.
- **Setback** at ~55% runtime, **twist** at ~70%.
- **2-beat ending:** setup line + quotable landing line (≤10 words).
- Voiceover ~120 wpm with `[pause]` cues (stripped before TTS).
- Voice LOCKED: `avocado_v2:briggs` (Husky Campfire), speed 88, en — every
  video (changed 2026-09-27 per user order; old `avocado_v2:MAI_01` retired —
  it felt old and was hurting views).
- The bottom-right AI label is cropped out with a ~5% centre zoom; the
  AI disclosure stays in each video's `description.txt`.

## 5. Competitor-level patterns the tool implements

Decoded from real channels; encoded as **craft, never as anyone's content**.
Study-first rule: decode the packaging mechanics, write original topics.

- **Presenter mechanics (frame-by-frame reference study):** §3 above IS this
  decode — cold open, chapter re-entries, interludes, hard cuts, caption
  style, gold bell, audio-only soft promo, full-screen outro.
- **Packaging (Owen Rensland):** exact money figure + timeframe in titles, odd
  non-round numbers, one bracket proof tag, ≤4-word thumbnails with proof
  element, description as PAS sales letter + fixed CTA stack, chapters with a
  value-proposition inside the first 10%.
- **Packaging (Chris Barrera):** money + timeframe always, figures exact WITH
  cents, dashboard-as-thumbnail, tool-logo trust badge, named-student proof
  with exact figures, time-boxed offers.
- **Packaging (@BarreraChris):** branded signature closer repeated
  catalog-wide, time-boxed transformation deadlines, portfolio proof
  (aggregate > single), 3-element thumbnail template (face + exact-figure
  dashboard + recurring chart motif), ~2/week cadence.
- **Avatar craft (Youri van Hofwegen):** 20s hook template
  (pattern-interrupt claim → 3 concrete failure modes → anti-theory promise →
  triad close), avatar-as-proof opener, verbal open loops instead of
  chapters, concrete numbers never adjectives, one visible result every
  2–3 min, burnt-in captions from frame 0, two-temperature lighting in
  realism prompts, voice-record-once checkpoint.
- **Channel craft (Albert AI):** clone-the-winner concept engine (one viral
  faceless channel rebuilt end-to-end per video), reusable verbatim cold-open
  script, forensic credibility beat before the tutorial, chapter-as-pipeline
  (one chapter per production step), final-payoff close. DECEPTIVE-ADJACENT
  FLAGS — never encode: unverified cents-exact income claims presented as
  fact, "I cloned [real creator]" framing, view-badge thumbnails implying
  the method earned them, missing affiliate disclosures.
- **Funnel (Storm $300K/90-day playbook):** video → free 1–2 page checklist
  lead magnet → landing page → 7-email sequence → sales page; price ladder
  $27–47 → $12–19 bump → $67–97 upsell → $197–297 flagship; avatar never on
  screen 100% (b-roll cut every 3–6s); 2 long-form + 5–7 Shorts/week; fix the
  EARLIEST broken dashboard step first, one variable at a time.

## 6. Per-video output spec

`output/<channel>/<slug>/`:

- `final.mp4` — 1920×1080, libx264 yuv420p, aac 44100 Hz; intro + per-beat
  segments (image zoompan, duration = beat audio length).
- `title.txt`, `description.txt` (carries the AI-disclosure line),
  `tags.txt`, `thumbnail-prompt.txt`, `costs.json`, `qc-report.json`,
  `script.json`.
- `thumbnail-prompt.txt` — **1 title = 1 thumbnail, starring the channel's
  own avatar.** When the channel has a character sheet (Settings → Channels
  → Character sheet…), the prompt is built by
  `mib/prompts/thumbnail.build_avatar_prompt()`: the sheet's identity-lock
  block replaces the default character spec, the avatar is staged large in
  the foreground looking into the camera with a title-matched expression,
  problem/solution visual beside/behind it. Overlay text auto-derived from
  the title (≤3 words, UPPERCASE). No sheet → legacy default template.
- **Packaging training (Kevis method, 2026-09-28):** before locking a
  channel's thumbnail template, scrape ~40–50 thumbnails + titles from a
  winning *adjacent-niche* channel (top 50 by popular for large channels),
  title embedded in each image, and feed them to the image model in
  ~20-image batches so it learns the title↔thumbnail pairings; every new
  thumbnail is then generated from that learned format with the channel's
  canonical avatar face attached ("Use the face attached as the avatar").
- `thumbnail.jpg` — rendered automatically by
  `mib/stages/package.render_thumbnail()` right after packaging, **exactly
  one per video**. The sheet/reference images go to the image model as
  visual input (Gemini) so the same avatar appears in every thumbnail.
  No image API key → "prompt file only" mode (txt only, no jpg).
- QC gates: mp4 exists, duration sane, audio+video streams present. Fail →
  `quarantine/` + reason. A bad run never kills the batch.

## 7. Title → packaging rules (promise-lock system)

- **Viral title generator:** the user's title brief → 20 ranked titles
  (strongest first, top 3 pre-ticked) via the script-LLM pool; no key →
  copy-paste prompt dialog (free fallback).
- **Promise-lock:** title + thumbnail + hook + script make the SAME promise;
  QC quarantines mismatches. Independent confirmation: Kevis Unfiltered's
  2026-09-28 full course teaches the identical rule ("the title and
  thumbnail and video all need to deliver the same promise").
- **Anti-boredom:** open loop in the first 18s, re-engagement beat at
  25–35%, twist ~70%, never 3 same-mode scenes in a row, pattern interrupt
  every 3–7s.
- **Subscribe conversion:** max 2 verbal CTAs per video, benefit-framed, one
  early + one at end; echoed in end screen, pinned comment, description.
- **Auto-pilot:** paste a YouTube URL → study (metadata + packaging formulas
  only) → 20 titles → top title → full pipeline in Amish storyteller mode →
  thumbnail prompt. Zero manual steps.

## 8. Voice rules

- **One locked voice per channel** = channel identity. Never switch voices
  between videos.
- Provider order: channel locked voice (AI33 Pro / `clone_` voice clone) →
  AI33 Pro default → free Edge TTS.
- Voice clone: upload 1–3 min clear speech (no music) → `clone_` id
  auto-locked to that channel's avatar.
- Voiceover is the backbone: lay voice first, cut visuals to fit. Verify the
  first 10s of every voiceover before mux.

## 9. AI-filmmaking consistency learnings (masterclass 2026-09-27)

Decoded from a 2-hour AI filmmaking masterclass (Dola AI + Seedance 2.5 +
Claude/ChatGPT workflows). Encoded as craft only — tool-agnostic, applies to
any keyframe/video pipeline including this tool's free static path.

- **Character sheet + lock list.** Before generating any visual, build one
  character sheet per video: every character's full design in words, one
  reference description each, and a LOCK LIST restated in EVERY image/video
  prompt: character identity, wardrobe (exact garments + colours), key props,
  exact in-frame position/posture, screen direction, and the framing rule
  "match the previous shot's end frame". Anything not named in the prompt
  WILL drift — the class demoed a prop (a key) becoming a different object
  in every shot because no prompt mentioned it.
- **Object sheets.** Key props get their own mini-sheet (shape, material,
  colour, size, distinguishing marks) and are named in every prompt where
  they appear — or in every prompt, period.
- **Text references beat photo references.** Their tests: pasting the full
  character description as TEXT in every prompt gave better consistency
  than attaching photo references (photo refs made the model copy the photo
  instead of the character). Keep identity-lock prompt blocks text-first.
- **Last-frame / first-frame check.** Verify continuity by comparing the
  last frame of scene N with the first frame of scene N+1 (same character
  position, wardrobe, lighting, prop placement). Make this a QC gate, not a
  spot check.
- **Never start from a blank page.** Their pipeline adapts an existing
  story/comic/novel, compresses it, locks the visual look, then expands to
  a shot list. For original work: the beat map + retention skeleton IS the
  "source" — lock it before any visual is generated.
- **Trailer-first planning.** Cut the hook/trailer before the full video;
  it forces the curiosity arc to be decided up front and becomes the
  video's packaging test.
- **Music bed rule.** Copyright-free ambient music under the voiceover at
  low level (~10–15% under dialogue) + light SFX. Voiceover-only videos
  feel empty by comparison; the music bed is a retention lever, not decor.
- **Motion-control awareness.** Their "white motion" trick: extract a motion
  template (skeleton/depth) from any reference video, then regenerate the
  same motion with new characters. Portable lesson: when real video-gen is
  available, drive motion from a reference template rather than text-only
  motion prompts — text-only motion is where glitches come from.

## 11. Raw iPhone capture style — Behind The Hug viral format (decoded 2026-09-28)

Decoded from the channel's #1 video ("This Dog Didn't Recognize Him… Until
This Moment ❤️", 335k views, 4:51). Use for raw-style emotional videos
(homecomings AND departures/breakdowns) — NOT for narrated story videos.

- **No narrator, ever.** Sound is diegetic only: the people's own unscripted
  lines ("I missed you", "I'm home", "Hey buddy"), dog sounds, ambient.
  The winner's transcript is 100% natural speech + `[music]` cues.
- **Light emotional music bed** (soft piano/strings), low under dialogue,
  swells gently at the emotional peak only.
- **Camera = family member with iPhone 15:** 4K30, handheld micro-shake,
  natural daylight only, no color grading, true smartphone colors,
  occasional autofocus hunting on close-ups, imperfect framing.
- **Edit:** hard cuts, no transitions, no slow motion, no title card. Hook =
  the most emotional ~15s FIRST, then the story rewinds naturally.
- **Opening caption teaser** (winner: "He waited every day.") + burned
  captions from frame 0: bold white on black box, lower-third.
- **Departure variant:** same grammar, emotion moved from reunion to the
  goodbye — dog blocks like a child, cries, lays down, refuses to let go.
- **Packaging mirrors the promise:** description opens with the emotional
  story + "filmed naturally… no fancy editing" framing; AI disclosure stays
  in the description per channel rule.

## 12. Guardrails (never break)

- AI disclosure in every video's description; never present the avatar as a
  real person.
- Never invent figures: subscriber counts, reader results, earnings —
  audited numbers only, placeholders like `[INSERT REAL READER RESULT]`
  otherwise.
- Study-first: decode competitor packaging mechanics, never copy content.
- One channel = one niche; cross-category mixing blocked.
- Free/local by default; paid APIs only as the user's opt-in toggles.
- Verify every "done" against live state before reporting it.
