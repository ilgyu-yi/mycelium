# Contracts

Part I is the core. Part II holds modules. Part III holds the tunables, the acceptance properties, the field extensions, the measures, the output content, and the commands.

# Part I. Core

## C0. How to read

- These contracts define observable behavior. An implementation declares its supported tools and environment.
- Tables that define attributes, states, or mappings are normative. **Must** and **never** state enforced requirements. An **advisory** rule states what an agent, a model, or a person is expected to do; nothing checks it, and the system behaves correctly either way.
- Each section starts with a **Basis** line: the DESIGN mechanism (M*n*) and the invariants (I*n*) it serves. A rule tagged with an invariant applies that invariant; it does not restate it. Conflicts follow PRINCIPLES Precedence.
- A **change class** in brackets, such as [team change], names how a change takes effect under M4. A rule with a class states no timing of its own.
- A *tunable* is in italics. Its default and bound are only in A1 or its module's table; a tunable that only DEFAULTS uses is in DEFAULTS.
- A marker is named in backticks, such as `no-record-yet`. The contract states what it tells the reader and when it shows. Its rendering, the wording of questions and injected text, command syntax, and test fixtures are in the reference, which is not normative.
- DEFAULTS.md lists recommended default behaviors. It is not normative: an implementation may choose otherwise while it keeps every contract here. A default that an invariant, a requirement, or the consistency of shared state turns out to need moves back here. A DEFAULTS entry never writes a link, cause, record, containment fact, or failure value, and never changes how one or a champion figure is derived from the log; it can propose one, which a member writes by confirming it, or advise the agent what to capture (advisory).
- A rule that needs a second member says what happens without one. A rule that uses a linked repository says what applies without one. A rule that needs recurring situations says so. Work outside agent sessions enters by the board, the outcome command, documents, and questions.

## C1. Deployment and principals

**Basis:** M0, M8, M12; I5.

| Component | Runs on | Role |
|---|---|---|
| Server | Team infrastructure or a hosted service | Keeps the log; builds derived data and indexes; serves the API, the tool protocol, and the board; enforces scopes. A host the team does not control is an outside service |
| Client | Each member's machine, or a service's environment | Commands, adapter, offline queue, session review, watched paths, safety checks before text leaves the machine |
| Shipped models | Server; safety classifiers also on the client | The **judge** (classifies, splits, compares, checks), the **writer** (views), the **builder** (learned records, rivals), a model that places text for similarity search, a model that orders candidates by relevance, small safety classifiers. Together: the **system models** |

- **Sign-in.** By the team's identity provider, else by the server acting as one; one credential per principal and device. During an outage, issued credentials stay valid for the *identity-provider grace*. Revocation applies at once.
- **Model qualification** (I1, I5). A model serves a function (safety, language, translation, compare, split, restatement, writing, building) only after it passes the *qualification thresholds* on the shipped fixture; for judged safety and translation, per enabled conversation language. Text in a language with no qualified safety model waits in the client queue and shows `language-not-qualified`, naming the language, until the quarantine age (C11 Translation). Enabling a language enables its locale rule set. With no qualified model, the function is off and shows `unavailable`, except where a rule names its fallback (above, C11 Translation, C12 Retrieval).
- **Reference deployments** (operational): the *small reference deployment* and the *large reference deployment*.

**Principals and roles.**

| Principal or role | Contract |
|---|---|
| Member | Has settings: conversation language, time zone, and **schedule** (working hours, which rotations set: off rotation is out of working hours, not offline; dated exceptions; set by the member or a watched calendar file, else derived from activity and marked `derived`) |
| Service | Writes `on_behalf_of` a member, a group, or a scope's foresters, who get its questions. Reads only scopes a forester grants, never more than its principal. A member's own cloud sessions use a **delegated session** credential |
| Forester | Answers questions as a member. Each scope has at least one; a member is the only forester of their private scope. Sets a scope's members, linked locations, and settings; widens it; releases and removes its records. A team-level forester also sets the data boundary and safety rules. **Safety rules** are the enabled locale rule sets, deny rules, and declared sensitive classes (C10); a declared sensitive class is set only by the foresters of its restricted scope who read it. Over a restricted scope only a forester who reads it acts. Nobody adds themself to a restricted scope |
| Group | A named set of members. A restricted scope lists included and excluded members and groups; exclusion wins |
| Owner | A subject or scope can name owning members or groups. Owners get questions and notices and follow what they own. They never gate a change (I8, I9). An owner file in a linked repository only proposes owners. A scope can name a **triage group** for routing |

- An extension may add guests and shared devices (X8).
- **Shared audiences** (I5). A service or shared surface that serves several people at once reads only the intersection of its identified audience's scopes (C10 intersection scope); with an unknown audience, only team-visible scopes. An audience with any unidentified person counts as unknown. A service acting for a group reads the intersection of its members' reads. Such a service or surface records no `seen` or acknowledgment for display unless the viewer identifies; its floor treats every warning as unseen.
- Requests about records in another member's private scope go to that member; the requester learns nothing else.

**Data boundary** (I5).

- A team-level forester keeps an allowlist of outside services: what each may receive (per scope where needed), retention terms, deletion support, and an optional spend cap per period that the entry states; once a cap is reached, no model call goes to that service until the next period. Nothing else receives anything. A model call takes text only from scopes whose boundaries all allow its service.
- Agent-tool providers are outside services. Recall never delivers a scope's records into a session whose model provider the scope excludes; the session is told why.
- A server on a host the team does not control accepts text only for scopes whose boundary allows that host.
- Sending more of a restricted scope's text anywhere needs its forester. Nobody on its exclusion list can propose or confirm it.

## C2. Log and derivation

**Basis:** M1; I1, I2, I9.

**The log.**

- Append-only. The server sequence number gives one team order. A write is acknowledged only once stored on more than one disk or machine. Backups follow the *backup schedule*; restores are tested on the *restore-test schedule*.
- Events are never edited; a change is a new event. Removal (C10) is the only deletion, and the log records it. Each event is encrypted at rest with its own key, wrapped by its scope's key; a removal destroys only the removed events' keys.

| Envelope attribute | Meaning |
|---|---|
| `v` | Schema version; older events are upgraded on read |
| `id` | Unique writer id; a resend with the same id is ignored |
| `seq` | Server sequence number |
| `time` | Server time of acceptance |
| `made_at` | Client time of the write, corrected by the last contact's clock offset and clamped between that contact and `time` |
| `author`, `on_behalf_of` | Member or service handle; a service's principal. An agent or reviewer writes as its member |
| `tool` | The agent tool or surface (`claude`, `codex`, `pi`, `cli`, `board`, `tool-protocol`, `review`, …) and its model when known |
| `session`, `scope` | Agent session id or null; scope id |
| `base_seq` | For a change: the sequence number of the state the writer saw |
| `type` | The event type (below) |

- Order and folds use `seq`. Classification and observed-time order use `made_at`, except observed time on attempts, observations, and records drafted from dated entries (`date-from-source`).
- Server schedules and every window for others to respond (undisputed period, objection lapse, question reroute and expiry) run from `time` (I9).
- An envelope has its event's scope (I5). Exports, the board, and the foresters' view show a restricted event's envelope only to its readers; others see at most a content-free count (C10).

