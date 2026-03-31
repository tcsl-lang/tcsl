

# TCSL v1.5 — Формальная EBNF-грамматика

```ebnf
(* ================================================================= *)
(* TCSL — TextToCAD Scripting Language                                *)
(* Formal EBNF Grammar — tcsl.ebnf                                   *)
(* Version: 1.5                                                       *)
(* Date: 2026-03-18                                                   *)
(* Conforms to: ISO/IEC 14977 (EBNF)                                 *)
(*                                                                    *)
(* Derived from TCSL Specification v1.5.                              *)
(* This file is the single source of truth for TCSL syntax.           *)
(*                                                                    *)
(* Changes from v1.4 draft EBNF:                                     *)
(*   - Literals are always non-negative (L1 fix)                      *)
(*   - InputDecl has its own optional leading minus (L1 fix)          *)
(*   - Units are reserved words (L3 fix)                              *)
(*   - Longest-match rule for units codified (L2 fix)                 *)
(*   - TagArgList uses Identifier, not StringArg (L4 fix)             *)
(*   - WS / OptWS formally defined (S1 fix)                           *)
(*   - ArgList split into AllPositional/AllNamed/MixedArgs (S2 fix)   *)
(*   - Parenthesis nesting depth noted (S3 fix)                       *)
(*   - Boolean ops split by arity: multi vs binary (S5 fix)           *)
(*   - select_by_tag gains mandatory "expect" parameter (M2 fix)      *)
(*   - Division-by-zero constraint added (M4 fix)                     *)
(*   - Integer-context constraint added (M5 fix)                      *)
(*   - New error codes: E018–E021, R008–R009                          *)
(*   - New enum: Expect                                               *)
(*                                                                    *)
(* Conventions (ISO/IEC 14977):                                       *)
(*   =     definition                                                 *)
(*   ,     concatenation                                              *)
(*   |     alternation                                                *)
(*   [ ]   optional (0 or 1)                                          *)
(*   { }   repetition (0 or more)                                     *)
(*   ( )   grouping                                                   *)
(*   (* *) comment                                                    *)
(*   "x"   terminal string                                            *)
(*   ?x?   special sequence (described in prose)                      *)
(*   ;     rule terminator                                            *)
(* ================================================================= *)


(* ================================================================= *)
(* §1  CHARACTER CLASSES                                              *)
(* ================================================================= *)

Letter          = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I"
                | "J" | "K" | "L" | "M" | "N" | "O" | "P" | "Q" | "R"
                | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z"
                | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i"
                | "j" | "k" | "l" | "m" | "n" | "o" | "p" | "q" | "r"
                | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z" ;

Digit           = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;

HT              = ?U+0009 horizontal tab? ;

SP              = " " ;                      (* U+0020 space *)

AnyCharExceptNL = ?any Unicode character except U+000A and U+000D? ;


(* ================================================================= *)
(* §2  WHITESPACE AND LINE STRUCTURE                                  *)
(*     Spec reference: §5.2                                           *)
(* ================================================================= *)

WS              = ( SP | HT ) , { SP | HT } ;        (* 1 or more *)
OptWS           = { SP | HT } ;                       (* 0 or more *)

Newline         = ( ?U+000D? , ?U+000A? )             (* CR+LF     *)
                | ?U+000A?                             (* LF        *)
                | ?U+000D? ;                           (* CR        *)


(* ================================================================= *)
(* §3  COMMENTS AND BLANK LINES                                      *)
(*     Spec reference: §5.3                                           *)
(* ================================================================= *)

Comment         = "#" , { AnyCharExceptNL } ;

BlankOrComment  = OptWS , [ Comment ] , Newline ;

(* Blank lines and comment-only lines may appear anywhere in the      *)
(* program without affecting the current zone.                        *)


(* ================================================================= *)
(* §4  IDENTIFIERS                                                    *)
(*     Spec reference: §5.4                                           *)
(* ================================================================= *)

Identifier      = Letter , { Letter | Digit | "_" } ;

(* CONSTRAINT SC-01: length(Identifier) <= 64.          → TCSL_E009  *)
(* CONSTRAINT SC-02: Identifier ∉ ReservedWords.        → TCSL_E008  *)
(* CONSTRAINT SC-02: Identifier ∉ UnitKeywords.         → TCSL_E008  *)
(* CONSTRAINT:       Identifier must not start with "_".→ TCSL_E010  *)
(*   (Already enforced by the rule: first symbol is Letter.)          *)


(* ================================================================= *)
(* §5  RESERVED WORDS                                                 *)
(*     Spec reference: §5.5                                           *)
(*     Listed for reference; enforced by the lexer.                   *)
(* ================================================================= *)

(* --- Fully reserved keywords ---                                    *)
(*                                                                    *)
(* "input"  "let"  "export"  "as"                                     *)
(* "box"  "plate"  "cylinder"  "sphere"                               *)
(* "translate"  "rotate"  "scale"                                     *)
(* "union"  "cut"  "subtract"  "intersect"                            *)
(* "linear_pattern"  "circular_pattern"                               *)
(* "select_edges_by_axis"   "select_edges_by_length"                  *)
(* "select_edges_parallel"  "select_edges_at_height"                  *)
(* "select_faces_by_axis"   "select_faces_by_area"                    *)
(* "select_by_tag"                                                    *)
(* "fillet"  "chamfer"  "shell"                                       *)
(* "tag"                                                              *)
(*                                                                    *)
(* --- Reserved unit keywords (also forbidden as identifiers) ---     *)
(*                                                                    *)
(* "mm"  "cm"  "m"  "in"  "ft"                                       *)
(* "mm2" "cm2" "m2" "in2" "ft2"                                      *)
(* "mm3" "cm3" "m3" "in3" "ft3"                                      *)
(* "deg" "rad"                                                        *)


(* ================================================================= *)
(* §6  NUMERIC LITERALS                                               *)
(*     Spec reference: §5.6                                           *)
(*                                                                    *)
(*     Literals are ALWAYS non-negative (v1.5 fix for L1).            *)
(*     Negation is handled by:                                        *)
(*       - Factor (in expressions, §9)                                *)
(*       - InputDecl (leading minus before literal, §11)              *)
(* ================================================================= *)

IntLiteral      = Digit , { Digit } ;

FloatLiteral    = Digit , { Digit } , "." , Digit , { Digit } ;

NumberLiteral   = FloatLiteral | IntLiteral ;

(* PARSE NOTE: FloatLiteral is attempted first so that the decimal    *)
(* point is consumed correctly and not mistaken for end-of-number.    *)


(* ================================================================= *)
(* §7  UNITS OF MEASUREMENT                                          *)
(*     Spec reference: §5.7, §5.7.1                                  *)
(*                                                                    *)
(*     Longest-match rule (v1.5 fix for L2):                          *)
(*     When the lexer encounters letters/digits immediately after a   *)
(*     NumberLiteral, it tries units in this order:                   *)
(*       1. VolumeUnit  (3-char suffix: mm3, cm3, m3, in3, ft3)      *)
(*       2. AreaUnit    (3-char suffix: mm2, cm2, m2, in2, ft2)      *)
(*       3. LengthUnit  (1–2 char: mm, cm, ft, in, m)               *)
(*       4. AngleUnit   (2–3 char: deg, rad)                         *)
(*     If no unit matches, the suffix is the start of the next token. *)
(*     If no whitespace separates them → TCSL_E018.                   *)
(* ================================================================= *)

VolumeUnit      = "mm3" | "cm3" | "m3" | "in3" | "ft3" ;

AreaUnit        = "mm2" | "cm2" | "m2" | "in2" | "ft2" ;

LengthUnit      = "mm" | "cm" | "ft" | "in" | "m" ;

AngleUnit       = "deg" | "rad" ;

Unit            = VolumeUnit
                | AreaUnit
                | LengthUnit
                | AngleUnit ;

(* CONSTRAINT: VolumeUnit checked before AreaUnit before LengthUnit   *)
(* to ensure "mm3" is not misread as "mm" + "3".                      *)


(* ================================================================= *)
(* §8  DIMENSIONAL AND DIMENSIONLESS LITERALS                         *)
(*     Spec reference: §5.8                                           *)
(* ================================================================= *)

DimensionalLiteral   = NumberLiteral , OptWS , Unit ;

DimensionlessLiteral = NumberLiteral ;

Literal              = DimensionalLiteral
                     | DimensionlessLiteral ;

(* PARSE NOTE: DimensionalLiteral is attempted first. If a Unit       *)
(* follows the number (possibly separated by whitespace), it binds    *)
(* to the number.                                                     *)

(* InputLiteral adds an optional leading minus for the INPUT zone.    *)
(* This is the ONLY place outside Factor where minus is allowed.      *)

InputLiteral         = [ "-" , OptWS ] , Literal ;


(* ================================================================= *)
(* §9  EXPRESSIONS                                                    *)
(*     Spec reference: §9.1                                           *)
(*                                                                    *)
(*     Standard precedence (highest to lowest):                       *)
(*       1. Parenthesised sub-expression                              *)
(*       2. Unary minus  (prefix "-")                                 *)
(*       3. Multiplicative  (* , /)                                   *)
(*       4. Additive  (+, -)                                          *)
(*                                                                    *)
(*     Unary minus is defined ONLY at the Factor level (v1.5 L1 fix).*)
(*     Parenthesis nesting depth >= 16 required (v1.5 S3 fix).       *)
(*     Exceeding limit → TCSL_E020.                                   *)
(* ================================================================= *)

Expression      = Term , { OptWS , AdditiveOp , OptWS , Term } ;

AdditiveOp      = "+" | "-" ;

Term            = Factor , { OptWS , MultiplicativeOp , OptWS , Factor } ;

MultiplicativeOp = "*" | "/" ;

Factor          = [ "-" , OptWS ] , Atom ;

Atom            = DimensionalLiteral
                | DimensionlessLiteral
                | Identifier
                | "(" , OptWS , Expression , OptWS , ")" ;

(* CONSTRAINT SC-06: dimensional type algebra per §6.3.               *)
(*   Mismatches → TCSL_E015 or TCSL_R001.                            *)
(* CONSTRAINT: division by zero → TCSL_R008.                          *)
(* CONSTRAINT: parenthesis nesting > 16 → TCSL_E020.                 *)


(* ================================================================= *)
(* §10  ARGUMENT LISTS                                                *)
(*      Spec reference: §10.10.1                                      *)
(*                                                                    *)
(*      v1.5 fix for S2: positional args must precede all named args. *)
(*      Three valid forms: all positional, all named, or mixed        *)
(*      (positional first, then named). Violation → TCSL_E019.        *)
(* ================================================================= *)

ArgList         = AllPositional
                | AllNamed
                | MixedArgs ;

AllPositional   = PositionalArg , { OptWS , "," , OptWS , PositionalArg } ;

AllNamed        = NamedArg , { OptWS , "," , OptWS , NamedArg } ;

MixedArgs       = PositionalArg , { OptWS , "," , OptWS , PositionalArg }
                  , OptWS , "," , OptWS
                  , NamedArg , { OptWS , "," , OptWS , NamedArg } ;

NamedArg        = Identifier , OptWS , "=" , OptWS , ArgValue ;

PositionalArg   = ArgValue ;

ArgValue        = Expression ;

(* CONSTRAINT SC-15: positional arguments must precede all named      *)
(*   arguments within a single ArgList. Violation → TCSL_E019.        *)
(* CONSTRAINT SC-19: enum values appear as bare Identifiers in        *)
(*   ArgValue position. The semantic analyser validates them against   *)
(*   the expected enum type of the parameter.                         *)


(* ================================================================= *)
(* §11  ZONE 1 — INPUT                                               *)
(*      Spec reference: §8                                            *)
(*                                                                    *)
(*      InputLiteral allows an optional leading minus — the ONLY      *)
(*      place outside Factor where negation is permitted (v1.5 L1).   *)
(* ================================================================= *)

ZoneInput       = { InputDecl | BlankOrComment } ;

InputDecl       = "input" , WS , Identifier
                  , OptWS , "=" , OptWS
                  , InputLiteral
                  , OptWS , [ Comment ]
                  , Newline ;

(* CONSTRAINT SC-04: right-hand side is InputLiteral only — no        *)
(*   expressions, no Identifier references. Violation → TCSL_E012.    *)
(* CONSTRAINT SC-03: no duplicate Identifier declarations across all  *)
(*   zones. Violation → TCSL_E006.                                    *)
(* CONSTRAINT SC-11: runtime override must match declared type.       *)
(*   Violation → TCSL_R006.                                           *)


(* ================================================================= *)
(* §12  ZONE 2 — LET                                                 *)
(*      Spec reference: §9                                            *)
(* ================================================================= *)

ZoneLet         = { LetDecl | BlankOrComment } ;

LetDecl         = "let" , WS , Identifier
                  , OptWS , "=" , OptWS
                  , Expression
                  , OptWS , [ Comment ]
                  , Newline ;

(* CONSTRAINT SC-05: Expression may reference only previously         *)
(*   declared input and let identifiers. Forward refs → TCSL_E013.    *)
(* CONSTRAINT SC-03: no duplicate Identifier declarations. → E006.    *)
(* CONSTRAINT: let-parameters cannot be overridden externally. → E014.*)


(* ================================================================= *)
(* §13  CONSTRUCTORS                                                  *)
(*      Spec reference: §10.2                                         *)
(*                                                                    *)
(*      "plate" is a semantic alias for "box" (v1.5 M1 clarification):*)
(*        - box   → volumetric parts (sides, posts, blocks)           *)
(*        - plate → flat parts (shelves, tops, panels, facades)       *)
(*      Geometrically identical; distinction is documentary.          *)
(* ================================================================= *)

ConstructorKeyword = "box" | "plate" | "cylinder" | "sphere" ;

ConstructorCall = ConstructorKeyword
                  , [ "$" , Identifier ]      (* inline tag *)
                  , OptWS
                  , "(" , OptWS , ArgList , OptWS , ")" ;

(* box(width, depth, height)     — Length, Length, Length → Geometry   *)
(* plate(width, depth, height)   — Length, Length, Length → Geometry   *)
(* cylinder(radius, height)      — Length, Length         → Geometry   *)
(* sphere(radius)                — Length                 → Geometry   *)

(* CONSTRAINT SC-07: all constructor parameters must be > 0. → R002.  *)


(* ================================================================= *)
(* §14  TRANSFORMS                                                    *)
(*      Spec reference: §10.3                                         *)
(* ================================================================= *)

TransformKeyword = "translate" | "rotate" | "scale" ;

TransformCall   = TransformKeyword
                  , OptWS
                  , "(" , OptWS , ArgList , OptWS , ")" ;

(* translate(x, y, z)                                                 *)
(*   — Length, Length, Length                                          *)
(*   : Geometry → Geometry                                            *)
(*                                                                    *)
(* rotate(angle, ax, ay, az, cx, cy, cz)                              *)
(*   — Angle, Scalar×3, Length×3                                      *)
(*   : Geometry → Geometry                                            *)
(*   cx, cy, cz default to 0 mm if omitted.                          *)
(*                                                                    *)
(* scale(sx, sy, sz)                                                  *)
(*   — Scalar, Scalar, Scalar                                        *)
(*   : Geometry → Geometry                                            *)


(* ================================================================= *)
(* §15  BOOLEAN OPERATIONS                                            *)
(*      Spec reference: §10.4, §10.4.1                                *)
(*                                                                    *)
(*      v1.5 fix for S5: explicit arity split.                        *)
(*        - union / intersect: 2 or more operands (multi-arity)       *)
(*        - cut / subtract:    exactly 2 operands (binary)            *)
(*      Arity violation → TCSL_E021.                                  *)
(* ================================================================= *)

(* --- Multi-arity keywords (2+ operands) ---                         *)

BooleanMultiKeyword  = "union" | "intersect" ;

(* --- Binary keywords (exactly 2 operands) ---                       *)

BooleanBinaryKeyword = "cut" | "subtract" ;

(* --- Standalone form: all operands listed explicitly ---             *)

BooleanOp       = BooleanMultiOp | BooleanBinaryOp ;

BooleanMultiOp  = BooleanMultiKeyword
                  , OptWS
                  , "(" , OptWS
                  , Identifier , OptWS , "," , OptWS , Identifier
                  , { OptWS , "," , OptWS , Identifier }
                  , OptWS , ")" ;

BooleanBinaryOp = BooleanBinaryKeyword
                  , OptWS
                  , "(" , OptWS
                  , Identifier , OptWS , "," , OptWS , Identifier
                  , OptWS , ")" ;

(* --- Pipeline form: left operand is implicit from pipe ---          *)

BooleanPipeCall  = BooleanPipeMulti | BooleanPipeBinary ;

BooleanPipeMulti = BooleanMultiKeyword
                   , OptWS
                   , "(" , OptWS
                   , Identifier
                   , { OptWS , "," , OptWS , Identifier }
                   , OptWS , ")" ;

BooleanPipeBinary = BooleanBinaryKeyword
                    , OptWS
                    , "(" , OptWS
                    , Identifier
                    , OptWS , ")" ;

(* CONSTRAINT SC-16: cut/subtract — exactly 2 operands total.         *)
(* CONSTRAINT SC-17: union/intersect — at least 2 operands total.     *)


(* ================================================================= *)
(* §16  PATTERNS                                                      *)
(*      Spec reference: §10.5                                         *)
(* ================================================================= *)

PatternKeyword  = "linear_pattern" | "circular_pattern" ;

PatternCall     = PatternKeyword
                  , OptWS
                  , "(" , OptWS , ArgList , OptWS , ")" ;

(* linear_pattern(count, dx, dy, dz)                                  *)
(*   — Scalar, Length, Length, Length                                  *)
(*   : Geometry → Geometry                                            *)
(*                                                                    *)
(* circular_pattern(count, angle, cx, cy, cz, ax, ay, az)             *)
(*   — Scalar, Angle, Length×3, Scalar×3                              *)
(*   : Geometry → Geometry                                            *)

(* CONSTRAINT SC-18: count >= 2 and must be integer. → R002, R009.    *)


(* ================================================================= *)
(* §17  SELECTORS                                                     *)
(*      Spec reference: §10.6                                         *)
(*                                                                    *)
(*      v1.5 fix for M2: select_by_tag gains mandatory "expect"       *)
(*      parameter (Enum Expect: "edges" | "faces") so that the        *)
(*      static type checker can determine the result type without     *)
(*      execution.                                                    *)
(* ================================================================= *)

EdgeSelectorKeyword = "select_edges_by_axis"
                    | "select_edges_by_length"
                    | "select_edges_parallel"
                    | "select_edges_at_height" ;

FaceSelectorKeyword = "select_faces_by_axis"
                    | "select_faces_by_area" ;

TagSelectorKeyword  = "select_by_tag" ;

SelectorKeyword = EdgeSelectorKeyword
                | FaceSelectorKeyword
                | TagSelectorKeyword ;

SelectorCall    = SelectorKeyword
                  , OptWS
                  , "(" , OptWS , ArgList , OptWS , ")" ;

(* select_edges_by_axis(axis)                                         *)
(*   — Enum Axis                                                      *)
(*   : Geometry → EdgeSet                                             *)
(*                                                                    *)
(* select_edges_by_length(min, max)                                   *)
(*   — Length, Length                                                  *)
(*   : Geometry → EdgeSet                                             *)
(*                                                                    *)
(* select_edges_parallel(dx, dy, dz)                                  *)
(*   — Scalar, Scalar, Scalar                                        *)
(*   : Geometry → EdgeSet                                             *)
(*                                                                    *)
(* select_edges_at_height(z, tolerance)                               *)
(*   — Length, Length                                                  *)
(*   : Geometry → EdgeSet                                             *)
(*                                                                    *)
(* select_faces_by_axis(axis, keep)                                   *)
(*   — Enum Axis, Enum Keep                                           *)
(*   : Geometry → FaceSet                                             *)
(*                                                                    *)
(* select_faces_by_area(min, max)                                     *)
(*   — Area, Area                                                     *)
(*   : Geometry → FaceSet                                             *)
(*                                                                    *)
(* select_by_tag(name, expect)                                        *)
(*   — Identifier, Enum Expect                                        *)
(*   : Geometry → EdgeSet (if expect=edges)                           *)
(*              | FaceSet (if expect=faces)                            *)

(* CONSTRAINT SC-10: empty selector result at runtime → TCSL_R005.    *)


(* ================================================================= *)
(* §18  MODIFIERS                                                     *)
(*      Spec reference: §10.7                                         *)
(* ================================================================= *)

ModifierKeyword = "fillet" | "chamfer" | "shell" ;

ModifierCall    = ModifierKeyword
                  , OptWS
                  , "(" , OptWS , ArgList , OptWS , ")" ;

(* fillet(radius)                                                     *)
(*   — Length                                                         *)
(*   : EdgeSet → Geometry                                             *)
(*                                                                    *)
(* chamfer(size)                                                      *)
(*   — Length                                                         *)
(*   : EdgeSet → Geometry                                             *)
(*                                                                    *)
(* shell(thickness, kind)                                             *)
(*   — Length, Enum Kind                                              *)
(*   : FaceSet → Geometry                                             *)
(*   kind defaults to "inward" if omitted.                            *)


(* ================================================================= *)
(* §19  TAG                                                           *)
(*      Spec reference: §10.8                                         *)
(*                                                                    *)
(*      v1.5 fix for L4: StringArg/ScopeArg replaced by Identifier.  *)
(*      First argument "name" is an Identifier (tag name).            *)
(*      Second argument "scope" is optional named; value is Enum Scope.*)
(* ================================================================= *)

TagCall         = "tag"
                  , OptWS
                  , "(" , OptWS , TagArgList , OptWS , ")" ;

TagArgList      = Identifier                           (* tag name *)
                  , [ OptWS , "," , OptWS
                      , "scope" , OptWS , "=" , OptWS
                      , Identifier                     (* Enum Scope value *)
                    ] ;

(* tag(Name)                    — scope defaults to "all"             *)
(* tag(Name, scope = edges)     — explicit scope                      *)
(* tag(Name, scope = faces)     — explicit scope                      *)
(*                                                                    *)
(* : Geometry → Geometry                                              *)
(*                                                                    *)
(* CONSTRAINT SC-19: scope value ∈ { "all", "edges", "faces" }.       *)


(* ================================================================= *)
(* §20  PIPELINE                                                      *)
(*      Spec reference: §10.9                                         *)
(*                                                                    *)
(*      The pipeline operator |> is left-associative.                 *)
(*      The entire pipeline must be on a single logical line.         *)
(* ================================================================= *)

PipelineStep    = TransformCall
                | PatternCall
                | SelectorCall
                | ModifierCall
                | BooleanPipeCall
                | TagCall ;

Pipeline        = "|>" , OptWS , PipelineStep ;

(* CONSTRAINT SC-08 (type compatibility per §10.9):                   *)
(*                                                                    *)
(*   Geometry → translate, rotate, scale,                             *)
(*              union, cut, subtract, intersect,                      *)
(*              linear_pattern, circular_pattern,                     *)
(*              select_edges_by_axis, select_edges_by_length,         *)
(*              select_edges_parallel, select_edges_at_height,        *)
(*              select_faces_by_axis, select_faces_by_area,           *)
(*              select_by_tag, tag                                    *)
(*                                                                    *)
(*   EdgeSet  → fillet, chamfer                                       *)
(*                                                                    *)
(*   FaceSet  → shell                                                 *)
(*                                                                    *)
(*   Violation → TCSL_R003.                                           *)


(* ================================================================= *)
(* §21  GEOMETRY EXPRESSIONS                                          *)
(*      Spec reference: §10.1                                         *)
(* ================================================================= *)

GeometryAtom    = ConstructorCall
                | BooleanOp
                | Identifier ;

GeometryExpr    = GeometryAtom , { OptWS , Pipeline } ;


(* ================================================================= *)
(* §22  ZONE 3 — GEOMETRY                                            *)
(*      Spec reference: §10                                           *)
(* ================================================================= *)

ZoneGeometry    = ( GeometryAssign | BlankOrComment )
                  , { GeometryAssign | BlankOrComment } ;

GeometryAssign  = Identifier
                  , OptWS , "=" , OptWS
                  , GeometryExpr
                  , OptWS , [ Comment ]
                  , Newline ;

(* CONSTRAINT: at least one GeometryAssign required (§7).             *)
(* CONSTRAINT SC-20: bare arithmetic assignment forbidden → TCSL_E002.*)
(*   The RHS must begin with a ConstructorCall, BooleanOp, or a       *)
(*   previously defined Geometry Identifier.                          *)


(* ================================================================= *)
(* §23  ZONE 4 — EXPORT                                              *)
(*      Spec reference: §11                                           *)
(* ================================================================= *)

ZoneExport      = ( ExportDecl | BlankOrComment )
                  , { ExportDecl | BlankOrComment } ;

ExportDecl      = "export" , WS , Identifier
                  , [ WS , "as" , WS , Identifier ]
                  , OptWS , [ Comment ]
                  , Newline ;

(* CONSTRAINT SC-09:  exported Identifier must have type Geometry.    *)
(*   Violation → TCSL_R004.                                           *)
(* CONSTRAINT SC-12:  at least one ExportDecl required.               *)
(*   Violation → TCSL_R007.                                           *)
(* CONSTRAINT SC-13:  all as-Identifiers must be unique.              *)
(*   Violation → TCSL_E016.                                           *)
(* CONSTRAINT SC-14:  each Geometry Identifier exported at most once. *)
(*   Violation → TCSL_E017.                                           *)
(* CONSTRAINT:        as-Identifier follows standard Identifier rules *)
(*   (§5.4): ASCII letters, digits, underscore, max 64 chars.        *)


(* ================================================================= *)
(* §24  PROGRAM (top level)                                           *)
(*      Spec reference: §7                                            *)
(* ================================================================= *)

Program         = { BlankOrComment }
                  , ZoneInput
                  , { BlankOrComment }
                  , ZoneLet
                  , { BlankOrComment }
                  , ZoneGeometry
                  , { BlankOrComment }
                  , ZoneExport
                  , { BlankOrComment }
                  , ?end of file? ;

(* CONSTRAINT SC-21: zones must appear in order                       *)
(*   INPUT → LET → GEOMETRY → EXPORT.                                *)
(*   Appearing out of order → TCSL_E005.                              *)
(*                                                                    *)
(* CONSTRAINT: ZoneInput and ZoneLet may be empty (zero declarations).*)
(* CONSTRAINT: ZoneGeometry must contain >= 1 GeometryAssign.         *)
(* CONSTRAINT: ZoneExport must contain >= 1 ExportDecl.               *)
(*   Violation → TCSL_R007.                                           *)


(* ================================================================= *)
(* §25  ENUMERATIONS                                                  *)
(*      Spec reference: §13                                           *)
(*                                                                    *)
(*      Enum values appear as bare Identifiers in argument positions. *)
(*      The parser treats them as Identifiers; semantic analysis      *)
(*      validates against the expected enum type.                     *)
(* ================================================================= *)

(* Axis   = "X" | "Y" | "Z"                                          *)
(*   Used in: select_edges_by_axis, select_faces_by_axis              *)

(* Keep   = "positive" | "negative" | "both"                         *)
(*   Used in: select_faces_by_axis                                    *)

(* Kind   = "inward" | "outward"                                     *)
(*   Used in: shell                                                   *)

(* Scope  = "all" | "edges" | "faces"                                *)
(*   Used in: tag                                                     *)

(* Expect = "edges" | "faces"                                        *)
(*   Used in: select_by_tag (v1.5 new enum, M2 fix)                  *)


(* ================================================================= *)
(* §26  SEMANTIC CONSTRAINTS SUMMARY                                  *)
(*      (Not expressible in EBNF)                                     *)
(* ================================================================= *)

(* SC-01  Identifier max length: 64 characters.           → E009     *)
(* SC-02  Identifier ∉ ReservedWords ∪ UnitKeywords.      → E008     *)
(* SC-03  No duplicate Identifier across all zones.        → E006     *)
(* SC-04  input RHS: InputLiteral only (no expressions).   → E012     *)
(* SC-05  let RHS: only backward references.               → E013     *)
(* SC-06  Dimensional type algebra per §6.3.               → E015,    *)
(*                                                            R001    *)
(* SC-07  Constructor parameters > 0.                      → R002     *)
(* SC-08  Pipeline type compatibility per §10.9.           → R003     *)
(* SC-09  export targets must be Geometry.                 → R004     *)
(* SC-10  Selector empty result at runtime.                → R005     *)
(* SC-11  Runtime input override type match.               → R006     *)
(* SC-12  At least one export.                             → R007     *)
(* SC-13  Unique as-identifiers.                           → E016     *)
(* SC-14  No duplicate exports of same identifier.         → E017     *)
(* SC-15  Positional args must precede named args.         → E019     *)
(* SC-16  cut/subtract: exactly 2 operands total.          → E021     *)
(* SC-17  union/intersect: at least 2 operands total.      → E021     *)
(* SC-18  linear_pattern / circular_pattern count >= 2,    → R002,    *)
(*        integer.                                            R009    *)
(* SC-19  Enum values must match expected enum type.       (semantic) *)
(* SC-20  No bare arithmetic assignments in GEOMETRY zone. → E002     *)
(* SC-21  Zone ordering is strictly monotonic.             → E005     *)
(* SC-22  Unit keywords have lexical priority over         → E018     *)
(*        identifiers when immediately following a                    *)
(*        numeric literal (longest match).                            *)
(* SC-23  Division by zero.                                → R008     *)
(* SC-24  Non-integer value in integer-required context.   → R009     *)
(* SC-25  Parenthesis nesting depth > 16.                  → E020     *)


(* ================================================================= *)
(* §27  ERROR CODE REFERENCE                                          *)
(*      Spec reference: §14                                           *)
(* ================================================================= *)

(* --- Syntactic errors (TCSL_E) ---                                  *)
(*                                                                    *)
(* E001  Missing unit on dimensional argument                         *)
(* E002  Bare arithmetic assignment in GEOMETRY zone                  *)
(* E003  Pipeline |> on a new line (line continuation not supported)  *)
(* E004  Unknown keyword / identifier                                 *)
(* E005  Zone ordering violation                                      *)
(* E006  Duplicate identifier declaration                             *)
(* E007  Invalid character in identifier                              *)
(* E008  Identifier is a reserved word or reserved unit               *)
(* E009  Identifier exceeds 64 characters                             *)
(* E010  Leading underscore in identifier                             *)
(* E011  Empty program (no instructions at all)                       *)
(* E012  Arithmetic expression in input declaration                   *)
(* E013  Reference to undeclared identifier                           *)
(* E014  Attempt to externally override a let-parameter               *)
(* E015  Dimensional result outside L⁰–L³ range                      *)
(* E016  Duplicate as-name in export                                  *)
(* E017  Duplicate export of same identifier                          *)
(* E018  Unknown unit or missing space between number and identifier  *)
(* E019  Positional argument after named argument                     *)
(* E020  Parenthesis nesting depth exceeded (>= 16)                   *)
(* E021  Boolean operation arity violation                            *)
(*                                                                    *)
(* --- Semantic / runtime errors (TCSL_R) ---                         *)
(*                                                                    *)
(* R001  Incompatible dimensions in arithmetic                        *)
(* R002  Zero or negative constructor parameter                       *)
(* R003  Incompatible type at pipeline input                          *)
(* R004  Export of non-Geometry type                                  *)
(* R005  Empty selector result (runtime)                              *)
(* R006  Type mismatch on runtime input override                      *)
(* R007  No export declarations in program                            *)
(* R008  Division by zero                                             *)
(* R009  Non-integer value in integer-required context                *)
(*                                                                    *)
(* --- Warnings (TCSL_W) ---                                          *)
(*                                                                    *)
(* W001  as-identifier lacks material keyword                         *)
(* W002  Declared input or let is unused                              *)


(* ================================================================= *)
(* §28  GRAMMAR STATISTICS                                            *)
(* ================================================================= *)

(* Production rules:    62                                            *)
(* Sections:            28                                            *)
(* Semantic constraints: 25 (SC-01 through SC-25)                     *)
(* Error codes:         21 syntactic (E001–E021)                      *)
(*                       9 semantic  (R001–R009)                      *)
(*                       2 warnings  (W001–W002)                      *)
(*                                                                    *)
(* LL(k) compatibility: k <= 3                                        *)
(*   Lookahead needed for:                                            *)
(*     - DimensionalLiteral vs DimensionlessLiteral (number + unit?)  *)
(*     - NamedArg vs PositionalArg (Identifier + "=" ?)               *)
(*     - BooleanMultiKeyword vs BooleanBinaryKeyword (keyword check)  *)


(* ================================================================= *)
(* END OF GRAMMAR — tcsl.ebnf v1.5                                   *)
(* ================================================================= *)
```
