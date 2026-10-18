# Later

- Features deferred from the first version by the maintainer's decisions of 2026-10-18.
- Each section keeps the text as it stood in CONTRACTS before the move, unchanged, with the section it came from. Partial sentences are the cut spans; `[…]` marks words that stayed in the core.
- Nothing here is normative. A feature returns when its text moves back to CONTRACTS and the mentions it needs are restored there.

## F04. Admin extras: import principal, membership source, follow, availability target and standby, several deployments

- From: C1, C6, C10, C15, A1.
- One deployment per session returned to C15 Session environment, as a session rule (G1 fix G25); the rest of Several deployments stays here.

From C1 Deployment:

> - **Availability** (operational). Shown against the *availability target*. A standby server can take over the log; the restore test covers failover.

From C1 Principals and roles (table row):

| Principal or role | Contract |
|---|---|
| Import principal | Adding a deployment export writes as one: original authors stay as `on_behalf_of` pseudonyms; the source deployment is a declared time source (I2); batch reverts apply (C6). A pseudonym maps to a member as `same` (`alias`) on the principal when the identity provider matches or the member claims it first-hand; authorship, routing, and counting then follow the alias |

From C1 Principals and roles:

> - **Membership source.** A team-level forester may declare identity-provider groups as the membership source (an access decision, I5). For a restricted scope, only its foresters who read it declare one, and a member the source adds takes effect only after the scope's foresters confirm.

From C1 Principals and roles:

> - **Follow.** A member can follow a scope, record, subject, or query; new matches show under "Since I last looked".

From C6 Classes (Access decision row):

> membership change or membership source […] after its confirmation, […] a membership-source member of a restricted scope

From C6 Batch reverts:

> or import principal

From C10 Removal (confirmations; the sentence as it stood, whose retention part is in X10):

> Each confirmation an operation waits on (a move, a content removal, a scope removal, a retention batch, a retention rule on knowledge records, a member added by a membership source to a restricted scope) is a confirmation question (C13). A retention rule on knowledge records, or a member added by a membership source to a restricted scope, that nobody confirms within the *removal confirmation timeout* is asked again of the scope's foresters who read it, each *removal confirmation timeout* (for a retention rule, of the child scopes' other readers); it never takes effect unconfirmed.

From C15 Session environment and installation:

> - **Several deployments.** A client holds one sign-in per deployment; a linked location resolves to at most one deployment; an unresolved directory gets one deployment proposal, as D-C10.1 gives one for a scope.

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Availability target | 99.5% a month | Hook health |

## F12. Version composition and other C3 details: concurrent Mycelium-written versions, `open-edit`, `induced`, assessors declared early

- From: C3, C5, C11, C13, A1, A2.
- Differing held dates returned to C3 Records (scope fix SC-08).
- The `induced` row and "Not induced" in the Recurrence class returned to C3 Cases and C11 Compare (G1 fix G30); the `supports` and `recurrence` degree additions and the A2 clause stay here.

From C3 Sources (Version order):

> Versions written through Mycelium that name one `base_version` compose by passage into one derived merged rendering: a passage changed by one takes that change; changed by both, an `editorial` change takes the later `seq`, and a `material` one makes a `counter` (`claim`) pair, differing.

From C3 Sources (Taking effect):

> At a location a forester sets `open-edit`, a changed passage sets `changed` on its drafted facts and replaces them only when it survives the next refetch (for a watched path, the next scan after the *watched-file quiet period*, a tunable of DEFAULTS).

From C3 Cases (table row):

| Attribute | Meaning (I2, I3) |
|---|---|
| `induced` | Optional: caused on purpose (drill, test); the injected fault is never an attempt |

From C3 Cases:

> - **Assessors declared early** (advisory). When over the *multi-assessor share* of a situation's closed cases have more than one assessor, `done_when` is stated when the case opens; one Reason question asks for it at the first attempt in such situations.

From C5 Degrees (`supports`):

> ; an induced test that held

From C5 Degrees (`recurrence`):

> , or an induced test that reproduced a failure

From C11 Compare (Recurrence row):

> Not induced; t

From C13 Reason row (trigger):

> ; the first attempt of a case with no `done_when` in a multi-assessor situation (C3)

From C13 Reason row (answers):

> ; for a multi-assessor situation, `done_when`

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Multi-assessor share | 50% of closed cases | Capture quality |

From A2 Acceptance properties:

> ; changes to different parts of one source are all current

From A2 Acceptance properties:

> ; an induced drill is not

## F21. Concern pooling and departed-speaker counting

- From: C3, X4.

From C3 Concerns:

