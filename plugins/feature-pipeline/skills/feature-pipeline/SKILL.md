---
name: feature-pipeline
description: >
  Multi-agent PM → Business Analyst → Architect → Developer pipeline for feature development.
  PM writes a product description, Business Analyst writes a functional spec, Architect designs
  the technical solution, Developer implements. Q&A loops: PM↔User, Business Analyst↔PM,
  Architect↔Business Analyst, Developer↔Architect — each agent asks the previous when clarification
  is needed, max 3 iterations per stage. All docs saved as markdown in ./agent-docs/.
  Trigger: /feature-pipeline <feature description>, "implement feature X with agents",
  "build feature with PM and business analyst", "use agent pipeline for X".
user-invocable: true
---

# Feature Pipeline

Orchestrate a `product-manager` → `business-analyst` → `architect` → `developer` pipeline.
Spawn four sub-agents in sequence. Each writes a markdown doc to `./agent-docs/`.
Q&A loops resolve ambiguity before advancing to the next stage.

---

## Setup

Before spawning any agent:

1. Run `pwd` — bind the result to `$PROJECT_ROOT`. Pass this absolute path literally in every agent prompt.
2. Get feature description: use `$ARGUMENTS` if provided; otherwise use `AskUserQuestion`.
3. Run `mkdir -p $PROJECT_ROOT/agent-docs`.
4. Check `ls $PROJECT_ROOT/agent-docs/project-*.md` — if a similar feature doc exists, ask: "A PM doc for a similar feature exists. Overwrite it, use a different name, or resume from the existing doc?" Resume = skip PM stage, use existing doc.

---

## Stage 1 — PM Agent

Set `pm_iter = 0`.

### Spawn PM Agent

Use the Agent tool with `subagent_type: product-manager`. Prompt:

```
Project root: $PROJECT_ROOT
Feature request: $FEATURE_DESC

Write the PM doc for this feature. Read README.md and CLAUDE.md for product context.
Save to: $PROJECT_ROOT/agent-docs/project-{slug}.md
Use the doc format defined in your instructions.
```

### PM Q&A Loop (max 3 iterations)

After agent finishes:
1. Read `$PROJECT_ROOT/agent-docs/project-{slug}.md` (read `## Slug` section first to get slug).
2. Strip whitespace from `## Questions` body. If exactly `_None_` → advance to Stage 2.
3. While `pm_iter < 3` and Questions ≠ `_None_`:
   - Ask all questions to the user in one `AskUserQuestion` call (numbered list).
   - Spawn PM Refinement (`subagent_type: product-manager`):
     ```
     Project root: $PROJECT_ROOT
     PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md

     Refine the PM doc based on these user answers:
     {paste numbered answers verbatim}

     Update relevant sections. If questions are resolved, set ## Questions to _None_.
     ```
   - Increment `pm_iter`. Re-read doc. Check `## Questions` again.
4. If `pm_iter == 3` and Questions ≠ `_None_`: proceed. Tell user: "Maximum PM Q&A iterations reached — proceeding with open questions noted in the doc."

---

## Stage 2 — Business Analyst Agent

Set `analyst_iter = 0`.

### Spawn Business Analyst Agent

Use the Agent tool with `subagent_type: business-analyst`. Prompt:

```
Project root: $PROJECT_ROOT
PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md

Write the functional spec for this feature based on the PM doc.
Save to: $PROJECT_ROOT/agent-docs/analyst-{slug}.md
Use the doc format defined in your instructions.
```

### Business Analyst Q&A Loop (max 3 iterations)

After agent finishes:
1. Read `$PROJECT_ROOT/agent-docs/analyst-{slug}.md`.
2. Strip whitespace from `## Questions for PM` body. If exactly `_None_` → advance to Stage 3.
3. While `analyst_iter < 3` and Questions for PM ≠ `_None_`:
   - Spawn PM Refinement (`subagent_type: product-manager`):
     ```
     Project root: $PROJECT_ROOT
     PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md

     The business analyst has questions about your PM doc:
     {paste "Questions for PM" section body verbatim}

     Answer them by updating the relevant sections. Set ## Questions to _None_.
     ```
   - Spawn Business Analyst Refinement (`subagent_type: business-analyst`):
     ```
     Project root: $PROJECT_ROOT
     Updated PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
     Current functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md

     Your questions to the PM have been answered. Read the updated PM doc and revise
     the spec sections that need updating. Clear ## Questions for PM to _None_
     if all questions are resolved, or write new ones if the answers raised new issues.
     ```
   - Increment `analyst_iter`. Re-read analyst doc. Check `## Questions for PM` again.
4. If `analyst_iter == 3` and Questions ≠ `_None_`: proceed. Tell user: "Maximum Business Analyst Q&A iterations reached — proceeding with remaining questions noted in the spec."

---

## Stage 3 — Architect Agent

Set `architect_iter = 0`.

### Spawn Architect Agent

Use the Agent tool with `subagent_type: architect`. Prompt:

