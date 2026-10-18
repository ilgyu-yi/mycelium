# Principles

- These principles serve MISSION and the requirements (R).
- DESIGN builds its mechanisms from the primitives below.
- CONTRACTS tags each rule with the invariants that it serves (I1–I10).

## How to read

- **Primitives** are the few kinds of thing that the system has. DESIGN expresses every mechanism with them.
- **Invariants** are rules that hold for every primitive.
  - A mechanism that needs an exception that the invariant does not name is a design error.
  - Named exceptions start with "Exception:".
- A new need gets a new value of an existing primitive (a new link type, a new status cause) before it gets a new mechanism.
- A new primitive needs a decision.
- **Operational rules** (capacity, budgets, telemetry, input conventions) serve a requirement directly and need no primitive.
  - They never change knowledge, standing, visibility, or what recall offers.

## Primitives

| # | Primitive | What it is | Serves |
|---|---|---|---|
| 1 | **Event** | One thing that happened, appended to the log: a capture, an edit, a use, an outcome, a correction, a removal. It is the only source of truth. | Durability, R15 |
| 2 | **Record** | A unit of knowledge built from events. There are four kinds: **source** (a document, kept unchanged), **case** (a problem, the attempts on it, their outcomes), **heuristic** (a candidate approach for a kind of situation), **fact** (a claim about a subject at a time). Pages, the board, and summaries are views, not records. | Findability, Reuse, R6, R7 |
| 3 | **Subject** | What records are about: a system, an account, a document, a pseudonymous person, a place. Subjects can contain other subjects. | Currency, Continuity |
| 4 | **Situation** | A kind of problem, found by grouping cases. Interest areas are sets of situations. | Reuse, Improves with use, R8 |
| 5 | **Link** | A typed relation from a record or a subject to a record or a subject. The types are a small fixed set (below). | Grounding, Currency and correction |
| 6 | **Scope** | Who can read something. Every event, record, subject, session, and derived item has exactly one. | Safety, R16 |
| 7 | **Moment** | A point in the work where knowledge is offered to an agent or a person. | Findability, Advisory, Low friction |
| 8 | **Question** | The only thing that the system asks of people. It is short, optional, and safe to ignore. | Low friction, R11 |

**Link types.** Each type has one meaning and one effect.

