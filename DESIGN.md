# Design

- This document owns the mechanisms of Mycelium: how the primitives of PRINCIPLES work together to meet MISSION, and why each mechanism has its shape.
- CONTRACTS owns the observable conditions of each mechanism.
- Section ids (M0–M15) are stable. CONTRACTS cites them.

## M0. Shape of the system

One team has one deployment:

- a **server** that keeps the team's event log, builds derived data, enforces scopes, and serves search, the tool protocol, and the board;
- a **client** on each member's machine that runs commands, connects agent tools, and keeps an offline queue. The client also reviews sessions and checks text before it leaves the machine;
- **adapters** that connect each agent tool to the client;
- the **board**, a screen for people;
- **outside services** that the team allows by name.

Knowledge moves in one loop:

```
work ─▶ capture ─▶ intake ─▶ log ─▶ records, links ─▶ support, status
  ▲                                                        │
  └──── use, outcome ◀──── delivery at a moment ◀──────────┘
```

- Every arrow is an event or is derived from events.
- A person never has to act for the loop to run (I8).
- A person can act at any point. Each act is an event under the same rules as a capture.

Why: MISSION asks for knowledge that accumulates as a side effect (**Low friction**), improves with use (**Improves with use**), and stays readable without the system (**Durability**). A loop over one append-only log gives all three with one source of truth.

## M1. Log and derivation

- The log is append-only and has one team order.
- Every other thing is a fold of the log or derived data built from it (I1).
- Derived data belongs to a **generation**. The generation records its models and prompts.
  - A new generation becomes current in one step, only after switch checks pass.
  - The previous one stays for rollback.
- Model output is never member input. So a model can be replaced and its output rebuilt (I1).
- A change is a new event that names the state that its writer saw.
- Two changes to one attribute from one base are **concurrent**.
  - The system never picks a winner by arrival order.
  - Both values are differing until evidence or a writer resolves them (I9).
- Removal is the one deletion. The log records that it happened, never what it removed (I1, I5).
- Text that a safety check holds waits in the **quarantine**, outside the log. It is never dropped silently (the I1 exception).

Why: R15 and **Durability** need a source of truth that survives model changes and concurrent writers. Folding by sequence, with explicit concurrency, makes **Many people** safe without locks or a referee.

## M2. Knowledge model

- **Records** have four kinds (PRINCIPLES):
  - A **source** is a document version that never changes. Passages locate spans in it.
  - A **case** is one problem or one decision, with ordered attempts and their outcomes.
  - A **heuristic** is a candidate approach for a kind of situation, stated by a person or learned from cases.
  - A **fact** is a claim about a subject at a time, with a speaker.
- Notes are facts with a speaker.
- Decisions are cases with a decision status.
- Work items are open cases with an owner or a taker. Finishing one records its outcome (D9, D27).
- **Subjects** are what records are about and what can change.
  - Each subject has a registered kind.
  - Subjects contain other subjects. A subject can rest on another subject.
- **Conditions are subjects**: a condition is a key (a subject kind) and a value (a subject).
  - So one mechanism for grouping, aliases, and successors serves subjects, conditions, and fields (I10).
- A **situation** is a group of cases that want the same outcome against the same conflict, where a practitioner takes the same first moves.
  - Situations are derived per generation.
  - Interest areas are sets of situations (D32).
- State that changes over time is a fact at a time. A newer qualified observation replaces an older one. It never contradicts it (I9, D14).

Why: **Reuse** needs a unit that carries across situations (the heuristic, D10) and a unit that carries the evidence (the case, D9). Conditions and fields are subjects, so one mechanism handles renames, splits, and grouping everywhere. There is no separate mechanism for each attribute.

## M3. Evidence

This mechanism decides what counts, and how much.

- **Links** have five types (PRINCIPLES).
  - Degrees refine a type without new effects.
  - Only `basis` carries status.
  - Its evidence degrees (`member`, `generalizes`, `supports`) carry evidence into support, never status (I4).
