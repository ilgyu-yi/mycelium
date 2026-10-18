# Defaults

Recommended default behaviors for an implementation of Mycelium. **These are not normative.** An implementation may choose otherwise, provided that it keeps every invariant (PRINCIPLES), requirement (REQUIREMENTS), and contract (CONTRACTS). Each entry names the CONTRACTS section it came from. A default that turns out to be needed (an alternative breaks an invariant, a requirement, or the consistency of shared state) moves back into CONTRACTS.

## C1. Deployment and principals

- **D-C1.1** (from C1 Components, shipped models): The small safety classifiers need no GPU.
- **D-C1.3** (from C1 Model qualification): A forester can replace any model.
- **D-C1.4** (from C1 Reference deployments): Install verification and the foresters' view at each *monthly check* compare expected model calls with capacity, propose budgets, and report a model change's build cost.
- **D-C1.5** (from C1 Reference deployments): A build reruns only functions whose model changed.
- **D-C1.6** (from C1 Principals and roles, Member): Routing, notices, and hand-offs use the member's settings.

## C2. Log and derivation

- **D-C2.1** (from C2 Envelope, `time`): `time` is ISO 8601 with seconds and offset.
- **D-C2.4** (from C2 Generations): Incremental updates apply between full builds. Full builds also run on request, within the *build budget* and *build deadline*, most-used first (a rebuild of the similarity index after its model is lost included).
- **D-C2.6** (from C2 Switch checks): A build that fails a switch check shows its failed limits.
- **D-C2.7** (from C2 Switch checks): A check at each *fidelity check cadence* rebuilds the *fidelity sample*.
- **D-C2.8** (from C2 Stable ids): Model text versions keep stable ids.
- **D-C2.9** (from C2 Ongoing budget): Model work in the budget runs in order: recall-time calls, intake classification, view rewrites, the rest. Scopes share fairly. Model calls for other drafted records wait behind those for live captures, ordered by `made_at`.
- **D-C2.15** (from C2 Folding and concurrency, Concurrent): Concurrent changes from one member's own sessions show to that member as one `changed-in-your-other-session` line, which one answer resolves.
- **D-C2.10** (from C2 Ongoing budget): Model text is reused until its inputs or model change.
- **D-C2.11** (from C2 Export): An export can be of a selection (records by subject, situation, or kind, with what they rest on, as for a forester's selective widening, C10).
- **D-C2.12** (from C2 Export): The Markdown rendering has one file per record and relative links.
- **D-C2.13** (from C2 Export): Approving a scheduled export destination asks for the declaration that it is readable only by the scope's readers.

## C3. Records

- **D-C3.1** (from C3 Held dates): Views show a held date in the reader's time zone and in its declared zone.
- **D-C3.3** (from C3 Cases, `hypotheses`): A hypothesis part has the id `<case>#h<n>`.
- **D-C3.4** (from C3 Cases, `set aside`): `set-aside-at` names the version set aside and the current version.
- **D-C3.5** (from C3 Drafting, advisory): Draft one case per problem; every attempt, failures included, with reason and sources in order; surprises as anomalies; a harm on the most specific subject named; for a failure, why the action made sense and the system factor, never blame; the cause as well as the fix.
- **D-C3.6** (from C3 Heuristics, Required): A heuristic with no stated condition and no `not_when` shows `unconditioned`.
- **D-C3.9** (from C3 Renderings): A fact drafted from a non-English source shows its passage's original wording.
- **D-C3.11** (from C3 `health`): Older `health` values fold into one line with a count and the latest date.
- **D-C3.12** (from C3 Concerns): Concerns about one decision that intake compares as alike fold into one line at its check moments; one departure raises one question per person asked, covering every concern it affects.

## C4

- **D-C4.1** (from C4 Registry): Shipped kinds also include a software set (code path, service, package), a non-software set (`document`, `site`, `sample`, `matter`), and dataset.
- **D-C4.5** (from C4 Subjects, `uses` between subjects): Intake proposes `basis` (`uses`) links between subjects from watched lockfiles and session conditions; a member confirms each.
- **D-C4.7** (from C4 Aliases and successors): Alias questions are bulk-answerable.
- **D-C4.8** (from C4 Ordered kinds): A kind can declare patterns that read values from tool results, prompts, and pasted text.
- **D-C4.10** (from C4 Interest areas): Each interest area has a one-sentence description (judge-written, member-edited).

## C5

- **D-C5.1** (from C5 Links): A link can carry a one-sentence reason.

## C6

- **D-C6.1** (from C6 Team decision by notice): A pending setting change shows in Settings until it takes effect.

## C7. Status and propagation

- **D-C7.1** (from C7 Severity order): The show command lists every cause with its history.
- **D-C7.5** (from C7 Propagation, Reached): Beyond the *path display depth*, a reached record shows `via-count`: the number of changed records on the path.
- **D-C7.10** (from C7 Clearing): A bulk answer names what separates the records it answers for.
- **D-C7.11** (from C7 One notice per person per trigger): The `change` notice also goes to owners of the reached records' subjects, owners of adopted decisions and procedures that rest on them, and adders of member-writing sources that rest on them.
- **D-C7.12** (from C7 One notice per person per trigger): For a record drafted from a document, the `change` notice also goes to the members in the speaker set of its passages (C3 Writer) who read the record's scope and to the source's subject owners, so that practice that contradicts a drafted procedure reaches the writers of its steps.
- **D-C7.13** (from C7 One notice per person per trigger): When an attempt that cites heuristic H achieves a better result than H's items on a declared measure, H's author and the owners of its founding cases' instances get one `teammate` line, within the budget.

## C8. Warnings

- **D-C8.1** (from C8 Trends, trend marker): The `warning-trend` marker shows the kind, the warned items of comparable items since a time, and the prompted share of the warned items.
- **D-C8.2** (from C8 Display): The floor line shows the warning's recurrence count on distinct instances, its acknowledgment count, and, for an `estimate` or `measure`, the worst value and its trend; exposure counts are in the show command.

## C9. Learning

- **D-C9.2** (from C9 Dead ends): A dead end shows its basis: `dead-end-basis` (failed attempts and their reasons) or `never-tried` (members who set it aside).
- **D-C9.5** (from C9 General heuristics): A general heuristic gives one example per field.
- **D-C9.18** (from C9 General heuristics): A team whose single-field dead ends and warned heuristics should reach other fields can lower the *general-heuristic fields* to 1 (a [team decision], A1). This applies to every heuristic; `general` records still arrive only under "From other fields".
- **D-C9.6** (from C9 Track records): When the record across situations triggers Inconsistency, each sufficient per-situation record shows with its direction.
- **D-C9.7** (from C9 Sensitivity stratum): The sensitivity stratum shows only to test whether a result changes without it.
- **D-C9.8** (from C9 Condition test): A builder `not_when` entry is marked `pre-declared` when its condition key appeared in a `hypotheses` part or `done_when` before the items' outcomes (I2 time order), else `post-hoc`.
- **D-C9.9** (from C9 Condition test): The test runs in both directions: a sharp fall in failures from one date raises an entry under "New approaches that worked".
- **D-C9.11** (from C9 Sufficiency and certainty): Any estimate that passes A2 can compute the effective sample.
- **D-C9.14** (from C9 Candidates): Dead ends show on their own line.
- **D-C9.17** (from C9 Rivals): The challenge question also covers facts and the most-cited reference sources.
- **D-C9.19** (from C9 Founding cases): A learned heuristic shows, for display only, "found in cases by" with the makers of its founding cases (pseudonyms where they apply). The member's own view lists the records learned from or stated by them, with uses by others since their last look. Neither changes ranking (I3).

## C10. Scopes and safety

- **D-C10.1** (from C10 A session's scope): In an unresolved directory, the agent proposes a scope (advisory).
- **D-C10.2** (from C10 Restrict, move, widen): After a foresters' bulk widening of a selection, the authors of the records get one notice.

## C11. Capture and intake

- **D-C11.1** (from C11 Capture paths, Live capture): Live capture also records what the agent notices, concerns it hears, and approaches it does not try, as notes (advisory).
- **D-C11.3** (from C11 Session review, timing): Session review runs after the *review idle time*, at once when work ends, and on sleep, within the *client review budget*; an explicit end may exceed the budget. Sessions with a hand-off or an open item go first.
- **D-C11.4** (from C11 Session review, drafts): Session review also proposes reference-only sources for documents relied on, and corrections where the session's evidence contradicts a delivered record; the member confirms each.
- **D-C11.5** (from C11 Session review, drafts): Session review also proposes `waiting-on` items from recommendations with an addressee, shaped as X4 drafts them; the member confirms each.
- **D-C11.6** (from C11 Intake, order): Judge calls for addressed items and live captures go first; a bulk import takes at most the *bulk import share*.
- **D-C11.8** (from C11 Intake step 4, compare): Class-dependent notices fire when the class is first derived.
- **D-C11.9** (from C11 Session review, drafts): At work end, for an achieved case whose stated conditions match other registered instances (found by the C7 Reached lookup), session review proposes one `waiting-on` hand-off per owner of a matched instance; the member confirms each.
- **D-C11.10** (from C11 Session review, drafts): Session review proposes a state-over-time fact for a declared-key value that the session's attempt changed on a registered subject; the member confirms it.

## C12. Delivery

- **D-C12.1** (from C12 Moments, start): The start moment chooses the documents and facts most cited in the session's situations or about its subjects.
- **D-C12.3** (from C12 Moments, start): A new scope uses the team's nearest situations for orientation (by its description and `project` facts before its first query).
- **D-C12.4** (from C12 Moments, subject): For a source or passage, the subject brief gives the cases that cited it and set it aside, with reasons, the reader's situation first.
- **D-C12.5** (from C12 Moments, subject): For a group, the subject brief goes one step down from the group.
- **D-C12.6** (from C12 Moments, subject): A source version leads with its own causes.
- **D-C12.7** (from C12 Moments, subject): Subject brief order: the floor, open items, an attempt that removed a serious or critical cause on the instance, then within relevance tier 1 records not yet delivered to or seen by this reader, then by observed time, newest first; then ranking; then an `older` pointer to records since the reader last saw the subject.
- **D-C12.8** (from C12 Moments, subject): A taker or successor also gets each referred case in full, with declined trials and refs.
- **D-C12.9** (from C12 Moments, situation): A procedure that no case used shows `no-uses`.
- **D-C12.10** (from C12 Moments, situation): The validity line shows `no-uses-here` when the situation has closed cases since the procedure's version and none cite it.
- **D-C12.11** (from C12 Moments, situation): In the known situation, warnings come first.
- **D-C12.12** (from C12 Moments, check): Check results are framed as checks.
- **D-C12.13** (from C12 Notice reasons): `item` notice: a work item waiting on the member is added, returned, or done.
- **D-C12.14** (from C12 Notice reasons, `condition`): The watch list also holds the condition groups a test or member linked to the warnings of the session's situations.
- **D-C12.15** (from C12 Notice reasons, `condition`): Watch-list matching is lexical, by bin, or by containment, never by a judge.
- **D-C12.16** (from C12 Notice reasons, `date`): At a date moment, the `date` notice is the moment's pointer, with the date moment's recipients and surfaces.
- **D-C12.17** (from C12 Detecting moments, Stuck): A stuck offer comes once per sign kind per session and situation.
- **D-C12.18** (from C12 Detecting moments, Before a conclusion): The conclusion check result has at most the *check reply lines*.
- **D-C12.19** (from C12 Detecting moments, Before a committing action): Commit patterns and conclusion phrases ship in each enabled conversation language.
- **D-C12.34** (from C12 Detecting moments, Before a committing action): A team that runs jobs adds job and run submission (queue, start a run or sweep, train, evaluate on a held-out set) to its *commit patterns*.
- **D-C12.36** (from C12 Detecting moments, Stuck): A team can add to its *stuck patterns* a sign that fires when the bound open case has failed attempts in two or more sessions.
- **D-C12.37** (from C12 Moments, check): A team whose actions often act on groups can make *check results*, at a committing-action check on a group, grow with the number of member instances that have results, up to the *group check results* (a [team decision], A1).
- **D-C12.38** (from C12 Urgent): In a `high-stakes` scope, an urgent floor item for a member with no live session also goes, content-free, to an outside notification service on the allowlist (C1 Data boundary).
- **D-C12.39** (from C12 Moments, subject): The subject brief also lists, after the contents C12 requires, within what remains of the *subject brief caps*, achieved cases with `checked` evidence or `mechanism: confirmed` on a sibling of the instance (C4) or resting by `uses` on the subject, that the reader has not seen.
- **D-C12.40** (from C12 Search and ranking, Applicability): At a committing-action check, an `if-applied` value on the other side of a bound that a stated hazard, contingency, or decision in force about the instance names shows one `crosses-bound` line, naming the record and its owners.
- **D-C12.41** (from C12 Moments, subject): Within what remains of the *subject brief caps*, the subject brief also lists the instance's own closed cases that the reader has not seen, within the scope's activity window: the latest one in the session's situation, whatever its outcome, and those with a serious or critical warning or a `recurrence` counter.
- **D-C12.42** (from C12 Moments, start): The changes line at start holds one `new-watches` pointer: the count of contingencies created in the scope since the member's last session, and the command that lists them.
- **D-C12.43** (from C12 Moments, situation and subject): For a record that reaches a session through a predecessor, a sibling, a `within` group, or a `same` partner, one differing-conditions line lists each declared state key or containment child that differs between the record's instance and the touched instance, or that only one of them holds. The line has no rank or applicability effect.
- **D-C12.44** (from C12 Moments, start): For a member who is not active in the scope (C4 Active in a scope) and is past the *orientation sessions*, the changes line at start holds one pointer: the command that lists pages of subjects the scope's open cases concern, decisions in force, and recurring situations. The list holds no procedure and no heuristic.
- **D-C12.20** (from C12 Search and ranking, Queries): The abstract restatement of a query runs off the hook path. Until it is ready, a moment shows "From other fields" as `pending`.
- **D-C12.22** (from C12 Search and ranking, Ranking): Applicability order: stated conditions matched; no conditions stated; `conditions-not-known`; `may-not-apply`.
- **D-C12.23** (from C12 Search and ranking, Ranking): Support order continues: for facts, the evidence ceiling, then reliability; for documents, reliability, then source use.
- **D-C12.24** (from C12 Search and ranking, Ranking): The last ranking step is the ranking-model score.
- **D-C12.25** (from C12 Search and ranking, Ranking): A `copy` or `equivalent` group is led by a member with no lowering cause, else the parent, marked `copies` with the count and the scope and passage where a variant differs.
- **D-C12.26** (from C12 Search and ranking, Missed searches): The board groups missed searches by situation until a bound case closes achieved or partial.
- **D-C12.27** (from C12 Interruption budget): Push order: notices on the session's open case, concerns, questions; then relevance.
- **D-C12.28** (from C12 Urgent): A `wrong` or `outdated` correction in full effect of a record the session applied, cited, or was given comes first within the budget.
- **D-C12.29** (from C12 Repetition): One action over many instances gets one delivery that lists them.
- **D-C12.30** (from C12 Repetition): Once per session: a subject brief per subject; a notice per record and reason.
- **D-C12.31** (from C12 Repetition): The `seen-before` line of a procedure also shows its validity line.
- **D-C12.32** (from C12 Repetition): Orientation never collapses.
- **D-C12.33** (from C12 Recording use): The show command lists `applied` uses, minus `execution` and `not adopted` failures, as `per protocol`.

## C13. Questions

- **D-C13.1** (from C13 Routing): The Questions panel is for any reader.
- **D-C13.3** (from C13 Delivery): The agent appends a question in the conversation language, with its choice keys.
- **D-C13.4** (from C13 Delivery): After the questions that CONTRACTS puts first, queues follow the kind order of the C13 table, then recall frequency.
- **D-C13.5** (from C13 Delivery): Outcome questions past `outcome_due` go as one line, `outcomes-waiting` (their count and the board), counted once.
- **D-C13.8** (from C13 Recurrence, limit): One per recurrence pair.
- **D-C13.9** (from C13 Support, limit): One per change.
- **D-C13.10** (from C13 Re-check work): Trigger: a copy passage edited with no parent change. Answers: Copy updated: a new version. Keep local variant, with a reason: a local difference. Parent wrong here, with a reason: a `parent wrong` local difference, which corrects the parent (C5).
- **D-C13.11** (from C13 Re-check work, limit): A bulk answer.
- **D-C13.13** (from C13 Same?, limit): Alias: one per write batch. `possible` basis: one batched question per owner and trigger.
- **D-C13.14** (from C13 Same?): The corrected record shows its `may-rest-on` count.
- **D-C13.15** (from C13 Same?): An expired question is raised again only on a new trigger (a new version of either side, a parent change reaching a copy, a member reading both), routed to a member who read both.
- **D-C13.17** (from C13 Still true, trigger): Of an aged or differing pair, the question asks about the most-recalled fact.
- **D-C13.18** (from C13 Still true): Trigger: a lapsed correction of a first-hand record (C6). Recipient: a member first-hand for the instance.
- **D-C13.19** (from C13 Still true, limit): Once per session; for a lapsed correction, once per correction; for an aged contingency, once per contingency each time it becomes `aged`.
- **D-C13.20** (from C13 Re-check work): An inherited `changed: material` on a decision in force also gives its owners the Re-check work question, one per owner and trigger.

## C14. People, continuity, and views

- **D-C14.2** (from C14 Hand-over in a scope): At work start the successor sees one `handed-over` line with the count of handed subjects and how to list them.
- **D-C14.4** (from C14 Work items and project state): Done and return each tell the author once.
- **D-C14.5** (from C14 Work items and project state): When refs name a running session, its questions route to the taker.
- **D-C14.7** (from C14 Work items and project state): The author and foresters see `receiver lost access`, `receiver left`, or `untaken` (with for how long) after the receiver's next working period, else after the *hand-off wait*.
- **D-C14.8** (from C14 Work items and project state): An untaken item's notice then goes, as a pointer, to whoever routing steps 3 and 4 name, in working hours.
- **D-C14.10** (from C14 Adding documents): A member can mark a source reference-only.
- **D-C14.11** (from C14 Adding documents): The agent adds documents the member asks for and ones it produces worth keeping (advisory).
- **D-C14.12** (from C14 Watched paths): A changed file is versioned after the *watched-file quiet period* or at work end.
- **D-C14.14** (from C14 References): In member writing, a dismissed document is proposed as set aside.
- **D-C14.15** (from C14 Finding documents): Find returns the best page with the sources it cites; then other pages and sources with title, scope, causes, version, publication time, summary, source use, the best-matching passage, how many cases cited it for a situation with a reason, and how many set it aside with the top reason (the reader's situation first); then "From other fields".
- **D-C14.16** (from C14 Finding documents): A procedure's verbatim steps come first, with its validity line.
- **D-C14.17** (from C14 Show): The show command on a source gives metadata, versions, causes, summary, a copy's local differences, and inbound links by type with counts; on request it lists those links, set-aside links included, or the nearest sources. On any record it can also show its `basis` links with author and time, its evidence class and ceiling (for a fact) or reliability (for a source), differing pairs, changes since a time, open items, citing records, the rules that shaped it, a page or source at a date, or a source's changed passages between two versions with writer and class.
- **D-C14.18** (from C14 Pages): A subject or situation with open differing pairs or pending corrections shows `open-disputes` (their count and a pointer).
- **D-C14.20** (from C14 View statement correction): When a linked record is wrong, the member is sent to correct that record.
- **D-C14.21** (from C14 Situation pages): A situation page, and a subject page with open cases or with cases in more than one situation (grouped by situation), also show what is established, ruled out (dead ends, set-aside options), open (open cases, untried ideas), in progress, and competing, with open concerns and decisions in force (C3 `decision`).
- **D-C14.24** (from C14 Show): The show command on a stated hazard, a contingency, or a heuristic lists the instances it matched, or the siblings of its founding cases' instances (C4), each with its state: matches, key unknown, no longer matches, acknowledged, used (with outcome), declined (with reason), or not delivered.

## C15. Integration

- **D-C15.1** (from C15 Common input and output): Each adapter ships the mapping for current tool versions; install verification checks it.
- **D-C15.2** (from C15 Logging): One log line per adapter call: event, duration, result (operational).
- **D-C15.3** (from C15 Session environment): The client passes the session id, the tool, and the scope to the agent's commands through the tool's environment mechanism, else a session file keyed by the session id (never shared by two live sessions).
- **D-C15.4** (from C15 Tool protocol): The scope of a tool-protocol session is asked at the session's first write.
- **D-C15.5** (from C15 Queue): Order (operational): text events first, then other payloads smallest first.
- **D-C15.6** (from C15 Queue): Work end sends at once; an unsent hand-off is printed.
- **D-C15.7** (from C15 Queue): On a recorded expected departure, the client runs pending session reviews and drains its queue before the leaving date, and shows the leaver and the foresters any backlog left at that date.

## Part II

- **D-PartII.1** (from Part II preamble): Each module states when it runs, and adds its own tunables and acceptance properties.
- **D-PartII.2** (from Part II Values while off): The writer of a value that only an off extension defines is told which module to turn on.

## X1

- **D-X1.1** (from X1 Menu): A candidate's inclusion probability rises with its smoothed age-weighted success rate.
- **D-X1.2** (from X1 Menu): The draw is reproducible from session, situation, and episode number.
- **D-X1.3** (from X1 Holdout): The champion line shows the decline that the holdout can detect.
- **D-X1.4** (from X1 Rescue and crossover): Members learn of holdouts in settings, in the show command, and once after the episode.
- **D-X1.6** (from X1 Analysis): The show command reports episodes per arm, missing outcomes, crossovers, rescues, and the difference on a declared measure, which decides nothing.

## X4

- **D-X4.1** (from X4 Documents as capture): The judge asks the adder when it is unsure of a document's class.
- **D-X4.2** (from X4 Documents as capture): Recounts and record streams propose work items from stated commitments; the adder when a member, else its principal, confirms them once per write batch.
- **D-X4.4** (from X4 Record streams): An entry naming an unregistered instance of a registered kind proposes a subject; the adder when a member, else its principal, confirms these once per write batch.

## X5

- **D-X5.1** (from X5 The board): The board also has these panels: Situations (situation pages; the reader's interest areas); Subjects (pages of owned subjects).
- **D-X5.2** (from X5 The board): The board shows an optional gloss in the member's conversation language beside the stored English.

## X10

- **D-X10.1** (from X10 X5 foresters' view): The foresters' view shows retention batches.

## X11

- **D-X11.1** (from X11 Declared patterns): The judge runs only on entries that are not routine and not counted exceptions.

## X12

- **D-X12.1** (from X12 A6 Foresters): The forester commands include the scope setting region.

## A5

- **D-A5.1** (from A5 Stuck output): The stuck output shows its sections in this order, and empty sections do not show: header; champion; challengers; differing positions; dead ends; untried ideas and retry candidates; from other fields; considered and set aside; similar cases; also working on this now; how to record a use.
- **D-A5.2** (from A5 Stuck output): The champion section also shows: kind and statement; its failure rate when shown against not shown, with its interval and episode count; its causes; its certainty with reasons.
- **D-A5.3** (from A5 Stuck output): A **dead ends** section shows dead ends with `why` and basis.
- **D-A5.4** (from A5 Stuck output): A **considered and set aside** section shows options set aside, with reason and case.
- **D-A5.5** (from A5 Stuck output): A **similar cases** section shows situation, attempt, outcome, causes, and one key fact with its observed time.
- **D-A5.6** (from A5 Stuck output): The **also working on this now** section also shows since when the other sessions work, and their open cases, only to readers of their scopes.
- **D-A5.7** (from A5 Stuck output): The last section tells how to record a use.
- **D-A5.8** (from A5 Stuck output): The ask command shows one section per kind, with `identical` records and copy groups collapsed.
- **D-A5.9** (from A5 Markers): Annotations follow the id: `degraded-search`, `checking`, `translated` (with the language), `reference-only`, `as-of` (with the date), `variant` (with the scope), and `from-restricted`.
- **D-A5.10** (from A5 Markers): The show command lists all markers.
- **D-A5.11** (from A5 Injected text): The injected text tells the agent, in this order (the items CONTRACTS keeps are in brackets):
  1. the team keeps shared knowledge, which informs and never directs;
  2. reason first; a team procedure, or what happened before on this subject, shows when the situation is recognized, so check it before improvising; [when stuck, run the stuck command; before a conclusion the member is likely to act on, run the check command; pass observed conditions;]
  3. [capture what you choose to try, with the sources you relied on, and later the outcome;] (outcome `open` if not known; sources in order); when several people will assess the outcome, state the success test when the case opens; note what you notice, ideas you do not try, concerns, and what the member tells you that you did not know; record the argument for any result you derive;
  4. [correct what proves wrong, or flag it;] (also outdated, broader than it says, or unsupported);
  5. if a result bears on what you are about to do, tell the member why; [always relay a warning that applies; if the member goes ahead, record the acknowledgment with its reason;] do not relay `may-not-apply` or `seen-before` results unless asked, except those the check moment lists as resting records; with the member's agreement, record one that bears on someone else's decision as a concern about that decision;
  6. restate a work item waiting on the member, then take it, or return it with a question; [at the end of a unit of work, record next items and hand-offs;] (and contingencies);
  7. record uses of suggestions; write records and queries in English; [name outsiders by role, not by name.]

## A6

- **D-A6.1** (from A6 Commands): Output uses full ids.

## Tunables of defaults

Only the defaults above use these tunables. An implementation that takes a default also takes its tunables; a forester changes them as CONTRACTS A1 says.

| Tunable | Default | Bound | Measured by | Used by |
|---|---|---|---|---|
| Build budget | 20 calls per record | None | Model cost | D-C2.4 |
| Client review budget | 100 calls per member per day | None | Model cost | D-C11.3 |
| Fidelity sample | 1% of derived items | None | Rebuild fidelity, Find success | D-C2.7 |
| Fidelity check cadence | Weekly | None | Hook health, Durability | D-C2.7 |
| Review idle time | 15 minutes | None | Capture coverage | D-C11.3 |
| Check reply lines | 2 lines | None | Stuck resolution, Push yield | D-C12.18 |
| Hand-off wait | 7 days without a schedule | None | Hand-over completeness | D-C14.7 |
| Watched-file quiet period | 2 hours | None | Page lag | D-C14.12 (also LATER) |
| Path display depth | 3 | None | Propagation reach | D-C7.5 |
| Bulk import share | Half of step 2 | None | Intake lag | D-C11.6 |
| Group check results | 5 | None | Push yield | D-C12.37 |

## Checks of defaults

These checks test the defaults above, in the form of CONTRACTS A2. They came from A2 properties that tested a default; the normative part of each stays in A2.

- **I1-c, failed limits** (tests D-C2.6): A build that fails a switch check lists the limits it failed.
- **I6-f, many instances** (tests D-C12.29): One action over many instances gives one delivery that lists the instances.
