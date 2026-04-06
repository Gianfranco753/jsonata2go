---
title: "Product Brief: jsonata2go"
status: "complete"
created: "2026-04-06"
updated: "2026-04-06"
inputs:
  - "User discovery (conversation)"
  - "_bmad/bmm/config.yaml"
  - "Public references: jsonata-js test suite, docs.jsonata.org, try.jsonata.org"
---

# Product Brief: jsonata2go

## Executive Summary

**jsonata2go** is a **Go code generator**: it reads **JSONata** transformation expressions and emits **ordinary, compiled Go functions** that map a **typed input struct** to a **typed output struct**. The goal is to keep **JSONata as the specification language** while delivering **native Go** at build time—no runtime interpretation of the original expression string, and **no fallback** to an embedded JSONata interpreter in generated code.

The project is **greenfield** with full control over layout and tooling. **Customer zero is the author**; after validation, the intent is to **open source** the toolchain. Correctness will be anchored to the **official JSONata conformance story** ([jsonata-js tests](https://github.com/jsonata-js/jsonata/tree/master/test), [documentation](https://docs.jsonata.org/overview.html), [try.jsonata.org](https://try.jsonata.org)), while **parity targets** are explicitly benchmarked against **mature Go ports**—at minimum the capability level of **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** (evolution of blues/jsonata-go) and alignment with the direction of **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)** for newer JSONata versions.

## The Problem

Teams that standardize on **JSONata** for data mapping get a **compact, portable spec**, but **production Go** today typically means **runtime evaluation** (`Compile` + `Eval` over `interface{}` or generic JSON trees). That pattern **carries interpreter cost**, **weak compile-time guarantees** between spec and Go types, and **harder reasoning** about performance and dependencies in security-sensitive pipelines.

Maintaining **hand-written Go** that mirrors JSONata is **error-prone** and drifts from the spec. What is missing is a **bridge**: the same mental model and test corpus as JSONata, with **generated Go** that is **idiomatic, typed, and statically compiled**.

## The Solution

A **transpiler** integrated with **`go generate`**: point it at a **`.jsonata` (or similar) file** and emit a **Go function** (or small package fragment) that performs the transform **purely in Go**. **Serialization and deserialization** (`encoding/json`, struct tags, wire formats) stay **out of scope**—callers pass **structs**; the generator focuses on **the transform logic**.

**Custom functions** are not interpreted strings: they are **implemented in Go** and **wired by convention** (or an agreed registration pattern) so generated code calls **real functions**.

**Parser reuse:** Use **either** **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** or **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)** **only** to obtain a **parse tree / AST** (or equivalent internal IR) from the expression text—**which library is better is not decided here**; it must be chosen with the criteria in **Open decisions** (below). **Emitted code must not** re-bundle expression evaluation as “run JSONata on this string.” The **artifact developers ship** is **compiled Go**.

**Typing:** Input and output shapes follow a **project convention** (naming, package layout, or generator directives—details to be defined in implementation) so `go generate` knows which Go types correspond to the JSONata input and expected output.

## What Makes This Different

| Alternative | Limitation for this vision |
|-------------|----------------------------|
| Go JSONata ports (e.g. [xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go), [jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)) | **Runtime** evaluation; strong for compatibility, not for **zero-interpreter** typed pipelines |
| Hand-written Go | **Diverges** from JSONata specs; expensive to keep aligned with **official tests** |
| This project | **Spec in JSONata**, **implementation in generated Go**, **conformance-tested** against upstream material |

The **moat is execution**: proving **broad language coverage** with **strict “no interpreter in output”** and **developer-grade** `go generate` ergonomics—not a proprietary language.

## Who This Serves

- **Primary (now):** You, as **author and first integrator**, proving the toolchain on real transforms and tightening the generator until it is trustworthy.
- **Next:** **Open-source consumers** (Go backends, data planes, integration teams) who want **auditable, compiled** mappings with **JSONata-expressible** behavior.

## Success Criteria

- **Correctness:** Progressively **cover the official JSONata test corpus** and **documented semantics** ([docs](https://docs.jsonata.org/overview.html)); track **explicit parity** against **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** and version goals consistent with **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)**.
- **Output quality:** Generated functions are **go vet / staticcheck clean**, **readable enough to debug**, and **free of hidden string-level JSONata eval** in the hot path.
- **Workflow:** `go generate` from a **JSONata file** produces a **working** transform for **convention-defined** input/output structs.
- **Extensibility:** **Custom functions** are **first-class Go**, with a clear **binding** story from JSONata call sites to Go symbols.
- **Release path:** After private validation, **publish as OSS** with a clear **license**, **contribution**, and **compatibility matrix** (JSONata version × supported language features).

## Scope

**In scope**

- JSONata → **Go source** codegen via **`go generate`**.
- **Typed** struct in → **typed** struct out (per **convention**).
- **Full-language ambition:** work through the **entire** surface area implied by official tests and port parity targets—not a toy subset as the end state.
- **Custom functions** supplied as **Go**, linked per project rules.
- **Parser leverage** from **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** *or* **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)** for **parse/AST (or IR) extraction only** (license and coupling to be managed deliberately).

**Out of scope (explicit)**

- JSON **marshaling/unmarshaling** and API design for HTTP/gRPC payloads.
- **Interpreter fallback** in generated or runtime-shipped transform code (no “degrade to eval”).
- Commitment to **non-Go** targets (future possibility only).

**Open decisions (to resolve during architecture/PRD)**

- **Parser implementation:** Pick **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** or **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)** after a short **spike** (same criteria for any future swap). **Decision drivers** (in priority order):
  1. **Parse surface:** Stable access to **AST / parse tree** (or an IR you can lower to Go) without forking; if only `Compile`+`Eval` is public, that library is a **poor fit** unless you vendor and expose internals deliberately.
  2. **Spec/version alignment:** Ability to track the **JSONata language version** you want to codegen against (note [jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)’s **versioned layout**—e.g. `v154`, `v206` and `Open("v2.0.6")`-style API—which may simplify **multi-version** or **exerciser-matching** goals).
  3. **Operational fit:** **go.mod** footprint, **maintenance** signal, **test-suite** runner story vs [jsonata-js tests](https://github.com/jsonata-js/jsonata/tree/master/test), and **MIT** (or compatible) **license** for embedding as a **build-time** dependency.
  4. **Coupling cost:** Frequency of **internal API** reliance—prefer an **adapter interface** in jsonata2go so the parser can be **swapped** if one fork stagnates or the other gains better AST access.

- **Pinned JSONata version(s)** and a **single source of truth** for “parity” when the two ecosystems differ.
- **Convention spec** for type binding (file layout, comment directives, naming).
- **Static codegen limits:** JSONata features that are **dynamic by nature** may require **narrowing rules**, **compile-time errors**, or **explicit runtime hooks**—without reintroducing full expression interpretation; these must be **named and tested**.

## Risks and Mitigations (review synthesis)

- **Ambitious parity:** Treat **official tests** as a **tracked backlog**; ship **incremental** language subsets with **explicit** “supported today” flags until coverage is complete.
- **Dynamic semantics vs static Go:** Document **which constructs** are **fully lifted** to Go, which require **helper runtime** (still not JSONata string eval), and which are **rejected at codegen** with actionable errors.
- **Parser coupling:** Isolate **parse adapter** behind an internal interface; vendor or pin the **chosen** parser module; monitor **LICENSE** compatibility (both candidates are **MIT** per their repositories).
- **Solo maintainer:** Automate **regression** against upstream **fixture** snapshots early so progress is **measurable** and **CI-guarded**.

## Vision

If this succeeds, **jsonata2go** becomes the **default path** for Go teams who want **JSONata-authored** transforms with **production-native** code: **fast**, **typed**, **diffable**, and **provably aligned** with the **same** specifications and exerciser behavior the ecosystem already trusts ([try.jsonata.org](https://try.jsonata.org)). Longer term, the same architecture could inspire **other static targets**, but **Go excellence** comes first.
