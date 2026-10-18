# Requirements and current decisions

Two lists. **Requirements** are conditions the product must meet; a design change never weakens one. **Decisions** are the current means of meeting them, stated as decisions; where a decision names a CONTRACTS section, its rules live only there; the loop may change a decision when a review shows a better one, and must log each change with its reason in `LOG.md` § "Decision changes" so the maintainer can reverse it.

## Requirements (R) — must hold

### Who and how much
- R1. **Many people.** Teams of dozens of people use one knowledge base at the same time, for years. Every mechanism must work when many people write, read, correct, and disagree concurrently, and when most of the knowledge was written by someone else.
- R2. **Scale.** On the order of 100,000 cases, plus documents, with no loss of quality in search, recall, or learning.
- R3. **Parallel work.** Each person runs several agent sessions in parallel and switches between projects and documents. Context-switching cost is the bottleneck to reduce.
- R4. **Agents in between.** People retrieve knowledge through their agents. Supported tools: Claude Code, Codex, and pi; nothing depends on a particular tool or model.
- R5. **A screen for people.** People see current state on a screen made for people, without asking an agent.

### What it holds
- R6. **Documents.** Artifacts made with agents, notes and writing by team members, and outside material (papers, PDFs, web pages, links), in one place and easy to find.
- R7. **Know-how and facts.** Problem-solving know-how (what to check first, how to diagnose, how to define, how to break a problem down) and facts are both in scope.
- R8. **Generalization.** Knowledge from one situation helps in other situations, including other domains.
- R9. **Current state.** Whether documents and knowledge are still valid, how work stands, and what changed recently.

### How it behaves
- R10. **Advisory.** Nothing the system holds directs or forces an agent or a person. The agent and the person keep their own judgment and usual checks.
- R11. **Low friction.** Nobody has to write knowledge articles or approve entries for knowledge to accumulate.
- R12. **Correctable.** Wrong knowledge can be found and corrected, and the correction reaches whatever relied on it.
- R13. **Learning stays alive.** Accumulated knowledge must not stop the team from finding better approaches.
- R14. **English.** Stored knowledge, documents, and queries are in English; other languages are for conversation.

### Durability and safety
- R15. **Durable.** No source record is lost, including under concurrent changes. Derived indexes and views can be rebuilt, so models can be replaced. History remains. A person can read the source records without the system.
- R16. **Safe.** Credentials and personal data stay out (masked before storage and before anything leaves for an outside service), leaked content can be removed completely, and access to sensitive material can be restricted. The team controls where its data goes.
- R17. **Infrastructure is open.** Servers, hosted services, and vector databases are allowed.