> Concerns about one decision that intake compares as `same` pool into one concern with its speaker count and first observed time; the departure rule raises one question per pooled concern for each departure (departures at one time share one question). A speaker's mark removes that speaker from the count. The pooled concern closes when a mark brings the count to zero. A speaker who is not an active member stays in the count; when a departure question for the pooled concern expires unanswered with no evidence, the speakers who are then not active members leave the count, and if no member speaker remains, the pooled concern lapses (`lapsed-concerns`).

From X4 Documents as capture:

> , and pools as C3 Concerns says

## F23. C4 details: `physical` successors, split-part assignment, stated applicability

- From: C4, A2.

From C4 Aliases and successors:

> A `replaces` between subjects has a degree: `rename` (the same unit under a new reference; its warnings carry to the successor) or `physical` (a new unit in the old one's place; only warnings whose cause names the design, the kind, or a shared `uses` subject carry, and the rest stay on the ended subject). Drafting from "replaced" or "swapped" wording proposes `physical`. Without a stated degree, `replaces` is `rename`; a split part takes `rename`. A `physical` degree that no member states is [team change]; until it takes effect, warnings carry.

From C4 Aliases and successors:

> Records `about` a split subject, or with it as a condition value, match each part with `part-not-known` (the old subject and the split time) until the judge assigns parts (derived, at once); unsure parts go to its owners in one batched question.

From C4 Conditions and fields:

> - **Stated applicability.** A forester can set `applicability: stated` on a subject kind [team decision]; values of that kind that the client adds are stored as stated conditions, marked `auto-stated`, and take part in applicability.

From A2 Acceptance properties:

> ; an `auto-stated` one does

## F42. Situation and passage trends; `use-fell`

- From: C8, C9, C12, A2.
- Group-instance and sibling trends returned to C8 Trends, C9 Condition test, C12 Floor, A1, and DESIGN M6 (maintainer's decision, G4).

From C8 Trends (table row):

| Target | Raises a trend when |
|---|---|
| A situation | Its recent warned share against its earlier strata or its partition's other situations exceeds the ratio |

From C8 Trends (instance row, as it stood):

> a passage, […] `caused-by` one passage or

From C8 Own rate:

> for a passage or

From C8 Trends:

> - A situation's trend goes onto the heuristics whose `member` links reach its warned items. It shows in the floor at the situation moment, naming by id only each of the situation's +1 heuristics (C9) whose use share fell, with `use-fell` (its earlier and later share of cases).

From C9 Condition test:

> a situation or

From C12 Floor:

> ; at the situation moment, also the situation's trends, naming heuristics by id only

From A2 Acceptance properties:

> Incidents across copies caused by one passage raise one trend on that passage, with one notice to its owner; a

## F46. Procedure practice and step comparison

- From: C3, C6, C9, C12, A2.
- C9 Uses now excludes an attempt that departs from a procedure at a step (G1 fix G8); step practice stays here.

From C3 Cases (`attempts`):

> ; `deviations`, drafted whenever it cites a procedure (each: the step passage of the cited version, what was done instead, `why`)

From C6 Differing (Resolution):

> A procedure step's record is its step comparison (C9); without one, resolution waits for evidence on the step.

From C9 Uses:

> , whose `deviations` form its **practice** by step

From C9 Restarts:

> , carrying unchanged steps' practice

From C9 Restarts:

> A copy's local difference, or a practice deviation at a step, forms a **step comparison** with the step's other items, stratified as in the observational record; the builder links the deviating attempt group's learned heuristic `about` the step.

From C12 situation moment (validity line):

> per step with a practice comparison: when sufficient, the deviation, how its failure rate differs and in which direction, the certainty, the observational mark, and the heuristic id, else the deviation with its uses, members, and `why`;

From A2 Acceptance properties:

> , and practice deviations,

## F50. C10 admin: scheduled widening, after-widening steps, membership review, evidence from outside, release offers, restricted read logs

- From: C3, C6, C10, C13, X3, A1, A2.
- The work-end offers returned to C10 Release offers at work end and the C13 Release row, and X10 retention batches became a C13 Release trigger (G1 fix G24, maintainer's decision). The periodic offers, the *release offer interval*, and the X3 departure offers stay here.

From C3 Held dates (list item):

> , scheduled widenings

From C6 Classes (Access decision row):

> a scheduled widening at its date

From C10 Narrowest input:

> , as for Evidence from outside

From C10 Scopes:

> - **Scheduled widening.** A dated access decision by the scope's foresters; a date change is the same decision; firing early is an ordinary widening. It covers only records whose `seq` precedes the schedule event; at firing, later records are listed in one confirmation question to the foresters and widen only on their answer. A rescan match at firing holds only the matching records, listed to the foresters.

From C10 Scopes:

> - **After a widening**, […]; a release and its original get `same` (`equivalent`) once both are visible; duplicate subjects get one alias question. […]

- The child-scope, intersection-reader, and quasi-identifier steps returned to C10 Restrict, move, widen (scope fix SC-02); the re-keying rule returned there, widened to every per-scope keyed hash and token (SC-03).
- The dormant-scope rule returned to C10 Scopes (maintainer's decision, G4). Its words "or membership-review" stay here with Membership review.

From C10 Scopes (Membership review):

> **Membership review.** Each *membership review interval*, a restricted scope's foresters get one question listing its inactive members, never its last active reader.

From C10 Partitions and counts:

> - **Evidence from outside.** When intake finds a likely `caused-by` target in a scope the author cannot read, only that scope's readers are told, content-free outside, as for cross-scope matching; they can write the link.

From C10 Partitions and counts:

> - **Release offers.** Each *release offer interval*, a restricted heuristic or method case with an achieved founding case, or a member's private record, about a situation with stuck calls or missed searches (counted content-free across overlays by abstract situation, at the *aggregate minimum*), gives its readers one release question with a judge-drafted restatement that passed the release check; a forester (for a private record, its member) who accepts writes it as their own release record. When work ends, the same offer goes out for the session's new heuristics and method cases in a restricted scope; the scope's forester accepts it in one action. When work ends in a session that narrowed, its facts and cases about team-visible subjects that pass the release check are offered to their author; accepting is the author's own widening, an access decision after the safety rescan (Restrict, move, widen).

From C10 Access logs and data subjects:

> - Reads of restricted records write `seen` with ids, visible only to the scope's foresters; a scope can extend this to its team records.

From C10 Access logs (departure):

> release offers and the private move question cover them before departure

From C10 Access logs (departure):

> release offers and the private move question cover such records before departure

From C13 Membership and last reader row:

> Membership review;

From C13 Membership and last reader row:

> Once per *membership review interval*; last reader:

From C13 Release row:

> ; the *release offer interval*; work ends with new heuristics or method cases in a restricted scope, or with facts and cases about team-visible subjects written while narrowed (C10)

From C13 Release row:

> ; for a private record, its member; for facts and cases written while narrowed, their author

From X3 Holder sets:

> A departing member's release offers come first, for private records about team situations with stuck calls or missed searches; the member decides.

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Membership review interval (per scope) | 90 days | Routed-load concentration, Holder-set share |

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Release offer interval | monthly | Safety findings, Question age |

From A2 Acceptance properties:

> writes `seen` for its foresters and

## F63. Measured outcome

- From: C11, X4.

From C11 Open cases and outcomes:

> - **Measured outcome** (I3). For an open case with a `measure` on a state key and instance, a state fact or `estimate` on that key and instance, under the condition group that the `measure` names, observed at or after `outcome_due`, with the baseline's instrument (or converted across a `bridged` mark, C7), writes a drafted assessment by the C3 Tests thresholds, `checked` when an instrument wrote the value, resting on that fact; with a `design`, per variant against its controls. The outcome question goes out only when no such fact arrives.

From X4 Record streams:

> ; a case with a `measure` closes by C11 Measured outcome

## F65. Subject brief extras: `prompted-run`, `instances-differ`, `copy-of`, passage production line

- From: C12, C14, A2.

From C12 subject moment:

> For a touched passage: its facts and the attempt that produced it, with its `why`.

From C12 subject moment:

> For a group, or a passage stating its value: how many instances differ, at which key, with which values (`instances-differ`). For a state key whose recent observations repeat an unchanged value and are `prompted` (C5 (b)): `prompted-run`, the run length and its prompted share; standing and support do not change.

From C12 subject moment:

> ; a copy shows its parent, the parent version it copied, the current parent version, and the passages where it differs (`copy-of`)

From C14 Documents and pages:

> A subject page shows `prompted-run` as the subject brief does (C12).

From A2 Acceptance properties:

> , and the passage lists the case and its `why`

## F70. Secondary question kinds: Follow-up, What changed, Held sample

- From: C3, C9, C11, C13, A1, A2, DEFAULTS.

From C3 Heuristics (Untried):

> , and a follow-up question asks whether it was done

From C9 Condition test:

> , and one question that asks what changed then

From C9 Condition test:

> one What changed question and

From C11 Gates:

> - Each *held sample interval*, a scope's members get a sample of its held items (*held sample size*) as one question; area-held items are only counted, and widening an area releases them.

From C13 Kinds (table row):

| Kind | Trigger | Recipient | Answers and the events they write | Limit |
|---|---|---|---|---|
| What changed | A heuristic's failures start from a time; a new builder `not_when` on a `within` group | Routing; for a group, its owners and independent readers | By content | Once per new builder `not_when` on a group |

From C13 Kinds (table row):

| Kind | Trigger | Recipient | Answers and the events they write | Limit |
|---|---|---|---|---|
| Follow-up | An untried heuristic named after a serious failure | Routing | By content | After the *follow-up delay* |

From C13 Kinds (table row):

| Kind | Trigger | Recipient | Answers and the events they write | Limit |
|---|---|---|---|---|
| Held sample | C11 | A scope's members | Revise; re-check; leave held | One sample each *held sample interval* |

From DEFAULTS (D-C3.8, moved here in the G4 fix; it named an owner for C1 Owner, which does not define one for a heuristic):

> After a serious failure, an untried heuristic can name an owner.

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Follow-up delay | 30 days | Question age |

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Held sample size | 10 items | Question age |

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Held sample interval | weekly | Safety findings, Question age |

From A2 Acceptance properties:

> with one question about the change

## F74. Dependencies and stalled views

- From: C14, X3, A1.
- "Stalled share" left A4 Continuity (G1 fix G31).

From C14 Documents and pages:

> - **Dependencies** (a view). The subject page and the show command on a subject list what the subject rests on and what rests on it by `basis` (`uses`): current links by default, grouped by kind, with counts and depth. Each list marks `sole-uses` where the subject rests on exactly one current instance for one declared key. It changes no knowledge.

From C14 situation page:

> A page with open cases whose recent authors fell below the *stalled ratio* of its earlier monthly average is **stalled** and stays on the board.

From X3 Holder sets:

> Subjects marked `sole-uses` (C14 Dependencies) are listed beside the drill candidates.

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Stalled ratio | 0.5 of the earlier monthly average | Stalled share |

## F76. Pack and offline recall

- From: C15, A1, A2, A6.
- C15 Failure now injects `recall-unavailable` and delivers the offline floor at reconnection (G1 fix G26); the pack and offline recall stay here.

From C15 Offline:

> - **Pack** (I6). Each *pack refresh*, within the *pack size*, the client keeps what the moments would draw first for the member's scopes: project state, their open questions (answerable offline), briefs for their subjects, recent achieved cases and dead ends, top candidates of active situations, heuristics delivered within the *pack delivered window*, a summary of each packed situation that offline recall can match a query against, menus drawn ahead; and, at any age, procedures, known-situation cases, and serious or critical warnings on their subjects and situations. Refresh is incremental by `seq`; the client setting `metered` limits it to floor items and project state.

From C15 Offline:

> - **Offline recall**: the pack only; items match by the words of the query and by the nearest packed situation summary. Results show `offline-since`, with the time of the last server contact. On reconnecting, changes to packed records come first.

From C15 Keys:

> Pack, q

From C15 Keys:

> ; the pack's floor warnings follow it

From C15 Keys:

> pack and

From C15 Keys:

> A scope forester can keep a scope out of packs; offline recall then shows `scope-not-offline`.

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Pack size; pack refresh; pack delivered window | 500 MB; hourly; 30 days | Hook health |

From A2 Acceptance properties:

> ; with the server unreachable, work start answers from the pack

From A6 Foresters (scope command settings):

> packs

From X1 Episode:

> , which travel with the case in packs

From X1 Menu:

> , so a menu drawn ahead for a pack matches the server's. Menus drawn ahead are logged on sync; an offline episode with none left counts in no effect.

From X1 acceptance property I7-g:

> ; an offline menu uses the server's probabilities

## F84. Copy batches

- From: X4.

From X4 Document drafting:

> - **Copies** (I3, I4). A source that largely matches another elsewhere gets one copy question per **copy batch** (files added together, from one folder, or named alike but for a date or term), which asks whether the batch is new versions of the same material, local copies of it, copies of a document not held here, new reports that reuse text, or unrelated. Until answered, each rests on its match by `basis` (`possible`). New versions join the version chain; local copies write `basis` (`copy`) to the parent version, differing passages stale until marked local differences; copies of a document not held here create a reference-only parent; for new reports that reuse text, each report stays a separate source, and only its carried-forward passages rest on the earlier report by `basis` (`copy`); unrelated removes the link.

## F89. A4 write-burst review

- From: A4, A1, C13, A2, X5.
- The diagnostic measures table, capture quality (now a core measure), the *capture-quality schedule*, and the *uptake share* returned to A4 and A1 (scope fixes SC-07, SC-11). What stays here is the write-burst review.

From A4 Diagnostic measures (Many people, many years row):

> write bursts (by `made_at`),

> ; a burst above the *write-burst threshold* is a review question to foresters, never a correction

From A1 Tunables (row cells as they stood):

| Tunable | Default | Measured by |
|---|---|---|
| Write-burst threshold | 10× the 90-day norm and 200 writes an hour | Write bursts |

From C13 Questions (Review row, trigger and recipient): "a write burst"; "a write burst to foresters". From A2 R15-c: "and raises no burst flag". From X5 foresters' view: "write bursts".
