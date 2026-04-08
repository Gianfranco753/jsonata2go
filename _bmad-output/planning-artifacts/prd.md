---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-02b-vision
  - step-02c-executive-summary
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
  - step-07-project-type
  - step-08-scoping
  - step-09-functional
  - step-10-nonfunctional
  - step-11-polish
  - step-12-complete
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-jsonata2go.md
  - _bmad-output/planning-artifacts/product-brief-jsonata2go-distillate.md
workflowType: prd
documentCounts:
  briefCount: 2
  researchCount: 0
  brainstormingCount: 0
  projectDocsCount: 0
classification:
  projectType: developer_tool
  domain: general
  complexity: medium
  projectContext: greenfield
---

# Product Requirements Document - jsonata2go

**Author:** Gcossani
**Date:** 2026-04-07

**Traceability (document flow):** Executive Summary (vision) → Success Criteria & Product Scope → User Journeys → Innovation & Novel Patterns → Developer Tool Specific Requirements → Project Scoping & Phased Development → Functional Requirements (**FR1–FR34**) → Non-Functional Requirements.

**Section index:** Executive Summary · Project Classification · Domain-Specific Requirements · Success Criteria · Product Scope · User Journeys · Innovation & Novel Patterns · Developer Tool Specific Requirements · Project Scoping & Phased Development · Functional Requirements · Non-Functional Requirements

## Executive Summary

**jsonata2go** is a **Go-oriented code generator**: it reads **JSONata** transformation sources and emits **ordinary, compiled Go** that maps a **typed input struct** to a **typed output struct**. The intended workflow is **JSONata files in repo → `go generate` → plain Go** that teams can diff, benchmark, and ship—**reviewable Go from JSONata sources**. **JSONata remains the specification language**; **generated Go is what runs in production**.

**Problem:** Teams that standardize on **JSONata** get a **compact, portable spec**, but production Go today often means **runtime evaluation** over generic trees, with **interpreter overhead**, **weak compile-time ties** between spec and types, and **hand-written Go** that **drifts** from the spec.

**Target users (initial):** The **author and first integrator** proving the toolchain on real transforms (**customer zero**); **later**, **open-source adopters** (Go backends, data planes, integration teams) who need **auditable, compiled** mappings that stay aligned with **JSONata semantics** and the **official exerciser story**.

**Scope boundary:** **Serialization** (`encoding/json`, wire formats, HTTP/gRPC) is **out of scope**—callers own marshaling; the generator focuses on **pure transform logic** between structs. **Custom functions** are **implemented in Go** and bound by an explicit project convention, not as interpreted strings.

**Runtime guarantee:** Generated transform code must **not evaluate the original JSONata source string at runtime** (no string-level JSONata eval in the hot path). **Parser dependencies are build-time only** for **AST/IR** extraction behind a **swappable adapter**.

### What Makes This Special

- **Static output, not a runtime port:** The artifact is **generated Go**, not “run JSONata on this string” in the hot path.
- **Conformance as the bar:** Correctness is **tracked against** the **official JSONata test material** and documentation. **Mature Go ports** are **capability and regression baselines**, not **runtime dependencies** of the shipped product.
- **Provable quality:** Generated code should be **reviewable** (`go vet` / **staticcheck** clean, readable enough to debug).
- **Extensibility without breaking the model:** **Custom functions** are **first-class Go**; **parser reuse** stays **build-time** for **AST/IR** behind the same **adapter** boundary.

## Project Classification

Classification: **developer tool**, **general** domain, **medium** engineering complexity, **greenfield**—authoritative fields in document frontmatter.

## Domain-Specific Requirements

**Regulated-industry** requirements (e.g. HIPAA, PCI) **do not apply**—this is **general** developer tooling. **Licensing, provenance, and production vs build-time clarity** are covered under **NFR-S1** and **FR23** / **FR30**. **Product boundaries** such as serialization and wire APIs remain as stated in the **Executive Summary** (not duplicated here).

## Success Criteria

### User Success

