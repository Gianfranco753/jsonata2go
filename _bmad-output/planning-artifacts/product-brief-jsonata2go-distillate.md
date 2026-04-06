---
title: "Product Brief Distillate: jsonata2go"
type: llm-distillate
source: "product-brief-jsonata2go.md"
created: "2026-04-06"
purpose: "Token-efficient context for downstream PRD creation"
---

# Product Brief Distillate: jsonata2go

Dense context from discovery + brief. Pair with `product-brief-jsonata2go.md` for narrative; use this for PRD/scoping agents.

## Product intent

- **Name/workspace:** jsonata2go (repo was effectively **greenfield** for product code at discovery time).
- **Core deliverable:** **Transpiler in Go** that turns **JSONata source files** into **generated Go functions**: **typed input struct → typed output struct**.
- **Integration:** **`go generate`** invoked with a **path to a JSONata file** (or project convention around that path) produces **compile-ready Go** (exact directive shape **TBD** in convention spec).
- **Non-goal:** **JSON serialization/deserialization** and wire/API design are **explicitly out of scope**; callers own `[]byte`/JSON ↔ structs.

## Requirements hints (from user)

- **Parity ambition:** Not a permanent “subset-only” product—**full language coverage** is the **north star**, driven by **official** material.
- **Conformance references:** [jsonata-js test tree](https://github.com/jsonata-js/jsonata/tree/master/test), [JSONata docs](https://docs.jsonata.org/overview.html), [try.jsonata.org](https://try.jsonata.org) for expected I/O behavior.
- **Benchmark ports (capability floor / alignment):** At least **[xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go)** (successor narrative to **blues/jsonata-go**) and version/direction alignment with **[jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)** (multi-version API, e.g. `Open("v2.0.6")`, versioned packages).
- **Typing contract:** **Convention-based** binding of JSONata inputs/outputs to Go structs (file layout, names, or generate directives—**undocumented detail** pending design).
- **Custom functions:** Implemented in **Go**, not interpreted JSONata strings; generated code must **call real Go** symbols per agreed binding.
- **Interpreter policy:** **No fallback** to interpreted evaluation in **shipped/generated** transform code—**no** “if unsupported, eval the original expression string.”
- **Parser reuse allowed:** May use **xiatechs/jsonata-go** *or* **jsonata-go/jsonata** **only** to get **AST / parse tree / IR** for codegen; **emitted product** remains **plain compiled Go**, not “bundle JSONata runtime for this expression.”

## Rejected / excluded directions

- **Runtime interpreter in output:** Rejected—user insisted **compiled Go only**, no degrade-to-eval.
- **Scope expansion to JSON I/O:** Rejected for v1 product boundary (keeps generator focused and testable on **pure transforms**).
- **Hand-maintained duplicate Go as the source of truth:** Rejected as **status quo** to replace; JSONata remains **spec language**.

## Technical context

- **Build-time vs run-time:** Parser dependency is conceptually **build-time** (or codegen-time); clarify in PRD whether **any** small runtime helper library is allowed for **non-string-eval** semantics (brief allows **named** hooks—**must not** reintroduce full JSONata interpretation).
- **Parser selection:** **Undecided** between [xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go) and [jsonata-go/jsonata](https://github.com/jsonata-go/jsonata). **Spike** should rank: (1) exported/stable **AST/IR** access, (2) **JSONata version** pinning vs exerciser, (3) **maintenance/test** story, (4) **adapter** boundary for swap.
- **License:** Both candidate parsers described as **MIT** on their repos—**re-verify** at `go get` time; jsonata2go **OSS intent** after private validation.

## Competitive / landscape intelligence

- **Go ports** ([xiatechs/jsonata-go](https://github.com/xiatechs/jsonata-go), [jsonata-go/jsonata](https://github.com/jsonata-go/jsonata)) solve **runtime JSONata**; **jsonata2go** differentiates on **static codegen + typed structs + no interpreter in output path**.
- **Upstream JSONata** ([embedding docs](https://docs.jsonata.org/embedding-extending)) emphasizes **runtime compile** to internal form—**not** a first-party “transpile user expressions to Go” product; **jsonata2go** fills that gap for Go.

## Users and GTM (minimal)

- **Now:** **Solo author** as integrator and correctness driver.
- **Later:** **Open source** after the toolchain is **proven in use**; target consumers are **Go teams** wanting **auditable** transforms (data/integration pipelines, backends).

## Success signals (measurable)

- **Coverage:** Progress against [jsonata-js tests](https://github.com/jsonata-js/jsonata/tree/master/test) + docs scenarios; **tracked** “supported today” vs **target** parity.
- **Quality bar:** Generated code passes **`go vet` / `staticcheck`**; **no hidden** string-level JSONata eval in generated hot paths.
- **DX:** `go generate` produces a **working** function for **convention-bound** types; **clear errors** when a construct is unsupported.
- **Release:** **Compatibility matrix** (JSONata language version × language features × generator version).

## Open questions (unresolved at distillate time)

- **Exact `go generate` directive** syntax and **file extensions** for JSONata sources.
- **Written convention spec** for **input/output type** resolution (packages, names, tags, multiple outputs).
- **Pinned JSONata version(s)** when **xiatechs** vs **jsonata-go/jsonata** semantics or supported surface **diverge**.
- **Dynamic features:** How `$eval`, highly dynamic typing, async, etc. map to **static Go**—**compile error**, **narrowed semantics**, or **small approved runtime helpers** (still **not** full JSONata eval).
- **Whether any shared runtime package** is acceptable for **helpers** (e.g. generic array/object operations) vs **fully inlined** generated code—brief allows documenting **lifted vs helper vs reject** per construct.

## Scope signals

- **In:** Codegen, struct-to-struct transforms, custom func binding in Go, parser spike, CI against fixtures, OSS path.
- **Out:** JSON marshal/unmarshal, HTTP APIs, non-Go targets (stretch only), interpreter fallback.

## PRD prompts (optional seeds)

- Define **MVP slice** of JSONata constructs that still delivers **real value** while **honesty-labeling** unsupported forms.
- Specify **adapter interface** for parser + **acceptance tests** that prove generated code matches **reference** (JS or chosen port) on **fixture subset**.
- Document **threat model** for “no interpreter”: what **forbidden patterns** in generated code would violate the guarantee.