```
Project root: $PROJECT_ROOT
PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
Functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md

Design the technical solution for this feature. Explore the codebase after reading both docs.
Save to: $PROJECT_ROOT/agent-docs/architect-{slug}.md
Use the doc format defined in your instructions.
```

### Architect Q&A Loop (max 3 iterations)

After agent finishes:
1. Read `$PROJECT_ROOT/agent-docs/architect-{slug}.md`.
2. Strip whitespace from `## Questions for System Business Analyst` body. If exactly `_None_` → advance to Stage 4.
3. While `architect_iter < 3` and Questions for System Business Analyst ≠ `_None_`:
   - Spawn Business Analyst Refinement (`subagent_type: business-analyst`):
     ```
     Project root: $PROJECT_ROOT
     PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
     Current functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md

     The architect has questions about your functional spec:
     {paste "Questions for System Business Analyst" section body verbatim}

     Answer them by updating the relevant sections. Set ## Questions for PM to _None_.
     ```
   - Spawn Architect Refinement (`subagent_type: architect`):
     ```
     Project root: $PROJECT_ROOT
     PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
     Updated functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md
     Current architecture doc: $PROJECT_ROOT/agent-docs/architect-{slug}.md

     Your questions to the Business Analyst have been answered. Read the updated functional spec
     and revise the architecture sections that need updating. Clear ## Questions for System Business Analyst
     to _None_ if all questions are resolved, or write new ones if needed.
     ```
   - Increment `architect_iter`. Re-read architect doc. Check `## Questions for System Business Analyst` again.
4. If `architect_iter == 3` and Questions ≠ `_None_`: proceed. Tell user: "Maximum Architect Q&A iterations reached — proceeding with remaining questions noted in the architecture doc."

---

## Stage 4 — Developer Agent

Set `dev_iter = 0`.

### Spawn Developer Agent

Use the Agent tool with `subagent_type: developer`. Prompt:

```
Project root: $PROJECT_ROOT
PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
Functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md
Architecture doc: $PROJECT_ROOT/agent-docs/architect-{slug}.md

Implement the feature. Study existing code before writing. Follow Implementation Order
from the architecture doc exactly. Write tests per Test Strategy.
Save dev report to: $PROJECT_ROOT/agent-docs/dev-report-{slug}.md
```

### Developer Q&A Loop (max 3 iterations)

After agent finishes:
1. Read `$PROJECT_ROOT/agent-docs/dev-report-{slug}.md`.
2. Strip whitespace from `## Questions for Architect` body. If exactly `_None_` → advance to Final Report.
3. While `dev_iter < 3` and Questions for Architect ≠ `_None_`:
   - Spawn Architect Refinement (`subagent_type: architect`):
     ```
     Project root: $PROJECT_ROOT
     Architecture doc: $PROJECT_ROOT/agent-docs/architect-{slug}.md

     The developer has questions about your architecture doc:
     {paste "Questions for Architect" section body verbatim}

     Answer them by updating the relevant sections (File-Level Plan, Implementation Order,
     APIs/Interfaces, Data Model, Tech Decisions). Set ## Questions for System Business Analyst to _None_.
     ```
   - Spawn Developer Continuation (`subagent_type: developer`):
     ```
     Project root: $PROJECT_ROOT
     PM doc: $PROJECT_ROOT/agent-docs/project-{slug}.md
     Functional spec: $PROJECT_ROOT/agent-docs/analyst-{slug}.md
     Updated architecture doc: $PROJECT_ROOT/agent-docs/architect-{slug}.md
     Dev report (continue from here): $PROJECT_ROOT/agent-docs/dev-report-{slug}.md

     Your questions to the Architect have been answered. Continue implementation based on
     the updated architecture doc. Update the dev report and clear ## Questions for Architect
     to _None_ if all questions are resolved.
     ```
   - Increment `dev_iter`. Re-read dev report. Check `## Questions for Architect` again.
4. If `dev_iter == 3` and Questions ≠ `_None_`: proceed. Tell user: "Maximum Developer Q&A iterations reached — completed what was possible, remaining blockers in dev report."

---

## Final Report

Report to the user:

- **PM doc:** `$PROJECT_ROOT/agent-docs/project-{slug}.md`
- **Functional spec:** `$PROJECT_ROOT/agent-docs/analyst-{slug}.md`
- **Architecture doc:** `$PROJECT_ROOT/agent-docs/architect-{slug}.md`
- **Dev report:** `$PROJECT_ROOT/agent-docs/dev-report-{slug}.md`
- **Implemented:** (from `## Implemented` in dev report)
- **Follow-ups:** (from `## Follow-ups` in dev report — things user must do manually)
- Any Q&A iterations that hit the cap (if applicable)

---

## Edge Cases

- **`$ARGUMENTS` empty**: ask user for feature description before spawning anything.
- **Slug collision**: ask — overwrite / new name / resume.
- **No README/CLAUDE.md**: PM proceeds with feature request alone.
- **Developer hits showstopper**: noted in `## Follow-ups`, never silently skipped.
- **Monorepo**: Architect reads root build file, locates relevant subproject from source tree.
- **`src` directory absent**: Architect adapts `find` command based on detected project structure.
