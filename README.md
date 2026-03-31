

# TCSL — TextToCAD Scripting Language

<p align="center">
  <img src="https://img.shields.io/badge/version-1.5-8B5CF6?style=for-the-badge&labelColor=0d1117" alt="Version 1.5">
  <img src="https://img.shields.io/badge/python-3.11+-3776AB?style=for-the-badge&logo=python&logoColor=white&labelColor=0d1117" alt="Python 3.11+">
  <img src="https://img.shields.io/badge/license-CC_BY_4.0-10b981?style=for-the-badge&labelColor=0d1117" alt="License CC BY 4.0">
  <img src="https://img.shields.io/badge/dependencies-zero-F59E0B?style=for-the-badge&labelColor=0d1117" alt="Zero Dependencies">
  <img src="https://img.shields.io/badge/grammar-62_rules-f54b68?style=for-the-badge&labelColor=0d1117" alt="62 Grammar Rules">
</p>

<p align="center">
  <strong>A declarative, dimensionally-typed language for transforming textual product descriptions into parametric 3D models.</strong>
</p>

<p align="center">
  <a href="#quick-start">Quick Start</a> •
  <a href="#what-is-tcsl">What is TCSL</a> •
  <a href="#four-zone-architecture">Architecture</a> •
  <a href="#language-reference">Reference</a> •
  <a href="#tooling">Tooling</a> •
  <a href="#examples">Examples</a> •
  <a href="#for-llm-agents">For LLM Agents</a> •
  <a href="#contributing">Contributing</a>
</p>

---

## Quick Start

```tcsl
# A parametric shelf — 4 lines, 1 exported part
input width = 600 mm
input depth = 300 mm
input thickness = 16 mm

shelf = box(width = width, depth = depth, height = thickness)

export shelf as Shelf_PB16mm_1
```

**Run the parser (zero dependencies):**

```bash
git clone https://github.com/texttocad/tcsl.git
cd tcsl
python src/parser.py examples/shelf.tcsl
```

```
✓ Parse OK — 0 errors, 0 warnings (3 ms)
```

---

## What is TCSL

TCSL is a **domain-specific language** designed as a universal contract between three parties:

| Party | Role |
|-------|------|
| **Human** | Specifies parameters — dimensions, materials, counts |
| **Machine** (LLM or transpiler) | Computes derived values, constructs geometry |
| **Geometric kernel** (FreeCAD, KOMPAS-3D, etc.) | Guarantees the physical result |

The contract works because each party can **verify its own part** before passing it to the next. The parser — an independent arbiter — verifies the contract in milliseconds, without launching any CAD application.

### Design Philosophy

TCSL is not a general-purpose programming language. It is a **precision tool for a single class of tasks**: transforming a textual description of a physical product into a parametric 3D model with names suitable for manufacturing accounting.

Every design decision flows from this constraint:

- **A number without a unit is not a length.** `100` is not `100 mm` — it is a dimensionless scalar. The type system is closed over L⁰–L³ plus a separate angular axis, verified before a single polygon is constructed.
- **No user-defined functions.** No `if`/`else`. No `for`/`while`. No nested scopes. Each of these enlarges the state space where an LLM can err. TCSL narrows that space to the minimum sufficient for describing a product.
- **One line, one thought.** The parser synchronizes on line boundaries. The LLM generates line by line. Errors are localized to one line.
- **An error is better than silent failure.** 32 diagnostic codes, each with its own explanation, example, and remediation path.

### Industry Scope

Cabinet furniture is the first proving ground — because furniture manufacturing rules are strict, well-understood, and easily verifiable. But the architecture is industry-neutral from the outset:

| Horizon | Industries |
|---------|-----------|
| **Current** | Cabinet furniture |
| **Near-term** | Steel structures, sheet metal |
| **Long-term** | Retail fixtures, mechanical engineering, aviation, construction/BIM |

Industry-specific logic lives in **pluggable modules** (constructor libraries, manufacturing rules, export templates) — the language core never changes.

---

## Four-Zone Architecture

Every TCSL program moves in one direction — from parameters to geometry to export. The transition is irreversible.

