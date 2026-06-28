# Veyne AI

**Narrative platform where the LLM is the actor, not the whole system.**

🌐 [Veyne AI site](https://katharz1z.github.io/Veyne-AI/) · 📬 [hello@veyne.ai](mailto:hello@veyne.ai)

---

## What this is

Veyne AI is an AI-powered interactive narrative platform. On the surface it looks like a character chat. Under the hood it works like a hybrid of a visual novel, a text RPG, a dating sim, and a story engine — with persistent memory, a relationship system, and structured story arcs.

Most character AI services degrade into one of three patterns: a helpful assistant wearing a costume, endless flirt with no structure, or beautiful prose that forgets everything after ten turns. Veyne is built to solve all three at the architecture level.

**The core principle:** the LLM generates only the visible character prose. Everything else — state, memory, relationship metrics, story progression, world constraints — is managed by the surrounding pipeline.

```
User message
    ↓
Context Pack Builder
    ↓
Call 1 — Character Response Generator  (LLM)
    ↓
Call 2 — State Update Extractor        (LLM)
    ↓
Reducer → New State
```

The user sees only the character's response. The rest is invisible.

---

## What makes this different

| Typical character chat | Veyne AI |
|---|---|
| LLM is the whole system | LLM is one stage in a pipeline |
| Character agrees with everything | Character has goals, resistance, limits |
| "Trust" = single number that grows fast | 6 separate relationship metrics with progression gates |
| Forgets previous sessions | Persistent multi-layer memory |
| Prompt rules enforce style | Authored playbooks and source bibles enforce narrative |
| One big prompt | Context Pack Builder selects only what's relevant per turn |

---

## Current state

**Version:** `v0.3.x` — Web Surface Sprint  
**Status:** Active development, not yet publicly playable

What exists today:

- CLI runtime with full state / reducer / relationship engine / context pack builder
- Authored Act 1 playbook (4 scenes, 5 beats, authored anchors)
- State schema v1.1 with 13 fields tracking scene, memory, relationships, story progress
- Relationship engine: trust, respect, caution, curiosity, attachment, attraction — each gated separately
- Web surface: Next.js + Supabase, character catalog, session persistence, resume
- Python FastAPI runtime adapter with package-backed execution
- Rocinante-X 12B connected via vLLM / Colab tunnel — first real LLM turns running

What's in progress:

- Full Context Pack Builder → web seam
- State extractor + reducer through the web runtime
- Clean Act 1 run end-to-end without collapse (target: 10–30 turns)

What is not public yet:

- no open playable build
- no image generation or TTS
- no finished creator marketplace
- no NSFW layer

The current proof target is simple and brutal: one saved story should continue for 10–30 turns without collapsing into a transcript dump, interrogation loop, romance leak, language drift, or state/prose mismatch.

---

## Characters and worlds

The long-term catalog target is **162 characters across 3 worlds**. That is a direction, not a launch-day magic trick. The current priority is proving that a small number of characters can hold memory, pressure, and story continuity first.

Currently in development:

**Lira Dreyvor** — Acting High Priestess of the Black Temple. 19 years old. Appointed three months ago after her predecessor's death under circumstances officially classified as an accident. Too young, too small, not enough experience — she compensates with severity, fast reaction, and absolute control over what she can control. Does not trust strangers. Does not open up fast. Does not become warm from sympathy alone.

**Adeline Veir** — Field Registrar of the Light Chancellery. 27 years old. Believes that records protect people from arbitrary violence. The problem: she has seen records become clean, lawful, very convenient sentences. Does not shout. Does not threaten first. Writes things down. That is often worse.

**Iney** — A wanderer between factions and routes, in the space where neither Light nor Dark has full control. He trades in paths, favors, and half-truths: rarely giving a straight answer, but always leaving the player with a choice that costs something later. Never on anyone's side — or on everyone's at once, which is more dangerous.

---

## Architecture

### Pipeline per turn

```
User message
    ↓
Context Pack Builder
  · Character Kernel (static)
  · Current State (dynamic)
  · Active Beat rules
  · Selected source fragments (Character Bible, World Bible, Arc Bible)
  · Relationship state + progression gates
  · Scene driver + character intent
  · Recent dialogue
    ↓
Call 1 — Character Response Generator
  · Produces visible prose in character
  · Constrained by authored playbook
  · Does not hallucinate world facts
    ↓
Call 2 — State Update Extractor
  · Extracts structured delta from what just happened
  · Updates memory, relationship metrics, story signals, scene state
    ↓
Reducer
  · Applies delta to current_state.json
  · Enforces relationship gate rules
  · Handles beat progression and exit conditions
    ↓
Next State
```

### Relationship engine

Six separate metrics. Each unlocks through different conditions. They do not move together.

```
trust       — willingness to rely on the player
respect     — recognition of player intelligence, honesty, resilience
caution     — threat-driven wariness, stays high even when trust rises
curiosity   — interest in who the player is
attachment  — locked until trust > 40 AND beat >= 4
attraction  — locked until trust > 50 AND attachment > 20 AND beat >= 5
```

Romance and NSFW are gated separately on top of these. The default product direction is SFW-first: intimacy must come from explicit user intent, story progress, relationship state, and consent logic — not from the model getting horny because a prompt sneezed.

### Memory

Memory is not a raw transcript. The system tracks:

- `known_facts` — what the character knows about the player
- `remembered_actions` — what the player said and did this run
- `unresolved_between_them` — open conflicts and threads
- `broken_promises`, `protected_or_pressured_character`

The extractor updates memory each turn. The reducer applies stale-fact cleanup. The context pack selects only what's relevant for the current call.

### World and canon

Characters do not hallucinate world facts. The source layer defines what may be true. The `knowledge_scope` and `lira_knows` fields define what is true right now in the active run. The context pack bridges them.

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
| LLM inference | OpenAI-compatible endpoint (vLLM, Colab, RunPod) |
| Current model | Rocinante-X 12B v1 |
| Target production models | Rocinante-X 12B (free tier), 20–25B model (low premium) |

---

## Roadmap

```
✅  CLI Runtime — state, reducer, relationship engine, context pack builder, Act 1 playbook
🔄  Web Runtime v0.3.x — full LLM seam, state extractor through web, clean Act 1 run
⬜  v0.9 — limited public access, text only, no images or TTS
⬜  v1.0 — public service foundation, free tier + low premium subscription, catalog expansion begins
⬜  v2.0 — 70B models, creator tools, branching Act 2, TTS
```

---

## GPU infrastructure

The project runs without external funding. Development happens on personal resources and Colab tunnels.

For stable production inference we need dedicated GPU cloud. GPU funding buys stable long-run sessions, not prettier landing pages.

- **~$400–600/month** — RunPod for Rocinante X 12B (stable inference for MVP testing and the future free tier)
- **~$600–1000/month** — 20–25B model for Low Premium tier (better prose quality and future image generation)

If you're interested in sponsoring or investing, or just want to follow along — write to [hello@veyne.ai](mailto:hello@veyne.ai).

---

## Contact

**Email:** [hello@veyne.ai](mailto:hello@veyne.ai)  
**Site:** [katharz1z.github.io/Veyne-AI](https://katharz1z.github.io/Veyne-AI/)  
**Discord:** coming soon

> *I opened the chat. But entered a world.*  
> *I wrote one line. But the character remembered ten things.*  
> *I came back the next day. And yesterday still mattered.*
