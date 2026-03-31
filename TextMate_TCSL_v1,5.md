

Я создам TextMate-грамматику для подсветки синтаксиса TCSL v1.5 в VS Code на основе предоставленной спецификации.

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
    { "include": "#patterns-ops" },
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
      "match": "\\b(input)\\s+([A-Za-z][A-Za-z0-9_]*)\\s*(=)",
      "captures": {
        "1": { "name": "keyword.declaration.input.tcsl" },
        "2": { "name": "variable.parameter.input.tcsl" },
        "3": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "zone-let": {
      "match": "\\b(let)\\s+([A-Za-z][A-Za-z0-9_]*)\\s*(=)",
      "captures": {
        "1": { "name": "keyword.declaration.let.tcsl" },
        "2": { "name": "variable.other.let.tcsl" },
        "3": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "zone-export": {
      "match": "\\b(export)\\s+([A-Za-z][A-Za-z0-9_]*)(?:\\s+(as)\\s+([A-Za-z][A-Za-z0-9_]*))?",
      "captures": {
        "1": { "name": "keyword.declaration.export.tcsl" },
        "2": { "name": "variable.other.geometry.tcsl" },
        "3": { "name": "keyword.control.as.tcsl" },
        "4": { "name": "entity.name.export-alias.tcsl" }
      }
    },

    "geometry-assignment": {
      "match": "^\\s*([A-Za-z][A-Za-z0-9_]*)\\s*(=)(?!=)",
      "captures": {
        "1": { "name": "variable.other.geometry.tcsl" },
        "2": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "pipeline-operator": {
      "name": "keyword.operator.pipeline.tcsl",
      "match": "\\|>"
    },

    "constructors": {
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
      "match": "\\b(translate|rotate|scale)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.transform.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "boolean-ops": {
      "match": "\\b(union|cut|subtract|intersect)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.boolean.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "patterns-ops": {
      "match": "\\b(linear_pattern|circular_pattern)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.pattern.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "selectors": {
      "match": "\\b(select_edges_by_axis|select_edges_by_length|select_edges_parallel|select_edges_at_height|select_faces_by_axis|select_faces_by_area|select_by_tag)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.selector.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "modifiers": {
      "match": "\\b(fillet|chamfer|shell)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.modifier.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "tag-call": {
      "match": "\\b(tag)\\s*(\\()",
      "captures": {
        "1": { "name": "entity.name.function.tag.tcsl" },
        "2": { "name": "punctuation.section.arguments.begin.tcsl" }
      }
    },

    "enums": {
      "patterns": [
        {
          "comment": "Axis enum values",
          "name": "constant.language.enum.axis.tcsl",
          "match": "\\b(X|Y|Z)\\b"
        },
        {
          "comment": "Keep enum values",
          "name": "constant.language.enum.keep.tcsl",
          "match": "\\b(positive|negative|both)\\b"
        },
        {
          "comment": "Kind enum values",
          "name": "constant.language.enum.kind.tcsl",
          "match": "\\b(inward|outward)\\b"
        },
        {
          "comment": "Scope enum values (tag)",
          "name": "constant.language.enum.scope.tcsl",
          "match": "\\b(all|edges|faces)\\b"
        }
      ]
    },

    "dimensional-literal": {
      "comment": "Number followed by a unit. Longest-match: volume > area > length > angle. Units may be preceded by optional whitespace.",
      "patterns": [
        {
          "comment": "Volume units (mm3, cm3, m3, in3, ft3)",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm3|cm3|m3|in3|ft3)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.volume.tcsl" }
          }
        },
        {
          "comment": "Area units (mm2, cm2, m2, in2, ft2)",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm2|cm2|m2|in2|ft2)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.area.tcsl" }
          }
        },
        {
          "comment": "Length units (mm, cm, ft, in, m) — 'in' and 'm' checked after longer variants",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(mm|cm|ft|in|m)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.length.tcsl" }
          }
        },
        {
          "comment": "Angle units (deg, rad)",
          "match": "(?<![A-Za-z_])(-?\\d+(?:\\.\\d+)?)\\s*(deg|rad)\\b",
          "captures": {
            "1": { "name": "constant.numeric.value.tcsl" },
            "2": { "name": "keyword.other.unit.angle.tcsl" }
          }
        }
      ]
    },

    "number-literal": {
      "patterns": [
        {
          "comment": "Float literal (must have digits on both sides of dot)",
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
          "name": "keyword.operator.assignment.tcsl",
          "match": "(?<![=!<>])=(?!=)"
        },
        {
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
      "match": "\\b([A-Za-z][A-Za-z0-9_]*)\\s*(=)(?!=)",
      "captures": {
        "1": { "name": "variable.parameter.named.tcsl" },
        "2": { "name": "keyword.operator.assignment.tcsl" }
      }
    },

    "identifier": {
      "name": "variable.other.tcsl",
      "match": "\\b[A-Za-z][A-Za-z0-9_]*\\b"
    }
  }
}
```

Теперь создам файл `package.json` для расширения VS Code и `language-configuration.json`:

**package.json** (корень расширения):

```json
{
  "name": "tcsl",
  "displayName": "TCSL — TextToCAD Scripting Language",
  "description": "Syntax highlighting for TCSL v1.5 (.tcsl files)",
  "version": "1.5.0",
  "publisher": "texttocad",
  "license": "MIT",
  "engines": {
    "vscode": "^1.75.0"
  },
  "categories": ["Programming Languages"],
  "contributes": {
    "languages": [
      {
        "id": "tcsl",
        "aliases": ["TCSL", "TextToCAD"],
        "extensions": [".tcsl"],
        "mimetypes": ["application/vnd.texttocad.tcsl"],
        "configuration": "./language-configuration.json"
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

**language-configuration.json:**

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

Структура файлов расширения:

```
tcsl-vscode/
├── package.json
├── language-configuration.json
├── syntaxes/
│   └── tcsl.tmLanguage.json
├── README.md
├── CHANGELOG.md
├── LICENSE
└── images/
    └── icon.png
```

---

Вот что покрывает эта грамматика и как именно она устроена.

**Зоны языка.** Каждое ключевое слово зоны (`input`, `let`, `export`) матчится вместе с последующим идентификатором и знаком `=`, что даёт разную раскраску переменным в зависимости от зоны: `variable.parameter.input` для input-параметров, `variable.other.let` для let-параметров, `variable.other.geometry` для геометрических объектов и экспортов. Конструкция `export ... as Identifier` парсит `as` как отдельное ключевое слово и подсвечивает alias-имя как `entity.name.export-alias`.

**Конструкторы.** Два паттерна: с инлайн-тегом (`box$Side(...)`) и без. Тег получает scope `entity.name.tag`, а сам конструктор — `entity.name.function.constructor`. Ключевое слово `plate` включено наравне с `box`, `cylinder`, `sphere`.

**Функции.** Трансформации, булевы операции, паттерны, селекторы, модификаторы и `tag` — каждая группа имеет собственный scope (`entity.name.function.transform`, `.boolean`, `.pattern`, `.selector`, `.modifier`, `.tag`), что позволяет настраивать цвета по категориям.

**Размерные литералы.** Четыре паттерна расположены в порядке longest match: объём → площадь → длина → угол. Каждый матчит число (с необязательным минусом и дробной частью), необязательный пробел и единицу. Число получает scope `constant.numeric.value`, единица — один из `keyword.other.unit.volume`, `.area`, `.length`, `.angle`. Такой порядок гарантирует, что `100mm3` корректно подсветится как число + единица объёма, а не как `100mm` + `3`.

**Перечисления.** Значения `X`, `Y`, `Z`, `positive`, `negative`, `both`, `inward`, `outward`, `all`, `edges`, `faces` — все подсвечены как `constant.language.enum.*`. Поскольку это bare identifiers в аргументах функций, TextMate не может проверить контекст; грамматика помечает их по значению.

**Оператор pipeline.** `|>` подсвечен как `keyword.operator.pipeline`.

**Language configuration.** Настроены: комментирование строк через `#`, автозакрытие скобок, сворачивание по маркерам `# ===...===` (типичные разделители зон в примерах), word pattern для навигации по идентификаторам.