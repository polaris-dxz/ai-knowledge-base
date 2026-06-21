# AGENTS.md

Instructions for Codex, Claude Code, Cursor, Gemini CLI, OpenHands, and future AI agents working in this repository.

---

## Repository Purpose

This repository is a **Personal Technical Operating System (PTOS)** — not a software project.

It accumulates and evolves knowledge in:

* AI Agent research
* AI Infrastructure research
* AI Platform architecture
* Technical investigation
* Architecture analysis
* Technical writing
* Knowledge management

Do not treat this as a source code repository unless explicitly instructed.

Primary objective: transform reading materials, project experience, and technical investigations into a **reusable, linked knowledge system**.

Guiding priorities:

```text
understanding > collection
abstraction > copying
knowledge graph > isolated notes
thinking system > note taking
```

Connect notes through concepts, systems, and architecture patterns whenever possible.

---

## Knowledge Evolution Model

All knowledge follows this flow:

```text
Source → Concept → System → Architecture → Output
```

| Layer | Responsibility |
|-------|----------------|
| Source | Preserve original references, summaries, and open questions |
| Concept | Atomic definitions reusable across systems |
| System | Complete analysis of a specific product or system |
| Architecture | Reusable patterns abstracted from multiple systems |
| Output | Publishable drafts derived from existing notes |

Do not skip layers when creating Output. Prefer incremental evolution over one-shot articles.

---

## AARRA Workflow

When studying a new system, follow:

```text
Acquire → Abstract → Reverse → Review → Articulate
```

| Phase | Action | Target Directory |
|-------|--------|------------------|
| Acquire | Collect docs, papers, source code, issues | `01 Sources/` |
| Abstract | Extract terms and atomic concepts | `02 Concepts/` |
| Reverse | Map architecture, components, data flow | `03 Systems/` |
| Review | Cross-check sources, resolve contradictions | Update Concepts + Systems |
| Articulate | Extract patterns, form views, draft articles | `05 Architecture/`, `06 Outputs/` |

---

## Architecture Analysis Framework

Every system analysis must follow:

```text
WHAT → HOW → WHY → TRADE-OFF → REFLECTION
```

### WHAT

* What problem does the system solve?
* Who is it for?
* What is in scope / out of scope?

### HOW

* Architecture overview
* Core components and responsibilities
* Data flow and control flow

### WHY

* Key design decisions
* Constraints and assumptions behind those decisions

### TRADE-OFF

* Advantages and limitations
* What was deliberately not optimized
* Alternative approaches

### REFLECTION

* Personal understanding (clearly labeled as opinion)
* Comparison with similar systems
* What to borrow or improve

Use this framework in `03 Systems/` and `04 Projects/` notes.

---

## Directory Responsibilities

```text
00 Inbox/        Quick capture; content must move out eventually
01 Sources/      Original references
02 Concepts/     One concept per note
03 Systems/      Product and system analysis
04 Projects/     Personal engineering experience
05 Architecture/ Reusable architecture patterns
06 Outputs/      Publication drafts
99 Templates/    Note templates
```

### 00 Inbox

* Capture ideas with minimal friction
* Do not leave permanent notes here

### 01 Sources

* Preserve source context and links
* Record summary and key findings — not conclusions alone

### 02 Concepts

* One concept per file
* Independent, reusable, linked to related concepts and systems

### 03 Systems

* Focus on understanding a specific system end-to-end
* Apply the WHAT → HOW → WHY → TRADE-OFF → REFLECTION framework

### 04 Projects

* Capture design decisions, lessons learned, architecture evolution
* Never store secrets or confidential business data

### 05 Architecture

* Reusable patterns abstracted from real systems
* Name the problem, solution, trade-offs, and typical implementations

### 06 Outputs

* Generated from existing notes, not directly from raw sources
* Preferred path: Source → Concept → System → Architecture → Output

### 99 Templates

* Use templates when creating new notes
* Do not modify templates unless asked

---

## Note Types

### Source Note

```text
Source
Author
Link
Reading Date
Summary
Key Findings
Questions
Related Concepts
```

