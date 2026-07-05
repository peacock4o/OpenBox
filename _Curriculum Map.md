---
tags:
  - node/index
---

# ⧉ OpenBox — Curriculum Graph

Open **Graph View** (left ribbon or `Ctrl/Cmd-G`). Colors and arrows are preconfigured.

## How to read the graph

Two node types: **lessons** (`lessons/*.md`, one per video) and **concepts** (`concepts/*.md`, one per topic — `jab`, `stance`, `combo`, ...). A lesson links out to concepts two ways:

- `## ⬆ Preconcepts` — what this lesson requires you already know.
- `## ⬇ Postconcepts` — what this lesson contributes to.

Concepts never link out; they're pure computed hubs — `## Producers` (lessons that list them as a postconcept) and `## Consumers` (lessons that list them as a preconcept) are rebuilt from scratch on every sync.

**Node colors** (node categories):

- 🟣 **hub** — a concept node (`concept__*`)
- 🔵 **theory** — concept explainers
- 🟢 **drill** — isolation practice
- 🟠 **tactic** — combinations / strategy / live sparring
- ▪️ **misc** — conditioning, warm-up, gear, analysis, channel/meta content

## Tiers

`tier` on each lesson is computed, not authored — see `_templates/Sync Graph From JSON.md`. Root (tier 0) = a lesson whose entire preconcepts/postconcepts is the same single concept (a self-contained lesson, not really dependent on anything else). Everything else is tiered by breadth-first search: the first time a lesson's preconcept is produced by an already-tiered lesson, it gets that lesson's tier + 1. To change a lesson's place in the graph, edit its `preconcepts`/`postconcepts` in `_data/lessons.json`, not the tier directly.