```
┌─────────┐    ┌─────────┐    ┌──────────┐    ┌─────────┐
│  INPUT   │───▶│   LET   │───▶│ GEOMETRY │───▶│ EXPORT  │
│          │    │         │    │          │    │         │
│ Customer │    │Engineer │    │ Machine  │    │  Cost   │
│ changes  │    │computes │    │ builds   │    │calculat.│
└─────────┘    └─────────┘    └──────────┘    └─────────┘
```

| Zone | Keyword | Purpose | Override |
|------|---------|---------|----------|
| **1 — INPUT** | `input` | Base parameters, literals only | Yes (GUI, CLI, API) |
| **2 — LET** | `let` | Derived parameters, formulas | No (automatic recompute) |
| **3 — GEOMETRY** | *(none)* | Solid construction, transforms, booleans | No |
| **4 — EXPORT** | `export` | Output declarations with cost-calculator names | No |

A minimal valid program requires at least one GEOMETRY line and one EXPORT line. INPUT and LET may be empty.

---

## Language Reference

### Dimensional Type System

The type system prevents an entire class of errors that are invisible in a finished model — a part offset by `100` instead of `100 mm` looks correct until it enters an assembly.

| Type | Dimension | Example |
|------|-----------|---------|
| `Scalar` | L⁰ | `3`, `0.5` |
| `Length` | L¹ | `732 mm`, `2.5 ft` |
| `Area` | L² | `10000 mm2` |
| `Volume` | L³ | `5000000 mm3` |
| `Angle` | separate axis | `90 deg`, `1.57 rad` |

**Arithmetic rules:** when multiplying, dimension exponents add; when dividing, they subtract. Results must stay in L⁰–L³. Exceeding this range → `TCSL_E015`. Mixing length and angle axes → `TCSL_R001`.

```tcsl
let floor_area = width * depth           # L¹ × L¹ = L² ✓
let volume = floor_area * height          # L² × L¹ = L³ ✓
let illegal = volume * height             # L³ × L¹ = L⁴ ✗ → TCSL_E015
```

### Units of Measurement

| Category | Units |
|----------|-------|
| Length | `mm` `cm` `m` `in` `ft` |
| Area | `mm2` `cm2` `m2` `in2` `ft2` |
| Volume | `mm3` `cm3` `m3` `in3` `ft3` |
| Angle | `deg` `rad` |

Space between number and unit is optional: `800mm` = `800 mm`. All units are **reserved words** — they cannot be used as identifiers.

### Constructors

| Constructor | Parameters | Notes |
|-------------|-----------|-------|
| `box(width, depth, height)` | Length × 3 | Volumetric parts — sides, posts |
| `plate(width, depth, height)` | Length × 3 | Semantic alias for `box` — flat parts (shelves, panels) |
| `cylinder(radius, height)` | Length × 2 | Z-axis, centered at origin |
| `sphere(radius)` | Length | Centered at origin |

All parameters must be strictly positive (> 0), otherwise → `TCSL_R002`.

### Pipeline Operator `|>`

Left-to-right data flow, **entire pipeline on a single line**:

```tcsl
# ✓ Correct
panel = box(width = 100 mm, depth = 50 mm, height = 16 mm) |> translate(x = 10 mm, y = 0 mm, z = 0 mm)

# ✗ Error — line break before |>
panel = box(width = 100 mm, depth = 50 mm, height = 16 mm)
    |> translate(x = 10 mm, y = 0 mm, z = 0 mm)
```

**Type compatibility:**

| Left type | Permissible operations |
|-----------|----------------------|
| `Geometry` | translate, rotate, scale, union, cut, subtract, intersect, linear_pattern, circular_pattern, select_*, tag |
| `EdgeSet` | fillet, chamfer |
| `FaceSet` | shell |

### Transforms

| Function | Parameters | Types |
|----------|-----------|-------|
| `translate(x, y, z)` | 3 offsets | Length × 3 |
| `rotate(angle, ax, ay, az, cx, cy, cz)` | angle + axis + center | Angle, Scalar × 3, Length × 3 |
| `scale(sx, sy, sz)` | 3 scale factors | Scalar × 3 |

### Boolean Operations

| Function | Arity | Description |
|----------|-------|-------------|
| `union(A, B, ...)` | 2+ | Union |
| `intersect(A, B, ...)` | 2+ | Intersection |
| `cut(A, B)` | exactly 2 | Subtraction |
| `subtract(A, B)` | exactly 2 | Alias for `cut` |

