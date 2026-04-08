---
stepsCompleted: [1, 4]
inputDocuments: []
session_topic: 'jsonata2go — type pipeline and scope (JSONata → Go)'
session_goals: 'Clarify whether/how to generate structs (e.g. from JSON Schema), repo boundaries, and prerequisites for the code generator'
selected_approach: 'informal conversational (formal technique library not used)'
techniques_used: []
ideas_generated:
  - 'Treat JSON Schema → Go structs as optional upstream; prefer existing generators in docs/examples first'
  - 'Keep jsonata2go core focused on JSONata → typed transform Go, not owning full schema codegen'
  - 'Separate concern ≠ separate repo by default; split repo only if the struct tool is a standalone product'
  - 'Typed Go mapping requires input/output types before final codegen; source can be hand-written or generated elsewhere'
  - 'Output types may need explicit per-transform schema or conventions—JSONata output is not always inferable from input schema alone'
context_file: ''
session_active: false
workflow_completed: true
---

# Brainstorming Session Results

**Facilitator:** Gcossani (self-facilitated with AI)
**Date:** 2026-04-08

## Session Overview

**Topic:** jsonata2go — architecture and boundaries around **typed Go** emitted from **JSONata**, including whether to build or bundle **JSON Schema → struct** generation.

**Goals:** Decide how struct generation fits the product, whether it belongs in another repo, and whether **types must exist before** the generator runs.

### Session setup

Session followed an **informal Q&A path** (no formal rotation through the CSV technique library). Outcomes are **convergent decisions and implications**, not a large divergent idea inventory.

## Themes and synthesis

**Theme 1: Core vs adjacent tooling**

- jsonata2go’s differentiator is **JSONata → reviewable, compiled Go** between **known types**.
- **JSON Schema → Go** is a crowded, solved-adjacent space; **documenting** a recommended toolchain beats **rebuilding** it inside the core unless there is a clear, unique value.

**Theme 2: Packaging and repos**

- **Separate module or package** can live in the **same repo** until reuse or release cadence justify extraction.
- **Another repo** when the struct generator is intentionally **standalone**, **multi-consumer**, or needs **independent versioning**.

**Theme 3: Type prerequisites**

- For the **typed mapping** product vision, **something** must supply **input and output types** (or equivalent) so emitted code is **checkable** and **stable**.
- **Optional** “no structs first” paths (`map[string]any`, heavy inference) **trade away** the PRD-style benefits unless tightly constrained.

## Prioritization (session conclusions)

| Priority | Conclusion |
|----------|------------|
| **High** | **v0:** Assume types exist or are produced **upstream**; jsonata2go consumes them. |
| **High** | **Do not require** a second repo for schema tooling unless it becomes its own product. |
| **Medium** | If you ship a helper, start as **same-repo module** or **thin docs + example** with an existing generator. |
| **Medium** | Treat **output shape** explicitly (schema per transform, user-supplied output type, or documented supported subset)—do not assume input schema alone defines output. |

## Action planning

1. **PRD / brief:** Add a short subsection: **type contract** (who owns input/output types, recommended optional JSON Schema → Go tools, out-of-scope for core).
2. **Customer zero path:** Define one **end-to-end example**: types (hand-written or from schema tool) → JSONata file → `go generate` → transform function.
3. **Revisit “own struct gen”** only after **real friction** with existing tools (naming, nullable, `oneOf`, JSONata-specific tags)—then decide **module in-repo** vs **new repo**.

## Session summary and insights

**Achievements**

- Aligned **scope**: parsing JSONata stays out of “hot path”; **type story** is about **contracts**, not necessarily **jsonata2go-owned codegen**.
- Clarified **repo strategy**: default **monolith / same repo**; **split** on evidence.
- Locked **prerequisite**: **types before codegen** for the **typed Go** promise.

**Reflection**

- The open design pressure is **output typing**, not input typing—worth a follow-up design spike when you pick **first real transforms**.

---

**Session status:** **Complete.** Document stored for future reference; optional follow-up: formal brainstorming with technique library if you want a **divergent** pass (e.g. DX, failure modes, ecosystem).
