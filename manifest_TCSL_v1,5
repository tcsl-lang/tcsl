

# TCSL Manifesto

**TextToCAD Scripting Language**

*Version 1.0 — March 18, 2026*

---

## Preamble

This document establishes the fundamental principles, philosophy, and commitments of the TCSL language. The manifesto does not replace the specification — it explains *why* the specification is designed the way it is. Every decision in the grammar, type system, and parser architecture traces back to the principles stated here. When ambiguity or conflict arises between possible design choices, the manifesto serves as the arbiter.

---

## I. Why TCSL Exists

Between a designer's intent and a three-dimensional model in a CAD system lies a gap. Traditionally, this gap is bridged in one of two ways: manual modeling, which is slow and non-parametric, or programming in Python, which is powerful but demands software engineering skill. TCSL exists so that a third participant can bridge this gap — a language model generating code from a textual description of a product.

TCSL is a universal contract between a human, a machine, and a geometric kernel. The human specifies parameters. The machine computes derived values and constructs geometry. The kernel guarantees the result. The contract works only if each party can verify its own part before passing it to the next. This is precisely why the parser exists — an independent arbiter that verifies the contract in milliseconds, without launching any specific CAD application.

TCSL is designed as an **industry-neutral language for describing physical products**. Cabinet furniture is the first proving ground on which the system is being refined, because furniture manufacturing rules are strict, well-understood, and easily verifiable. However, the language architecture is built for scaling from the outset: steel structures, mechanical engineering, retail fixtures, aviation components, building construction — any industry in which raw material becomes a physical object. Industry-specific logic is introduced through modular sets of manufacturing rules without affecting the language core.

---

## II. Ten Principles

### Principle 1. A Unit of Measurement Is Part of the Type, Not an Annotation

A number without a unit is not a length. `100` is not `100 mm` — it is a dimensionless scalar. TCSL does not allow adding millimeters to degrees, or multiplying volume by length to produce a nonexistent dimension L⁴. The type system is closed over the range L⁰–L³ plus a separate angular axis, and this closure is verified before a single polygon is ever constructed.

This is not pedantry — it is protection against a class of errors that are impossible to detect visually in a finished model. A part offset by 100 instead of 100 mm looks correct until it enters an assembly. This principle is equally critical for a furniture shelf, a machined engine component, and a structural building element.

### Principle 2. Four Zones, Four Responsibilities

A TCSL program moves in one direction: from parameters to geometry, from geometry to export. The zones INPUT, LET, GEOMETRY, and EXPORT follow strictly in that order, and the transition is irreversible. This is not a limitation on expressiveness — it is an architectural decision reflecting the real design process in any industry.

INPUT is what the customer changes. LET is what the engineer computes. GEOMETRY is what the machine builds. EXPORT is what the cost calculator consumes. This sequence is universal: it applies equally to a cabinetmaker specifying wardrobe width, a sheet metal fabricator specifying a K-factor for bending, and a mechanical engineer specifying a fit tolerance. Mixing these responsibilities means losing the ability to automate each one independently.

### Principle 3. What Is Declared Is Recomputed

If the parameter `total_width` changes, every value that depends on it updates automatically. In TCSL, there is no way to "forget" to update a derived parameter, because derived parameters are formulas, not numbers. `let inner_width = total_width - 2 * board` is a live relationship, not a dead record.

The separation between `input` (literals, overridable externally) and `let` (formulas, computed automatically) guarantees that a GUI configurator displays exactly what needs to be changed and hides what must not be changed. This mechanism works identically regardless of industry — only the content of the parameters changes, not the principle of their linkage.

### Principle 4. One Line, One Thought

TCSL is a single-line language. Each instruction occupies exactly one line. Line continuation is not supported. The pipeline operator `|>` is the only way to express a complex operation, and it stays on the same line.

This is a deliberate sacrifice of compactness for the sake of unambiguity. The parser synchronizes on line boundaries. The LLM generates code line by line. The human reads code line by line. An error is localized to one line. A diagnostic points to one line.

### Principle 5. Names Matter Beyond the Code

A variable in TCSL is not merely a memory address. The name `corpus_left_side` describes a part for a human. The name `Bok_LDSP16mm_1` after `export ... as` describes the same part for a cost calculator. Both names live on the same line, and both are validated by the parser.

TCSL bridges the world of engineering design (readable variable names) with the world of manufacturing accounting (codified part names) through a single syntactic construct: `export ... as`. The naming convention for exports adapts to the industry: for furniture it might be element-material-thickness, for steel structures it might be profile-grade-length, for mechanical engineering it might be part-tolerance-operation. The format is validated by the parser through configurable templates.

### Principle 6. An Error Is Better Than a Silent Failure

TCSL defines 32 diagnostic codes not because the language is complex, but because each error deserves its own explanation. `TCSL_E015` is not "type error" — it is "the resulting dimension falls outside the range L⁰–L³." `TCSL_R008` is not "runtime error" — it is "division by zero." Each code is an address in the specification where one can find an explanation, an example, and a remediation path.