- **Trustworthy transforms:** A developer can point **`go generate`** at a **JSONata source**, bind **input/output structs** via the project convention, and get a **working** typed transform without hand-maintaining parallel Go.
- **Customer-zero “done”:** At least **one non-trivial transform** (or a **documented reference pipeline**) runs **fully** on **generated code**, with **no eval fallback** and **no** manual escape hatch to interpreted JSONata for that path.
- **Reviewable output:** Generated code is **readable enough to debug** and passes **`go vet`** and **staticcheck** (or the project’s agreed static analysis), so teams can **diff** and **review** changes like normal Go.
- **Clear failure modes:** When a construct cannot be lifted to static Go, the generator yields **actionable errors** (not silent fallback to **string eval**).
- **Extensible behavior:** **Custom functions** are wired to **real Go** symbols via an explicit binding story; call sites in generated code are **obvious** in review.

### Business Success

- **Customer zero validation:** The **author** successfully uses the toolchain on **real transforms** and tightens the generator until it is **trustworthy** for day-to-day use.
- **Open source readiness:** Ship as **OSS** with a clear **license**, **contribution** path, and **public compatibility matrix** (JSONata language version × supported features × generator version).
- **Adoption signal (post-OSS):** Concrete milestones such as **first external contributor** or **first downstream repo** outside the author’s control (even if **N=1**), in addition to generic “third-party teams can adopt.”

### Technical Success

- **Conformance:** Progress **against the official JSONata test corpus** and **documented semantics**; **regression** is **CI-guarded** with **fixture** snapshots tied to upstream material.
- **Risk-tiered testing:** Separate **P0** scenarios (must never regress; merge gate) from **full-corpus / stretch** coverage, so progress is meaningful—not a single vanity percentage.
- **Deterministic gates:** Conformance and codegen tests are **deterministic**; **flaky runs are treated as product risk**, not noise.
- **Baseline alignment:** **Explicit parity** against **mature Go ports** as **regression baselines** (not runtime dependencies)—capability tracked over time.
- **No-interpreter guarantee (enforceable):** Generated (and any approved small **runtime helpers**) do **not** **evaluate the original JSONata source string** in the transform hot path; enforcement includes **documented checks** (e.g. **lint rules**, **codegen self-check**, **forbidden-pattern** gates) where feasible.
- **Maintainable coupling:** **Parser adapter** boundary allows **swapping** the build-time parser without rewriting the whole codegen; validated by tests that **substitute** a parser fixture or stub.

### Measurable Outcomes

- **Coverage metrics:** **P0 vs backlog** (or **smoke vs full corpus**) tracked explicitly; **P0 required for merge** to main/release branches.
- **Quality gates:** **CI** runs **generator tests**, **generated-code** checks, **conformance** runs, and **deterministic** jobs on a **pinned** or **matrixed** JSONata version policy.
- **Developer experience:** Time to go from **JSONata file change** to **regenerated, compiling Go** in a **reference workflow** (documented path).

## Product Scope

### MVP - Minimum Viable Product

- **End-to-end path:** **`go generate`** (or equivalent documented invocation) from a **JSONata file** to **generated Go** for a **convention-bound** **input struct → output struct**.
- **Pinned language version:** MVP targets **one pinned JSONata language version** on the compatibility matrix (multi-version is **post-MVP** unless explicitly pulled in).
- **Honest subset:** A **defined** initial subset of JSONata with **explicit** unsupported-form errors; **no** “degrade to eval” in generated code.
- **Conformance seed:** A **repeatable** harness with a **P0** suite derived from **official** fixtures (or a clearly scoped excerpt); **P0** is **CI-gated**.
- **Parser path behind adapter:** A **chosen** build-time parser behind the **adapter**, with tests proving the adapter is **substitutable** (fixture/stub acceptable).

### Growth Features (Post-MVP)

- **Broader language coverage** toward the **full** surface implied by **official tests** and docs.
- **Multi-version matrix** and richer **version policy** as the generator matures.
- **Richer type binding** (multiple outputs, more convention options) as real integrators need them.
- **Stronger static guarantees** (additional checks, better diagnostics, performance work on codegen output).

### Vision (Future)

