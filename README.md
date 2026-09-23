# Muse Sub-Skills & Knowledge

Custom skills and knowledge bases built for the Muse personal AI agent.
Copy any skill folder into `~/workspace/skills/` to install it.

## Skills

### screenwriting-craft
Screenwriting craft for narrative writing: format rules, 3-act/beat structure,
dialogue, scene construction, emotional beats — and how to apply them to
narrated YouTube storytelling (hooks, retention beats, voiceover).
Use when writing or improving stories, voiceovers, scripts, or narrative outlines.

- `SKILL.md` — skill definition and usage
- `references/` — structure beats, dialogue & scenes, emotional craft,
  format reference, voiceover application, sources
- `assets/beat-sheet-template.md` — reusable beat-sheet template

### etsy-q4-research
Etsy Q4 research & product discovery playbook: niche research, product
validation, and listing strategy for the Q4 printables line.

- `SKILL.md` — skill definition and usage

### ai-avatar-faceless
Train and run AI-avatar faceless YouTube channels end to end: avatar realism
(inputs, eye test, artifact checklist), lip-sync tool picks and cost traps,
the full automation pipeline (script → voice → avatar → b-roll → edit →
publish), retention hooks and editing rules, subscribe-conversion placements,
and community-building routines.

- `SKILL.md` — skill definition and usage
- `RESEARCH.md` — field research and operator playbooks

### free-finance-intel
50 free websites showing what billionaires buy and read: investor portfolios
(dataroma, whalewisdom), insider/politician trades, SEC filings, Buffett/Dalio/
Marks letters, valuation data (Damodaran), market screeners, backtesting tools,
and investing education.

- `SKILL.md` — skill definition and usage

### jev-decision-layer
Decision-routing layer for multi-agent and multi-step work: turn a messy agent
graph into a controlled loop of state → scored routes → confidence threshold →
execute or die → verify winners → next state. Separates reasoning (expensive
model) from decision-making (cheap explicit routing).

- `SKILL.md` — skill definition and usage

## Install

```bash
cp -r screenwriting-craft etsy-q4-research ai-avatar-faceless free-finance-intel jev-decision-layer ~/workspace/skills/
```

## Notes

- Skills are plain Markdown playbooks — no code, no dependencies.
- Personal memory and user data are intentionally NOT included in this repo.
