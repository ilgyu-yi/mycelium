# Mission

## What this exists for

Mycelium lets someone without the original context — the same user months later, a fresh agent session, or a teammate the user shares the knowledge base with — reuse the way the user thinks through problems. Each time a decision is made, it records the case: how the problem was actually solved. Over time it distills the principles that recur across cases. When a new decision point comes up, it puts both in front of the agent, whichever supported agent tool is running and whatever the domain.

What it keeps is problem-solving know-how: what to check first, how a symptom was diagnosed, which definition ended a dispute, how a large problem was broken down — together with the team-specific facts that know-how depends on. It also tracks project status, so that a new session knows where the work stands.

## Success looks like

- **Reuse** — an agent with no memory of the original work uses a past principle or case at a decision point and makes a better decision because of it, including when the new problem is in a different domain from the one the knowledge came from.
- **Advisory recall** — principles inform the agent; they never direct it. The agent decides whether a principle applies, and setting aside one that does not fit is correct use.
- **Grounding** — every principle that synthesis produces stays linked to the cases it came from, so the conditions and facts behind it are one step away.
- **Synthesis** — as cases accumulate, principles get sharper, not just more numerous: broad principles split, overlapping ones merge, and cases where a principle did not fit narrow the conditions under which it applies.
- **Low friction** — capturing a case costs the user one short multiple-choice answer, asked only at a natural point. New principles reach the agent without the user reviewing them, and nobody has to write a knowledge article.
- **Continuity** — a new session in a project starts knowing the project's phase, health, next steps, and what is waiting on the user.
- **Agent-agnosticism** — one knowledge base serves the Claude Code, Codex, and pi agent tools, and nothing depends on a particular model or tool. Korean and English work equally well in capture and search.
- **Durability** — the event log is append-only apart from deliberate removal of sensitive content, and readable by people; every index, view, and principle list can be rebuilt from it.
- **Safety** — credentials and personal data stay out of the log, and anything that gets in can be found and removed.

## Explicitly NOT goals

- Being a general document store, wiki, or search engine.
- Replacing the agent's own judgment or its usual checks. A principle adds to what the agent would have done; it never stands in for it.
- Enforcing principles. Nothing blocks or forces an action because of a principle.
- Deriving a principle from a single case. A pattern seen once stays a case. The user can still enter a principle directly.
- Keeping transcripts. Only decision points become part of the log; prompt text is held briefly on the device to verify quotes and then deleted.
- Requiring a server, a hosted service, or a vector database. One machine and a git remote are enough.
- Governing knowledge between people — ownership, access control, reconciling competing views from different authors. This repository specifies one user's knowledge base, which a teammate can read or extend through the same git remote.

## Stakeholders

Maintainer: ilgyu-yi (decision maker). Primary users: one person who works with coding and research agents, and the agents that read and write the knowledge base on that person's behalf.

## Last reviewed: 2026-10-15
