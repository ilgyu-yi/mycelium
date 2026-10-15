# SPEC — single source of truth for behavior

This document is the single source of truth for how Mycelium behaves. Every data shape, state rule, command, integration contract, and acceptance criterion is defined here. `MISSION.md` sets direction; this SPEC defines behavior. It refers to MISSION success criteria by name (shown in bold) and never restates them.

## Table of contents

<!-- TOC START — line numbers for targeted reads; update on every edit -->
| Section | Title | Line |
|---|---|---|
| §0 | Scope and conventions | 72 |
| &nbsp;&nbsp;§0.1 | What the SPEC governs | 76 |
| &nbsp;&nbsp;§0.2 | Relation to MISSION | 80 |
| &nbsp;&nbsp;§0.3 | How to read this document | 84 |
| &nbsp;&nbsp;§0.4 | Normative words | 88 |
| &nbsp;&nbsp;§0.5 | Terms | 92 |
| §1 | Knowledge model | 113 |
| &nbsp;&nbsp;§1.1 | Capture, synthesis, and recall | 117 |
| &nbsp;&nbsp;§1.2 | Cases | 130 |
| &nbsp;&nbsp;§1.3 | Principles | 144 |
| &nbsp;&nbsp;§1.4 | Links | 154 |
| &nbsp;&nbsp;§1.5 | Domains and the glossary | 168 |
| &nbsp;&nbsp;§1.6 | States | 182 |
| &nbsp;&nbsp;§1.7 | What an agent can see | 205 |
| §2 | Event log | 222 |
| &nbsp;&nbsp;§2.1 | Layout | 226 |
| &nbsp;&nbsp;§2.2 | Fields on every event | 251 |
| &nbsp;&nbsp;§2.3 | Event types | 266 |
| &nbsp;&nbsp;§2.4 | Ordering and folding | 289 |
| &nbsp;&nbsp;§2.5 | Writing | 299 |
| &nbsp;&nbsp;§2.6 | Payloads | 309 |
| §3 | Search and recall | 424 |
| &nbsp;&nbsp;§3.1 | Case search | 428 |
| &nbsp;&nbsp;§3.2 | The principle list | 442 |
| &nbsp;&nbsp;§3.3 | How the two connect | 453 |
| &nbsp;&nbsp;§3.4 | `myc ask` | 459 |
| §4 | Capture | 475 |
| &nbsp;&nbsp;§4.1 | Decision points | 479 |
| &nbsp;&nbsp;§4.2 | Questions | 483 |
| &nbsp;&nbsp;§4.3 | Answers | 493 |
| &nbsp;&nbsp;§4.4 | Pacing | 497 |
| §5 | Synthesis | 507 |
| &nbsp;&nbsp;§5.1 | When | 511 |
| &nbsp;&nbsp;§5.2 | What it produces | 516 |
| &nbsp;&nbsp;§5.3 | Model contract | 536 |
| &nbsp;&nbsp;§5.4 | Review | 547 |
| §6 | Agent integration | 551 |
| &nbsp;&nbsp;§6.1 | Common input and output | 555 |
| &nbsp;&nbsp;§6.2 | Mapping to each tool | 570 |
| &nbsp;&nbsp;§6.3 | Hook command | 585 |
| &nbsp;&nbsp;§6.4 | Rules for every hook run | 594 |
| &nbsp;&nbsp;§6.5 | Injected text | 610 |
| &nbsp;&nbsp;§6.6 | Automatic case search | 633 |
| &nbsp;&nbsp;§6.7 | Conclusion nudge | 640 |
| §7 | Background job | 649 |
| &nbsp;&nbsp;§7.1 | Running | 653 |
| &nbsp;&nbsp;§7.2 | Background review | 660 |
| &nbsp;&nbsp;§7.3 | Headless runs | 671 |
| §8 | Commands | 678 |
| &nbsp;&nbsp;§8.1 | For the user | 682 |
| &nbsp;&nbsp;§8.2 | For the agent | 697 |
| &nbsp;&nbsp;§8.3 | Internal | 714 |
| §9 | Safety | 718 |
| &nbsp;&nbsp;§9.1 | When and what | 722 |
| &nbsp;&nbsp;§9.2 | Checks and actions | 729 |
| &nbsp;&nbsp;§9.3 | Purge | 742 |
| §10 | Audit and notification | 754 |
| §11 | Measurement | 773 |
| §12 | Tunable settings | 797 |
| §13 | Acceptance criteria | 813 |
<!-- TOC END -->

## 0. Scope and conventions

This section covers what the SPEC governs, how it relates to `MISSION.md`, how to read it, and the terms it uses.

### 0.1 What the SPEC governs

The SPEC governs one user's knowledge base: the append-only event log; the two kinds of node it holds, cases and principles, and the links between them; the indexes and views built from the log; the `myc` command; its integration with Claude Code, Codex, and pi; the background job that reviews sessions and synthesizes principles; the safety checks on everything written; and the measurements used to tune its settings. Governing knowledge between people is out of scope (`MISSION.md`, § "Explicitly NOT goals").

### 0.2 Relation to MISSION

Every requirement here derives from a criterion in `MISSION.md` § "Success looks like" — **Reuse**, **Advisory recall**, **Grounding**, **Synthesis**, **Low friction**, **Continuity**, **Agent-agnosticism**, **Durability**, **Safety** — or from § "Explicitly NOT goals". If the two documents disagree, MISSION wins on direction and this SPEC is amended.

### 0.3 How to read this document

Sections are numbered `## N.` and `### N.M` and listed in the table of contents with their line numbers, so a reader can jump straight to one; the table is updated in the same commit as every edit. Each top-level section opens with a sentence saying what it covers. Tables that define fields, states, or mappings are normative. A value marked **tunable** is a setting that the measurements in §11 may justify changing; §12 lists all of them, and changing one amends this SPEC. Korean strings in this SPEC are data — default values or test inputs for Korean text handling — never prose.

### 0.4 Normative words

**Must** and **never** state requirements an implementation either meets or breaks. A rule marked **advisory** describes what an agent, a model call, or the user is expected to do; nothing checks it, and `myc` behaves correctly whether or not it is followed. Every other rule is enforced by `myc` itself, by its lint, or by its audit (§10).

### 0.5 Terms

| Term | Meaning |
|---|---|
| **node** | A case or a principle. A node's id is the `id` of the event that created it |
| **candidate** | A node with no answer yet (state `tentative`) |
| **proposal** | A change to the principle structure that synthesis suggests and the user must confirm (§5.2) |
| **link** | A typed connection from a case to a principle, or between two principles (§1.4) |
| **answer** | The user's response to a question, a node, or a proposal, recorded as an `answer` event (§4.3) |
| **log** | The append-only event files under `events/` (§2.1) |
| **decision point** | A point where an agent is stuck, settles a definition, interpretation, or direction, reaches a conclusion, or drops a hypothesis |
| **recall** | An agent noticing a principle in the principle list and opening it with `myc show` |
| **apply** | An agent weighing a recalled principle against the problem at hand. The result — `fit`, `misfit`, or `declined` — is recorded as an `applied` link |
| **question** | The multiple-choice question that `myc capture` returns for the agent to add to its reply (§4.2) |
| **tally** | A principle's count of `applied` links by outcome: `fit`, `misfit`, and `declined` |
| **background job** | The process that runs every 10 minutes for background review, synthesis, sync, and audit (§7) |
| **headless run** | A one-shot, non-interactive agent call that `myc` makes for background review or synthesis, with `MYC_HEADLESS=1` set (§7.3) |
| **autonomous run** | An agent session that runs without a user present, started with `MYC_ORIGIN=autonomous` |
| **position** | The number of lines in a session's transcript file at a given moment (§6.1) |
| **review position** | How far background review has read a session's transcript (§7.2) |

## 1. Knowledge model

