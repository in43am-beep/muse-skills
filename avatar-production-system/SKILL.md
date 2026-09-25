---
name: avatar-production-system
description: The complete craft bible of the Avatar Production by MIB tool: how the avatar/presenter system works inside the video (appearance counts, segment planner, visual grammar), every script style start-to-end (rules engine, LLM competitor-level, Amish storyteller, Behind The Hug retention skeleton), the per-video output spec, and the decoded competitor-level patterns the tool implements. Use when building, changing, reviewing, or explaining anything the tool produces.
---

# Avatar Production System — the tool's craft bible

Companion to the `ai-avatar-faceless` skill (channel strategy). This file is about the TOOL: Avatar Production by MIB (repo: https://github.com/in43am-beep/avatar-production-mib, v1.0.14). Everything below is implemented in `mib/` and verified by tests + selftest.

## 1. The avatar in the video — how it works

Two render paths, one plan:

- **Free static presenter (default).** A presenter PNG with a slow Ken Burns push-in (ffmpeg zoompan), tight face close-up framing (forehead-to-beard, face centred on the upper third), speaking the voiceover slice at that point. Fast, deterministic, renders in seconds.
- **HeyGen talking-head (opt-in, paid).** The whole narration is rendered ONCE as a talking-head video (avatar3 ~1 credit/min, avatar4 ~4/min); presenter clips are cut from it at the same plan positions, lip-synced. Any failure (no key, no avatar id, API error) falls back silently to the static path — the pipeline never stalls. Cost-trap rule: **finalize audio before any HeyGen render** — script edits force full re-renders.

The segment planner decides every appearance:

| Spot kind | What it is | Length |
|---|---|---|
| `intro` | Cold open at 0.0s, full-screen close-up | `seconds` (2–15s, default 6) |
| `chapter` | At rejoin beats; opens full-screen close-up, then hard cut to 50/50 split-screen | up to 45s (close-up = 12s or 40% of the segment) |
| `interlude` | Short split-screen check-in dropped into narration gaps > 110s | 7s |

Beats shorter than 8s never get a chapter segment. The presenter returns roughly every 1–3 minutes. Every appearance speaks NEW, continuing script — the spoken slice is cut OUT of the beat audio, so no line is ever repeated.

## 2. Appearance count — the exact rules

The user picks on the wizard Presenter step: **Intro only / Intro + 1 / Intro + 2 / Intro + 3 / Intro + 5** (1, 2, 3, 4, or 6 total appearances). A plain-language line under the buttons shows real timestamps for the channel's video length.

Capping logic: mid-video spots are capped at `appearances − 1`; the intro is always kept. Chapters (real content boundaries) win over gap-fill interludes. Chapter segments never overlap the cold open.

Worked example — 30-min video, "Intro + 3" (4 appearances): cold open at 0:00 (~6s close-up); chapter segments at three evenly spaced rejoin beats (~7:30, ~15:00, ~22:30, each ≤45s: 12s close-up → split-screen); plus 7s interludes wherever narration runs >110s without the presenter.

## 3. Visual grammar (decoded frame-by-frame from the reference)

- **Cold open, no title card.** First line lands within seconds on a full-screen tight face close-up.
- **Split-screen:** B-roll LEFT, presenter RIGHT, straight vertical divide, hard edges. The presenter is ALWAYS on the right.
- **All hard cuts.** No fades, dissolves, slides, or animated wipes.
- **Captions:** bold white text on a solid black box, centred lower-third, progressive word-by-word reveal that clears per sentence.
- **Persistent watermark:** small gold bell, bottom-right corner, entire video.
- **Soft promo is audio + caption only**, once early-mid — never a visual takeover. Off by default.
- **Outro:** full-screen presenter sign-off, then the YouTube end screen (not baked in).
- **Masking rule:** never leave the avatar full-screen longer than ~20s.

## 4. Script styles, start to end

### A. Local rules engine (free default)
Title + target minutes → hook + beats + CTA. Hook: curiosity + emotion, ≤ ~18s. Beats: problem → complication → setback → twist → payoff → landing (~140 wpm). Rejoin transition lines baked into evenly spaced beats (never beat 0). CTA: subscribe + comment keyword. TTS input sanitised — no headers, stage directions, or bracketed cues ever spoken.

### B. LLM competitor-level script (Gemini / AI33 Pro)
Channel Pattern fed into the prompt; original competitor-grade narration. Then Claude review: strict mistake check, fixes auto-applied, every issue logged; failures keep the original script.

### C. Amish storyteller mode (persona: Elias Yoder)
Shocking pain-point hook, ancestral knowledge, hidden secret, industries-profit-from-ignorance angle, ≥3 personal stories, ≥2 neighbour examples, curiosity loops, never bullet points, continuous narration, next-video teaser. 8 sections scaled to target minutes × 130 wpm (±10%): HOOK 300–500 / AUTHORITY + STORY 500–1000 / THE HIDDEN SECRET 1000–2000 / WHY MODERN PEOPLE GOT IT WRONG 1000–1500 / STEP-BY-STEP METHOD 1000–2000 / COMMON MISTAKES 500–1000 / FINAL LESSON 500–1000 / NEXT VIDEO TEASER 200–300.

### D. Behind The Hug retention skeleton
Hook 0:00–0:18 (strange behaviour + unanswered question, no intro first). Setback ~55%, twist ~70%. 2-beat ending: setup + quotable landing line (≤10 words). ~120 wpm with [pause] cues. Voice LOCKED: avocado_v2:MAI_01 (Warm), speed 85, en.

## 5. Competitor-level patterns the tool implements
Decoded as craft, never as anyone's content. Presenter mechanics (§3) from the frame-by-frame reference study. Packaging: Owen Rensland (exact figures + timeframe, proof thumbnails, PAS descriptions), Chris Barrera (cents-exact figures, dashboard thumbnails, named-student proof), @BarreraChris (branded signature closer, time-boxed transformation, 3-element thumbnails), Youri van Hofwegen (20s hook template, avatar-as-proof opener, verbal open loops, concrete numbers), Albert AI (clone-the-winner engine, reusable cold-open, forensic credibility beat, chapter-as-pipeline — never its unverified income claims), Storm $300K playbook (video → free checklist → 7 emails → sales page; avatar never on screen 100%).

## 6. Per-video output spec
`output/<channel>/<slug>/`: final.mp4 (1920×1080, libx264 yuv420p, aac), title.txt, description.txt (AI-disclosure line), tags.txt, thumbnail-prompt.txt (locked-character template: 40% character / 60% problem-or-solution, ≤3-word auto overlay), costs.json, qc-report.json, script.json. QC fail → quarantine/ + reason; a bad run never kills the batch.

## 7. Title → packaging rules
Viral title generator (20 ranked titles, LLM pool + copy-paste fallback). Promise-lock: title + thumbnail + hook + script make the SAME promise. Anti-boredom: open loop in first 18s, re-engagement at 25–35%, twist ~70%, pattern interrupt every 3–7s. Max 2 verbal CTAs, benefit-framed. Auto-pilot: YouTube URL → study → titles → script → thumbnail → video, zero manual steps.

## 8. Voice rules
One locked voice per channel = channel identity; never switch between videos. Provider order: channel locked voice (AI33 Pro / clone_) → AI33 Pro default → free Edge TTS. Voiceover is the backbone: lay voice first, cut visuals to fit; verify the first 10s before mux.

## 9. Guardrails
AI disclosure in every description; never present the avatar as a real person. Never invent figures — audited numbers only. Study-first: decode mechanics, never copy content. One channel = one niche. Free/local by default; paid APIs only as opt-in toggles. Verify every "done" against live state.