Arity violations → `TCSL_E021`.

### Patterns

```tcsl
pin |> linear_pattern(count = 6, dx = 32 mm, dy = 0 mm, dz = 0 mm)
pin |> circular_pattern(count = 8, angle = 360 deg, cx = 0 mm, cy = 0 mm, cz = 0 mm, ax = 0, ay = 0, az = 1)
```

`count` must be ≥ 2 and integer. Non-integer → `TCSL_R009`.

### Selectors & Modifiers

```tcsl
# Edge fillet
top = box(width = 500 mm, depth = 400 mm, height = 16 mm)
rounded = top |> select_edges_by_axis(axis = Z) |> fillet(radius = 5 mm)

# Shell
base = box(width = 300 mm, depth = 200 mm, height = 100 mm)
shell_part = base |> select_faces_by_axis(axis = Z, keep = positive) |> shell(thickness = 3 mm, kind = inward)
```

### Tagging

Two equivalent syntaxes:

```tcsl
# Infix (via $)
side = box$Side(width = 16 mm, depth = 550 mm, height = 900 mm)

# Pipeline (via tag())
side = box(width = 16 mm, depth = 550 mm, height = 900 mm) |> tag(Side, scope = edges)
```

### Export & Cost Calculator Naming

```tcsl
export left_side   as Side_PB16mm_1
export right_side  as Side_PB16mm_2
export back_panel  as BackPanel_HDF3mm_1
export drawer_front as Front_MDF19mm_1
```

The `as` identifier follows the pattern `Element_Material_Number` for cost calculator integration. Missing material keyword triggers warning `TCSL_W001`.

### Enumerations

| Enum | Values | Used in |
|------|--------|---------|
| `Axis` | `X`, `Y`, `Z` | select_edges_by_axis, select_faces_by_axis |
| `Keep` | `positive`, `negative`, `both` | select_faces_by_axis |
| `Kind` | `inward`, `outward` | shell |
| `Scope` | `all`, `edges`, `faces` | tag |
| `Expect` | `edges`, `faces` | select_by_tag |

---

## Error Codes

TCSL defines 32 diagnostic codes — not because the language is complex, but because each error deserves its own explanation.

### Syntactic Errors (E001–E021)

| Code | Description |
|------|-------------|
| `E001` | Missing unit on dimensional argument |
| `E002` | Bare arithmetic assignment in GEOMETRY zone |
| `E003` | Pipeline `\|>` on a new line |
| `E005` | Zone order violation |
| `E006` | Duplicate identifier |
| `E008` | Identifier is a reserved word/unit |
| `E012` | Expression in `input` (only literals allowed) |
| `E013` | Reference to undeclared identifier |
| `E015` | Dimension outside L⁰–L³ range |
| `E016` | Duplicate `as` name |
| `E017` | Duplicate export of same identifier |
| `E019` | Positional argument after named argument |
| `E020` | Parenthesis nesting depth exceeded |
| `E021` | Boolean operation arity violation |

### Semantic Errors (R001–R009)

| Code | Description |
|------|-------------|
| `R001` | Incompatible dimensions in arithmetic |
| `R002` | Zero or negative constructor parameter |
| `R003` | Incompatible type at pipeline input |
| `R004` | Export of non-Geometry type |
| `R005` | Empty selector result (runtime) |
| `R008` | Division by zero |
| `R009` | Non-integer value in integer-required context |

### Warnings (W001–W002)

| Code | Description |
|------|-------------|
| `W001` | `as` identifier lacks material keyword |
| `W002` | Declared `input` or `let` is unused |

The parser catches **30 of 32 checks** statically (94% coverage). Only `R005` (empty selector) and `R006` (runtime override mismatch) require execution.

---

## Tooling

### Parser

The reference parser runs on **Python 3.11+ standard library only** — zero external dependencies. Three-phase architecture:

```
Characters → [Lexer] → Tokens → [Parser] → AST → [Analyzer] → Typed AST + Diagnostics
```

Each phase receives the output of the previous one with no backward dependencies.

