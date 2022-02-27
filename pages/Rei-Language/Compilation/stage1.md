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

# Lexical Analysis

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

## Error Recovery

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

### Input Buffering

We should buffer the inputs to know for sure the most encapsulated lexeme. The most prioritised token should be mapped to a certain lexeme

- we cannot be sure a sequence is an `identifier` until we see the end of the character that does not belong to the `identifier` pattern. So space, operator, etc
- we cannot be sure a single character operator like `=` doesnt actually mean `==` when we look at the next character. Also non existent operators like `===` which should bring about an error or `unidentified` tag

### Buffer Pairs

We start with two lexeme buffers that are reloaded one after the other. Each buffer is of size `N`, e,g, a page size like 4KiB. So we can read a disk block into RAM at a time. Two virtual pages are mapped to these buffers which the analyzer can use to `read()`

- we know that a page contains the end of the source file if it contains `eof` or is less than size `N`

We then keep 2 pointers, `lexeme_begin`, `forward`. These point to the beginning of the current lexeme and a scanner that keeps moving forward until a pattern match is found

So when the next lexeme is determined, `forward` is set to the character at its right end. We record that lexeme as an attribute value of a token returned by the parser. And set `lexeme_begin` to the character right after `forward`

- we will never overwrite the lexeme in its buffer before determining it. As long as we dont look ahead of the current lexeme so that L + D > N

### Sentinel

The sentinel character is a special char that cannot be part of the source program. We usually use `eof`. We combine the buffer-end test with the test for the current character by making each buffer hold `eof` at the end. If we moved off the current buffer, we must load the next buffer

### Specification of Tokens

Lexemes should be short like in most languages. Usually one-two character lookahead is good enough

- so a buffer size N = 4096 is ample, though some problems like character string literals may extend over many lines. This means the lexeme can be longer than N
- to solve this, we can treat them as a concatenation of each line. We can simply use something like the `+` operator before each line end to concat the next line's string, and do so in the lexer program
- but when arbitrarily long lookaheads are needed, you can treat keywords like `fn` as identifiers rather than actual keywords. Then let the parser resolve the full meaning in conjunction with a namespace level symbol table

Lexical analysers are basically regex engines that convert regular expressions into algorithms that perform token recognition

Let us look at an alphabet, which is a finite set of symbols. ASCII is an important set that we can use to build a programming language. Now consider a lookahead code:

```
switch forward {
    case eof:
        if foward.index == buffer_1[N-1]:
            reload buffer_2 // call read() on the next sequence
            forward = buffer_2[0]
        elif foward.index == buffer_2[N-1]:
            reload buffer_1
            forward = buffer_1[0]
        else:
            exit
    case 'A'
    ... // other chars
}
```

- multiway branches like the switch statement can be compiled in a single shot by simply storing each case's address in a lookup table and calculating a perfect hash function (randomised) for it

$\epsilon$ -> empty string \
$s$ -> non-empty string \
$|s|$ -> length of $s$

So a `language` is any countable set of strings `s` over some fixed alphabet `A`.

### Parts of a String

`prefix` -> any string obtained by removing $\geq 0$ symbols from the end of `s` \
`suffix` -> from the start \
`substring` -> deleting any prefix and any suffix from `s` \
`proper` prefix or suffix -> results that are not $\epsilon$ or `s` itself \
`subsequence` -> delete $\geq 0$ symbols that are not necessarily consecutive

To concatenate strings, we represent them like `ss`

### Operations on Languages

Note position does matter in a string, so we talk about permutations on strings.

A bunch of set theory like unions and closures. Know that $L^4$ means set of all 4-letter strings in language $L$.

L U D means set of letters and digits. 62 in the English Language.

L* means set of all strings of letters. So if you can imagine an infinite set of strings that contain all the possible letter permutations.

L(L U D)*-> (L U D)* means the set of strings containing all possible letter and digit permutations. L means set of letters. So the whole thing means strings with a letter in the first position and any letter+digit permutation afterwards.

D+ is the set of all strings with $\geq 1$ digit

## Regex

Each regular expression $r$ represents a language $L(r)$. $r$ can then be defined recursively by $r$'s subexpressions

Most languages have identifiers that begin with a letter can contain letters, numbers and underscores:

r"[\l_][\l_\d]"

$\epsilon$ is a regular expression of "" and $L(\epsilon) = \{\epsilon\}$ \
if $a$ is a symbol in `ALPH` then $a$ is a regular expression. This also means $L(a) = \{a\}$ -> same idea as $\epsilon$

- $(r)|(s)$ is a regex denoting the language $L(r) \cup L(s)$
- $(r)(s)$ denotes $L(r)L(s)$ concatenated alternatives
- $(r)*$ denotes $L(r)^*$
- $(r)$ denotes $L(r)$ -> basically we can add additional pairs of brackets without changing the language it represents

### Order of precendence

\* > concat > |

### Extensions of Regex

- `+` -> one or more instances of the previous group. Called the 'positive' closure `(r)+` denoting `L(r)+`
- `?` -> zero or more instances
- $a_i$  -> character classes where a_1 | a_2 ... a_n can be replaced with `[a_1a_2...a_n]`. So `[abc]` means `a | b | c`

### Regular Definitions

We can give names to certain regular expressions and use their names in subsequent expressions. We can also define the set of values which the regex maps to

Examples:

`letter -> A | B | C ... | Z | a | b | ... | z |`\
`digit -> 0 | 1 | ... | 9`\
`underscore -> _`\
`identifier -> (letter|underscore)(letter|underscore|digit)*`

Example of optionals:

`optional_fraction -> .digit+|eps`\
`optional_exp -> (E(+|-|eps)digit+)|eps`\
`number -> digit+ optional_fraction optional_exponent`

Note the spaces shouldnt important in the this case. `1.1` should be the same as `1 . 1` and `1E+10` is the same as `1 E + 10`. But they are, so only the no space ones are taken, i.e. `1.1` and `1E+10` for simplicity sake

## How to recognise Tokens



## Syntax Analysis