- **Broad conformance** with the **official JSONata exerciser** story and **documented semantics**, with a **public compatibility matrix** and **predictable** upgrade notes.
- **Ecosystem fit:** jsonata2go is a **credible default** for Go teams who want **JSONata-authored** transforms with **production-native**, **diffable** Go—without trading away **ecosystem-trusted** semantics.

## User Journeys

### Journey 1 — Author (customer zero): first successful pipeline

**Persona:** Solo author integrating jsonata2go into a real service that today either uses **runtime JSONata** or **hand-written Go**.

**Opening:** They have a **JSONata file** that encodes a mapping they trust, but they want **compiled Go**, **typed structs**, and **reviewable diffs** on upgrade.

**Cold start:** Documentation supports a **zero-to-first** path: from **clean checkout** to **first successful `go generate` + compile** along a **published reference** (exact **time-to-first-green** target is TBD and should be measured).

**Docs integrity:** **Documented** invocation (**flags**, **paths**, **layout**) matches what **CI** runs and what **reference / golden examples** use—**no drift** between README and pipeline (if docs lie, onboarding fails before real work starts).

**Rising action:** They add the **generator** to the repo, define **input/output structs** per the **documented convention**, wire **`go generate`**, and run the build. They call the **generated function** from production code with **structs** they own (marshaling stays outside).

**Climax:** A **non-trivial transform** runs **end-to-end** on **generated code only**—no **string eval** path—so they trust they can **benchmark** and **audit** it like any other Go package.

**Resolution:** The mapping **lives in JSONata**; **Go stays the implementation of record** in git, with a **clear upgrade story** when the expression changes.

### Journey 2 — Author: unsupported language feature (edge / recovery)

**Opening:** They extend the JSONata and hit a construct the **MVP subset** cannot lift to static Go (or hits a **known limitation**).

**Rising action:** They run **`go generate`** and get a **specific error** pointing at **source location** and **what is unsupported**, not a **silent fallback** or **runtime eval**.

**Climax:** They choose a **real workaround**: narrow the expression, add an **approved custom function** in Go, or **defer** until the feature is supported—without **shipping** a hidden interpreter path.

**Resolution:** Trust is preserved: **failure is explicit** and **policy-aligned** with the **no-eval guarantee**.

### Journey 3 — Maintainer / CI (“operations” for this product)

**Persona:** **CI** and **repo maintainers** guarding **regressions** on generated output and **conformance**.

**Opening:** A PR changes the **generator**, **fixtures**, or **pinned JSONata** policy.

**Rising action:** **P0 conformance** and **static checks** run on the PR. **Generated code** is **diffed** intentionally; **deterministic** jobs either **pass** or **fail** the merge.

**Breaking changes:** A release may **change generated API shape** or **file layout**; consumers expect **semver / changelog** discipline for **generated packages** and clear **migration notes** when upgrades are not drop-in.

**Release / OSS publish:** Before **tagging** a release: **P0** is green; **changelog** and **compatibility matrix** are updated; **module semver** aligns with **generated API / layout** promises—releases are not “just a git tag.”

**Climax:** A **regression** is **caught before release** because **P0** is **merge-gated**, not “we’ll notice later in production.”

**Resolution:** **Velocity with guardrails**: contributors know **what must stay green** and **why**.

### Journey 4 — OSS adopter (integrating team)

**Persona:** A **Go team** that did not author jsonata2go but pulls it as a **dependency** / uses the **CLI** pattern in their monorepo.

**Opening:** They need **JSONata-specified** behavior with **Go-native** artifacts for **security or performance** reasons.

**Rising action:** They follow **published docs**: **install**, **compatibility matrix**, **convention** for types, **regenerate** on spec changes.

**Security / review:** **Reviewers** can see **what ships in production**—**generated Go** plus any **documented small runtime helpers**—versus **build-time-only** parser/tooling; **supply chain** and **audit** concerns are addressable without reverse-engineering the pipeline.

**Climax:** They can **onboard** without reverse-engineering internals—**matrix + examples** match **their pinned language version**.