- **Evidence classes**, in order of weight:
  1. **direct**: an observation of the subject itself (its file at a version, its log, a measurement with its instrument) that a citation check supports, or a re-check against a re-checkable citation;
  2. **first-hand**: a speaker's own observation of the instance;
  3. **cited**: a statement whose citation supports it;
  4. **self-reported** and **relayed**: statements with no check;
  5. **claim** and **interpretive**: beliefs, and readings beyond the words of a source.

  Classes 1–3 are **evidence for full effect** (M4). Class 5 is evidence only for a dispute. A higher class decides against any number of items of a lower class (I3).
- **Independence** is one rule. An item supports a record only when all of these are true:
  - their `basis` paths do not meet;
  - the item's author was not given the record (unless the item is a separate occurrence);
  - they share no original speaker.
- A failed test makes an **echo**. An echo counts as use, never as support (I3).
- **Counting** counts distinct original speakers, never recorders, adders, or repeats.
  - Marks (`same source`, `separately sourced`, `coordinated`) correct the count.
  - Marks are changes under M4.

Why: **Grounding** and **Many people** fail when support is headcount. One evidence order and one independence rule replace the separate rules for echoes, copies, lineage, citation checks, and confirmations.

## M4. Change protocol

- Every change to what others see goes through one protocol.
- This protocol is the general form of I9 and of the display and standing rule (PRINCIPLES Precedence).
- Each operation in CONTRACTS names its class. It states no timing rule of its own.

| Class | What it covers | Takes effect |
|---|---|---|
| **Own view** | What only the actor sees: collapse, acknowledge, own filters, own areas | At once |
| **Toward safety** | A reversible action that shows more risk or hides suspected leaked text: show a warning, pause a holdout, hide, narrow, quarantine | At once, reversible |
| **Own withdrawal** | An author makes their own record less shown, when nobody else used it or rests on it | At once, reversible |
| **State over time** | A newer qualified observation of a changing state (M2) | At once. The older one is kept as `replaced` |
| **Team change** | Knowledge others see: more shown, content, standing, another's record, a mark or link on another's record, a release | By evidence (below) |
| **Access decision** | A widening, membership, or sensitive-class decision by those who hold the access (I5) | At once, after a safety rescan. Some wait for a confirmation (a move; an extension may add a retention rule). |
| **Team decision** | A decision that no evidence can support: an access role, a child scope, a setting that changes what others are shown or asked, or how standing is derived, except a change toward safety | By notice (below) |

**Team change by evidence.**

1. With evidence for full effect (M3 classes 1–3), of at least the highest class in the target's basis: at once, reversibly. Evidence of a lower class makes both sides differing.
2. Without it: **pending**.
   - The change is shown beside the original, which keeps its rank.
   - One question asks one routed member.
   - The change takes full effect after the **undisputed period** with no dispute that has evidence.
   - The period counts only days on which the people who would answer are available. It has a cap.
   - It never overrides a target whose basis holds direct or first-hand evidence.
3. A dispute with evidence makes both sides **differing**. A dispute without evidence lapses after the undisputed period.
4. Differing sides stay differing until new independent evidence resolves them, or conditions show that both hold. No person, role, count, or time resolves them (I3, I9).

**Team decision by notice.**

- One notice goes to the people affected.
- The decision takes effect after the undisputed period. It lapses on a dispute that a second independent affected member supports.
- Evidence does not shorten the period, because the decision is not a claim about the world.
- A setting that changes how items are derived applies only to items observed after it takes effect.
- Existing warnings, trends, standing, and current values keep theirs until evidence changes them (PRINCIPLES Precedence; C6 Settings apply forward).
- **Access roles** are the main case.
  - When no holder of a role exists to grant it, two independent active members name one.
  - Until the naming takes effect, the named person takes only actions toward safety (PRINCIPLES Precedence).

