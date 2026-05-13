# _INDEX.md — Live Routing Map

> **The indexer. Maps what's linked to what *right now*. Not facts (those live in `_KNOWLEDGE.md`). Not reasoning (that lives in `_STRATEGY_STATE.md`). Just the lookup table that makes the other two cheap to read.**

## How to use this file

- Read **after** `_INSTRUCTIONS.md` and **before** `_KNOWLEDGE.md` and `_STRATEGY_STATE.md`'s deeper content.
- The hot path tells re-anchor which `_KNOWLEDGE.md` sections to load eagerly.
- The alias table resolves shorthand to canonical references — check it before searching `_KNOWLEDGE.md`.
- Update on routing changes only — not on every chunk, not on new facts.

---

<!-- Optional configuration. Both protocol variants read what's relevant to them. Uncomment and adjust as needed.
## Configuration

# Used by both standard and scaled variants:
- Turn warning threshold: <N> (default off; set a number to enable a soft prompt at that turn count)
- Turn warning renag interval: 20 turns (default 20; controls how often the warning re-surfaces if dismissed)
- Export capture window: 30 turns (default 30; how many recent turns EXPORT writes to _HANDOVER.md)

# Used by scaled variant only:
- Hot path capacity: 300 (default 300)
- Pin decay window: 100 turns (default 100)
- Search broadening: enabled (default enabled)
-->

## Hot path

`_KNOWLEDGE.md` sections in active rotation this session. Most-recent / most-frequently cited at top.

- `_KNOWLEDGE.md > [section]` — _why it's hot_
- 

## Read-state tracker

Coverage state for substrate-grade material in this project. Records what the model has actually read, not what has been mentioned. The model honours the read-state when answering: entries marked `[skim]` or `[partial]` trigger Tier 1 refusal-on-drift before the model produces output that depends on them.

States:
- `[skim]` — model has seen file metadata and at most a sample. Cannot speak with confidence about contents.
- `[partial: §X]` — model has read specific sections fully (e.g., `[partial: chapters 1-3]`). Can speak with confidence about read sections only.
- `[full]` — model has read the file in its entirety via the chunked-read procedure.

| Source material | Path | State | Notes |
|---|---|---|---|
| _e.g. The Trust Hallucination_ | `/novels/TTH_chapters/` | `[skim]` | _8 chapters, ~80k words, not yet ingested_ |
|  |  |  |  |

Update this table after each chunked-read pass. The model never advances a state from `[skim]` to `[full]` without having actually executed the deep-read.

## Aliases

Shorthand → canonical reference. Resolve before lookup.

| Shorthand | Canonical reference |
|---|---|
|  |  |

## Cross-references

Links between items in `_STRATEGY_STATE.md` and the `_KNOWLEDGE.md` sections that support them. Updated when a decision or open question gains a stable knowledge dependency.

- **Decision [ref]** ⇄ `_KNOWLEDGE.md > [section]`
- **Open question [ref]** ⇄ `_KNOWLEDGE.md > [section]`

## Cold list

Sections explicitly *not* relevant this session. Re-anchor skips these even if they're cited elsewhere. Prevents stale routing from pulling in irrelevant knowledge.

- 

## Polysemy clusters

Terms that legitimately carry multiple distinct meanings, with context-resolution rules. Cluster head sits here; sense entries live in `_KNOWLEDGE.md` with scope qualifiers in entry names.

| Cluster head | Senses | Context resolution |
|---|---|---|
|  |  |  |

Lookup of a polysemous term hits the cluster head first. If chat context resolves the sense unambiguously, proceed to that sense. If context is genuinely ambiguous, surface all senses and ask the user.

## Routing notes

Project-specific shortcuts, recurring patterns, gotchas about how this session navigates the files.

- 


---

*Part of The Context Modification, MIT licensed. Copyright (c) 2026 A. J. Wiadrowski. See `LICENSE` in the pack root for full terms. This file alone is incomplete; the pack lives at continuityconstellation.com.*