**Resolution:** Adoption is **repeatable**; **upgrades** are **predictable** using **published** compatibility notes.

### Journey 5 — Contributor / support: triage a conformance mismatch

**Persona:** **Contributor** (or **author** wearing a support hat) when **official fixture** behavior and **generated output** disagree.

**Opening:** **CI** flags a **P0** failure or a user reports a **semantic** mismatch.

**Rising action:** They **minimize** a repro from **fixtures**, identify whether the bug is **codegen**, **adapter**, or **unsupported semantics** documentation.

**Climax:** Fix lands with a **regression test** so the **class of bug** stays **covered**.

**Resolution:** The project stays **honest**: **conformance** is **actionable** and **trending up**, not tribal knowledge.

### Journey 6 — Upgrade / migration (generator or language pin)

**Persona:** **Author** or **adopter** moving to a **new generator version** or a **new pinned JSONata language** on the matrix.

**Opening:** They intend to **upgrade** and expect **controlled churn** in **generated output** or **API surface**.

**Rising action:** They read **release notes**, **compatibility matrix**, and **migration guidance**; they regenerate and **review diffs**; they validate using the same **P0** suite and gates as **mainline CI**—migration is not a **local-only** shortcut around project quality bars.

**Climax:** Either the upgrade is **safe** (expected diffs, tests green) or the team **blocks release** until **breakage** is understood—without **surprise** runtime behavior changes hidden from review.

**Resolution:** **Upgrades are a first-class journey**: **diffable Go**, **explicit** policy, and **documented** expectations for **breaking** changes to **generated** APIs.

### Journey Requirements Summary

| Area | What these journeys require |
|------|-----------------------------|
| **Authoring & DX** | Clear **`go generate`** path, **type-binding convention**, **docs** and **examples**, **fast** regen loop; **cold-start** path to **first green** build (**time-to-first-green** TBD, measurable). |
| **Docs & trust** | **Documented** invocation matches **CI** and **golden / reference examples** (flags, paths, layout). |
| **Errors & policy** | **Actionable** codegen errors; **no** silent eval; **documented** unsupported set for MVP. |
| **Quality & CI** | **P0** suite, **deterministic** jobs, **generated-code** checks, **merge** policy; **semver/changelog** expectations when **generated** APIs change. |
| **OSS & adoption** | **License**, **contribution** guide, **compatibility matrix**, **pinned version** story; **reviewer-friendly** clarity on **prod vs build-time** artifacts. |
| **Maintenance** | **Repro** workflow from **fixtures**, **test** patterns for **adapter** and **codegen**, **regression** discipline. |
| **Lifecycle & upgrades** | **Migration** notes, **matrix** alignment, **diff review** workflow for **generator** and **language pin** bumps; **same P0 gates** as CI for **migration**. |
| **Release** | **P0** green, **changelog** + **matrix** updated, **module semver** aligned with **generated API** promises before **tagging**. |

## Innovation & Novel Patterns

### Detected Innovation Areas

- **Spec/implementation split with static output:** Use **JSONata** as the **authoritative mapping spec** while the **runtime artifact** is **generated Go** between **typed structs**, rather than **interpreting** the expression string in production. The intended **substitutes** teams are choosing against are **runtime JSONata-in-Go** and **hand-maintained Go**—not “replacing” the **JavaScript** reference engine as a deployment artifact.
- **Terminology:** “**JSONata source string**” means the **authoring expression text** (the spec on disk), not **JSON strings** as runtime **data values**—this keeps the **no-eval** invariant unambiguous in reviews and docs.
- **Provable product invariant:** “**No JSONata string eval** in the generated transform hot path” is treated as an **engineering guarantee** backed by **tests and checks**, not a slogan. Success looks like teams treating **generated Go** as **the** integration artifact—not a **view** over a hidden interpreter.
- **Helper-runtime boundary as part of the pattern:** Any **small runtime helpers** must stay **narrowly approved** primitives; anything that becomes **“eval by another name”** is **out of policy** and must be **rejected** or redesigned—this boundary is **core** to the innovation, not an afterthought. **Bright line:** helpers must not implement **mini parsers** for **JSONata-like** syntax or **expression-shaped** control languages; if it smells like **eval**, it is **out**.
- **Conformance-native toolchain:** Progress is measured against **official JSONata material** and **documented semantics**, with **mature Go implementations** used only as **baselines**, not shipped runtime engines.
- **Extensibility without losing the model:** **Custom functions** resolve to **real Go** symbols via an explicit binding model—avoiding “extend by embedding more JSONata strings.”