The parser must catch everything that can be caught without running the geometric kernel. Two errors (`R005` — empty selector result, `R006` — type mismatch during runtime override) are deliberately left beyond the parser's reach — they require execution. Everything else is verified statically.

### Principle 7. The Grammar Serves the Generator, Not the Other Way Around

TCSL is designed so that an LLM can generate it step by step, without backtracking. The four zones correspond to four generation phases: first the base parameters (the LLM extracts them from the brief), then the formulas (the LLM computes dependencies), then the geometry (the LLM constructs parts), then the export (the LLM assigns names for the cost calculator).

The absence of user-defined functions, nested blocks, conditional constructs, and loops is not a deficiency — it is a requirement. Each of these constructs enlarges the state space in which the LLM can err. TCSL narrows that space to the minimum sufficient for describing a product. As new industries are added, the language grows not through syntactic complexity but through an expanding library of built-in constructors and manufacturing rules.

### Principle 8. The Parser Is the First Line of Defense

Between code generation and code execution stands the parser. It runs in milliseconds, requires no specific CAD system, consumes no GPU tokens, and creates no files. Its sole task is to say "yes" or "no, and here is why."

The parser verifies everything that can be verified without geometry: lexemes, syntax, zone ordering, dimensional algebra, pipeline type compatibility, operation arity, export uniqueness, positivity of constructor parameters, integrality of counters, and division by zero. That is 30 checks out of 32 — the parser covers 94% of the specification.

### Principle 9. The Specification Is the Single Source of Truth

The parser's behavior is defined by the specification, not by the implementation. If the implementation diverges from the specification, that is a bug in the implementation, not in the specification. The EBNF grammar with its 62 productions and 25 semantic constraints is a formal contract against which an alternative implementation can be written and expected to produce identical behavior.

The parser is a reference implementation, but not the standard. The standard is the text of the specification. The parser is its executable embodiment.

### Principle 10. Simplicity Is a Property That Must Be Defended

TCSL v1.5 contains 62 grammar rules, 23 functions, 5 enumerations, 17 units of measurement, and 32 error codes. That is not many. It is enough to describe products of arbitrary complexity within the current domain. As new industries are added, the library of constructors and rules grows, but the syntactic complexity of the core does not. Every element of the language must justify its existence by a task that cannot be accomplished without it.

A proposal to add a new construct must answer three questions: what task is impossible with existing facilities, how much harder does it make LLM generation, and how much harder does it make error diagnosis. If the answer to any of the three is unsatisfactory, the construct is not added.

---

## III. Architectural Commitments

### Commitment 1. Zero Runtime Dependencies

The TCSL parser runs on the Python 3.11+ standard library and nothing else. This guarantees that the parser can be launched in any environment where Python is available: a CI server, a Docker container, a developer's local machine, a Jupyter notebook, a cloud function. Dependency on external libraries is a debt that must eventually be repaid through updates, version conflicts, and loss of reproducibility.

### Commitment 2. Three Phases, Three Responsibilities

The lexer is responsible for characters and tokens. The parser is responsible for structure and grammar. The analyzer is responsible for types and semantics. Each phase receives the output of the previous one and has no backward dependencies. This makes it possible to test each phase in isolation, to replace one phase's implementation without affecting the others, and to debug problems starting from the lowest level.

### Commitment 3. An Error Is Not the End of Parsing

The parser must collect the maximum number of errors in a single pass. Upon encountering an error on line 5, it synchronizes on the line boundary and continues from line 6. The semantic analyzer runs even when syntactic errors are present — on those portions of the AST that were successfully parsed. This is critical for the LLM agent, which receives all errors in a single batch and corrects them in one iteration, rather than through a series of "fixed one, discovered the next" cycles.

### Commitment 4. Every Position Is Precise

Every token, every AST node, and every diagnostic contains a `Span` — a precise indication of file, line, start column, and end column. This is not a luxury but a requirement for three consumers: the IDE (error underlining), CI (machine-readable reports), and the LLM (automatic correction by position).

### Commitment 5. Determinism

An identical input file always produces an identical AST, an identical symbol table, and an identical list of diagnostics. The order of diagnostics is determined by position in the file, not by AST traversal order. This enables snapshot testing and guarantees reproducibility in CI.

### Commitment 6. Industry Neutrality of the Core

The language core — grammar, type system, dimensional algebra, zone structure, parser — contains no industry-specific logic. All specificity is encapsulated in pluggable modules: constructor libraries (geometric primitives and operations), sets of manufacturing rules, and export name templates. Adding a new industry does not require changes to the parser, the analyzer, or the core specification — only the addition of a new rule module.

---

## IV. Language Boundaries

TCSL deliberately does not support, and will not support, the following constructs. These limitations are not technical debt. They are architectural decisions that protect the simplicity and universality of the language.

**User-defined functions.** Every TCSL function is built in, with a fixed signature and semantics. User-defined functions introduce abstraction that requires a type system for return values, scoping rules, and a calling mechanism. For a domain-specific language that describes physical products, this complexity is not justified.

**Conditional constructs and loops.** `if`/`else` and `for`/`while` turn a declarative description into an imperative program with branches that the LLM must anticipate. Patterns (`linear_pattern`, `circular_pattern`) cover the need for repetition without a general looping mechanism.