Why: one timing rule for every change keeps behavior predictable for **Many people**. It also keeps any one person from changing the team's view (I9).

## M5. Status and propagation

- A record's **status** is `current` or a set of **causes**.
  - Each cause has a kind, a reference, a time, and a severity.
  - There is one cause family (I4).
- A cause set on the record is **direct**.
- A cause that reached it along `basis` is **inherited** and shows its path.
- Only direct causes take a record out of automatic recall.
- Inherited causes rank lower, except that inherited serious warnings stay in the floor (I6).
- **Propagation** is one rule.
  - A change of direct causes, a `replaces` link, a removal, or a qualifying state change is a **trigger**.
  - One source version or one write batch is one trigger.
  - The trigger reaches every record that rests on the changed one, at any depth.
- **One notice per person per trigger**, never one per dependent.
- Closed work that rests directly on a seriously changed record gets one question per owner per trigger.
- **Derived first**: views and learned records rebuild before any person is asked.
- An inherited cause clears when its source cause clears.
- A warning clears only by its standing rule (M6).

Why: **Currency and correction** needs corrections to reach whatever relied on them. **Low friction** needs that reach to cost each person one line. One cause family and one traversal replace the separate stale, changed, relies-on, recheck, and anomaly mechanisms.

## M6. Warnings

A warning is a cause with a severity. It has a **standing** for the team and a **display** for each person (I6).

- **Standing** falls only on evidence about its cause, or evidence that the event did not occur.
  - Use, exposure, success, acknowledgment, withdrawal by its speaker, and time never lower it.
- **Display** is per reader.
  - Unseen warnings come first, each named up to a display bound.
  - Past the bound, unseen serious warnings fold by cause group.
  - Critical ones and those whose conditions match the session stay named (I6).
- **Trends** are warnings on a target (a heuristic, an instance, a group, a subject). A target gets a trend when its warned items do one of these:
  - exceed a ratio against a base rate (for a heuristic, from distinct independent assessors);
  - recur as separate occurrences (critical warnings, and targets with no history, use the absolute count; other targets whose earlier occurrences already pass the recurrence count use their own earlier rate).
- A trend clears only by evidence about its warnings' causes.
- The **floor** is every applicable serious or critical warning that bears on what a moment concerns.
  - No budget, cap, area, repetition, hide, or revert holds it back (I6 before I10).
  - A display fold is not a hold-back.
- A restricted warning that applies outside its scope shows outside only as a content-free count.
  - Its readers are asked whether to release a version (I5 before I6).

Why: **Advisory** and **Safety** pull against **Low friction**. Standing and display are separate. Thus the system can stop repeating a warning to one person, and never weaken it for the team. This is the mechanism against normalization of deviance (O1).

## M7. Learning

- **Outcomes.**
  - An attempt's outcome comes from its assessments.
  - Each assessor's latest assessment counts.
  - `checked` assessments decide over others.
  - Disagreeing deciding assessments make the outcome differing.
  - Each outcome maps to a failure value.
- **Learned heuristics.**
  - The builder groups attempts across cases by what was done and why.
  - A group with a success, or with enough failures, becomes a learned heuristic.
  - A group of only failures, for reasons of approach or unknown, becomes a dead-end heuristic.
  - Founding cases are fixed at creation. Later attempts are **uses**.
- **Track records** have three tiers, per situation:
  - **observational**: a heuristic's items against comparable items, within strata. Certainty is at most low, or moderate for a large consistent effect.
  - **controlled**: paired comparisons of variants under the same conditions. Certainty is low; moderate when randomized or for a large consistent effect; high when randomized with checked outcomes. The champion role stays with the effect tier.
  - **effect**: randomized episodes from the champion module. Certainty is up to high.