This section covers the three activities that produce and use knowledge, the two kinds of node and the links between them, domains, states, and what an agent can see.

### 1.1 Capture, synthesis, and recall

| Activity | When | Result |
|---|---|---|
| **Capture** | During a session, at a decision point (§4) | A **case**: the problem as it looked, what was checked first, the facts and conditions involved, and how it was resolved |
| **Synthesis** | Outside any session, once cases accumulate (§5) | **Principles** that recur across cases, and links from cases to principles |
| **Recall** | At the next decision point (§3) | The agent recalls a principle, reads the facts and conditions of the cases behind it, and the result is recorded as an `applied` link. This is where **Grounding** pays off |

Two rules follow from **Advisory recall** and **Grounding**:

1. Nothing in `myc` acts on a principle. Principles are listed and shown; the agent decides whether to apply one.
2. A principle never contains team-specific facts. Facts live in cases and are reached through links, because a principle cut off from its cases loses the facts its advice depends on.

### 1.2 Cases

A case is the record of one decision point. Its fields are defined in §2.6. Rules:

- `first_checks` and `facts` carry most of a case's value. `myc` refuses a case with both empty, unless the user asked to save it (§4.2). A `definition` or `approach` case may have no `facts`.
- A quote longer than 500 characters keeps its first and last 240 characters, joined by `…`.
- A quote with `who: user` is checked against the prompts stored for its session (§6.4), ignoring `…` joins and masked spans. If it does not appear verbatim, `myc` drops that quote and keeps the case. If no stored prompts are available — the session id is null, per-device state was lost, or the case came from background review — the quote is kept with `status: unverified`.
- A `rejection` case must have `polarity: dead-end`.
- `myc` fills `related` with the last five candidates from the same session.
- The capturing agent (**advisory**):
  - writes `problem` as it looked before the cause was known, not in terms of the fix, so that later problems described the same way will match it;
  - writes `first_checks` without domain-specific terms where it can ("the target metric", not "the reward"), and keeps domain detail in `facts`;
  - looks through the session's commands, outputs, and code before deciding that `first_checks` and `facts` are empty.

### 1.3 Principles

A principle is a piece of problem-solving know-how written without domain-specific terms, so that it can carry over to other domains. Its fields are defined in §2.6: a core shared by all principles, and a `body` whose fields depend on `kind`. Rules:

- A principle from synthesis must have at least two `supports` links from cases that are `confirmed` or `chosen-only`. `myc` refuses one without them.
- A principle the user enters needs no supporting cases; it is shown with `[no cases yet]` until a case supports it.
- `myc` refuses a principle whose `first_checks`, `discriminate`, or `not_when` is empty. Whether those actions are concrete, rather than an attitude such as "doubt the result," is **advisory**.
- Lint reports a `name` or `when` that contains a domain name or a term listed under any domain in the glossary (§1.5), because the agent reads them to decide whether a principle fits a problem in another domain.
- Synthesis never puts team-specific facts or figures in a principle (**advisory**).

### 1.4 Links

| `rel` | From → to | Meaning | Written by |
|---|---|---|---|
| `supports` | case → principle | The principle was drawn from this case | synthesis, `myc link` |
| `applied` | case → principle | A recalled principle was weighed in this case; `outcome` is `fit`, `misfit`, or `declined` | `myc capture --applied`, synthesis, `myc link` |
| `refines` | principle → principle | The source is a narrower branch of the target | confirming a `refine` proposal |
| `absorbed-into` | principle → principle | The source has been folded into the target | confirming an `absorb` proposal |

- A link is identified by `from`, `to`, and `rel`. It holds from an `add` until a later `remove` with the same three values, whatever the state of either end. A later `add` of an `applied` link replaces its `outcome` and `note`. A link is not a node and has no state.
- Support counts and tallies include only links whose case is `confirmed` or `chosen-only`, the same cases synthesis reads (§5.3).
- When a principle is absorbed, its `supports` and `applied` links count toward the principle that absorbed it.
- `misfit` applications are what synthesis uses to narrow a principle's `not_when` (§5.2).

### 1.5 Domains and the glossary

`glossary.md` in the knowledge base root is edited by the user, never by `myc`. It has two sections that `myc` reads:

```
## Domains
- <domain>: <term>, <term>, ...

## Stop words
<word>, <word>, ...
```

A domain's terms are the domain-specific words that lint looks for in principles (§1.3). The domain values `myc` accepts are those in the glossary plus every domain named in a `project.created` event; any other value triggers a lint warning but is stored. Domain matching uses a case's `domains` field (§2.6); the event-level `domain` (§2.2) records the session's context only. A case with empty `domains` belongs to every domain. A session's **current domain** is the domain of the project linked to its working directory; a session with no linked project has none. Principles have no domain, because they are written to apply across domains.

### 1.6 States

Cases and principles go through the same states:

| From | Event | To |
|---|---|---|
| — | `case.candidate` or `principle.candidate` | `tentative` |
| any except `retired` | answer `confirm` | `confirmed` |
| any except `retired` | answer `chosen-only` | `chosen-only` |
| any except `retired` | answer `reject`, reason `unneeded` (the default) | `rejected` |
| any except `retired` | answer `reject`, reason `wrong` | `misinterpreted` for a case; `rejected` for a principle |
| any except `retired` | answer `same-as` (cases only) | `merged` |
| any | `retire` | `retired` |
| `retired` | `retire` with `restore` | the state before the matching `retire` |

- If a node has several answers, the last one in fold order (§2.4) wins. An answer to a `retired` node is stored and ignored.
- A `tentative` case leaves the question queue (§4.4) after 30 days (**tunable**) without an answer; it stays `tentative`.
- A case is **superseded** when the answer that set a `confirmed` case's state has a `replaces` field pointing to it. This is computed, never stored, and chains are followed to the last confirmed case. Only cases can be superseded.
- A principle is **absorbed** while it has an `absorbed-into` link whose target is `confirmed`.
- `myc` refuses a write that would create a cycle through `replaces`, `refines`, or `absorbed-into`. If such a cycle arrives from another device, the event that closes it in fold order is ignored and reported by lint. Any other event that breaks a rule in this section is also stored, ignored, and reported.

Proposals have three states: `tentative`, then `confirmed` or `rejected`. Their first answer is final; later answers are stored and ignored.

### 1.7 What an agent can see

| State | Case | Principle | Marker |
|---|---|---|---|
| `confirmed` | returned by search | listed | none |
| `chosen-only` | returned | listed | `[unconfirmed]` |
| `tentative` | not returned | listed if from synthesis | `[unconfirmed]` |
| `misinterpreted` (cases) | returned | — | `[misinterpreted]`; the wrong option is indexed, and the answer's `note` is attached as a fact |
| superseded (cases) | not returned; the replacing case is | — | `[replaces: <summary>]` on the replacement |
| absorbed (principles) | — | not listed; the absorbing principle is | none |
| `merged`, `rejected`, `retired`, purged, flagged (§9.2) | not returned | not listed | |

- A principle from synthesis is listed while still `tentative` so that synthesis reaches the agent even if the user never reviews it (**Low friction**). A `tentative` case is not returned, because an unreviewed case may contain an unchecked fact.
- A user-entered principle with no supporting case is marked `[no cases yet]`.
- A case with `polarity: dead-end` is marked `[don't try]` when it is shown in the current domain.
- A principle's line shows its tally: `[fit N · misfit M · declined K]`.

## 2. Event log

This section covers where the log lives, the fields every event has, the event types and their payloads, how events are ordered and combined into current state, and how they are written.

### 2.1 Layout

