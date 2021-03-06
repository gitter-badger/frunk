# Frunk [![Crates.io](https://img.shields.io/crates/v/frunk.svg)](https://crates.io/crates/frunk) [![Build Status](https://travis-ci.org/lloydmeta/frunk.svg?branch=master)](https://travis-ci.org/lloydmeta/frunk)

> **frunk** *frəNGk*
>  * Functional programming toolbelt in Rust.
>  * Might seem funky at first, but you'll like it.
>  * Comes from: funktional (German) + Rust → Frunk

The general idea is to make things easier by providing FP tools in Rust to allow for stuff like this:

```rust
use frunk::monoid::*;

let v = vec![Some(1), Some(3)];
assert_eq!(combine_all(&v), Some(4));

// Slightly more magical
let t1 =       (1, 2.5f32,                String::from("hi"),  Some(3));
let t2 =       (1, 2.5f32,            String::from(" world"),     None);
let t3 =       (1, 2.5f32,         String::from(", goodbye"), Some(10));
let tuples = vec![t1, t2, t3];

let expected = (3, 7.5f32, String::from("hi world, goodbye"), Some(13));
assert_eq!(combine_all(&tuples), expected);
```

## Table of Contents

1. [Semigroup](#semigroup)
2. [Monoid](#monoid)
3. [HList](#hlist)
4. [Validated](#validated)
5. [Todo](#todo)
5. [Contributing](#contributing)
6. [Inspirations](#inspirations)

## Examples

### Semigroup

Things that can be combined.

```rust
use frunk::semigroup::*;

assert_eq!(Some(1).combine(&Some(2)), Some(3));

assert_eq!(All(3).combine(&All(5)), All(1)); // bit-wise && 
assert_eq!(All(true).combine(&All(false)), All(false));
```

### Monoid

Things that can be combined *and* have an empty/id value.

```rust
use frunk::monoid::*;

let t1 = (1, 2.5f32, String::from("hi"), Some(3));
let t2 = (1, 2.5f32, String::from(" world"), None);
let t3 = (1, 2.5f32, String::from(", goodbye"), Some(10));
let tuples = vec![t1, t2, t3];

let expected = (3, 7.5f32, String::from("hi world, goodbye"), Some(13));
assert_eq!(combine_all(&tuples), expected)

let product_nums = vec![Product(2), Product(3), Product(4)];
assert_eq!(combine_all(&product_nums), Product(24))
```

### HList

Statically typed heterogeneous lists. Pop as much as you want from one of these; everything
remains typed.

```rust
#[macro_use] extern crate frunk;
use frunk::hlist::*;

let h = hlist![true, "hello", Some(41)];
let (h1, tail1) = h.pop();
assert_eq!(h1, true);
assert_eq!(tail1, hlist!["hello", Some(41)]);

// HLists also have a .into_tuple2() method that convert HLists with 2 or more 
// items into nested Tuple2s for a nicer type-signature and pattern-matching experience
let hl = hlist!["Joe", "Blow", 30, true];
let (f_name, (l_name, (age, is_admin))) = hl.into_tuple2();
```

### Validated

`Validated` is a way of running a bunch of operations that can go wrong (for example,
functions returning `Result<T, E>`) and, in the case of one or more things going wrong, 
having all the errors returned to you all at once. In the case that everything went well, you get
an `HList` of all your results. 

Mapping (and otherwise working with plain) `Result`s is different because it will 
stop at the first error, which can be annoying in the very common case (outlined 
best by [the Cats project](http://typelevel.org/cats/tut/validated.html)). 

Here is an example of how it can be used.

```rust
#[derive(PartialEq, Eq, Debug)]
struct Person {
    age: i32,
    name: String,
    street: String,
}

fn get_name() -> Result<String, Error> { /* elided */ }
fn get_age() -> Result<i32, Error> { /* elided */ }
fn get_street() -> Result<String, Error> { /* elided */ }

// Build up a `Validated`
let validation = get_name().into_validated() + get_age() + get_street();
// When needed, turn the `Validated` back into a Result and map as usual
let try_person = validation.into_result()
                           .map(|hlist| {
                               let (name, (age, street)) = hlist.into_tuple2();
                               Person {
                                   name: name,
                                   age: age,
                                   street: street,
                               }
                           });

assert_eq!(try_person,
           Result::Ok(Person {
               name: "James".to_owned(),
               age: 32,
               street: "Main".to_owned(),
           }));
}

/// This next pair of functions always return Recover::Err 
fn get_name_faulty() -> Result<String, String> {
    Result::Err("crap name".to_owned())
}

fn get_age_faulty() -> Result<i32, String> {
    Result::Err("crap age".to_owned())
}

let validation2 = get_name_faulty().into_validated() + get_age_faulty();
let try_person2 = validation2.into_result()
                             .map(|_| unimplemented!());

// Notice that we have an accumulated list of errors!
assert_eq!(try_person2,
           Result::Err(vec!["crap name".to_owned(), "crap age".to_owned()]));
    
```

## Todo

### Stabilise interface, general cleanup

Before a 1.0 release, would be best to revisit the design of the interfaces
and do some general code (and test cleanup).

### Benchmarks

Benchmarks would be nice but they're an unstable feature, so perhaps in a different branch ?

### Not yet implemented 

Given that Rust has no support for Higher Kinded Types, I'm not sure if these
are even possible to implement. In addition, Rustaceans are used to calling `iter()` 
on collections to get a lazy view, manipulating their elements with `map`
or `and_then`, and then doing a `collect()` at the end to keep things
efficient. The usefulness of these following structures maybe limited in that context.

0. `Functor`
1. `Monad`
2. `Apply`
3. `Applicative`

## Contributing

Yes please ! 

The following are considered important, in keeping with the spirit of Rust and functional programming:

- Safety (type and memory)
- Efficiency
- Correctness

## Inspirations

Scalaz, Cats, Haskell, the usual suspects ;)