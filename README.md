<p align="center">
    <img height="230" src="jake.png">
    <img align="right" height="335" src="logo.png">
</p>

Welcome to the documentation for the Jake programming language.

## Introduction
If one were to think about it, Jake is what you would get if Rust, TypeScript, and Haskell had a beautiful baby. Or a bratty one maybe. This baby is highly opinionated, like his second cousin Brisk, though he does operate on a philosophy.

- *Be performant.* All constructs map very closely to WebAssembly, and abstractions are thin.
- *Be maintainable.* Jake will complain when you write code only God could possibly understand.
- *Be real.* Jake will not cater to non-WebAssembly programmers. Bid farewell to booleans, short-circuit evaluation, and traditional control flow semantics. May their bindles serve them well.

## Types

Jake has 5 categories of types.

### Void type

The void type is used to signify either an absence of a value, in the case where it's used in a return value, thus:

```ts
function main(): void {
    // returns nothing, function may have side effects
}
```

or an unspecified type, in the case where it's used in a pointer:
```ts
function main(x: *void) {
    let float = *<*f64>x;
    // or
    let *float = <*f64>x;
}
```

### Stack types

Stack types are the most performant types, and those that you will use in every Jake program you write.

```ts
i32 // a 32-bit signed integer.
u32 // a 32-bit unsigned integer.
f32 // a 32-bit float.
i64 // a 64-bit signed integer.
u64 // a 64-bit unsigned integer.
f64 // a 64-bit float.
isize // a 32 or 64-bit integer.
usize // a 32 or 64-bit integer.
```

> **Note**
> Since unsigned integers are abstractions, conversions of unsigned to signed integers and vice versa are cost-free. They do not require an instruction.

### Heap types

Heap types cannot be used in computation. Instead, they can be used in the following type categories.

```ts
i8 // an 8-bit signed integer.
u8 // an 8-bit unsigned integer.
i16 // a 16-bit signed integer.
u16 // a 16-bit unsigned integer.
```

### Prefix types

Prefix types imbue existing types with new properties. Some you've seen in other languages already.

```ts
&type // an aliased, immutable borrow.
&mut type // a mutable borrow.
&[type] // a fat reference.

// here be dragons 🐉
*type // an immutable raw pointer. 
*mut type // a mutable raw pointer.
*[type] // a fat pointer.
```

Fat pointers are pointers with associated length. That is, it keeps track of the length of the object it's pointing to. It is internally a product type of `[usize, *T or &T]`.

### Infix types

The infix types hold their roots in type theory and category theory.

#### Exponential types

Exponential types are equivalent to the notion of function types in other languages. Jake follows the notion of currying to describe functions with multiple parameters.

```ts
function foo(): u32;
// foo has type `void -> u32`

function bar(x: u32): void;
// bar has type `u32 -> void`

function baz(x: u32, y: i32): i64;
// baz has type u32 -> i32 -> i64
```

#### Product types

Product types are the equivalent of structs and tuples present in almost every language.

```ts
type Foo = [u32, u32];

function main([x, ..]: Foo) {
    // do something with x, the first field
}
```

to declare type destructors, use the `:` type operator.

```ts
type Product = [
    x: u32,
    y: u32
];

function main(): u32 {
    let foo: Product = [1, 5];
    return foo.x;
}
```

#### Packed types

Packed types are a special, optimized case of product types, where every field is a `bool`, `u8`, `u16`, or `u32`. Packed types are limited to a combined size that is less than 64 bits.

```ts
type Packed = packed [ myBoolean1: bool, myBoolean2: bool, myU8: u8, myU16: u16 ];

function main(): u32 {
    let foo: Packed = [false, false, 16, 257];
    foo.myU8 = 10;
    return foo.myU16; // loaded as a u32
}
```


#### Sum types

Sum types are the equivalent of algebraic enums in Rust, or the disjoint union in general. They are dual to product types, and so are their semantics.

```ts
type Sum = Foo | Bar; // Foo and Bar are type constructors

function main(Foo): *[u8] {
    return "you passed foo";
}

function main(Bar): *[u8] {
    return "you passed bar";
}
```

Sum types may also contain values.

```ts
type Option<T> = Some: T | None;

function main(x: Some<u32>): u32 {
    return x;
}

function main(None): u32 {
    return 0;
}
```

> **Note**
> Sum types without values are always defined internally as `usize`
> If they have values, then it's `[usize, *mut T]`.

#### Syntax ambiguity

A consequence with our current syntax so far is that the type
```ts
type MyProduct = [foo: u64 | u32, ..];
```
is ambiguous, since Jake doesn't know if `foo` is a type constructor or destructor. In this case, Jake will always interpret `foo` as a type destructor. If you intended the latter, abstract the sum type into a separate type declaration, thus:
```ts
type MySum = Foo: u64 | Bar: u32;
type MyProduct = [foo: MySum, ..];

// this is technically possible, but discouraged:

type MyProduct = [sum: Foo: u64 | Bar: u32, ..];
```

#### Union type 🐉

Union types are unsafe, in that a single memory area is designated several types at once.

```ts
type Unsafe = union [float: f64, int: i64];

function main(n: i64): f64 {
    let beware = union [ int: n ];
    return beware.float; // will reinterpret `n` from i64 to f64
}
```

## Functions

All functions (and consequently methods) are public to discourage code duplication.

### Methods

Functions may be used as methods.

```ts
function double(x: u32): u32 {
    return x * 2;
}

function main(): u32 {
    return 5.double();
}
```