```
<knowledge base root>/                       a git repository
  events/<author>/<device>/<YYYY-MM>.jsonl   the log; append-only
  eval/<author>/<device>.jsonl               confirmed search test pairs (§11)
  projects/<project_id>/STATUS.md            project notes written by the user
  glossary.md                                domains and stop words (§1.5)
  .myc-rules                                 safety rules and allowances (§9)
  .myc/                                      rebuildable cache; never committed
    index.sqlite                             case index and folded state
    principles.txt                           the current principle list (§3.2)
~/.config/myc/
  bin/myc                                    the binary
  device.json                                author handle, email, device id
  installed.json                             everything install registered
~/.local/state/myc/                          per-device state; never committed
  sessions/<session_id>.jsonl                prompts, turns, positions, injected ids, call log
  questions.json  review.json  hooks.jsonl  cron.jsonl  metrics.jsonl  notify.json  cron.lock  git.lock
```

- `author` is a short handle the user picks during installation. The email address behind it is kept only in `device.json`, so no address appears in the log or its paths (**Safety**). `device` is a random identifier created during installation. Each author and device has its own folders, so two devices never write to the same file.
- `.myc/` can be deleted at any time; nodes, links, states, and review positions are rebuilt from `events/` (**Durability**).
- Losing per-device state weakens quote checking, question pacing, and duplicate-injection checks for up to 7 days, and resets background review's daily queue and failure counts; it never changes a node's state or a review position. Once a day, the background job deletes per-device entries older than 7 days (`sessions/`), 30 days (`hooks.jsonl`, `cron.jsonl`, `questions.json`), or 90 days (`metrics.jsonl`).

### 2.2 Fields on every event

| Field | Format | Meaning |
|---|---|---|
| `v` | integer | Schema version; this SPEC defines version 1 |
| `id` | ULID | Unique event id |
| `time` | ISO 8601 with seconds and offset | Set by `myc`, never by the caller. If that time is earlier than the latest event about any node the event names, `myc` uses that event's time plus one second |
| `type` | string | §2.3 |
| `author`, `device` | string | §2.1 |
| `origin` | `live`, `autonomous`, or `background` | Taken from `MYC_ORIGIN`; `live` when unset. The background job always writes `background` |
| `tool` | `claude`, `codex`, `pi`, or `cli` | Taken from `MYC_TOOL` (§6.4); `cli` when unset |
| `session_id` | string or null | Taken from `MYC_SESSION_ID` (§6.4) |
| `project_id` | string or null | Stable project slug |
| `domain` | string or null | The session's current domain (§1.5) |

### 2.3 Event types

| Type | Payload | Written by |
|---|---|---|
| `project.created` | `name`, `domain` | `myc project new` |
| `project.renamed` | `name` | `myc project rename` |
| `project.repo` | `op` (`link` or `unlink`), `remote` | `myc project link`, `unlink` |
| `project.state` | `field` (`phase` or `health`), `value` | `myc project set` |
| `project.item` | `list` (`next` or `needs_me`), `op` (`add` or `remove`), `item_id`, `text` | `myc end`, `myc item` |
| `work` | `items` (strings) | `myc end` |
| `case.candidate` | §2.6 | `myc capture`, background review |
| `principle.candidate` | §2.6 | synthesis, `myc principle add`, confirming a proposal |
| `principle.edit` | §2.6 | synthesis, `myc review`, `myc principle edit` |
| `proposal` | §2.6 | synthesis |
| `link` | §2.6 | §1.4 |
| `answer` | §2.6 | `myc answer`, `myc capture --save`, `myc principle add`, `myc review` |
| `retire` | `target`, `reason` (string), `restore` (boolean) | `myc retire` |
| `reviewed` | `source_session` (`{tool, session_id}`), `upto` (position) | background review |
| `synth.run` | `upto` (time of the latest answer read), `proposed` (counts by output kind) | synthesis |
| `purge` | `target` | `myc purge` (§9.3) |

In a `project.repo` event, `remote` is the normalized remote URL, the hash of the earliest root commit for a repository without a remote, or `path:<absolute path>` for a folder that is not a git repository. A `path:` remote is valid only on the device that wrote it.

### 2.4 Ordering and folding

Current state is computed by folding events in this order:

1. Lines with the same `id` are collapsed into one. If the copies differ, a purged copy (§9.3) wins; otherwise the first one read wins.
2. Events are sorted topologically by reference — an event that names another node, proposal, or link in `target`, `same_as`, `replaces`, `from`, or `to` comes after it — with ties broken by `time`, then by `id`. Every device therefore produces the same order.
3. An event whose referent is missing (not yet pulled, or purged) is held and applied once the referent appears; one that refers to a purged event is ignored.

`related` and `compared` do not affect ordering. A `principle.edit` overwrites the named fields of the principle, in fold order; `myc review` undoes an edit by writing another edit that restores the earlier values.

### 2.5 Writing

- Each event is one line, appended while holding an OS file lock. If the file does not end with a newline, `myc` writes one first.
- Every command that writes events commits the files it wrote under `events/<author>/<device>/` and `eval/<author>/` to the knowledge base's git repository, with a message naming the event types. `myc` never stages any other path; the user commits `glossary.md`, `.myc-rules`, and `STATUS.md`. `myc install` writes a `.gitignore` that excludes `.myc/`.
- Every git operation `myc` runs, in a command or in the background job, holds the OS file lock `git.lock`. A command that cannot take `git.lock` at once leaves its files uncommitted; the next sync commits them (§7.1).
- When `MYC_HEADLESS=1` is set (§7.3), every `myc` command that writes events refuses and exits with an error.
- Per-device state files are read and written under the same kind of lock.
- An unreadable line is skipped, and lint and audit report where it is.
- An event with a schema version newer than the reader understands is skipped and reported by `myc review`. Older versions are upgraded as they are read.

### 2.6 Payloads

Field types use JSON. `?` marks an optional field; every other field is required. `myc` fills the fields marked *(myc)*, and values a caller supplies for them are ignored. Every payload below also carries `redacted` and `flagged` *(myc)*, as defined for `case.candidate`.

**`case.candidate`**

| Field | Type | Meaning |
|---|---|---|
| `kind` | string | `diagnosis`, `technique`, `approach`, `definition`, `fact`, or `rejection` |
| `polarity` | string | `solution` or `dead-end` |
| `summary` | string | One line |
| `problem` | string | The problem as it looked before the cause was known, in one or two sentences |
| `first_checks` | string array | What was actually checked or tried first: concrete checks, attempts, or ways of looking at the problem |
| `facts` | object array | Each `{text, source, as_of}`. `text` is quoted exactly (**advisory**); `source` is `user`, `tool_output`, `file`, or `doc`; `as_of` is the commit, data version, or date at which it held |
| `resolution` | string | How the problem was resolved, in one or two sentences |
| `conditions` | string array | At least one condition under which the case holds, or `["unknown"]`. If the case depends on a tool, library, or policy, it includes its version or the premise it relies on (**advisory**) |
| `quotes` | object array? | Each `{who, text, turn, occurred_at, status}`; `who` is `user`, `agent`, or `tool`; `occurred_at` is the time the quoted text appeared; `status` *(myc)*, for `who: user` quotes only, is `verified` or `unverified` (§1.2) |
| `flow` | object array? | The sequence of hypotheses, each `{step, text}` with `step` one of `suspect`, `evidence`, `reject` |
| `keys` | object | `{symptom: string array, claim: string array}`, phrases for the case index |
| `synonyms` | object | `{ko: string array, en: string array}`, at least one phrase in each |
| `options` | object array? | Other interpretations offered in the question, each `{summary, first_checks, facts}`. The case's top-level fields form the main option. In the question, `myc` labels the main option `a` and the others `b`, `c`, … in order; `s` and `r` are reserved |
| `domains` | string array | One or more domains (§1.5), or empty |
| `related` | string array | *(myc)* Up to five earlier candidates from the same session |
| `source_session` | object | *(myc)* `{tool, session_id}` |
| `span` | object | *(myc)* `{from, to}` transcript positions, or empty when no transcript is readable. For background review, `myc` converts the model's `excerpt_lines` (§7.2) to positions |
| `path` | string | *(myc)* `user` (the user asked to save it), `live`, or `review` (§7.2) |
| `redacted` | object array | *(myc)* Each `{field, kind}` for a value masked by the safety checks |
| `flagged` | boolean | *(myc)* Set by the safety checks (§9.2) |

