# Scenario Approach — Process Notes
### How each scenario deliverable is built

---

## Inputs

Every scenario is built from two sources:

1. **Task Brief** — defines the deliverables (approach explanation, TO-BE workflow update, visuals, rules/validations, improvements, one-page process summary) and the reviewer's constraint: brief, well-processed, no long AI-generated prose.
2. **Scenario description** — the one- or two-line operational problem as stated by the client.

The scenario description is always treated as a starting question, not a complete specification.

---

## Step 1 — Reframe the problem

The stated scenario is usually a symptom, not the real constraint. Before generating options, the problem is rewritten by layering in the full DPO context: who the actors are, what they can and can't see, what data exists today, and where the pain actually lives.

The reframing is tested with one question: *"If I solved only the literal request, would the underlying problem still exist?"* If the answer is yes, the framing isn't done.

Every later design choice is anchored to the reframed problem, not the original sentence.

---

## Step 2 — Generate options broadly

Multiple candidate mechanisms are drafted before any is evaluated. The goal at this stage is coverage, not quality — including options that are clearly weak, because they sharpen the trade-offs for the ones that survive.

Each option is pressure-tested against the same small set of questions:

- Does it violate any hard constraint (confidentiality, commercial model, client contract)?
- How does it behave at the performance edges (under-delivery and over-delivery)?
- Does it require data, integrations, or vendor behavior we don't actually have?
- Can it be explained to a stakeholder in two sentences?

Options that fail any of these are cut, but the reasoning is kept — it's what makes the final recommendation defensible.

---

## Step 3 — Combine, don't just pick

The strongest scenarios rarely come from a single mechanism. A primary mechanism handles the 95% case cleanly; a secondary mechanism acts as a safety net for the edge cases the primary can't cover. The combination is named explicitly so the approach is referable in conversation.

---

## Step 4 — Surface corner cases before drafting

Before writing any deliverable, the unresolved decisions are listed explicitly. These are the questions where reasonable people could disagree and where the answer materially changes how the approach reads to a stakeholder.

Each corner case is presented to the decision-maker with trade-offs, not buried inside a recommendation. Answers are locked in before drafting, so the final deliverable reads as decisive rather than exploratory.

Factual questions about the current system (how things work *today*) are separated from design choices. Design choices are decided; factual questions are flagged as open items for the platform team, in line with the reviewer's instruction to avoid assumptions.

---

## Step 5 — Write for the audience

Each scenario deliverable is written for an **executive audience** unless specified otherwise. Structural conventions:

- **One-sentence problem statement** at the top so the reader doesn't reconstruct context.
- **Named mechanisms** so the approach can be referenced by name later.
- **"Why this works" as prose**, not bullets — stakeholder narratives read better than lists.
- **Operating rules as a table** — the only place where density matters more than flow.
- **Open questions isolated at the bottom**, visible and unmissable.

Length target: one page rendered. Anything that doesn't earn its place is cut. The writing pass is where most of the brevity is achieved, not the drafting pass.

---

## Step 6 — Document the process alongside the deliverable

A short process note accompanies each scenario deliverable, describing how the approach was built, which corner cases were surfaced, what the decision-maker locked in, and which tools were used. This is deliverable #6 from the Task Brief and exists so the reader can audit the reasoning, not just the conclusion.

---

## Tools used (standard across scenarios)

- **Claude (Cowork mode)** — problem reframing, option generation, trade-off analysis, clarifying-question structuring, drafting, formatting.
- **User (decision-maker)** — locks in corner-case decisions, overrides recommendations where business context demands it.
- **Source materials** — Task Brief, scenario descriptions, and existing DPO platform research notes and documentation.

No other AI or automation tools are used unless explicitly noted in a scenario's own process file.
