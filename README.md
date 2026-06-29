# Veyne AI

**Narrative platform where the LLM is the actor, not the whole system.**

🌐 [Veyne AI site](https://katharz1z.github.io/Veyne-AI/) · 📬 [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com)

---

## What this is

Veyne AI is an AI-powered interactive narrative platform. On the surface it looks like a character chat. Under the hood it works like a hybrid of a visual novel, a text RPG, a dating sim, and a story engine — with persistent memory, relationship state, structured story arcs, world pressure, and progression gates.

Most character AI services degrade into one of three patterns: a helpful assistant wearing a costume, endless flirt with no structure, or beautiful prose that forgets everything after ten turns. Veyne is built to solve all three at the architecture level.

**The core principle:** the LLM generates the visible character prose. The surrounding runtime manages state, memory, relationship metrics, story progression, world constraints, scene drivers, and canon boundaries.

```text
Player message
    ↓
Context Pack Builder
    ↓
Runtime package + active scene state
    ↓
Call 1 — Character prose generation  (LLM)
    ↓
Visible response guard / repair
    ↓
State, memory, relationship, story update
    ↓
Reducer → New State
```

The player sees only the character's response. The rest is invisible.

---

## What makes this different

| Typical character chat | Veyne AI |
|---|---|
| LLM is the whole system | LLM is one stage inside a narrative runtime |
| Character agrees with everything | Character has goals, resistance, limits, and pressure |
| "Trust" is a single number that grows fast | Six separate relationship metrics with progression gates |
| Forgets previous sessions | Persistent multi-layer memory and session resume |
| Prompt rules enforce style | Source bibles, authored playbooks, TurnPlans, and guards enforce narrative |
| One giant prompt | Context Pack Builder selects only what matters for the current turn |

---

## Current state

**WIP checkpoint:** `r1.40` — Rocinante live runtime audit handoff  
**Status:** Active development, not yet publicly playable

What exists today:

- Web surface: landing, character catalog/profile, auth shell, saved sessions, resume, chat UI
- Supabase-backed session persistence with local fallback for prototype work
- Python FastAPI runtime adapter behind the web chat
- Runtime package loader for `lira-dreyvor` and `adeline-veir`
- Context pack / source-fragment path feeding the character model
- Rocinante-X 12B connected through an OpenAI-compatible vLLM / Colab path
- Live LLM turns reaching the UI and being persisted
- Visible response guard / repair layer for language, dead-end questions, and playable affordances

What is in progress:

- Auditing the Beat 2/3 live path after the sign reaction
- Tightening the alignment between visible prose, committed state, and playable next move
- Mapping responsibility across runtime package state, TurnPlan, context pack rendering, source fragments, reducer, and visible guard
- Building minimal regressions before the next runtime patch

What is not public yet:

- No open playable build
- No image generation or TTS
- No finished creator marketplace
- No NSFW layer

The current proof target is simple and brutal: one saved story should continue for 10–30 turns without collapsing into a transcript dump, interrogation loop, romance leak, language drift, unsupported world invention, or state/prose mismatch.

---

## Latest runtime signal

The current live stack has crossed the important "is the LLM seam alive?" line.

```text
Web chat UI
    ↓
Next runtime bridge
    ↓
Python FastAPI adapter
    ↓
Runtime package + context pack
    ↓
Rocinante-X 12B
    ↓
Visible guard / repair
    ↓
UI + persistence
```

Latest live exports showed:

```text
engineStatus: remote
adapterMode: llm/generated
model: rocinante-x-12b
messageFormat: single_user
```

That is good. It is also not victory champagne time. The live response is more vivid, but Beat 2/3 still exposes the real product problem: good prose is useless if it commits state the visible scene did not clearly earn.

Known current failures:

- Sign reaction can over-expand into spectacle.
- Rocinante can invent unsupported physical details.
- Visible prose may not clearly perform the same status decision the state commits.
- Beat 3 can close in state while the playable scene still feels unresolved.

The next work is not another landing page polish pass. It is runtime control: the scene must stay beautiful **and** playable. Annoying, yes. Necessary, also yes.

---

## Characters and worlds

The long-term catalog direction is **162 characters across 3 worlds**. That is a direction, not a launch-day magic trick. The current priority is proving that a small number of characters can hold memory, pressure, and story continuity first.

Currently in development:

**Lira Dreyvor** — Acting High Priestess of the Black Temple. 19 years old. Appointed three months ago after her predecessor's death under circumstances officially classified as an accident. Too young, too small, not enough experience — she compensates with severity, fast reaction, and absolute control over what she can control. Does not trust strangers. Does not open up fast. Does not become warm from sympathy alone.

**Adeline Veir** — Field Registrar of the Light Chancellery. 27 years old. Believes that records protect people from arbitrary violence. The problem: she has seen records become clean, lawful, very convenient sentences. Does not shout. Does not threaten first. Writes things down. That is often worse.

**Iney** — A wanderer between factions and routes, in the space where neither Light nor Dark has full control. He trades in paths, favors, and half-truths: rarely giving a straight answer, but always leaving the player with a choice that costs something later. Never on anyone's side — or on everyone's at once, which is more dangerous.

---

## Architecture

### Pipeline per turn

```text
Player message
    ↓
Context Pack Builder
  · Character kernel
  · Current scene state
  · Knowledge scope
  · Active arc / beat rules
  · Selected source fragments
  · Relationship state + progression gates
  · Scene driver + character intent
  · Recent dialogue
    ↓
Call 1 — Character Response Generator
  · Produces visible prose in character
  · Constrained by runtime package, playbook, and source layer
  · Must not invent unsupported world facts
    ↓
Visible response guard / repair
  · Rejects or repairs language drift, dead-end lore homework, unsupported invention, missing affordance
    ↓
State / memory / relationship / story update
    ↓
Reducer
  · Applies delta to runtime snapshot
  · Enforces relationship gates
  · Handles beat progression and exit conditions
    ↓
Next state
```

### Relationship engine

Six separate metrics. Each unlocks through different conditions. They do not move together.

```text
trust       — willingness to rely on the player
respect     — recognition of player intelligence, honesty, resilience
caution     — threat-driven wariness, stays high even when trust rises
curiosity   — interest in who the player is
attachment  — locked early; cannot be faked by tactical protection
attraction  — locked behind story progress, trust, attachment, and consent logic
```

Romance and NSFW are gated separately on top of these. The default product direction is SFW-first: intimacy must come from explicit user intent, story progress, relationship state, and consent logic — not from the model getting horny because a prompt sneezed.

### Memory

Memory is not a raw transcript. The system tracks:

- `known_facts` — what the character knows about the player
- `remembered_actions` — what the player said and did this run
- `unresolved_between_them` — open conflicts and threads
- `broken_promises`, `protected_or_pressured_character`

The extractor/reducer updates memory each turn. The context pack selects only what is relevant for the current call.

### World and canon

Characters do not get to hallucinate world facts because the prose felt moody that day. The source layer defines what may be true. State and `knowledge_scope` define what is true right now in the active run. Context Pack Builder bridges those layers into the model call.

### What continuity should look like

The target experience is not a prettier first message. It is a session that can return to yesterday without pretending nothing happened.

Example:

```text
Turn 1: the player lies about where they came from.
State: caution rises, trust drops, unresolved thread is added.
Turn 8: the character brings that lie back as pressure before making a new decision.
```

That is the product thesis in miniature: visible prose plus invisible state, memory, and consequence.

---

## Tech stack

| Layer | Stack |
|---|---|
| Web frontend | Next.js App Router, TypeScript, plain CSS |
| Database / auth | Supabase |
| Runtime adapter | Python, FastAPI, uvicorn |
| LLM inference | OpenAI-compatible endpoint through vLLM / Colab / future hosted GPU |
| Current character model | Rocinante-X 12B v1 |
| Runtime package path | JSON packages + context fragments + guard/repair layer |
| Target production models | Rocinante-X 12B for free tier, 20–25B model for low premium |

---

## Roadmap

```text
✅  CLI runtime foundation
    State, reducer, relationship engine, context pack builder, Act 1 playbook.

✅  Web surface skeleton
    Character pages, auth shell, sessions, chat, persistence, resume.

✅  Python runtime bridge
    FastAPI adapter, package-backed execution, Rocinante-compatible LLM seam.

🔄  Live runtime control
    r1.40 audit: Beat 2/3 sign reaction, state/prose alignment, playable next move.

⬜  MVP continuity proof
    One saved Act 1 story, 10–30 turns, no collapse.

⬜  v0.9 — limited public access
    Text only, no images/TTS, early testers.

⬜  v1.0 — service foundation
    Free tier + low premium, catalog expansion begins.

⬜  v2.0 — platform expansion
    70B models, creator tools, branching Act 2, TTS.
```

---

## GPU infrastructure

The project runs without external funding. Development happens on personal resources and temporary Colab tunnels.

For stable production inference we need dedicated GPU cloud. GPU funding buys stable long-run sessions, not prettier landing pages.

- **~$400–600/month** — RunPod or equivalent for Rocinante-X 12B, stable inference for MVP testing and the future free tier
- **~$600–1000/month** — 20–25B model for Low Premium tier, better prose quality and future image generation experiments

If you're interested in sponsoring or investing, or just want to follow along — write to [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com).

---

## Contact

**Email:** [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com)  
**Site:** [katharz1z.github.io/Veyne-AI](https://katharz1z.github.io/Veyne-AI/)  
**Discord:** coming soon

> *I opened the chat. But entered a world.*  
> *I wrote one line. But the character remembered ten things.*  
> *I came back the next day. And yesterday still mattered.*