**`principle.candidate`**

| Field | Type | Meaning |
|---|---|---|
| `kind` | string | `diagnosis`, `technique`, `redefinition`, `interpretation-check`, or `decomposition` |
| `name` | string | A name that states the relationship, for example "When a total changes, break it into its parts" |
| `when` | string array | Two or three descriptions of the problem it addresses |
| `first_checks` | string array | One to three concrete actions |
| `discriminate` | string array | What each possible result of those actions tells you |
| `not_when` | string array | Situations where the principle does not hold |
| `why` | string | One line on why it works |
| `body` | object | Fields that depend on `kind` (below; **tunable**) |
| `synonyms` | object | As for cases |
| `compared` | string array | The cases synthesis compared to find it; empty for a user-entered principle. Support is counted from `supports` links only |
| `supports` | string array? | In synthesis output only: at least two case ids. `myc` writes the principle and one `supports` link per id in a single locked append, or writes none of them |
| `proposal` | string? | *(myc)* The proposal whose confirmation wrote it (§5.2) |
| `path` | string | *(myc)* `synth` or `user` |

| `kind` | `body` fields |
|---|---|
| `diagnosis` | `symptom`, `check`, `readings` (what each result means), `rule_out` (alternatives to exclude) |
| `technique` | `problem`, `tradeoff` (what you gain and what you give up), `solution`, `limits` |
| `redefinition` | `old_trouble` (what the old definition got stuck on), `new_definition`, `separates` (what the new criterion tells apart) |
| `interpretation-check` | `observed`, `rivals` (competing explanations), `baseline` (a test that tells them apart), `prediction` |
| `decomposition` | `problem_type`, `subgoals` (each with a name), `first_step` |

**`link`**

| Field | Type | Meaning |
|---|---|---|
| `op` | string | `add` or `remove` |
| `from`, `to` | string | Node ids, as §1.4 allows for `rel` |
| `rel` | string | `supports`, `applied`, `refines`, or `absorbed-into` |
| `outcome` | string? | For `applied`: `fit`, `misfit`, or `declined` |
| `note` | string? | One line on why the link holds. `myc capture --applied` writes the case's `summary` |
| `proposal` | string? | *(myc)* The proposal whose confirmation wrote it |

**`principle.edit`**

| Field | Type | Meaning |
|---|---|---|
| `target` | string | The principle |
| `fields` | object | Each named field's new value, replacing the old one |
| `reason` | string | One line; for a scope edit, the `misfit` cases that share the condition |

**`proposal`**

| Field | Type | Meaning |
|---|---|---|
| `kind` | string | `refine` or `absorb` |
| `target` | string? | For `refine`, the principle being split. For `absorb`, an existing principle to absorb into; absent when a new one is drafted |
| `drafts` | object array | Principle payloads, each with a `supports` array of at least two case ids. For `refine`, one per branch; for `absorb`, none or one |
| `absorbs` | string array? | For `absorb`, the principles to fold in |
| `reason` | string | One or two sentences |

**`answer`**

| Field | Type | Meaning |
|---|---|---|
| `target` | string | The node, proposal, or `principle.edit` event being answered. Only `confirm` applies to an edit; it records that the user kept it |
| `result` | string | `confirm`, `chosen-only`, `reject`, or `same-as` |
| `choice` | string? | The question letter of the chosen option; `a` (the main option) when absent |
| `edits` | object array? | Each `{path, value}`, where `path` is a JSON Pointer into the target's payload, for example `/facts/0/as_of` |
| `reason` | string? | For `reject`: `unneeded` (default) or `wrong` |
| `wrong` | string? | When `reason` is `wrong`: the question letter of the option that was wrong |
| `note` | string? | Evidence in the user's own words. An answer recorded by an agent quotes it verbatim from a prompt stored for the same session; if it is not found there, `myc` drops the note and keeps the answer. An answer given in `myc review` takes it as typed. Either way, it goes through the safety checks |
| `same_as` | string? | For `same-as`: the existing case, resolved to the end of any chain of merges |
| `replaces` | string? | When "replaces existing X" was chosen: X. The `result` is then `confirm` |
| `proposal` | string? | *(myc)* The proposal whose confirmation wrote it |

`myc answer <id> <result> [--choice X] [--edit PATH=VALUE ...] [--reason R] [--wrong X] [--note TEXT] [--same-as ID] [--replaces ID]` writes an `answer`. `result` is `confirm` when the user picks an option and agrees with it, and `chosen-only` when the user picks one but holds off on judging it. A `same-as` answer adds the candidate's quotes, facts, and any new `first_checks` to the existing case.

**Question output** (returned by `myc capture` on standard output):

```
[Question <short id>] <summary of the main option>
a) <summary> — check first: <first item of first_checks>
b) <summary> — check first: <...>
s) Same as existing case <short id>: <summary>
r) Replaces existing case <short id> (it was wrong): <summary>
Masked: <count> <kind>, ...
Reply with a letter, "skip", or a correction in your own words.
```

The `s`, `r`, and `Masked` lines appear only when they apply. A **short id** is the last six characters of a ULID, which are random, extended until it is unique among events this device has read; every command accepts it wherever an id is expected. If a short id matches more than one event, the command lists the matches and exits with an error.

## 3. Search and recall

This section covers the two ways knowledge comes back at a decision point — text search over cases, and the full principle list that the agent reads — how they connect, and the output of `myc ask`.

### 3.1 Case search

Case search finds cases by their wording. It is how facts and conditions come back.

- `myc` keeps the case index in `.myc/index.sqlite`. For each log file it stores how far it has read and a hash of the file's first 4 KB. On each update it reads only new lines, unless the file is shorter than the stored offset, the hash has changed, or a `purge` event has been read, in which case it rereads that file from the start. After each sync (§7.1), the background job also hashes each file up to its stored offset and rereads any file whose hash differs, so a rewrite pulled from another device is never missed. The index stores a format version and is rebuilt when the version changes.
- Indexed text: `summary`, `problem`, `first_checks`, `facts`, `resolution`, `conditions`, `keys.symptom`, `keys.claim`, and `synonyms`, using the option and edits from the answer that set the node's current state.
- Documents and queries are tokenized the same way:
  1. Remove punctuation. Split on whitespace and wherever Hangul meets a non-Hangul character. Keep Latin letters and digits together (`5xx`, `p99`) and lowercase them.
  2. Trigram column: overlapping three-character pieces within each chunk. A chunk shorter than three characters is kept whole.
  3. Hangul bigram column: overlapping two-character pieces within each Hangul chunk.
- For example, `평가에서` yields `평가`, `가에`, `에서`, and `평가 오차` yields `평가`, `오차`; the two match on `평가`.
- Score: each piece's IDF is `ln(1 + (N − df + 0.5) / (df + 0.5))`, where `N` is the number of indexed cases and `df` the number containing the piece. For each column, the score is the summed IDF of the query pieces that match, divided by the summed IDF of all query pieces. The trigram column has weight 1 and the bigram column 0.5 (**tunable**); columns that have query pieces are averaged by weight, giving a score between 0 and 1.
- Each hook folds and indexes new lines within its time limit. Only when the index is missing or has the wrong format version is it rebuilt in the background; until then, case search returns nothing and the hook injects "index not ready".

### 3.2 The principle list

Principles are not found by text search. They contain no domain-specific terms, so the wording of a problem rarely matches them. Instead, the agent is given the whole list and picks by meaning.