| Type | Meaning | Effect |
|---|---|---|
| `about` | The record concerns this subject, situation, or case (a concern, a contingency, an option set aside). | Recall by subject and situation. Shown with the case. |
| `basis` | The record rests on that one (cites, uses, was drafted from, copies, adapts). It may end at a record, a subject (a record that depends on the subject's state), or an event. | Status flows along it (Invariant 4). Independence is judged along it (Invariant 3). |
| `same` | Both describe one thing (repeat, duplicate, equivalent document, a subject's alias). | Evidence pools. One is shown. |
| `counter` | Degree `outcome`: same situation and approach, different result. Degree `claim`: two claims that cannot both hold. | Outcome counters keep both outcomes as evidence. Claim counters show both sides as differing. |
| `replaces` | The newer record or subject takes the place of the older one. | The older one leaves automatic recall for work about a time after the replacement (Settled readings). It stays findable. |

## Invariants

1. **The log is the truth.** (R15, Durability, Agent-agnosticism)
   - Everything except events is derived and can be rebuilt.
   - System models (judge, writer, builder, embedding, reranker) produce only derived data. The data is stored with the version of the model that produced it, so the models can be replaced.
   - What an agent or reviewer writes for a member is that member's input. It is marked as drafted, with its tool and model version.
   - A draft that a person accepts is that person's input.
   - Exception: a write that a safety check holds before the log (a quarantine) is held durably, in backups, as that member's pending input.
     - It is logged when the check passes, or returned to its author, who can resubmit it.
     - It is dropped only by its author, or by the system so that it can be fetched again; never silently.
   - Exception: when a qualified model translated or masked a member's input before storage, the result is stored as that member's input, with the model version.
     - The reason: the original is not kept (R14, R16).
2. **Every record shows where it came from.** (Grounding)
   - Each record has `basis` links down to events or sources, and the author and time.
   - A record with no basis is marked as such.
   - Times carry their source. The source is one of these:
     - Mycelium's own time;
     - the entry time of an outside system that a forester declared a time source;
     - a date that the record's source states, shown as such.
   - Only Mycelium's time and declared time sources can show that a reason came before an outcome.
3. **Support is evidence, not headcount.** (Grounding, Improves with use, Many people)
   - Support counts only evidence that is independent along `basis` links.
   - Authorship, repetition by the same source, and confirmations without evidence add nothing.
   - The kind of evidence sets a ceiling on certainty:
     - Observation gives at most low, but a large effect that is consistent across units and strata may reach moderate.
     - Only randomized comparison rises above moderate.
     - A computational comparison counts as randomized when the experimenter sets every input, and variants and replicates are declared before outcomes.
   - Until support is sufficient, the record says so and shows its basis, not figures.
   - Its warning count and critical-failure bound are part of that basis and are shown.
4. **Validity is derived, never maintained by hand.** (Currency and correction, R9, R12)
   - A record's status is computed from its own events and from the status of what it rests on.
   - When something changes, the change flows along `basis` links to every dependent record, as one marker with a cause.
   - The owners of dependent records get one notice.
   - There is one marker family and one propagation rule.
   - Status flows along `basis` links that carry dependence.
   - The links by which a heuristic rests on its cases carry evidence (founding cases and uses, generalized heuristics, supporting records). This evidence reaches the heuristic through its support (Invariant 3), not as status.
5. **Visibility is the narrowest of the inputs.** (Safety, R16)
   - Anything built from or about other things gets the narrowest scope among its inputs. This applies to a record, a subject, a session, an event, a derived view, and a delivery.
   - The inputs count along `basis` links as well as by text.
   - A session that reads a restricted record narrows to that scope for the rest of the session.
   - A member who wants to write more widely starts a new session.
   - Text is masked for each scope that stores or reads it, and before it crosses to any service that the team does not control.
   - A scope's readers may hold quasi-identifiers and pseudonymized persons exactly. Masking applies wherever text leaves the scope.
   - Special-category data stays masked.
   - A widening is an access decision, not a change of knowledge.
     - The author makes it for their own record. The scope's foresters make it for a scope.
     - It takes effect only after a safety rescan.
   - Removal is complete: it reaches every copy and every derived record.
   - A forester's release is not an exception. It is a new record that a person writes.
   - Exception: a content-free count (no text, no ids) computed over restricted records may be shown outside them when it is above a minimum. Thus serious harm is not hidden (Invariant 6), and support can pool.
   - Exception: a hide withholds a record's text, not its content-free effects. Readers of its former scope keep its warnings' kind and severity in their floor, and its items in counts and track records.
   - Exception: a person scope may hold special-category data exact for its readers when its foresters turn on the field extension for it. Masking still applies wherever text leaves the scope.
   - Exception: inputs count as they were visible at write time. When an input is later hidden, restricted, or moved:
     - its dependents keep their scope and show `basis not visible`;
     - a safety rescan narrows only a dependent whose own text holds the narrowed content.
6. **Recall offers; it never directs.** (Advisory, Low friction, R10)
   - Knowledge arrives only at moments. It is ranked by relevance, then applicability to the current conditions, then support.
   - One budget limits interruptions for each person.
   - The budget and interest areas never hold back anything that could cause serious harm and applies to the current conditions.
   - A warning has a **standing** and a **display**.
   - Its standing is whether it holds for the team. Only evidence about its cause lowers it.
   - Repeated use, repeated exposure, successful outcomes, and time never lower its standing.
   - Its display is per person:
     - Warnings that this person has not seen come first, each named up to a display bound.
     - Past the bound, serious ones fold by cause group, naming the newest and the most severe.
     - Critical ones and those whose conditions match the session stay named.
     - A warning that this person has acknowledged returns only on another instance, another kind of action, changed conditions, or a change of the warning itself (its severity, cause, or evidence).
   - A severity that a model drafted is marked as drafted until a member's first-hand assessment replaces it.
   - Warnings that apply to the current instance are shown regardless of the interest areas (Invariant 6 comes before Invariant 10).
   - Knowledge that is not a warning may fold when it repeats.
   - When a warning in a restricted scope applies to a session outside it, the system shows nothing outside except a content-free count above the minimum.
     - The system tells the scope's readers. They decide whether to release the warning.
   - Exception: before a conclusion, the check may take one extra agent turn, at most once per conclusion. That turn carries only the check and asks for no action.
7. **Use teaches.** (Improves with use, R13)
   - Every delivery and its outcome are events.
   - Outcomes update support (Invariant 3).
   - Some recall slots go to less-shown candidates, so that better approaches can appear.
   - Where the work allows randomized comparison, a champion is tested against challengers, unless a scope's foresters turn the comparison off for that scope.
8. **Nobody has to curate.** (Low friction, R11)
   - Knowledge enters as a side effect of work.
   - No entry needs approval. A gate holds an item back from display, never from entry.
   - Only a safety check (Invariant 5) may hold a write before the log, in the quarantine (Invariant 1).
   - The only requests to people are Questions.
9. **No one person changes the team's view alone.** (Many people, R1, D4)
   - An author's change that makes their own record, which no one else has used or rests on, less shown takes effect at once and can be reversed (withdrawing, narrowing). So does a revert of a damaged import.
   - A warning's standing is not lowered this way. It follows Invariant 6.
   - A change that makes knowledge more shown, changes its content, or makes less shown a record that bullet 1 does not cover, except toward safety, follows its evidence. Examples: a correction or retirement of another's record, a release of a held record.
     - With new independent evidence (an observation, a re-check), it takes effect at once, whatever command records it.
     - Without evidence, it shows beside the original until independent support arrives, or until an undisputed period passes.
     - Silence never lets it override a record that has evidence.
   - Only a dispute with evidence makes both sides differing. A dispute without evidence lapses after the undisputed period.
   - Two values written without either writer seeing the other are concurrent. They are differing until one writer withdraws or evidence resolves them.
   - State that changes over time (a value, a phase) is a fact at a time. A newer observation replaces an older one and does not contradict it.
   - Disagreements are shown. Neither the system nor a forester resolves them:
     - both sides show as differing;
     - neither side is treated as valid;
     - new independent evidence resolves them.
   - Exception: accepting a generation is tending conditions.
     - A forester may accept, with a reason, a generation that failed only the stability check.
     - The reason this is allowed: its changes are derived from the log and roll back in one step.
     - Champion changes still wait for the monthly check.
10. **Focus by interest, not by field rules.** (R8, D32)
    - The core treats every field the same way.
    - A team's interest areas weight recall.
    - Needs that only one field has are extensions. They are turned on per interest area, or per scope for safety and visibility extensions.
    - At automatic moments, knowledge crosses fields only as an abstract restatement of a method, marked with where it came from.
    - Explicit search returns every visible record.

## Precedence

When two invariants pull in different directions, the earlier one in this list wins:

1. **Invariant 5** (visibility and safety)
2. **Invariant 6** (serious harm is shown)
3. **Invariant 9** (no one person changes the team's view alone)
4. The other invariants, which do not conflict when applied as written

One rule follows from this order and applies everywhere:

- A single person may change at once only these:
  - what is **displayed to that person** (collapse, acknowledge, own filters);
  - any **reversible action toward safety** (showing a warning, pausing a holdout, hiding suspected leaked text);
  - any **reversible change toward less shown** that Invariant 9 allows at once (an author withdrawing their own record that no one else used or rests on, a revert of a damaged import).
- These change only by Invariant 9, with evidence or independent support:
  - what the team is shown;
  - a record's **standing** (validity for the team, its track record, a champion's role, another member's record).
- When a setting change (a tunable, a key declaration) alters how items are derived, it applies only to items observed after it takes effect, except a change toward safety (C6).
- Existing warnings, trends, standing, and current values keep theirs until evidence changes them.
- Releasing a held record to the team follows Invariant 9.

Access roles follow the same rule:

- Someone who already holds that access names a forester, or a reader of a restricted scope.
- When no holder answers (no active forester, or a scope whose readers have all left), two independent active members may name one (one of them a team-level forester when one answers); with one active member, that member names (a team of one is a team, MISSION).
- All active members get one notice.
- The naming takes effect after the undisputed period. It lapses on a dispute that a second independent active member supports.
- Until the naming takes effect, the named person may take only reversible actions toward safety.
- A scope's foresters may name a custodian in advance. The custodian becomes a reader when the scope has none.
  - That naming is an access decision under Invariant 5.

## Settled readings

These readings of the invariants are decided. A change to one needs the maintainer's decision.

- A team decision is a decision about access or settings, for which no evidence is possible. It lapses only on a dispute that a second independent affected member supports.
- Harm that many uses cause together is a shared outcome among those uses.
- Work can state the time that it is about (`as-of`). Recall reads what was valid then.
- Cross-field transfer: abstract method restatements only, with their origin (Invariant 10).
- Four record kinds:
  - Notes are facts with a speaker.
  - Adopted decisions are cases with a status.
  - Work items (project state, next steps, hand-offs) are open cases with an owner or a taker. Finishing one records its outcome.
- Work-item coordination (who took an item; the urgency order of a member's own items) is state outside the knowledge invariants.
  - The first take wins.
  - Items are listed by urgency, not ranked as recall.
- Invariant 5 has four named exceptions:
  - content-free counts above a minimum;
  - a hide's content-free effects;
  - special-category data kept exact in a person scope that turns on the field extension;
  - dependents of an input narrowed after they were written.
- An outside system's entry times count when a forester declares it a time source. Risk-of-bias discounts still apply.
- Links may start and end at subjects (aliases, successors).
- No forester settle and no time-based "latest value wins". Differing values stay differing until evidence resolves them.
- An author's change toward less shown of their own record that no one else used or rests on, and a revert of a damaged import, are immediate and reversible.
- A change toward more shown, or of content, needs independent support or an undisputed period with no opposing evidence.
- Once someone takes the other side with evidence, the two are differing. An objection without evidence lapses after the undisputed period (D4).
- Restricted scopes can match a subject across two scopes only when the foresters of both scopes turn it on.
  - The match tells only the readers of the warning's scope that work on the same subject exists elsewhere.
  - No content crosses.
  - A release may go to one named scope.
- A scope can be the intersection of other scopes. Its readers are the members who can read all of them.
- A session that reads two or more restricted scopes narrows to their intersection, not to the member's private scope.
- An access role with no holder to grant it is named by two independent active members, after one notice to all and the undisputed period. With one active member, that member names, after the same notice and period.
  - A dispute supported by a second independent member lapses the naming.
  - Until then, only reversible safety actions are allowed.
  - Custodians named in advance avoid this path.
- The role with extra permissions is called **forester**. It tends conditions and never judges knowledge.
