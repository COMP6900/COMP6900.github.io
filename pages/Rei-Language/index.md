---
layout: default
title: Rei
has_children: true
---

Rei is a human-centric language for building systems and applications. As such it comes with a larger standard library, package management and building semantics, simplified and clean syntax without "junk".

## Programming Languages

Tbh I dont really know much about how languages are designed and built. I know a bit about parsers, semantic analysis, assembly and assemblers and some executable formats like ELF. But Idk about LL(1), LR(1), parse trees/abstract parse trees, EBNF, priority of regex and tokens.

So i'll just put some stuff here as I go. Will be mostly from first principles of the maths

## Rei prototype

- no semicolons, not even optional. Language designed around good spacing/indentation and new lines
- optional braces for one-line statements e.g. `if`, replaced with colon
- `let` and `const` variable declaration semantics
- `static` and `dynamic` lifetime modifiers
- high reliance on annotations for structuring, clealiness and metaprogramming
- supports async/await

```
@private
let x: Int = 10

class A {
    @construct // by default, new(...) methods are assigned @construct
    fn new() {
        this.__val = dynamic String("Class A")
    }

    fn new(val: const& String) {
        this.__val = val.clone() // copy constructor of String
    }

    @del // by default, del() method is assigned @del
    fn del() {
        delete this.__val
    }

    let __val: String
}

// In String
class String {
    ...
    __str: *u8
    size: UInt64

    @copy
    fn copy(copy_from: const& String) {
        this.size = copy_from.size
        // alloc should be used for heap. Used on the `dynamic` operator
        // this.__str = alloc(this.size * u8)
        // =operator performs shallow copy into memory addr (stack/heap/data)
        // and could be used instead of the above if not `dynamic`
        // this.__str = copy_from.__str

        // __str was not initialised. If was previously initialised, compiler would
        // call delete __str before this
        this.__str = [u8; this.size]

        for i in size.range() {
            // value copy without context
            _str[i] = copy_from.__str[i]
        }
    }
}
```