- `myc` rewrites `.myc/principles.txt` whenever it folds an event that changes a principle, so the list is available without the index. If the file is missing, `session_start` folds principle events directly within its time limit, or injects "principle list not ready".
- The list contains every principle that §1.7 lists, one line each: `<short id> · <name> · when: <first when> · not when: <first not_when> · <marker> <tally>`.
- Order: `confirmed`, then `chosen-only`, then `tentative`. Within each group, principles created in the last 14 days (**tunable**) come first, then the rest by most recent activity, the later of creation and last application.
- A principle with `refines` branches is shown as one line followed by `(N branches: myc principles --expand <id>)`. Its branches are not listed separately.
- The list is limited to 2,000 tokens (**tunable**), estimated as characters divided by 3.5. If it is longer, lines are removed from the end of the order above and replaced by "and N more: myc principles --all".
- `myc show <principle id>` prints the full principle and its tally. With `--for "<situation>"`, it also prints the two linked `supports` cases that score highest against the situation, and the highest-scoring linked case with a `misfit` application, if there is one.

### 3.3 How the two connect

- From a case to principles: `myc ask` follows the `supports` and `applied` links of the cases it returns and lists those principles.
- From a principle to cases: `myc show <principle> --for` returns linked cases (§3.2).
- Either way, the agent is one step from the other (**Grounding**).

### 3.4 `myc ask`

```
Cases (best wording match first)
[marker] <summary> · check first: <one line> · key fact: <one fact> (as of <as_of>)
  conditions: <conditions> · <state> · decided <time> · principles: <short id name> · myc show <short id>
Principles linked to these cases
<short id> · <name> · when: <first when> · <tally>
```

- Cases: the top three in the current domain and the top two from other domains; with no current domain, the top five overall. Cases scoring below the minimum (§6.6) are left out.
- A case from another domain shows up to two `first_checks` lines instead of a key fact; its facts are available through `myc show`.
- Principles: at most three, ordered by the rank of the case that led to them, then by `fit` count.
- No id appears twice. `decided <time>` is the time of the answer that set the current state.
- The output always ends with: "These principles and cases are suggestions. Check that they fit the problem at hand, keep your usual checks, and re-check any figures under current conditions."

## 4. Capture

This section covers when a case is captured, how the user is asked about it, how answers are recorded, and how questions are paced so that capture stays quiet (**Low friction**).

### 4.1 Decision points

At a decision point, the agent can recall principles and search cases (§3). Once the agent gets unstuck or reaches a decision — including reversing an earlier one — it can capture a case. The injected text (§6.5) prompts both; whether the agent does them is **advisory**. Background review (§7.2) captures what a session missed.

### 4.2 Questions

`myc capture` reads a `case.candidate` payload as JSON from standard input, stores it as a candidate, and prints a question (§2.6) for the agent to add to its reply.

- At least one option offers a different interpretation from the main option (**advisory**).
- Before storing the case, `myc capture` searches existing `confirmed` and `chosen-only` cases in the same domain with the new case's `summary` and `keys.claim`, keeping those that score above the minimum (§6.6). Of these, the highest-scoring case that shares at least one condition other than `unknown` is offered as "same as existing". The highest-scoring case that has the opposite `polarity`, or whose `first_checks` score below the minimum against the new case's, is offered as "replaces existing". The question offers at most one of each.
- `--applied <principle id>:<fit|misfit|declined>`, repeatable, records an `applied` link for each principle the agent recalled at this decision point.
- `--from-turn N` starts the case's `span` at the position recorded for turn `N`.
- The agent passes `--save` when the user asks to save something (**advisory**). `myc` then writes the candidate and a `confirm` answer together and prints no question. If `first_checks` and `facts` are both empty, the case is still stored and listed in `myc review` as "needs details".

### 4.3 Answers

The `answer` payload and the `myc answer` syntax are defined in §2.6. When the user replies to a question in plain words, the agent turns the reply into `myc answer` (**advisory**); the pending question (§4.4) tells it which id to use.

### 4.4 Pacing

Questions are asked only in sessions whose `origin` is `live`.

- **One question per turn.** `myc capture` prints a question only if no question has been asked in the current turn and at least 10 minutes (**tunable**) have passed since the last question on this device. Otherwise it stores the candidate without asking. Every question asked is recorded in `questions.json` with its time and turn.
- **Queued questions.** Unanswered `tentative` cases form the question queue. When no question was asked in the previous turn and the interval has passed, `user_prompt` injects one queued question for the agent to add: the newest from this session if there is one, otherwise the oldest from the project, otherwise the oldest on this device. Injecting it counts as asking.
- **Pending question.** At the next `user_prompt`, the injected text includes the pending question's short id and option labels, so that an answer given in plain words can be recorded.
- **Session-start summary.** At `session_start`, the agent is told how many questions are queued and, if the interval has passed, given one of them. If any proposals are waiting for review, the summary says so, at most once a day.
- Principles and proposals are never asked about during a session. Synthesized principles are listed as `[unconfirmed]` (§1.7), and proposals wait in `myc review` (§5.4).

## 5. Synthesis

This section covers when synthesis runs, what it reads, what it produces, the contract with the model, and how the user reviews its output (**Synthesis**).

### 5.1 When

- The background job runs synthesis once at least 5 cases (**tunable**) have been answered `confirm` or `chosen-only` since the `upto` of the last `synth.run`, at most once a day (**tunable**), and only if no hook has fired in the last 15 minutes.
- The user can run it at any time with `myc synth`.

### 5.2 What it produces

| Output | Written as | Takes effect |
|---|---|---|
| New principle | `principle.candidate` with at least two `supports` links | Immediately, listed as `[unconfirmed]` |
| New support | `supports` link from a newly answered case to an existing principle | Immediately |
| Earlier application | `applied` link with an `outcome`, where a case shows a principle was followed | Immediately |
| Narrower scope | `principle.edit` adding to `not_when`, where several `misfit` applications share a condition | Immediately; `myc review` shows it so the user can keep or undo it |
| Refinement | `proposal` of kind `refine` | Only when confirmed |
| Absorption | `proposal` of kind `absorb` | Only when confirmed |

- A case that fits no principle stays unlinked.
- Every event written by confirming a proposal carries `proposal: <id of the confirming answer>`. When folding, only events whose `proposal` names the proposal's first answer in fold order are applied; events from any later confirmation, for example from a second device that confirmed before syncing, are ignored.
- Confirming a `refine` proposal writes each draft as a `principle.candidate` with its `supports` links, a `confirm` answer, and a `refines` link to the target.
- Confirming an `absorb` proposal first writes the draft, if there is one, as a `principle.candidate` with its `supports` links and a `confirm` answer. It then writes a `confirm` answer for an existing `tentative` target, and an `absorbed-into` link from each principle in `absorbs` to the target or the new principle.
- Rejecting a proposal changes nothing else.
- A scope edit only narrows: it adds to `not_when` and never removes from `first_checks`. A narrower principle is offered in fewer situations; at worst, that is the same as not having it.
- Lint reports two principles whose names score 0.8 or higher against each other in either direction, using the §3.1 score with IDF computed over principle names.
- `synth.run` records the time of the latest answer read and how many of each output synthesis wrote.

### 5.3 Model contract

Synthesis is one headless run (§7.3), or, with more than 200 cases, one run per domain followed by a reconciling run. `myc` sends, as JSON on standard input:

- `cases`: every `confirmed` and `chosen-only` case — or, with more than 200 such cases, those of one domain at a time, followed by a final run that reconciles the results — with `id`, `kind`, `polarity`, `problem`, `first_checks`, `conditions`, `resolution`, and `domains`, after safety checks;
- `principles`: every principle in any state, with its current fields, links, and tally;
- `proposals`: every earlier proposal and its state;
- `instructions`: the rules in §1.3 and §5.2, and a request not to restate a relationship that an existing principle already states, whatever its state (**advisory**).