### Market Context & Competitive Landscape

- **Mature Go JSONata libraries** optimize for **compatibility** and **runtime evaluation**—valuable, but a different product category than **static codegen**.
- **Hand-written Go** diverges from specs; **jsonata2go** targets **one spec** (JSONata) and **one implementation shape** (compiled Go) tied together by **generation**.
- **jsonata-js** and the **official exerciser** remain the **semantic reference** for behavior; **jsonata2go** competes with **Go runtime ports** and **manual duplication** in **deployment**, not with “running JSONata in Node” as the primary integration story for Go services.
- **Upstream JSONata** emphasizes **runtime compilation** to internal forms for embedding—**not** “emit idiomatic Go source for arbitrary expressions” as the primary artifact.

### Validation Approach

- **Official corpus & docs:** Track **official tests** and **documented behavior**; maintain **P0** gates and **deterministic** CI.
- **Differential / reference checks:** For the **supported subset**, compare outputs on a **shared fixture set** against **reference-expected** behavior (per **official** fixtures / **documented** semantics and **approved** baselines)—**semantic parity**, not informal eyeballing. Exact harness mechanics are an implementation detail; the requirement is **demonstrable equivalence** for supported cases.
- **Baseline comparisons:** Use **selected Go ports** as **regression references**, clearly labeled as **non-shipping baselines**.
- **Invariant checks:** Enforce the **no-eval** rule with **lint/codegen self-checks** and **forbidden-pattern** gates where feasible, plus **review** of **generated** output—explicitly to **rebut** the “**hidden interpreter**” failure mode. **Invariant failures are release-blocking** even if other tests are green.
- **Reviewer-readable artifacts:** Generated code stays **diffable** and **inspectable** so the **invariant** can be **audited** without trusting opaque codegen.

### Risk Mitigation

- **Parity risk:** Ship **incremental** language support with **explicit** “supported today” labeling until coverage grows; avoid **silent** partial semantics.
- **Dynamic semantics risk:** Document **lifted vs helper-runtime vs rejected** behaviors; **reject at codegen** when static lowering is unsafe or would require **full** interpretation.
- **Helper creep risk:** Treat **runtime helper** growth as a **product risk**—review regularly so helpers don’t become a **shadow JSONata runtime**; enforce the **bright line** on **mini parsers** and **eval-shaped** helpers.
- **Parser coupling risk:** **Adapter boundary**, **pinned** dependencies, and a **swap** story if a parser path stalls.

## Developer Tool Specific Requirements

### Project-Type Overview

**jsonata2go** is a **developer tool** in the **codegen / SDK** sense: a **Go module** (plus documented **CLI** invocation if applicable) used at **build time** to turn **JSONata sources** into **generated Go**. Per `developer_tool` defaults, **skip** consumer-style concerns (**visual design**, **store compliance**).

### Technical Architecture Considerations

- **Distribution:** Delivered as **Go modules** with a **semver** policy aligned to **generated API** and **layout** promises.
- **Runtime split:** **Build-time** parser/tooling vs **generated code** (and **narrow** approved helpers) in **downstream** binaries; same **no–JSONata-authoring-text-eval** invariant as the **Executive Summary** (generated hot path).
- **Integration:** Primary integration is **`go generate`** (or equivalent documented invocation) in **consumer repos**; **CI** is expected to run **generator** and **static checks** deterministically.

### language_matrix

- **Go:** Publish a **supported Go version range** for the **generator** and for **generated code** (policies may start unified and split later). State a **minimum Go** for each once the implementation relies on **generics** or other version-gated features; align **`go.mod`** **`toolchain`** with **CI** images.
- **JSONata:** MVP ships against **one pinned JSONata language version** on the **compatibility matrix**; **multi-version** is **post-MVP** unless explicitly pulled forward.