```bash
# Validate a file
python src/parser.py examples/cabinet.tcsl

# JSON output for CI
python src/parser.py examples/cabinet.tcsl --format json

# Exit codes: 0 = no errors, 1 = errors present
echo $?
```

**Key properties:**

- **Error recovery** — upon encountering an error on line 5, synchronizes on line boundary and continues from line 6. All errors returned in a single batch.
- **Determinism** — identical input always produces identical AST, symbol table, and diagnostics list (ordered by file position).
- **Precise spans** — every token, AST node, and diagnostic contains file, line, start column, and end column.

### VS Code Extension

Syntax highlighting via TextMate grammar with scope-level granularity:

```bash
cd tcsl-vscode
npx @vscode/vsce package
code --install-extension tcsl-1.5.0.vsix
```

Features: comment toggling (`Ctrl+/`), bracket matching, folding on zone separators (`# ===...===`), word navigation aligned to TCSL identifier rules.

### TCSL Viewer (Browser)

A fully client-side application — no server required:

- TCSL editor with syntax highlighting
- Real-time 3D viewport (Three.js + three-bvh-csg)
- Parametric sliders (auto-generated from `input` declarations)
- TCSL → Python/FreeCAD transpiler
- Cost calculator with 3 currencies and configurable markups
- Commercial quote → PDF (2 pages with 3D render)
- Share via compressed URL (LZString)
- 14 built-in templates

---

## Examples

### Minimal — Single Panel

```tcsl
panel = box(width = 100 mm, depth = 50 mm, height = 16 mm)
export panel
```

### Parametric Shelf Unit

```tcsl
input w = 732 mm
input board = 16 mm

let inner = w - 2 * board

side = box(width = board, depth = 400 mm, height = 600 mm)
shelf = box(width = inner, depth = 400 mm, height = board)

export side as Side_PB16mm_1
export shelf as Shelf_PB16mm_1
```

### Boolean Subtraction

```tcsl
input w = 200 mm
input d = 200 mm
input h = 200 mm
input hole_r = 40 mm

block = box(width = w, depth = d, height = h)
hole = cylinder(radius = hole_r, height = h)
result = block |> cut(hole)

export result as Block_PB16mm_1
```

### Full Dimension Chain (L → A → V → A → L)

```tcsl
input w = 400 mm
input d = 300 mm
input h = 200 mm

let floor_area = w * d              # → 120000 mm2  (Area)
let volume = floor_area * h         # → 24000000 mm3 (Volume)
let cross_section = volume / h      # → 120000 mm2  (Area)
let effective_w = cross_section / d # → 400 mm       (Length)

block = box(width = effective_w, depth = d, height = h)
export block as Block_PB16mm_1
```

### Profile Tube (Metal Structure)

```tcsl
input tube_w = 40 mm
input tube_d = 40 mm
input tube_t = 2 mm
input tube_len = 750 mm

let tube_inner_w = tube_w - 2 * tube_t
let tube_inner_d = tube_d - 2 * tube_t

tube_outer = box(width = tube_w, depth = tube_d, height = tube_len)
tube_inner = box(width = tube_inner_w, depth = tube_inner_d, height = tube_len + 2 mm) |> translate(x = tube_t, y = tube_t, z = -1 mm)
tube_post = tube_outer |> cut(other = tube_inner)

export tube_post as Post_Tube40x40x2_1
```

### Complete Assembly — Cabinet with Four Drawers

<details>
<summary><strong>Expand full example (~50 lines)</strong></summary>

