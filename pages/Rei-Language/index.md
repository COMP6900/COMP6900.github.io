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
- `let` and `const` variable declaration semantics. By default all `let`s are mutable. If you dont want mutability, then use `const`
- `static` and `dynamic` lifetime modifiers
- high reliance on annotations for structuring, clealiness and metaprogramming
- supports async/await
- every field (class, function, variable, constant) in a namespace is `private` by default, use `@export` or `@export-default` per object

```
let x: Int = 10

@export-default, template(T: B)
class A {
    construct() {
        self.__val = dynamic String("Class A")
    }

    construct(val: &const String) {
        self.__val = val.clone() // copy constructor of String
    }

    @ignore-warnings // suppress no-explicit-initialisation warning
    construct(val: Int) {
        // Warning! construct method does not explicitly initialise all fields: __val
        // will use default constructor
    }

    del() {
        delete self.__val
    }

    operator+(add_from: &Self) -> Self {
        return Self(&add_from.__val)
    }

    fn get_val() -> &String {
        return self.__val
    }

    let __val: String
    @skip // tell compiler not to include this field
    let b: B
}

// local-scope bound allocated
// when a goes out of scope, `delete a` is called (del() function is called)
let a = A()
// same as let a = new A. `dynamic` is a keyword in rei to mean heap allocated
// when the number of references to a = 0, a is deallocated (del() function is called)
let a = dynamic A()
// same as
@dynamic
let a = A()

fn accept_a(_a: A) => println!("ayy!") 

// to pass a as a reference, make sure to pass &a
// for functions that accept references/values
// otherwise will always pass by copy on the stack
// not like C++ where it automatically casts it into a reference for you

// pass by reference
accept_a(&a)

// pass by value
accept_a(a)

fn accept_a_ref(_a: &A) => println!("references only!")

// create a temporary value a, and pass that by reference
// doesnt actually affect this.a
accept_a_ref(a)

// pass by reference
accept_a_ref(&a)

// MORAL OF THE STORY: look at IDE annotations and autoparameter values closely
// to see how your passing
// always create a value, not a reference. Always define a parameter as a value, not a reference
// then just write &a once when you are calling the function
// NOTE: references are actually a type annotation. A variable with type T is just a variable
// A variable with type T and annotation Reference is a reference/symlink to that variable

// In String
class String {
    ...
    __str: *u8
    size: UInt64

    copy(copy_from: const& String) {
        this.size = copy_from.size
        this.__str = [u8; this.size]

        for (i in size.range()) {
            // value copy without context
            _str[i] = copy_from.__str[i]
        }
    }
}

// Defining an annotation
annotation example {
    // define what object types this applies to, e.g. class, variable, function

    // when used on a function, can hook onto functional patterns
    function => {
        // do something to the function, e.g. check the inputs and outputs before or after calling
        // with arrkia: can hook onto arrkia render cycles
    },

    // when used on a class, can hook onto creation, copy, destruction, method calls
    class => {
        on create:
            {
                // check constructor inputs
            }
    }

    // when used on a variable, can hook onto creation, destruction, updates
    var => {
        on update:
            {
                // check new value
                // e.g. @range(0, 100)
                // maybe if it is not in a certain range, raise an exception
            }
    }
}

// Annotation with arguments
// Highly recommended to make all labels meaningful, esp the params
annotation range(begin, end) {
    var<Numeric> => {
        on update:
            {
                if (self < begin || self > end) throw OUT_OF_RANGE
            }
    }
}

// Defining a macro
// Note should be in upper camel case
macro DoThis {
    () => {
        println("I did something")
    },
    (x: Numeric, y: Numeric) => {
        x + y
    }
    (x: String) => {
        println("String: $x")
    }
}
```