### installation_methods

- **Module install:** `go get` / `go install` paths documented for **released** versions; **pseudo-versions** acceptable for dev workflows.
- **Reproducibility:** Encourage **pinned** tool versions in **CI** so **generated** output is **stable** under the same inputs.
- **Optional:** Evaluate and document **`go tool`** / **module tool** workflows when the project adopts a **pinned** tool path without relying on **global** installs.

### api_surface

- **Default integration pattern:** Document a **blessed** default (exact shape TBD): e.g. **`go generate` invokes a versioned generator** with **stable** **CLI flags** and **exit codes** across **minor** versions where possible.
- **Generator surface:** Documented **flags**, **inputs** (JSONata paths, type binding), **outputs** (generated file paths / package layout), and **error** shapes.
- **Generated surface:** **Generated functions** and **types** follow a **stable convention**; **breaking** changes require **semver** + **migration** notes.

### code_examples

- **Reference repo or `examples/` tree:** **Minimal** “happy path” plus **one** **custom function** example and **one** **unsupported construct** error example.
- **Golden examples** must match **CI** invocation (**docs integrity**).

### migration_guide

- **Deterministic codegen:** **Stable** ordering/formatting of **generated files**; standard **generated file** banner (e.g. **DO NOT EDIT**); **diff-friendly** output as a **requirement**, not style-only.
- **Generator upgrades:** **Changelog** entries, **expected diffs**, **P0** rerun expectations, and **rollback** guidance.
- **Language pin bump:** **Matrix** updates, **migration** steps, and **compatibility** notes for **generated** API changes.

### Implementation Considerations

- **Testing:** **Unit** tests for **codegen**, **adapter** substitution tests, **golden** generated output, **P0 conformance** harness, **invariant** checks.
- **Docs:** **Installation**, **quickstart**, **compatibility matrix**, **contributing**, and **release** process tied to **P0** + **invariant** gates.

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP approach:** **Problem-solving / correctness MVP** first: prove **JSONata → generated Go** is **trustworthy** on a **pinned** language version with a **defined subset**, **CI-gated P0**, and **enforceable no-eval** rules—before chasing **full** ecosystem coverage.

**Resource requirements:** Initially **solo maintainer** (author as **customer zero**); skills: **Go**, **language/codegen**, **testing**, **build/release**; **community** scale is **post-OSS**.

### MVP Feature Set (Phase 1)

**Phase 1 definition of done:**

- **Public OSS repo** with a **documented** path from **JSONata file** → **`go generate`** (or documented equivalent) → **compiling** generated Go for **at least one non-trivial transform**.
- **P0** suite **green** in **CI** on **main**; jobs **deterministic**.
- **Compatibility matrix** published with **one pinned JSONata language version** for MVP.
- **No-eval** invariant **enforced** by agreed **checks** (lint/self-check/forbidden patterns as applicable) plus **reviewable** generated output.
- **Capability contract touchpoints:** **FR16** (invariant violations **release-blocking**) and **FR28** (per-release **canonical** supported-surface inventory).

**Core user journeys supported (minimum):**

- **Journey 1** (author **first pipeline**) with **cold start** and **docs/CI alignment**.
- **Journey 2** (**explicit errors** for unsupported constructs).
- **Journey 3** (**P0** + **merge** gates; **release** checklist **minimal** but **real**).
- **Journey 6 (MVP boundary):** **In scope** for **minor generator upgrades** while the **JSONata language pin** stays the same—**P0** must pass. **Changing the JSONata pin** and **large breaking** generator output changes are treated as **major** migration work—**post-MVP** unless explicitly scheduled; **pre-1.0**, consumers should expect **churn** documented in **release** notes.

**Must-have capabilities:**