### Getters

Getters are special functions that 

- Have one parameter.
- Must be referentially transparent (have no side effects). 
- *Should* be O(1).

Getters do not need parentheses and serve as a generalization of fields.

```ts
function double(x: u32): u32 {
    return x * 2;
}

function main(): u32 {
    return 5.double; 
}
```

### Exported functions

Exported functions are exported for use to the host. Only stack types, fat pointers, and regular pointers can be exported.

```ts
export function add(x: u32, y: u32): u32 {
    return x + y;
}
```

### Imported functions

Imported functions require the host to provide a function for use with Jake.

```ts
import function add(x: u32, y: u32): u32;

function main(): u32 {
    return add(1, 1);
}
```

### Generics

Generics allow functions to have the same implementation for multiple types.

```ts
function call<A, B>(f: A -> B, a: A): B {
    return f(a);
}

function double(x: u32) -> u32 {
    return x * 2;
}

function main() {
    return call(double, 5); // double(5) = 10
}
```

## Imports

Imports are similar to TypeScript.

```ts
import "./calculus.jk" with { derivative as derive, integrate }; // import these items
```
```ts
import "./graphs.jk" as Graph with { dominators, tarjan }; // import these items into this namespace
```
```ts
import "./algebra.jk" without { svd, rank }; // import everything but these items
```
```ts
import "./categories.jk" as Category; // import everything into this namespace
```

## Control flow

Much like WebAssembly, there are only three control flow constructs.

### If

`if` functions like any other language, excluding the borrowed `if let` from Rust. Pattern matching and function overloading are encouraged as a primary alternative to `if let`.

```ts
type Option<T> = Some: T | None;

function main(x: Option<u32>): u32 {
    if let num: Some = x {
        return num;
    } else {
        return 0;
    }
}
```
is the same as
```ts
function main(num: Option<u32>): u32 {
    return num;
}

function main(None): u32 {
    return 0;
}
```

### Loop

Loop is inverted, unlike other languages. That is, a `continue` keyword must be added to keep the loop going.

```ts
function factorial(n: 0): u32 {
    return 1;
}

function factorial(mut n: u32): u32 {
    let mut num: u32 = 0;
    loop {
        num *= n;
        n -= 1;
        continue if n > 0;
    }
    return num;
}
```

```ts
function fib(n: 0): u32 {
    return 0;
}

function fib(n: u32): u32 {
    let mut a = 0;
    let mut b = 1;

    loop {
        n -= 1;

        let t = a + b;
        a = b;
        b = t;

        continue if n > 0;
    }

    return b;
}
```

### Inline functions

Inline functions are always inlined into their call sites as WebAssembly blocks. Consequently, they allow for easy abstraction and flexibility within your code.

```ts
inline calculate(n: u32, d: 0): u32 {
    return None; // returns in the function above this one
}

inline calculate(n: u32, d: u32): u32 {
    break n / d; // returns the result to the caller
}

// main will return None
function main(): Option<u32> {
    calculate(5, 0);
}
```

## A note about booleans

Jake despises booleans. What a waste of space! Instead, you should use flags.

Jake does not support short-circuit evaluation. This means that the `&&` and `||` operators are not implemented.

```ts
function main() {
    return 1 & side_effects(); // will call side_effects!
}
```

### `~` vs `!`

Conventionally, `~` is the operator that inverts all the bits in an integer. WebAssembly does not offer this functionality with one instruction, so `~` is implemented as `x ^ -1`. However, `!` is the equivalent of `x == 0`, and is implemented with a single instruction: `eqz`.

## Lambda lifting

Lambda lifting is a technique to convert closures into performant functions. Although Jake doesn't support legitimate closures, he supports lambda lifting.

In short, lambda lifting "lifts" nested functions by converting the environment into parameters.

```ts
function main() {
    let x: u64 = 0;
    function add(n: u64) {
        x += n;
    }

    add(9);
    add(10);
}
```

Then Jake performs lambda lifting internally.

```ts
function add(x: &mut u64, n: u64) {
    *x += n;
}

function main() {
    let x: u64 = 0;

    add(&mut x, 9);
    add(&mut x, 10);
}
```

### What NOT to do

```ts
function instantiate(): void -> void {
    let x: u64 = 0; // unused variable
    function closure() {
        x += 1;
    }
    return closure; // ERROR! closure is of type `&mut u64 -> void`
}
```

## Memory management

Jake uses Rust's ownership and borrow checking scheme. Thus, he does not use a garbage collector and his ways can be mastered with some practice.

### Lifetimes

Jake doesn't like explicit lifetimes, so he simply infers all of them. This requires an intuition to understand certain errors caused by lifetime elision.

```ts
type Foo = [x: &u32, y: &u32];

function main() {
    let x = 0; // 'a

    function inner(): &u32 {
        let y = 1; // 'b
        let foo: Foo = [&x, &y]; // allowed, internal lifetimes are ['a, 'b]
        return foo.x;
    }

    assert(x == *inner());
}
```

```ts
function foo(x: &u32, y: &u32): &u32 { // x: 'a, y: 'a, return: 'a
    if random() {
        return x;
    } else {
        return y;
    }
}

function bar(x: &u32): &u32 {
    let y = 42; // 'b
    let borrow = foo(&x, &y); // the higher lifetime is coerced to the lower one (&x: 'a -> &x: 'b)
    return borrow; // error, cannot return a pointer whose lifetime will end
}

function main() {
    let x = 1; // 'a
    let z: &u32 = bar(&x);
}
```