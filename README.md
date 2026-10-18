# Mycelium

Mycelium is the shared memory of a team that works with AI agents. This repository holds its design. There is no implementation yet.

## Documents

There are three groups. Each decision has one home.

**Specification.** These four documents define the product and are normative. Read them in this order.

| Document | Owns | How to read it |
|---|---|---|
| [MISSION.md](MISSION.md) | Purpose, success criteria, scope | In full |
| [PRINCIPLES.md](PRINCIPLES.md) | Primitives, invariants (I1–I10), precedence, settled readings | In full |
| [DESIGN.md](DESIGN.md) | Mechanisms (M0–M15) and why each has its shape | In full |
| [CONTRACTS.md](CONTRACTS.md) | Observable behavior: core (Part I), extension modules (Part II), tunables, acceptance properties, and other tables (Part III) | Look up |

**Input.** The specification meets this document.

| Document | Owns | How to read it |
|---|---|---|
| [REQUIREMENTS.md](REQUIREMENTS.md) | The maintainer's requirements (R) and the current decisions (D) | Look up |

**Reference.** These documents are not normative.

| Document | Owns | How to read it |
|---|---|---|
| [DEFAULTS.md](DEFAULTS.md) | Recommended default behaviors for an implementation | Look up |
| [LATER.md](LATER.md) | Features deferred from the first version | Look up |

## Rules between the documents

- A lower document refines a higher one. It never silently amends it.
- A contradiction is a defect in the lower document.
- A clause stays normative only when its removal breaks an invariant, a requirement or decision, or the consistency of state that the team shares. Other behavior is in DEFAULTS.md.

## History

- `SPEC.md` (the first single-document specification) is replaced by PRINCIPLES.md, DESIGN.md, and CONTRACTS.md. It stays in the git history.
