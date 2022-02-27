---
layout: page
title: Stage 1
parent: Compilation
grand_parent: Rei
nav_order: 1
---

## Overview

All about the first steps that compilers usually take to compile a string

We usually tokenise and parse a long string (e.g. combining files into namespace {} mod_strings automatically) to build something that can then be transformed into a lower level representation.

This IR is closer to machine code and can be further analyzed and optimised, e.g. LLVM. Finally it can be assembled into actual machine code for a certain platform, e.g. x86_64 linux elf

## Lexical Analysis

Lexical analyzers do two things: Scanning and Lexing. They basically tokenise a fixed sequence of input data into a list of key: value pairs representing a list directives/instructions

- some directives can be resolved at compile time, like `let x = 5`. Some cannot, like `fn(x: Int) -> none => this.y = x` which usually gets compiled into operations on identifiers on the stack and heap instead of immediates/.data values
- adding things like `static` and `const` makes things even more interesting and uses `.rodata` and may require prev rules to be changed

Token -> pair consisting of a label and an attribute value. The label is an abstract symbol that represents a lexical unit. E.g. a keyword, identifier, etc

Pattern -> description of the form that lexemes of a token can take. Simply a sequence of chars that form the keyword

Lexeme -> sequence of chars in the source that matches the pattern for a token. Can be identified by the analyzer as an instance of that token

| Token | Description | Examples |
| --- | --- | --- |
| let | r"let" | `let` |
| identifier | r"[\w\W]{1}[\w\W\d]+" | `val`, `student_id` |
| numeric | r"\d{0-1}\.{0-1}+\d+" | 0, 4.055215, -90251 |
| literal string (with newlines) | r"\\"[.*\n]+\\""| "Hello, World!" |

So 1 token represents all its identifiers, though one or more tokens can represent a constant

- we need some way to prioritise certain tokens over the other

### Attributes

If more than a single lexeme can match a pattern, the analyzer must provide extra info about that lexeme to the next phase. E.g. if we have `0` and `1`, we need the code generator to know which lexeme was found, hence we would turn back to the parser

- tokens have at most a single associated attribute. This attribute can have several substructures
- the most important is the token `identifier` which we need to associate with a lot of info. `identifier` tokens should be kept in a symbol table, which contains its lexeme, type, location
- so for `identifier` tokens, we assign a `symbtab_pointer` attribute to it since it has a bunch of important subinfo stored in a global container

Example: `E = M * C ** 2`

- the lexed token: attribute pairs

```
<id, pointer to local symtab for E>
<assign_op>
<id, pointer to local symtab for M>
<mult_op>
<id, pointer to local symtab for C>
<exp_op>
<numeric, int32 value 2>
```

Note how there is no way for the lexer to tell if a certain sequence is 'correct'. If you have in rei:

```
go die (person: Person) {
    person.dead()
}
```

we have several things:

- `go` and `die` get transformed into `identifier` tokens, which dont mean anything, and is nowhere close to a correct function declaration
- how do we know type `Person` exists with a method `@instancemethod fn dead()`?

The answer is we dont. The lexer's job is to produce tokens -> label: attribute pairs of 'preliminary' meaning within the syntax itself or "local sequences", not check for semantics. To do that, we would have to pass them onto the parser that checks for meaning from the entire syntax and context for the list of tokens

### Error Recovery

Maybe a sequence of characters makes the analyzer unable to proceed since none of the patterns for tokens matches any prefix of the reamining input. This can happen, but idk how. To recover we can:

- delete successive characters until the analyzer can find a well formed token
- delete a single character from the reamining input
- insert a missing character into the remaining input
- replace a character by another character
- transpose two adjacent characters

We can try these transformations to repair the input sequence. Easiest way is to whether a prefix of the remaining input can be transformed into a valid lexeme by a single transform. But we want a more general strategy to find the smallest number of transformations needed to convert the source code into one that consists only of valid lexemes -> would be quite expensive, but languages like rust usually implement a small fixer that works for the first error only and give an error message like "undefined token X did you mean Y?"

Example decomposition of Rei:

```
// return the value of a result
fn get_val(res: Result) -> Val {
    return res.val()
}
```

```
<comment>

<function_decl>
<identifier symtab pointer to get_val>
<bracket_left>
<identifier symtab pointer to res>
<colon_operator>
<identifier symtab pointer to Result>
<bracket_right>
<right_arrow>
<identifier symtab pointer to Val>
<brace_left>

<return>
<identifier symtab pointer to res>
<dot_operator>
<identifier symtab pointer to val>
<bracket_left>
<bracket_right>

<brace_right>
```

Example decomposition of HTML:

```html
Here is a photo of <B>my house</B>:
<P><IMG SRC = "house.gif"><BR>
See <A HREF = "morePix.html">More Pictures</A> if you
liked that one.<P>
```

```
<literal "Here is a photo of">
<tag_B_begin>
<literal "my house">
<tag_B_end>
<literal ":">
<tag_P_begin>
<tag_IMG_begin>
<attr_SRC>
<equals_operator>
<literal "house.gif">
<tag_BR_begin>
<literal "See ">
<tag_A_begin>
<attr_HREF>
<equals_operator>
<literal "morePix.html">
<literal "More Pictures">
<tag_A_end>
<literal " if you liked that one.">
<tag_P_end>
```

## Syntax Analysis