| Event type | Payload | Written by |
|---|---|---|
| `scope.changed` | Name, restriction, foresters, owners, parent, linked locations, watched paths, settings, time sources, hand-over | Members, foresters |
| `member.changed` | Join, leave, role, groups, devices, schedule, expected departure, pseudonym | Foresters, access-role naming, device commands; members for their schedule |
| `config.changed` | Setting, scope or area, value, reason, the measure it should move; member preferences | Foresters; members for their own |
| `record.created` | Kind and attributes (C3); speaker; observed time | Capture, members, document adds, watched paths |
| `record.revised` | Changed attributes; for a source, a new version with label and `base_version` | The author; watched paths; refetch |
| `subject.changed` | A subject's kind, references, containment, owners, `breaking` mark | Members |
| `signal` | An outside change: subject or source; kind (`commit`, `deploy`, `fetch`, or a subject kind's own); the parts named; result | Client watchers, refetch worker, **deploy recorders** (services that write `deploy` signals) |
| `attempt.added` | Case id and attempt attributes, numbered in order by the server | Capture, members |
| `outcome.recorded` | Target attempts, outcome or score, assessment attributes (C3) | Capture, members, answers |
| `link` | Add or remove; from, type, degree, to; reason; position; `since`, `until`; credit | Capture, session review, members |
| `correction` | Target (record, link, attribute, view statement, or batch); kind (C6); reason; evidence; replacement | Members, agents, foresters |
| `support` | Target correction; `confirm` or `dispute`; reason; evidence | Members, agents |
| `record.scoped` | Move, restrict, widen, or hide one record | Members (own; any reader may hide another's record, C10); foresters restrict, move, or hide only |
| `release` | Gate releases of injection-held items (C11). A release record is a `record.created` with `basis` to the original (C10) | Foresters |
| `delivered` | Session, moment, query, record ids and text versions, situation, episode, probabilities, seed, slot rule; `missed`, `crossover`, `rescue`, `offline` | Client, server |
| `trial` | Heuristic or warning, session, `applied`, `declined`, or `acknowledged`, reason, instance, action kind, `late` | Client |
| `question` | Ask, answer, or expire: kind, target, member, text, choices, answer | Client, board, server |
| `seen` | Member, scope; warning ids read (content-free, for display per reader) | Every read path |
| `eval` | Test query, expected ids, supersede | Members |
| `generation.switched` | Generation, model versions, checks, rollback, acceptance with reason | Server, foresters |
| `export.made` | Requester, scopes, destination | Export command, schedule |
| `removal` | Target or deny-rule ids, never content; per-scope counts; reason; places reached and not reached | Foresters |
| `client.report` | Counts only: audits, hook errors, purge acknowledgments | Client |

**Folding and concurrency** (I9).

- Current state is the fold of the log in `seq` order. A later event sets an attribute only when its writer is the record's author or the attribute's earlier writer. For an open case or work item, its current owners and attempt adders also count as earlier writers of pre-outcome attributes; a change to them after the first outcome is [team change] and shows `criteria-after-outcome`. An assessment replaces only the same assessor's. List attributes (conditions, `not_when`, subjects, fields) change by add and remove. Any other change to what another wrote is a correction (C6).
- **Concurrent**: an event whose `base_seq` predates the latest change to its attribute. Both values are differing (C6) and show to both authors under "Changed while you edited". No value wins by arrival order.

**Generations** (I1).

- Derived data belongs to a generation that records its models and prompts. Recall serves the current one. A full build runs on a model change and at least each *build cadence*.
- **Switch checks.** A generation becomes current in one step only after: every correction survives; rebuild fidelity holds; find success stays within the *switch find limit*; unexplained changes to champions, builder `not_when` entries, situations, and subject groupings, and the share of learned text materially rewritten, stay within the *switch stability limit*. Derived data that the current generation did not rebuild stays as the previous generation made it, marked with its model version. After a forester accepts a build under the I9 exception, regrouped records show `changed: regrouped` until the next *monthly check*, and a rollback undoes the acceptance. The previous generation is kept for rollback.
- **Member events bind every build** (I1, I9). Corrections (of view statements too), `not_when` entries, and member edits bind every later rewrite and generation. A corrected fact is never restated from its passage. Member events carry to every part of a split and to a merge's survivor. A retired heuristic stays retired in its successors. A new heuristic that overlaps a retired one beyond the *retired overlap* gets a derived `same` link to it; through that link the retirement reaches only learned heuristics, unless later evidence supports the new one; on a stated heuristic it is a pending correction.
- **Stable ids.** Situations and learned heuristics keep their ids at a member overlap of at least the *id carry-over share*.
- **Ongoing budget** (operational). Model work other than judged safety checks and translation shares the *ongoing model budget*. A drafted record that reports serious harm (I6) does not wait behind live captures (a capture from the offline queue is live).
- A revert of one system model's work never reaches member input (C6).

**Export** (I1, I5).

- Export gives the log and a Markdown rendering of every record and view the requester can see, with views marked derived. An export to a destination outside the team's infrastructure, other than an encrypted partition or a destination its foresters declare readable only by the scope's readers, is text leaving the scope: the *generalization rules* and the masking rescan apply to it.
- Every export (on demand or on the *export schedule*) is partitioned by scope. The team partition is plain text. Each restricted partition is encrypted with an export key its foresters hold, rewrapped at every forester change, unless its forester sets `export: plain` for a destination they declare readable only by the scope's readers. Each restricted export also carries its key wrapped for the custodian and for each current reader, rewrapped at every reader change. Key recovery: the server holds a copy wrapped for the custodian, and gives it to them, or to a person named by access-role naming (C14), only when they become a reader. A scheduled export goes only to a destination its forester approves. Install verification checks for a scheduled export destination off the server's infrastructure; the foresters' view shows the last export's age. Each export with a restricted partition sends one line (requester, destination) to that scope's other foresters.

## C3. Records

**Basis:** M2; I1, I2, I4.

| Common attribute | Meaning |
|---|---|
| `id`, `kind`, `scope`, `author` | Identity; kind (source, case, heuristic, fact); its one scope; writer (member, service, or system model) |
| `speaker` | The author; `member <handle>` for a member's words that an agent records; an outside person's role plus a per-scope keyed hash (C4 `person`), never a name; or the source passage it was drafted from; for a learned record, the makers or outside performers of its founding cases |
| `observed time` | When observed or done, with its time source |
| `conditions` | Key–value pairs (C4), each marked `auto` (client-added), `recorder` (who recorded the work), `after` (known only after the attempt), or `unknown` |
| `review by` | Optional re-check date; after it, `aged` (C7) |
| `fields` | `about` links to `field` subjects, derived by the judge at intake from the record's situation, subjects, scope's declared `fields`, and abstract restatement; member links override them, as for condition values (C4) |
| `about`, `basis` | Links (C5). With no basis, `no-basis`, which names the speaker |
| `status` | Derived (C7) |

- Learned heuristics and facts drafted from a source are derived (I1).
- **Held dates** (`due`, `outcome_due`, `review by`, a stated end of validity, dated subjects) are an instant with an offset, or a date with a declared time zone.
- **Differing held dates.** For a held date with differing values, date moments use the earliest and show both; a work item's severity uses the higher value.
- **Time sources** (I2): Mycelium; a time source a forester declares for a scope (`time-from` names the system); a date the source states (`date-from-source`: a document's stated data date, else `published_at` with `date-inferred`; never the fetch time).
- **Language** (R14). Distinctive symptoms and an attempt's literal inputs (queries, commands, formulas) stay verbatim. Other-language spans are quoted, tagged with their language, and glossed. Intake asks the client to redraft the rest. Translation: C11.

**Sources.**

- A source version is immutable (I1). Attributes: title, `publisher` (venue), `origin` (authors, sponsor or funder, data source), location, identifiers, version label (the site's revision id when it has one), `published_at`, fetch time, adder, **writer**, per-version summary, optional `review by`, `peer-reviewed` (yes/no), and **source kind**: `primary` (a measurement, or a log that an instrument or system writes, of its subject), `compiled` (a dataset or register about other subjects; a review or synthesis), `peer-reviewed`, `maker` (below), `member` (member writing: handbooks, design docs, notes), `preprint` (an unreviewed research report, ranked with `member`; it takes the peer-reviewed kind through its `equivalent` version), `vendor`, `opinion`.
- **Kind order**: `primary` > `maker` > `peer-reviewed` > `compiled` > `member` = `preprint` = `vendor` > `opinion`. `maker`: by the subject's maker or operator; it outranks other kinds only for statements about its subject's specification or state, and ranks as `vendor` for effect or performance claims about its own subject. Peer review is also an attribute of a source version: a peer-reviewed `compiled` version ranks as `peer-reviewed`. A `compiled` synthesis is never outranked by a source on its own `basis` path.
- A funder from the funding passage, or a shared dataset id, is a derived `origin` value resting on that passage.
- **Identifiers**: `document` (DOI, ISBN, standard number, canonical URL; a match is a new version, in a linked repository only within one ref, Version order) or `subject` (CVE id, ticket number; writes an `about` link). Only a fixed-publication identifier (also patent number, standard edition) makes a version **version-pinned**: it stops only `aged`, and its refetched changes are read only for retraction, withdrawal, erratum, or successor wording. A `member` source with no refetch location ages by the shortest *record age* of its subjects' kinds.
- **Passages**: spans of one version, located by the document's own scheme (section, clause, page, cell), else heading path and paragraph, else line range. A passage carries to the next version by `same` (`passage`), else by locator with matching text, else to the most similar span above the *passage carry-over similarity*; between `equivalent` sources, likewise through their English renderings. A passage has **changed** when its text differs beyond formatting or nothing carries it.
- **Writer** (I2, I3): the `author` of the version's `record.revised`; for a watched path, the commit author when the repository is linked (one that maps to no principal is an outside speaker, C3 `speaker`; a bot account registered as a service is that service; then only committed or pushed states are versions; uncommitted edits stay in the member's private scope until committed, I5); else, for a watched path, the member whose own client observed the local edit (the file changed on that device before any sync brought it); else, for a local folder that only one member's clients watch, that no linked scope location covers, and that the client cannot see shared with another person (by its sync service, where the client can read it), that member, unless a forester of the scope that its client settings name declares it shared [team decision]; for a path in a member's private scope, that member; else, for a location that several members write, where a forester declares the location's system of record a time source, its author field; otherwise `writer-unknown`, which never equals the scanning member or the adder, and whose `material` changes are [team change]. `writer-unknown` passages count once per source for C5. A passage's **speaker set** is the writers of the versions that changed it. These are a `member` source's speakers, `first-hand` or relayed per passage.
- **Version order**: declared labels, else `published_at`, else arrival. It sets only the label sequence, never current content (I9). Where the location has an outside system of record (linked repository, synced folder, URL), that system orders these versions and sets current content: its newest committed or saved state is current. In a linked repository, each ref that the linked location names (default: the default branch) is its own location, and a file at each ref is its own source; a version on one ref never sets another ref's current content. Intake never matches sources across refs by identifier, hash, or location; copies across refs are linked `same` (`identical`) while identical; otherwise they get no link and no question. A version inserted before the latest sets no cause.
- **Taking effect.** A new version is logged at once as what the document says, by every path. An `editorial` passage change takes effect at once, whoever wrote it. A `material` change by a writer who is not an earlier writer of that passage: the records drafted from it are [team change]. Such a change carries, as its evidence (C6), the evidence for full effect of the case whose attempt `produced` it, where that evidence bears on the passage.
- **Reference-only source** (I5): identifiers, metadata, location, and masked text of cited passages, with `reference-only`. A later full text is its next version.
- **Renderings** (I1). A non-English or non-text source is kept as is. Its English rendering (for non-text: extracted text, metadata, model description) is derived per version, sets no cause, and is indexed.

**Cases.**

| Attribute | Meaning (I2, I3) |
|---|---|
| `problem` | The problem as it looked before the outcome, within the *problem statement length*, not naming the choice; intake maps it to a situation (C4) |
| `goal`, `done_when` | What the work wanted; an optional success test declared before the outcome (criteria, or an assessor or role) |
| `measure` | Optional name (or a state key, its instance, and an optional condition group), unit, direction, baseline (with its observed time and its instrument or condition kinds), `achieved` and `partial` thresholds, no-effect band (default: baseline to the `partial` threshold). It holds one primary measure |
| `hypotheses` | Optional rival explanations, options, or claim elements (`open`, `supported`, `set aside`); each a part that can rest on passages and inherit causes |
| `set aside` | `about` (`set aside`) links to options and documents not taken, with who, why, and version; display only |
| `decision` | Optional `proposed`, `adopted`, `rejected`, `withdrawn`, `deferred`, with an optional stated end of validity; advisory. **In force** while `adopted`, no newer decision `replaces` it, and its stated end of validity has not passed (C3 End of validity); `differing` does not end it |
| `design` | Optional `controlled` (variants side by side under the same conditions), also `randomized` (assigned by chance, or a computational comparison that I3 counts as randomized). With a design, `answer`: `valid`, `invalid`, or `inconclusive` (the test's validity) |
| `induced` | Optional: caused on purpose (drill, test); the injected fault is never an attempt |
| `severity` | Optional `minor`, `serious`, or `critical`, declared before the outcome; a work item's severity |
| `outcome_due`, `outcome_by_absence` | When an outcome can be known (with none, capture time plus the situation's outcome horizon, C4, marked `due-derived`, for the Outcome question, the date moment, and the open-case cutoff); the window and named check for an inferred outcome (C11) |
| `attempts` | Ordered. Each: `action`; `why` (or `unknown`); `why_source` (`stated` at the time, or `inferred`, with `reason-reconstructed`); `from` (`self`, `member <handle>`, `suggestion <heuristic>`, or `outside <role>` with a masked `performer`); `stated_scope`; observed time; `mechanism` (`confirmed`, `unchecked`, `refuted`, `not-applicable`); assessments |
| `assessments` | Per attempt and assessor: outcome or score, `observed`, `result`, optional `estimate` (value, interval, n), `evidence` (`checked`: a test, measurement, later observation, or checkable source; else `self-reported`), `selection` (`random sample` with rule and probability declared before outcomes, `triggered`, or `routine`), `severity`, `failure_reason` (`approach`, `execution`, `not adopted` (advice an outside decision maker did not follow), `wrong fact <id>`, `conditions`, `inherent risk`, `unknown`), `anomaly`, `harm` with severity, `intercepted-by` (the safeguard subject that caught the event, and the assessed severity, outcome, and failure reason without it, defaults `failed` and `unknown`: a warning at that severity on the guarded subject; wherever the attempt's outcome, failure value, failure reason, or assessed severity is read, the attempt counts with those values from any deciding assessment that records them, under the C9 rules for deciding assessments and horizon strata, and is never a founding success) |
| `quotes` | Short quotes, speaker as handle or role |

- **Outcomes**: `achieved` (goal met), `partial`, `failed` (not met, or worse), `no-effect`, `inconclusive` (no answer possible; closes the attempt), `open`.
- **Final outcome**: the last closing attempt's, partitioned like other summable data (C10): each reader sees the outcome from assessments in scopes they read; when an assessment sits in a narrower scope, outcome questions and the outcome worker route only to that scope's readers; missing while a later attempt is open, until the *open-case cutoff*. A **dead-end case** has only failed or no-effect attempts.
- A `goal`, `measure`, or `done_when` set after the first outcome shows `criteria-after-outcome` and is not used for records. A stated limit that widens is a warning, `criteria-loosened`, which names the earliest limit, its record, and the new value: a `measure` threshold looser than the earliest declared for the same measure in the situation; a material passage change, a `when` or `not_when` change, or a decision that accepts a value beyond the earliest stated limit on an `ordered` condition or measure. "Earliest" counts from the latest `breaking` mark with evidence on the measure's instrument or condition kind; a widening with no such mark still warns. Its severity is the highest among what the limit guards: warnings and stated hazards on its record or resting on its passage or record by `basis`, and stated hazards and contingencies whose stated conditions name its key (if any) on the same instance or `within` group; else the *default warning severity*.
- **Tests.** With a `design`, `answer` sets the outcome for `check`, `frame`, and `decompose` heuristics about running the test. Each variant attempt is assessed on `measure` against the control: `achieved` or `partial` when its interval lies beyond its threshold; a variant chosen after outcomes as the best of several enters the controlled record only through its pre-declared comparison, else with `selection: triggered`; `no-effect` when its interval is inside the no-effect band; `failed` when worse; else, or unless `answer: valid`, `inconclusive`. A claim test with no intervention variant is met by `answer: valid`: a valid negative result is `achieved` and the claim a hypothesis `set aside` with the condition values the test held; a reader under unknown or other values reads it as `open`, with `conditions-not-known` for unknown keys; `failed` only with no valid answer; `inconclusive` when it could not discriminate. An assessment across a `breaking` mark of the baseline's instrument or condition kind shows `before-change` against the baseline and is `inconclusive` by thresholds until a baseline after the mark exists. A `measure` outcome with no `design` shows `uncontrolled`, with the baseline's time, wherever it is shown (I3).

**Heuristics.**

| Attribute | Meaning |
|---|---|
| `kind` | `action` (what to try), `check` (what to look at next), `explanation` (what is likely going on), `frame` (how to define the problem), `decompose` (how to split it), `dead-end` (what usually fails, and why) |
| `level` | `field`, or `general` (field-free; only it crosses fields, I10). A stated `general` heuristic is recalled as `field`, in its author's session fields, until `generalizes` links it to field heuristics in the *general-heuristic fields* |
| `statement` | One sentence, or ordered steps |
| `when` | Its situations as they look before the outcome; for a contingency, the trigger |
| `why` | Why it tends to help; `unknown — observed to work` only for a learned one with an achieved founding case and no serious warning on a founding case; `not-stated-in-source` when drafted from a passage |
| `not_when` | Where it does not apply; each entry names who or what added it |
| `reading` | What each possible result means |
| `origin` | `stated` (a member or a source passage) or `learned` (C9) |

- Required: `statement`, `when`, `why` (except as above). No team figures in its text (I1, I2).
- **Stated**: by the state command, or drafted from a passage stating a rule as a rule (C11; I3). A passing remark is a fact.
- **Binding rules.** A heuristic drafted from a passage that states a rule as binding is never a randomized candidate and retires only by a source version, a withdrawal, or `replaces`; its record shows its uses by application.
- **Procedure**: a `decompose` heuristic stated in a source as ordered steps, each a passage, with `basis` to the source version; linked `about` its situations at intake.
- **Untried** (I7): a stated heuristic with no use, listed under "Untried ideas". With no use within the *dead-end age*, it is `aged`, which only loses it ties.
- **Contingency** (I6): a planned response; an `action` heuristic `about` one target (case, subject, or situation), `when` its trigger, optional severity; on the watch list (C12). Without serious severity and with a target that cannot close, with no match after the larger of its scope's activity window and twice the median interval between readings on its trigger key, same-bin readings included (else the subject kind's *record age*), it becomes `aged`: it loses ties and stays on the watch list, and one Still true question goes to its target's owners. It leaves the watch list only by a correction, a withdrawal, or its target's closing.

**Facts.**

| Attribute | Meaning |
|---|---|
| `text` | One English statement |
| `kind` | `fact` (default), `observation` (something noticed), `claim` (believed without a checkable source), `concern` (a worry about a pending decision), `rating` (an assessed value with assessor and declared method; a declared rating method states its repeatability, a tolerance on its scale), `estimate` (a quantity under conditions: value, interval, n, `design`), `hazard` (a stated hazard, C8: a subject or subject range, stated conditions, severity) |
| `speaker`, `basis` | Who says it; what it rests on: a passage, a file range at a version, a log, a tool output, or a member's statement event (**first-hand** when they observed the subject instance, else **relayed**). A measurement's instrument version, and an `estimate`'s benchmark version, are `basis` (`uses`) links. A figure drafted from a model rendering of a non-text source shows `proxy <extractor version>` and counts as self-reported until a member re-checks it against the original; each disagreement is an adverse item for that extractor version. **First-hand**: the speaker (a member, or a stated outside performer, pseudonymized or not) states that they observed the instance, with the time, or the session read it, or it is a declared state (In effect); one test everywhere; counting follows C5. A first-hand statement by an outside speaker known only by a per-source pseudonym, in an `opinion` or other unreviewed source, is not `first-hand` for C3 In effect, C6, C11 copies, or C13, and counts as self-reported for evidence class, unless a member re-checks it or the source is `primary`; it still counts as a separate occurrence (C5 (b)) until it is confirmed as a `copy` |
| `observed time` | For a version-pinned subject or a version of a kind registered with versions (C4), that version: the fact holds over the whole version unless it states `since`, and In effect still treats an item written later on the same key and version as later, not as about the same time; else as for every record |
| `conditions` | Where it holds; a stated end of validity follows End of validity (below) |
| `design` | `controlled`, `randomized`, else observational; drafted from the methods passage; shown as `observational` or `randomized`; sets the ceiling (C9). A fact resting by `uses` on facts takes the highest ceiling they allow, one lower if they have `before <change>`, `withdrawn`, or `differing` causes (I3) |
| `severity` | For a concern or a hazard |

- A fact rests by `basis` (`uses`) on the facts it is drawn from and on the subject instance whose state it states.
- **Classifications** (I3). The first-hand class covers the observation: that the observer was there, then, and what they recorded. A classification of what was observed (a species, a diagnosis, a root-cause label) is a `rating`: its observer is the assessor, and the protocol, if any, the method. Stream entries on a key declared `assessed` (C4) are drafted this way.
- **State over time** [state over time]. A fact about a changing state (value, phase, owner, containment) holds at its observed time, or over its optional valid range `since`–`until` (which defaults to the observed time; a backdated state is recorded this way). Where a valid time is stated, state over time orders by it. A reading with the current fact's value extends its valid range and writes no new fact. A qualifying reading (In effect) in the same `ordered` bin with a different value replaces the current fact at once [state over time], credit `kept`; it is no propagation trigger and writes no `changed: input` (C7). Any other same-bin reading with a different value is a pending later state (In effect). On a key whose values are estimates, In effect applies instead. When the current fact's bin lies inside a stated hazard's or contingency trigger's `serious` or `critical` range, the reading still writes a warning occurrence on the instance at that severity, counted for trends by C5 (b) and kept as a count on the current fact, as a routine repeat is (C11). Such an occurrence is not a new unseen warning for Display (C8).
- **End of validity.** A stated end of validity sets `until` on a fact's valid range; on a decision case it ends "in force" at that date. It is a held date with lead times (C12 date moment). When it passes with no successor item on the key and instance, the server derives the key's state `ended` from the stated end. `ended` replaces the current value at once [state over time] on any key, as a categorical change, so C7 Trigger and Reached fire; bounds read it as unknown. A fact with no key and instance only leaves its valid range (`until`).
- **In effect.** A statement is **in effect** from its observed time. A later item on the same key and instance `replaces` it at once, credit `kept`, only if it is: a `first-hand` or direct observation (a declared state such as `phase`, owner, or containment is first-hand by its declarer); a cited re-check; a declared time source's entry; or a dated statement in effect from a source of the same or higher kind, from a `member` source only when the passage speaker is `first-hand` for the instance. A source statement replaces a value only when the value's own evidence is of the statement's class or a lower one (C5 Class before kind). Against a current `first-hand` or direct observation of the same instance by another speaker, a statement of a lower class never replaces it: about the same time it is a `counter` (`claim`); about a later time it is a pending later state (below) and raises the Successor question. A record-stream entry stating a value of a declared state key of a registered subject is such a fact, unless it is an `estimate` or a `rating` by the rules for estimates (below) and classifications (above). A value recorded with an instrument, or under a procedure registered as an instrument (a survey protocol, a scanner), or on a key declared `sampled` (C4), is an `estimate` in the key's series, with effort as a condition. The replace-at-once rule for direct observations above does not apply to it; it changes the current value only by the rule below. On a key declared `presence` (C4), a detection is a point state and a non-detection is an estimate. The current value of an estimate key other than a `parameter` key (C4) is the one value of the latest time window (its only estimate, else as Ratings and estimates gives it) when its interval lies wholly in one bin (on a key without fixed bins, on one side of every bound named for the instance, C7 Crossing a bound; for a `presence` non-detection, below the key's declared detection threshold); else the earlier current value stays and the new estimate shows beside it. An estimate in the same bin as the current value (on a key without fixed bins, on the same side of every such bound) writes the same-bin warning occurrence (State over time); an estimate is always written in its series. An estimate with no stated interval shows `interval-unknown`, does not count as lying in one bin, and so never changes the current value alone; its point value still warns by C7 Crossing a bound. The current value is kept per instance and stated condition group: items under different stated condition groups never replace each other, and the "same conditions" qualifier of Ratings and estimates applies here and in C7 Crossing a bound. Any other later item on the key and instance is a **pending later state**, `reported-later-state`, shown beside the current value. It is not a correction and never lapses as a position. Pending later states on one instance and key raise one Successor question (C13) per write batch to the instance's owners at once, urgent when a value lies in a `serious` or `critical` range (C7 Crossing a bound). A pending later state observed before the current value's observed time no longer shows beside it. An item that says the earlier one was wrong is a correction (C6). Only facts about one time that cannot both hold are `counter`.
- `health` is per writer: In effect applies within each writer's values; project state shows each active member's latest value within the scope's activity window, worst first, never unknown.
- **Ratings and estimates.** A `rating` and an `estimate` each have an observed time window. A rating's time window is the window of what it rates: the observation, inspection, or entry it rests on, else the inspection period it states, else its observed day. A rating of an earlier observation (a review, a re-classification) takes that observation's window. Rating windows that overlap, directly or through a chain of overlapping rating windows, count as one time window. Ratings by different assessors in one time window stand side by side, never `counter`. Ratings in different time windows form a series: the latest window is current where one value is needed; earlier ones show on pages and are `replaced`, credit `kept`. Estimates of one quantity, under the same conditions and time window, from different sources are `counter` (`claim`) when their intervals exclude each other, else side by side; an estimate with no stated interval shows `interval-unknown`, is never a `counter` side, and counts as self-reported for risk of bias; a correction `wrong` of an estimate takes full effect only through this test, or by a re-check showing that it misreports its own source; a resolved reproduction counter is an adverse item for the source's reliability. Estimates for different time windows form a series, except on a `parameter` key (C4), where they form none; earlier ones stay, shown on pages, never `replaced`. State over time applies only to point states. An assessor's later rating replaces its own. Where one value is needed, for ratings within the latest window only: on a declared hazard scale (severity, risk), the least favourable by declared direction among ratings with `checked` evidence or a declared method; for other ratings, the `checked` ones when they agree; for estimates on a `parameter` key, the controlled record's (C9) or an extension's pooled value; for other estimates, the latest window's value (on a declared state key, the one In effect gives); else unknown, and all show (I3).
- **Concerns** are `about` a pending decision (a decision case or work item), routed to its owner, given at its check moments, and open until their speaker marks them addressed. A change by anyone else, the owner included, is a correction that needs evidence about what it names. No outcome or speaker departure closes a concern. On the speaker's departure, the owner and independent readers are asked what addressed it; if no answer about this concern comes before its *question expiry* (C13), and the concern has no evidence and no `serious` or `critical` severity that counts for full effect (below), it lapses: it shows only in a count, `lapsed-concerns`, until new evidence. A concern whose speaker is not an active member follows this departure rule from the time it is drafted. A concern of severity `serious` or `critical` that counts for full effect never lapses and is a floor item at the check moments of its decision while the decision is pending (I6 before I9); a concern whose speaker is not a member is one only with a member's first-hand severity or evidence for full effect, as C8 requires before a commitment-under-concern warning counts. The warning for committing under a concern takes the concern's severity.
- An outside person's dispute of a statement about them is a `claim`, their role as speaker, linked `counter` to it (I9).

## C4. Subjects, conditions, situations

**Basis:** M2; I5, I9, I10.

**Subjects.**

- A subject is anything records are about that can change; each has a registered kind.
- **Scope** (I5): that of its creating event. A restricted subject, with its aliases, links, and questions, reaches only its readers. Naming a team subject from a restricted session does not change its scope.
- **Registry** (I8). A kind has a reference pattern, its own `signal` kinds, an age, and optional lead times, order, versions, and type-defining condition keys. Foresters add kinds; members add subjects. At intake, a condition key that names no registered kind proposes a kind: derived, in effect at once for the writer's scope, grouped by the alias and grouping rules for condition values (Conditions and fields). A member or forester can merge or rename it [team change]. Only foresters add or change kinds with safety meaning (`person`, declared sensitive classes). A kind for timeless claims (theorem, definition) registers with record age `never`. A kind with versions, other than `source` and document-backed kinds (whose versions are source versions, C3), has a declared state key `version`, whose value is a subject that names one version of that kind (registered on first use); an instance's **version state** is its current fact on that key. Shipped kinds: source (with versions), outside author or organization, **document-backed** (with versions; `signal`: its source gets a version), **dated** (`signal`: a date such as a contract term), **named entity**, **instrument** (with versions), **field**, **project** (one per project scope; its owners default to the members of its project scope), **person**.
- **Containment** (I9) is a fact about the child (key `within`), with optional `since` and `until`. A subject first created in a project-scope session gets a derived `within` fact naming that scope's `project` subject. An instance can be within several groups, among them its project and when its kind declares type-defining condition keys, its element type (a group subject per kind and the values of those keys), which intake proposes from its kind and conditions; its **siblings** share one. Records about a group are recalled for its members one step down. A child with no owner of its own routes to its container's owners, one step up. An instance's fact that differs from its group's at the same key is an **instance difference**, not a `counter`.
- A subject can rest on another by `basis` (`uses`) with `since` and `until`. A declared state key other than `within` whose value is a subject writes a derived `uses` link to that subject, with `since`; a newer value of that key sets `until` on the old link (state over time).
- **Aliases and successors** (I9). `same` (`alias`) also covers a rename; `replaces` also joins a split subject to each part. A `replaces` between subjects is a `rename` (the same unit under a new reference; its warnings carry to the successor). Records `about` a split subject, or with it as a condition value, match each part. A merge carries links over only once its alias is in full effect. References are normalized. A rename in a linked repository, or a source passage declaring an alias or successor, writes a derived `same` (`alias`) or `replaces` resting on it. A reference can be reused: a subject's reference carries optional `since` and `until`. When intake finds a reference whose stated kind or type-defining conditions differ from its subject's, or that a member or source states now names another unit, the Same? question (C13) proposes a new subject for the reference; the old subject keeps its records, and instances whose version state predates the reuse keep resting on it by `uses`. A pending alias proposal above the *alias similarity* counts as touching both subjects for the floor and the brief's warnings, shown as `alias-pending` [toward safety].

**Conditions and fields** (I10).

- A condition's key is a subject kind and its value a subject. The judge groups same-meaning values by derived `same` links, which member links override; conditions compare by group. A renamed value compares as its successor's group; a split value by the split rule. A redefinition is a `breaking` mark on the condition kind (C7).
- **Declared keys.** A forester can declare a state key `sampled` (its values are estimates, C3 In effect), `assessed` (its values are classifications, drafted as ratings, C3), or `presence` with a detection threshold (detections are point states and non-detections are estimates, C3 In effect), or `parameter` (a quantity not expected to change: its estimates form no time series, C3 Ratings and estimates) [team decision], applied forward (C6 Settings apply forward). An undeclared key whose subject kind has record age `never` is a `parameter` key.
- **`ordered` kinds** (I6): `numeric` (unit and fixed bins) or `ordered` (a declared order such as semantic versions). A value can be a union of intervals, checked by containment, never regrouped.
- **Fields** are subjects of kind `field`, grouped like condition values and by `within` containment at any depth. A scope setting `fields` [team decision] names a scope's fields. A **session's fields** are those of its scope and the fields of records `about` the subjects it touched. A query's current fields are those of its *current-field neighbours*; with too few neighbours with fields, the session's fields; else no field filter. **Another field** means no shared field group.

**Situations and areas** (I10).

- **Abstract restatement.** Each generation, the judge restates situations, attempt actions, source sections, and recent queries field-free: what is wanted, what conflicts and constrains, and what kind of move was made.
- **Situation** (I1): cases that want the same outcome against the same conflict, where a competent practitioner would take the same first moves. Derived per generation and partition; stable id; at least the *situation minimum size* cases; spread under the *situation spread cap*. Smaller groups map to their nearest case's situation within its spread; else they have no situation (`outside-situations`) until a generation forms one within spread. Such a case has no recurrence interval, takes the open-case cutoff as its outcome horizon, and, for C11 Compare, shares a situation with cases within the *situation spread cap* of it.
- A query maps to the situation of most of its *query-situation neighbours*. Sources and heuristics link `about` the situations their restatement matches above the *known-situation similarity*.
- **Outcome horizon**: the *outcome horizon percentile* of attempt-to-outcome time; below the *outcome horizon minimum* closed attempts, the situation's median declared `outcome_due`, else the open-case cutoff. **Recurrence interval**: the median gap between bursts of its cases. An undefined recurrence interval (fewer than two bursts) drops out of every maximum that uses it. A scope's **activity window**: the largest of the *base activity window* and its situations' horizons and recurrence intervals. **Active in a scope**: wrote there within that window, or has an open case there.
- **Interest areas** (D32): sets of situations with a stable id, derived per partition at each *monthly check* of the default scope for the team partition, and of the overlay's scope for an overlay. A situation qualifies when cases or queries from the *area member threshold* members (all active members of the cases' scope, if fewer) map to it within its scope's activity window, or when it has a procedure, a critical failure, an owned subject, or a holder set. Qualifying situations share an area when their fields overlap above the *area link overlap*, ignoring fields on more than the *broad field share* of in-area cases. A situation leaves after the *area exit checks*.
- A member's area change, or their query that maps to an out-of-area situation (then counted in area), applies at once to their own sessions [own view]; for others it is [team decision], with its notice to members whose sessions map to the affected situations.
- **In area**: a record `about` an in-area situation, or whose fields share a field group with an area's fields above the *area link overlap*; for a session, also a subject it touched. A fact or source is out of area only when everything resting on it is. With no area, everything is in area. Out-of-area records show `outside-areas` and rank one tier lower, or, when the *out-of-area handling* is `hold`, are `held: area`. An applicable serious warning is never out of area (I6 before I10).

## C5. Links and evidence

**Basis:** M3; I2, I3, I4.

**Links.** A link is identified by from, type, degree, and to. It holds until a `link` event removes it.

| Type | Endpoints | Effect |
|---|---|---|
| `about` | Record → subject, situation, record | Recall (C12); a record `about` a case shows with it |
| `basis` | Record, passage, hypothesis, subject → record, passage, attempt, subject, event (a member's statement, a tool output) | Status flows only along it (C7); independence (below) |
| `same` | Record, passage, or subject pairs | Evidence pools by degree |
| `counter` | Record, passage, or event pairs (corrections, assessments, attribute values) | Both shown with the differing conditions; `claim` makes differing (C6) |
| `replaces` | Record → record; subject → subject | The older leaves automatic recall for work whose reference time is after the replacement took effect, and keeps its serious and critical warnings in the floor |

| Type | Degrees |
|---|---|
| `basis` | `cites`: relied on that passage, statement, or record. `uses`: depends on that fact, or on that subject's state. `drafted`: a model drafted it from that passage. `copy`: a copy of that source version, passage, or fact. `produced`: this passage was written or changed by that attempt. `member`: a heuristic on a founding case or use. `generalizes`: a general heuristic on its field heuristics. `supports`: evidence beyond uses. `part`: a larger problem on a sub-problem; a summary on the cases it recounts. `caused-by`: a later case caused by an earlier attempt, a passage, or a subject. `possible`: may rest on it; shares an instance, situation, and time window, citing nothing |
| `same` | `identical`: no passage differs in a figure, condition, step, or negation. `one event`: two captures of one event, or a capture continuing an open case. `repeat`: same situation, attempt, and outcome at another time or instance. `equivalent`: one work in another form (translation, publication stage, patent family member). `passage`: a later version's passage continues an earlier one. `alias`: two names for one subject |
| `about` | `set aside`: an option or document the case did not take |
| `replaces` | Between subjects: `rename` (C4) |
| `counter` | `outcome`: two cases or uses that both happened. `recurrence`: an `outcome` counter for a problem back after an achieved outcome. `claim`: statements, positions, or decisions |

- **`replaces` credit**: `kept` (valid in its time: dependents inherit a cause; a dependent whose own valid range is open is reached as C7 Reached says for state facts); `re-evaluate` (valid then, not enough now: closed work is reached too); `wrong` (an error: a correction with a replacement, C6). Between two facts it is derived: `kept` for a later observation of a changing state or an unchanged value; `wrong` when the newer version says the older was in error; else `re-evaluate`.
- **Copies.** Passages of a `copy` that differ from the parent are stale unless marked **local differences** (deliberate variants with a reason). A mark clears `changed` at once only when nobody else rests on the copy; else [team change]. A local difference with reason `parent wrong` also corrects the parent passage `wrong` or `outdated` (C6) [team change], which reaches the parent and every copy by propagation, one notice per owner.
- `identical` and `one event` count as one. `repeat` adds a track-record item. Heuristics of one `kind`, `statement`, and `when` meaning get a derived `same` (`identical`, or `repeat` when `why` differs only in wording) that adds no item: their speakers pool as stated-by, never as support; the earliest is shown. `equivalent` pools source use and reliability and carries passages; status stays per source.
- Status follows every `basis` degree except `possible`, which carries only `may rest on`, and the evidence degrees `member`, `generalizes`, and `supports`, which carry none: their evidence reaches the heuristic through track records (C9) and trends (C8). A release's link stops at its scope (I5 before I4).
- `set aside` is neither `basis` nor a use. A reason resting on a record is a `basis` (`cites`) link from the case.
- **Who writes links.** Members and agents write links by command and in captures; session review drafts them; these are events. System models propose links at intake, in the *duplicate sweep*, and in builds: derived, in effect at once, except alias and successor proposals that no passage declares, and copy proposals, which wait for a member's answer. An answer writes the link as an event. Removing another member's link, and any link, mark, or answer that pools, lowers, raises, or clears a record another member wrote or rests on, is [team change].
- A link corrected `wrong` stops being a basis or use; propagation runs from its from-end.
- A classification is an `about` link; containment is only between subjects. A classification subject's revision maps old to successor by `replaces`, which recall follows; others' links stay.
- A link is visible only to a reader who sees both ends (I5). An attached event (assessment, correction, repeat) keeps its own scope. Records of different scopes are never merged.

**Evidence classes** (I3).

| Class | What counts |
|---|---|
| Direct | An observation of the subject itself (its file at a version, its log, a measurement with its instrument recorded) with a `supports` citation check, or its re-check. It outweighs any number of member-only observations |
| First-hand | A speaker's own observation of the instance (C3), not a classification of what was observed on a key declared `assessed` (C3 Classifications). It outweighs any cited statement, whatever its source kind |
| Cited | A statement whose citation check `supports` it |
| Self-reported, relayed | A statement with no check; each counts once |
| Claim, interpretive | A belief without a checkable source; an observation resting only on an interpretive reading. Evidence only for a dispute |

- **Class before kind.** Evidence class decides first. Source kind (C3 kind order) orders items only within one class.
- **Evidence for full effect** (M4): direct, first-hand, a re-check against a re-checkable citation, or a `supports` citation.
- **Citation check** (I2). At intake, for a fact cited to a passage, file, log, or tool output the server can read, the judge records whether the span `supports`, `partly` supports, or `does not support` it, is `interpretive` (relevant, but the statement goes beyond its words), or is `not checkable`. `does not support` holds the fact (C11); `partly` sets `citation partial`.
- **Re-check** (I3, I9). A re-check against a re-checkable citation (a file range at a version, a log, a tool output, a measurement, a passage with a `supports` check) is independent whatever the checker was given, with `re-checked` (checker and time). Against an `interpretive` citation it is one assessment by its checker, not direct, and never resolves a differing pair, however many agree (C6 Resolution). A re-check with no citation records `reviewed` (member and time). Only a re-check whose citation is off the record's own `basis` path and observed after the record clears `aged` or `changed`, or a first-hand observation of an unchanged value (a state-over-time observation); a re-check against the record's own passage records `reviewed` only. Sources whose content is "what that source says" are unchanged by this. Neither clears an inherited cause.

**Counting** (I3).

- Rules that count people count distinct original speakers, never recorders. An attempt counts for its maker or outside performer; its recorder counts once per **work unit** (one case with its `part` sub-cases; in a session, the span that work end or the closing of an open case ends), for concentration. A fact counts for its speaker; a set-aside entry for whoever set it aside.
- A role (`client`, `reviewer`) counts once per case or source; distinct pseudonyms count apart. When role, scope, and context identify one person, or a `same source` mark is in full effect, it counts once across all. A pending mark does not lower a trend's assessor count.
- A source-drafted record counts for the source's `origin` (for member writing: the passage speaker set; in a record stream: the author an entry states, or a declared time source's author field, mapped to a member handle when it names one, and also its `speaker`), and for its `publisher` only in kinds where the venue is the speaker (`maker`, `vendor`, `opinion`); once per work unit it records; never for its adder. As a concentration or sufficiency member it counts for the case's maker or the source's stated author; a data system is an item condition.
- **Marks**: `same source` (two speakers or sources are one original); `separately sourced` (two that share an `origin` value, publisher, or speaker are independent in fact); `coordinated` (a group that counts as one member; one principal's services count as one). Any member can set a mark with a reason [team change].

**Independence.** An observation, source, or member is **independent** of a record only when all three hold:

- (a) Their `basis` paths do not meet: following `basis` from the passages each rests on, and `same` (`identical`, `one event`, `equivalent`), at any depth, only the record itself is reached from both. Not followed: `possible`, and the links that make it evidence for the record (`member`, `supports`, `generalizes`, `repeat`). A copy is never independent of its parent, nor a memo of the fact it was written from.
- (b) For a member's observation, or a document a member or agent wrote: its session was given neither the record nor anything resting on it; or it contradicts the record; or it is a **separate occurrence** (a use's outcome, a warning on a member's own attempt, or a `first-hand` observation of the instance at a new observed time), which counts, for trends too, prompted or not (the item records `prompted`). A `first-hand` observation of the instance is also independent of a fact from a `compiled` source.
- (c) They share no `origin` value (for member writing, no passage speaker), no counting publisher, and no speaker who is one person by the counting rules, unless a mark says separately sourced. Records drafted from `primary` sources that share only a data-source or instrument `origin` value are independent when they observe different instances or observed times. Records from any source kind that report separate occurrences (different instances or observed times, (b)) and share only a funder or sponsor `origin` value are independent; Concentration (C9) still lowers certainty. An item whose speaker or `origin` is an outside organization that makes, operates, or supplies the warned subject (a `maker` source, or that organization as speaker) is never independent of a warning or record about that subject. Members and the team's own services are never that organization.

An item that fails is an **echo**: it counts for use, never as evidence.

## C6. Changes

**Basis:** M4; I9, Precedence.

**Classes.** Each operation takes effect by its class.

| Class | Operations | Effect |
|---|---|---|
| Own view | Collapse, acknowledge, own filters, own interest areas | At once |
| Toward safety | Show a warning; pause a holdout; hide for a suspected leak; move text to the author's private scope; quarantine; a tunable change toward safety; revoke a device or credential | At once, reversible |
| Own withdrawal | An author withdraws, retires, restricts, or narrows their own record that nobody else used or rests on; a member reverts their own session's writes that nobody else used or rests on (the others become [team change] retirements in the same batch) | At once; the restore command reverses it |
| State over time | A later qualified observation of a changing state (C3) | At once; the older one `replaced`, credit `kept` |
| Team change | Every other change to what others see: a correction, retirement, restore, or revision of another's record; a link, mark, or answer that changes another's record (C5); a material passage change by a non-earlier writer (C3); an author calling their own failed case `wrong`, or retiring their own record that others used; a `breaking` mark, or removing another's; a gate release of an injection hold (C11); lowering the failure value of an attempt in another's record | By evidence (below) |
| Access decision | A widening, membership change (naming a forester or reader included), sensitive-class declaration, or turning on a safety or visibility extension (Part II), by those who hold the access (I5; below, C1, C10) | At once, after a safety rescan |
| Team decision | A change that no evidence can support, because it is a decision about access-role naming, scope structure, or settings: access-role naming for a role with no holder (C14); a child-scope link without the enclosing forester (C10); a tunable change that alters what others are shown or asked, or how standing is derived, in either direction, except a change toward safety (A1); turning on or off a field extension that is not a safety or visibility extension (Part II Turning off), or turning X1 off or back on | By notice (below) |

**Team change by evidence.**

1. **With evidence** for full effect (C5) against the target, attached or later, of at least the highest class in the target's basis: full effect at once, reversibly; of a lower class, the two are differing (step 3). A failed re-run of its re-check disputes it.
2. **Without evidence**: `correction-pending`, shown beside the original, which keeps its rank, at the *pending-correction weight*, until **independent support** (such evidence, or a supporting use or repeat) or the *undisputed period* passes with no dispute that has evidence. The period counts only days on which the target's author or the routed member is available: they had server contact that day, their queue held no writes from that day when it next synced, and their schedule did not mark them offline (C13); up to the *undisputed period cap*. It never overrides a target whose basis holds a `first-hand` or direct observation or a cited re-check; such a pending correction with no evidence of its own lapses after the *objection lapse* as a position does (Lapse). One support question asks one routed member (C13). In a team of one, the period passes unopposed.
3. **Disputes.** With evidence, the change and its target are differing (below); a disputed `wrong` leaves a case's uses at full weight. Without evidence, the dispute lapses after the undisputed period and the change proceeds.
**Team decision by notice.** One notice goes to the people it affects (all active members for access-role naming; for a tunable, the members whose sessions it affects, and Settings; for a module switch, the members whose sessions the module affects where it is switched). The decision takes effect when the undisputed period has passed for every person the notice goes to, each counted as step 2 counts them, and at the latest at the *undisputed period cap*. A team decision lapses when a dispute by an affected active member is supported by a second independent (as in C14) affected active member within the period, until the decision takes effect; no evidence is needed, because none can exist for a decision. Evidence does not shorten the period.

**Settings apply forward** (Precedence). A setting change (a tunable, a key declaration) that alters how items are derived applies only to items observed after it takes effect. Except for a change toward safety (below), every build applies to each item the setting in force at the item's observed time, and an item derived from several items (a trend, a current value, a support weight) counts each of them under the setting in force at that item's observed time: a trend's test uses, for each occurrence, the ratio or count in force when that occurrence was observed, and the test runs whenever its inputs change; a trend so raised clears only as C8 A trend clears says. A setting that alters only what is shown or asked (display bounds, budgets, question limits) applies to every display and question from the time it takes effect. A change toward safety applies at once and may raise warnings or trends from existing items; it never lowers one.

**Corrections.**

- The correct, retire, and restore commands write corrections: a target record or link; a kind `wrong`, `outdated`, `narrower`, or `unsupported`; a reason; optionally a replacement and evidence. Anyone corrects what they can see, corrections included. The marker takes the target's scope; reason and evidence keep theirs. Evidence from a narrower scope gives full effect only inside it; elsewhere it counts as no evidence unless a release record carries it (I5).

| Target | `wrong` | `outdated` | `narrower` | `unsupported` |
|---|---|---|---|---|
| Fact | Ranks lower; with a replacement, leaves automatic recall | `aged` | The reason becomes a condition | Weighs as a record with no basis, `no-basis` |
| Case | Uses and episode leave records at the next check | Counts only for earlier records | The reason becomes a condition | — |
| Heuristic | Pending beside the original without evidence; differing with evidence, or once disputed with evidence; retirement only by C9 | Record restarts | The reason becomes `not_when` | — |
| Source | As for a heuristic; with evidence, also `corrected wrong` on the named passages at once: records drafted from them inherit it and go to the sensitivity stratum (C9). Source-level `withdrawn` needs a cited notice | `aged` | A fact about the source | — |
| Link | No longer a basis or use | — | The reason becomes a condition | — |

- `wrong` on a case means the record is inaccurate; a bad result is an outcome.
- A revision is `material` when the judge finds a changed value, scope, claim, or negation, whatever the author's label; else `editorial`, which does not propagate. A material revision of a record others rest on propagates as `corrected revised`, with `revised` (time and changed attributes), beside the earlier version [team change].
- **Same partners** (I3, I5). A correction with evidence in full effect on a fact pools to its `same` partners as the same evidence: each gets `corrected-via-same` (kind and partner id); a partner in another scope only where the evidence is visible, else `via-hidden`.
- **Batch reverts** (forester; one batch `correction` event). A revert undoes a damaged import: what one service wrote in a window, or everything drafted from one source. Only its member reverts a member's session (classes as in the table); a forester's revert of a member's session is [team change]. A revert retires the records, removes the batch's links and subject changes, and ends its subjects with `until`; others' writes stay. Its outcomes count as missing in both bounds (C9), never dropped. Its warnings keep standing and floor display, with `from-reverted-batch` (the batch id), until the standing rule lowers them. It is less shown, at once; a revert can be undone; nothing is removed.
- A revert of one system model version rebuilds the derived data that version produced; no member event changes (I1).

**Differing.** An unresolved `counter` (`claim`) makes both ends **differing**: heuristics or adopted decisions that rule each other out; procedures that differ at a step (`differs-at-step`: the other id and the step); facts about one time that cannot both hold, sources included; and, as `counter` event pairs, a change and its target disputed with evidence, concurrent changes of one attribute (C2), and disagreeing deciding assessments (C9). Opposite `replaces` links between one pair (answers to one Conflicting decisions question included) are a `counter` (`claim`) pair. An `outcome` counter never makes differing.

- A local difference is a position, shown at the situation moment as `local-variant` (the parent's id, the step, and the reason).
- **Neither is valid** (I6, I9). A computation that needs the value (a condition, a decision status) treats it as unknown; a list keeps both; a warning on either side stays in the floor, marked `differs`.
- **Resolution** only by: one writer withdrawing their side (a later change by one of the two writers from a base that includes both sides is that writer's withdrawal plus a new change); new independent evidence on one side (a direct observation, a first-hand observation against a side with only cited or lower evidence, a re-checkable citation, a sufficient record whose interval excludes the other side (for two heuristics: one side's ratio against the other's items in shared strata, C9, entirely on one side of 1), a confirmation with evidence); or conditions under which both hold, which makes each side an item under its own conditions. A direct observation on one side and none on the other corrects the other side `wrong`.
- **Order** of sides: by each side's own support (sufficient record, certainty, independent evidence, then the observed time of the value's origin along `copy`), never by author, role, or repeat count (I3).
- **Where shown.** At the stuck moment, up to the *differing shown* positions with speaker and `why`, then `also-differing` (the count of the others and a pointer); elsewhere, the marker.
- **Conflicting decisions** both show as in force and as differing, and appear in each other's check moments.
- **Lapse.** A position with no use, evidence, or case of its own lapses after the *objection lapse*, only when the opposite side has a use, evidence, or case of its own (else the pair stays differing); a source passage alone, a claim, or an interpretive item is not evidence of its own. Lapsed ones show as one line, `earlier-objections` (their count), stop opposing, and return on new evidence.

## C7. Status and propagation

**Basis:** M5; I4, I6.

A record's **status** is `current` or a set of **causes**: kind, reference, time, severity. A warning also carries the warned item's non-`auto` conditions and linked condition groups. A cause set on the record is **direct**; one that reached it along `basis` is **inherited**, shows its path, and at most ranks lower: only direct causes take a record out of automatic recall, and inherited serious and critical warnings stay floor items. A cause on a passage applies to that passage only.

| Cause | Set by | Marker: what it shows | Effect on recall | Clears by |
|---|---|---|---|---|
| `changed` | Own causes (below) | `changed`: what changed, and since when | Ranks lower | A new version or rewrite that materially changes the passage carrying the cause; a re-check (C5 Re-check); for a heuristic, a use or `repeat` item with an outcome under the changed state; on a `copy`, only a passage that matches the parent's or is marked a local difference |
| `record changed` | A heuristic's support weight changes sign, or leaves +1, at a monthly check (C9); derived (I1) | `record-changed`: the old and new support weight, and the check | None beyond the support weight; a trigger | The next monthly check that confirms the new weight |
| `aged` | Own causes; a contingency's expiry age (C3 Contingency) | `aged`: not re-checked since a time | Ranks lower only below a record on its key and instance with a later observed time | Cited re-check, observation, or retry; for a contingency, also a Keep it answer (C13 Still true) |
| `unreachable` | Own causes | `unreachable`: since a time | None at first | Reachable again |
| `before <change>` | All it rests on predates a `breaking` mark | `before-change`; later `re-confirmed`: the count of items since the change | Support weight 0 | Items since the mark sufficient alone (C9) |
| `basis not visible` | A record it rests on was hidden, restricted, or moved (the I5 exception) | `basis-not-visible`: since a time | Ranks lower | That record visible again, or a cited re-check |
| `withdrawn`, `corrected <kind>` | A correction (C6, C11); `pending` before full effect | `withdrawn`: time and reason; `corrected`: kind, member, reason; `correction-pending`: the pending correction's id | Full effect: ranks lower; `wrong` with a replacement leaves automatic recall. Pending: shown beside the target | A restore, or a correction of the correction |
| `replaced` | `replaces` with credit `kept` or `re-evaluate` | `replaced`: the replacement's id, and since when | As `replaces` (C3 Links) | Link removed |
| `retired` | Retirement in full effect; `retired: evidence` (C9); a retired `same` group (C2) | `retired`: the reason | Explicit search only | Restore in full effect |
| `source unreliable`, `citation partial` | Adverse share sufficient (C9); citation check `partly` | `source-unreliable`: corrected facts of the source, of how many; `citation-partial` | Ranks lower | Share falls; a re-check that `supports` |
| `warning` | C8 | `warning`: kind and severity; `warning-trend`; `critical-failure`: its time | Applicable serious and critical ones are the floor; critical ones and trends bar the least-shown slot, and the champion role as X1 Entry and Exit say | Only the standing rule (C8) |
| `differing` | Unresolved `counter` (`claim`) | `differs`: the other record's id | Both shown; neither valid | Resolution (C6) |
| `may rest on` | `basis` (`possible`) proposal | `may-rest-on`: the cause and the record's id | None | An answer that it does not: removal at once on the answerer's own records, else [team change]; a real `basis` on another's record [team change] |
| `held` | Gate `shape`, `citation`, `area`, or `injection` (C11) | `held`: the reason | Explicit search only | Gate release |

- **Severity order** for display: critical and serious warnings, `corrected`, `withdrawn`, `differing`, `before <change>`, `basis not visible`, `changed`, `record changed`, `source unreliable`, `citation partial`, `aged`, `unreachable`, other warnings, `may rest on`.

**Own causes.**

| When | Cause |
|---|---|
| A version `signal` from a linked location (`commit` for a repository; `revision` for a document) changes cited lines or a symbol (a rename moves the citation) | `changed` on records citing them |
| A source's new latest version changes a passage | `changed` on records resting on it (for a cited source, only citing passages and what rests on them), except facts replaced by a fact from the new version. Material: `changed: material <passage>`; formatting and renumbering stay plain. On a `copy`, a changed local difference: `changed: parent changed <passage>, local variant kept` |
| A rollout `signal` (`deploy`, or a kind's own) names changed parts, settings, or versions | `changed` on records citing them. With a successful `result` that names an instance and a version, also that instance's state fact on its `version` key (C4 Registry; declared, first-hand by the recorder, C3 In effect), which writes the C4 derived `uses` link; a failed or partial `result` writes no state for the parts it did not reach. Naming nothing, it is only a condition of later captures |
| A linked repository moves outside a record's version range | No cause; `may-not-apply` in those sessions (C12) |
| A subject kind's own `signal` | As configured; one that names parts reaches only records citing them |
| A `signal` on a subject | `changed: subject <id>`, inherited by records resting on it observed before the signal. On a group, only records resting on the group; a recorder that changes members names them as parts |
| An accepted generation regroups the record (C2); its translator falls below the canary band (A4) | `changed: regrouped`; `changed: translation <version>` |
| Its URL or watched file is gone | `unreachable`; after the *unreachable to aged*, also `aged` |
| A `breaking` mark (reason, optional affected keys) on a subject version, an `ordered` value, or a condition kind. With evidence (a changelog or release-note passage, a `signal` other than a version signal, a re-check, a judge proposal from a passage) at once; else [team change], with `breaking-pending` (the pending mark's id). | `before <change>` on records resting by `basis` (`uses`) on the marked version or kind whose items, observations, and founding cases all predate it. With affected keys, only records whose conditions or `not_when` name one (a member's keys win) |
| No observation or re-check within the subject kind's *record age*; `review by` passes; a fetch is denied; a dead end not retried, or an untried heuristic not used, within the *dead-end age*; a champion with no randomized evidence within the *champion evidence age*; a procedure that none of a situation's closed cases since its version cites, when those cases come from the *method-case members* within the *dead-end age*; never for a fact whose speaker is a non-`member` source and whose content is what that source says, except a drafted fact that rests by `uses` on an instance's state | `aged` |
| A trend (C8) | `warning` (trend) on its target |

- `signal` events let a rebuild reproduce these causes (I1).

**Propagation.** One rule carries every change.

- **Trigger**: a change of a record's direct causes, a `replaces` link, a removal, a decision leaving `adopted` or passing its stated end of validity, a key's state derived as `ended`, or a fact with no key and instance passing its stated end of validity with no successor (C3 End of validity), or a new fact about an instance's state that holds a stated hazard, a serious or critical warning, or a `first-hand` or direct observation that replaces an earlier state with a new categorical value or a new `ordered` bin (none on a key with a declared order or unit but no fixed bins, where only Crossing a bound applies; on a kind with neither, any different value); or a new `serious` or `critical` stated hazard or contingency with stated conditions. One source version, or one **write batch** (one session or one service within the *write-batch window*), is one trigger. On a key whose values are estimates or ratings, a state fact and its replacement of an earlier state, here and below, mean a change of the key's current value as C3 In effect defines it; on a `parameter` key, a change of its pooled value (C3 Ratings and estimates).
- **Reached**: the changed record and every record or subject resting on it, along `basis` backwards at any depth, skipping links whose `until` precedes the start of the trigger's range; for a fact about an instance's state, the records resting on that instance whose observed time, or whose `uses` link's `since`–`until`, overlaps the fact's valid range. A subject reached along `basis` (`uses`) carries the inherited cause as a record does and passes it on along `uses` at any depth, within the trigger's valid range and each link's `since`–`until`; the reach continues in the same way from the instances that a stated hazard matched. A cause may carry a `from` time: records and closed cases observed before it get only `before-cause` and no question. For a state fact that replaces an earlier state with a new categorical value or a new `ordered` bin, each record that rests by `uses` on the replaced fact, or on the instance at that key (by `uses` with conditions or `not_when` naming the key, or by a `uses` link derived from that key, C4), and whose own valid range is open (a current-state fact, an estimate, a decision in force) gets a direct `changed: input <key>` cause, and its owners get the re-check question as for credit `re-evaluate` (Closed work). `before-cause`, and the `kept` credit's exemption from questions, apply only to dependents whose valid range ended before the trigger. The first value after a `breaking` mark with evidence on the key's condition kind or instrument is no such trigger. For a stated hazard or contingency with stated conditions, the reach also holds every registered instance whose current state facts, or its `within` group's one step up, match those conditions by the applicability lookup (C12); each owner gets one `subject` line per trigger listing the matched instances; unknown keys, on instances whose kind declares the key or that hold a fact on it, join one Anomaly or difference question (C13). `produced` runs from case to passage, never back. Containment is not a path. Each reached record gets `via` (path, cause, id). Propagation stops at a scope boundary (I5): when it would reach records outside a restricted scope, the scope's readers get the restricted-warning release question (C8), one per trigger, and decide whether to write a warning-only release. `via-hidden` marks only a path through a record the reader cannot see inside scopes they read.
- **Crossing a bound.** A state-over-time fact (C3) whose old and new values lie on different sides of a bound named in the conditions, `not_when`, or contingency trigger of a record that is `about` the instance, rests on it by `uses` (or rests by `uses` on the instance's `within` group), or is a stated hazard or contingency whose stated conditions the instance matches (Reached), is a trigger: the record gets `changed: condition crossed <key>`, and its owners get one `range-moved` line. When the new value enters a contingency trigger or a stated hazard's range of severity `serious` or `critical`, the new state fact is also a warning occurrence on the instance at that severity, counted for trends by C5 (b); its standing falls as a stated hazard's does (C8). The contingency's target owners, or for a stated hazard the instance's owners, get an urgent `contingency` notice naming the contingency or hazard and the fact, shown under Waiting on me. Any other value on the key that lies in a `serious` or `critical` contingency trigger or stated-hazard range also writes one warning occurrence at that severity, once, when the value is recorded or becomes a side of a current differing pair, and the urgent `contingency` notice [toward safety], marked with the value's evidence class (and `interval-unknown` where it applies). These values are: a shown pending later state (C3 In effect) in a bin other than the current value's, either side of a current differing pair, and an estimate of the latest time window, which lies in the range when its interval's bound on the hazard side does, or, with no interval, when its point value does. Such an occurrence counts for trends only with evidence for full effect (C5). The current value, support, and standing do not change. The occurrence's standing falls as a stated hazard's does. A linked repository's version is such a state. A contingency trigger or stated hazard with stated conditions reads the value for its condition group (C3 In effect). A value observed before a `breaking` mark with evidence on the key's condition kind or instrument is not a side. The first value after such a mark sets the baseline; it is a warning occurrence only when it lies in the hazard or trigger range on its own terms.
- **Possible basis.** For a changed record with subjects, propose `basis` (`possible`), marked `may-rest-on`, to cases and facts that cite nothing on the path, share a subject instance with it or with a reached record, map to a reached case's situation, and were observed between its first visibility and the trigger (for a state fact with a valid range, inside that range). When a record's `ordered` range or `not_when` changes, also to cases, facts, and subjects whose value lies inside exactly one of the old and new ranges; owners get one `range-moved` line saying whether the item is now inside or outside.
- **Work product.** A committing action (C12) that sends, signs, files, or publishes a document proposes `until` on that **work product**'s `copy` link, so the record of what was done stays.
- **Closed work** (I8). A closed case is **asked** when its attempt rests directly on the changed record, the change's credit is not `kept` (a version that cites a changed state is `kept`), the case's instance has no `until` before the trigger, its goal names a lasting state (the Recurrence test, C11), and the cause is `corrected wrong` or `outdated` in full effect, `withdrawn`, a material revision or `changed: material` on its passage, `replaced` with `re-evaluate`, a critical warning or `record changed` on a heuristic it applied, a serious or critical direct warning on the record it rests on, or an instance-state fact whose range holds its observed time. A closed case with a `design` or `measure` is also asked when an instance-state trigger's range holds the observed time of an attempt that rests on that instance; its items go to the sensitivity stratum (C9) at once only when the triggering fact carries evidence for full effect; else the move is [team change]. Items leave the stratum only by a re-assessment or a re-check with evidence for full effect; an unevidenced own answer clears only the displayed cause. Each owner gets one re-check question per trigger, counted once in the budget, listing their asked cases by situation, with a bulk answer. Other closed cases keep only the inherited cause.
- **Derived first** (I1). Views, heuristics, and drafted records rebuild before anyone is asked.
- **Clearing.** An inherited cause clears when its source cause clears, or by its own row. A warning clears only by the standing rule. An unevidenced answer that the work still holds, or does not apply, clears the cause only on the answerer's own records; on others', only a cited re-check or the parent's clearing.

**One notice per person per trigger.**

- Running sessions that were given or are applying a reached record get the change in their agent context, as a delivery that does not count as a push to the person.
- One line in the person's **changes list**, and one `change` notice per person, within the budget, in their next requested session or as a `switch` line, to: owners and authors of reached records; members who applied a reached heuristic within the *delivered-to window* or were given one at a decision point.
- **Subject owners** follow what they own: a new case on an owned instance with a serious or critical warning, a trend, a new concern or contingency, a stated hazard, or a direct cause change of a fact about it gives a `subject` notice to owners who can read it (I5). For a person the trigger already reached, it joins their one line.
- A group's notice goes once per trigger, to members in working hours or with live sessions, else to the first to start.
- The triggering version's or batch's adder gets one summary of items routed to nobody; if the adder left or is a **worker** (a server process that writes for no member), routing step 4 applies (C13).

## C8. Warnings

**Basis:** M6; I6, I3, I5.

- **Sources of warnings**: an anomaly, a harm, a dissent (C9), commitment while a concern is open, a `wrong fact` failure, loosened criteria (C3), an interception (C3 `intercepted-by`; never a harm or a critical failure), a **stated hazard** (a fact of kind `hazard`, C3; it applies by the record's conditions), and a trend. Severity is `minor`, `serious`, or `critical`, as assessed or stated. A **critical failure** is a critical harm or anomaly. Dissent takes the *default warning severity*. A commitment-under-concern warning always shows, but enters the warning ratio, trends, and the champion bar only when the concern held evidence for full effect at commitment, or the case later records the harm or failure the concern named (I3).
- **Standing** (I6) falls only on: a correction in full effect with evidence about the cause, or that the event did not occur or not at its recorded severity (evidence observing that occurrence: its instance over its observed time; that occurrence only); a `mechanism: confirmed` case naming the cause, with evidence for full effect that the cause produced the warned outcome under the warning's conditions (a reproduction or a direct trace, not that the fix worked) from an assessor independent of the warned record, plus a re-check, by a speaker independent of any outside organization that makes, operates, or supplies the warned subject (C5 (c)), that observes the cause (not the outcome) removed under the warning's stated and linked conditions, the subject versions that the warned attempt rests on by `basis` (`uses`) included, for standing only (never for applicability, C12); a re-check lowers standing only for the subject versions it observed, and for condition values not observed the warning keeps its standing; for a stated hazard, a re-check that its condition does not hold, or a later source version stating it fixed, only for the instances, subject versions, and condition values it names (a named fixed version narrows the hazard's range); for a warning from a crossed bound (C7), also a later state fact whose value lies outside the trigger or range. A speaker's withdrawal or revision shows only `withdrawn-by-speaker` (with its time). Lowering a severity below `serious` follows this standing rule. For floor membership, a warning takes the least favourable severity among its assessors, checked or not, until evidence decides (I6 before I9; D14); an assessed harm or anomaly linked `about` a stated hazard (C9 Harm) is one of that hazard's assessors. A re-check about one instance never lowers standing elsewhere.
- **A lowering rests on its evidence.** A fall in standing rests by `basis` on the evidence that caused it. When propagation reaches that evidence (`changed`, corrected, removed), the warning returns to the floor at once [toward safety], marked with the change, and the fixed subject's owners get a re-check question. A later occurrence of the warned kind, on an instance that rests by `uses` on a subject version the re-check covered, under conditions that do not mismatch, disputes the mechanism case (C6 step 3) with the same effect, and `until-removal` marks drop. Acknowledgment never lowers standing (I6), and no release by a count of clean cases exists.
- A model-drafted severity shows `severity-drafted` (I6). A stated hazard keeps its stated severity.
- A warning `about` a subject that a `signal` ended keeps its standing but applies only to sessions touching that subject or its successor.
- **One origin** (display). Inherited copies of one cause, and warned cases linked `same` (`repeat`, `one event`) for one cause, show as one item, named with the path to the record at hand, applying only where the origin's conditions do not mismatch. A critical assessment with `failure_reason: inherent risk` folds by origin too; it keeps its standing and joins counts.
- **Display** (I6). In the floor, the display bound is the *floor lines* per moment. A **cause group** is the warnings of one kind about one subject instance; a warning about no instance groups by its record. Past that bound also, a predecessor's unseen `serious` warnings whose stated conditions do not match the session fold into one `inherited-from` line instead of cause-group lines; its `critical` and condition-matching ones stay individual. Seen warnings follow, one line per kind and severity, each named with its latest id and a pointer to the rest of that instance's floor, never a bare count. An **acknowledged** warning (a trial with a reason) also returns to that reader when its severity, cause, or evidence changes (I6). For a warning that reaches a check through a subject the acted-on instance rests on by `uses` (C12 Floor), another instance means another instance of the warning's own target or group; another dependent instance does not return it. An `auto` value outside the range seen across the warning's warned items (`ordered` kinds) counts as changed conditions for this return, with no applicability effect. A returning floor warning shows `still-applies` with the count of the reader's sessions it was seen in.

**Trends.**

| Target | Raises a trend when |
|---|---|
| A heuristic | Its warned items of one kind against comparable items (C9) exceed the *warning trend ratio* |
| An instance group | Its warned share against its siblings under the same `within` parent exceeds the ratio |
| An instance, a group's instances, a subject | Warnings on one instance, on distinct instances of one group (only where that group's warned share is above its siblings' or no siblings have items), or `caused-by` one subject, pass the *trend recurrence* |

- **Own rate.** Recurrence keeps the absolute count for `critical` warnings. A target whose occurrences before the *trend recurrence* window already exceed the *trend recurrence* raises a recurrence trend only when its recent rate exceeds the *warning trend ratio* against its own earlier rate. Its recent rate is its occurrences in that window; its earlier rate is its occurrences per *trend recurrence* window length over the period from the earliest observed time of any item on the target (for a cause, any item that cites it; for a group's instances, any item on a member instance) to the start of that window. A target with fewer earlier occurrences keeps the absolute count.
- For a heuristic, only occurrences from distinct independent assessors count (C5). For an instance, group, or subject, separate occurrences count (C5 (b)): distinct instances or observed times. Prompted ones count. For trends, `one event` counts once; each `repeat` is an occurrence. Independence is judged against the target.
- A trend is a `warning` on its target at the warnings' highest severity (a recurrence trend whose warnings name one confirmed cause or share one `caused-by` target in full effect (not a proposal or `possible` link), at least `serious`), shown as `warning-trend`. A trend raised on an instance target returns once to readers who acknowledged the warnings it counts (Display); later occurrences change only its count.
- A group's trend is recalled and shown in the floor for its member instances one step down; they carry no inherited cause.
- **A trend clears** only when the standing rule lowers each of its warnings, or a `mechanism: confirmed` case names a common cause whose removal is re-checked. A warning whose event did not occur leaves it; one whose cause was removed counts only before the removal, marked `until-removal`.

**Restricted warnings** (I5 before I6). When a serious or critical warning in a restricted scope applies at a check moment outside it, by its subject or a cross-scope match, the scope's readers get one urgent release question per warning and situation (C13). The question names the outside member and subject only to readers who can read the outside session's scope; to others it is content-free. Any reader who answers may write a warning-only release record (kind, severity, applicable conditions, the subject in the receiving scope's terms) under the release check, as [team change] carried by the warning's evidence, or add the outside member as a reader (an access decision, C10, with a confirmation).

## C9. Learning

**Basis:** M7; I3, I7.

**Assessments and outcomes.**

- Each non-`open` `outcome.recorded` is one assessment by its assessor (default: the author). A later one replaces only the same assessor's. **Deciding assessments**: the latest ones with `checked` evidence if any, else all latest ones.
- An assessment in a later horizon stratum (below) than the first deciding one is a later state: it names a horizon condition, or takes one derived from the time elapsed since the attempt. The outcome takes the latest horizon; the earlier value stays as an `after` condition (`horizon: <h>`).
- An assessor or role named in `done_when` counts as `checked` for its criterion, never for harm, anomaly, or severity. With several criteria, `achieved` needs all, `partial` some. A score takes the outcome its declared thresholds give.
- Agreeing deciding assessments give the outcome. A **horizon stratum** is a bin of the derived horizon, each bin one outcome horizon of the situation (C4) wide: the first ends at one horizon, the next at two, and so on. When deciding assessments in one horizon stratum differ, or one is disputed with evidence, the outcome is **differing**: failure value missing, left out of observational records, both bounds in the randomized analysis. An assessment worse than the outcome is a **dissent**, a warning with `assessors-disagree` (dissenting and total assessors).
- **Harm** (I4, I6). A harm noticed later is an assessment on the causing attempt, linked from the later case by `basis` (`caused-by`), which the judge proposes when a case names an earlier attempt's instance and time. A cause with no attempt (an omission in a standard, a supplier) is the passage or subject itself, proposed when a supported hypothesis or `failure_reason` names it (a question when unsure). It reopens any inferred outcome and warns the upstream decision and its heuristics. A harm or anomaly whose `caused-by` link in full effect ends at a subject or passage is also a warning on it at the assessed severity (C8). The judge also proposes any assessed harm or anomaly `about` a stated hazard whose event and conditions it matches (a Same? question when unsure). When the causing attempt applied or produced a copy, the judge also proposes `caused-by` on the parent passage; C5 Copies then reaches every copy.

| Outcome | Failure value |
|---|---|
| `achieved` | 0 |
| `partial`; a fix whose problem recurred with no other cause given; an inferred outcome | *partial failure value* |
| `failed`, `no-effect`; an achieving attempt with `mechanism: refuted`, for `check` and `explanation` | 1 |
| `achieved` with a serious or critical anomaly or harm | *partial failure value*; never a founding success |
| `inconclusive`, `open`, unbound | missing |

- A shared outcome on several cases is split equally among them, from the next monthly check.
- **Aggregate harm** (I3, I7). A harm that many uses cause together (resistance, retry storms, erosion of a shared resource) is a fact about a subject. When it has `caused-by` evidence that names a heuristic or procedure through a cited comparison, it is a shared outcome: a failure share is split among that heuristic's uses inside the harm's window, by the rule above. Until a recurrence question is answered, the earlier attempt keeps its failure value, and the change is missing in both bounds.

**Learned heuristics.**

- **Attempt groups** (I1, I7). At each *monthly check*, the builder groups attempts across cases by similarity of what was done and why; a new generation regroups the attempts of each past monthly check as of that check, and C2 Switch checks compare the result. A group becomes a learned `field` heuristic with one `achieved` or `partial` attempt, with failures from the *failed-attempt minimum* cases, or with one case whose failure has `failure_reason: approach` in an assessor's own assessment (not drafted) and a `stated` `why` (a dead end); from a single case it is **provisional**. Its `kind`: `explanation` for a confirmed hypothesis or mechanism, `check` for attempts that only observe, else `action` (dead ends and methods below).
- **Dead ends** (I3, I6). A `dead-end` heuristic comes from a group of only failures for reasons `approach` or `unknown`, or from set-asides of one option (never a document) or `not_when` entries with one reason from the *set-aside members*, only when the option has no `achieved` or `partial` attempt in the situation under the set-aside's conditions; otherwise a set-aside reason that names a condition is handled as a decline reason (Condition test). A successful retry adds a `not_when` entry and a positive heuristic. A dead end whose basis has a cause other than `aged` or `unreachable` is a **retry candidate** under "Untried ideas", with its basis's serious or critical warnings.
- **Other inputs** (never uses). Method cases from the *method-case members* in one situation form a learned `decompose` heuristic with ordered steps. `observation` or `claim` facts about one situation from the *fact speakers for a heuristic* form a provisional `explanation` or `check`, stated scopes first for `when`. Each of these inputs forms nothing below its own threshold.
- **Founding cases**, fixed at creation: the group's cases at that check; for a stated heuristic, the linked or grouped cases whose attempt came first. A later attempt is a use.
- **Text** (I1, I2). The builder writes `statement`, `when`, `why`, `not_when`, and `reading` from founding cases, without team figures, from `stated` reasons only. Learned text changes only at a monthly check, in a generation's regrouping of past checks, or by a correction. A rewrite by a new model version from the same founding cases keeps the id and the record. Only a regrouping that gives different founding cases with a materially changed text, a material member correction of the text, or an `outdated` correction starts a new heuristic or restarts the record.
- A group that matches a stated heuristic attaches its cases to it. One that differs in what was done or why is a separate learned heuristic, linked `counter` (`claim`) only when the two rule each other out.
- **General heuristics** (I10). When attempt groups from the *general-heuristic fields* match by abstract restatement, the builder writes a field-free `general` heuristic with `basis` (`generalizes`) to each field heuristic, and its origin.

**Uses.**

- A **use** of heuristic H is an attempt outside H's founding cases that does what H says (in H's group, or `from: suggestion <H>`); for a procedure, an attempt citing it at a version. An attempt that the judge finds departs from H (for a procedure, at a step) is not a use and rests on H by `basis` (`cites`); the judge result, with its model version, shows on the attempt and in H's record as `departs-from` (for a procedure, `departs-at-step` with the step). **Prompted**: a delivery of H bound to its case came first. A delivery not acted on is not a use.
- **Which outcome counts**: the attempt's for `action` and `dead-end` (inverted); a variant's by C3 Tests; the case's final outcome for `check`, `frame`, and `decompose`, or, about running a designed test, its `answer` (`valid` as `achieved`, `invalid` as `failed`); for `explanation`, the matching hypothesis or mechanism, else the final outcome.
- **Items**: each use, and each `repeat` of an item's case, with its own value, member, time, conditions, and warning. **Comparable items**: same situation, time stratum, and condition group, whose conditions H's `when` and stated conditions match by the applicability lookup (C12; unknown keys excluded, with `condition-known-on`); not H's own. A stratum with no comparable items adds nothing; when none remain in any stratum, the record is insufficient.

**Track records**, per situation at each *monthly check*.

| Part | Holds | Shown |
|---|---|---|
| Basis | Founding cases; `counter` cases and facts; `supports`; controlled comparisons; conditions of successes and failures; uses (member, instance), warned ones marked; share of closed cases that used H | Always, as links; counts only as sufficiency allows |
| Observational record | Relative failure risk (RFR) of H's items against comparable items, compared only within strata (scope, *time stratum*, `after` conditions, stuck binding, the attempt's position in its case (first, or after failed attempts), the bound case's declared `severity`) and pooled across strata; age-weighted by the *record half-life*; items of one member or performer, source, event, or case treated as correlated | When sufficient; at most `low` (`moderate` for a large consistent effect, below) |
| Controlled record | Paired comparisons of variant outcomes from controlled cases where one variant uses H; weighted by stated `estimate` precision where present, else equal per comparison; pooling across condition groups allows between-group variation; randomized cases alone when sufficient | When sufficient; `high` if randomized with `checked` outcomes, `moderate` if otherwise randomized, else `low` (`moderate` for a large consistent effect, below) |
| Effect | Randomized episodes (X1) | In decidable situations; kept after |

- The situation's record is used when sufficient, else the one across H's situations. A sufficient record for the session's conditions comes first and names those conditions.
- The *record half-life* grows with the *half-life factor* times the larger of the situation's recurrence interval and outcome horizon, never below its A1 bound. The *time stratum* grows with the recurrence interval, never below its A1 bound.
- **Warning ratio** (I3, I6): H's warned items against comparable items, overall and per kind, never age-weighted. Overall entirely above 1 bars positive weight and the champion role. One kind above the *warning trend ratio* raises a trend. Any warned item shows `warning-count` (warned and total uses). For a dead end, warned retries support it.
- **Critical-failure bound** (I6). With a serious failure or harm in its basis, H shows `critical-failure-bound`: the uses since with no critical failure, and the upper bound on the critical-failure rate that those uses still allow at the *critical-failure bound level*. It never lowers the warning.
- On a **restart** (Text; I3, I4), the earlier record shows until the new one suffices. A procedure's record is per version; a `copy`'s uses count toward its parent at unchanged steps. Subject versions are item conditions.
- Items that rest on a version before a `breaking` mark on H's subjects or condition kinds (by the version they rest on; by observed time only where no version is known) form a permanent separate stratum: the shown record uses only later items; earlier ones show as basis, with `before-change` and their figure, adding nothing to sufficiency or support weight.
- **Sensitivity stratum** (I3, I4). Every record drafted from passages under `withdrawn` or `corrected wrong`, direct or inherited (cases, facts, and their items), is kept out of support. Items that C7 Closed work moves there join it; they leave only as C7 Closed work says.
- **Risks** (a view), shown with H: harms on its uses, open trends, and `after`- and `recorder`-condition results.

**Condition test.** At each check, the builder tests each condition group, time stratum and declared severity included, on failures against successes and warned against clean items.

- A one-sided test valid at small counts that passes the *condition-test level* with the *condition-test minimum* adds a `not_when` entry with `found-by-builder` (matching and total items), shown with the counts. An `after` or `recorder` condition adds a risk instead, never a `not_when`.
- A decline reason that names a condition adds it as a key tested on outcomes, and to the watch list. `declined-under` shows only when that test passes; else it shows with no count, as a hypothesis.
- The time stratum adds a `from` entry with its start.
- A key `unknown` on over the *unknown-key share* of clean items is untested, with `condition-known-on` (clean items with the key known, and total), until facts about the instance at the attempt's time or one batched question fill it.
- The test also runs on the warned and clean items of an instance-group trend, and on `critical` harms in one situation within the trend window, including `uses` subjects and conditions taken at each harm's observed exposure time. A subject whose concentration in warned items passes the condition test at the *condition-test level* is proposed as `basis` (`possible`) with `caused-by` as the proposed degree, with one Same? question (C13 Independent questions), never to the subject's owners; nothing changes standing until it is answered or evidence arrives.
- An entry is removed only when the test's p-value is above the *condition-test removal level*. It never hides H. A member's removal of a builder entry is [team change]; later evidence can add it again.

**Sufficiency and certainty.**

- A record is **sufficient** when each side has an effective sample of the *sufficiency sample*, H's side comes from the *sufficiency members* (all members in a smaller team; not required for a controlled record of `randomized` cases with `checked` outcomes, or when H's items have `checked` outcomes written by an instrument or system on distinct instances; concentration still lowers certainty), and its interval at the *interval level*, valid under repeated monthly looks, has an upper-to-lower ratio below the *sufficiency interval ratio*. The **effective sample** is the item count discounted for unequal age weights and for correlation within correlated groups (A2 I3-g). For the effect: episodes with an outcome per arm.
- Where `random sample` assessments exist, rates and the outcome worker's silence test use them, weighted by inverse probability.
- An insufficient record shows `no-record-yet` and no figure, only its basis display: warning count, critical-failure bound, and, for a dead end, basis counts. The agent judges it by `why`, conditions, and basis.
- A sufficient record shows its figure and a **certainty** (`high`, `moderate`, `low`, `very low`) with reasons. Ceiling (I3): the effect, and a controlled record of `randomized` cases with `checked` outcomes, start at `high`; a controlled record of other randomized cases at `moderate`; others at `low` (`moderate` for a large consistent effect, below), shown with `observational`. Each factor lowers it by one:

| Factor | When |
|---|---|
| Risk of bias | Over the *bias share* of outcomes are self-reported or inferred, set after the outcome, timed by an outside source (except outcomes whose measure and analysis a declared time source timestamped before the outcome, I2), bound to `late` trials, or selected `triggered` |
| Imprecision | The interval crosses the decision threshold, 1 |
| Missing outcomes | Worst-case bounds cross 1; the estimate does not |
| Crossover | Randomized only: over the *crossover share* of an arm's episodes |
| Inconsistency | Differs across scopes, situations, or condition groups beyond chance at the *inconsistency level* |
| Indirectness | Most items come from condition groups unlike the session's |
| Concentration | One member, performer, service, non-primary source `origin`, or coordinated group supplies over the *concentration share* of the weights the record uses (its effective sample; the dominant contributor's share shows as a basis note) (a `primary` or `compiled` data source's records count for their performers or stated authors, C5) |

- Agreeing controlled comparisons raise it by one, up to the ceiling. For observational or `controlled` evidence, the ceiling rises to `moderate` when the interval's upper end lies below the *large-effect bound* and the effect is consistent across units and strata; the factors then apply; the champion role stays randomized-only (X1). Recall shows the effect, else the controlled, else the observational record.

**Sources and reliability** (I3, I4).

- A fact whose speaker is a source states what it says. A fact that records a value with an instrument (a procedure registered as an instrument, or a key declared `sampled`, included: C3 In effect) is drafted as an `estimate`, with the instrument's stated repeatability as its interval (C3 decides `counter`). Two other such facts from different sources about one time that differ are `counter`, with `differs-in` (the other source and its value); ratings never are.
- A fact copied from an earlier one (a later source with no observation of the subject) becomes its `copy` once an answer confirms the judge's `possible` link: an echo that takes the parent's changes, never a separate side.
- **Reliability**, per source, publisher, `origin` value, and instrument version: derived counts of its facts that were corrected `wrong` or `unsupported`; replaced with credit `wrong`; on the losing side of a resolved `counter` (`claim`; estimates differing by sampling never count); named in a `wrong fact` failure; or re-checked as holding. When the adverse share is sufficient (observational, at most `low`), the source gets `source unreliable`, which flows as any cause. The condition test by condition group and passage section can limit it to those conditions. Members get no team-visible figure.

**Exploration** (I7).

- **Candidates** for a stuck or ask query: the relevant heuristics of its situation in the session's partition, provisional, untried, released, and rival ones included; held records never. A query farther from its nearest situation than that situation's spread shows `outside-situations` and no champion; its candidates are the relevant heuristics by retrieval (C12) over `statement` and `when` across situations in the session's fields, ranked as usual, then "From other fields".
- **Support weight** (I3, I6): +1 when the shown record is sufficient, its interval below 1, and its warning ratio not above 1; −1 when its interval is above 1; else 0. Multiplied by the *very-low weight factor* at `very low`. A change of sign, or a fall from +1, at a monthly check sets `record changed` (C7). The **false-discovery control** admits +1 across the heuristics evaluated per partition at each monthly check, so that the expected share of false +1 weights stays within the *false-discovery level* under repeated monthly looks. Ties: controlled wins minus losses.
- **Least-shown slot.** Outside decidable situations, recall shows up to the *heuristics shown* candidates with a `more` pointer. When more are relevant, the last slot goes to the qualifying candidate (insufficient record, no `may-not-apply` match, no serious or critical warning, fewer than the *exploration exposures* deliveries through this slot within the *least-shown window* with no use; it keeps its normal rank) with the fewest deliveries in the situation within the *least-shown window*, a `same` group counting as one; ties go to one not `aged`, then to the one delivered or created longest ago. One author's stated heuristics take it at most once per window.
- **Rivals** (I1). Monthly, for the most-used heuristics and procedures (the *most-used share*), the builder derives a **rival** (generation data; a revert of the judge discards it) that explains the same cases, warnings included, differently, or a different move. For a dead end, the rival is a condition under which its move works. A rival is a candidate, with `rival-of`, eligible for the least-shown slot and the draw. A challenge question about each most-used record goes to an active member who neither wrote nor recently applied it; with none, it waits on the board, marked `no-independent-member` (C13).
- **Retirement by evidence** (I9). A heuristic with any use of failure value below 1, or any case not its author's, retires only with evidence: a sufficient effect or record entirely above 1 (for a dead end, from inverted retries); `counter` links that address its conditions with evidence for full effect (C5) of a class at least as high as the strongest in H's basis and uses (M3) (not for a dead end); or a trend with independent support. The builder derives `retired: evidence`, which a member can dispute.

**Randomized comparison** is module X1. Its outputs (decidable state, champion, effect, menu) are the only ones that core sections use.

## C10. Scopes and safety

**Basis:** M8; I5, I1.

**Scopes.**

- **Everything has exactly one scope**: the team-visible default scope, a team-visible project scope, a restricted scope, a member's private scope, an intersection scope, or a hide scope. Any member creates one (optionally restricted to named readers) and becomes its forester. Only its readers see a restricted scope's name.
- **Linked locations.** A forester links a repository or path prefix, folder id, or workspace URL; no two scopes link one prefix. A link inside another scope's location makes the new scope its child, by a forester of the enclosing scope, else [team decision]. Settings, time sources included, inherit from the parent unless set. A child of a restricted scope inherits its exclusions, and its readers are a subset of the parent's; a less restricted child is refused. A scope is **wider** than a session's scope when every reader of the session's scope reads it.
- **A session's scope**: the scope linked to the most specific location holding it; else the member's choice for that directory; else the default scope, or the private scope under `unresolved sessions: private`. A location linked to a scope the member cannot read resolves to the nearest enclosing one they read; they are told only that the location is linked elsewhere. Moving to another linked location resolves it again, but never wider than a narrowing made earlier in the session (I5). A member can narrow the session at any time [toward safety]; it applies to writes not yet logged. A linked location may name a branch or ref pattern inside a repository; the most specific match wins.
- **Session narrowing.** The agent says once that the session narrowed. After reads from two or more restricted scopes, the session narrows to their **intersection scope**: readers are the members who read all of them; tended by their foresters; created on first need; named only to its readers; reused for the same set.
- **Narrowest input** (I5) counts `basis` targets and subjects, whoever wrote the links, and restricted text a write matches in scopes the author reads or the session or device read. For any other write, the write keeps its scope and its author learns nothing; the restricted scope's readers get one notice, never naming the restricted match: it names the matching record only to readers who read the write's scope (they can hide it for `confidential`, Hide); to the others it is content-free. Inputs that are not nested give their intersection scope (Session narrowing). A span does not narrow when it is also in a source readable in the write's scope on which the restricted record rests (`copy`, `cites`, `drafted`), or that predates it in a wider scope. The author is told why and can drop the link; widening is a release.
- **Private scope.** Read only by its member; an overlay like any restriction. Each *private move interval*, its member is asked whether to move records about another scope's subjects.
- **Restrict, move, widen.** Restricting and moving never widen without the member's action. Narrowing another member's record for a reason other than safety is [team change]. An author widens their own record at once only when it neither rests on nor matches text of another author in that scope; otherwise the widening is a release under the release check, decided by the scope's forester. A scope's foresters may widen a selection of the scope's records (named, or by kind or subject), whoever wrote them, after a per-record safety rescan and a matching check against the records that stay restricted; records that rest on or match still-restricted records stay and are listed. A move re-keys per-scope keyed hashes and tokens (C3 `speaker`; Masking, names of natural persons), and `person` references (X2), for the destination scope.
- **After a widening**, child scopes keep their restriction; intersection readers are recomputed. Foresters see how many records keep exact quasi-identifiers and generalize, keep, or restrict them.
- **Dormant scopes.** A scope whose `project` subject has `phase: closed` and no open cases sends no forester-fallback questions until new activity.
- Where a reader or forester is needed and no holder answers, access-role naming (C14) is the one way in.
- Every read path applies scopes: recall, search, views, exports, tool calls, the board.

**Partitions and counts.**

- **Partitions.** Summable data (basis lists, records, warning ratios, certainty, least-shown counts, reliability) is kept per scope and summed at read time over the session's scope and wider ones (over all a reader's scopes only in a session narrowed to them). Non-summable data (situations, decidability, menus, effects, champions, interest areas, holder sets, drill candidates, model text) is computed per **partition**: the team partition from team-visible records, and an **overlay** per restricted or private scope. An overlay case joins a team situation only within that situation's spread (C4); other overlay cases form overlay situations by the C4 rule, visible only to the scope's readers. A reader of several restricted scopes sees the session's overlay, else the one they choose, marked `variant` (the scope). A restricted text variant is written only where a scope exceeds the *variant share* of a situation.
- **Summed restricted occurrences** (the I5 count exception). Separate occurrences of `serious` or `critical` warnings `about` one team-visible target, across restricted and private scopes (C5 (b)), are summed; when they pass the *trend recurrence* and the *aggregate minimum*, the target shows a content-free `restricted-trend` (kind, count, window, no ids), and each contributing scope's readers get the C8 release question at once.
- **Content-free counts** (the I5 exception). Shown outside only when contributing scopes and members each reach the *aggregate minimum*: `restricted-critical` (a critical failure exists in a restricted scope), with its month only when that month also reaches it; `restricted-trend` (a trend exists in a restricted scope) with its kind; `pooled-restricted` (the number of restricted scopes pooled) on a team-visible heuristic or procedure, when sufficient, counts and interval only, at most `low`. Nothing else crosses a scope. Operational counts about a restricted scope (hides, quarantine) show only to its foresters, and in team-wide figures only above the minimum.
- **Release records.** A forester who reads a restricted scope, or any reader who answers its Release question (C8, C13), writes, from one of its records, a **release record**: a new record of their own (`record.created` with `basis` to the original) for the team or one named scope, optionally sanitized. The `basis` is visible only to the original's readers. Release and original count once for independence and pooled counts. Propagation stops at the scope (I5 before I4): when the original changes, its readers are asked once whether to update the release or keep it. The **release check** passes a release only with no matching span and no identifying subject, quasi-identifier, pseudonym, figure, quote from a speaker in the scope, or other fact from it (a named scope's own subject excepted, in its pseudonym). A `sanitized` release of a source or passages names the passages it keeps and may hold matching spans and other facts when none of the rest remains after the *generalization rules*. Every release record from a restricted scope shows `from-restricted`.
- **Release offers at work end.** A session's new heuristics and method cases in a restricted scope go to its readers in one Release question (C13), with judge-drafted restatements that pass the release check. A narrowed session's facts and cases about team-visible subjects that pass the release check go to their author; accepting is their own widening (Restrict, move, widen).

**Masking.** Checks run on every free-text write: first on the client, before text leaves the machine, by the shipped classifiers or a `team-controlled` model (team infrastructure, keeping nothing); then in memory at intake, also for text with no client (tool protocol, refetch). They run again before text leaves for an outside service, on export, and on a rule change (a **rescan**). A **mask** replaces a span and shows the kind of what it hides.

| Found | Action | Check |
|---|---|---|
| Credentials: keys, tokens, private keys, passwords, connection strings | Masked by kind | Fixed |
| Personal data by enabled locale rule sets: national ids, card and account numbers, phone numbers, home addresses, email addresses and online identifiers (IP addresses, device and advertising ids, platform account ids) other than principals'; organization contact data where a forester allows | Masked by kind | Fixed |
| Text matching a deny rule | Masked as removed, or as the pseudonym; the writer is told the field and date | Fixed |
| Personal data in bulk (*bulk personal-data threshold*), or in a table column | Masked as bulk for the attribute or passage; a column in every row | Fixed |
| Names of natural persons who are not members (principals' handles, published authorship, and published case parties allowed; a non-member's handle is a name; a forester can exempt a public role) | A role plus a per-scope pseudonymous token that only that scope's keys can reproduce and that does not reveal the name; queries naming the person search by it | Judged |
| Special-category data about an identifiable person | Masked by kind; content about a role is kept, unless the role statement's quasi-identifiers single out one person in the scope: then the special-category content is masked by kind and a content-free statement of role, site, and date stays | Judged |
| Quasi-identifiers about `person` subjects, and a scope's identifying subjects and figures, where enabled (I5) | Generalized by the *generalization rules* wherever text leaves the scope | Judged |
| A **declared sensitive class**: a subject kind or a subject list, plus the quasi-identifier kinds that co-occur with it, set by a restricted scope's foresters who read it (an access decision, I5) | In a write to any scope whose readers are not all readers of the declaring scope, those quasi-identifiers are generalized by the declaring scope's *generalization rules*, and the exact entry moves at once to the declaring scope [toward safety], as text moves for a confidential outside person. The author is told only that a sensitive class was generalized. A rescan runs when the class changes. The class is matched only on the server (at intake and in rescans), never on clients | Fixed by pattern, else Judged |
| A member's health or private life; a confidential outside person (one the source expects to stay unnamed: referrer, complainant, witness, minor, patient) identified by role with quasi-identifiers | After the masks above, moves at once to the author's private scope [toward safety]; for a service author, to the private scope of its single-member principal, else to the intersection scope of its group's members (C1 Shared audiences); each such routing counts under the *private routing threshold*. The author keeps, restricts, or removes it. Another member's private life goes to the intersection of both members' private scopes, or is masked | Judged |
| Evaluative statements about a member as a person | The span is masked as evaluative (the I1 exception) and replaced by a neutral role statement, marked drafted; critique of work stays | Judged |
| A span that no qualified model checks, at the *quarantine age* | Masked as `unchecked-language` with the member's English gloss (C11 Translation); the capture proceeds; the author is told and keeps the original locally until the capture is logged | Fixed |
| Text that tries to override an agent's instructions | `held: injection`; after a gate release, fenced with `instruction-fenced`, which tells the agent that the text is quoted and not addressed to it | Judged |

- The board applies deny matchers in the reader's browser, as a client. When a scope with deny rules has no reader with a client, its foresters are told once through the last-reader question.
- A non-text source is checked through its extracted text and metadata; personal metadata is stripped. A payload with no extractor becomes a reference-only source (hash, stripped metadata, location); it is never queued.
- Masked values are noted by attribute and kind; the agent or adder is told, and warned above the *masked-cell warning*.

**Hide** [toward safety].

- Any reader can hide a record for a suspected leak, with reason `credential`, `personal-data`, or `confidential` (text that belongs to a narrower scope). It moves at once to a **hide scope** read by its author, the hider, and the scope's foresters; others see `hidden-pending`; model calls and text-bearing derived data drop it. The hider can undo a hide.
- Until a forester removes or restores it, its content-free effects stay (the I5 exception); its warnings show in the floor as `hidden-warning`.
- A hide for another reason takes effect at once only on the hider's own unused record; else [team change].
- A forester who is not the hider, when one exists, removes the record, restores its scope, or moves it to another scope, confirmed by a forester of the destination who reads both scopes. A move runs the removal steps for derived copies outside the destination. After the *hide escalation*, parent or team-level foresters, or its other readers, are asked; with no second person, the sole forester then acts.

**Removal.**

- Removal is only for personal data (a departed member's author-only records included), credentials, and legal obligations; an extension may add a retention rule (X10). Otherwise a record is retired.
- **Remove a record** (a forester; an author asks by hiding). For personal data or credentials it takes effect at once, is logged, and reaches every event in the record's fold and its spans quoted elsewhere. Removing a source removes what was drafted from it, except a drafted record with independent support, which is re-derived from that support alone and checked for paraphrase. A removal can also select a batch, by author and optional window or by source, as a revert does.
- **Remove a scope** (a forester, under a declared legal obligation, confirmed by a reader of each child scope). It removes every event in the scope and its children, and keeps `removal` events and release records.
- **Remove content** (a pattern). Every scope is searched; the requester sees counts only for scopes they forester; other scopes' foresters confirm there (after the *removal confirmation timeout*, a team-level forester who reads the scope, else its readers). The pattern lives only in memory.
- **What removal does** (I1). The `removal` event holds no content. The server:
  - deletes the content from the log, indexes, derived data and history, the rollback generation, renderings, test queries, canary items, caches, and reachable exports;
  - makes the old content of changed events unrecoverable from every stored copy, backups included (personal data and credentials at once, others on the *removal batch schedule*);
  - rebuilds derived data and checks for paraphrase;
  - unless the remover opts out, installs a **deny rule** per matched scope: a matcher that only that scope's readers' clients can apply, so the server cannot test a guessed name; the remover may set an end date; text with no client waits in the quarantine;
  - withholds client keys until clients purge;
  - reports every place reached and not reached (outside services and agent-tool providers, member exports, local transcripts, unacknowledged clients).

  Records resting on removed content inherit a cause.

**Data subjects and departure.**

- "About me" lists the readable records that name the member, who can ask a forester to remove a statement or replace their name with "a member". Requests about records the member cannot read go to their scopes' foresters, who answer only by a release record or a removal; the requester learns nothing else.
- A departed member's handle is pseudonymized on read; no event changes (I1). A departed member's private records are never reassigned for reading. After the *author-only hand-off*, their **author-only records** are removed unless they moved them. A private record is author-only when the member's own cause put it there: their own choice, or routing of their own health or private life. Records routed there for another cause (the confidential-outside-person rule, or an install setting such as `unresolved sessions: private`) are not author-only: the private move question covers them before departure. Other private records stay, unread. A team-level forester can apply outside-person removal to a departed member's private scope by count, with no read. A forester sees counts only.
- Per-member measures are shown only to the member. Foresters also see hide counts and the hide release rate, to reverse a member who hides too much (I9).

## C11. Capture and intake

**Basis:** M9; I1, I5, I8.

**Capture paths.**

| Path | What it produces |
|---|---|
| Live capture | Once the agent chooses what to try, a case (outcome `open` when unknown) |
| Member input, pasted material | Facts, corrections, reasons, ideas, and concerns a member gives or pastes, the member as speaker (advisory) |
| Session review | What live capture missed (below) |
| Documents | Adds, watched paths, drafting (below; C14) |
| Members | Stated heuristics, notes, outcomes, re-checks, corrections, on the board or through an agent |

- **Session conditions** (I2, I6). The client adds to each capture, marked `auto`, the versioned subjects and condition values the session named (prompt, pasted text, tool calls), read, or wrote, and a recorded deploy; marked `recorder`, the agent tool and model, time zone, and hour band. Neither affects applicability unless a member links one as a real condition [team change]. `auto` ones feed strata, indirectness, and the condition test; `recorder` ones only a risk.
- **Concern routing.** When a capture, or a document entry that X4 drafts as a concern, names a subject with an open case or work item with `outcome_due`, intake proposes the `about` link (derived), so the note becomes a concern routed to its owner.
- **Pausing.** A member can pause and resume capture and review for a session; a scope forester can set `capture: off` [team decision], with notice to the scope's members ([toward safety] only when it stops suspected leaked or special-category text from being captured; it is reversible). Explicit member commands keep writing (D5).
- **Session review** (I5, I8).
  - The client reviews sessions from the transcript, which never leaves the machine. Only excerpts that passed the safety checks go to the shipped models (which keep nothing) or a model the member names. A tool with no transcript gets a session record from what the tool reports at each moment; a tool-protocol session, one from its calls; kept at most the *session record retention*.
  - It drafts: cases for missed decision points, set-asides, stated rules, supplied facts and reasons, anomalies, and outcomes; `next` and `waiting-on` work items from stated commitments (owner, `due` when stated, `about` the subject); `basis` (`cites`) links in reading order, `basis` (`possible`) from other observed reads, and `basis` (`produced`) from passages written; one **method case** per closed work unit (expectation, order of checks with sources, framing, split); and a safety-checked `member` source, the member as speaker, for each derivation the session produced (a chain of reasoning, a calculation, an argument supporting a recorded fact or case), the fact resting on its passages by `basis` (`produced` or `drafted`).
  - Drafts are member input, marked drafted, with the reviewer's model version, after the session's own captures (I1).
  - A *capture audit* counts missed decision points in a sample; only the counts leave the machine.
- **Translation** (the I1 exception). After its checks pass, text in another language is translated by a model qualified for it; the checks run again on the English. The event stores the English, marked `translated` (with the source language). An agent's English rendering of a member's words is a translation too, by a qualified model, with its version. Until then the text waits, marked `translation-pending`. At the quarantine age, a span that no qualified model can check follows the masking table. Its English gloss is member input (the member, or their agent marked drafted with its tool and model) that passes the English checks; it is not a translation, and the I1 translation exception does not cover it. The client queue wait (C1) ends at the quarantine age by this path.

**Intake.** Records and subjects go through all steps; other events with free text, through steps 1–2. Judge calls run in parallel and in any order; steps 3–6 commit in arrival order per scope.

1. **Fixed checks** (I5) before the write returns: fixed safety checks and shape gates.
2. **Judged checks**, in memory: safety and translation. Nothing enters the log before they pass: a passing or masked item becomes one event, a returned one nothing. The judged shape gate and the citation check run after logging; the item is `held` until they pass (a display gate).
   - The **quarantine** (the I1 exception) holds waiting text outside the log, under its own key, in backups but in no index or model call, up to the *quarantine age*: text with no client, until a reader's client applies deny rules; client-checked writes, with `server-check-pending`, while a judge or translator is down or a spend cap is reached.
   - Addressees (on the board, never in an agent's context) and the author's devices see a client-checked write, marked `checking` (or the original, marked `translation-pending`); text with no client shows only `checking`. When the checks pass, the item enters the log and the original is discarded.
   - At the quarantine age, a client-checked write returns to its author's queue as `returned`, marked `check-timed-out`, and is resubmitted when checks are available. A clientless write from the tool protocol stays in the quarantine for its author past the age, marked `checking-unavailable`, on the board's Questions panel, until the author drops or resubmits it. A refetch is dropped and retried; a watched-path write is re-queued at the next scan, shown in the foresters' view with its age.
3. **Split.** The judge proposes splitting a case whose attempts address several problems, linked `basis` (`part`) where nested; the author can revise it.
4. **Compare** with the top *compare candidates* of the same kind the author can see (cases and facts by concrete and abstract text and conditions; heuristics by `statement` and `when`; sources by `document` identifier, hash, location, then text; subjects by reference and alias), in this order:

| Class (I3) | Test | Derived result |
|---|---|---|
| One event | Same subject instance and the judge finds one event, or the capture continues an open case | `same` (`one event`); attempts join the earlier case |
| Recurrence | Not induced; a case in any situation (a candidate whatever its similarity), achieved before this case's first attempt, whose goal or `done_when` names a lasting state of this case's instance or its declared predecessor (C12 Known situation), and whose outcome came within the **Recurrence bound** before this case (the longest of the *recurrence window* and the *half-life factor* times the outcome horizon of its situation, and the lasting period that its `goal` or `done_when` states); this case's `problem` reports a state that contradicts that lasting state (the judge reads it against that case); else Repeat, or New for a new burst | `counter` (`recurrence`); one recurrence question |
| Repeat | Same situation, attempt, and outcome at another time or instance | `same` (`repeat`), keeping its own text, conditions, time, and warnings; a routine repeat (no warning, anomaly, condition outside the group, `serious` or higher `severity`, or, for a stream entry, exception) on one instance and procedure version within the *time stratum* is kept as a count on the earlier item, with no separate text record; each counted occurrence, with its time, is an item as a `repeat` is (C9 Items) |
| Counter | Same situation and attempt, different outcome | `counter` (`outcome`), with the differing conditions |
| Same heuristic | Same `kind`, `statement`, and `when` meaning | `same` (C5) |
| New | None | — |

   - An outcome that closes or changes an attempt reruns Repeat and Counter.
   - The *duplicate sweep* runs every class.
   - When the nearest same-situation cases carry one warning kind above the *anomaly share* and the new case has none, it shows `anomaly-not-stated` (the kind), and one anomaly question names the kind; not checked counts as missing, not clean.
   - Facts: another observation of the same thing is linked `same`; one from a later source with no observation of the subject (not `primary`, `first-hand`, or a re-check) is proposed as its `copy`; a later state of the key, or a later source version on the same passage or key, follows state over time (C3); another conflicting one about the same time (the same observed time, or a stated valid range that holds the other's observed time), `counter` (`claim`).
   - Sources: a match by `document` identifier, hash, or location (for a linked-repository source, within one ref, C3 Version order) is a new version when the text differs. In a restricted session, a match to a readable source resolves to it, its links keeping the session's scope (I5). A later team add of a restricted-only source is offered to its readers to rest on. Near-identical text is `same` (`identical`) when it passes the C3 Links test; else a copy question (across refs of a linked repository, none, C3 Version order).
5. **Uses.** Derive `basis` (`uses`) from an attempt to each subject instance it consumed or measured with (named in its action or conditions, not every `auto` value; an `auto` value of a kind with registered `signal` kinds that the attempt's tool calls wrote to or ran on counts as consumed), and where validity depends on a subject version or condition kind.
6. **Weigh** (value weight, C12) and **notify** the builder, the view writer, propagation, and the notice check.

- An offline write is classified at its `made_at`; later records are classified again.

**Gates** (I8; D15). A gate sets `held`; explicit search returns the record.

| Gate | Holds | Criterion | Check |
|---|---|---|---|
| Shape | Case | No attempt | Fixed |
| Shape | Heuristic | No `statement` or `when`; no `why` except as C3 allows or `not-stated` on an untried idea note | Fixed |
| Shape | Heuristic | An attitude ("be careful"), not a move; a `frame` passes | Judged |
| Shape | Fact | No text, or no basis and no author | Fixed |
| Citation | Fact | Citation check `does not support`; its author is told | Judged |
| Area | Any | Out of area under `hold` (C4) | Fixed |
| Injection | Any | Masking table | Judged |

- A differing or corrected `why` from another member does not trigger the shape gate; it shows `why-differing` beside the heuristic, which keeps its record and rank.
- **Gate release**: `citation` by a re-check that `supports` or a correction; `shape` by a revision that passes the gate (held cases are mapped to situations; they count in no situation figure or page until released); `area` by an area change; `injection` by a forester, at once with a re-check (the fenced rendering passes the judged check again, or a second reader confirms), else [team change], seen meanwhile only by the forester and the author.

Documents are drafted into records by module X4.

**Open cases and outcomes.**

- **Outcome worker** (I3). When an `outcome_by_absence` window passes with no recurrence, counter, warning, or open concern, and the named check is observed or is "no return on the subject" (no new case, capture, or `one event` on that instance), it derives `achieved`, marked `outcome-inferred`, only when the situation's own closed cases with an explicit outcome after a silent window of the same length were mostly achieved, by the sufficiency rule: at the partial failure value, self-reported, counting only while the attempt has no other assessment. It infers nothing when the `outcome_by_absence` window is shorter than the situation's outcome horizon. A later recurrence or harm reopens it. Otherwise the case stays open, the entry's author gets an outcome question, and the open-case cutoff applies.
- Past the *open-case cutoff* (from `outcome_due` or capture), a case stays open but unasked until its subjects get new activity.
- A later outcome (a client's acceptance, a review months later) is another assessment, entered by command or on the Questions panel.

## C12. Delivery

**Basis:** M10; I6, I7, I10, I5.

**Moments.** A moment is defined by what happens in the work. C15 maps each tool's events to these work events.

| Moment | When it occurs | Candidates |
|---|---|---|
| **start** | Work starts in a session | Project state; work items waiting on the member; open cases by urgency (C14); changes since the last session, with owner notices; related documents and facts (*start caps*). **Orientation** (the first *orientation sessions* in a scope): pages of subjects the scope's open cases concern, decisions in force, procedures, and recurring situations first. No heuristic outside orientation |
| **subject** | The work names, reads, or edits a known subject; it reads a source about the subject; a member takes a work item; a successor starts (C14); a logged record-stream entry or document version names the subject (for its owners, under Waiting on me and at their next work start; it counts as a check moment outside a scope for C8) | The **subject brief**: current facts of every kind, open cases and items on the instance, decisions in force, open contingencies, within the *subject brief caps*. A new owner gets the full brief, open items first, its subject page first when the scope's open cases concern it |
| **situation** | The work maps to a situation or matches a procedure's `when`, through a member request, a stuck call, a check, a capture, or a failed action | **Procedure**: the top-ranked procedure `about` the situation; with no situation or no procedure, the top one whose `when` matches the trigger text at the *known-situation similarity*. Its verbatim steps and a **validity line**: causes; the record as counts when sufficient, else as basis; the conditions it last succeeded under against the current values, differing `auto` conditions marked `auto`, with no applicability effect; each procedure that differs from it, with the first differing step. **Known situation**: the nearest achieved case above the *known-situation similarity* on an instance the session touched, its declared predecessor, or else a sibling: its situation, attempt, outcome, and causes, then this situation's record (achieved and warned shares of its closed cases, concerns, and counters; counts when sufficient, else basis). With none, other sessions' open cases in the situation show as "Also working on this now". **Learned method**: the top learned `decompose` heuristic, marked `learned-method` with its count of method cases and basis links, or `no-record-yet`. **Held**: for an area-held situation, the count of held cases and heuristics and a pointer to explicit search (`held-knowledge`), counting only records of the session's scope or wider |
| **stuck** | The agent or member says the work is stuck | The stuck output (below) |
| **check** | A conclusion is about to be stated, or a committing action is about to run; for an action outside the agent, a work item's done or take, for its subjects; a decision case's status change toward `adopted`, for the decision's subjects and open concerns, shown before the status is logged and never blocking it | The floor, then up to the *check results* relevant: open concerns and contingencies, counter-cases, recurrences, minor warnings, dead ends, rival explanations, set-aside options, changed or corrected records the work rests on, new facts about the instance, differing decisions, better alternatives on a declared measure, and `check`, `action`, and `frame` heuristics whose `when` matches; each record the conclusion or action rests on along `basis` (`cites`, `uses`) at any depth within C7 Reached, folded as `via-uses` lines, that is `may-not-apply` for the session, or `no-record-yet` with only claim or interpretive basis (for a fact: its evidence class, or a `uses` premise's, is claim or interpretive), naming the key. A builder `not_when` found on items that include a serious warning is a floor item when the session's value matches |
| **date** | A held date on a work item, decision case, or dated subject reaches a lead step (*lead times*); the item maps to a situation as a check does | For its owners (for a decision case, also its concern speakers), else its author, else foresters, at their next work start and expanded under the item in Waiting on me: the floor, the subject brief of the item's subjects, and the situation moment's procedure |
| **notice** | A push (below) | One line at the next member request: the reason, the record, and its causes; in the budget unless urgent |
| **ask** | An explicit search or read (ask, find, show, stuck, or check) | Search results (C14 for documents) |

| Notice reason | Fires when |
|---|---|
| `change` | Propagation reaches a record the session was given or applies (C7); or a floor item of an offline check moment (C15 Failure) |
| `teammate` | Another member or session in the session's situation records a counter-case with new conditions or serious severity, an achieved new approach, a dead-end case, a case closing on the session's instance, a warning, or a concern |
| `switch` | The first member request here after the member wrote in another session: one line of what their other sessions recorded on this session's cases, subjects, and items since it last ran; records of this scope or wider only, others as one content-free pointer (I5) |
| `subject` | A new fact, source, or version about an instance the session touched; an owner notice (C7) |
| `contingency` | A value on a key enters a contingency trigger or stated-hazard range of severity `serious` or `critical`, as C7 Crossing a bound says; urgent; to the contingency's target owners (for a stated hazard, the instance's owners), naming the contingency or hazard and the fact |
| `condition` | An action result or capture matches the session's **watch list** with a value that appeared or changed in the session; the line names the condition now present and the record that failed or warned under it. The list holds the `not_when` entries, risks, and contingency triggers of records given or applied in the session or about subjects it touched; never an unnamed `auto` key or a condition every case carries |
| `date` | A held date (`review by`, `outcome_due`) enters a lead step, or any held date passes |

- Heuristics arrive only at the situation, stuck, and check moments and on request, and return only by a `condition` notice (D20). Every recall writes `delivered`.
- A **pointer** (such as `more`) gives the count of what a cap left out and the command that lists it.

**Detecting moments.**

- **Stuck.** Besides the injected text (advisory), the client matches **stuck signs** (*stuck patterns*) on action results and reply ends. On a match it injects a **stuck offer**: the agent may be stuck, and the stuck command returns what others tried in similar situations.
- **Before a conclusion.** At each reply end, the client matches *conclusion phrases*, unless a check on a similar query ran within the *check suppression window*; at most once per *check frequency*. The result comes as one extra turn (the I6 exception). When the tool is already continuing because of an earlier extra turn, or has no extra turn, it comes at the next member request, marked `after-conclusion`.
- **Before a committing action.** At each action intent, the client matches *commit patterns* and injects the check; it never denies the action (C15).
- **Binding** (derived; I1, I7). A delivery or stuck call binds to the first match: the session's open case whose stored situation matches (captures and deliveries store their situation id); a case in a matching situation that continues the session's work; a later case in a matching situation by the same member and scope within the *binding window* that shares a subject instance or cites a delivery; the case a hand-off's taker captures when its refs name the deliveries. Else none. Computed from events and stored ids, it survives a rebuild.

**Search and ranking.**

- **Queries** are English, written by the agent (advisory), and restated abstractly.
- **Retrieval** fuses lexical and vector results over concrete and abstract text, so that a record ranked high in any one list ranks high in the fused list. It filters by scope and by causes that leave automatic recall, and a ranking model reorders the top fused results (*rerank depth*). Automatic moments deliver only records of the session's scope or wider (I5); other restricted records come only as content-free counts, the readers' question, or an explicit read. A record's indexes also hold, scoped and masked, the queries of bound deliveries and the missed searches of cases that applied or cited it. **Relevant** means above the *relevance threshold*. Fallback: fusion scores, then lexical only, marked `degraded-search`.
- **Reference time.** A case or work item can state `as-of`: the time the work is about, never a condition. Without it, the reference time is now. Non-warning knowledge (its applicability, the subject brief, and recall) reads the state facts and source versions valid at the reference time, and records `replaced` after it; the newer record of such a pair shows beside the older one, marked `later-replacement` (its id and the time it took effect). The floor, warnings, and causes always use the current state and all causes.
- **Applicability** is a lookup: session condition groups against each record's stated and linked conditions and `not_when` (never its `auto` or `recorder` ones), with containment on `ordered` kinds. For each key a record names, the current state fact of each touched instance (C3), then of its `within` group one step up (C4), counts as a session value; a mismatch shows `may-not-apply` for that instance only, naming the fact. Where an instance has values under several stated condition groups (C3 In effect), the session value is the one under the group the session matches, else the one under no stated group, else unknown (`conditions-not-known`); a stated hazard without stated conditions matches when the value under any group matches. A mismatch shows `may-not-apply`, naming the entry; an unstated key shows `conditions-not-known`, naming the keys. For a group target, a warning's key matches when any current member's value matches, naming those members. At a committing-action check, a value that the action names for a key also counts as a session value, only to add floor items, marked `if-applied` (the warning applies once the action takes effect); it never causes `may-not-apply`. A judge places a new session value once for the team; until then `check-incomplete`.
- **Ranking**, within each kind:
  1. **Relevance tier**: tier 1 is within the *tier width* of the top score, and so on. One tier down: an out-of-area record; a case whose achieving attempt already appears in the top results without it (its **value weight**, D19: a cheap per-case estimate, checked headless on the *value-weight sample*; it ranks and never hides).
  2. **Applicability** to the session's conditions.
  3. **Support**: the support weight (C9), then records without a lowering cause before those with one.

  Never by author (I3). A `copy` or `equivalent` group is one result.
- **From other fields** (I10). One separate section: `general` heuristics whose examples share no field group with the query's current fields, each with one example and its origin. At every moment, other cases, facts, sources, and `field` heuristics must share a field group with those fields; the rest come only by explicit search. Records about an instance the session touched are in field for that session; the floor is exempt (I6 before I10).
- **Missed searches.** An explicit search with nothing relevant writes `delivered` (`missed`).

**Floor, budget, repetition.**

- **Floor** (C8): every serious or critical warning whose conditions do not mismatch the session's and that bears on what the moment concerns: at a committing-action or conclusion check, the instance or record acted on or concluded about, and the serious and critical causes, direct or inherited, that the acted-on instance carries through the subjects it rests on by `uses` (C7 Reached), one `via-uses` line per origin (the origin and its `uses` path), with the group trends shown for those subjects, folded past the display bound as C8 folds a predecessor's warnings; at start and the subject, situation, and date moments, the session's or item's situation, records it applied, cited, or was given, and instances it touched or the item's subjects. A warning about a `within` ancestor of an instance the moment concerns, at any depth and within each containment fact's `since`–`until`, bears on that instance; at a check on a group, so do the warnings on its current members one step down, naming those members. Both fold past the display bound as `via-uses` lines do. Unknown keys show `conditions-not-known` and join the C7 Reached question. No budget, busy state, cap, area, repetition, limit, revert, hide, or cause that leaves automatic recall holds it back; an injection-held record's warning shows without its text. Floor items take one line per record per session, ordered by C8 Display, then by ranking.
- **Interruption budget.** Pushes (questions, notices, concerns outside a reached moment) share the *interruption budget*; recall at a reached moment does not count. Over budget, questions move on; notices and concerns wait for the next check moment or go to the board.
- **Busy.** A session with its own stuck call, stuck sign, or committing-action match within the *busy window* is busy: non-urgent pushes wait and never go to another busy session.
- **Urgent.** A notice that carries a serious or critical warning on an instance the recipient owns or touched, or on an instance that an instance the recipient owns rests on by `uses`, at any depth (C7 Reached), or on an ownerless instance it is routed for (C13), is urgent: outside the budget, counted in floor load. An urgent question with no live session moves on after the *urgent reroute*.
- **Caps.** After the floor, results follow ranking, then a `more` pointer. When more heuristics are relevant than fit, the last slot follows the least-shown rule (C9).
- **Repetition**, one rule at every moment: per reader, per record or originating warning, per instance and conditions. In a session a record returns only if it changed, a `condition` notice fired, or the action concerns another instance. Once per session: a procedure and a known situation per situation. A non-warning delivered unchanged in more than the reader's *seen-before sessions* within the scope's activity window shows only in one `seen-before` line with its count. Repetition for a date moment is per lead step. Floor warnings follow C8 Display.

Output content (stuck output, markers, injected text): A5.

**Recording use** (I7).

- The agent records each use with a trial (advisory): `applied` or `declined` with a reason, and `acknowledged` for a warning the member went ahead under, with the instance and action kind. Declining is a correct choice, not a use. A trial after the outcome is `late`. Trials change no use and no arm. Advice and an outside performer's choice are separate attempts; only the attempt that does what H says is a use.
- A **new approach**: an achieved or partial attempt matching no heuristic delivered in its session; one classed a repeat is a **rediscovery**.

## C13. Questions

**Basis:** M11; I8, I9, I3, I5.

- Questions are multiple-choice; no knowledge waits on them. Only a safety or access operation may wait on a confirmation (operational). An answer is the answering member's event, under that event's own rule (mostly C6). Ignoring any question is safe.
- **Routing** (I9, I5). One reader at a time, who must read the target's scope (else the next step applies; notice recipients likewise): (1) the target's active owners, live sessions and working hours first, in a group the one with fewest open routed items (Load rule); (2) else, for an outcome, who last added an attempt, otherwise the active author; (3) else up to the *routing candidates* members who most recently applied or rest on the record, by the load rule (below); (4) else, for a source's items, its subject owners; else the scope's triage group; else the foresters' view and the Questions panel. Never a model or a departed member; a service's questions go to its principal. A scope's foresters are a group for every forester-addressed question and confirmation (the step 1 group rule). **Load rule** (step 3, C14 owner proposals, forester-fallback candidates): ties go to the candidate with the fewest open routed items (questions, re-check work, owned open items); a candidate whose share of the scope's open routed items is above the *routed-load cap* is skipped while another candidate remains. A question moves on after the *question reroute* and expires after the *question expiry*, neither while the member's schedule marks them offline or their queue holds writes newer than the question. The target's causes stay.
- **Independent questions** (I3): critical warnings, warnings past the *floor-exposure threshold*, trends, and condition-test `caused-by` proposals (C9) go only to active readers independent of the record (C5), else wait on the board, marked `no-independent-member`. They never expire; after the *independent moves* they rest open on the board, grouped by record and warning kind with counts. No role adds standing to an answer.
- **Delivery** (I5, I6). A pushed question counts toward the budget. The agent appends it in a non-busy session of the target's scope; with none within its activity window, a wider session gets only a content-free pointer, `question-pointer` (an opaque handle that resolves only in a session of the target's scope or on the board, and where to answer). Critical review questions lead each queue, after the questions that C14 puts first (last reader, holder-set reasons).

| Kind | Trigger | Recipient | Answers and the events they write | Limit |
|---|---|---|---|---|
| Critical review | A critical failure; a warning past the *floor-exposure threshold*; a warning acknowledged on the *acknowledged instances* distinct instances; a speaker's withdrawal or revision of a warning; a critical `inherent risk` assessment | Independent readers; the warned case's author or subject owner gets a notice | Cause removed, did not happen as recorded, or smaller than recorded: a correction with evidence, as C8 Standing allows. Not known: no change | Threshold form: once per *question expiry* per reader; `inherent risk`: once per instance group and *inherent-risk question interval* |
| Recurrence | The Recurrence class (C11) | Routing | The fix did not hold: the earlier attempt becomes `partial`. It handled that occurrence, not the cause: the pair is a Repeat. Else by content | — |
| Support | A team change without evidence (C6) | One routed member | Agree with evidence: independent support. Disagree: a dispute | — |
| Re-check work | A closed case is asked (C7); a record with an open valid range gets `changed: input` (C7) | Each owner of asked cases or of records with `changed: input` | Still holds, or not applicable: clearing by C7 | One per owner and trigger |
| Outcome | An open case past `outcome_due` with no inferred outcome (C11) | Routing step 2, then step 1, then step 3 onward | An assessment: `achieved`, `partial`, `no-effect`, `failed`; the outcome plus a `harm` assessment; a valid negative result | — |
| Anomaly or difference | `anomaly-not-stated`; a `counter` (`outcome`) with no differing conditions; a key unknown on a set of runs (C9); a matched instance's unknown keys (C7 Reached) | Routing | Present: the anomaly on the case. Not present, checked: clean for that kind. Not checked: missing. A value or difference: condition values, the member as speaker | C7 Reached: one per owner and trigger, bulk answer |
| Mechanism | — | Routing | A confirmation: `mechanism: confirmed`; else by content | — |
| Reason | A missed decision point or set-aside; a procedure step whose `why` the source does not state; an achieved case of a holder-set member | Routing | `why` text or conditions, the member as speaker: `stated` when given before the outcome, else `inferred`, marked `reason-reconstructed` | — |
| Review | A flag; an occurrence assessed above its stated hazard's severity (C8); a serious concern open at commitment (as an independent question) or whose speaker left; an unreachable source past the *unreachable to aged* | Routing; a departed speaker's concern as C3 Concerns says | By content | — |
| Same? | An alias near match; a `possible` basis; a duplicate; a copied fact; a restricted-only source added to the team; an unsure harm cause; an unsure hazard occurrence (C9 Harm); a reference reuse (C4 Aliases and successors) | Routing | Yes: the `same`, `copy`, real `basis`, `about` (hazard occurrence), or part link (C6 on another's record). For a reuse, No, a new unit: a new subject with the reference from now, the old reference's `until` set (C4 Aliases and successors) | A link confirmed after its trigger applies the trigger's cause and closed-work question as if it had existed then |
| Successor | Intake is unsure whether a later item is a later state; a pending later state (C3 In effect) | Routing; for a pending later state, as C3 In effect says | It changed: `replaces`, credit `kept`. The older was wrong: a `wrong` correction. They conflict: `counter` (`claim`). Each holds under its own conditions: a condition on each fact | — |
| Conflicting decisions | Conflicting adopted decisions (C6) | Both owners | One replaces the other: `replaces` (opposite answers are a `counter` pair, C6 Differing). Each holds under its own conditions: the conditions, which resolve the differing | — |
| Challenge | The most-used records (C9) | As C9 Rivals says | By content (a case, a correction, a re-check) | — |
| Confirmation and hide review | A move, content removal, scope removal, or hide (C10) | The confirming foresters or readers | Confirm: the operation proceeds. Hide review: remove, restore, or move | Escalation after the *hide escalation* or *removal confirmation timeout* |
| Membership and last reader | The private move question; one active reader left or an expected departure (C14) | The scope's foresters; the last reader | Membership changes; a successor, a custodian, or a release | — |
| Interpretation | A document class; a translator below the *canary band*; other judge doubt | Routing | The chosen class; that the English differs: a material revision | — |
| Release | A restricted warning applies outside (C8); work end (C10 Release offers at work end); a released original changed | The scope's readers, else its custodian, else whoever access-role naming names; for facts and cases written while narrowed, their author | Write a version: a release record (C10), or a warning-only release (C8). Add the outside member as a reader: an access decision with a confirmation. Update or keep a release. For the author: widen (C10 Restrict, move, widen). Decline: no change | Restricted warning: one per warning and situation, urgent |
| Still true | An aged fact or differing fact pair on an instance the session touched; an aged contingency (C3) | The session's member; for an aged contingency, its target's owners | Yes, checked, with a citation: a re-check. Yes, I know it first-hand: from a member first-hand for the instance, an observation of the unchanged value; from anyone else, `reviewed`. A side or a new value: `first-hand` by the one test (C3), else relayed (C6). For an aged contingency, Keep it: renews the contingency (clears `aged`). Not checked: no change | — |

- "By content" means the answer writes ordinary events (a condition, a correction, a link, a re-check), each under its own rule.

## C14. People, continuity, and views

**Basis:** M12; I5, I8, I9.

| Event | Effect |
|---|---|
| Member joins | Reads team-visible records and their groups' scopes |
| Member leaves | Credentials and keys revoked; queued writes follow C15; questions move on; items show `receiver left`; owned items listed for reassignment; private records follow C10 |
| Hand-over in a scope | A scope membership change with a leaving date and an optional successor: owned items there go to the successor as a proposal; the leaver is asked once to move private records about the scope; the successor gets orientation for the scope and, per owned subject, the full new-owner brief at its first subject moment or first lead step, whichever is first |
| Access narrowed | Keys for lost scopes revoked; derived data rebuilt; the scope's foresters see what left under the wider visibility |

- **Owner transfer.** The member, the successor, or a forester reassigns **owned items** (subjects, open cases other than work items, decision cases) in bulk. A departed or handed-over member's work items (owned or taken) return to untaken with `receiver left` and go to the successor as a proposal to take. For a subject, open case (a returned work item excepted), or decision case that a departure or hand-over left unassigned, the most active member (most recent applier, citer, or adder, or a holder-set member), by the C13 load rule, is proposed as owner; they may decline, then the next, up to the *forester candidates*; then the scope's triage group, then its foresters, get it as routing targets, never as owners.
- **Forester fallback** (I5, I8). A forester whose questions wait past the *forester wait*, or who is inactive in the scope (a sync counts), gets a proposed candidate: the scope's most active member, by the C13 load rule, who may decline, then the next, up to the *forester candidates*. In a restricted scope only readers qualify. A parent-scope or team-level forester names the candidate, else access-role naming applies. With no candidate and no holder to name one, the scope **freezes** until a naming takes effect: no release, membership, or visibility change; only credentials and personal data can be removed, by a team-level forester with a reader's confirmation.
- **Access-role naming** [team decision]. With no holder to answer (no team-level forester, or no active reader of a restricted scope), two independent active members (not one person by the C5 counting rules, and not in one `coordinated` group), one of them a team-level forester when one answers, name a forester or reader for a scope. Members excluded from the scope or an ancestor scope can neither name nor be named, and their disputes do not count. The notice to all active members carries an opaque scope handle, the named member, and the namers, never the scope name or records; disputes cite the handle; the scope's former readers and custodian see the name. With no second active member, the sole member names after the same notice and period. Until it takes effect, a fallback candidate or named person takes only actions toward safety.
- **Custodian** (I5). Set in advance by a scope's foresters in the scope setting `custodian` (a member or group). With no active reader, the custodian becomes one after a safety rescan. When a restricted scope has only one active reader left, or an expected departure would leave none, the last reader and the foresters get one last-reader question, first in the queue, once per departure: name a successor or a custodian, or release a version of the records.

**Work items and project state.**

- **Project state** is a view over the scope's work items and the facts about its `project` subject (`phase`; `health`: `green`, `yellow`, `red`; work done, as observations), each with speaker and observed time.
- A **work item** is an open case with an owner or a taker, or one returned to untaken (Owner transfer), optional `due`, severity, and refs; finishing it records its outcome. A `next` item belongs to whoever takes it; a `waiting-on` item names who it waits on, and one that waits on an outside role or organization belongs to its author unless it names an owner (X4) and is never untaken except by its holder's return or departure (Owner transfer); a **hand-off** is one addressed to a member or group. An untaken hand-off to a group waits on the members that the C7 group-notice rule picks.
- Work end records work done and state: `next` items, `waiting-on` items with whom they wait on, refs, `due`, and severity. Item and project commands record later changes.
- Taking an item records the owner; for a group, the first take wins. Only a take, a return, or Owner transfer changes who holds an item. Done records an achieved outcome; return gives the item back with an attempt on its case (the question as `why`, outcome `open`).
- An item for a member or group who cannot read the scope is refused (I5); the author can send a hand-off text instead: a content-free pointer, and the text after the narrowest-input check.
- **Urgency order** for items and open cases (coordination, not recall ranking): the floor and the item's severity; an open concern or contingency; a date within its lead window or passed; recency. It hides nothing.

**Documents and pages.**

- **Adding documents.** The add command adds a source from a file, a URL, or an identifier. A member can mark it urgent: hazards it states are `serious`, and the urgent refetch schedule applies.
- **Watched paths** (I8): folders whose files the client adds as sources (meeting notes and chat exports included, drafted as record streams by X4); in a linked repository by scope settings, in local folders by client settings that name their scope. A changed file is versioned with `base_version`. A file moved within the watched paths of one scope and, in a linked repository, of one ref, with the same hash, is the same source; a move to another scope's watched paths is a move (C10 Restrict, move, widen).
- **Refetch** (I4). URL sources, version-pinned ones with a location included, are fetched on the *refetch schedule*; each fetch is a `signal` (`fetch`). A change is a new version. An unchanged fetch re-checks the facts cited to it: it clears `aged` but is not independent. A refused fetch or a missing page sets a cause; the last version stays.
- **References.** The writer resolves each source's references, by identifier or compare, as matches arrive: into `basis` (`cites`) links from the citing passage, with its sentence as the reason; shared translation or publication-stage identifiers into `same` (`equivalent`). An unversioned reference resolves to the version in effect at the citing record's observed time, by version order, marked `version-by-date`.
- Sources go through intake; the masking unit is the passage or the table column (I5).
- **Finding documents.** The find command searches sources and pages by description, within bounds on author, publication date, an as-of date, and conditions. An `equivalent` or copy group is one result. Date bounds filter by `published_at` before ranking; an as-of date returns each source's version and status at that date, marked `as-of`.
- **Source use** is derived (I3): the distinct closed cases whose attempts rest on a source, weighted by outcome and by age with the record half-life (the unweighted count beside it), plus sources citing it; summed over `equivalent` sources; only cases and sources independent along `basis` count. Echoes count as use, never as support. With session conditions, only matching cases count.
- A **page** is a view of one subject or situation, written from the records `about` it; navigation is by containment, `same` aliases, and situation proximity. A subject has a page once a record is `about` it; every source has one. A situation has a page when its cases within the *base activity window* come from at least the *situation page members* (all members in a smaller team). Variants follow C10. Every statement links to its passage, fact, case, or heuristic and gets the citation check against them: `does not support` is not shown; `partly` shows `citation-partial`.
- A page is rewritten when a record it rests on changes (I4). A replaced statement stays in history, marked `superseded` with its time. A page can be rebuilt as it was at any date (I1).
- Page statements follow the output markers, with a pending correction beside its statement.
- **View statement correction.** Each page statement has a stable reference. A member corrects one page statement as `wrong` or `narrower`, with a reason [team change], as a correction of another's record. It overrides model text and changes no knowledge (I1).
- **Sides** (I3, I9). When inputs differ, the page shows each side with its evidence and speaker, ordered by C6. Copies count once with their parent. Differing procedures are listed by step, never merged.

The board: X5.

## C15. Integration

**Basis:** M13; I6, I1, I5.

- **Common input and output.** Each adapter turns its tool's events into one common input and turns one common output back into the tool's form. The common input names the tool, the work event, the session id, the working directory, and the transcript path (or none). Per event it adds the member request text, the tool call (with a summary of its result after it runs), or the agent's last message, and `continuing`: whether the tool is already continuing a turn because of an earlier extra turn. The common output holds text to inject (`inject`) and an instruction to continue the turn (`continue_with`); each can be empty.
- The table below is the only place where tool event names appear.

| Work event | Common event | Claude Code, Codex | pi | Reaches |
|---|---|---|---|---|
| Work starts | `session_start` | `SessionStart` | extension `session_start` / first `before_agent_start` | Start and date moments; injected text; scope resolution |
| Member request | `user_prompt` | `UserPromptSubmit` | `before_agent_start` | Subject and situation moments; notices |
| Action intent: a tool call is about to run | `tool_intent` | `PreToolUse` | `tool_call` | Check moment (commit patterns) |
| Action result: a tool call has run | `tool_use` | `PostToolUse` | `tool_result` | Subject and situation moments (a failed action); stuck signs; `condition` notices |
| Reply end: the agent ends a reply | `turn_end` | `Stop` | `agent_before_settle` | Check moment (conclusion phrases); stuck signs |
| Work ends | `session_end` | `SessionEnd` | `session_shutdown` | Watched-file versioning; session review |
| Output: inject | `inject` | `additionalContext` | prompt section | — |
| Output: continue | `continue_with` | `decision: block` with `reason` | appended entry with `continue: true` | — |

- Another tool joins through an adapter that produces the common input; a missing tool event loses only the moments that need it.
- A **turn** is one agent reply, up to its reply end, or one tool-protocol call.

| Rule | Requirement |
|---|---|
| Time limit | The *hook time limit*, server calls included. On timeout the adapter returns empty and queues a dropped floor item for the session's next moment (I6); at a check it instead injects `recall-unavailable`, and the dropped floor items arrive as urgent `change` notices |
| Failure | Empty output (except the `recall-unavailable` marker), exit status 0; dropped floor items as for the time limit. With the server unreachable, the adapter injects `recall-unavailable` (warnings may be missing); at reconnection, the floor items of the offline check moments that apply arrive as urgent `change` notices |
| Never blocks | At an action intent the adapter only injects; it never denies a tool call (I6) |
| Loops | No `continue_with` when `continuing` is true |
| Headless | In a run that carries the client's headless flag, every adapter call returns at once with nothing; any other unattended run is a normal session of its principal (C1 Service) |

**Session environment and installation.**

- A session belongs to one deployment: by linked location, else the member's choice. Another deployment's client in it delivers and captures nothing (I5).
- At work start the client resolves the session's scope (C10) and gives the agent's commands the session id, the tool, and the session's current scope.
- Install signs the member in on this device (or a service, with its credential and principal; it can also issue a delegated-session credential), detects agent tools and their model providers, registers the adapters, starts the session reviewer and watched-path scanner, and runs one non-interactive session per tool to confirm injection and transcript access. Install verification repeats this with the capacity and export checks.
- The first member to install a deployment becomes its team-level forester. Install creates the default scope and asks where sessions in unlinked locations go: to the default scope (the default) or to each member's private scope. This and adding members are the one-time setup. No other forester duty is needed for knowledge to accumulate (**Low friction**).
- Server workers and the client reviewer call models through the data boundary. A client-side headless run sets the headless flag in its environment, so its adapters do nothing, and starts its prompt with the headless marker, so review skips it.

**Tool protocol.**

- The server, and the client for local use, expose every agent command (below) as a Model Context Protocol tool with the same inputs, outputs, safety checks, and scopes.
- For an agent that reaches Mycelium only this way: the injected text is the tool server's instructions; capture relies on the agent (advisory) and on a session record built from its calls with their inputs and outputs, under the *session record retention* and the safety checks; session review runs where that record is: on the client for the client's tool server, on the server for the server's, through the same data boundary (D26); its moments are start (as the instructions), explicit check and stuck calls, the subject moment for subjects named in its calls, and notices in tool results.
- Scope: chosen per tool-protocol session id. The server keeps its narrowing per session id; narrowing carries only to a later session of the same connection with no chosen scope; a call's `scope` argument can narrow, never widen (I5).
- Session id: from its first call or the agent. The session ends at work end or the *tool-protocol idle end*.

**Offline.**

- **Queue** (I1). The client keeps every write, encrypted, with its `id` and `made_at`, until the server reports it logged. A write returned from the quarantine stays queued as `returned` and is resubmitted when checks are available.
- **Keys** (I5). Queue and session records are encrypted per scope with per-device keys in memory or the system key store. Unrenewed keys lapse after the *key lifetime*; on a device with no server contact, after the *outage key lifetime* (set by a team-level forester). A lapse stops reads of session records, never the queue; renewal at reconnection comes before any purge. Departure, lost access, and removal stop renewal. A device revoke takes effect at once; its earlier queued writes go at the next contact authenticated by another member credential or a forester.

Commands: A6. Measures and model quality: A4.

# Part II. Modules

A module is a self-contained group of contracts behind one switch or one view. Core sections use only its named outputs. A module is a field extension (I10), a core option that runs under stated conditions (X1), or a core part kept apart (X3 knowledge holders, X4 document drafting, X5 the board).

X2 and X6–X12 are extensions (I10, M15). X1 is a core option: it runs in each situation while it is decidable, and a scope's foresters can turn it off for that scope (M7). X3–X5 are core.

- **Turning off.** A safety or visibility extension (X2, X8, X10, X12, and the A3 rows for cross-scope `person` matching and special-category data, a declared confidential reporting scope, and a scope the server cannot decrypt) is never turned off once it is on for a scope. Its own settings and records change only by the rules that govern them (matching consent is withdrawn by unsetting `match`, X2; a guest is removed by an access decision; a hold ends as a scope setting, [team decision]; a shared device is revoked toward safety). For any other extension, Values while off covers every item from the time the switch-off takes effect; existing warnings, trends, standing, and current values keep theirs (C6 Settings apply forward). For X1, X1 Turn off says what the switch-off does.
- **Values while off.** While an extension is off, an attribute that only it defines is kept in the log and has no effect. A value that only an extension defines, of an attribute or role that the core defines (a source kind, a `design`, an assessment's `evidence`, a `speaker`, a role), is kept in the log and reads in every core rule as the core default that the extension's section names.

## X1. Randomized comparison (champion and challengers)

**Basis:** M7; I7, I6, I3. On for decidable situations (D23); a scope's foresters can turn it off (Part II). Self-contained.

Turn on: none needed; the module runs in each situation while it is decidable (Decidability below), and elsewhere its outputs are empty. Turn off: per scope (Part II). An episode opens only where X1 is on for the scope of its case at its first stuck call (with no case yet, the session's scope); the episode is in that scope. When the switch-off takes effect, the following apply to stuck calls on a case in that scope (with no case, in a session in that scope):
  - an episode that is running continues with its menu and arm until it ends;
  - no new episode opens;
  - outside a running episode, a stuck call's champion and challenger lines and holdouts are empty, and recall treats each situation as outside decidable situations, so the least-shown slot applies (C9 Least-shown slot);
  - the partition's decidability, champions, and effects are still computed from the episodes they count.

Inputs: episodes, draws, and bound cases' outcomes. Outputs, the only ones used elsewhere: each situation's **decidable** state and **champion**, each candidate's **effect** with certainty, and each episode's **menu**.

- **Episode.** Opens at a session's first stuck call that maps to a situation, and binds to its case, not the session: later stuck calls on that case, in any session, reuse its menu and arm. It ends when the case closes or at the *episode idle end*; unbound, with its session. Its outcome is its bound case's final failure value; only the first episode bound to a case counts; unbound means missing.
- **Decidability.** Monthly, per situation and partition, over the later of the *decidability window* and the outcome horizon, with a baseline failure rate clamped to the *baseline clamp*: decidable when the expected episodes with an outcome per window, at the *decidability holdout*, give the *decidability power* to detect a rise of the baseline by the *detectable ratio* at the one-sided *decidability alpha*. A situation leaves after the *decidability exit checks* failing checks; its champion becomes an ordinary candidate; earlier randomized data keeps counting.
- **Menu.** Drawn once per episode in a decidable situation, logging every candidate's probability: the champion with probability 1 − h; up to the *challengers drawn* challengers (one more without a champion), drawn without replacement at a fixed number. Each candidate's inclusion probability is known before the draw, logged, within the *inclusion bounds*. Its candidates never appear under "From other fields" or outside their arm. Procedures are never candidates. A heuristic whose interval lies entirely above 1 after the *exclusion minimum* episodes is not a candidate unless the session's condition group differs from those of the excluding episodes.
- **Holdout** (I6). h is the smallest share in the *holdout range* that detects a rise of the champion's failure rate by the *detectable ratio* within one window. h is 0 in a scope set **high-stakes**, in a situation with a critical failure within the *critical-failure memory window*, for an episode whose bound case or item has severity `serious` or `critical`, and for a member who turned holdouts off for themselves [own view].
- **Rescue and crossover.** A held-out champion is shown (`rescue`) at the episode's first failed attempt, when its stuck calls reach the *rescue stuck calls*, or at a relevant committing-action check. A menu candidate shown outside its arm, in any session on the bound case, is a `crossover`. Either way the episode keeps its arm.
- **Estimand.** H's effect is its relative failure risk by intention to treat: episodes whose draw showed H against those where H was a candidate and not shown.
- **Analysis** is design-based, per episode, from the logged draw probabilities only, with these properties:
  - each arm's rate is unbiased under the logged draw probabilities;
  - the interval of each arm's rate, and of their ratio, is valid under repeated monthly looks and continuous monitoring, and the ratio's interval holds at the *interval level*;
  - missing and differing outcomes are bounded both ways; entry, replacement, and exit need both bounds to agree;
  - admission of champion entries applies the same control as the false-discovery control (C9 Support weight), over champion entries across all situations: the expected share of false champion entries stays within the *false-discovery level*, under the same repeated looks;
  - a champion's figure uses only episodes since entry; effects are never pooled across situations or age-weighted.
- **Randomization check.** When observed arm sizes fail a goodness-of-fit test against the logged probabilities at the *decidability alpha*, the situation's episodes are held out of the analysis and foresters are told.
- **Entry.** H becomes champion when its effect is sufficient, certainty is at least `moderate`, it has no critical warning or trend, its warning ratio is not entirely above 1, the control over champion entries (Analysis) admits it, and its interval's upper end is below 1; the lowest upper end wins. A qualifying challenger replaces the champion when its upper end is below the champion's estimate.
- **Exit** (I6, I9): when the lower end of its interval since entry, restarted every *decline restart*, lies above 1 (exit reason `recent decline since <check>`, not a cause); when certainty falls below `low`; or at once when a trend opens, or a critical warning with `checked` evidence or a second independent assessor. A self-reported critical report alone pauses the holdout [toward safety], with `critical-report-pending`. Evidence all before a `breaking` mark makes it a challenger at the next check. Between checks, only these and a generation switch that passed every check change a champion.

Tunables:

| Tunable | Default | Measured by |
|---|---|---|
| Challengers drawn; inclusion bounds | 2; 1/(2k) to 0.9, k = number of candidates | Challenger exposure |
| Decidability window; decidability holdout; decidability power; baseline clamp; decidability exit checks; episode idle end; holdout range; decline restart | 180 days; 20%; 80%; 10%–45%; 3; 24 hours; 2%–20% (bound: within 1%–25%); 365 days | Decidable share |
| Critical-failure memory window (per situation) | max(90 days, one recurrence interval) | Critical-failure memory |
| Rescue stuck calls | 3 | Capture quality |
| Exclusion minimum | 10 episodes | Challenger exposure |
| Detectable ratio; decidability alpha | 2 (a doubling); one-sided 5% | Grounding |

Acceptance properties:

- **I7-d Entry and draws (X1).** A sufficient randomized effect that meets the entry rule becomes champion at the next check; logged assignment probabilities match draw frequencies.
- **I7-e Valid under repeated looks (X1).** On simulated logs with known effects, intervals cover the truth at their stated level, false champion entries stay within the *false-discovery level*, and a heuristic with no effect seldom becomes champion.
- **I7-f Rescue and exit (X1).** A held-out champion is rescued at the first failed attempt, and the episode keeps its arm; an evidenced critical warning, or an interval since entry that shows it fails more often, ends the role.
- **I7-g Episodes (X1).** One case worked across sessions and members is one episode with one arm; a candidate shown outside the arm is a `crossover`.

## X2. Person scopes

**Basis:** M8; I5, I6. Per scope; the scope's foresters turn each part on.

Turn on: a scope whose work is about outside people (social services, health, law, genome) turns it on to hold them as `person` subjects with exact values for its readers.

- **`person` kind** (I5). The scope keeps only keyed hashes of identifying values. Readers' clients hold the key, wrap it for each new reader and under the export key, and hash variant forms of each value (normalized, transliterated, phonetic, initials with birth date) as for deny rules. A full match takes the subject's pseudonym; a partial overlap is an alias question. Outside the scope each record shows a different pseudonym. The map is removable as a unit.
- **Cross-scope matching** (I5, I6). With the consent of both scopes' foresters, readers' clients also send variant forms of touched subjects, hashed by the server in memory under a team **match key** that is never written to the log or indexes. A match goes only to the warning scope's readers (C10).
- A **person scope** is a restricted scope with the `person` kind on; never a team-visible one. `person` is not unset once set. A forester of either scope withdraws matching consent by unsetting `match` [toward safety]: matching, and the A3 matching beyond warnings, stops at once, and earlier matches stay.
- **Linkage check.** Each *linkage check interval*, attribute combinations that single out a person outside a record's scope become a question to its foresters, who generalize or withhold them.
- **Outside people.** A forester's person search matches the person's values without the server holding them in clear, and lists matches only in scopes the forester reads; other scopes' foresters get the request, and the requester learns nothing of them. A forester can export an outside person's records from a person search, confirmed per scope as for removal, other people's data masked. Removal for an outside person removes or anonymizes the person's records and map entries after per-scope confirmation, and installs deny rules.
- C3 Common attributes (`speaker`) adds: or in a `person` scope a pseudonym. While X2 is off, a pseudonym reads as the outside person's role recorded with it, else as the role `unstated role` (C3 `speaker`); it counts as one person per pseudonym.
- C10 Masking adds, for personal data in bulk: or pseudonymized in a `person` scope; for names of natural persons: or the pseudonym in a `person` scope; for special-category data: except exact for readers of a person scope with the field extension on (the I5 exception); for a confidential outside person, its routing applies only outside a person scope that holds them.

Tunables:

| Tunable | Default | Measured by |
|---|---|---|
| Linkage check interval | monthly | Safety findings, Question age |

Acceptance properties:

- **I5-f Matching.** A cross-scope match tells only the readers of the warning's scope; nothing shows in the other scope until a release to it, which then reaches its floor.
- R16-b adds: Outside a person scope, records about one person show different pseudonyms; near-identical names raise one alias question. Text that identifies an outsider stays exact in a person scope.
- **R15-d Successors.** After all original foresters and readers of a person scope leave, a successor decrypts a past export, and a new reader's query on a known name takes its pseudonym.

## X3. Knowledge holders

**Basis:** M12; I8. A view and its questions; on by default.

- **Holder sets** (a view, I8). A situation's or subject's **holder set** is the smallest set of active members that supplies the *holder share* of its achieved attempts, method cases, and stated reasons. Sets of at most the *holder set size* are listed at each *monthly check*. Their reason questions lead their queues, within the *holder-set question rate*, on a set member's recorded expected departure or scope hand-over leaving date (a held date with lead times), a project `phase` of `closed` or `paused`, or active set members falling below the set's size. A **drill candidate** (on the board) is a subject owned by, or with facts only from, a holder set, with no cases. Both are per partition; a restricted overlay's show only to its scope's readers and foresters.

## X4. Document drafting

**Basis:** M9; I1, I2, I3, I4.

**Documents as capture.** Records drafted from a document are derived (I1, I3): they rest on its passages by `basis` (`drafted`), re-derive when it changes, go with it, and count once per source `origin` and work unit. The judge classes a document as a recount, a record stream, a decision record, or a statement of rules and facts (M9). Recounts and record streams draft one method case per closed work unit they describe. Every source a member, an agent, or a watched path adds is also a subject of kind `source` (C4) at intake: its passages that claim something about the document's own topic then qualify for drafting. A stated recommendation with an addressee (a role, a group, or the subject's owners), as in an inspection report or an audit finding, is drafted as one `waiting-on` work item per named addressee (a named group is one addressee), owned by the subject's owners, else the adder when a member, else a service adder's principal, else routing step 4 (C13), with `due` and severity from the stated priority; a later passage that restates it joins each addressee's open item by `one event` (C11) and adds to its restatement count, which shows the first observed time. A recount or record-stream entry in which a speaker states a worry about a pending decision (an open decision case or work item on a named subject) is not routine: it is drafted as a `concern` with that speaker, through C11 Concern routing. A drafted assessment in a later horizon stratum than the deciding assessment takes the derived horizon condition (C9).

- **Recounts** (I2). A recount (a postmortem, a test report) reports attempts and outcomes; cases and facts are drafted from it. Observed time decides what came first, never dates in the text: parts in versions, or in entries timed by a declared time source, before the one that states the outcome are pre-outcome, their reasons and criteria `stated`; other parts, entries inside one untimed version included, are `inferred`, marked `reason-reconstructed`. A passage recounting cases already held (by identifier, or matching several in one situation) rests on each by `basis` (`part`), adding no item or recurrence. An entry that states an outcome of a held case is drafted as an assessment on its attempt: the speaker is the assessor, `checked` where the document is a check, resting on the passage, marked drafted.
- **Record streams** (a diary, site log, lab notebook, register, ticket queue) are drafted per entry. An entry whose action runs a held procedure or repeats a held case's attempt is always a use: a case, or a `same` (`repeat`) item, its values as `auto` conditions. Else a case comes only from an entry recording an attempt and outcome, or an exception. A **routine** entry (no attempt toward a goal) gives no fact unless cited, except a value of a declared state key of a registered subject, a state fact by C3 In effect; its other values become `auto` conditions of that date's drafted cases. A hand-entered entry is member writing, its stated author the speaker, first-hand by the C3 test; only an instrument- or system-written log is `primary`. An entry's stated author, or a declared time source's author field, is the speaker; within routing step 2 (C13), that member when active comes before the adder for its outcome questions; its reason questions go to that member when active, then to the adder, then Routing. A drafted case with no outcome takes `outcome_by_absence` through the stream's latest version, checked by "no later entry on the instance" (routine in-range entries do not count); a later entry whose declared-key value lies outside the case's problem range closes the attempt as an observation, `checked` when an instrument wrote it.
- **Decision records** (an RFC, an architecture decision record, a policy memo) yield one decision case, updated by new versions: the stated status, options as hypotheses, rejected ones set aside with reasons, the choice as an attempt with the author's `why`.
- **Rules, procedures, facts.** A passage stating a rule as a rule becomes a stated heuristic resting on it, the author as speaker; ordered steps for a situation become a procedure. Facts are drafted, the source as speaker (from member writing, the passage speaker set), only from passages a record cites, that state a hazard (a `hazard` fact), or that claim something about a registered subject. A conclusion, rating, hazard severity, or decision that a passage states as resting on premises rests by `basis` (`uses`) on the drafted premise facts. Such a premise that states an envelope, bound, or assumption is a `claim`, never that record's `conditions`; no stated bound or assumption becomes an estimate with no interval.
- **Withdrawal and errata** (I4). A version that reads as a retraction writes a derived correction, `withdrawn`, on the source or its named passages (one naming a successor follows C4). A source that names another by `document` identifier as retracted, withdrawn, or corrected writes the same on it. Erratum wording (a statement that earlier text was corrected or listed incorrectly) gives the replaced facts credit `wrong`. A passage that names other passages as substituted, amended, or revoked writes a derived `replaces` or `withdrawn` on them, resting on the amending passage; the `replaces` credit is `re-evaluate` when the passage states reach over existing work (existing installations, earlier approvals), so C7 Closed work asks the matching closed cases, else `kept`.

## X5. The board

**Basis:** M12; I5, I8. R5.

**The board** (I5, I8).

| Panel | Contents |
|---|---|
| Ask | Ask, find, and check, for people (C14 Finding documents) |
| Decisions | Decisions by status: in force, what each replaces, and differing ones |
| Waiting on me | Work items with their date moments expanded, date notices, and concerns about decisions the member owns, in urgency order; items they wrote, with state |
| Since I last looked | For the member's scopes and owned items: new and changed records, versions, causes, links, corrections, aliases (*board caps*), each kind by severity, then ranking, then a `more` pointer; first "Changes to records you were given" and "Changed while you edited" |
| Status | Records with causes, by recall frequency; differing pairs; counter-cases by situation and month with differing conditions; recurrences; trends; changes grouped by source version |
| Recurring situations | Situations with most cases in the *recurring windows* (one event counted once) |
| New approaches that worked | By situation, with any result better, on a declared measure, than the champion's, else the situation's top procedure or most-used heuristic |
| Questions | Unanswered questions, aged contingencies, overdue outcomes, and clientless writes waiting for their author, answerable together |
| Projects | Phase, health, next items, and last activity of the member's scopes; one idle beyond its activity window hides |
| Searched for, not found | Missed searches by situation, with counts; a query's text and members only to readers of its scope; counts from restricted sessions outside their scope only at the aggregate minimum |
| About me | C10 |
| Settings | For foresters: scopes, members, subject kinds, time sources, data boundary, safety rules, tunables, builds, with reason and measure for non-defaults; pending setting changes |

- The **foresters' view** adds: core measures; backlogs and builds; the oldest check wait and quarantine item; area-held counts; devices; unqualified functions and capacity; hides per member, removals, purges, held removal requests; rescan matches; pseudonym proposals; counts of departed members' author-only records; captures routed to private scopes by the confidential-outside-person rule, per scope (above the *private routing threshold*, one question to the scope's foresters names the person-scope option); holder sets and drills; foresters' open questions by age. Counts about a restricted scope follow C10.
- The board shows only what the member can see. Each visit writes `seen`.

## X6. Evidence synthesis

**Basis:** M7; I3, I2. Per interest area.

Turn on: a team turns it on for interest areas whose work compares designed arms over units, populations, or held-out sets (trials, field studies, unit-level experiments, benchmark suites).

- C3 Common attributes (`conditions`) adds server-derived `auto` values: the phase of year, `rater-calibration`.
- C3 Cases (`measure`) adds optional guardrail measures, each with its own thresholds.
- C3 Cases (`design`) adds: `unit comparison` (a treated unit's rate before and after a declared change, against declared control units over the same periods, declared before the after-period outcomes: controlled at the observational ceiling, risk of bias stated); or `allocated` (assigned by a rule the workers did not choose, declared before outcomes: controlled at the observational ceiling, risk of bias stated, never `randomized`). With a design, optional `interference`: the unit of randomization and the unit of interaction, or `shared pool` (arms share a resource or meet).
- C3 Cases (`attempts`) adds `effort` (optional: trials or compute).
- C3 Cases (`assessments`) adds `audience` (optional: the population condition group the assessment covers, Populations below); `blind` (optional, declared: `yes`, the assessor did not know the arm; `no`, the assessor knew it; C9 Risk of bias).
- C3 Facts (`design`) adds `allocated`, `unit comparison`.
- While X6 is off, the `design` values `unit comparison` and `allocated`, on cases and facts, read as no design (observational).
- C4 Registry adds: `held-out` is a declared attribute of an instrument version that is used only for final evaluation (Adaptive reuse below).
- C9 Track records adds the observational stratum `rater-calibration`.
- C9 Condition test adds: Before such an entry, the test tests the phase of year; a time-stratum change that repeats at the same phase is reported as that condition, with no `from` entry.
- C9 Risk of bias adds outcomes assessed in a controlled case by a person whose assessment has `blind: no` (never values an instrument or system wrote).
- C11 Session conditions adds: The **phase of year**, derived from observed time in the scope's *phase-of-year bins*, is an `auto` value.
- X4 Recounts adds: Assessments drafted from a recount whose source kind is not `member` or `primary` take `selection: triggered` unless the source declares its selection (a declared time source records the outcome and population before the outcome's observed time, I2, or the source states a pre-specified analysis), and form their own stratum.

| Attribute | Meaning (I2, I3) |
|---|---|
| `replicates` | Optional replicate key (a seed, a rerun index), declared before outcomes. Replicates of one attempt pool into one assessment `estimate` (value, interval, n), count as one item and one occurrence, and never raise a `counter` or a difference question |

- **Populations** (I3). An assessment with an `audience` covers that population. Assessments that differ only by `audience` are not compared as deciding: each population gives its own outcome and its own item of the use, with the population as an item condition, correlated within the case as items of one case are. Each guardrail measure is assessed against its own thresholds (C3 Tests); a guardrail outside its threshold is a `harm` of that assessment at the severity its assessor gives, else the *default warning severity* (C8 Sources of warnings), never an item of the record. The case's own outcome (founding cases, dead ends, the Recurrence test, outcome questions) is the primary measure's outcome for the population the case's conditions name; with no named population and differing population outcomes, the case is `open` while any population is `open`; else it is `partial` when at least one population is `achieved` or `partial`; else it takes the least favourable population outcome in the order `failed`, `no-effect`, `inconclusive`. A case's population items count as one case for *sufficiency members* and Concentration. A population or guardrail declared after the first outcome takes `selection: triggered`.
- **Rater calibration** (I3). A declared rating method together with its assessor is a reliability `origin`. A paired independent re-rating of the same instance within the same time window that differs beyond the method's stated repeatability is an adverse item for that origin. Such an origin never gets `source unreliable` and shows in no reliability display; its adverse share is used only as the calibration condition. Each rating carries the `auto` condition `rater-calibration`, derived on the server: `uncalibrated` when its origin's adverse share is sufficient (observational, as Reliability); `calibrated` when the record of its origin's paired re-ratings is sufficient (Sufficiency and certainty) and the share is not; else `unknown`; it joins the strata of the observational record and the condition test. A stratum or condition-test entry on it is shown only when it holds at least the *aggregate minimum* members. It never shows as a figure per person.

Certainty factors (C9), each lowering certainty by one:

| Factor | When |
|---|---|
| Adaptive reuse | The `held-out` instrument version behind most items has more than the *held-out looks* attempts in the situation |
| Interference | Arms of a controlled case share a resource or meet (`shared pool`), or its unit of randomization is smaller than its unit of interaction |
| Unequal effort | Arms of a controlled case whose `effort` differs beyond the *effort ratio*, or is unstated on one arm |

Tunables:

| Tunable | Default | Measured by |
|---|---|---|
| Held-out looks; effort ratio | 20 attempts; 2× | Record calibration |
| Phase-of-year bins (per scope) | Calendar months | Decline rate |

## X7. Legal and contract authority

**Basis:** M3; I3, I2. Per interest area.

Turn on: a team turns it on for interest areas whose sources include law, regulation, standards, or signed contracts.

- C3 Sources adds the source kinds `governing`, `agreement` (below).
- While X7 is off, `governing` and `agreement` read as `maker` (C3 Kind order): they outrank other core kinds, `primary` excepted, only for the specification or state they declare for their subject, and rank as `vendor` for effect or performance claims.
- **C3 Kind order** with this module: `primary` > `governing` > `agreement` > `maker` > `peer-reviewed` > `compiled` > `member` = `preprint` = `vendor` > `opinion`. `governing`: by a body with authority over the subject (legislature, regulator, court, standards body). A `governing` source is authority on whether a rule is in force and on states it declares, never on whether an approach works. `agreement`: a document its bound parties signed; it is authority on whether a term is in force between its parties and on the states it declares for them, never on whether an approach works. A party's unilateral statement about such a term ranks as `vendor` and shows beside the term until an amending `agreement` passage replaces it (X4 Withdrawal and errata).
- **Publisher order.** A scope setting [team decision] can order publishers within a kind, used wherever the kind order is used.
- C3 In effect adds: A fact or passage may carry a held date `in effect from`. Before it, the new fact shows as `scheduled` beside the current one and has no applicability effect; when the date passes, state over time and propagation fire. A statement is in effect from that date, else from its observed time.
- C5 Counting adds `governing` to the kinds whose `publisher` counts, and: an `agreement` counts once for each bound party.
- X4 Rules, procedures, facts adds: A passage that binds a named outside party ("Supplier shall give notice") drafts a fact about the agreement source's own `source` subject (C4 Registry), not a heuristic. Its key is `obligation <term> <bound party>`, a key of the `source` kind, one per term and bound party, with the passage's conditions.
- X4 Withdrawal and errata adds to an amending passage: with an optional `in effect from`.

## X8. Guests and shared devices

**Basis:** M12, M8; I5. Per scope; its foresters turn it on.

Turn on: a scope's foresters turn it on to let outside partners into a restricted scope, or to let one device serve several people (shop floor, clinic).

| Principal or role | Contract |
|---|---|
| Guest | Reads and writes only the restricted scopes that list them |

While X8 is off, `Guest` holds no role and reads nothing.

- **Shared devices.** A service or device that serves several people records the performer each entry states; stated performers count as distinct members (C5) only when the performer identifies to the device when the entry is made (as a viewer does for display), else the device's entries count as one member. Display follows C1 Shared audiences.
- C10 Narrowest input adds: Text from a scope whose readers are not all readers of a scope that lists a guest enters that scope only as a release record written by a reader of both, under the release check and the *generalization rules* (Release records). In a scope that lists a guest, text matching records of the team or project scopes is checked as restricted text is; its notice goes to the guest scope's foresters.
- C10 Partitions adds: A guest uses its scope alone.
- C10 Release records adds: A reader of both may also write a release record from team or project text into a scope that lists a guest; the same check, `basis` visibility, and update question apply, except that subjects the guest scope is `about` may stay named; such a release shows `from-team`.

Acceptance properties:

- I5-e adds `from-team` for team or project text released into a guest scope.

## X9. Instruments and scales

**Basis:** M5, M7; I4, I3. Per interest area.

Turn on: a team turns it on for interest areas that measure with versioned instruments or scales (laboratories, inspection, field monitoring).

- C3 Cases (`assessments`, `evidence`) adds `proxy <instrument>`: a check on a stand-in. While X9 is off, it reads as `self-reported`.
- C3 Tests adds: a baseline converted across a `bridged` mark (C7) counts as one after it, and the conversion's interval widens the assessment's interval.
- C4 Registry adds: `valid range` (a key and an `ordered` interval) is a declared attribute of an instrument version: the input range it is valid for (C12 Applicability).
- C7 Causes (`before <change>`, set by) adds: a `breaking` mark that is not `bridged`, or has no mapped value across a `bridged` one.
- C7 Own causes adds to version signals `recalibration` for an instrument; to the `breaking` row: a newer instrument version is an item condition and a proposed mark, which takes effect only with evidence of a measurement change (a changelog or release-note passage that names a measured quantity, or a bridging re-check whose values disagree beyond the instrument's stated repeatability; an agreeing bridging re-check clears the proposal), never by the undisputed period. A mark with evidence may carry the credit `bridged` with a stated conversion: a declared mapping, as for a classification subject's revision (C5), or an `estimate` from the bridging re-check, with its interval.
- C7 Reached adds to the first value after a mark, unless the mark is `bridged` and the converted earlier value lies in another bin.
- C7 Crossing a bound adds to a value observed before a mark, unless the mark is `bridged` and its converted value is used.
- C9 Assessments and outcomes adds: A `proxy` assessment decides only when no direct one exists; each disagreement with a direct one is an adverse item for that instrument version.
- C9 Track records (breaking strata) adds: Across a `bridged` mark (C7), items with a mapped value stay in the same stratum with converted values, and Indirectness lowers certainty by one; unmapped items follow the separate-stratum rule.
- C12 Applicability adds: A record or attempt that rests by `uses` on an instrument version, with a value at a key of the version's `valid range` (in its conditions or the session's) outside that range, shows `may-not-apply`, naming the instrument and key.

Acceptance properties:

- I4-c adds: a new instrument version is an item condition and proposes a mark; the mark takes effect only with evidence of a measurement change, and an agreeing bridging re-check clears it.
- I4-d adds: Across a `bridged` mark, earlier uses with a mapped value keep their weight at converted values, at one lower certainty.

## X10. Retention and legal hold

**Basis:** M8; I1, I9, I5. Per scope; its foresters turn it on.

Turn on: a scope's foresters turn it on when law or contract requires records to expire by age, or to be held during litigation.

| Event type | Payload | Written by |
|---|---|---|
| `retention` | Content-free aggregates from events a retention batch expires: counts per heuristic, situation, and arm | Server |

- **Retention and legal hold** (I1, I9). A scope's retention rule names kinds and an age, which may run from a dated fact on the scope's `project` subject (`phase: closed`) or a dated subject. For usage events (`seen`, `delivered`, `trial`), a forester sets it. For knowledge records it names only kinds that a declared legal obligation names, cited in the `scope.changed` reason; it is an access decision by the scope's foresters (I5), confirmed by a reader of each child scope, as a scope removal is. A forester confirms each batch by count, one per *retention batch interval*. A due `delivered` keeps what X1 needs; before a batch expires, one `retention` event records the content-free aggregates later builds need. In a team-visible scope (the default scope or a project scope), a learned heuristic or method keeps its text when its founding or method cases expire, and its record continues from the `retention` aggregates. In a restricted scope, before a batch expires founding or method cases, one Release question (C13) goes to the scope's foresters who read it, else as the C13 Release row says, for the affected learned heuristics and methods. The judge drafts a restatement of each; a draft that fails the release check is not offered, and the question goes without it; such a heuristic keeps its text only through a release record written on that question, and its record continues from the `retention` aggregates. In a private or intersection scope, a learned heuristic or method keeps its text in that scope, and no question goes. A hide scope holds no learned records. A legal hold targets a scope, or a subject (the records `about` it or resting on it). It stops every removal in its target, a scope removal included, except of credentials; leaked personal data instead moves to a hide scope read only by its foresters, and is removed when the hold ends.
- C6 Classes (Access decision) adds to its operations: retention rule.
- C10 Removal adds: a retention rule on knowledge records that nobody confirms within the *removal confirmation timeout* is asked again of the scope's foresters who read it, each *removal confirmation timeout* (of the child scopes' other readers); it never takes effect unconfirmed.
- C10 Remove a scope adds: a scope removal also keeps content-free `retention` aggregates.
- C10 Data subjects and departure adds: A team-level forester can also apply retention to a departed member's private scope by count, with no read. A legal hold suspends that retention and the outside-person removal there, and grants no read.
- C13 Confirmation and hide review adds the triggers retention batch and retention rule on knowledge records. C13 Release adds the trigger: a retention batch in a restricted scope that expires founding or method cases.
- A6 Foresters adds the scope settings retention, legal hold.

Tunables:

| Tunable | Default | Measured by |
|---|---|---|
| Retention batch interval | monthly | Safety findings, Question age |

Acceptance properties:

- R16-e adds: under a legal hold, leaked personal data moves to a foresters-only hide scope, and other removals are refused and listed.

## X11. Advanced record streams

**Basis:** M9; I1, I3, I8. Per interest area.

Turn on: a team turns it on for interest areas whose record streams are site logs, lab notebooks, or registers with repeated exceptions, consumed material, or dated terms.

- **Counted exceptions** (X4 Record streams): An exception entry on a registered instance that matches an earlier exception of the same kind on that instance and version, by the kind's declared pattern and with no judge call, is a warning occurrence kept as a count on the earlier item's warning, as a same-bin occurrence is (C3 State over time), and counted for trends by C5 (b). It takes the severity of the earlier exception's drafted case or warning, else the *default warning severity*. A case is drafted only for a new instance, version, or exception kind, a crossed bound, or a trend.
- **Declared patterns**: A kind's declared patterns read declared-key values from entries.
- **Consumed instances and replicate groups**: An entry that names a registered instance it received, consumed, loaded, or measured with drafts `basis` (`uses`) with `since` at the entry's observed time; a later discard or use-up entry sets `until`. Entries that share a declared replicate group pool as replicates while X6 is on, else each entry is its own item; entries that name a group id declared on an open designed case join it as variant attempts.
- **Dated subjects from entries**: An entry (or a watched calendar file entry) that states a future-dated event naming a registered subject drafts a subject `within` it of the registered dated kind the entry names (a contract term, a last-time buy, a delivery due), else of the shipped `dated` kind, its date the held date.

## X12. Storage region and pseudonymous authors

**Basis:** M8; I5, I1, I2. Per scope; its foresters turn it on.

Turn on: a scope's foresters turn it on when its data must stay in one region, or when its safety reports need authors hidden from readers.

- **Storage region.** A scope forester can set one. The scope's payloads, indexes, quarantine, backups, export, and model calls stay in it; the team log keeps only envelopes. Builds over it run there as its own overlay; only content-free counts and releases leave, unless its forester lets the team-partition build run in the region.
- **Pseudonymous authors.** Under `authors: pseudonymous`, others see a per-record pseudonym, applied on read, for the author and member speakers, notice and "also working" lines included. The log keeps the real author for counting, routing, removal, and `seen` (I1, I2). A span naming another member as the actor in a failure becomes a role.
- C10 Masking (names of natural persons) adds: under `authors: pseudonymous`, a member named as the actor in a failure.
- **Ending.** Ending `authors: pseudonymous` or a storage region, or changing the region, applies forward: records written under it keep their pseudonyms, and every Storage region rule above holds for them in the region they were written in. Such records move (C6) only to a scope with the same region.

# Part III. Appendices

## A1. Tunables

**Basis:** M14; I9.

Values are team-wide unless a row says per scope, subject kind, or source. A forester changes one with `config.changed`, naming the measure it should move; Settings shows non-default values. A row that carries an invariant states its bound; a value outside it is refused. Any change that alters what others are shown or asked, in either direction, or how standing is derived, is [team decision] and applies forward as C6 Settings apply forward says: its notice goes to the members whose sessions it affects, it shows in Settings, it takes effect after the undisputed period, and it lapses on a dispute supported by a second independent affected active member (C6). Only a change toward safety applies at once [toward safety]. One row holds the tunables measured by one measure; a new tunable joins the row of its measure or adds a row.

| Tunable | Default | Measured by |
|---|---|---|
| Base activity window; orientation sessions; start caps; subject brief caps; board caps | 90 days (per scope); 5 sessions; 10 changes, 5 open cases, 5 documents and facts; 5 per subject, 10 per session; 50 per kind | Start completeness |
| Lead times (per subject kind and field) | Dated subject: 90, 30, 7 days; `due`, `outcome_due`: 1 day; `review by`: 30 days | Push yield |
| Key lifetime; outage key lifetime; hook time limit; tool-protocol idle end; identity-provider grace | 30 days (a scope can shorten it; bound: at most 90 days); 90 days, the key lifetime while the device has had no server contact (C15); 1.5 seconds; 1 hour; 7 days (bound: at most 14 days) | Hook health |
| Backup schedule; restore-test schedule; export schedule | Daily; monthly; daily | Restore test |
| Canary schedule; canary band; qualification thresholds | Weekly; 95%; per function: 95% canary agreement, fixture-median capture quality, zero fixture safety misses | Canary agreement |
| Build cadence; build deadline; ongoing model budget; value-weight sample; variant share | Quarterly (bound: at least yearly); one quarter; 200 calls per active member per day, refilled hourly, burst cap 4 hours, plus 15 per new record; 10% of new cases; 20% of a situation's members | Model cost |
| Switch find limit; switch stability limit; tuning split | within 2 points; 5% each; 70% tuning | Rebuild fidelity, Find success |
| Find success depth | Top 5 | Find success |
| Retired overlap; id carry-over share; passage carry-over similarity; pending-correction weight | 50%; half of the old one's members; 90%; 0.5 (bound: below 1) | Correction survival |
| Compare candidates; duplicate sweep | 5; weekly | Duplicate rate |
| Recurrence window (per scope); recurring windows | max(the scope's setting, default 365 days; three recurrence intervals of the situation (in the C11 Recurrence row, the earlier case's)); 30 days and the recurrence window | Recurrence rate |
| Situation minimum size; situation spread cap; query-situation neighbours; current-field neighbours; known-situation similarity | 3 cases; 90th-percentile spread of all situations; 10 cases; 20 cases; per ranking model, so that 90% of tuning-set matches are judged the same situation | Find success, Rediscovery rate |
| Area member threshold; area link overlap; broad field share; area exit checks; out-of-area handling | min(3, active members of the cases' scope); 0.5 overlap of linked sets; 50% of in-area cases; 3 checks; `lower` (`hold` allowed) | Out-of-area finds |
| Open-case cutoff (per scope); partial failure value; outcome horizon percentile | 90 days; 0.5; 90th over 2 years (below 10 closed attempts: the median declared `outcome_due`, else the cutoff) | Outcome closure |
| Question reroute; question expiry; routing candidates; independent moves | 14 days; 3 moves or 60 days; 3; 3 | Question age |
| Session record retention; capture audit | 7 days; 5 sessions per month | Capture coverage |
| Failed-attempt minimum; dead-end age; method-case members; fact speakers for a heuristic | 2 cases; 2 years; 3 (bound: at least 2); 2 (bound: at least 2) | Heuristic density, Dead-end revival |
| Binding window; stuck patterns (per scope); commit patterns (per scope); conclusion phrases; check suppression window; check frequency; also-working window; injected text | 7 days; one error 3 times, 4 turns without a passing check, one draft revised 3 times unaccepted, 4 turns without a new hypothesis, 3 hypotheses dropped, one plan restated 3 times, or 2 proposals rejected in a row; merge, push, deploy, release, launch, ramp, ship, enable, restart, fail over, roll back, delete, drop, scale, migrate, send, file, publish, present, hand off, sign, approve, order, submit, close, issue, tender, award, certify; shipped list per language; 5 turns, at the similarity calibrated per ranking model; once per 3 turns and per conclusion; 24 hours; the shipped text (reference) | Stuck resolution, Push yield |
| Record half-life (per situation, C9); half-life factor; time stratum (per situation); bias share; crossover share; concentration share; large-effect bound | max(365 days, factor × max(recurrence interval, outcome horizon)) (bound: at least 365 days); 4; max(half a year, two recurrence intervals) (bound: at least half a year); 80%; 10%; 30% (bound: at most 50%); relative risk 0.2 (bound: at most 0.5) | Record calibration |
| Monthly check (per scope or interest area) | Monthly | Champion stability |
| Sufficiency sample; sufficiency members; sufficiency interval ratio | 8 effective items per side (bound: at least 4); 2 members (bound: at least 2 when active members are 2 or more); 4 | Champion coverage |
| Most-used share | Top 1% monthly | Anchoring |
| Heuristics shown; least-shown window; exploration exposures | 3 with a `more` pointer; 90 days; 20 deliveries | Challenger exposure |
| False-discovery level | 10% | Record calibration, Decidable share |
| Warning trend ratio; trend recurrence; anomaly share; unknown-key share | Above 2 (heuristic trends, from 3 distinct independent assessors; also recurrence against a target's own earlier rate, C8 Own rate; bound: at most 3); 3 separate occurrences on one target within max(365 days, three recurrence intervals; for an instance target, also three times the median interval between readings on the warned key, same-bin readings included) (instance, group, and subject trends; bound: at most 5); 60%; 30% | Warning trends |
| Alias similarity | Per similarity model, so that 90% of tuning-set alias proposals above it are judged the same subject | Duplicate rate |
| Acknowledged instances; private routing threshold | 3 distinct instances (bound: at most 5); 10 routings a month | Floor load, Safety findings |
| Default warning severity (dissent, loosened criteria without a guarded warning, counted repeat exceptions, X11); floor-exposure threshold; floor lines | `minor`; 50 floor sessions (bound: at most 200); 7 per moment (bound: at least 3) | Floor load |
| Condition-test level; condition-test minimum | 5% after correction; 5 failures or 5 warnings | Decline rate |
| Forester wait; forester candidates; holder share; holder set size; holder-set question rate; routed-load cap | 30 days; 3; 80%; 3; 5 a week; 25% of the scope's open routed items | Routed-load concentration, Holder-set share |
| Interruption budget; busy window; urgent reroute; inline markers | 1 push per hour per session, 10 per member per week; 5 turns; 1 hour; 2 | Push yield |
| Check results; seen-before sessions; aggregate minimum; confidential reporting multiple (A3) | 2; 3 sessions; 3 scopes and 3 members, 3 events for a month (bound: never lower); 3× the member minimum | Critical-failure memory |
| Undisputed period (per scope or interest area); objection lapse; differing shown | 14 days (bound: at least 7 days); max(180 days, twice the outcome horizon); 4 | Disagreement load |
| Rerank depth; relevance threshold; tier width | Top 50 fused results, reordered to 10; per ranking model, with a fusion-score fallback; 0.1 of the ranking-model score | Find success |
| Record age (per subject kind); unreachable to aged | 180 days for facts and sources (`never` allowed); 90 days | Cause share |
| Refetch schedule (per source and kind) | weekly, daily for urgent sources | Page lag |
| Situation page members | 3 members | Page lag |
| Delivered-to window (per situation); write-batch window | max(90 days, one recurrence interval); one session or one service, within 1 hour | Propagation reach |
| Hide escalation; removal confirmation timeout; removal batch schedule; masked-cell warning; author-only hand-off; quarantine age | 3 days; 2 days; daily, except personal data and credentials at once; 20%; 30 days (bound: at least 14 days); 7 days (bound: at most 30 days) | Safety findings |
| Bulk personal-data threshold; generalization rules (per scope); safety sample schedule | 10 items; date to month, location to region, age and figures to bands; monthly | Safety misses |
| Short id length | 6 characters | Operational |
| Small reference deployment; large reference deployment | one CPU-only server (8 cores, 32 GB), 12 active members, 30,000 records, shipped models; 50 members with up to 4 concurrent sessions each, 100,000 cases with up to 500 attempts each, 5 years of log, hosted or GPU models | Hook health, Durability |
| Problem statement length; general-heuristic fields; set-aside members; outcome horizon minimum; capture-quality schedule | 1–3 sentences; 2; 2; 10 closed attempts; monthly | Capture quality |
| Uptake share | 50% of the situation's new cases | Uptake lag |
| Interval level; critical-failure bound level; condition-test removal level; inconsistency level; very-low weight factor | 90%; 95% (gives about 3/n with no failures); p > 0.5; 10%; 0.5 | Grounding |
| Champion evidence age; undisputed period cap | 365 days; twice the undisputed period (bound: at least the undisputed period) | Disagreement load |
| Private move interval; inherent-risk question interval; safety sample schedule | weekly; monthly; monthly | Safety findings, Question age |

Method examples: reference.


## A2. Acceptance properties

**Basis:** every mechanism.

Each property must hold for every input of its class. It is checked on a scripted deployment with recorded work-event inputs, over the fixtures in the reference, without a live model unless a fixture says so; a fixture states its own numbers. A new fixture may join any property; a property passes only when all its fixtures pass. An extension's properties are in its module (Part II); every property here passes with every extension off.

### A2.1 I1 — the log is the truth

- **I1-a Rebuild.** A full build from the log alone reproduces every derived item: records, links, causes (from `signal` events too), deliveries, trials, answers, and settings. Model text is reproduced or marked with its model version.
- **I1-b Replaceable models.** Reverting one model version's output changes no member event; the current model version rebuilds what it wrote.
- **I1-c Gated generations.** A build whose model change exceeds a switch limit fails. A forester's acceptance with a reason applies it, and every changed record shows the cause.
- **I1-d One base, two writers.** Changes from one base to one attribute or one passage are differing at that point.

### A2.2 I2 — every record shows where it came from

- **I2-a Produced and derived.** Every attempt that edits a passage gets a `basis` (`produced`) link from that passage. Every result an agent derives rests on a `member` source that holds the derivation.
- **I2-b Time order.** Only Mycelium's time and a declared time source can make a reason `stated` as before its outcome; every other time source gives `inferred`.

### A2.3 I3 — support is evidence, not headcount

- **I3-a Ceiling.** No observational record shows a figure above `low` (`moderate` for a large consistent effect, C9) or becomes champion, whatever its uses and members. An insufficient record shows only its basis, warning count, and critical-failure bound.
- **I3-b Speakers.** Support counts distinct independent speakers: one source naming many members is one speaker, lowered for concentration; many records of one remark make no heuristic; distinct authors in one shared stream count as distinct speakers.
- **I3-c Echoes.** A member who repeats what they were given adds an echo; a member who observes otherwise adds independent evidence.
- **I3-d Kind of evidence.** A higher kind of evidence decides against a lower one, whatever the count on the lower side: an assessment with higher validity decides, and the other version gains one adverse item; one direct observation with a supporting check resolves a pair against any number of confirmations.
- **I3-e Assessors.** When assessors of one outcome disagree, a checked assessment outweighs self-reports and the disagreement shows with its count; when `done_when` names an assessor whose assessment conflicts, the outcomes are differing; two assessors years apart who disagree give a later state, not a differing outcome.
- **I3-f Pooled restricted support.** Support pooled from restricted scopes shows outside them only at the aggregate minimum, marked as pooled and at most `low`; below it, nothing shows.
- **I3-g Effective sample.** On simulated logs with known within-group correlation and age weights, the sufficiency interval covers the truth at the *interval level*, and one correlated group alone never makes a record sufficient, except where C9 Sufficiency waives the members.

### A2.4 I4 — validity is derived

- **I4-a Copies.** A change to a parent passage marks every copy at that passage and sends each copy owner one notice; keeping a local variant clears the marker and never makes the parent differing.
- **I4-b Reach.** A change of validity reaches every dependent along `basis`, at any depth, as one cause; a range change marks each dependent that may rest on it; each closed case that C7 Closed work asks gets one re-check question, and an answer clears only its own case.
- **I4-c Precision.** A change signal marks only records that rest on what changed.
- **I4-d Breaking marks.** A `breaking` mark with evidence starts a new stratum: uses before it carry no weight until uses after it suffice alone, later uses restore no earlier weight, and dead ends before it become retry candidates. A mark without evidence stays pending and changes no weight.
- **I4-e Shared causes.** A cause with a start date marks only cases from that date, with one notice per owner and one question per case.

### A2.5 I5 — visibility is the narrowest input

- **I5-a Reads.** A member reads only the scopes they can read. An exclusion holds through groups and child scopes; a session under a child scope gets its parent scope's facts. A restricted repeat of a team case changes no team figure, champion, page, or export.
- **I5-b Writes narrow.** A write that cites or matches restricted text its author can read takes that scope, with the reason. A non-reader's matching write keeps its scope and tells its author nothing; the restricted scope's readers get one notice, which names the matching record only to those who read the write's scope. Text that is also visible in the wider scope does not narrow a write. A subject made in a restricted session stays out of the team registry.
- **I5-c Restricted harm.** Outside its scope, a restricted critical failure shows only as a content-free count at the aggregate minimum. An outside session on its subject causes one question to its readers that names the outside member only to readers who can read the outside session's scope.
- **I5-d Sessions narrow.** Opening a restricted record narrows later captures; reading a second restricted scope narrows to their intersection; no restricted record arrives automatically in a wider session.
- **I5-e Releases.** A release passes the release check, carries `from-restricted`, and counts once with its original; a change to the original asks its readers once and sets no cause outside.
- **I5-g Later narrowing.** When an input is hidden after its dependents were written, they keep their scope and show `basis not visible`.
- **I5-h Restricted-only work.** A deployment whose cases are all in one restricted scope still finds its situations, visible to that scope's readers.
- **I5-i Audience.** A service that serves several people delivers only records that every member of its audience can read; with any unidentified person present, only team-visible records.

### A2.6 I6 — recall offers and never directs

- **I6-a Start.** For every agent tool, work start delivers state and changes, and no heuristic outside orientation.
- **I6-b Parallel sessions.** An outcome recorded in one session on a case that a parallel session touches reaches that session once, as a `switch` line at its next member request.
- **I6-c Procedures.** A matching procedure arrives with its steps and validity line, also with no cited uses; procedures that differ show by step.
- **I6-d Never blocks.** A committing action with a relevant result gets the result injected before it runs; no action is ever denied.
- **I6-e Stuck and conclusion.** One stuck sign gives one stuck offer; a relevant result at a conclusion arrives by `continue_with`.
- **I6-f Repetition.** An unchanged non-warning folds into `seen-before` after the *seen-before sessions*.
- **I6-g Floor.** An applicable serious or critical warning shows whatever the count of successes, exposures, or dependents: warnings this reader has not seen come first, the others named; an acknowledgment holds only for that instance and action kind; one harm over many dependents is one floor item.
- **I6-h Condition return.** A delivered heuristic returns once by a `condition` notice when a condition on the watch list appears; never for a condition every case carries.
- **I6-i `auto` conditions.** No unmarked `auto` condition causes `may-not-apply`.
- **I6-j Floor before areas.** Under `hold`, an out-of-area record comes only by explicit search, but an applicable out-of-area critical failure still reaches the check moment (I6 before I10).
- **I6-k Severity.** No evidence about the extent of other occurrences, or about success, moves an applicable warning out of the floor; disagreeing severities take the least favourable for the floor.

### A2.7 I7 — use teaches

- **I7-a Exploration.** In a non-decidable situation, repeated stuck calls show every qualifying candidate; no candidate is starved.
- **I7-b Learned conditions.** A condition concentrated in warned uses becomes a `not_when` entry; failures that start at one date become a dated entry.
- **I7-c Anomalies.** Uses that mostly carry an anomaly add no support weight, show their warning count, and never make a champion; a new case without the usual anomaly is marked and asked about once.
- **I7-h Learning from cases (live model).** Hidden heuristics of every kind are recovered from synthetic cases across fields with no shared content words; each analogue pair gets one `general` heuristic; a negative control pair stays unlinked.
- **I7-i Outside situations.** A stuck call outside every situation gets the relevant heuristics of its fields, ranked, with no champion.

### A2.8 I8 — nobody has to curate

- **I8-a Case boundaries.** Attempts on separate problems make separate cases; reports of one run make one case.
- **I8-b Bulk sources.** A log yields cases only from its problem-and-fix entries, and facts from routine entries only when cited or for declared state keys (a reading of an unchanged value extends the current fact); an entry whose value leaves the problem range closes the attempt; routine in-range entries do not block the absence check; routine success yields items, not dead ends; silence infers no outcome; a test of a claim without an intervention sets the hypothesis aside, and a dead end needs failed interventions; a recount of held cases adds no item or recurrence.
- **I8-c Recurrence.** A new case on an instance after an achieved fix, within the Recurrence bound (C11), is a `counter` (`recurrence`) with one question.
- **I8-d No write is lost to a hold.** While a check is down, writes wait in the quarantine, readable as `checking`; past the *quarantine age* they return to their author. Without a writing model, text in another language waits for translation and is then stored in English only.
- **I8-e Gates hold display.** A heuristic that states an approach passes with its kind; a vague one is held from display, never refused entry; a procedure from a source with no stated reasons enters with its reasons marked as not stated.
- **I8-f Ignoring is safe.** No ignored question, of any kind, makes anything wait.

### A2.9 I9 — no one person changes the team's view alone

- **I9-a Another's record.** A correction or retirement of another's record without evidence, a `wrong` on a procedure included, shows beside the original, which keeps its rank, and asks one member. Unopposed, it takes full effect after the undisputed period, unless the target rests on a first-hand observation or re-check; disputed with evidence, both are differing; disputed without, the dispute lapses. With a re-check it takes full effect at once.
- **I9-b One's own record.** An author narrowing their own record takes effect at once; retiring their own record that others used, or calling their own failed case `wrong`, follows the evidence rule; a corrector's own confirmation adds no support.
- **I9-c Reverts.** A forester's revert of a damaged import takes records out of recall at once and is reversible; their critical harm stays in the floor, marked as from a reverted batch.
- **I9-d Standing needs evidence.** A self-reported critical harm enters the floor at once and pauses the holdout, but the champion keeps its role until `checked` evidence or a second assessor. A stated hazard withdrawn by its speaker stays, marked. A re-recorded outcome without evidence sits beside the earlier one. Interpretive disputes stay differing until a side is withdrawn or other evidence decides; an interpretive objection to a re-checked fact stays pending.
- **I9-e State over time.** A later value of a changing state replaces the earlier one at once, credit `kept`; two claims about one time stay differing, and no forester settles them; ratings by different assessors stand side by side within one time window, and where one value is needed the latest window's ratings are used.
- **I9-f Conflicting decisions.** Adopted decisions that exclude each other both show as in force and as differing, with one question to both owners.
- **I9-g Concerns.** An open concern stays open after an achieved outcome or an owner's answer.
- **I9-h Departure.** A leaving member's owned subjects are listed for reassignment, and each new owner gets subject orientation. Their private records never become readable by another; author-only ones are removed after the hand-off, others are kept and counted.

### A2.10 I10 — focus by interest

- **I10-a Situations.** Grouping finds labelled hidden situations across fields at the fixture's precision and recall, stable across one model change.
- **I10-b Default areas.** Situations with cases from at least the *area member threshold* are in the default areas; others are not.
- **I10-c Fields.** A case from another field with no `general` heuristic, not on a touched instance, never arrives at an automatic moment, and explicit search finds it; another field's critical warning on a touched instance reaches the floor.

### A2.11 Safety

- **R16-a Credentials.** A credential in any write is masked on the client and at intake, and is never sent outside.
- **R16-b Outsiders.** No outsider's name is stored. Text that identifies an outsider goes to the author's private scope in the default scope.
- **R16-c Data boundary.** No unmasked outsider name or special-category text reaches an outside model; no record of a scope reaches a provider its boundary excludes.
- **R16-d Complete removal.** Removed content is in no event, index, derived item, cache, or export, and later text that carries it waits in the quarantine for a reader's client.
- **R16-e Hides.** A hide as a leak takes effect at once and keeps the warning, without text, in its readers' floor; a hide of another's used case for another reason is a pending correction; a confirmed move returns a record to its scope and removes derived copies outside.
- **R16-f Masking keeps findings.** A judgment of a person is rewritten while a finding about work is kept; masking acts on the column, not on the whole table.

### A2.12 Durability and scale

- **R15-a Generations.** During a build, recall serves the current generation; a rollback restores the previous one.
- **R15-b Degraded service.** Without an embedding model, lexical results arrive within the *hook time limit*, marked `degraded-search`.
- **R15-c Offline writes.** A capture queued offline is logged exactly once at reconnection; no write is lost. Bulk synchronization classifies every capture at its `made_at` within the *build deadline* and reports intake lag per member.
- **R1-a Access with no holder.** A restricted scope whose last reader leaves with no custodian freezes; a reader named by two independent active members takes effect after one notice to all and the undisputed period, or lapses on a dispute that a second independent active member supports, and then gets the scope's serious warnings.
- **R2-a Scale.** At the reference size, search latency, notices (one per owner), questions (one per owner and case), find success against a small subset, correction survival, question load, and complete removal stay within the fixture's bounds; install verification reports full-build cost for the largest fixture.


## A3. Field extensions

**Basis:** M15; I10.

Needs of one field or a few; an entry is not a commitment. A team-level forester turns one on per interest area or per scope (`config.changed`, I10). Safety and visibility extensions (the ones Part II Turning off lists) are per scope only: the scope's own foresters who read it turn them on (for matching, both scopes' foresters); they are never turned off once on (Part II Turning off). A team-level forester turns any other extension off at the level it was turned on. One acting at intake follows the record's scope. One acting later covers an item that was in area (C4 In area) at its observed time; area changes affect only later items (C6 Settings apply forward). For an item in several areas whose extension settings differ, the value toward safety applies, else the extension's default (off, or the default of its tunable). Every extension keeps every invariant and requirement, using only the exceptions PRINCIPLES names.

| Need | Fields |
|---|---|
| Finding people with experience of a situation, with load limits; benchmarks across clients | Consulting, professional services |
| Effective dates; status `not in force`; consolidation as `replaces` (credit `kept`); an amendment as a new source version, or `replaces` on the amended passages; consolidated text; territory versions; transitional rules | Legislation, regulation, standards |
| Locators resolved by legal time at the citing document's date; typed references as existing links (`follows` as `basis` (`cites`), `overrules` as `replaces` with credit by its stated reach and a `from` time, a stay as a contingency on the affected situation, a `judicial` reading of `governing` (X7) with a court level, `distinguishes` as a `not_when` or condition with a `cites` reason); citation stances | Law, patents, standards |
| Outside classification schemes with versioned code mappings; a scheme change as a classification subject's revision (C4) | Patents, medical coding, statistics |
| Cross-scope matching of one `person` beyond warnings, as C4 (both foresters on; only the asking scope's readers learn of it; counts only at the aggregate minimum); special-category data kept exact in a person scope that turns it on (the I5 exception); such a scope defaults to `high-stakes` (a forester can turn it off with a reason, [team decision]) | Social services, health, law (criminal, employment, personal injury) |
| Disagreeing instrument versions; an adverse instrument losing direct-observation weight | Laboratory measurement |
| Exclusion sets on `ordered` kinds | Serial-numbered equipment, software security |
| Performer profiles as conditions | Sports, performing arts |
| Ecosystem version comparators; severe advisories through the floor (C12), never as a ranking | Software security |
| Evidence synthesis, further (beyond X6; also machine learning: benchmarks, ablations, sweeps): per-study bias ratings; PICO conditions; precision-weighted pooling of `estimate` facts with heterogeneity; registered-against-reported outcomes from a declared registry time source; certainty per outcome; interrupted-time-series analysis beyond a unit comparison (X6) | Clinical medicine, public health, social policy, education, ecology, online experimentation, product analytics |
| `design: deductive` for facts whose basis is a derivation: a formal checker's output counts as direct observation (C5), an informal proof as interpretive; marked `proved-checked` with the checker, or `proved-informal` with the count of independent re-checks | Mathematics, theoretical computer science, formal methods, theoretical physics |
| Factorial and response-surface designs inside one controlled case, analysed by main effect (with interactions) as `estimate` facts linked `supports` to the heuristic that names each factor, under the same ceilings | Manufacturing, process engineering, agronomy, machine learning |
| Plausibility checks on manual monitoring records (digit preference, low variance, batched entry times): one review question to the scope's foresters; never a judgment of a person | Food safety, clinical monitoring, field inspection |
| A reusable holdout that rations results from a `held-out` instrument version (X6) | Machine learning, forecasting |
| An assessment that names the assessment it sets aside; with a `checked` citation, a correction with evidence of the earlier one | Appeals, audit, peer review |
| Ranked comparisons over variants in one controlled case, analysed as pairwise comparisons clustered by assessor | Participatory variety selection, sensory panels |
| A declared confidential reporting scope with at least the *confidential reporting multiple* of the member minimum in contributors substitutes for the scope part of the aggregate minimum; the minimum is not lowered | Safety reporting |
| Location conditions recorded as zone subjects, so generalization keeps applicability | Agriculture (agro-ecological zones) |
| State-transition and survival estimates per instance group and condition group, from state-over-time series with censoring, written as `estimate` facts under the observational ceiling | Asset management (bridges, pavements, pipes, utilities, fleets) |
| A scope the server cannot decrypt (readers' clients build its derived data and run its checks; the server keeps ciphertext and envelopes; no counts); checks for source-identifying artifacts; minimized timing metadata | Journalism, whistleblowing |

## A4. Measures and model quality

**Basis:** M14; I7, I1, I9.

**Core measures** run continuously on the foresters' view; find success, stuck resolution, capture quality, and safety misses also per conversation language.

| Core measure | Definition | Criterion |
|---|---|---|
| Find success | Share of held-out test queries whose expected record (following `replaces` and `same`) is within the *find success depth*; documents and passages apart | Findability |
| Missed-search rate | Share of explicit searches `missed`, and of missed groups later closed | Findability |
| Stuck resolution | Share of stuck calls whose bound case later records an achieved or partial outcome, with the delay | Reuse |
| Reuse share | Share of closed cases whose attempt uses, cites, or repeats an existing record, by moment, against `new` ones | Reuse |
| Known-failure repeat rate | Share of failed cases classed Repeat or Counter of an earlier failure, or matching a dead end or a warned case, split by whether that record was `delivered` in the session, and by moment | Reuse |
| Outcome closure | Share of attempts closed within their cutoff, by recorded or inferred and by outcome | Currency and correction |
| Correction survival | Share of corrections still applied in a new generation | Currency and correction |
| Propagation reach | Share of recalls after a trigger that carried the inherited cause; open inherited causes by age; reach per trigger; expired re-check questions | Currency and correction |
| Question age | Median and 90th-percentile age of open questions, per kind | Many people, many years |
| Push yield | Per moment, reason, kind, and pattern: deliveries per session-hour and member-week; share to busy sessions; share followed by any recorded action; repeat exposure; below its band, top records and patterns | Low friction |
| Capture coverage | Per tool, adapter or tool protocol, kind, and tenure: active sessions with a capture or review draft; audit counts; work items and decisions that appear first in documents or the board with no session capture | Low friction |
| Intake lag | Time from write to visibility and classification | Low friction |
| Hook health | Error, timeout, and offline rates per tool and round-trip band; share of `check-incomplete` and `degraded-search`; members with failing adapters are told | Agent-agnosticism |
| Challenger exposure | Stuck deliveries with the least-shown slot or "Untried ideas" filled; new approaches per situation since its latest `breaking` mark | Improves with use |
| Floor load | Floor items per session by instance and situation; share of floor deliveries followed by any action | Low friction, Advisory |
| Routed-load concentration | Share of questions, notices, and re-check work that the busiest members receive | Many people, many years |
| Safety misses | Share of a sample, on the *safety sample schedule*, in which a stricter judge finds unmasked personal data or credentials | Safety |
| Capture quality | A judge score, on the *capture-quality schedule*, of sampled records against the drafting rules | Low friction, Advisory |

**Diagnostic measures** run on request, or continuously once a forester sets a band; champion-module ones only while a situation is decidable.

| Criterion | Diagnostic measures |
|---|---|
| Findability | Rediscovery rate; held release rate; value-weight agreement; out-of-area finds |
| Reuse | Cross-field use of `general` heuristics |
| Grounding | Records with a basis, conditions beyond `auto`, stated reasons, `checked` outcomes, citation checks and failures |
| Improves with use | Heuristic density; dead-end revival; uptake lag (per situation: time from a heuristic's first support weight +1 until it is used in the *uptake share* of the situation's new cases; share of situations not yet there); decline rate (never optimized); critical-failure memory; champion stability; record calibration; looks per `held-out` instrument version (X6); champion coverage, decidable share, anchoring, randomization health |
| Currency and correction | Recurrence rate; warning trends (raised and explained); cause share; page lag |
| Continuity | Start completeness; hand-over completeness |
| Many people, many years | Duplicate rate; holder-set share; disagreement load (share of in-area records with pending corrections); quality by size; hides, corrections, and disputes against each principal's norm (per principal only to that principal); model cost |
| Durability | Canary agreement; restore test; rebuild fidelity |
| Safety | Hide release rate; masked values; removals; rescan matches; open hide and purge age |

- **Test queries** come from bound deliveries and searches whose record an achieved or partial case then cited, and from missed searches later closed; members add more. All are split at random into a tuning set (*tuning split*) and a held-out set for switches and setting changes.
- **Canary** (I1). A labelled set of judge decisions (classifications, splits, merges, safety checks, translations per language, inferred outcomes), kept apart from records, runs on the *canary schedule*: a shipped fixture and items members label with consent, restricted ones only by their readers. Below the *canary band*, that judge's `identical` and `one event` links, splits, and inferred outcomes become questions, and foresters are told. A translator below the band sets `changed: translation` and asks each active speaker, in their language, whether the English says what they meant; an answer that it differs is a material revision.

## A5. Output content

**Basis:** M10; I6.

- **Stuck output** has these sections:
  - a header saying these are suggestions from past work, not instructions;
  - the **champion**, only in a decidable situation: the critical-failure bound when its basis holds a serious failure; its `why`, `not_when`, failed conditions, the count of counter and differing records against it, and a pointer;
  - **challengers**, up to the *heuristics shown*: kind and statement; when sufficient, its failure rate against other attempts here (with conditions when the figure holds only in them), interval, uses, members, and the observational mark, else `no-record-yet`; causes; `why`, origin and speaker, a basis case, a case against it, and the conditions it succeeded and failed in;
  - **untried ideas** with speaker, and **retry candidates** with the cause that reopened them;
  - **from other fields**: each `general` heuristic with one example and its field;
  - **also working on this now**: the count of other readable sessions in the situation within the *also-working window*, their members only to readers of their scopes.
- **Markers.** Serious and critical warnings always show inline and never fold into a `more` count, except the C8 floor fold. Besides them, at most the *inline markers* show inline: causes by severity, then applicability, `outside-areas`, and support markers; the rest fold into a count.
- **Injected text** comes at work start, or as tool-server instructions (*injected text*). It never contains heuristics. It tells the agent: capture what it tries, with the sources relied on, and later the outcome; correct or flag what proves wrong; when stuck, run the stuck command; before a conclusion the member is likely to act on, run the check command; pass observed conditions; always relay a warning that applies, and record the acknowledgment and its reason when the member goes ahead; at the end of a unit of work, record next items and hand-offs; name outsiders by role, not by name.

## A6. Commands

**Basis:** M13; I5.

Every writing command can restrict its write to named members or groups. Input takes short ids: the last *short id length* characters, extended until unique among ids the user can see (operational, I5). Each command writes the events of the rule it names; syntax is in the reference.

| Who | Commands |
|---|---|
| Agents and members (also tool protocol) | stuck; check; ask; find; show; capture; note (an `observation` or `claim`, a `concern`, a `hazard`, an untried idea, or a contingency with its condition); state; outcome (one assessment, or one shared outcome over several cases); trial; revise (own record; another's is a correction); link; decide (decision status with a reason); correct, retire, restore; support (confirm or dispute with reason and evidence); flag; answer; recheck (optionally against one citation); add; subject (subjects, owners, containment, `breaking` marks with a reason); hide; end, item, project |
| Members | restrict, scope move; area (list, add, remove, describe interest areas); scope (create, use); about-me; pause, resume; holdout (own setting); device (list, revoke); role (access-role naming); eval (add, remove, supersede test queries); export; install, uninstall |
| Foresters | release (a gate release, or a release record, optionally sanitized); scope (link, unlink, set, name members and foresters; settings for keys, generalization, time sources, export, high-stakes, `person`, `match`, custodian); remove (a record, content, a person, an author's writes in a window, a source, with or without a deny rule, or a scope); person search; revert (an author's, a session's, a source's, or a judge's writes, or undo a revert); member (add, remove, change, pseudonymize); config (read and set tunables); build (build, roll back, accept a generation) |