- **Sufficiency** gates every figure. An insufficient record shows its basis, its warning count, and its critical-failure bound, never a figure (I3).
- **Certainty** starts at the ceiling of its tier and falls by named factors.
- **Exploration** (I7).
  - One recall slot goes to the least-shown qualifying candidate.
  - Monthly, the builder writes rivals for the most-used records.
  - A challenge question goes to a member who neither wrote nor recently applied the record.
  - Retirement needs evidence.
- **Champion module.** A self-contained randomized module, on for decidable situations (D23); a scope's foresters can turn it off.
  - In a decidable situation, it draws a menu per episode, with logged probabilities.
  - It names at most one champion, by its effect.
  - Its only outputs are decidability, champion, effect, and menu.
  - It must satisfy statistical properties, not a named method.
- **Conditions learned.** The builder tests condition groups on failures against successes.
  - A passing test adds a `not_when` entry.
  - It never hides the heuristic.
- **Source reliability** (C9 Reliability).
  - The builder counts, per source and origin, the facts that later proved wrong against those re-checked as holding.
  - A sufficient adverse share gives the source a cause, `source unreliable`, which flows as any cause (I4).
  - Members get no team-visible figure.

Why: **Improves with use** and **Learning stays alive** (R13) need outcomes to change support. They also need protection against early lock-in. Tiers with ceilings keep observational records from looking like proof. The least-shown slot and rivals keep new approaches visible.

## M8. Scopes and safety

- **One scope each** (I5).
  - Every event, record, subject, session, and derived item has exactly one scope.
  - Scopes form a tree. A child is never less restricted than its parent.
  - A scope can be the intersection of two or more scopes.
- **Narrowest input.**
  - A write takes the narrowest scope among its inputs, by links and by matching text.
  - A session that reads a restricted record narrows for the rest of the session.
- **Partitions.**
  - Summable data is kept per scope and summed at read time.
  - Data that cannot be summed (situations, champions, areas) is computed per partition: the team, and an overlay for each restricted scope.
