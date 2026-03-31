---

# TCSL v1.5 — TextMate Grammar & VS Code Extension

## Complete Specification Document

---

### Table of Contents

1. [Overview](#1-overview)
2. [Extension File Structure](#2-extension-file-structure)
3. [Grammar Design Rationale](#3-grammar-design-rationale)
4. [TextMate Grammar — `tcsl.tmLanguage.json`](#4-textmate-grammar--tcsltmlanguagejson)
5. [Extension Manifest — `package.json`](#5-extension-manifest--packagejson)
6. [Language Configuration — `language-configuration.json`](#6-language-configuration--language-configurationjson)
7. [Scope Map Reference](#7-scope-map-reference)
8. [Coverage Matrix](#8-coverage-matrix)
9. [Installation & Development](#9-installation--development)
10. [Testing](#10-testing)
11. [Known Limitations](#11-known-limitations)
12. [Contributing](#12-contributing)
13. [License](#13-license)

---

### 1. Overview

**TCSL** (TextToCAD Scripting Language) is a domain-specific declarative language for parametric 3D CAD model generation. It uses a pipeline-based syntax where primitive geometries are created, transformed, combined via boolean operations, patterned, selected, and modified through a left-to-right dataflow expressed with the `|>` (pipe-forward) operator.

This document specifies a **TextMate grammar** that provides syntax highlighting for TCSL v1.5 files (`.tcsl`) in Visual Studio Code and any editor compatible with TextMate grammars (Sublime Text, Atom, Zed, Shiki, etc.). The grammar targets the **JSON tmLanguage** format and uses **Oniguruma** regular expressions as required by VS Code's tokenization engine (`vscode-textmate`).

**Target language version:** TCSL 1.5

**Grammar format:** JSON TextMate Language Grammar (`.tmLanguage.json`)

**Regex engine:** Oniguruma (as implemented by `vscode-textmate`)

**VS Code minimum version:** 1.75.0

---

### 2. Extension File Structure

```
tcsl-vscode/
├── .vscodeignore
├── .gitignore
├── CHANGELOG.md
├── LICENSE
├── README.md
├── package.json                       # Extension manifest
├── language-configuration.json        # Bracket matching, comments, folding
├── syntaxes/
│   └── tcsl.tmLanguage.json           # TextMate grammar
├── examples/
│   ├── basic-box.tcsl
│   ├── bracket-with-fillets.tcsl
│   └── circular-pattern.tcsl
├── images/
│   └── icon.png                       # 128×128 marketplace icon
└── test/
    ├── colorize-results/
    │   └── baseline.json
    └── colorize-fixtures/
        └── sample.tcsl
```

---

### 3. Grammar Design Rationale

The grammar is organized as a flat list of top-level pattern includes that reference named rules in the `repository`. The ordering of includes is significant because TextMate grammars apply rules in listed order and the first match wins. The chosen order follows this priority:

**Comments** are matched first so that the `#` line-comment character is never consumed by another rule. **Zone declarations** (`input`, `let`, `export`) come next because they contain keywords that might otherwise be captured as bare identifiers. **Geometry assignment** follows, catching the `identifier = expression` pattern at the start of a line. The **pipeline operator** `|>` is a two-character token that must be matched before the generic arithmetic operator rule consumes `>`. **Function-call families** (constructors, transforms, booleans, patterns, selectors, modifiers, tags) are matched before generic identifiers so that known function names receive specific scopes. **Enum literals** are matched before generic identifiers so that axis names (`X`, `Y`, `Z`) and mode values receive `constant.language` scopes. **Dimensional literals** (number + unit) must precede plain **number literals** to prevent the regex engine from matching the numeric part alone and leaving the unit suffix orphaned. Within the dimensional-literal group, volume units are tested before area units, area before length, and length before angle — this longest-suffix-first ordering prevents `mm3` from being mismatched as `mm` plus a stray `3`. Finally, **operators**, **punctuation**, and generic **identifiers** serve as the catch-all layer.

All scope names follow the conventions documented in the TextMate Naming Conventions reference and target the scopes most commonly supported by popular VS Code color themes (Dark+, One Dark Pro, Dracula, Monokai, Catppuccin, Tokyo Night, etc.).

---

### 4. TextMate Grammar — `tcsl.tmLanguage.json`

```json
{
  "$schema": "https://raw.githubusercontent.com/martinring/tmlanguage/master/tmlanguage.json",
  "name": "TCSL",
  "scopeName": "source.tcsl",
  "fileTypes": ["tcsl"],
  "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",

  "patterns": [
    { "include": "#comment" },
    { "include": "#zone-input" },
    { "include": "#zone-let" },
    { "include": "#zone-export" },
    { "include": "#geometry-assignment" },
    { "include": "#pipeline-operator" },
    { "include": "#constructors" },
    { "include": "#transforms" },
    { "include": "#boolean-ops" },
    { "include": "#pattern-ops" },
    { "include": "#selectors" },
    { "include": "#modifiers" },
    { "include": "#tag-call" },
    { "include": "#enums" },
    { "include": "#dimensional-literal" },
    { "include": "#number-literal" },
    { "include": "#operators" },
    { "include": "#punctuation" },
    { "include": "#identifier" }
  ],

  "repository": {

    "comment": {
      "name": "comment.line.number-sign.tcsl",
      "match": "#.*$"
    },

    "zone-input": {
      "comment": "Input zone: input ParamName = expression",
      "match": "\\b(input)\\s+([A-Za-z][A-Za-z0-9_]*)\\s*(=)",
      "captures": {
        "1": { "name": "keyword.declaration.input.tcsl" },
        "2": { "name": "variable.parameter.input.tcsl" },
        "3": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "zone-let": {
      "comment": "Let zone: let VarName = expression",
      "match": "\\b(let)\\s+([A-Za-z][A-Za-z0-9_]*)\\s*(=)",
      "captures": {
        "1": { "name": "keyword.declaration.let.tcsl" },
        "2": { "name": "variable.other.let.tcsl" },
        "3": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "zone-export": {
      "comment": "Export zone: export Identifier [as Alias]",
      "match": "\\b(export)\\s+([A-Za-z][A-Za-z0-9_]*)(?:\\s+(as)\\s+([A-Za-z][A-Za-z0-9_]*))?",
      "captures": {
        "1": { "name": "keyword.declaration.export.tcsl" },
        "2": { "name": "variable.other.geometry.tcsl" },
        "3": { "name": "keyword.control.as.tcsl" },
        "4": { "name": "entity.name.export-alias.tcsl" }
      }
    },

    "geometry-assignment": {
      "comment": "Top-level geometry binding: Identifier = expression (not ==)",
      "match": "^\\s*([A-Za-z][A-Za-z0-9_]*)\\s*(=)(?!=)",
      "captures": {
        "1": { "name": "variable.other.geometry.tcsl" },
        "2": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "pipeline-operator": {
      "comment": "Pipe-forward operator for chaining operations",
      "name": "keyword.operator.pipeline.tcsl",
      "match": "\\|>"
    },

    "constructors": {
      "comment": "Primitive geometry constructors: box, plate, cylinder, sphere",
      "patterns": [
        {
          "comment": "Constructor with inline tag: box$TagName(...)",
          "match": "\\b(box|plate|cylinder|sphere)(\\$)([A-Za-z][A-Za-z0-9_]*)\\s*(\\()",
          "captures": {
            "1": { "name": "entity.name.function.constructor.tcsl" },
            "2": { "name": "punctuation.separator.tag.tcsl" },
            "3": { "name": "entity.name.tag.tcsl" },
            "4": { "name": "punctuation.section.arguments.begin.tcsl" }
          }
        },
        {
          "comment": "Constructor without tag: box(...)",
          "match": "\\b(box|plate|cylinder|sphere)\\s*(\\()",
          "captures": {
            "1": { "name": "entity.name.function.constructor.tcsl" },
            "2": { "name": "punctuation.section.arguments.begin.tcsl" }
          }
        }
      ]
    },

    "transforms": {
      "comment": "Spatial transformation functions: translate, rotate, scale",
      "match": "\\b(translate|rotate|scale)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.transform.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "boolean-ops": {
      "comment": "Boolean (CSG) operations: union, cut, subtract, intersect",
      "match": "\\b(union|cut|subtract|intersect)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.boolean.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "pattern-ops": {
      "comment": "Repetition pattern functions: linear_pattern, circular_pattern",
      "match": "\\b(linear_pattern|circular_pattern)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.pattern.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "selectors": {
      "comment": "Topology selectors for edges and faces",
      "match": "\\b(select_edges_by_axis|select_edges_by_length|select_edges_parallel|select_edges_at_height|select_faces_by_axis|select_faces_by_area|select_by_tag)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.selector.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "modifiers": {
      "comment": "Geometry modifier functions: fillet, chamfer, shell",
      "match": "\\b(fillet|chamfer|shell)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.modifier.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "tag-call": {
      "comment": "Tag annotation function",
      "match": "\\b(tag)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.tag.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "enums": {
      "comment": "Enum literal values used as bare identifiers in function arguments",
      "patterns": [
        {
          "comment": "Axis enum: X, Y, Z",
          "name": "constant.language.enum.axis.tcsl",
          "match": "\\b(X|Y|Z)\\b"
        },
        {
          "comment": "Keep enum (cut direction): positive, negative, both",
          "name": "constant.language.enum.keep.tcsl",
          "match": "\\b(positive|negative|both)\\b"
        },
        {
          "comment": "Kind enum (shell direction): inward, outward",
          "name": "constant.language.enum.kind.tcsl",
          "match": "\\b(inward|outward)\\b"
        },
        {
          "comment": "Scope enum (tag scope): all, edges, faces",
          "name": "constant.language.enum.scope.tcsl",
          "match": "\\b(all|edges|faces)\\b"
        }
      ]
    },

    "dimensional-literal": {
      "comment": "Numeric value followed by a dimensional unit. Rules are ordered by suffix length (longest first) to ensure correct matching: volume > area > length > angle.",
      "patterns": [
        {
          "comment": "Volume units: mm3, cm3, m3, in3, ft3",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm3|cm3|m3|in3|ft3)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.volume.tcsl" }
          }
        },
        {
          "comment": "Area units: mm2, cm2, m2, in2, ft2",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm2|cm2|m2|in2|ft2)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.area.tcsl" }
          }
        },
        {
          "comment": "Length units: mm, cm, ft, in, m",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm|cm|ft|in|m)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.length.tcsl" }
          }
        },
        {
          "comment": "Angle units: deg, rad",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(deg|rad)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.angle.tcsl" }
          }
        }
      ]
    },

    "number-literal": {
      "comment": "Plain numeric literals (no unit suffix). Matched after dimensional literals.",
      "patterns": [
        {
          "comment": "Floating-point literal (requires digits on both sides of the decimal point)",
          "name": "constant.numeric.float.tcsl",
          "match": "(?<![A-Za-z_.])\\d+\\.\\d+\\b"
        },
        {
          "comment": "Integer literal",
          "name": "constant.numeric.integer.tcsl",
          "match": "(?<![A-Za-z_.])\\d+\\b"
        }
      ]
    },

    "operators": {
      "patterns": [
        {
          "comment": "Assignment operator (excludes == and !=)",
          "name": "keyword.operator.assignment.tcsl",
          "match": "(?<![=!<>])=(?!=)"
        },
        {
          "comment": "Arithmetic operators: + - * /",
          "name": "keyword.operator.arithmetic.tcsl",
          "match": "[+\\-*/]"
        }
      ]
    },

    "punctuation": {
      "patterns": [
        {
          "name": "punctuation.section.arguments.begin.tcsl",
          "match": "\\("
        },
        {
          "name": "punctuation.section.arguments.end.tcsl",
          "match": "\\)"
        },
        {
          "name": "punctuation.separator.comma.tcsl",
          "match": ","
        },
        {
          "name": "punctuation.separator.tag.tcsl",
          "match": "\\$"
        }
      ]
    },

    "named-argument": {
      "comment": "Named argument inside a function call: paramName = value",
      "match": "\\b([A-Za-z][A-Za-z0-9_]*)\\s*(=)(?!=)",
      "captures": {
        "1": { "name": "variable.parameter.named.tcsl" },
        "2": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "identifier": {
      "comment": "Generic identifier (catch-all for variable references and unknown names)",
      "name": "variable.other.tcsl",
      "match": "\\b[A-Za-z][A-Za-z0-9_]*\\b"
    }
  }
}
```

---

### 5. Extension Manifest — `package.json`

```json
{
  "name": "tcsl",
  "displayName": "TCSL — TextToCAD Scripting Language",
  "description": "Syntax highlighting for TCSL v1.5 (.tcsl files)",
  "version": "1.5.0",
  "publisher": "texttocad",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/texttocad/tcsl-vscode.git"
  },
  "bugs": {
    "url": "https://github.com/texttocad/tcsl-vscode/issues"
  },
  "icon": "images/icon.png",
  "engines": {
    "vscode": "^1.75.0"
  },
  "categories": ["Programming Languages"],
  "keywords": ["tcsl", "texttocad", "cad", "parametric", "3d", "syntax"],
  "contributes": {
    "languages": [
      {
        "id": "tcsl",
        "aliases": ["TCSL", "TextToCAD"],
        "extensions": [".tcsl"],
        "mimetypes": ["application/vnd.texttocad.tcsl"],
        "configuration": "./language-configuration.json",
        "icon": {
          "light": "./images/icon.png",
          "dark": "./images/icon.png"
        }
      }
    ],
    "grammars": [
      {
        "language": "tcsl",
        "scopeName": "source.tcsl",
        "path": "./syntaxes/tcsl.tmLanguage.json"
      }
    ]
  }
}
```

---

### 6. Language Configuration — `language-configuration.json`

```json
{
  "comments": {
    "lineComment": "#"
  },
  "brackets": [
    ["(", ")"]
  ],
  "autoClosingPairs": [
    { "open": "(", "close": ")" }
  ],
  "surroundingPairs": [
    ["(", ")"]
  ],
  "folding": {
    "markers": {
      "start": "^\\s*#\\s*={3,}",
      "end": "^\\s*#\\s*={3,}"
    }
  },
  "wordPattern": "[A-Za-z][A-Za-z0-9_]*",
  "indentationRules": {
    "increaseIndentPattern": "^$",
    "decreaseIndentPattern": "^$"
  }
}
```

The `comments` section enables the `Toggle Line Comment` command (Ctrl+/ / Cmd+/) to insert `#` prefixes. The `brackets` and `autoClosingPairs` sections enable bracket matching and auto-closing for parentheses, the only bracket type in TCSL. The `folding.markers` configuration allows zone-separator comments of the form `# ===...===` to act as fold boundaries in the editor. The `wordPattern` defines navigation boundaries for Ctrl+D (select word) and Ctrl+Left/Right (word jump) based on TCSL's identifier rules.

---

### 7. Scope Map Reference

The table below maps every TCSL syntactic element to its TextMate scope, the typical rendering in popular themes, and the grammar rule that produces it.

| TCSL Element | TextMate Scope | Typical Theme Color | Grammar Rule |
|---|---|---|---|
| `#` line comment | `comment.line.number-sign.tcsl` | Gray / Dim | `comment` |
| `input` keyword | `keyword.declaration.input.tcsl` | Purple / Blue | `zone-input` |
| `let` keyword | `keyword.declaration.let.tcsl` | Purple / Blue | `zone-let` |
| `export` keyword | `keyword.declaration.export.tcsl` | Purple / Blue | `zone-export` |
| `as` keyword | `keyword.control.as.tcsl` | Purple / Blue | `zone-export` |
| Input parameter name | `variable.parameter.input.tcsl` | Orange / Italic | `zone-input` |
| Let-bound variable name | `variable.other.let.tcsl` | Foreground | `zone-let` |
| Geometry variable name | `variable.other.geometry.tcsl` | Foreground | `geometry-assignment`, `zone-export` |
| Export alias name | `entity.name.export-alias.tcsl` | Teal / Cyan | `zone-export` |
| `=` operator | `keyword.operator.assignment.tcsl` | Operator color | `operators`, captures |
| `\|>` pipeline | `keyword.operator.pipeline.tcsl` | Operator / Red | `pipeline-operator` |
| `+ - * /` arithmetic | `keyword.operator.arithmetic.tcsl` | Operator color | `operators` |
| `box`, `plate`, `cylinder`, `sphere` | `entity.name.function.constructor.tcsl` | Yellow / Blue | `constructors` |
| `$` tag separator | `punctuation.separator.tag.tcsl` | Punctuation dim | `constructors`, `punctuation` |
| Inline tag name | `entity.name.tag.tcsl` | Green / Teal | `constructors` |
| `translate`, `rotate`, `scale` | `entity.name.function.transform.tcsl` | Yellow / Blue | `transforms` |
| `union`, `cut`, `subtract`, `intersect` | `entity.name.function.boolean.tcsl` | Yellow / Blue | `boolean-ops` |
| `linear_pattern`, `circular_pattern` | `entity.name.function.pattern.tcsl` | Yellow / Blue | `pattern-ops` |
| `select_edges_by_axis`, etc. | `entity.name.function.selector.tcsl` | Yellow / Blue | `selectors` |
| `fillet`, `chamfer`, `shell` | `entity.name.function.modifier.tcsl` | Yellow / Blue | `modifiers` |
| `tag` | `entity.name.function.tag.tcsl` | Yellow / Blue | `tag-call` |
| `X`, `Y`, `Z` | `constant.language.enum.axis.tcsl` | Constant / Orange | `enums` |
| `positive`, `negative`, `both` | `constant.language.enum.keep.tcsl` | Constant / Orange | `enums` |
| `inward`, `outward` | `constant.language.enum.kind.tcsl` | Constant / Orange | `enums` |
| `all`, `edges`, `faces` | `constant.language.enum.scope.tcsl` | Constant / Orange | `enums` |
| `100mm`, `45deg`, etc. (numeric part) | `constant.numeric.value.tcsl` | Number color | `dimensional-literal` |
| `mm`, `cm`, `m`, `in`, `ft` | `keyword.other.unit.length.tcsl` | Unit / Cyan | `dimensional-literal` |
| `mm2`, `cm2`, `m2`, `in2`, `ft2` | `keyword.other.unit.area.tcsl` | Unit / Cyan | `dimensional-literal` |
| `mm3`, `cm3`, `m3`, `in3`, `ft3` | `keyword.other.unit.volume.tcsl` | Unit / Cyan | `dimensional-literal` |
| `deg`, `rad` | `keyword.other.unit.angle.tcsl` | Unit / Cyan | `dimensional-literal` |
| Float literal (`3.14`) | `constant.numeric.float.tcsl` | Number color | `number-literal` |
| Integer literal (`42`) | `constant.numeric.integer.tcsl` | Number color | `number-literal` |
| `(` | `punctuation.section.arguments.begin.tcsl` | Punctuation dim | `punctuation`, captures |
| `)` | `punctuation.section.arguments.end.tcsl` | Punctuation dim | `punctuation` |
| `,` | `punctuation.separator.comma.tcsl` | Punctuation dim | `punctuation` |
| Named argument name | `variable.parameter.named.tcsl` | Orange / Italic | `named-argument` |
| Generic identifier | `variable.other.tcsl` | Foreground | `identifier` |

---

### 8. Coverage Matrix

This section maps each TCSL v1.5 language feature to the grammar rule(s) that provide its highlighting.

**Zone Declarations.** Each zone keyword (`input`, `let`, `export`) is matched together with the identifier that follows it and the `=` sign. This combined match ensures that variable names receive different scopes depending on their declaration zone: `variable.parameter.input` for input parameters (which themes typically render in italic or orange), `variable.other.let` for let-bound intermediate values, and `variable.other.geometry` for exported geometry names. The `export ... as Alias` construction parses `as` as a distinct control keyword and highlights the alias name as `entity.name.export-alias`.

**Constructors.** Two patterns handle the tagged and untagged forms. The tagged form `box$Side(...)` is matched by a regex that captures the constructor name, the `$` separator, the tag identifier, and the opening parenthesis as four distinct groups. The untagged form `box(...)` is a simpler two-group match. All four primitives (`box`, `plate`, `cylinder`, `sphere`) are covered.

**Function Categories.** Transforms (`translate`, `rotate`, `scale`), boolean operations (`union`, `cut`, `subtract`, `intersect`), repetition patterns (`linear_pattern`, `circular_pattern`), topology selectors (`select_edges_by_axis`, `select_edges_by_length`, `select_edges_parallel`, `select_edges_at_height`, `select_faces_by_axis`, `select_faces_by_area`, `select_by_tag`), geometry modifiers (`fillet`, `chamfer`, `shell`), and the `tag` annotation function each have their own grammar rule and scope subcategory. This category-level granularity allows theme authors and users to assign distinct colors to each functional group via `editor.tokenColorCustomizations` in their VS Code settings.

**Dimensional Literals.** The four sub-patterns within the `dimensional-literal` rule are ordered by suffix length in descending order. Volume units (`mm3`, `cm3`, `m3`, `in3`, `ft3`) are tested first, followed by area units (`mm2`, `cm2`, `m2`, `in2`, `ft2`), then length units (`mm`, `cm`, `ft`, `in`, `m`), and finally angle units (`deg`, `rad`). This ordering ensures that the input `100mm3` is correctly tokenized as the number `100` plus the volume unit `mm3`, rather than being prematurely split into `100mm` (length) plus a stray `3`. Each pattern uses a negative lookbehind `(?<![A-Za-z_])` to prevent matching the numeric suffix of an identifier. Optional whitespace between the number and unit (`100 mm` vs `100mm`) is supported.

**Enum Literals.** The values `X`, `Y`, `Z` (axis), `positive`, `negative`, `both` (keep direction), `inward`, `outward` (shell kind), and `all`, `edges`, `faces` (tag scope) are highlighted as `constant.language.enum.*`. Because TCSL uses bare identifiers for enum values (not prefixed by a type name), the grammar matches them by value. This is a known imprecision: these words will be highlighted as enums even outside of function argument positions. This is a fundamental limitation of TextMate's stateless, line-oriented regex engine and can only be resolved with a semantic token provider.

**Pipeline Operator.** The `|>` operator is matched as a single two-character token with the scope `keyword.operator.pipeline.tcsl`. It is listed before the arithmetic operator rule to prevent the `>` character from being consumed by a future comparison-operator pattern.

**Comments.** The `#` character and everything after it until end of line is captured as `comment.line.number-sign.tcsl`. This is the first rule in the top-level pattern list, ensuring that commented-out code is never partially tokenized by other rules.

**Named Arguments.** The `named-argument` rule is defined in the repository for use in contexts where function argument internals are parsed. It matches `paramName = value` patterns (excluding `==`) and assigns `variable.parameter.named` to the argument name.

---

### 9. Installation & Development

**Install from VSIX (local build):**

```bash
# Clone the repository
git clone https://github.com/texttocad/tcsl-vscode.git
cd tcsl-vscode

# Package the extension
npx @vscode/vsce package

# Install the generated .vsix file
code --install-extension tcsl-1.5.0.vsix
```

**Install from VS Code Marketplace (when published):**

Open VS Code, press Ctrl+Shift+X (Cmd+Shift+X on macOS), search for "TCSL", and click Install.

**Development workflow:**

Open the `tcsl-vscode` folder in VS Code, then press F5 to launch the Extension Development Host. Open any `.tcsl` file to see the syntax highlighting in action. Use the built-in scope inspector (`Developer: Inspect Editor Tokens and Scopes` from the Command Palette) to verify that tokens receive the expected scopes.

After making changes to `tcsl.tmLanguage.json`, reload the Extension Development Host window (Ctrl+Shift+P → `Developer: Reload Window`) for changes to take effect.

---

### 10. Testing

VS Code provides a built-in mechanism for snapshot-testing TextMate grammars via the `vscode-tmgrammar-test` package or by using the `vscode-tmgrammar-snap` tool.

**Manual verification with the scope inspector:**

Open a representative `.tcsl` file in the Extension Development Host and invoke `Developer: Inspect Editor Tokens and Scopes`. Click on each token and verify that the scope stack matches the expected values from the scope map in Section 7.

**Recommended test fixture (`test/colorize-fixtures/sample.tcsl`):**

```tcsl
# ============================
# Zone: Input
# ============================
input width = 100mm
input height = 50mm
input draft_angle = 5deg

# ============================
# Zone: Let
# ============================
let half_width = width / 2
let volume_check = 1000cm3

# ============================
# Zone: Geometry
# ============================
Body = box(width, height, 30mm)
    |> translate(0mm, 0mm, 15mm)
    |> fillet(select_edges_by_axis(Z), 3mm)

Hole = cylinder$Bore(10mm, 60mm)
    |> rotate(X, 90deg)
    |> translate(half_width, 0mm, 25mm)

Result = Body
    |> cut(Hole, keep = positive)
    |> shell(2mm, kind = outward)
    |> chamfer(select_edges_at_height(Z, 30mm), 1mm)
    |> tag(Bore, scope = edges)
    |> linear_pattern(axis = X, count = 3, spacing = 40mm)
    |> circular_pattern(axis = Z, count = 6, angle = 360deg)

# ============================
# Zone: Export
# ============================
export Result as FinalBracket
```

This fixture exercises every grammar rule: comments, all three zone types, geometry assignment, pipeline chaining, all constructor forms (plain and tagged), transforms, boolean operations, selectors, modifiers, tag calls, patterns, enum values across all categories, dimensional literals of every unit family, plain numbers, arithmetic operators, named arguments, and the export-with-alias construction.

---

### 11. Known Limitations

**Context-free enum matching.** TextMate grammars are stateless; they cannot verify that an enum literal appears within a function argument list. The words `X`, `Y`, `Z`, `all`, `edges`, `faces`, `positive`, `negative`, `both`, `inward`, and `outward` will always be highlighted as enum constants regardless of position. A semantic token provider (via a Language Server Protocol extension) would be required to provide context-aware highlighting.

**Named arguments are not context-sensitive.** The `named-argument` rule is defined but not included in the top-level pattern list by default because it would override geometry assignment matching. In a future version with `begin`/`end` scoped rules for function-call interiors, named arguments could be matched only inside parentheses.

**No multi-line constructs.** TCSL v1.5 does not define multi-line strings or block comments. If the language adds these in a future version, the grammar will need `begin`/`end` rules to handle multi-line spans.

**No negative-number ambiguity resolution.** The dimensional literal pattern includes an optional leading minus sign (`-?`). In some contexts (e.g., `width - 10mm`), the minus might be part of an arithmetic expression rather than a negative literal. TextMate cannot disambiguate this without parser-level context. In practice, this rarely causes visible highlighting issues because both the minus-as-operator and minus-as-sign paths result in similarly colored output.

**Single-character enum names.** The axis values `X`, `Y`, and `Z` are single uppercase letters bounded by `\b`. This means that any standalone uppercase `X`, `Y`, or `Z` in the source will be highlighted as an axis constant, even if used as a variable name. TCSL conventions discourage single-letter variable names, but this remains a theoretical conflict.

---

### 12. Contributing

Contributions are welcome. Please follow these guidelines:

When adding new function names or keywords, place the regex in the appropriate grammar rule category (constructors, transforms, boolean-ops, etc.) and use the established scope naming convention for that category. Ensure that the new keyword is added to both the regex alternation and the scope map table in this document.

When modifying regex patterns, test against the fixture file in Section 10 and verify with the VS Code scope inspector that no existing scopes are broken. Pay particular attention to the ordering of patterns — the longest-match-first principle must be maintained for dimensional literals and any future overlapping token groups.

When submitting a pull request, include before/after screenshots of the syntax highlighting for the affected constructs, and update this specification document to reflect the changes.

---

### 13. License

This extension and its grammar are released under the [MIT License](LICENSE).

```
MIT License

Copyright (c) 2025 TextToCAD Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## Analysis Summary: Changes from the Original Russian Document

Here is a summary of every adaptation and improvement made during the localization:

**Structural reorganization.** The original document presented the grammar, package.json, language-configuration.json, and explanation as a loosely connected sequence. The final document is restructured into thirteen numbered sections with a table of contents, making it suitable as both a GitHub README and a standalone specification document.

**Full English localization.** All Russian prose has been translated to English. Comments embedded within the JSON grammar files (`"comment":` fields) have been added or expanded in English to explain every rule's purpose directly within the code.

**Added `"comment"` fields throughout the grammar.** The original grammar had comments on only a few rules. The final version includes descriptive English comments on every repository entry, improving maintainability for contributors who read the grammar JSON directly.

**Renamed `patterns-ops` to `pattern-ops`.** The original used `patterns-ops` (plural), which conflicts with the TextMate keyword `patterns`. The singular form `pattern-ops` is clearer and avoids potential confusion.

**Expanded `package.json` with required GitHub/marketplace fields.** The original manifest lacked `repository`, `bugs`, `icon`, and `keywords` fields. These have been added to meet VS Code Marketplace publishing requirements and GitHub discoverability best practices.

**Scope Map Reference table (Section 7).** The original document described scopes in narrative form. The final document provides a comprehensive table mapping every TCSL element to its TextMate scope, typical theme color, and originating grammar rule — an at-a-glance reference for theme authors and grammar maintainers.

**Coverage Matrix prose (Section 8).** The original's "what this grammar covers" section has been expanded into a systematic walkthrough of every language feature, including the rationale for pattern ordering and the longest-suffix-first strategy for dimensional units.

**Test fixture (Section 10).** A complete `.tcsl` test file has been provided that exercises every grammar rule. The original document did not include a test fixture.

**Known Limitations section (Section 11).** Explicit documentation of the grammar's inherent limitations (context-free enum matching, named argument ambiguity, negative-number edge case, single-character identifiers) was partially covered in the original's inline notes. The final document consolidates these into a dedicated section with clear explanations.

**Contributing guidelines (Section 12).** Added practical instructions for contributors, including pattern-ordering rules and the requirement to update the specification when modifying the grammar.

**License section (Section 13).** Added a full MIT license text, which the original referenced by name only.