```tcsl
# ===============================
# ZONE 1: INPUT — base parameters
# ===============================
input total_width   = 732 mm
input total_depth   = 550 mm
input side_height   = 900 mm
input board         = 16 mm
input back_panel_t  = 3 mm
input guide_w       = 13 mm
input niche_h       = 150 mm
input num_drawers   = 4
input gap           = 3 mm
input face_gap      = 2 mm
input front_board   = 19 mm

# ===============================
# ZONE 2: LET — derived parameters
# ===============================
let inner_width     = total_width - 2 * board
let inner_depth     = total_depth - back_panel_t
let drawer_width    = inner_width - 2 * guide_w
let usable_height   = side_height - 2 * board - niche_h
let drawer_step     = usable_height / num_drawers
let drawer_height   = drawer_step - gap
let face_width      = drawer_width - 2 * face_gap
let face_height     = drawer_height - face_gap

let z_bot           = 0 mm
let z_bot_top       = z_bot + board
let z_niche_top     = z_bot_top + niche_h
let z_d1            = z_niche_top + gap
let z_d2            = z_d1 + drawer_step
let z_d3            = z_d2 + drawer_step
let z_d4            = z_d3 + drawer_step
let z_top_base      = side_height - board

# ===============================
# ZONE 3: GEOMETRY
# ===============================
left_side  = box(width = board, depth = total_depth, height = side_height)
right_side = box(width = board, depth = total_depth, height = side_height) |> translate(x = total_width - board, y = 0 mm, z = 0 mm)
bottom     = box(width = inner_width, depth = total_depth, height = board) |> translate(x = board, y = 0 mm, z = z_bot)
top_panel  = box(width = inner_width, depth = total_depth, height = board) |> translate(x = board, y = 0 mm, z = z_top_base)
back_panel = box(width = inner_width, depth = back_panel_t, height = side_height - 2 * board) |> translate(x = board, y = total_depth - back_panel_t, z = z_bot_top)
front_1    = box(width = face_width, depth = front_board, height = face_height) |> translate(x = board + guide_w + face_gap, y = 0 mm - front_board, z = z_d1 + face_gap)
front_2    = box(width = face_width, depth = front_board, height = face_height) |> translate(x = board + guide_w + face_gap, y = 0 mm - front_board, z = z_d2 + face_gap)
front_3    = box(width = face_width, depth = front_board, height = face_height) |> translate(x = board + guide_w + face_gap, y = 0 mm - front_board, z = z_d3 + face_gap)
front_4    = box(width = face_width, depth = front_board, height = face_height) |> translate(x = board + guide_w + face_gap, y = 0 mm - front_board, z = z_d4 + face_gap)

# ===============================
# ZONE 4: EXPORT
# ===============================
export left_side   as Side_PB16mm_1
export right_side  as Side_PB16mm_2
export bottom      as Bottom_PB16mm_1
export top_panel   as Top_PB16mm_1
export back_panel  as BackPanel_HDF3mm_1
export front_1     as Front_MDF19mm_1
export front_2     as Front_MDF19mm_2
export front_3     as Front_MDF19mm_3
export front_4     as Front_MDF19mm_4
```

</details>

---

## For LLM Agents

TCSL is explicitly designed to be generated by language models. The architecture makes this practical:

### Why TCSL is LLM-Friendly

**Predictable structure.** The four zones correspond to four generation phases — the LLM extracts base parameters from the brief, computes dependencies, constructs geometry, assigns export names. No backtracking required.

**No branching constructs.** No `if`/`else`, no `for`/`while`, no user-defined functions. Each of these constructs would enlarge the state space where the LLM can err. Repetition is handled by `linear_pattern` / `circular_pattern`.

**Error recovery.** The parser collects the maximum number of errors in a single pass — the LLM receives all errors in one batch and corrects them in one iteration, rather than through repeated "fixed one, discovered the next" cycles.

**Precise positions.** Every diagnostic contains file, line, start column, and end column — suitable for automatic correction by position.

### Generation Rules (Summary)

1. **Units are mandatory** — `800 mm`, not `800`
2. **Zone order is strict** — INPUT → LET → GEOMETRY → EXPORT
3. **Pipeline on a single line** — no line breaks before `|>`
4. **Derived values go in `let`** — not precomputed `input`
5. **Fasteners use `box`** — never `cylinder + rotate`
6. **Each part gets a separate `export`** with material keyword in the `as` name
7. **Height marks** — calculate Z-levels top-down, verify arithmetic (δ ≤ 12 mm)

### LLM Fix Protocol

The TCSL Viewer includes an "LLM Fix" button that generates a structured report:

```
=== ERRORS ===
[3:15] TCSL_R001: Incompatible dimensions in arithmetic
[7:1]  TCSL_E005: Zone order violation — let after GEOMETRY

=== SOURCE CODE ===
   1 | input width = 732 mm
   2 | input angle = 45 deg
   3 | let bad = width + angle
   ...
```

This format is optimized for LLM consumption: errors with precise positions, full source with line numbers, clear instructions.