### Process
- R18. Do not coin a central term without logging it as a decision change.
- R19. English deliverables are written natively in English, not translated.
- R20. Follow the gitjig-pi document form: MISSION ("What this exists for", "Success looks like" with bold keyed names and an em dash, "Explicitly NOT goals", "Stakeholders", "Last reviewed"), PRINCIPLES, DESIGN, and CONTRACTS, each decision in one home; each contract has a Basis line. Living documents pass the erasure test: no plans, no "yet", no history narration. (Changed after round 23 by the maintainer's decision: four documents instead of MISSION and SPEC.)

- R21. **Broad MISSION.** MISSION states direction and observable outcomes as broadly as possible. Mechanisms (timing of recall, gates, champion/challenger, data layout) belong in DESIGN and CONTRACTS, never in MISSION.

## Decisions (D) — current means, changeable with a logged reason

Source: the maintainer's answers on 2026-10-15 (Q1–Q21, O1–O8, later agreements).

- D1. Document hub and knowledge reuse are two jobs of equal weight. (Q1)
- D2. Syncing from external work tools is out of scope. (Q2)
- D3. Shared by default across the team; restricted projects and documents are the exception. (Q10)
- D4. Conflicting heuristics from different people, and conflicting adopted decisions, are all shown; the user decides. (Q11; decisions added in round 11.)
- D5. Capture is automatic with no confirmation step; an occasional short multiple-choice question is allowed and may be ignored. Pausing and `capture: off` are opt-outs. Text about a member's private life, and text identifying a confidential outside person, is routed away from wider scopes. Rules: CONTRACTS C10 (Masking), C11 (Pausing). (Q8, Q17-6; rounds 16, 18, 20.)
- D6. Trust comes from provenance, results of use, correction with history, and marking unconfirmed knowledge. (Q9)
- D7. Transcripts are not kept; only recorded knowledge and supporting quotes are. (Q15)
- D8. Documents are rewritten into linked wiki pages kept current; originals are kept unchanged as sources. (Q18)
- D9. A decision is a hypothesis; learning comes from attempts and outcomes; failures count. (Q19)
- D10. The know-how unit is a **heuristic**: a candidate for how to approach a kind of situation — what to try, what to check, what is likely going on, how to frame it, how to split it, or what usually does not work; stated at a field level or a general level. (Changed in rounds 1 and 3; see LOG.) Stated by a person or learned from cases; a person's statement does not make it true.
- D11. A heuristic may come from a single case.
- D12. Until a heuristic's record is sufficient, it is judged by reasoning, conditions, and its basis, and shows no figures except its warning count and critical-failure bound. Sufficiency uses statistical confidence with a minimum count. Certainty has a ceiling by kind of evidence, as PRINCIPLES I3 states. Rules: CONTRACTS C9 (Sufficiency and certainty). (Rounds 11, 12, 14; v2 round 3: large consistent effects and fully set computational comparisons.)
- D13. An attempt with no outcome stays open and counts for little. (Q20)
- D14. Facts are trusted by source and time of observation, plus results of use. A newer qualified observation of a changing state replaces the older one; it is neither a contradiction nor a correction. Ratings by different assessors stand side by side within one time window; ratings in different windows form a series, and a single value is taken from the latest window only, by the rule for hazard scales or for agreeing checked ratings. Rules: CONTRACTS C3 (State over time; In effect; Ratings and estimates). (Q21; rounds 19, 20, 23; v2 round 4.)
- D15. Gating holds items back from being shown; it never deletes them. (O1)
- D16. Every incoming item is checked for an existing or similar case, using a vector database.
- D17. Same situation and attempt with a different outcome: keep both as counter-cases (`counter` degree `outcome`); both outcomes count, and neither is marked differing. (O2; degree added in round 19.)
- D18. A repeat adds evidence to the existing case; repeated problems appear on the people's screen. (O3)
- D19. "Would the same judgment be hard without this record?" is a weight, against the model plus the existing knowledge base, stored with the model version. (O4) Computed cheaply for all cases from recall, with a headless model check on a sample that calibrates it. (Changed in round 2.)
- D20. Heuristics arrive only at decision points (stuck, check, situation), with procedures also in orientation and contingencies in their target's subject brief; a delivered heuristic returns once when a condition it failed or warned under appears. Rules: CONTRACTS C12 (Moments). (O5; rounds 2, 13, 15, 18, 20.)
- D21. Team-specific facts arrive early, when the agent starts to work on what they describe. A new fact or source about a subject the session touched also arrives once, during the session, as a notice. (O6; the in-session notice added in round 14.)
- D22. "Stuck" is detected by the agent's own judgment and by mechanical signals. (O7)
- D23. When stuck, show at most one champion and the challengers; only a randomized effect in a decidable cluster names a champion, so most clusters have none. The randomized comparison is on for decidable situations; foresters can turn it off per scope. Warnings have a standing that only evidence about the cause lowers, and a per-person display. Trends rise by ratio or recurrence and clear only by evidence. Rules: CONTRACTS X1 (champion), C8 (warnings, trends). (O8; rounds 11, 12, 14, 16, 18, 19, 20; v2 rounds 2 and 3; scope fix, GS76.)
- D24. Git is not part of the product.
- D25. The source of truth is an append-only event log kept by a server, exportable as plain text.
- D26. Background review of sessions runs where the session record already is: on each person's machine for client sessions (transcripts never leave it), and on the server only for a tool-protocol session record that the server already holds (no transcript moves). By default, excerpts that passed every safety check (on the client, or at intake for a server-held record), judged ones included, go to the shipped models on the team's server, which keep nothing; a person can name a local or another allowed model. (Changed in round 12; clarified in round 16.)
- D27. Facts are one kind of object, shared by wiki pages and cases.
- D28. Restricted knowledge reaches outsiders only as a release record that a person writes, or as a content-free count above the aggregate minimum; below that, nothing shows. When a serious or critical restricted warning applies outside, the scope's readers decide whether to release a version; a release is also offered at work end and before a retention batch in a restricted scope expires founding or method cases. Rules: CONTRACTS C8 (Restricted warnings), C10 (Partitions and counts), C13 (Release), X10, X2 (matching). (Rounds 2, 15, 18, 19, 20.)
- D29. Complete removal is the only exception to append-only; the log records that a removal happened. Deny rules are per scope and applied by readers' clients. Text that waits for a check is held durably in the quarantine until it is logged, returned to its author, or dropped by its author or for a refetch; nothing is dropped silently (the I1 exception). Rules: CONTRACTS C10 (Removal), C11 (Intake step 2), C15 (Queue). (Rounds 16, 18, 19, 20, 23.)
- D30. The occasional question mostly asks about outcomes ("How did that turn out?").
- D31. At session start, give project state and related documents and facts. When the situation is recognized, the procedure, the learned method, and a known situation (from a touched instance, its predecessor, or a sibling in a shared group) may arrive once; with none, open cases of other sessions show. Other heuristics wait for the stuck and check moments. Rules: CONTRACTS C12 (start and situation moments). (Rounds 11, 13, 16, 18, 20.)
- D32. A team has **interest areas**, by default from the situation clusters it works in; members can change them. Knowledge outside them is captured and weighted lower, or held out of automatic recall. Field-specific extensions (CONTRACTS A3 and the Part II extension modules) are turned on per interest area, or per scope for safety and visibility modules. Rules: CONTRACTS C4 (Situations and areas). (User decision after round 15.)
- D33. DESIGN and CONTRACTS are built on PRINCIPLES.md: eight primitives (event, record, subject, situation, link, scope, moment, question), five link types (`about`, `basis`, `same`, `counter`, `replaces`), and ten invariants. A mechanism that needs an exception the invariant does not name is a design error; PRINCIPLES names each exception. A new need is met by a new value of an existing primitive before a new mechanism. Records have four kinds (source, case, heuristic, fact); notes are facts with a speaker; adopted decisions are cases with a status. Knowledge crosses fields only as an abstract restatement of a method, marked with its origin (the reading of R8). (User decision after round 17; named exceptions after the rule audit of round 20.)
