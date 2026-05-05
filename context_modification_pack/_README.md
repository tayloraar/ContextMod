> **Human-only documentation. Not part of the model's read protocol. Models should ignore this file and read `_INSTRUCTIONS.md` instead.**

# Memory Protocol — Deployment Guide

This is the human-side install guide. The protocol itself lives in `_INSTRUCTIONS.md`; this file tells you how to deploy it across different platforms, when to use it, and when to skip it.

Read this once on first install. Refer back when adding a new project, switching platforms, or troubleshooting.

---

## What this protocol is

A **behavioural mod for AI chats**. Five files (or six, in scaled mode) that sit alongside any AI chat and raise the resolution at which the model handles memory and context across long sessions. The mod doesn't touch the model. No weight changes, no API hooks, no jailbreaks. It sits on top, hooking into the model's existing file-reading capabilities through a documented protocol.

The five files:

- `_INSTRUCTIONS.md` — standard protocol rules (the model reads this every session)
- `_INDEX.md` — routing map (what's where)
- `_KNOWLEDGE.md` — stable facts (glossary, entities, constraints)
- `_STRATEGY_STATE.md` — live working state (goal, decisions, next step)
- `_README.md` — *this file* (human-only)

A sixth file ships alongside but only activates when explicitly invoked:

- `_INSTRUCTIONS_SCALED.md` — scaled variant of the protocol, for projects that have outgrown manual hot-path curation. See *When the protocol scales* below.

The model reads the four protocol files in a specific order; you don't need to manage that. Your job is to drop the files in the right place for your platform, prompt the model to read `_INSTRUCTIONS.md` (or `_INSTRUCTIONS_SCALED.md` for the scaled variant), and use the `User commands` section (`SAVE`, `COMMIT`, `STATUS`, etc.) to steer.

**This is a mod, not training data and not a save file.** Training data changes the model's weights. Save files capture state but don't change behavior. This protocol changes how the model behaves at runtime (its read patterns, write cadence, conflict resolution) without touching the model itself. Like a Skyrim mod, it's installable, uninstallable, configurable, stackable, and ecosystem-friendly. The engine is unchanged; the mod sharpens how the engine uses its existing capabilities.

**You're not the first person here.** Markdown-file-based memory for LLMs is a recognized direction the field is moving in. Manus, Basic Memory, OpenClaw, and chat.md are all in the same neighborhood. This protocol's contribution isn't the underlying idea (markdown as substrate). It's the specific synthesis of altitude structure, rotation discipline, lateral lookup with flag-and-confirm, and contradiction protocol with polysemy clusters, packaged as a portable pattern that works across deployment surfaces. If this protocol doesn't fit your workflow, others might.

---

## License

The Context Modification is open source under the **MIT License**. You can use it, fork it, modify it, embed it in your own workflow, and distribute it commercially or otherwise. The only requirement is that you preserve the copyright notice and license text in copies or substantial portions.

Full terms are in the `LICENSE` file at the pack root.

Copyright (c) 2026 A. J. Wiadrowski.

## When to use the protocol

**Use it for:**
- Long-running projects that span many sessions
- Multi-stage work with goals that need to survive compression
- Cross-chat or cross-project work where coherence matters
- Anything where you've previously lost track of decisions or context

**Skip it for:**
- One-off questions
- Single-turn tasks
- Pure execution where you have full context already

---

## When the protocol scales

The standard protocol works for most projects. Five files, three altitudes, manual hot-path curation. The substrate stays small enough to read cleanly. The model's first read of the index covers the working set without skim risk.

This breaks at scale. When a project has been running for months, the substrate has hundreds of entries, and the hot path no longer fits cleanly in one quick read, the standard protocol's manual curation becomes operational pain. You forget to add things. You forget to demote things. The model skims the long index and misses entries that are sitting right there. Retrieval starts failing in ways that are hard to diagnose because the failure mode is the model's attention, not the substrate's content.

**The scaled variant exists for that case.** Two protocols ship together. You pick which one your project uses by changing the read directive at session start.

| Variant | Read directive | When to use |
|---|---|---|
| **Standard** | `Read _INSTRUCTIONS.md and run the chat installer.` | Default. New projects, small substrates, short-running work. |
| **Scaled** | `Read _INSTRUCTIONS_SCALED.md and run the chat installer.` | Projects where the hot path is straining, where retrieval misses entries the substrate has, or where the substrate has hundreds of entries actively in use. |

**What the scaled variant adds:**

- **Hot path becomes automatic LRU.** Default capacity 300, configurable. Most-recently-retrieved entries stay accessible; oldest unpinned entries fall off when the cache fills. The first thing the model reads is now bounded and recency-ordered, so skim behaviour cooperates with the protocol instead of fighting it.
- **Pin flag for entries you don't want evicted.** `PIN [pointer]` flags an entry as exempt from LRU. `UNPIN [pointer]` releases it. Pins are a flag, not a separate compartment; they sit in the hot path alongside working-cache entries, just immune to churn.
- **Pin decay surfacing.** A pin not retrieved within the decay window (default 100 turns) gets surfaced for review at the next flush. The user decides whether to keep or unpin. No auto-eviction.
- **Graduated search on hot-path miss.** When retrieval misses the hot path, a five-stage search runs: exact match, name substring, content substring, semantic broadening, then surface failure. Failure surfaces honestly rather than getting hallucinated past, which is the strongest signal that subdivision may eventually be needed.
- **Configuration block at the top of `_INDEX.md`.** Three tunable parameters: hot-path capacity, pin decay window, search broadening on/off. Per-project overrides live here.

**What's identical:** purpose, four-layer model, altitude semantics, lateral lookup, contradiction protocol with polysemy clusters, Tier 2/3, re-anchor, pre-write-up guard, retroactive bootstrap, all destructive recovery commands. The two variants are siblings, not parent-and-child. Switching between them changes runtime behaviour, not substrate content.

**The escalation path:**

When the standard protocol stops feeling lean, before reaching for the scaled variant, try these in order:

1. **Tighten the manual hot-path curation.** Demote entries actively. Move stale routing notes to cold list. The standard protocol works longer than most users think if curation is honest.
2. **Switch to the scaled variant.** When manual curation becomes the work rather than the result, the scaled variant's automatic mechanisms take that work back.
3. **Compartmentalisation.** When the scaled variant's index management itself starts straining (graduated search broadening past stage 3 frequently, retrieval still missing entries that exist), the substrate may need subdivision into sub-files with their own indexes. This is deferred future work, mentioned here so you know the scaling road has a third lane if needed.

**Switching between variants is reversible.** Run `SAVE`. Change the read directive. The new variant picks up the existing substrate. Pins are preserved; the standard variant ignores the flag, the scaled variant respects it. Configuration block is read by the scaled variant, ignored by the standard. Substrate content doesn't migrate; only the protocol file the model reads changes.

**Most users will never need the scaled variant.** It exists for the projects that do, so the protocol can grow with them rather than capping at the standard variant's natural ceiling.

---

## Knowing when to switch chats

Every long AI session eventually hits a wall. Different platforms hit it at different points, and the same platform behaves differently across subscription tiers, operating modes, and even individual sessions. The mod can't tell you when your chat is about to degrade because the variance is too granular and changes too fast.

What the mod can do is count, and then move the work cleanly when you say so. Two pieces work together. The `Turn` field in `_STRATEGY_STATE.md > Current` increments on every Tier 1 flush. Set a threshold in `_INDEX.md > Configuration > Turn warning threshold` and the protocol surfaces a soft prompt when the count exceeds it. The prompt suggests running `SAVE` to checkpoint the substrate, then `EXPORT` to capture the recent conversation, then opening a fresh chat pointed at the same files. The substrate carries the facts; the export carries the texture. The new chat reads both and picks up where you left off.

`EXPORT` writes the last 30 turns (configurable) to `_HANDOVER.md` at chat altitude. The new chat reads the file last, after the substrate, and treats it as recent context rather than canonical fact. If anything in the export conflicts with the substrate, the substrate wins. After consuming, the file auto-archives so subsequent re-anchors don't re-read stale handover content. Use `EXPORT` for two situations: moving devices, or escaping runtime degradation on platforms without effective compression. Don't use it when the substrate itself is corrupted; that's what `REFLASH` and `PURGE` exist for. The export is in-platform only — substrate transfers cleanly across platforms, but conversational texture doesn't, because each platform's model has its own voice baseline.

Calibration is the user's call. Run your normal workflow. Notice when chats start feeling sluggish, generic, or repetitive. Set the threshold to fire ten or twenty turns before that point. Adjust as your platform changes or your work changes. If you have no baseline yet, pick a number, run with it for a few sessions, and adjust. The right number is the one that gives you a useful nudge before the chat gets heavy.

This is calibration-based, not predictive. Whatever number you set is when the prompt fires. The protocol stays platform-agnostic by design; tier-specific guidance would age out faster than this README can be updated. You know your platform and your work better than external documentation can.

---

## Choose your surface

Different platforms have different file-handling and persistence semantics. Find your platform below for the right install pattern.

| Surface | Persistence | Install method |
|---|---|---|
| **Claude Cowork** | Full filesystem (VM) | Drop files in workspace root |
| **Claude Code** | Full filesystem (terminal) | Place files in repo root or working dir |
| **Lovable.dev / vibecode** | Full filesystem (project repo) | Place files in repo root, `.gitignore` by default |
| **Claude.ai (Project)** | Project knowledge persists | Upload as project knowledge files |
| **Claude.ai (one-off)** | Session-bound | Paste contents inline; expect bootstrap each session |
| **ChatGPT (Project / Custom GPT)** | Project knowledge persists | Upload as Knowledge files; instructions in system prompt |
| **ChatGPT (one-off chat)** | Session-bound | Paste contents inline; expect bootstrap each session |
| **Gemini (Gem)** | Gem instructions persist | Instructions in Gem prompt; knowledge as uploaded reference |

---

## Per-surface install guides

### Claude Cowork

The design target. Everything works at full strength.

**Install:**
1. Drop all five files into the workspace root the chat has access to.
2. Open the chat. First message: `Read _INSTRUCTIONS.md and run the chat installer.`
3. Answer the installer's questions when prompted.

**Notes:**
- `_STRATEGY_STATE.md` lives at the chat altitude. If you have multiple chats in the same project, give each its own state file or accept they'll collide.
- Project-level files (`_INDEX.md`, `_KNOWLEDGE.md` at project altitude) live in the project subfolder; root-level versions live at the workspace root.
- All commands work at full strength.

### Claude Code

Filesystem-backed in a terminal context. Works natively.

**Install:**
1. Place the files in the repo root or your active working directory.
2. Start the chat with: `Read _INSTRUCTIONS.md and run the chat installer.`
3. Add the four protocol files to `.gitignore` by default (see Git handling below) unless you're publishing methodology.

**Notes:**
- Claude Code may suggest committing files; decline unless intentional.
- Works well for code-heavy projects where decisions and rationale otherwise live nowhere durable.

### Lovable.dev and other vibecode platforms

Filesystem-backed via the project repo. Works, but Git handling is critical here because these platforms often default to public or shared repos.

**Install:**
1. Place files in the repo root.
2. **Add to `.gitignore` immediately** (see Git handling below) before any commit.
3. Prompt the build agent: `Read _INSTRUCTIONS.md and run the chat installer.`

**Notes:**
- Project state (`_STRATEGY_STATE.md`) often contains user reasoning, draft content, and decisions that don't belong in a public repo. Default to ignoring.
- The `_KNOWLEDGE.md` file may contain proprietary canon or business logic — keep ignored unless explicitly open-sourcing.
- Build agents may try to "help" by reading or modifying these files; the protocol's read rules will keep that contained.

### Claude.ai with Projects

Project knowledge persists across chats within the project.

**Install:**
1. In the Project, upload `_INSTRUCTIONS.md`, `_KNOWLEDGE.md`, and `_INDEX.md` as project knowledge files.
2. Add to project instructions: *Read `_INSTRUCTIONS.md` and follow its protocol. Run the chat installer on first turn of new chats.*
3. `_STRATEGY_STATE.md` is per-chat — the model maintains it as a Canvas/Artifact in each chat.
4. `_README.md` does not need uploading (human-only).

**Notes:**
- Updates to `_KNOWLEDGE.md` or `_INDEX.md` require re-uploading the file to project knowledge — not as fluid as filesystem-backed.
- `STATUS`, `COMMIT`, `SAVE`, `SEARCH WIDE`, `GUIDE`, `HELP`, `GRADUATE` work at full strength. `REWIND`, `PURGE`, `REFLASH` are best-effort within a session.

### Claude.ai one-off chats (no Project)

Session-bound. Bootstrap each time.

**Install:**
1. Paste `_INSTRUCTIONS.md` contents at the start of the chat.
2. Paste current `_KNOWLEDGE.md` and `_INDEX.md` if you have them from prior work.
3. Prompt: *Run the chat installer.*
4. Export the artifact contents at end of session if you want to carry state forward.

**Notes:**
- This pattern wastes the protocol's compression-survival benefits. Use Projects instead if available.
- Recovery commands (`REWIND`, `PURGE`) are best-effort only.

### ChatGPT with Projects or Custom GPTs

Similar to Claude Projects.

**Install:**
1. In the Project or Custom GPT, paste `_INSTRUCTIONS.md` content into the system prompt or instructions field.
2. Upload `_KNOWLEDGE.md` and `_INDEX.md` as Knowledge files.
3. Add to instructions: *Run the chat installer on first turn of new chats. Maintain `_STRATEGY_STATE.md` as a Canvas document.*

**Notes:**
- ChatGPT's built-in Memory feature may try to "help" by remembering across chats. Either disable Memory in the GPT or accept some duplication.
- Canvas behaves like a real file within a session; the protocol works as designed.
- All commands work at full strength within a session; recovery degrades across sessions.

### ChatGPT one-off chat

Same as Claude.ai one-off — paste contents inline, bootstrap each session, export to carry forward.

### Gemini (Gem)

**Install:**
1. Paste `_INSTRUCTIONS.md` content into the Gem's instructions.
2. Upload `_KNOWLEDGE.md` and `_INDEX.md` as reference files.
3. State explicitly in the Gem instructions: *Run the chat installer on first turn. Maintain `_STRATEGY_STATE.md` inline as a regenerated block each turn.*

**Notes:**
- Gemini's instruction adherence on multi-step protocols varies more than Claude/ChatGPT. Run a few sessions and watch whether the Tier 1 checklist actually fires.
- If checklist compliance drops, tighten the phrasing in `_INSTRUCTIONS.md` to be more imperative for Gemini specifically.

---

## Git handling

**Default: ignore the protocol files.**

The four protocol files contain working knowledge, in-flight reasoning, decisions logs, and routing maps. For most projects, especially anything in a public or team repo, these should not be committed.

**Sample `.gitignore` entries:**

```gitignore
# Memory protocol files — local working knowledge, not source code
_INSTRUCTIONS.md
_INDEX.md
_KNOWLEDGE.md
_STRATEGY_STATE.md
_README.md

# Snapshots created by SAVE command (if your platform creates them)
.snapshots/
```

**Opt-in (commit the files) when:**
- You're explicitly open-sourcing your methodology / "vibecoding behind the scenes"
- The project is solo and the repo is private
- You want collaborators to share the protocol substrate
- The project IS the protocol (e.g., publishing this README)

If opting in, consider committing only `_INSTRUCTIONS.md` (universal protocol) while keeping `_KNOWLEDGE.md`, `_INDEX.md`, and `_STRATEGY_STATE.md` ignored — that shares the methodology without exposing project-specific working state.

---

## Command degradation by surface

Not all commands work equally well on every platform. This table tells you what to expect.

| Command | Filesystem-backed | Project-persistent | Session-bound |
|---|---|---|---|
| `SAVE` | Full | Full within session | Session-only |
| `COMMIT` | Full | Full | Session-only |
| `STATUS` | Full | Full | Full |
| `SEARCH WIDE` | Full | Full | Limited to attached files |
| `PROMOTE` | Full | Full | Session-only |
| `GRADUATE` | Full | Full | Full |
| `HELP` | Full | Full | Full |
| `GUIDE` | Full | Full | Full |
| `REWIND` | Full | Best-effort | Best-effort |
| `PURGE` | Full | Best-effort | Best-effort |
| `REFLASH` | Full | Full (clears local) | Full (clears canvas) |
| `UNINSTALL` | Full | Full | Full |

"Best-effort" means the command runs from in-context state rather than true rollback. It usually works for the most recent change but degrades quickly with depth.

### Confirmation tiers for destructive commands

The four recovery commands have two tiers of safeguard before they execute:

- **Standard confirm** (`REWIND`, `PURGE`): the model states what will be removed and asks `yes` / `no`. A simple affirmative proceeds.
- **Typed confirm** (`REFLASH`, `UNINSTALL`): the model states what will be removed and asks you to type a specific phrase — `CONFIRM REFLASH` or `CONFIRM UNINSTALL`. Bare "yes" or "go ahead" is **not** sufficient. This prevents accidental triggering, since these commands either wipe your chat substrate or remove the protocol entirely.

If the typed confirmation phrase doesn't match exactly, the model aborts and asks again. This is intentional — close-but-not-exact ("confirm refresh", "yes uninstall") doesn't proceed.

For `UNINSTALL` specifically, the model will also ask whether you want the four protocol files archived to `.uninstalled/` (recoverable later) or deleted outright. Default is archive on filesystem-backed surfaces; on canvas-only surfaces, the canvas documents are removed and there's no archive option.

---

## First-session walkthrough

What to expect rotation 1–10 across surfaces. **Be ready for the first 10 rotations to feel heavier than steady state — this is by design.** The protocol asks more of you up front so it asks less of you later. If the friction isn't paying back for your workload, you have two clean exits: `GRADUATE` to skip ahead to lighter mode, or `UNINSTALL` to remove the protocol entirely.

**Rotation 1** — Chat installer fires (or you skip it). Goal captured. Inherited context surfaced from higher altitudes if any. `Rotation: 1 of 10` set. The model should announce roughly: *"The first 10 rotations run heavier — more phrasing capture and confirmations. Run `GRADUATE` to skip ahead, or `UNINSTALL` to remove the protocol."*

**Rotations 2–9** — Heightened discipline:
- Phrasing capture is full on all logged decisions
- Lateral reads (cross-project) always surface for confirmation
- Promotion decisions surface ("should this go to project altitude?")
- If `GUIDE` is on, occasional inline tips will appear

**Rotation 10** — Last heightened-discipline rotation.

**Rotation 11 (graduation flush)** — One-time consolidation: phrasing-captured locks lift to `_KNOWLEDGE.md > Constraints` at the right altitude, log entries become paraphrased pointers, counter switches to "graduated — steady state." Model announces it.

**Steady state** — Lighter discipline:
- Phrasing only on lock-creating decisions
- Lateral reads only surface when ambiguous
- Promotion judgment is silent for clear cases

**Early exits at any time:**
- `GRADUATE` — skip the heavy phase. Use when you'd rather scaffold knowledge yourself and correct things as they come up rather than have the protocol build out exhaustively. Common case: experienced users on familiar workloads.
- `UNINSTALL` — remove the protocol entirely. Use when this isn't the right tool for your workload. The protocol files get archived to `.uninstalled/` (or removed entirely if you confirm), and the chat continues as a normal AI chat. No regret cost.

---

## Common gotchas

**Voice contamination from sibling projects.** If drafting Project A and the model "wanders" into Project B's voice, sideways reads have leaked. Recovery: `REWIND` if recent, `PURGE` if accumulated. Prevention: keep universal voice rules at root altitude in `_KNOWLEDGE.md > Constraints`.

**Missed laterals.** Model didn't surface a sibling-project hit during an early-session lookup. Usually means lateral-read confirmation wasn't enforced. Check `_STRATEGY_STATE.md > Current > Rotation` — if it's already graduated, the surfacing rule has eased. Use `SEARCH WIDE` explicitly when you suspect a connection exists.

**Over-promotion.** Project `_INDEX.md` cluttering with chat-specific noise. Means the promotion check is firing too liberally. Tune the heuristic in `_INSTRUCTIONS.md` toward "promote only when referenced in two or more chats."

**Under-promotion.** Same alias re-derived in five chats. Opposite problem. Tune toward "promote on second mention across chats."

**Command misfires.** Capitalized words in normal prose accidentally triggering commands. The protocol requires standalone tokens — adjust if needed by adding a prefix like `/SAVE` instead of bare `SAVE` for your projects.

**README drift.** A model accidentally reads this file and gets oriented to it instead of `_INSTRUCTIONS.md`. The protocol has three defenses (filename convention, masthead, exclusion rule), but if it happens, prompt: *Ignore `_README.md`. Read `_INSTRUCTIONS.md` from the top.*

**Compressed-away decisions.** Bootstrap can only reconstruct from what's currently readable. Decisions made before compression with no surviving summary are gone. Mitigation: more frequent `SAVE` calls, especially before risky moments.

**Substrate is only as good as the curator.** The protocol's reliability is upper-bounded by the quality of what's written into the substrate. Models will preferentially trust structured input — that's the whole point. If a `_KNOWLEDGE.md > Constraints` entry is wrong, the protocol amplifies that error to the same reliability with which it would amplify a correct one. The failure mode shifts from "model hallucinated" to "user wrote down something wrong, and the protocol now treats it as canonical." Locked phrasing in the substrate becomes load-bearing in a way unstructured context does not. Discipline cost: every conflict the contradiction protocol surfaces requires user attention and honest resolution. Resolving lazily — accepting whatever the user just said as canonical — corrupts the substrate. The protocol is not a license to stop checking your own work; it's a structure that makes your work checkable.

**Polysemy hoarding.** The contradiction protocol's differentiation branch creates polysemy clusters when the same term legitimately means multiple things. Used genuinely, this scales; used lazily as a way to avoid resolving real contradictions, it accumulates near-duplicate entries and chokes the index with disambiguation noise. Differentiation is intentionally the most expensive branch (four writes per resolution) to discourage abuse. If your `_INDEX.md > Polysemy clusters` is growing faster than your project's actual conceptual divergence, you're hoarding. Roll back recent clusters and resolve as supersession or correction instead.

---

## Adding a new project

When adding a new project to an existing protocol setup:

1. Create the project folder under root.
2. Copy template versions of `_INDEX.md` and `_KNOWLEDGE.md` to the project folder. (No `_INSTRUCTIONS.md` — that lives only at root.)
3. New chats started in that project will read root → project → chat altitudes automatically.
4. Run the chat installer on the first chat, identifying any sibling projects to flag for cross-coherence.

## Adding a new chat to an existing project

1. Open the chat with access to the project folder.
2. First message: *Read `_INSTRUCTIONS.md` and run the chat installer.*
3. Answer installer questions. Inherited context flows down from project and root.
4. Begin work.

---

## Benchmarking — community callout

The protocol has not been formally benchmarked. The author built it for a real workload (multi-novel publishing project) and observed qualitative improvements in long-session coherence, post-compression recovery, and decision durability. Whether those translate to measurable token savings, output quality lift, or compute cost reduction across other models and workloads is an open question.

**If you want to help answer it, here's what useful data looks like.**

### A. Same-task A/B comparison

Run the same plan-heavy task twice on the same model, once with the protocol installed and once without. Suggested task shapes:

- **Multi-stage research compilation** — synthesize 5+ sources into a structured report
- **Code refactor across multiple files** — non-trivial, requires holding decisions across turns
- **Long-form drafting** — outline → draft → revise across a single deliverable
- **Multi-session workflow** — spread the same task across 3+ sessions with deliberate breaks

Capture per run:

- Total tokens consumed (input + output)
- Number of turns to completion
- Whether compression fired during the run (and how the model recovered if so)
- Output quality on a fixed rubric (you choose; consistency matters more than the rubric itself)
- Number of times you had to re-explain context or correct contradictions

### B. Cross-model comparison

Run the same workload across model tiers (e.g. Haiku / Sonnet / Opus, or GPT-4 / Claude / Gemini) with and without the protocol. Hypothesis worth testing: smaller models gain *more* than larger ones because the protocol externalizes working memory, which helps where capacity is tightest. The author predicts but cannot prove this.

### C. Long-running compression survival

Set up a session designed to hit compression at least once (long context, lots of tool use, large outputs). Measure recovery quality on the next turn after compression — does the model know what was decided 50 turns ago?

### D. What to publish

If you run any of the above, a short write-up with raw numbers is more valuable than a polished essay. Format suggestions:

- Model + version
- Workload description (one paragraph)
- Method (with vs without protocol; how many runs)
- Numbers (tokens, turns, errors, subjective scores)
- One-paragraph takeaway

Tag the author or post in the issues of the public repo so others can find it. Reproducibility beats authority — "I ran this, here are my numbers" carries more weight than "the protocol works."

### Caveats worth naming

- The protocol's biggest wins are likely on long-running, plan-heavy, multi-session workloads. It will look like overhead on short tasks. Benchmark workloads should match the protocol's intended use shape.
- Token efficiency is platform-dependent — surfaces that always reload context lose the scoped-read benefit. Intent durability is universal.
- Models vary in how reliably they follow the Tier 1 mandatory checklist. Compliance is itself a variable; observe it before drawing conclusions about the protocol.
- This protocol is not magic. If your workflow is fine without it, you don't need it.

---

## Updating the protocol

The `_INSTRUCTIONS.md` file evolves over time. When you change it:

1. Edit at root altitude only — there should be one canonical copy.
2. Active chats may continue with the old version until they re-anchor or restart.
3. To force-update an active chat, prompt: *Re-read `_INSTRUCTIONS.md` and apply changes.*
4. Consider a `SAVE` before doing this — protocol changes can interact unexpectedly with in-flight state.

---

## Reaching out

This protocol is a working system, not a finished product. Expect to tune thresholds, adjust the rotation count, modify the promotion heuristic, and add commands as your usage matures. The protocol is meant to evolve with you — `_INSTRUCTIONS.md` is editable for exactly that reason.

When something doesn't work, the diagnostic order is usually:

1. Check `STATUS` — is the rotation counter where you expect?
2. Check `_INDEX.md` hot path — is routing pointing at the right substrate?
3. Check `_KNOWLEDGE.md > Constraints` at relevant altitudes — are locks captured?
4. If state looks corrupted, escalate: `REWIND` → `PURGE` → `REFLASH`.

Most issues come from the protocol not actually being run (the model skipped a step) rather than the protocol being wrong. `STATUS` is your first check.


---

*Part of The Context Modification, MIT licensed. Copyright (c) 2026 A. J. Wiadrowski. See `LICENSE` in the pack root for full terms. This file alone is incomplete; the pack lives at continuityconstellation.com.*