The model returns JSON: `{principles: [...], links: [...], edits: [...], proposals: [...]}`, each item in the payload shape of §2.6 (`principle.candidate` with `supports`, `link`, `principle.edit`, `proposal`), with ids taken from the input. `myc` validates every item and writes the valid ones as events; it discards the rest and logs why in `cron.jsonl`. The model does not write events itself; `myc` refuses writes under `MYC_HEADLESS=1` (§2.5).

### 5.4 Review

`myc review` lists synthesis output: confirm, edit, or reject a principle; confirm or reject a proposal; keep a scope edit (a `confirm` answer on the edit) or undo it (a reverse edit). Reviewing new principles and scope edits is optional, because they are already in effect. Proposals change what the agent sees, so they wait for the user. Output left unreviewed for 7 days is reported by audit (§10), and the session-start summary mentions waiting proposals (§4.4).

## 6. Agent integration

This section covers the hook contract the three agent tools share, how it maps onto each tool, the rules every hook run follows, the text injected into sessions, automatic case search, and the conclusion nudge.

### 6.1 Common input and output

```json
{"tool": "claude|codex|pi", "event": "session_start|user_prompt|turn_end|session_end",
 "session_id": "...", "cwd": "...", "transcript_path": "... or null",
 "source": "startup|resume|clear|compact (session_start only)",
 "prompt": "user_prompt only", "last_message": "turn_end only", "continuing": false}
{"inject": "text to add to the context, or null", "continue_with": "instruction to keep going, or null"}
```

- A turn is one user input. The turn number is the count of `user_prompt` hook runs recorded for the session, so it needs no transcript.
- All three tools write transcripts as one JSON object per line, so a **position** is a line count and needs no parsing of entry contents. At `user_prompt` and `turn_end`, `myc` records the turn and, when `transcript_path` is readable, the current position in per-device state. Where no transcript can be read, case `span` is empty and that tool is not reviewed in the background.
- `continuing` is true when this `turn_end` follows a `continue_with` returned by the previous one.
- `session_end` is logged only.

### 6.2 Mapping to each tool

| Common | Claude Code | Codex | pi (extension) |
|---|---|---|---|
| `session_start` | `SessionStart` (with `source`) | `SessionStart` | prepared at `session_start`, injected at the first `before_agent_start` |
| `user_prompt` | `UserPromptSubmit` | `UserPromptSubmit` | `before_agent_start` |
| `turn_end` | `Stop` | `Stop` | `agent_before_settle` |
| `session_end` | `SessionEnd` | `SessionEnd` | `session_shutdown` |
| `continuing` | `stop_hook_active` | `stop_hook_active` | set by the extension when it requested the previous continuation |
| `inject` | `additionalContext` | `additionalContext` | prompt sections added in `before_agent_start` |
| `continue_with` | `decision: block` with `reason` | `decision: block` with `reason` | an entry appended in `agent_before_settle`, with `continue: true` |

- pi is integrated through a TypeScript extension that builds the same input, calls `myc hook`, and shows the missing-binary notice itself when `myc` is not installed. pi's `turn_end` fires after every model request, so the extension uses `agent_before_settle`, which fires once when a run would otherwise stop. Messages the extension sends itself, including continuations, are not stored as prompts, not counted as turns, not searched, and do not trigger queued questions.
- Codex runs hooks only after the user has trusted them; `myc install` prints the step and `myc install --verify` checks it.

### 6.3 Hook command

For Claude Code and Codex:

```
if [ -x ~/.config/myc/bin/myc ]; then ~/.config/myc/bin/myc hook <event> --from <tool>
else <session_start only: print "myc is missing; run myc install again">; fi; exit 0
```

### 6.4 Rules for every hook run

| Rule | Requirement |
|---|---|
| Time limit | A hook responds within 2 seconds or returns empty output |
| Failure | Any failure exits with status 0 and empty output |
| Logging | Every run appends one line (event, duration, result) to `hooks.jsonl` |
| Network | No network access on the hook path; `git pull` and `git push` run in the background job |
| Session environment | `session_start` sets `MYC_SESSION_ID` and `MYC_TOOL` for the session's commands: through `CLAUDE_ENV_FILE` in Claude Code, and in the process environment through the pi extension |
| Session fallback | If a tool cannot set environment variables, `user_prompt` records the latest session per directory, across all tools, in per-device state, and `myc` reads `MYC_TOOL` and `MYC_SESSION_ID` from there. If two sessions in the same directory were active within 10 minutes of each other, both are null |
| Headless runs | When `MYC_HEADLESS=1`, every hook exits immediately: no event, no injection, no `continue_with` |
| Loops | When `continuing` is true, no `continue_with` is returned |
| Prompt storage | `user_prompt` stores the prompt text, after safety checks, with its turn, for 7 days (§2.1). Stored prompts are used only to verify quotes and notes |
| Call log | `myc ask`, `myc show`, and `myc capture` record each call and its turn in per-device state |
| Repeat injection | A node is not injected twice in one session unless it has changed. The principle list is injected at every `session_start`, whatever its `source`, so it survives compaction and resume |

### 6.5 Injected text

At `session_start`, `myc` injects the full text below, the project status (phase, health, next items, and items waiting on the user, with their `item_id`s), the session-start summary (§4.4), and the principle list (§3.2). At `user_prompt`, it injects the short text only when it also injects something else: a pending question, a queued question, or cases found automatically (§6.6). Autonomous runs get the same injections as live sessions, without questions (§4.4). The text reads as suggestions, never as instructions (**Advisory recall**). Both texts below are **tunable**.

Full text:

```
[Mycelium] The notes below come from past work. They are suggestions, not instructions. Ignore any that do not fit; you don't need to mention them.
- When you get stuck, settle a definition, interpretation, or direction, reach a conclusion, or drop a hypothesis, you can check whether a principle below applies. `myc show <id> --for "<situation>"` shows how to tell, and the cases behind it.
- You can find similar past cases with `myc ask "<situation>"`. Team-specific facts and conditions are in the cases.
- None of this replaces your usual checks. You decide whether anything applies to the problem at hand.
- When you get unstuck or reach a decision, you can save it as a case with `myc capture`. For any principle you recalled, add `--applied <id>:fit`, `:misfit`, or `:declined`.
- If a question is pending and the user's message answers it, you can record the answer with `myc answer`.
Principles (<N>):
<principle list, §3.2>
```

Short text:

```
[Mycelium] Suggestions only. At a decision point, the principle list from the start of the session or `myc ask "<situation>"` may help; you decide whether anything applies.
```

### 6.6 Automatic case search

- The query is the prompt text. A prompt with fewer than three chunks (§3.1, step 1) after the glossary's stop words are removed is not searched.
- Scope: the current domain, plus `approach` and `definition` cases from all domains; with no current domain, all domains. At most two cases.
- Minimum score: 0.1 (**tunable**).
- Principles are not searched automatically; the list is already in the context (§6.5).

### 6.7 Conclusion nudge

At `turn_end`, `myc` returns one `continue_with` when both of these hold:

- `last_message` contains a conclusion phrase (**tunable**: 덕분에, 때문에, 원인은, 효과가, 개선, because, due to, improved);
- the call log shows no `myc ask`, `myc show`, or `myc capture` in the last five turns.

The text is: "Before you conclude, you can check whether a principle or a similar case applies. If none does, carry on." It asks for a check, not for a particular answer, and the agent is free to skip it. A session gets at most three nudges (**tunable**).

## 7. Background job

This section covers the job that captures cases a live session missed, keeps the log in sync, and starts synthesis and audit.

### 7.1 Running

