# Mission

## What this exists for

- Mycelium is the shared memory of a team that works with AI agents.
- It keeps these in one place:
  - the documents that the team produces and gathers;
  - what the team learns from doing the work;
  - the facts that both rest on.
- Its purpose is that anyone on the team, or any agent that works for them, can do these things:
  - find what is already known;
  - use what others learned in situations that they never saw;
  - know what is still true.
- Nobody has to write or curate a knowledge base by hand for this.

## Success looks like

- **Findability** — a person, directly or through their agent, finds anything the team holds that they may see, however it was worded and whoever wrote it.
- **Reuse** — a person or an agent without the original context does better because of what someone else learned, including in a different domain.
- **Grounding** — every piece of knowledge leads to where it came from and what it depends on, and shows how well it is supported.
- **Improves with use** — what happens when knowledge is used makes it better, and the knowledge never stops the team from finding better approaches.
- **Advisory** — knowledge informs and never directs.
- **Currency and correction** — people can see what is still valid, how the work stands, and what changed. Wrong knowledge gets corrected, and the correction reaches everything that relied on it.
- **Continuity** — a new session, or a person who joins, returns, switches between parallel work, or takes over another's work, starts already knowing where the work stands.
- **Low friction** — knowledge accumulates as a side effect of the work, and reaches people without getting in the way of the work.
- **Many people, many years** — the quality holds when many people write, read, correct, and disagree at once, for years. The upkeep stays small and spread across them.
- **Agent-agnosticism** — nothing depends on a particular agent tool or model.
- **Durability** — nothing the team wrote is lost by accident, knowledge outlives changes of models and tools, and people can read the records without the system.
- **Safety** — credentials and personal data stay out, and anything that gets in can be removed completely. The team controls who sees what and where its data goes.

## Explicitly NOT goals

- Replacing the judgment or the usual checks of an agent or a person.
- Enforcing any rule because the knowledge base contains it.
- Assuming a kind of work, a field, or a tool. A team of one is a team.
- Syncing from outside work tools, or acting as a version-control system.

## How the documents relate

Four documents define the product. Each decision has one home.

- **MISSION** (this document) owns purpose, success, and scope.
- **PRINCIPLES** owns the primitives and the invariants that hold for every one of them, with their precedence and named exceptions.
- **DESIGN** owns the mechanisms: how the primitives work together to meet the success criteria, and why each mechanism has its shape.
- **CONTRACTS** owns observable behavior: the data model, the conditions and effects that an implementation must produce, the tunables, and the acceptance properties.
  - Its core holds what every deployment needs.
  - Its modules hold self-contained parts behind one switch or one view.
  - Its appendices hold tables to look up.

How each document is read:

- MISSION, PRINCIPLES, and DESIGN are for one person to read in full.
- CONTRACTS is for looking up.

How the documents stay consistent:

- A lower document refines a higher one. It never silently amends it.
- A contradiction is a defect in the lower document. The fix goes where the decision lives.
- Checks run both ways:
  - consistency downward: each contract cites the mechanism and the invariants that it serves;
  - sufficiency upward: the mechanisms together must yield every success criterion.
- A new rule comes only when the existing mechanisms leave an important failure open. Before that, the existing mechanism is applied or gets a new value.
- Renderings, wordings, examples, and test fixtures are not normative. They live in the reference.

## Stakeholders

- Maintainer: ilgyu-yi (decision maker).
- Primary users: teams of people who work with AI agents, and the agents that read and write the knowledge base for them.

## Last reviewed: 2026-10-17