- **Crossing a scope.** Only these cross:
  - the content-free effects that I5 names as exceptions (counts above a minimum, a hide's effects);
  - a content-free match notice to a scope's readers;
  - a **release record**. It is not an exception, but a new record that a person writes.
- **Checks before storage and before leaving.**
  - Text is checked on the client, before it leaves the machine, and again at intake.
  - Credentials and personal data are masked.
  - Special-category data stays masked, except in a person scope with the field extension.
  - Text with no check yet waits in the quarantine (M1).
- **Data boundary.**
  - Only allowed outside services receive text, and only from scopes whose boundaries allow them.
  - Agent-tool providers are outside services.
- **Hide and removal.**
  - Any reader can hide suspected leaked text at once (toward safety, M4).
  - Removal reaches every copy and every derived item.
  - Removal installs a deny rule that only readers' clients can apply.

Why: **Safety** and R16 require that what one session reads cannot leak through what it writes. The narrowest-input rule, along links and text, closes the leak without asking anyone. Partitions let restricted work learn without changing team figures.

## M9. Capture and intake

- **Capture paths.**
  - The agent captures cases and notes as it works.
  - The client reviews each session after it ends and drafts what the agent missed. The transcript never leaves the machine (D7, D26). A tool-protocol session record that the server holds is reviewed on the server; no transcript moves (D26).
  - Documents enter by command or watched paths.
  - Members write through commands or the board.
- **Drafts are member input**, marked drafted with their tool and model version (I1).
- **Intake** runs every write through the same steps: fixed checks, judged checks, split, compare, uses, weigh.
  - Compare classifies each new item against similar items as one event, recurrence, repeat, counter, same heuristic, or new.
  - Compare writes the matching link (D16, D17, D18).
- **Gates** hold an item from display, never from entry (I8, D15).
- **Documents as capture.**
  - The judge reads a document as a recount, a record stream, a decision record, or a statement of rules and facts.
  - The judge drafts records that rest on its passages.
  - Drafted records rebuild when the document changes.
- **Outcomes without asking.** An outcome can be inferred from silence only where the situation's own history shows that silence usually means success.

Why: **Low friction** (R11) rules out approval steps. Session review and document drafting make capture a side effect. One intake pipeline means that a capture, a document, and a command produce the same records.

## M10. Delivery

Every delivery runs one pipeline:

1. A **moment** occurs in the work (start, subject, situation, stuck, check, date, notice, ask). The moment names its candidate sources.
2. **Visibility**: only records of the session's scope or wider. Held and retired records come only by explicit search (I5, I8).
3. **Floor**: applicable serious and critical warnings, outside every limit below (M6).
4. **Rank** within each kind: relevance tier, then applicability to the session's conditions, then support. Never by author (I3).
5. **Fields**: at automatic moments, knowledge from another field arrives only as a `general` heuristic with its origin (I10).
6. **Caps** with a pointer to what was left out.
7. **Repetition** per reader: a record returns only when it changed, a watched condition appeared, or the work concerns another instance.
8. **Budget**: pushes between moments (questions, notices, concerns) share one interruption budget per person; recall at a moment does not count. Busy sessions wait (I6).

Timing and kinds of delivery:

- Recall of non-warning knowledge reads what was valid at the work's reference time: now, or the `as-of` date that a case or work item states.
- The floor and warnings always use the current state.
- Every delivery is an event.
- Heuristics arrive only at decision points (stuck, check, situation). They return only when a condition that they failed under appears.
- Procedures also arrive in orientation, and contingencies in the subject brief of their target (D20).
- Facts and cases arrive early (D21).

Why: **Findability** and **Advisory** need relevant knowledge at the right point. **Low friction** needs it to stay small. One pipeline replaces separate budgets, channels, and caps per moment. The moment table then states only what each moment considers.

## M11. Questions

- A question asks a person for an event that any member could write unasked.
  - The event is one of these: an outcome, a reason, a condition, a link, a re-check, a support or dispute, a confirmation.
- The answer is that event.
- It takes effect by its own rule, mostly the change protocol (M4). Being an answer gives it no extra weight (I8, I9).
- A **kind** names a trigger, a recipient rule, the answer choices, and a limit. It adds no new effect.
- **Routing** is one chain, in this order:
  1. active owners;
  2. the attempt's last adder or the author;
  3. members who applied or rest on the record;
  4. subject owners;
  5. the triage group;
  6. the board.

  Routing never goes to a model or a departed member.
- **Independent questions** go only to readers independent of the record.
- Ignoring a question is always safe.
- Only a safety or access operation may wait on a confirmation.
- Every pushed question counts in the budget (M10).

Why: **Low friction** permits only short, optional questions (D5, D30). If answers had special effects, the person who gets a question would gain power over the team's view. An answer is an ordinary event, so it has no such power, and no kind needs its own rule.

## M12. People, roles, and continuity

- **Principals**:
  - members;
  - services, which act for a principal and never read more than it. A service that serves several people reads only what all of them read;
  - guests (restricted scopes only; an extension, X8).
- **Foresters** tend conditions: scopes, members, settings, models, the data boundary.
  - They never judge knowledge.
  - They answer as members (D33, PRINCIPLES).
- **Owners** of subjects and scopes get notices and questions. They never gate a change (I8, I9).
- **Access roles** with no holder follow M4. A **custodian** named in advance avoids that path.
- **Departure and hand-over.**
  - A leaver's owned items are listed for reassignment.
  - A successor gets orientation and new-owner briefs.
  - Private records never become readable by another person.
- **Work items and project state.**
  - Work items are open cases.
  - Project state is facts about the scope's project subject.
  - Who took an item, and the urgency order of one's own items, are coordination outside the knowledge invariants. The first take wins (PRINCIPLES Settled readings).
- **Views**: pages for subjects and situations, the board, and exports.
  - Every line of a view leads to its records.
  - A view changes no knowledge.

Why: **Continuity** needs state that a new person or session can read at once. **Many people, many years** needs roles that keep working when people leave, without a central curator.

## M13. Integration

- **Adapters** turn each tool's events into one common input, and turn one common output back.
  - Only one mapping table names tool events.
  - A missing tool event loses only the moments that need it (R4).
- **Never blocks.**
  - The integration only adds text. It never denies an action (I6).
  - The one extra check turn before a conclusion is the I6 exception.
- **Tool protocol.** Agent commands are also exposed as protocol tools, with the same checks and scopes, for agents that have no adapter.
- **Offline.**
  - The client queues every write until the server logs it.
- **Commands** are thin: each writes one kind of event, or reads one view.

Why: **Agent-agnosticism** and R4 require that no mechanism depends on one tool. The common input keeps moments working when a tool is missing.

## M14. Operation

- **Measures** tie to the success criteria.
  - Each core measure names its criterion.
  - A tunable names the measure that it moves.
- **Tunables** hold every number.
  - A tunable that carries an invariant has a bound.
  - A change that alters what others are shown or asked, or how standing is derived, is a team decision (M4).
  - A change to how items are derived applies only to items observed after it takes effect (C6 Settings apply forward).
  - A change toward safety applies at once.
- **Models** serve a function only after qualification on a shipped fixture.
  - A labelled canary set watches each judge.
  - When a judge falls below its canary band, its decisions become questions.
- **Budgets** limit model work and order it. Nothing is skipped silently.
- **Reference deployments** fix the scale at which the acceptance properties must hold (R2).

## M15. Extensions

- A need that only one field or a few fields have is an extension.
- A forester turns it on per interest area, or per scope for safety and visibility extensions.
- The forester level that turns an extension on turns it off; safety and visibility extensions stay on once on (CONTRACTS Part II).
- The champion module (X1) is not an extension. It is a core option: it runs in each situation while it is decidable, and a scope's foresters can turn it off for that scope (M7, CONTRACTS Part II).
- An extension uses only the primitives and the named exceptions. It never adds an exception (I10).

## Coverage

Each success criterion, and the mechanisms that carry it.

| Criterion | Mechanisms |
|---|---|
| Findability | M9, M10, M2 (situations), M7 (exploration) |
| Reuse | M2 (heuristics, situations), M7, M10 (fields) |
| Grounding | M3, M5, M7 (sufficiency and certainty), M9 (documents) |
| Improves with use | M7, M10 (use events), M11 (outcome questions) |
| Advisory | M6, M10, M13 (never blocks) |
| Currency and correction | M4, M5, M6 |
| Continuity | M10 (start, subject), M12 |
| Low friction | M9, M10 (budget), M11 |
| Many people, many years | M3, M4, M12, M14 |
| Agent-agnosticism | M13 |
| Durability | M1, M14 |
| Safety | M8, M6 (restricted warnings) |

Each mechanism, and the invariants it applies.

| Mechanism | Invariants |
|---|---|
| M0 Shape of the system | I1, I8 |
| M1 Log and derivation | I1, I2, I9 |
| M2 Knowledge model | I1, I2, I4, I5, I9, I10 |
| M3 Evidence | I2, I3, I4 |
| M4 Change protocol | I9, Precedence |
| M5 Status and propagation | I4, I6 |
| M6 Warnings | I6, I3, I5 |
| M7 Learning | I3, I7 |
| M8 Scopes and safety | I5, I1 |
| M9 Capture and intake | I1, I5, I8 |
| M10 Delivery | I6, I7, I10, I5 |
| M11 Questions | I8, I9, I3, I5 |
| M12 People, roles, and continuity | I5, I8, I9 |
| M13 Integration | I6, I1, I5 |
| M14 Operation | I1, I7, I9 |
| M15 Extensions | I10 |