- The job runs every 10 minutes, under launchd on macOS and cron elsewhere. It takes an OS file lock (`flock` on `cron.lock`) and exits immediately if the lock is held; the lock is released if the process dies. Its first action is to append a line to `cron.jsonl`.
- At `session_start`, if `cron.jsonl` has no line from the last 20 minutes, `myc` starts one run, detached from the session and with a clean environment.
- Each run, in order: apply any unapplied `purge` events to this device's files (§9.3); sync; review sessions; run synthesis if due (§5.1); clean up per-device state (§2.1); aggregate metrics (§11); audit (§10).
- Sync: commit any of this device's event files that are not yet committed, then `git pull --rebase --autostash`, then `git push`. Nothing happens if the repository has no remote or upstream. If the rebase fails, `myc` aborts it and leaves local commits in place; audit reports it as "Push pending" (§10).

### 7.2 Background review

- Choosing sessions, without calling a model:
  1. Read recent sessions from each tool's session store.
  2. Skip headless runs: sessions whose first user input begins with `[myc-headless]` (§7.3).
  3. Keep sessions whose transcript has grown past their review position and that have been idle for 15 minutes (**tunable**).
  4. Skip sessions with no activity in the last 3 days (**tunable**).
- A session's **review position** is the `upto` of its latest `reviewed` event, or 0. Capturing a case does not move it, so a session where the agent captured some decision points is still reviewed for the rest.
- For each chosen session, `myc` reads the transcript lines after the review position, applies the safety checks to them, and sends them in one headless run with the session's existing case candidates. The model returns `case.candidate` payloads, each with `excerpt_lines` (`{from, to}`, line numbers within the excerpt), for every place the agent got unstuck or reached a decision that no existing candidate covers; which places qualify is **advisory** for the model. `myc` validates each one, writes it with `path: review`, and then writes `reviewed` with the new `upto`.
- Daily limit: 5 sessions (**tunable**), sessions with the most new activity first; the rest carry over to the next day. The queue and failure counts are kept in `review.json`. A session whose run fails moves to the back of the queue. After three consecutive failures on different sessions of one tool, review of that tool stops until `myc install --verify` passes, and the user is notified right away, with the tool and the last failed session named (§10). Synthesis failures are logged in `cron.jsonl` and do not count.

### 7.3 Headless runs

- A headless run sends a prompt and JSON on standard input to one installed agent tool and reads JSON from standard output: `claude -p`, `codex exec -`, or `pi --print`. `myc` uses the tool the reviewed session came from when it is installed, and otherwise the first tool that passed `myc install`.
- The prompt's first line is `[myc-headless]`, and the run has `MYC_HEADLESS=1` and `MYC_ORIGIN=background`, so its own hooks do nothing, `myc` refuses its writes, and background review never picks it up.
- Its input has already passed the safety checks, so the transcript the tool keeps of a headless run holds only masked text.
- Output that is not valid JSON in the expected shape is discarded and logged in `cron.jsonl`.

## 8. Commands

This section covers the commands the user runs, the commands an agent runs, and the internal commands.

### 8.1 For the user

| Command | What it does |
|---|---|
| `myc install` | Copies the binary to `~/.config/myc/bin/myc`, creates the knowledge base as a git repository, asks for the author handle, detects the agent tools, registers hooks, the pi extension, the background job, and a git `pre-push` hook in the knowledge base, and writes `.gitignore` and a default `.myc-rules`. It then runs each tool once non-interactively, with a prompt that starts with `[myc-headless]` and without `MYC_HEADLESS`, checks that `hooks.jsonl` shows the `session_start` injection, checks whether the tool can set environment variables for its session, and records the results in `installed.json`; no event is written. `--root` and `--state` set paths; `--verify` re-checks an existing install |
| `myc uninstall` | Removes exactly what `installed.json` lists |
| `myc review` | Steps through, in the terminal: queued questions, synthesis output (§5.4), "needs details" cases, scan findings, and audit items. `--json` prints the same list for scripts |
| `myc synth` | Runs synthesis now |
| `myc principle add`, `edit` | Enters a principle directly (written with a `confirm` answer) or edits one |
| `myc scan` | Re-checks the whole log and stored prompts against the current safety rules (§9.1) |
| `myc purge <id>` | Removes an event's content (§9.3); `--device <device>` applies purges to the folder of a device that is no longer used |
| `myc eval add <query> <id>` | Adds a confirmed search test pair; the id may be a case or a principle |

`myc` runs on macOS, Linux, and WSL2. Installation stops if it detects a native Windows shell, a shared OS account, or a network or Windows-mounted drive (NFS, `/mnt/c`) under the knowledge base or state paths, and asks for local paths instead. It also checks how long each tool keeps transcripts and warns that background review may miss sessions from any tool that keeps them for less than a day.

### 8.2 For the agent

| Command | Input | Output |
|---|---|---|
| `myc context` | working directory | Project status, current domain, list items with their `item_id`, and the principle list |
| `myc end` | what was done; `--next`, `--need`, `--done <item_id>` | `work`, `project.item` |
| `myc item add`, `remove` | list name, and text or `item_id` | `project.item` |
| `myc project new`, `rename`, `link`, `unlink`, `set` | name, domain, repository or `--path`, phase or health | `project.*` |
| `myc capture` | case payload on stdin; `--save`, `--from-turn N`, `--applied <id>:<outcome>` | A candidate and a question (§4.2) |
| `myc answer` | §2.6 | `answer` |
| `myc ask "<situation>"` | | §3.4 |
| `myc show <id>` | `--for "<situation>"` | A case in full, or a principle with its tally and linked cases (§3.2) |
| `myc principles` | `--expand <id>`, `--all` | The principle list, a principle's branches, or every listed principle |
| `myc link <from> <to> <rel>` | `--outcome`, `--remove`, note | `link` |
| `myc retire <id>` | reason; `--restore`; `--cascade` | `retire`. Lists any case that would stop being superseded, or principle that would stop being absorbed, if this one were retired; `--cascade` retires those too |
| `myc hook <event> --from <tool>` | §6.1 | §6.1 |

### 8.3 Internal

`myc cron`, `myc audit`, `myc lint`, `myc index`, `myc eval`.

## 9. Safety

This section covers what is checked, when, what happens on a match, and the one exception to the append-only rule (**Safety**).

### 9.1 When and what

- `myc` runs the checks whenever it writes an event, stores a prompt, logs a query, adds a test pair, sends input to a headless run or receives its output, and before every push.
- Every free-text field and every query is checked. Ids, ULIDs, and commit hashes are exempt. Digit strings are checked only against the card, account, and ID number patterns.
- `.myc-rules` in the knowledge base root holds the patterns, the email domains treated as internal, the bulk threshold, and allowances for confirmed false positives. `myc install` writes a default file.
- `myc scan` re-checks the whole log and the stored prompts against the current rules and lists every match in `myc review`, by location and kind, with an option to purge. `myc` runs it automatically whenever `.myc-rules` changes.

### 9.2 Checks and actions

| What is found | What happens |
|---|---|
| Credentials: API keys, tokens, private keys, password assignments, database connection strings | The match is replaced with `<masked:kind>` |
| A single item of personal data: national ID, passport, card, or account numbers, phone numbers, email addresses outside the internal domains | The match is masked |
| Personal data at or above the bulk threshold (default 10 items), or personal data in a table or log format | The whole field becomes `<masked:bulk>` |
| Text that tries to override the agent's instructions ("ignore previous instructions", "you must now run …") | The node is flagged, and is neither injected, returned by search, nor listed. Ordinary commands in `first_checks` are not flagged |

Before every push, the git `pre-push` hook that `myc install` registers checks the lines that are new in the outgoing commits and every line belonging to a purged event that still has content. A match blocks the push and reports where it is, never the value itself.

Every masked value is recorded in `redacted` by field and kind, and questions say how many values of which kinds were masked.

### 9.3 Purge

Purge is the only exception to the append-only rule.