- **`go generate`** (or documented equivalent) → **generated Go** for **convention-bound** **struct → struct**.
- **Pinned JSONata language version** (single row on matrix for MVP).
- **Honest subset** + **actionable** errors; **no** eval fallback.
- **P0 conformance harness** + **deterministic** CI jobs.
- **Parser adapter** with **substitutable** tests.
- **Invariant enforcement** path (lint/self-check + **reviewable** output).
- **Public docs**: install, quickstart, matrix stub, contributing skeleton.

### Post-MVP Features

**Phase 2 (growth) — sequencing:**

1. **Broader JSONata coverage** toward the **official** corpus; expand **P0** vs backlog tiers.
2. **Multi-version matrix** and **version policy** once coverage strategy is stable (default order **1 → 2** unless multi-version is explicitly needed to unlock a coverage milestone).

**Additional Phase 2 items:**

- Stronger **diagnostics** and **performance** characteristics of generated code.
- **OSS adoption** mechanics: **first contributor** path, **issue** templates, **release** rhythm.

**Phase 3 (expansion / vision):**

- **Broad** alignment with **try.jsonata.org** / **docs** semantics for supported versions.
- **Ecosystem position**: credible **Go** path for **JSONata-authored** → **compiled** mappings.

### Risk Mitigation Strategy

**Technical risks:** **Parity** and **dynamic semantics**—mitigate with **incremental** support labels, **reject** at codegen, **strict** helper boundary, **adapter** swap story.

**Market risks:** Mitigate with **customer-zero** real transforms, **diffable** output, **conformance** credibility, and explicit milestones for **first external user** or **first external contributor** (not “investor” milestones).

**Resource risks:** **Solo** bandwidth—mitigate with **automation** (fixtures, CI), **ruthless MVP** scope, **clear** unsupported surface, and a **single Phase 1 definition of done**.

## Functional Requirements

### Transformation specification & lifecycle

- **FR1:** A **developer** can maintain transformation logic as **JSONata source artifacts** suitable for version control and review.
- **FR2:** A **developer** can invoke the **code generator** through a **documented** build-time interface.
- **FR3:** A **developer** can **regenerate** implementation artifacts when JSONata sources change.

### Structural binding & generated transforms

- **FR4:** A **developer** can bind JSONata inputs and expected outputs to **declared structural types** using a **documented convention**.
- **FR5:** A **developer** can obtain a **callable transform** that maps bound input values to bound output values **without evaluating JSONata authoring text in the transform execution path**.
- **FR6:** A **developer** can **inspect** generated implementation artifacts as ordinary source intended for audit and diff review.

### Extension via native functions

- **FR7:** A **developer** can attach **native implementations** to JSONata function call sites using a **documented registration/binding** mechanism.
- **FR8:** A **reviewer** can see **which native symbols** satisfy **which** JSONata function references from generated artifacts or **documented** linkage.

### Diagnostics for unsupported language surface

- **FR9:** A **developer** receives **explicit diagnostics** when a JSONata construct is **outside the supported subset**.
- **FR10:** Diagnostics **identify** the JSONata source **location** and the **reason** the construct is unsupported.
- **FR11:** A **developer** can **distinguish** “unsupported language feature” from **internal generator failure** categories of errors.

### Conformance & reference validation

- **FR12:** A **maintainer** can run an **automated conformance suite** grounded in **official JSONata corpus material** for **in-scope** scenarios.
- **FR13:** A **maintainer** can run **must-not-regress** scenario tiers separately from **broader backlog** coverage.
- **FR14:** A **maintainer** can compare transform results against **reference expectations** on **shared fixtures** for supported scenarios.
- **FR15:** A **maintainer** can run **optional baseline checks** against **external reference implementations** that are **not** shipped as runtime dependencies of generated transforms.

### Engineering policy enforcement

- **FR16:** A **maintainer** can treat **invariant violations** (including policy checks protecting the **no-eval** guarantee) as **release-blocking** even if other tests pass.
- **FR17:** A **maintainer** can enforce **static analysis expectations** on generated implementation artifacts in automation.
- **FR18:** A **maintainer** can enforce **forbidden-pattern** checks intended to prevent **hidden interpretation** of JSONata authoring text in generated paths.
- **FR19:** A **developer** can **validate**—through **documented** checks—that transform execution does **not** depend on reading **JSONata authoring text** (the policy is **demonstrable**, not only asserted).
- **FR20:** A **maintainer** can validate the **parser integration boundary** using **substitutable** test doubles without changing consumer-facing generator contracts.