### Concept Note

```text
What Is It
Problem It Solves
Core Ideas
Related Concepts
Related Systems
Common Misunderstandings
My Understanding
```

### System Analysis

```text
Summary
WHAT
HOW
WHY
TRADE-OFF
REFLECTION
Related Concepts
Related Patterns
```

### Architecture Pattern

```text
Pattern
Problem
Solution
Advantages
Limitations
Typical Implementations
Applicable Scenarios
Related Systems
```

### Output Draft

```text
Target Platform
Audience
Core Thesis
Outline
Draft
References
```

---

## Obsidian Linking Rules

Use wiki links when relationships exist:

```md
[[AI Agent]]
[[MCP]]
[[Codex]]
[[Leader Worker Pattern]]
```

Rules:

* Every note should link to at least one related note when applicable
* Link concepts to systems, systems to patterns, patterns back to concepts
* Prefer existing note titles; ask before creating duplicates
* Use descriptive link targets, not generic names

Example — Codex system note should link:

```md
[[AI Agent]]
[[MCP]]
[[Computer Use]]
[[Agent Loop]]
```

---

## Naming Conventions

Use descriptive, stable names:

```text
MCP.md
AI Agent.md
Computer Use.md
Leader Worker Pattern.md
Codex Architecture.md
KubeRay Architecture.md
```

Avoid:

```text
note1.md
tmp.md
new.md
test.md
```

* Dates are acceptable for source notes
* Do not use dates as primary identifiers for concept notes

---

## Writing Principles

1. Prefer Chinese for explanations; use English for standard technical terms.
2. Rewrite in your own words — do not copy-paste without synthesis.
3. Separate facts from opinions. Label opinions explicitly.
4. Be concise and structured.
5. Link related notes.
6. Optimize for future reuse, not one-time reading.
7. Preserve important context and source attribution.

Preferred style:

```text
MCP is a protocol for connecting tools and resources to AI Agents.
Its primary goal is to standardize how tools and contextual resources are exposed to agents —
not merely to invoke functions.
```

Avoid:

```text
MCP is very important.
MCP is the future.
Everyone should learn MCP.
```

Prefer reasoning over slogans.

---

## Security Rules

Never store:

* API keys, access tokens, passwords, credentials
* Private certificates or internal secrets
* Sensitive customer data

Use placeholders:

```text
<API_KEY>
<ACCESS_TOKEN>
<PRIVATE_REPO>
<INTERNAL_API>
```

---

## Git Rules

Before committing:

* No secrets in content or filenames
* Meaningful filenames
* Wiki links added where relationships exist
* Preserve existing content; avoid unnecessary rewrites

Suggested commit messages:

```text
docs: add MCP concept note
docs: add Codex architecture analysis
docs: add leader worker pattern
docs: update AI platform notes
chore: reorganize knowledge base structure
```

Obsidian-specific files: respect `.gitignore`. Do not commit `workspace.json` or other per-device state.

---

## Agent Behavior

When adding knowledge:

1. Identify the correct note type and directory.
2. Follow the required template.
3. Add wiki links to related notes.
4. Preserve existing notes; prefer incremental edits.
5. Ask before deleting or restructuring content.

When processing new material:

1. Create a Source note (Acquire).
2. Extract Concepts (Abstract).
3. Write or update System analysis (Reverse).
4. Cross-check and update (Review).
5. Suggest Architecture patterns and Output topics (Articulate).

When working from another repository (e.g. a monorepo via symlink):

* Extract insights into this knowledge base using the evolution model
* Do not copy source code wholesale unless documenting a specific snippet for analysis
* Link back to external references, not internal paths that others cannot access

When uncertain:

* Ask which directory and note type to use
* Propose links to existing notes before creating new ones
* Prefer updating an existing note over creating a duplicate

---

## Long-Term Goal

This repository should evolve into:

* A personal knowledge graph across AI Agent, AI Infrastructure, and AI Platform
* A reusable architecture pattern library
* A technical writing pipeline

The objective is not note-taking.

The objective is building a **reusable technical thinking system**.
