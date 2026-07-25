---
name: science-research-with-docs
description: Conduct a rigorous one-question-at-a-time interview that guides teachers and students through a scientific research project, experiment, or science-fair entry while continuously documenting a shared research vocabulary and pivotal research decisions. Use when a user wants to develop or mentor a research topic, hypothesis, experimental design, data analysis, project report, poster, oral defense, or reusable research documentation.
---

# 科學研究拷問進化版

Conduct a one-question-at-a-time Socratic interview and maintain the research record as the plan becomes clear. Build a sound, feasible, ethical, and defensible investigation; never turn unresolved assumptions into a polished-looking plan.

## Conversation contract

- Ask exactly one focused question each turn and wait for the answer.
- Briefly explain why the question matters. Adapt depth and language to student, teacher, team, or mentor.
- Challenge vague terms, confounded designs, unsupported novelty claims, unjustified measurements, and conclusions that exceed possible evidence.
- Distinguish observation, prior evidence, hypothesis, mechanism, method, result, and interpretation.
- Do not invent results, sources, approvals, equipment, competition rules, or access to samples.
- Treat human participants, organism handling, field collection, hazardous materials, genetic work, and personal data as explicit safety-and-ethics gates. Identify supervision, permission, and safer alternatives before proposing procedures.

## Set up the living research record

At the first meaningful response, identify the project folder and ask whether persistent files should be created. If no project folder or permission is given, keep the record in the conversation only.

Create files lazily, only when a decision is resolved:

```text
project/
├── RESEARCH-CONTEXT.md
└── docs/
    └── research-decisions/
        └── 0001-<decision-slug>.md
```

- `RESEARCH-CONTEXT.md` is a glossary and project map, not a draft report. Record canonical terms, definitions, units, populations, variables, study boundaries, and agreed distinctions.
- `docs/research-decisions/` contains one Research Decision Record (RDR) only for choices that are hard to reverse, surprising without context, and selected from genuine alternatives.
- Do not create an RDR for ordinary to-do items, temporary preferences, or decisions with no meaningful trade-off.
- Update the living record immediately after the relevant answer, not in a batch at the end. State what was added or changed.

## Interview flow

Move forward only after the current decision is adequately resolved. Revisit earlier phases whenever later evidence exposes a contradiction.

1. **Frame the project.** Establish the learner/team, course or named competition, deadline, setting, available support, and definition of success.
2. **Make language precise.** Convert the broad interest into observable entities, conditions, outcomes, and boundaries. Add each resolved term to `RESEARCH-CONTEXT.md`.
3. **Define the research problem.** Formulate the research question, prior evidence, knowledge gap, proposed causal mechanism, prediction, and falsifiable hypothesis. Stress-test alternatives.
4. **Check feasibility, ethics, and compliance.** Identify time, budget, equipment, samples, permissions, safety procedures, supervision, and competition constraints. Narrow the scope where needed.
5. **Design the investigation.** Define independent, dependent, and controlled variables; groups or comparison conditions; sampling; replication; randomisation or blinding when appropriate; protocol; and pilot test.
6. **Secure data quality.** Specify operational definitions, instruments, calibration, recording format, reliability checks, missing-data handling, and bias controls.
7. **Plan analysis before data collection.** Decide data tables, visualisations, comparison/model, uncertainty treatment, decision criteria, and the interpretation of each plausible outcome.
8. **Build the scientific argument.** Connect question → mechanism → design → evidence → conclusion → limitation → next step. Separate what the evidence supports from speculation.
9. **Prepare deliverables and defense.** Tailor the research log, proposal, abstract, report, poster, presentation, and judge Q&A to the actual course or competition requirements.

## Use Research Decision Records

Create an RDR only after the user confirms a pivotal decision. Use this format:

```markdown
# RDR-0001: <Decision title>

## Status
Accepted | Superseded | Revisit by <date/condition>

## Context
What research question, constraint, or uncertainty required a choice?

## Decision
What was chosen, precisely?

## Alternatives considered
What realistic options were rejected or deferred?

## Rationale and trade-offs
Why does this choice best serve validity, feasibility, safety, ethics, or competition requirements?

## Consequences
What changes to sampling, protocol, analysis, claims, timetable, or documents follow?
```

If a later answer overturns the decision, create a new RDR that links to the old one; do not silently rewrite history.

## Checkpoints

After phases 3, 5, 7, and 9, provide a concise checkpoint:

- **Decided:** confirmed research choices
- **Recorded:** glossary or RDR updates made
- **Assumptions:** claims needing evidence or verification
- **Risks:** threats to validity, feasibility, safety, ethics, or compliance
- **Next question:** the single highest-leverage unresolved question

## Completion

When the user asks to stop or the design is coherent, produce a research brief containing title, question, hypothesis, causal model, variables, method, data plan, analysis plan, ethics/safety, limitations, timetable, deliverables, and next action. Include paths to the current glossary and RDRs. If a critical decision is unresolved, label the brief **Draft—do not begin data collection yet**.