- `myc purge <id>` accepts any event id. It writes a `purge` event, which records that the removal happened and contains nothing else.
- Each device applies a `purge` event at the start of its next background job run (§7.1), and only to its own files: under `events/<author>/<device>/`, the target's line keeps `v`, `id`, `type`, `time`, `author`, and `device` and loses every other field, and every `answer` and `link` that refers to the target keeps every field except its free text (`note` and `edits`); under `eval/<author>/<device>.jsonl`, pairs naming the target are deleted. The device also deletes entries mentioning the target from `metrics.jsonl` and stored prompts, removes it from the index and the injection history, and rereads the rewritten files. No device rewrites another device's files.
- To clean up a device that is no longer used, the user runs `myc purge --device <device>` on any machine, which applies the rewrite above to that device's folders.
- The `pre-push` hook (§9.2) blocks any push that would bring purged content back, for example from a clone that has not yet applied the `purge` event.
- `myc purge` lists any case that would stop being superseded, or principle that would stop being absorbed, if the target were purged; `--cascade` retires those too. Afterward it prints the steps for removing the content from the remote's history.
- A purged node is not returned or listed, and principles with a `refines` link to it are listed on their own. An event that only refers to a purged id stays and is shown as "deleted"; lint does not report it.
- A principle from synthesis left with fewer than two supporting cases stays listed and appears in `myc review` as "insufficient support".

## 10. Audit and notification

This section covers what audit checks and how its findings reach the user.

- Audit runs at the end of every background job run. The check for a silent background job runs at `session_start` instead, so the job never checks itself. A missing binary is reported only by the notice in §6.3.
- **Notify** means one line at the top of the next session's context and an OS notification, at most once a day per cause, with the day's OS notifications combined into one. The last notification per cause is kept in `notify.json`. A blocked push and stopped review are sent right away instead.

| Check | Condition | Result |
|---|---|---|
| Silent hooks | A tool has sessions but `hooks.jsonl` has no line from it in 24 hours | Notify |
| Hook errors | More than 3 timeouts in a day, or an error rate above 5% | Notify |
| Silent background job | No `cron.jsonl` line within 20 minutes of a session start, after a detached run was started | Notify |
| Review stopped | Three consecutive headless-run failures on different sessions of one tool | Notify right away, naming the tool and the last failed session |
| Queued questions | Any | `myc review` and session-start summary |
| Synthesis waiting | Output unreviewed for 7 days | `myc review` |
| Scan findings | `myc scan` matches not yet purged | `myc review` |
| Push pending | Local commits ahead of the remote for 24 hours | Notify; if a safety check is the cause, give the location and `myc purge` |
| Lint errors | Any | `myc review` |

## 11. Measurement

This section covers what is measured to tune the settings and to check whether the MISSION success criteria hold.

- `myc ask` and `myc show --for` log the first 300 characters of the query, the ids returned, and their scores to `metrics.jsonl`. Automatic search logs only the ids returned and their scores, because its query is the user's prompt.
- `myc review` lists queries that returned nothing; the user can turn one into a test pair with `myc eval add`.
- `myc eval` reports, over the test pairs, how often the expected case is in the top five of `myc ask`, and how often an agent given the principle list names the expected principle first. With fewer than 30 pairs, the bigram weight (§3.1) and the minimum score (§6.6) are not changed.

| Metric | Definition | MISSION item |
|---|---|---|
| Capture, confirmation, and unanswered rates | Candidates per session; share confirmed; share still `tentative` after 7 days | **Low friction** |
| Noise rate | Share of questions rejected as unneeded or left unanswered | **Low friction** |
| Fidelity | Share of a sample of week-old confirmed cases that the user confirms again in `myc review` | **Reuse** |
| Search precision | Share of top injected cases the user judges relevant in a sample shown in `myc review` | **Reuse** |
| Miss rate | Share of case test pairs whose expected case is not in the top five | **Reuse** |
| Principle pick rate | Share of principle test pairs where the agent names the expected principle first | **Reuse**, **Advisory recall** |
| Recall rate | Share of turns with a decision-point capture that also have a `myc show` of a principle | **Advisory recall** |
| Correct declines | Share of recalled principles recorded as `declined`; counted as correct use | **Advisory recall** |
| Tallies | `fit`, `misfit`, and `declined` counts for each principle | **Synthesis** |
| Synthesis acceptance | Share of synthesis output that the user confirms or keeps | **Synthesis** |
| Operations | Hook error rate, headless-run count and failures, push delay | **Agent-agnosticism**, **Durability** |

The maintainer decides whether these measurements justify changing a setting, by amending this SPEC (**advisory**).

## 12. Tunable settings

This section covers every setting marked **tunable** and the measurement that would justify changing it.

| Setting | Value | Measured by |
|---|---|---|
| Question interval | 10 minutes | Noise, rejection, and unanswered rates |
| Question queue expiry | 30 days | Unanswered rate; how many expired cases the user later confirms in `myc review` |
| Background review: daily limit, idle time, lookback | 5 sessions; 15 minutes; 3 days | Sessions chosen, candidates per run, headless-run count |
| Synthesis trigger | 5 newly answered cases; at most once a day | Synthesis acceptance; how often principles are edited or absorbed |
| Principle list: size limit, new-principle window | 2,000 tokens; 14 days | Principle pick rate and recall rate as the list grows |
| Bigram weight, minimum score | 0.5; 0.1 | `myc eval` with at least 30 test pairs |
| Conclusion phrases, nudges per session | §6.7; 3 | How often a nudge leads to a recall |
| Injected text | §6.5 | Recall rate, correct declines, noise rate |
| Principle body fields by kind | §2.6 | The maintainer's reading of principles of each kind, and their tallies |

## 13. Acceptance criteria

This section covers the checks an implementation of this SPEC must pass. Each runs against a scripted knowledge base and recorded hook inputs, without a live model unless it says so.

- For each of the three tools, a recorded `session_start` input with each `source` value produces the session-start context and the principle list.
- In a project with a phase, a health value, a next item, and an item waiting on the user, `session_start` injects all four (**Continuity**).
- A scripted ten-turn session that captures a case in every turn produces at most one question per turn and no two questions within the interval.
- With the interval passed and no question asked in the previous turn, a queued question appears in the next `user_prompt` injection and in the next session-start summary.
- `myc ask "평가에서"` returns a confirmed case summarized as "평가 오차", and `myc ask "loss가 튐"` returns one summarized as "loss 급등", each ranked first, and `myc ask "latency spike after deploy"` ranks first a case summarized as "p99 latency spikes after deploys", in a knowledge base with fewer than ten confirmed cases.
- On at least 30 principle test pairs whose queries share no domain-specific term with their principle, for each installed tool, the median principle pick rate of three `myc eval` runs with the tool's default model is at least 90%.
- `myc show <principle> --for` returns up to two supporting cases, plus the highest-scoring linked `misfit` case when one exists.
- `myc` refuses a principle from synthesis with fewer than two `supports` links, and a principle with an empty `first_checks`, `discriminate`, or `not_when`.
- With a live model, on a fixture where one principle is supported by cases from two distinct sub-patterns, synthesis writes a `refine` proposal in at least 4 of 5 runs; on a fixture where two principles are supported by largely the same cases, it writes an `absorb` proposal in at least 4 of 5 runs.
- Every link synthesis writes names ids that appear in its input or among the principles it writes in the same run.
- Confirming a `refine` proposal writes its branches and `refines` links; confirming an `absorb` proposal writes its `absorbed-into` links, and the absorbed principles leave the list.
- A principle written by synthesis appears in the next session's list without any answer.
- A credential in a quote is masked when written; a match in an outgoing commit, or a purged event that still has content, blocks `git push`.
- One author writing from two devices in the same month causes no merge conflict, including after a purge.
- Deleting `.myc/` and rebuilding produces the same nodes, links, states, and review positions.
- Hooks with `MYC_HEADLESS=1` write no event, inject nothing, and return no `continue_with`, and background review never chooses a session whose first input begins with `[myc-headless]`.
- `turn_end` never keeps a session going more than once in a row.
- Removing the binary produces the one-line notice at the next session start.
