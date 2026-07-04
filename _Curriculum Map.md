---
tags:
  - node/index
---

# ⧉ OpenBox — Curriculum Graph

Open **Graph View** (left ribbon or `Ctrl/Cmd-G`). Colors and arrows are preconfigured.

## How to read the graph

**Arrow direction** — an arrow `A → B` means *A requires / belongs to B*. Arrows therefore flow **toward the foundations** (warm-up, stance). Follow them backwards to find your watch-order.

**Node colors** (node categories):

- 🟣 **hub** — a facet grouping node (`facet__*`)
- 🔵 **theory** — concept explainers
- 🟢 **drill** — isolation practice
- 🟠 **application** — combinations / tactics (multi-prerequisite)
- 🟡 **sparring** — live observation
- 🩵 **conditioning**, ⚪ **warm-up**, 🔴 **analysis**, 🟤 **gear**, ▪️ **meta**

**Edge kinds** (Graph View can't color edges natively, so kind is shown by the *section* a link sits under):

- `member-of` — video → its facet hub
- `prereq` — facet hub → upstream facet hub (the backbone DAG)
- `cross-link` — application video → several facet hubs (the emergent multi-parent edges)

> [!tip] Want literally colored/labelled edges?
> Stock Graph View can't do it. Install **Juggl** (typed edges) or **Breadcrumbs** (hierarchical fields). The `type` / `kind` frontmatter and typed sections here are already compatible starting points.

## Tiers (facet hubs by watch-order)

**Tier 0** — [[facet__analysis|analysis]], [[facet__gear|gear]], [[facet__meta|meta]], [[facet__warmup|warmup]]
**Tier 1** — [[facet__cond|cond]], [[facet__stance.T|stance.T]]
**Tier 2** — [[facet__block.T|block.T]], [[facet__foot.T|foot.T]], [[facet__punch.T|punch.T]], [[facet__stance.D|stance.D]]
**Tier 3** — [[facet__foot.D|foot.D]], [[facet__hook.T|hook.T]], [[facet__jab.T|jab.T]], [[facet__punch.D|punch.D]], [[facet__punchNuance.T|punchNuance.T]], [[facet__slip.T|slip.T]], [[facet__upper.T|upper.T]]
**Tier 4** — [[facet__block.D|block.D]], [[facet__cross.T|cross.T]], [[facet__hook.D|hook.D]], [[facet__jab.D|jab.D]], [[facet__pend.T|pend.T]], [[facet__slip.D|slip.D]], [[facet__upper.D|upper.D]]
**Tier 5** — [[facet__bag|bag]], [[facet__defvs|defvs]], [[facet__onetwo.T|onetwo.T]], [[facet__pend.D|pend.D]]
**Tier 6** — [[facet__combo|combo]], [[facet__distance|distance]], [[facet__onetwo.D|onetwo.D]]
**Tier 7** — [[facet__counter|counter]], [[facet__vs_short|vs_short]], [[facet__vs_tall|vs_tall]]
**Tier 8** — [[facet__pressure|pressure]]
**Tier 9** — [[facet__sparring|sparring]]