---

## Repository Structure

```
tcsl/
├── spec/
│   ├── tcsl-manifesto-v1.0.md       # Language philosophy & principles
│   ├── tcsl-spec-v1.5.md            # Formal specification
│   └── tcsl-guide-v5.3.md           # LLM generation manual
├── grammar/
│   ├── tcsl.ebnf                     # Formal EBNF grammar (62 rules)
│   └── tcsl.tmLanguage.json          # TextMate grammar for syntax highlighting
├── src/
│   ├── lexer.py                      # Tokenizer
│   ├── parser.py                     # Parser + semantic analyzer
│   └── codegen_freecad.py            # TCSL → Python/FreeCAD transpiler
├── tcsl-vscode/
│   ├── package.json                  # VS Code extension manifest
│   ├── language-configuration.json
│   └── syntaxes/
│       └── tcsl.tmLanguage.json
├── examples/
│   ├── minimal.tcsl
│   ├── shelf_unit.tcsl
│   ├── cabinet_4drawers.tcsl
│   ├── profile_tube.tcsl
│   └── ...
├── tests/
│   ├── valid/                        # Snapshot tests for valid programs
│   └── invalid/                      # Expected error diagnostics
├── LICENSE
└── README.md                         # ← You are here
```

---

## Specification Documents

| Document | Description |
|----------|-------------|
| **[Manifesto v1.0](spec/tcsl-manifesto-v1.0.md)** | Foundational principles, design philosophy, ten commandments, architectural commitments, language boundaries, industry scaling strategy |
| **[Specification v1.5](spec/tcsl-spec-v1.5.md)** | Formal language definition — lexical structure, type system, four zones, all functions, 32 error codes, examples of valid and invalid programs |
| **[EBNF Grammar v1.5](grammar/tcsl.ebnf)** | 62 production rules, 25 semantic constraints, LL(k≤3) compatible — the single source of truth for syntax |
| **[LLM Guide v5.3](spec/tcsl-guide-v5.3.md)** | Complete generation manual for external LLMs — data collection, critical rules, construction patterns, 5 system prompts, pre-delivery checklist |
| **[VS Code Extension](tcsl-vscode/)** | TextMate grammar, language configuration, scope map, coverage matrix |

---

## Grammar at a Glance

```
Production rules:     62
Sections:             28
Semantic constraints: 25 (SC-01 through SC-25)
Error codes:          21 syntactic (E001–E021)
                       9 semantic  (R001–R009)
                       2 warnings  (W001–W002)
LL(k) compatibility:  k ≤ 3
```

---

## Language Evolution

TCSL follows `<major>.<minor>` versioning.

**Minor version** (1.4 → 1.5) — backward compatible: new constructs, error codes, industry modules. A program valid in v1.4 remains valid in v1.5.

**Major version** (1.x → 2.0) — may introduce incompatible changes.

Every proposed change must pass four filters:

| Filter | Question |
|--------|----------|
| **Necessity** | What task is impossible with current facilities? |
| **Generability** | How much harder will LLM generation become? |
| **Diagnosability** | Can it be verified statically? What error codes are added? |
| **Compatibility** | Will it break existing programs? |

If any filter yields a negative answer, the change is deferred or rejected.

---

## Contributing

Contributions are welcome. Please read the following before submitting:

**Parser changes** must conform to the EBNF grammar in `grammar/tcsl.ebnf`. If the implementation diverges from the specification, that is a bug in the implementation, not in the specification.

**New functions or constructors** belong in industry modules, not the language core. The core grammar must remain stable across industry additions.

**New error codes** require an explanation, a triggering example, and a remediation path — matching the format in section 14 of the specification.

**Test coverage** — every valid example must have a snapshot test; every invalid example must verify the expected error code and position.

---

## License

- **Specification & Manifesto:** [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)
- **Parser & Tooling:** [MIT](LICENSE)
- **VS Code Extension:** [MIT](tcsl-vscode/LICENSE)

---

<p align="center">
  <sub>TCSL v1.5 · Specification adopted 2026-03-18 · Parser: zero dependencies, Python 3.11+</sub>
  <br>
  <sub>A language that describes exactly what is needed guarantees exactly what it promises.</sub>
</p>