### Trust, documentation, and transparency

- **FR21:** A **developer** can follow documentation whose **documented generator invocation** matches **automation** and **reference examples**.
- **FR22:** A **developer** can follow a **cold-start** path from **repository checkout** to **first successful generation and compile** of a reference workflow.
- **FR23:** A **reader** can understand **which artifacts are intended for production** versus **build-time-only** tooling from published materials.

### Release, compatibility, adoption, and support

- **FR24:** A **maintainer** can publish **release notes** with **compatibility matrix** updates for supported JSONata language scope.
- **FR25:** A **reader** can determine **which JSONata language scope** a release targets from published compatibility information.
- **FR26:** A **maintainer** can publish **migration guidance** when generated surfaces change in breaking ways.
- **FR27:** An **adopter** can perform **minor generator upgrades** while the **pinned JSONata language scope** remains unchanged, guided by **documented expectations** and **automated gates**.
- **FR28:** A **maintainer** can publish, per release, a **clear inventory** of **supported** JSONata language surface (and **explicit exclusions**), with a **single canonical** entry point suitable for **user-facing** docs and **downstream** tooling.
- **FR29:** An **adopter** can supply **standard repro metadata** (generator/tool version, **compatibility matrix** row, pinned JSONata scope) when reporting defects.

### Provenance & third-party materials

- **FR30:** A **maintainer** can publish **attribution** and **license** information for **third-party** conformance inputs (fixtures, corpus excerpts, vendored materials) used by the project.

### Hermetic automation expectations

- **FR31:** A **developer** can execute the **documented** reference generation workflow in **automation** without **undocumented** network dependencies required for routine generation (**documented** exceptions, if any).

### Contribution & maintenance

- **FR32:** A **contributor** can reproduce **conformance or correctness** issues using **fixture-based** reproducers.
- **FR33:** A **contributor** can add **regression coverage** that prevents recurrence of fixed **conformance/codegen** defect classes.
- **FR34:** A **contributor** can file issues using a **documented** template aligned to **fixture-based** reproduction.

## Non-Functional Requirements

### Performance

- **NFR-P1 (codegen latency):** For **reference** repository shapes documented by the project, **routine regeneration** stays within an agreed **wall-time budget** on **CI-class** hardware (numeric targets **TBD** once baselines exist; regressions investigated like functional defects).
- **NFR-P2 (generated runtime):** For **benchmarked** representative transforms, **generated** execution does not regress beyond an agreed threshold versus **prior release** or versus an **equivalent** **hand-written** reference implementation where one exists (thresholds and fixtures **TBD**).

### Security

- **NFR-S1 (licensing clarity):** Releases publish **clear** **OSS licensing** for the project and include **dependency/provenance** posture sufficient for **corporate** review (e.g. **LICENSE**, **NOTICE**/attribution as applicable).
- **NFR-S2 (no secret leakage via codegen):** **Default** generation paths must not embed **secrets**, **credentials**, or **host-specific** sensitive paths into generated artifacts (validated by **policy checks** where feasible).
- **NFR-S3 (supply chain hygiene):** **Build-time** dependencies are **pinned** or otherwise **reproducibly** resolved per documented practice; **notable** dependency changes are called out in **release** notes.

### Integration

- **NFR-I1 (Go toolchain compatibility):** The generator and **documented** workflows run on a **published** **supported Go** range aligned with **`go.mod` / `toolchain`** policy (**minimum** versions **TBD**).
- **NFR-I2 (CI determinism & operability):** **P0** jobs produce **deterministic** results under **documented** CI constraints (caching rules, parallelism, timeouts **TBD**); **flaky** jobs are triaged as **quality** defects.

**Out of scope for v1 NFR sections:** **Scalability** (hyperscale multi-tenant growth) and **product UI accessibility**—revisit if the surface expands beyond **docs** and **CLI**.
