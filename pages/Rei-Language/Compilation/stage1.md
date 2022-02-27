---
layout: page
title: Stage 1
parent: Compilation
grand_parent: Rei
nav_order: 1
---

All about the first steps that compilers usually take to compile a string

We usually lex, parse, analyze for semantics to build something that can then be transformed into a lower level representation. This IR is closer to machine code and can be further analyzed and optimised, e.g. LLVM. Finally it can be assembled into actual machine code for a certain platform, e.g. x86_64 linux elf

## Lexical Analysers

Lexical analyzers do two things: Scanning and Lexing

Token -> pair consisting of a label and an attribute value. The label is an abstract symbol that represents a lexical unit. E.g. a keyword, identifier, etc

Pattern -> description of the form that lexemes of a token can take. Simply a sequence of chars that form the keyword

Lexeme -> sequence of chars in the source that matches the pattern for a token. Can be identified by the analyzer as an instance of that token

| Token | Description | Examples |
| --- | --- | --- |
| let | r"let" | `let` |
| identifier | r"[\w\W]{1}[\w\W\d]+" | `val`, `student_id` |
| number | r"\d{0-1}\.{0-1}+\d+" | 0, 4.055215, -90251 |
| literal string (with newlines) | r"\\"[.*\n]+\\""| "Hello, World!" |

So 1 token represents all its identifiers, though one or more tokens can represent a constant
- we need some way to prioritise certain tokens over the other

### Attributes

If more than a single lexeme can match a pattern, the analyzer must provide extra info about that lexeme to the next phase. E.g. if we have `0` and `1`, we need the code generator to know which lexeme was found, hence we would turn back to the parser

- tokens have at most a single associated attribute. This attribute can have several substructures
- the most important is the token `identifier` which we need to associate with a lot of info. `identifier` tokens should be kept in a symbol table, which contains its lexeme, type, location
- so for `identifier` tokens, we assign a `symbtab_pointer` attribute to it since it has a bunch of important subinfo stored in a global container



## Syntax Analysis

