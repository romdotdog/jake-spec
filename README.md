<p align="center">
    <img height="230" src="jake.png">
    <img align="right" height="335" src="logo.png">
</p>

Welcome to the documentation for the Jake programming language.

## Introduction
If one were to think about it, Jake is what you would get if Rust, TypeScript, and Haskell had a beautiful baby. Or a bratty one maybe. This baby is highly opinionated, like his sister [Brisk](https://github.com/spotandjake/Brisk), though he does operate on a philosophy.

- *Be performant.* All constructs map very closely to WebAssembly, and abstractions are thin.
- *Be rational.* Even though Jake may be an imperative language, it closely follows functional and theoretical principles according to [homotopy type theory](https://en.wikipedia.org/wiki/Homotopy_type_theory).
- *Be real.* Jake will not cater to non-WebAssembly programmers. Bid farewell to booleans, short-circuit evaluation, and traditional control flow semantics. May their bindles serve them well.

## Types

Jake has 5 categories of types.

### Void type

The [void type](https://en.wikipedia.org/wiki/Void_type) is a type with no size, functioning as the [unit type](https://en.wikipedia.org/wiki/Unit_type) and equivalent to the [top type](https://en.wikipedia.org/wiki/Top_type). It is an empty tuple, and can be constructed with `[]`. When functions don't have a return statement, a `return [];` is implicitly inserted.

```ts
function main(): void {
    // function may have side effects
    // return []; is implicitly added here, which is assignable to `void`
}
```

Variables may have `void` types.

```ts
function main(): void {
    let myVar: void = [];
    return myVar;
}
```

In the case where it's used inside a pointer, it is used to show that the pointee has no specified type.

```ts
function main(x: *void) {
    // we can unsafely cast the pointee to an arbitrary type 🐉
    let float = *<*f64>x;
}
```

### Never type

The `never` type is inherited from TypeScript and is equivalent to the [bottom type](https://en.wikipedia.org/wiki/Bottom_type). That is, it can be used to construct a value for any type, and thus a value for `never` can never be constructed. If it is, then either your program had a semantic error, is running forever elsewhere, or the world is ending (Jake had an oopsie please report!).

A function can be marked with `never` to signify that it loops forever.

```ts
function recurse(): never {
    return recurse();
}
```

### Stack types

Stack types are the most performant types, and those that you will use in every Jake program you write.

```rs
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

Heap types cannot be used in computation. Instead, they can be used in the type categories that follow.

```rs
i8 // an 8-bit signed integer.
u8 // an 8-bit unsigned integer.
i16 // a 16-bit signed integer.
u16 // a 16-bit unsigned integer.
```

### Prefix types

Prefix types imbue existing types with new properties. Some you've seen in other languages already.

```rs
&type // an aliased, immutable borrow.
&mut type // a mutable borrow.
&[type] // a fat reference. 
// this is also known as a "slice" in Rust.

// here be dragons 🐉
*type // an immutable raw pointer.
*mut type // a mutable raw pointer.
*[type] // a fat pointer.
```

Fat pointers are pointers with associated length. It can be likened to an array in that it keeps track of how many objects are at the pointer's destination. It is internally a product type of `[usize, *T or &T]`.

### Infix types

The infix types hold their roots in type theory and category theory.

#### Exponential types

[Exponential types](https://en.wikipedia.org/wiki/Function_type) are equivalent to the notion of function types in other languages. Jake follows the notion of currying to describe functions with multiple parameters.

```ts
function foo(): u32;
// foo has type `void -> u32`

function bar(x: u32): void;
// bar has type `u32 -> void`

function baz(x: u32, y: i32): i64;
// baz has type `u32 -> i32 -> i64`
```

Some additional ways to use functions follow.

##### Methods

Functions may be used as methods.

```ts
function double(x: u32): u32 {
    return x * 2;
}

function main(): u32 {
    return 5.double();
}
```

##### Pure functions

Pure functions, put simply, are functions that must return the same output, given the same input multiple times. Pure functions are referentially transparent, meaning that calls to pure functions may be replaced with the evaluation without side effects.

Pure functions are defined with the `pure` keyword. Pure functions offer the advantage of being compatible with Homotopy Type Theory (specifically, the transport lemma).

```ts
pure function add(x: u32, y: u32): u32 {
	return x + y;
}
```

##### Getters

Getters are special functions that 

- Have one parameter.
- *Must* be pure.
- *Should* be [O(1)](https://en.wikipedia.org/wiki/Constant_time).

Getters do not need parentheses and serve as more versatile fields.

```ts
pure function double(x: u32): u32 {
    return x * 2;
}

function main(): u32 {
    return 5.double; 
}
```

##### Host functions

Host functions are exported for use to the host. Only stack types, fat pointers, and regular pointers can be exported.

```ts
host function add(x: u32, y: u32): u32 {
    return x + y;
}
```

##### Imported functions

Imported functions require the host to provide a function for use with Jake.

```ts
import host add: u32 -> u32 -> u32;

function main(): u32 {
    return add(1, 1);
}
```

##### Generics

Generics allow functions to have the same implementation for multiple types. This feature relies on [dependent types](https://en.wikipedia.org/wiki/Dependent_type), which will come much later. 

A and B are `*`, which is the [type of types](https://en.wikipedia.org/wiki/Kind_(type_theory)). They belong in angle brackets because given `f`, the values of `A` and `B` can be inferred and do not need to be supplied. 

```ts
function call<A: *, B: *>(f: A -> B, a: A): B {
    return f(a);
}

function double(x: u32) -> u32 {
    return x * 2;
}

function main() {
    return call(u32, u32, double, 5); // double(5) = 10
    // note: virtual parameters (u32) will be inferred if they are omitted
}
```

Otherwise, if the type can't be inferred, the parameter must be placed within parentheses.

```ts
function sizeof(T: *): u32 {
	/* intrinsic */
}
```

#### Product types

[Product types](https://en.wikipedia.org/wiki/Product_type) are the equivalent of structs and tuples present in almost every language.

```ts
type Foo = [u32, u32];

function main([x, ..]: Foo) {
    // do something with x, the first field
}
```

to declare type destructors, a generalization of fields, use the `:` type operator.

```ts
type Product = [
    x: u32,
    y: u32
]; 

// x and y are now destructors
// the destructor x has type Product -> u32
// the destructor y has type Product -> u32

function main(): u32 {
    let foo: Product = [1, 5];
    return foo.x; // x is used as a getter
}
```

#### Packed types

Packed types, a generalization of [bit fields](https://en.wikipedia.org/wiki/Bit_field), are a special, optimized case of product types, where every field is a `bool`, `u8`, `u16`, or `u32`. All of these fields are fit into a single `u32` or `u64` and thus require a single local on the stack. Packed types are limited to a combined size that is less than 64 bits.

```ts
type Packed = packed [myBoolean1: bool, myBoolean2: bool, myU8: u8, myU16: u16];

function main(): u32 {
    let foo: Packed = [false, false, 16, 257];
    foo.myU8 = 10;
    return foo.myU16; // loaded as a u32
}
```

#### Sum types

[Sum types](https://en.wikipedia.org/wiki/Tagged_union) are the equivalent of algebraic enums in Rust, or the [disjoint union](https://en.wikipedia.org/wiki/Disjoint_union) in general. They represent a memory region to be one of a range of types.

```ts
type Foo = u32;
type Bar = &[u8];
type Sum = Foo | Bar; // Foo and Bar are now also constructors as well as types

// the constructor Foo has type Foo -> Sum
// the constructor Bar has type Bar -> Sum

function main(x: Foo): u8[] {
    return x.toString().utf8(); // TODO: owned borrows, relative pointers
}

function main(m: Bar): u8[] {
    return m;
}
```

Sum types may also contain values.

```ts
type Option(T: *) = Some: T | None: [];

function main(x: u32): u32 {
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

[Union types](https://en.wikipedia.org/wiki/Union_type) are unsafe, in that a single memory region is designated several types at once.

```ts
type Unsafe = union [float: f64, int: i64];

function main(n: i64): f64 {
    let beware = union [ int: n ];
    return beware.float; // will reinterpret `n` from i64 to f64
}
```

## Imports

Imports are similar to TypeScript.

```ts
import "./calculus.jk" with { derivative as diff, integrate }; // import these items
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

`if` functions like any other language. There is also the addition of flow-sensitive pattern matching:

```ts
type Option(T: *) = Some: T | None: [];

function main(num: Option(u32)): u32 {
    if num: Some {
        return num; // num is u32 here
    } else {
        return 0;
    }
}
```
For more complex functions, *definition by case analysis* is more ergonomic:
```ts
function main(num: u32): u32 {
    return num;
}

function main(None): u32 {
    return 0;
}
```

### Loop

`loop` is equivalent to `while(true)` in other languages. However, it is inverted, unlike other languages. That is, a `continue` keyword must be added to keep the loop going. If there is no `continue` keyword in a `loop`, the compiler will not hesitate to notify you.
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

#### Rationale behind the absence of a `for` loop

There is no `for` loop because the construct can be replaced by methods that take functions.
```ts
import function report(num: u32): void;

function range(start: u32, end: u32, callback: u32 -> void) {
    let n = start;
    loop {
        callback(n);
        continue if n < end;
    }
}

function main() {
    range(0, 10, report);
}
```

### Template functions

Template functions are always [inlined](https://en.wikipedia.org/wiki/Inline_expansion) into their call sites as WebAssembly blocks. Consequently, they allow for easy abstraction and flexibility within your code.

```ts
template calculate(n: u32, d: 0) {
    return None; // returns in the function above this one
}

template calculate(n: u32, d: u32) {
    break n / d; // returns the result to the caller
}

// main will return None
function main(): Option(u32) {
    calculate(5, 0);
}
```

## A note about booleans

Jake despises booleans. What a waste of space! Instead, you should use packed types.

Jake does not support short-circuit evaluation. This means that the `&&` and `||` operators are not implemented.

```ts
function main() {
    return 1 & side_effects(); // will call side_effects!
}
```

### `~` vs `!`

Conventionally, `~` is the operator that inverts all the bits in an integer. WebAssembly does not offer this functionality with one instruction, so `~` is implemented as `x ^ -1`. However, `!` is the equivalent of `x == 0`, and is implemented with a single instruction: `eqz`.

## The `liberal` option

Jake doesn't support indirect calling by default. That is, `liberal=0`.

Here is a sample of what happens no matter the setting of `liberal`:

```ts
function callWith5(f: u32 -> u32): u32 {
	return f(5);
}

function main() {
	return callWith5(add(1));
}
```

Since `callWith5` is inlinable, this becomes

```ts
function main() {
	return add(1)(5);
}
```

This removes the indirect call.

### Non-inlinable functions

```ts
function fib(n: u32, adjust: u32 -> u32): u32 {
	if (n == 0 || n == 1) {
 		return n;
	} else {
		return adjust(fib(n - 1, adjust) + fib(n - 2, adjust));
	}
} 

function main() {
	return fib(5, someFunction);
}
```

if `liberal=0`, this code would fatally error because recursive functions are non-inlinable and thus `adjust` cannot be passed directly. If `liberal=1`, `adjust` would be registered into the WebAssembly table and passed into `fib` appropriately.

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

## Lambda lifting

[Lambda lifting](https://en.wikipedia.org/wiki/Lambda_lifting) is a technique to convert some closures into performant top-level functions. Although Jake doesn't support legitimate closures, he supports lambda lifting.

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
