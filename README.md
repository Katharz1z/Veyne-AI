# Veyne AI

**Narrative platform where the LLM is the actor, not the whole system.**

🌐 [Veyne AI site](https://katharz1z.github.io/Veyne-AI/) · 📬 [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com)

---

## What this is

Veyne AI is an AI-powered interactive narrative platform. On the surface it looks like a character chat. Under the hood it works like a hybrid of a visual novel, a text RPG, a dating sim, and a story engine — with persistent memory, relationship state, structured story arcs, world pressure, progression gates, and playable consequences.

Most character AI services degrade into one of three patterns: a helpful assistant wearing a costume, endless flirt with no structure, or beautiful prose that forgets everything after ten turns. Veyne is built to solve all three at the architecture level.

**The core principle:** the LLM writes the visible character prose. The surrounding runtime carries the story: state, memory, relationship metrics, progression, world constraints, scene drivers, source truth, and guardrails.

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

The player sees the character response. The product remembers the rest.

---

## What is already updated and ready to inspect

**Public WIP checkpoint:** `r1.40` — Rocinante live runtime path + positive public surface refresh  
**Product stage:** active development, sponsor/collaborator-ready WIP, not an open public release yet

Ready today:

- **Web product shell** — landing, character catalog/profile, auth shell, saved sessions, resume surface, chat UI.
- **Session model** — Supabase-backed UUID sessions, persisted transcript, runtime events, state snapshots, local fallback for prototype work.
- **Runtime bridge** — Next runtime route talks to a Python FastAPI adapter instead of baking behavior directly into React.
- **Package-backed execution** — runtime packages for `lira-dreyvor` and `adeline-veir` can be loaded behind the adapter.
- **Rocinante live path** — Rocinante-X 12B is connected through an OpenAI-compatible vLLM / Colab route.
- **Live turn delivery** — generated LLM responses can reach the web UI and persistence layer.
- **Guard / repair layer** — visible output is checked for Russian language, dead-end hidden-lore questions, unsupported inventions, and missing playable affordance.
- **Source architecture** — Founder Vision, Project Vision, World/Character/Arc Bibles, Context Pack rules, State Schema, TurnPlan patterns, and testing docs are consolidated into an active source set.

This is no longer just a concept page or CLI toy. The core product loop now has a web surface, a runtime boundary, persistence, package loading, and a live character-model seam.

---

## Latest runtime signal

The current live stack has crossed the important integration line:

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

That means the main seam is alive: the web app can route a turn to the Python adapter, call the RP model path, receive generated prose, and persist the result.

The next engineering pass is about **runtime control quality**: making sure beautiful prose also completes the correct scene move, aligns with committed state, and leaves the player with a playable next action. In less polite terms: not letting the model paint a gorgeous mural while quietly kicking the state machine in the teeth.

---

## What makes this different

| Typical character chat | Veyne AI |
|---|---|
| LLM is the whole system | LLM is one stage inside a narrative runtime |
| Character agrees with everything | Character has goals, resistance, limits, and pressure |
| "Trust" is one number that grows fast | Six relationship metrics with progression gates |
| Forgets previous sessions | Persistent session state, memory, resume, and runtime events |
| Prompt rules enforce style | Source bibles, authored playbooks, TurnPlans, and guards enforce narrative |
| One giant prompt | Context Pack Builder selects what matters for the current turn |

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
✅ CLI runtime foundation
   State, reducer, relationship engine, context pack builder, Act 1 playbook.

✅ Web surface skeleton
   Character pages, auth shell, sessions, chat, persistence, resume.

✅ Python runtime bridge
   FastAPI adapter, package-backed execution, Rocinante-compatible LLM seam.

✅ Public WIP surface refresh
   README and GitHub Pages now describe what is built and ready to inspect.

🔄 Live runtime control
   Beat 2/3 scene-move alignment, state/prose sync, playable next action.

⬜ MVP continuity proof
   One saved Act 1 story, 10–30 turns, no collapse.

⬜ v0.9 — limited public access
   Text only, no images/TTS, early testers.

⬜ v1.0 — service foundation
   Free tier + low premium, catalog expansion begins.

⬜ v2.0 — platform expansion
   70B models, creator tools, branching Act 2, TTS.
```

---

## What funding unlocks

The project currently runs on personal resources and temporary Colab tunnels. Stable inference needs dedicated GPU cloud. GPU funding buys stable long-run sessions, not prettier landing pages.

- **~$400–600/month** — RunPod or equivalent for Rocinante-X 12B, stable inference for MVP testing and the future free tier.
- **~$600–1000/month** — 20–25B model for Low Premium tier, better prose quality and future image generation experiments.

If you're interested in sponsoring or investing, or just want to follow along — write to [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com).

---

## Contact

**Email:** [veyne.ai@gmail.com](mailto:veyne.ai@gmail.com)  
**Site:** [katharz1z.github.io/Veyne-AI](https://katharz1z.github.io/Veyne-AI/)  
**Discord:** coming soon

> *I opened the chat. But entered a world.*  
> *I wrote one line. But the character remembered ten things.*  
> *I came back the next day. And yesterday still mattered.*
