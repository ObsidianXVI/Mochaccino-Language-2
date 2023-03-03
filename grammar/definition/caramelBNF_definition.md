# Caramel BNF (CBNF) Usage Guide

Mochaccino's grammar is described using Caramel BNF, a variation of BNF, which will be defined in this file.

# CBNF 1.0.0

## 1.0: Productions
A production rule can be expressed as follows:
```ebnf
symbol = rule;
```
- The `;` can be on a separate line, by itself
## 2.0: Symbols
- A symbol can either  be a `terminal `or `nonterminal`.
- A `terminal` does not expand to `nonterminals`.
- DO define keywords as terminals
- DO use uppercase symbols for terminal names
```ebnf
IF = "if";
END_IF = "end if";
```
- DONT define symbols for special characters (you psycho)
- DO use `@` to reference use literals that come predefined with CBNF:
```
stmt.print = PRINT @string;
```
- See [`4.0: Predefined Symbols`](#40-predefined-symbols) for a list of predefined symbols
- DO use dot notation to add hierarchy to symbols
```ebnf
foo.bar.baz = foobar;
```

## 3.0: Rule Syntax
### 3.1: Adjacence
- Adjacence, instead of concatenation using `+`, defines order in a symbol's rule
```ebnf
stmt.if = IF "(" expression ")" stmt;
```
### 3.2: Repetition
- Square brackets `[]` are used to identify a section of a rule that can be repeated
- The repetition rule is placed in curly braces `{}` immediately before the repeating section
```
stmt.block = "{" {*}[stmt] "}";
foo.bar = "{" {20}[baz] "}";
```
- The following are valid repetition rules:
    - `+`: Occurs at least once
    - `*`: Occurs 0 or more times
    - `?`: Occurs at most once (i.e., is optional)
    - `<int>`: Occurs exactly `int` number of times (omit the angled brackets when using this rule)

### 3.3: Either/Or
- The pipe character `|` can be used to express multiple either/or options
```ebnf
foo = bar | baz | qux;
```

### 3.4: Expression Hierarchy
- DO use rounded brackets `()` to enforce hierarchy among expressions and to disambiguate rules
```
stmt.var_decl = (VAR | FINAL) @identifier "=" expression ";";
```

### 3.5: RegEx Usage
- DO surround RegEx expressions with angled brackets `<>`;
- DO use the PCRE2 RegEx engine
- PREFER to paste expressions directly from [regex101.com](https://regex101.com)
- DO place modifiers immediately after the RegEx expression's surrounding brackets
```
string_literal = <\".*\">gm;
```

## 4.0: Predefined Symbols
```
@string = <\".*\">gm;

@int = @signed_int;
@signed_int = <^-?(0|[1-9]\d*)$>gm;
@unsigned_int = <^(0|[1-9]\d*)$>gm;

@double = @signed_double;
@signed_double = <^-?(0|[1-9]\d*)(\.\d+)?$>gm;
@unsigned_double = <^(0|[1-9]\d*)(\.\d+)?$>gm;

@identifier = <^\w+$>gm;
@type = @identifier ;

@eof = "";
```

## 5.0: The `.cbnf` File
- DO use the `.cbnf` extension for Caramel BNF files
- DO specify the version of CBNF in the first non-comment line of the file, via the `use` keyword:
```
# Author: Foo the Bar
use cbnf:0.0.1;

symbol = rule;
```

### 5.1: Comments
- DO use single-line comments, prefixed with `#`, to split up sections of the CBNF file
```py
##################
# Section Title
##################
```