**Nested blocks and scopes.** All identifiers live in a single flat symbol table. Nested scopes — blocks, closures — complicate name resolution and make the code less predictable for the LLM.

**String literals.** TCSL operates on numbers, units, identifiers, and enumerations. Strings are unnecessary: tag names and `as`-identifiers are identifiers, subject to the same lexical rules.

**Import and modularity at the user-code level.** Each `.tcsl` file is a self-contained unit describing one product. Dependencies between files create a graph that must be resolved, versioned, and cached. For describing a single product, this is excessive. Modularity exists at the system level — in the form of pluggable industry libraries — but not at the user-code level.

**Hard coupling to a single industry.** TCSL is not "a language for furniture." Furniture is the first proving ground. The language is designed so that constructors for sheet metal bending, milling operations, structural elements, or aviation components can be added as extensions to the standard library without affecting the grammar.

If a task requires constructs beyond TCSL, the solution is not to extend TCSL but to use a Python wrapper around the generated code.

---

## V. Commitments to Users

**To the design engineer.** TCSL reads like a product specification, in any industry. Variable names are meaningful, comments are explanatory, and the structure is consistent. A TCSL program can be shown to a colleague unfamiliar with the language, and they will understand what is described — whether it is a cabinet, a welded frame, or a machined housing.

**To the LLM agent.** TCSL is generated in a single pass, without backtracking, with a predictable structure. The four zones correspond to four prompt steps. Errors are returned with precise positions and codes suitable for automatic correction. When switching between industries, the set of available constructors and rules changes, but the generation logic does not.

**To the cost calculator.** Every exported part has a name from which the element, material, thickness, and sequence number can be extracted. The name format is validated by the parser (warning `W001`), not after the fact during import into the calculator. The format template adapts to the industry.

**To the transpiler developer.** The AST is typed, the symbol table is complete, and all checks have been performed. The transpiler can trust its input and focus on generating code for a specific CAD system (FreeCAD, KOMPAS-3D, SolidWorks, Fusion 360) without duplicating validation logic.

**To CI/CD.** Validation takes milliseconds, the exit code is unambiguous (0 for no errors, 1 for errors present), and output is available in both plain text and JSON format. TCSL validation integrates into a pipeline as naturally as a linter or a type checker.

---

## VI. Industry Scaling Strategy

TCSL expands to new industries on the principle of a **stable core with a growing periphery**.

The **language core** — grammar, type system, dimensional algebra, zone structure, parser — remains unchanged when a new industry is added. The core is a constitution that cannot be amended for every new law.

An **industry module** consists of three components: a constructor library (geometric primitives and operations specific to the industry), a set of manufacturing rules (constraints verified during model validation), and an export name template (the part-naming convention for manufacturing accounting in the given industry).

The **order of industry onboarding** is determined by the maturity of rules and the availability of training data:

- **Current proving ground:** cabinet furniture.
- **Near-term:** steel structures and sheet metal.
- **Long-term horizon:** retail fixtures, mechanical engineering, aviation and defense, construction and BIM.

**Integration with the CAD ecosystem** is achieved through the standard STEP format (ISO 10303), which eliminates vendor lock-in and enables TCSL to work with any CAD application that supports this format.

---

## VII. Language Evolution

TCSL follows a `<major>.<minor>` versioning scheme.

A **minor version** (1.4 → 1.5) adds new constructs, error codes, and industry modules, and refines semantics. It is backward compatible: a program valid in v1.4 remains valid in v1.5, except where specification errata are corrected.

A **major version** (1.x → 2.0) permits incompatible changes: removal of constructs, changes to the semantics of existing ones, and restructuring of zones.

Every change to the specification must pass through four filters:

- **Necessity.** What real-world task is impossible to accomplish with current facilities?
- **Generability.** How much harder will LLM generation become? Will the error rate increase?
- **Diagnosability.** Can the new construct be verified statically? What error codes are added?
- **Backward compatibility.** Will the change break existing programs?

If any of the four filters yields a negative answer, the change is deferred or rejected.

---

## VIII. Conclusion

TCSL is not a general-purpose programming language. It is a universal tool for a single class of tasks: transforming a textual description of a physical product into a parametric 3D model with names suitable for manufacturing accounting. Every decision in the language — from dimensional typing to the four-zone structure, from the prohibition of user-defined functions to mandatory units of measurement — is subordinated to this class of tasks.

Cabinet furniture is the first proving ground, not the final destination. The TCSL architecture has been built from the beginning as industry-neutral: a stable core, pluggable industry modules, standard integration formats. The team is building not "a program for furniture design" but a universal bridge between human language and real-world manufacturing — for any industry in which raw material becomes a physical object.

The simplicity of TCSL is not an initial state from which the language will "grow." It is a target property that must be defended with every change. A language that can describe everything can guarantee nothing. A language that describes exactly what is needed guarantees exactly what it promises. As it expands to new industries, the periphery grows — libraries of constructors and rules — but the core remains compact, rigorous, and predictable.

---

## License

This manifesto is released under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).

---

*TCSL Manifesto v1.0. Adopted March 18, 2026.*
