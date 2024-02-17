# Preliminaries (not part of Rust By Example)

## Meta

This was written after someone came to URLO with a few questions they
had while working through the [Rust By Example](https://doc.rust-lang.org/rust-by-example/index.html)
book.

When I started out, I intended it to be a "CliffsNotes" for the book,
adding more detail or clarifications.  However, as I went along, I
found outdated and incorrect sections (and also burnt out a bit).  So
parts of these notes are phrased in more of a "things that should be
corrected" manner.

I also skimmed more the further I went (it's not a short book).

My conclusion is that RBE needs some care and attention to bring it
up-to-date and correct some inaccuracies.  Many sections have not
been touched in years despite the parts of the language they cover
evolving.  There's at least one section that talks about types which
were renamed before Rust 1.0!

So I'm not sure what to do with these notes long term.  A lot of them
could and should go away as RBE gets updated.  Others may go into more
depth than RBE needs or cares to.

So don't count on these notes (or their URL anchors) being permanent.

## RBE's Code Style
The code examples in RBE have a quirky, non-idiomatic style.  In particular,
I've never seen anyone format their `where`-clause-using functions the way
that RBE does:
```rust
fn print_ref<'a, T>(t: &'a T) where
    T: Debug + 'a {
    println!("`print_ref`: t is {:?}", t);
}
```
Something more idiomatic would be:
```rust
fn print_ref<'a, T>(t: &'a T)
where
    T: Debug + 'a,
{
    println!("`print_ref`: t is {:?}", t);
}
```

If in doubt, run `rustfmt`.  I don't agree with everything `rustfmt` does
(or refuses to do), but it will at least be idiomatic, unlike many
examples in RBE.

## OK let's get started

The rest of these notes follow RBE's structure (at the time of writing).

# [Hello World](https://doc.rust-lang.org/rust-by-example/hello.html)

Almost no-one invokes `rustc` directly.  [A future section](https://doc.rust-lang.org/rust-by-example/cargo.html)
introduces `cargo`; you should use `cargo` instead of `rustc` directly.

If you do use `rustc` directly, you should specify the latest stable edition
in order to get access to more modern language features and to help ensure
your code is more idiomatic.

As I am writing this, that would be: `rustc --edition=2021`.

But you can also see which editions are available for yourself:
```bash
rustc --edition=
# error: argument for `--edition` must be one of: 2015|2018|2021|2024. (instead was ``)

rustc --edition=2024
# error: the crate requires edition 2024, but the latest edition supported by this Rust version is 2021

rustc --edition=2021
# error: no input filename given

echo yay
```

## [Comments](https://doc.rust-lang.org/rust-by-example/hello/comment.html)

There are also block doc comments, starting with `/**` or `/*!`.
```rust
/** ...outer block comment like /// */
/*! ...inner block comment like //! */
```

(But they are less commonly used.)

## [Formatted print](https://doc.rust-lang.org/rust-by-example/hello/print.html)

This is at the bottom of the long example and thus easy to miss:
```rust
    // For Rust 1.58 and above, you can directly capture the argument from a
    // surrounding variable. Just like the above, this will output
    // "    1", 4 white spaces and a "1".
    let number: f64 = 1.0;
    let width: usize = 5;
    println!("{number:>width$}");
```
(Rust 1.58 was released on January 13th, 2022.)

This style is now more common than either positional or named parameters.
Without format specifiers, it is even simpler than their example.
```rust
    println!("{number}");
```

# Primitives

## [Arrays and slices](https://doc.rust-lang.org/rust-by-example/primitives/array.html)

It's not uncommon for even official documentation to be sloppy with terminology around
slices, and Rust By Example is no exception.

Let us be a bit more formal and say that:
- `[T]` is a slice
- `&[T]` is a shared slice
- And there are other variations like a boxed slice (`Box<[T]>`)

Then a slice is the type of the contiguous collection of `T`s of some length not
known at compile time.  Because the length is not known, the *size* of the type
is also not known -- it is a dynamic property, depending on how many elements
are in the slice.  This also means that the following bound:
```rust
[T]: Sized
```
can never hold.  This is also called being "unsized" or being a "dynamically
sized type (DST)".

Incidentally, when you declare a generic parameter `T`, that parameter has an
implicit `T: Sized` bound.  You can remove it with `T: ?Sized`.

A *shared* slice (`&[T]`) is the type which consists of two `usize`-sized values,
a pointer to the data and a count of the elements.

This two-value reference is also called a "wide reference" or "wide pointer"
or "fat pointer", as it's twice the size of "normal" ("thin") reference to
a `Sized` value.

The order between the length and the pointer is not specified, ignore what RBE
says about "first" and "second" words.  You almost surely won't need to destructure
a shared slice until you're much further in your Rust journey, if ever, but if you
do -- find out how to do so without `unsafe`; transmuting in particular is incorrect.
(*Constructing* a wide reference often requires `unsafe`, but `transmute` is still not the answer.)

`&[T]: Sized` because its size is a compile-time property -- the two values
we just described.  `Box<[T]>` and the other variations are also generally
also `Sized` and consist of a wide pointer.

It is common to call all of `[T]`, `&[T]`, `&mut [T]`, etc "slices", so don't count
on anything you read being careful about distinguishing the unsized type and
the (`Sized`) references or pointers to the slice.

There's a diagram of a `[T]` and a `&[T]` (and a `Vec<T>` and `&Vec<T>`) further down
these notes.

# [Custom Types](https://doc.rust-lang.org/rust-by-example/custom_types.html)

This is sloppy wording, as `const`s and `static`s are distinct things:
> Constants can also be created via the const and static keywords.

More about that in the specific section.

## [Structures](https://doc.rust-lang.org/rust-by-example/custom_types/structs.html)

Structs with named fields are sometimes called "nominal structs".  I don't think
it's common to call them "C structs".  In fact, Rust does have features for
creating data structures compatible with C-based FFI, but it requires specific
annotations; referring to your typical named-field struct as a "C struct" is
misleading.

## Enums

### [`use`](https://doc.rust-lang.org/rust-by-example/custom_types/enum/enum_use.html)

If you import the variant names, take care not to make any typos or you'll
introduce a binding instead.

[Example thread about the hazard.](https://users.rust-lang.org/t/why-does-match-allow-arbitrary-identifiers/106476)

You'll generally get a warning or three about it.

I don't think the guide mentions this incidentally: Rust warnings are generally
pretty good.  Don't just ignore them like you might be used to from some other
languages.

Full warnings and errors in the console (e.g. `cargo check`) are generally better
than the abbreviated warnings and errors you might get in your IDE.

## [constants](https://doc.rust-lang.org/rust-by-example/custom_types/constants.html)

`static`s are particular values that persist for the entire run time of your program.
They are not duplicated.

`const`s are evaluated where you use them, as if you had pasted the definition in
place.  If the `const` happens to create a static value, for instance, the compiler
is free to create multiple static values.  [See this issue](https://github.com/rust-lang/rust/issues/72004)
for an example.

`static mut` is so wildly hard to use correctly that the experts get it wrong.
[It will probably be deprecated](https://github.com/rust-lang/rust/issues/53639)
as better alternatives become available in the language and standard library,
for example:
- https://doc.rust-lang.org/core/cell/struct.OnceCell.html
- https://doc.rust-lang.org/std/sync/struct.OnceLock.html
- https://doc.rust-lang.org/core/cell/struct.LazyCell.html (unstable)
- https://doc.rust-lang.org/std/sync/struct.LazyLock.html (unstable)

Don't use `static mut`.

Using a `static` when you want or need to have only one globally shared value,
not because you want a "mutable constant".

# Variable Bindings
## [Mutability](https://doc.rust-lang.org/rust-by-example/variable_bindings/mut.html)

Without a `mut` binding, you can't
- Create a `&mut _` to the variable (including implicitly, e.g. when calling methods)
- Overwrite the variable (`variable = new_value`)

But note that this is, roughly speaking, a compiler-enforced lint and not some
additional intrisic property of the value in question; the type of the variable
is the same with or without `mut`, for example.  And you can still do things
like this:
```rust
let supposedly_immutable = String::new();
let mut tmp = supposedly_immutable;
tmp.push_str("Gottem");
let supposedly_immutable = tmp;
```
(It also doesn't prevent destructors from running.)

In contrast, `&mut T` and `&T` are distinct types with distinct capabilities
and other behavior.  Despite the spelling, you should think of `&mut T` as an
*exclusive* reference and `&T` as a *shared* reference.  Why?  Well, Rust has
"interior mutability", a.k.a. "shared mutability", which allows mutation
through a `&T` in some cases.  By thinking of references as "exclusive vs.
shared" from the start, you'll have less to unlearn and relearn once you
encounter shared mutability types.  (There's an example later in these notes.)

Additionally, ensuring exclusivity is actually how `&mut T` works/is defined.

## [Declare first](https://doc.rust-lang.org/rust-by-example/variable_bindings/declare.html)

The wording here is pretty poor...
> However, this form is seldom used, as it may lead to the use of uninitialized variables.
> [...]
> The compiler forbids use of uninitialized variables, as this would lead to undefined behavior.

*Using* unitialized variables can't be the reason it's seldom used, since that's not something
that even compiles.  I would say it's more that *having potentially unitialized bindings* is
a rare need.

So how about an example where it's actually useful?
```rust
use std::sync::{Arc, Mutex};
fn some_condition() -> bool { true }

fn example(mtx: Arc<Mutex<Vec<String>>>, other_data: &mut [String]) {
    let mut lock;
    let data = if some_condition() {
        lock = mtx.lock().unwrap();
        &mut **lock
    } else {
        other_data
    };
    
    // ... do work on the data ...
}
```
Here, we may or may not need to lock the mutex to operate on some data.
We don't want to lock the mutex if we don't have to.  But if we do lock
the mutex, the mutex guard (`lock`) needs to stick around until the end
of the function.  So this won't compile because the lock goes out of scope:
```rust
    let data = if some_condition() {
        let mut lock = mtx.lock().unwrap();
        &mut **lock
    } else {
        other_data
    };
```
Instead we declare outside the `if` and initialize only where we have to.
The compiler will compile things in such a way that the lock gets dropped
at the end of the function if and only if it was initialized.

## [Freezing](https://doc.rust-lang.org/rust-by-example/variable_bindings/freeze.html)

This is a weird example.  For one, it can only work with types that implement
`Copy` as it is currently written.  For another, you can just as easily
"unfreeze" a variable by moving it into a `mut` binding.  For a third, this
doesn't really prevent mutation in all cases, namely, when you have indirection
via something like a reference.
```rust
fn example(v: &mut String) {
    {
        // Haha, no `mut`!
        let v = v;
        
        // Oh yeah, we don't need `mut v: &mut` in order to mutate *through* `v`
        v.push_str("...");
    }
}
```

More or less this just goes back to `mut` bindings being like a lint that
prevents you from accidentally overwriting a variable or taking a `&mut`
to it.  Moving something to a non-`mut` binding turns on the lint, moving
something to a `mut` binding turns off the lint.

You *can* use variable shadowing without moving the original variable to
make it inaccessible, but this won't kill any outstanding borrows on its
own...
```rust
fn example() {
    let mut s = String::new();
    let r = &mut s;
    
    // Haha, no `mut`!
    let s = ();

    // ...drat
    r.push_str("...");
}
```
...so you need something more convoluted if that's your goal.
```rust
fn example() {
    let mut s = String::new();
    let r = &mut s;
    
    // Haha, no `mut`!
    // *And*, taking the `&mut s` ensures there are no outstanding
    // borrows at this point.  Moving would be another way.
    let s = { let _tmp = &mut s; () };

    // This is now a borrow check error
    r.push_str("...");
}
```

Actually using these sorts of code patterns is rare.

The "unfreezing" pattern is discussed in the section on ownership for some reason.
The two sections could be combined and rewritten in a way that just highlights
that you can rebind things to add or remove the `mut` qualifier.  And to be
more clear about what omitting `mut` actually prevents (`&mut` and direct overwites).

# [Types](https://doc.rust-lang.org/rust-by-example/types.html)

You can also type-erase values to `dyn Trait`, which is covered
in [a future section.](https://doc.rust-lang.org/rust-by-example/trait/dyn.html)

## [Aliasing](https://doc.rust-lang.org/rust-by-example/types/alias.html)

There is also a form of opaque aliasing related to
[`impl Trait`](https://doc.rust-lang.org/rust-by-example/trait/impl_trait.html)
that I'll write about in that section.

# [Conversion](https://doc.rust-lang.org/rust-by-example/conversion.html)

Other conversion traits include
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html) and [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)
- [`Borrow`](https://doc.rust-lang.org/std/borrow/trait.Borrow.html) and [`BorrowMut`](https://doc.rust-lang.org/std/borrow/trait.BorrowMut.html)

One thing this section doesn't currently point out is that the blanket
implementations on these traits limit or prevent you from writing
blanket implementations for your own types, due to the possibility
of overlapping implementations.

So this doesn't work, for example.
```rust
struct Mine(String);
impl<T: From<String>> From<T> for Mine { /* ... */ }
```

# Flow control
## [`if`/`else`](https://doc.rust-lang.org/rust-by-example/flow_control/if_else.html)

More weird formatting here; you'd just have the `if` on the same line as the `let big_n =` typically.

## [`for` loops](https://doc.rust-lang.org/rust-by-example/flow_control/for.html)
### [`for` and iterators](https://doc.rust-lang.org/rust-by-example/flow_control/for.html#for-and-iterators)

`for` works with the [`IntoIterator` trait,](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html)
not the `Iterator` trait.  It's true that every `I: Iterator` implements `IntoIterator` by returning itself,
but `IntoIterator::into_iter` is actual the functionality that `for` uses.
[More on that here.](https://doc.rust-lang.org/std/iter/index.html#for-loops-and-intoiterator)

Not every `I: IntoIterator` implements `Iterator`.

RBE then says:
> `into_iter`, `iter` and `iter_mut` all handle the conversion of a collection into an iterator in different ways, by providing different views on the data within.


`iter` and `iter_mut` are methods on individual collection types, not trait
methods. (Some collections may *also* define an `into_iter` method on the collection
type that does the same thing as their `IntoIterator` implementation.)
Their `iter` and `iter_mut` names are only by convention.  Some types have other methods too,
like [`values_mut`.](https://doc.rust-lang.org/std/collections/hash_map/struct.HashMap.html#method.values_mut)

By convention,
- `collection.iter()` will do the same thing as `impl IntoIterator for &Collection`
- `collection.iter_mut()` will do the same thing as `impl IntoIterator for &mut Collection`

Which means you can write the examples like so.
```rust
fn main() {
    let names = vec!["Bob", "Frank", "Ferris"];

    // for name in names.iter() {
    for name in &names {
        // ...
    }
}
```
```rust
fn main() {
    let mut names = vec!["Bob", "Frank", "Ferris"];

    // for name in names.iter_mut() {
    for name in &mut names {
        // ...
    }
}
```

These are conventions, not requirements.

## [`match`](https://doc.rust-lang.org/rust-by-example/flow_control/match.html)
### [Destructuring](https://doc.rust-lang.org/rust-by-example/flow_control/match/destructuring.html)

I recommend [reading this article.](https://h2co3.github.io/pattern/)  It explains a lot about
how patterns work in the absense of binding modes (more about that in a moment).  The main things
it doesn't cover are `@`, `ref`, and `ref mut`.

I'll assume after this you've read it.

#### [pointers/`ref`](https://doc.rust-lang.org/rust-by-example/flow_control/match/destructuring/destructure_pointers.html)

This page explains `ref`/`ref mut` adequately well.

So -- binding modes (a.k.a. "`match` ergonomics").  What's that?  It's something that was introduced
to the language so that you don't have to destructure references and use `ref` and `ref mut` so
much.

The basic idea is that if you match on a reference without destructuring the reference, the
"default binding mode" changes from by-value to `ref` or `ref mut`.  If you later do explicitly
destructure a reference, the default binding mode changes back to by value.

Binding modes can be pretty confusing when the patterns are complicated, but they're pretty
intuitive at a basic level once you get used to them.  Let's see an example:
```rust
fn example(opt: &Option<String>) {
    // Without binding modes (1):
    match *opt {
        None => {}
        Some(ref s) => {
            println!("{s}")
        }
    }

    // Without binding modes (2):
    match opt {
        &None => {}
        &Some(ref s) => {
            println!("{s}")
        }
    }

    // With binding modes:
    match opt {
        None => {}
        Some(s) => {
            println!("{s}")
        }
    }
}
```
These all do the same thing.  In the last `match`, using the pattern `Some(...)` against
the value which has type `&Option<_>` results in a `ref` binding mode.  So when you
introduce the `s` binding, it binds like `ref s` does in the first two `match`es.

Things get less obvious when the pattern moves in and out of different `ref` modes,
or when there are otherwise a lot of different binding modes depending on where you
are in the pattern.  If you run into such a situation and want to "disable" binding
modes so that the [duality of patterns](https://h2co3.github.io/pattern/) is restored,
making the pattern more explicit, you can set the
[`pattern_type_mismatch`](https://rust-lang.github.io/rust-clippy/master/index.html#/pattern_type_mismatch)
Clippy lint to `deny`.

[Example.](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=59801ce06e83a36b196471eb136116ae)
(You can run Clippy under Tools, top-right.)

# Functions
## Closures
### [As input parameters](https://doc.rust-lang.org/rust-by-example/fn/closures/input_parameters.html)

This doesn't really cover the heirarchical nature of the `Fn` traits.
- Everything that implements `Fn` implements `FnMut`
- Everything that implements `FnMut` implements `FnOnce`

Or when you want to use which.
- `FnOnce` is the easiest bound to meet; use it if you only need to call the closure once
- `FnMut` is the next easiest, and usually not too restrictive for the callee; prefer this if you need to call the closure more than once
- `Fn` is the most restrictive and is only rarely needed by the callee

> On a variable-by-variable basis, the compiler will capture variables in the least restrictive manner possible.

When defined in a place annotated with a `Fn`-trait bound -- like when passing a closure into a method
with such a bound -- the bound will influence how the compiler decides to capture variables and which
`Fn` traits get implemented.  The compiler's closure inference isn't perfect, and such annotations are
one of the main ways of controlling the inference.

### [Type annonymity](https://doc.rust-lang.org/rust-by-example/fn/closures/anonymity.html)

The code style here is still weird.

> meanwhile implementing the functionality via one of the traits: `Fn`, `FnMut`, or `FnOnce`

More than one, if required by the heirarchy mentioned above.

> However, an unbounded type parameter <T> would still be ambiguous and not be allowed.

It's not ambiguous, it's the fact that you can't use capabilities of generic types
which aren't specified in the bounds.

### [Input Functions](https://doc.rust-lang.org/rust-by-example/fn/closures/input_functions.html)

> As an additional note, the `Fn`, `FnMut`, and `FnOnce` traits dictate how a closure captures variables from the enclosing scope.

What they mean is that the *bound* will influence how the compiler decides to capture variables and which
`Fn` traits are implemented, as mentioned above.

Other related notes...
- Like closures, functions ("function items") each have their own unnameable type
- But there are also function pointer types like `fn()` or `fn(&str) -> String`, etc.
- Function items can be coerced to function pointers
- Non-capturing closures can also be coerced to function pointers

### [As output parameters](https://doc.rust-lang.org/rust-by-example/fn/closures/output_parameters.html)

> so we have to use impl Trait to return them

No, you don't.  You can return `Box<dyn Fn()>` or the like too.

> Beyond this, the `move` keyword must be used, which signals that all captures occur by value. 

No, it's not always needed.  Not all closures capture, for one...
```rust
fn create_fn() -> impl Fn() {
    || println!("This is a: {}", "Fn")
}

fn create_fnmut() -> impl FnMut() {
    || println!("This is a: {}", "FnMut")
}

fn create_fnonce() -> impl FnOnce() {
    || println!("This is a: {}", "FnOnce")
}
```
...others don't need additional annotation to capture by value, for another...
```rust
fn create_fnonce() -> impl FnOnce() -> String {
    let s = "FnOnce".to_string();
    || s
}
```
...and closures that don't capture by value can also be returned:
```rust
fn create_fn(s: &str) -> impl Fn() + '_ {
    || println!("This is a: {}", &*s)
}
```
N.b. this last example relies on the edition 2021+ closure capture
semantics, which illustrates why you should always use the latest
edition / use cargo.

### [Higher Order Functions](https://doc.rust-lang.org/rust-by-example/fn/hof.html)

> These are functions that take one or more functions and/or produce a more useful function.

Uh, nothing in the example produces a function.  They do take closures.  There are
other combinators that don't take closures, too.

"Higher order function" isn't terminology I see being used in the community.

Probably this section just needs some wording adjustment.

### [Diverging functions](https://doc.rust-lang.org/rust-by-example/fn/diverging.html)

Another way to describe `!` is "uninhabited type".

Enums with no variants are also uninhabited, but don't have `!`'s magical coercive properties.

The property is useful for things like
```rust
let thing: Thing = match result {
    Ok(thing) => thing,
    Err(e) => {
        do_other_stuff(e);
        return
    }
}
```
Because both branches have to have the same type (`Thing`); the `Err` branch
achieves this by coercing `!` to `Thing`.

This page includes an example that uses a `#![feature(...)]`.  You can only
use features on nightly (which this guide hasn't introduced).

Diverging functions never return, but they might unwind due to non-aborting panic.
I.e. they don't have to loop forever or stop thread execution.  (Unwinds can be caught.)

# Modules
## [Struct visibility](https://doc.rust-lang.org/rust-by-example/mod/struct_visibility.html)

The fields of tuple structs are also private by default, but you can choose visibility.
```rust
pub struct Tup(i32, pub String);
```
The variants of `enum`s all have the same visibility as the `enum` itself.

The section should mention [`non_exhaustive`.](https://doc.rust-lang.org/reference/attributes/type_system.html#the-non_exhaustive-attribute)

You can't destructure or use [struct update syntax](https://doc.rust-lang.org/reference/expressions/struct-expr.html#functional-update-syntax)
with types that have fields you can't name or foreign types that are `non_exhaustive`.

Hmm, do they cover SUS anywhere?  I don't think they do.  The general idea is that here:
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn example(user1: User) {
    // This part is the struct update syntax
    let user2 = User {
        email: String::from("another@example.com"),
        active: true,
        ..user1
    };
    
    let email = user1.email;
}
```
all the fields that are *not* explicitly specified (`username`, `sign_in_count`)
are copied or moved from the expression after `..` -- in this case, from `user1`.

Note that it *does not* act like this:
```rust
    let user2 = {
        let mut tmp = user1;
        tmp.email = String::from("another@example.com");
        tmp.active = true;
        tmp
    };
```
Instead it acts like this:
```rust
    let user2 = User {
        email: String::from("another@example.com"),
        active: true,
        username: user1.username,
        sign_in_count: user1.sign_in_count,
    };
```
And that is why you can use the unmoved fields of `user1`.
```rust
    let email = user1.email;
```
Because the thing after `..` can be any expression that has the same type as
the struct, a common idiom is:
```rust
    let user2 = User {
        email: String::from("another@example.com"),
        active: true,
        ..Default::default(),
    };
```
But note that this requires that `User: Default`, and constructs an entire `User`
to pull fields out of.  (Then the rest of the temporary value drops).  That is,
`Default::default` is not magical here, and it does not act field-by-field.

## [File hierarchy](https://doc.rust-lang.org/rust-by-example/mod/split.html)

`my/mod.rs` is an alternative to `my.rs`:

```
$ tree .
.
├── my
│   ├── inaccessible.rs
│   ├── mod.rs           // <---
│   └── nested.rs
└── split.rs
```
Some prefer this as it keeps all of the `my` module's code under the `my` directory.

Other projects use `mod.rs` for legacy reasons.

I'm not going to go into details, but many things about modules changed in edition
2018, so this topic is another reason to always specify an edition (or just use cargo).

# Cargo

Cargo should be introduced in chapter 1 (or 0), with a forward link to advanced features.

## [Dependencies](https://doc.rust-lang.org/rust-by-example/cargo/deps.html)

You can add dependencies with [`cargo add`.](https://doc.rust-lang.org/cargo/commands/cargo-add.html)

# [Generics](https://doc.rust-lang.org/rust-by-example/generics.html)

They introduce turbofish [in the next section,](https://doc.rust-lang.org/rust-by-example/generics/gen_fn.html)
but turbofish isn't specific to functions.  You need turbofish whenever you
need to specify a generic paramter in an expression context (as opposed to a type context).

```rust
struct SingleGen<T> { field: T }

fn main() {
    //         vvvvvvvvvvvvvvvv annotations are a type position
    let _char: SingleGen<usize> = SingleGen { field: 0 };
    let _char = SingleGen::<usize> { field: 0 };
    //          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ this is an expression
}}
```

## [Implementation](https://doc.rust-lang.org/rust-by-example/generics/impl.html)

> Similar to functions, implementations require care to remain generic.

Their examples "require care" because they named a struct `S`.

This isn't really a hazard unless you name your types the same way
you name your generic type parameters.

## [Traits](https://doc.rust-lang.org/rust-by-example/generics/gen_trait.html)

`Drop::drop` doesn't take things by value.  This section needs a better example.
[`Add`](https://doc.rust-lang.org/std/ops/trait.Add.html) maybe.

## [Bounds](https://doc.rust-lang.org/rust-by-example/generics/bounds.html)

`where` should probably be introduced here, not later.

### [Testcase: empty bounds](https://doc.rust-lang.org/rust-by-example/generics/bounds/testcase_empty.html)

`Copy` is a bad example.  While it doesn't have methods, it definitely has
different capabilities.  Magical ones, even.

## [Where clauses](https://doc.rust-lang.org/rust-by-example/generics/where.html)

It's worth highlighting that there is no difference between a bound specified
in a `where` clause and a bound specified inline:
```rust
fn printer<T: Display>(t: T) {
    println!("{}", t);
}

fn printer<T>(t: T)
where
    T: Display,
{
    println!("{}", t);
}
```

And that inline bounds are still called "`where` clauses".

This is a bad example:
```rust
// Because we would otherwise have to express this as `T: Debug` or 
// use another method of indirect approach, this requires a `where` clause:
impl<T> PrintInOption for T where
    Option<T>: Debug {
```
As "another indirect approach" is just to:
```rust
impl<T: Debug> PrintInOption for T { // ...
```
So this does not require an (explicit) `where` clause.

A better example would be one that literally can't be written inline,
e.g. a bound on something that's not a parameter of the function.
```rust
trait Iterator {
    fn count(self) -> usize
    where
        Self: Sized,
```

## [New Type Idiom](https://doc.rust-lang.org/rust-by-example/generics/new_types.html)

Rust has ["orphan rules"](https://rust-lang.github.io/rfcs/2451-re-rebalancing-coherence.html#concrete-orphan-rules)
which basically mean that you can only implement a foreign trait for a foreign type
in very specific circumstances.

But you can always implement a trait (foreign or local) for local types (provided any required bounds are met).

Because of that, another common use of the new type pattern is to wrap up a foreign type so that you can
implement traits (or inherent methods!) on it.

## [Associated items](https://doc.rust-lang.org/rust-by-example/generics/assoc_items.html)

[You can have associated `const`s too.](https://doc.rust-lang.org/std/primitive.i32.html#implementations)

### [Associated types](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html)

You can have generic associated types (GATs) too.

```rust
trait Useful {}
trait Example {
    type Gat<'a>: Useful where Self: 'a;
    fn example(&self) -> Self::Gat<'_>;
}

impl Useful for &str {}
impl Example for String {
    type Gat<'a> = &'a str;
    fn example(&self) -> Self::Gat<'_> {
        &**self
    }
}
```

## Phantom type parameters
### [Testcase: unit clarification](https://doc.rust-lang.org/rust-by-example/generics/phantom.html)

The goal of this section is to demonstrate why `PhantomData` is useful.

However, this could be rewritten to use zero-sized types instead of uninhabited `enum`s:
```rust
struct Inch;
struct Mm;
```
(And IMO it would be an improvement.)

But there are other scenarios where you *need* `PhantomData` because
- The type you need as a parameter isn't trivial
- You need a lifetime or such
- You need to influence auto-traits or variance
- Probably more

Perhaps this section should just [refer elsewhere](https://doc.rust-lang.org/nomicon/phantom-data.html)
like [the official docs on `PhantomData` does.](https://doc.rust-lang.org/std/marker/struct.PhantomData.html)

# [Scope](https://doc.rust-lang.org/rust-by-example/scope.html)

I recommend taking care to distinguish between Rust lifetimes (`'_`),
lexical scopes (like `{ /* ... */ }` blocks), and the liveness scope
of values -- when values go out of scope.

Lifetimes -- as in, those `'_` things -- haven't been directly tied to
lexical scopes since NLL (non-lexical lifetimes), 5+ years ago.  Probably
they don't want to restructure the book in a way that breaks a lot of
links, but the naming of this chapter is unfortunate.

The main way lifetimes and scopes interact is that a value is considered
to get used when it goes out of scope, and that use is incompatible
with the value being borrowed.
```rust
{
    let x = ();
    let borrow = &x; // Say this has lifetime `'b`
}
// `x` goes out of scope at the `}`
// `'b` cannot be alive here as that would mean `x` is still borrowed
```

## [RAII](https://doc.rust-lang.org/rust-by-example/scope/raii.html)

> you'll never have to manually free memory or worry about memory leaks again!

Nit: You might have to worry about reference counter loops with `Rc` and `Arc`.
And naturally you'll need some sort of manual garbage collection if you have
a cache in a long-running process or the like.

Eh, don't sweat it until you run into it though.  (E.g. if you make some cyclic structures.)

### [Destructor](https://doc.rust-lang.org/rust-by-example/scope/raii.html#destructor)

Types that *don't* implement `Drop` can also have non-trivial destructors.
`Drop` is more about custom destructors than having a destructor at all.

A `T: Drop` bound is more-or-less useless; a type that doesn't meet the bound may
still have a destructor.

## [Ownership and moves](https://doc.rust-lang.org/rust-by-example/scope/move.html)

> resources can only have one owner

Shared ownership can be modeled in Rust.  They say this themselves in the sections
on `Rc` and `Arc` later.

I'm not sure why they didn't bring up `Copy` in this section.
Here, just [go read the docs](https://doc.rust-lang.org/std/marker/trait.Copy.html)
about "move semantics" versus "copy semantics".

Types that implement `Copy` don't have destructors (or have trivial no-op destructors, if you prefer).

### [Mutability](https://doc.rust-lang.org/rust-by-example/scope/move/mut.html)

This is a weird thing to put under ownership in my opinion.  I'm not sure why
this wasn't part of the "freeze" section.

Anyway, we've already covered this above: You can move a variable between `mut` and non-`mut` bindings.

### [Partial moves](https://doc.rust-lang.org/rust-by-example/scope/move/partial_move.html)

You can't partially move something that has a `Drop` implementation.  

```rust
struct Foo {
    a: String,
    b: String,
}

impl Drop for Foo {
    fn drop(&mut self) {}
}

fn example(foo: Foo) {
    // compilation errors
    let a = foo.a;
    let Foo { a, ref b } = foo;
}
```

You can't destructure types from foreign traits that have private fields
or the [`non_exhaustive`](https://doc.rust-lang.org/reference/attributes/type_system.html#the-non_exhaustive-attribute)
attribute.

## [Borrowing](https://doc.rust-lang.org/rust-by-example/scope/borrow.html)

N.b. there's an implicit auto-deref going on in some of the code.
```rust
fn borrow_i32(borrowed_i32: &i32) {
    println!("This int is: {}", borrowed_i32);
}
// ...
    let boxed_i32 = Box::new(5_i32);
    borrow_i32(&boxed_i32); // Same as `&*boxed_i32`
```

Note: Shared refernces (`&T`) implement `Copy`.

### [Mutability](https://doc.rust-lang.org/rust-by-example/scope/borrow/mut.html)

Again I suggest a more precise terminology:
- `&mut` are exclusive references
- `&T` are shared references
  - "immutable" is inaccurate in the general case due to interior (shared) mutablity

This book doesn't seem to cover interior mutability at all.  Interior mutability
underlies mechanisms like `Rc`, `Arc`, `Mutex`, atomics, and other synchronization
primitives.

[Interior mutability removes the immutability guarantee for shared references.](https://doc.rust-lang.org/core/cell/struct.UnsafeCell.html)

Here's a simple example of mutating through a shared reference:
```rust
use core::cell::Cell;
fn example(i: &Cell<i32>) {
    i.set(42);
}

fn main() {
    // no `mut`!
    let i = Cell::new(0);
    example(&i);
    println!("{}", i.get());
}
```

Like the docs say, there is no way to relax the exclusiveness guarantee for `&mut _`.

### [Aliasing](https://doc.rust-lang.org/rust-by-example/scope/borrow/alias.html)

`&mut T` does not implement `Copy`.  But you can reborrow through it.

Instead of rewriting everything, [I'll just link here.](https://quinedot.github.io/rust-learning/st-reborrow.html)

## [Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)

It's clear from the introduction that there have been some updates to dinstinguish
between scopes and lifetimes.  That's good!  However, RBE is using the term
"lifetime" both for the liveness scopes of values and for those `'_` things.

That's not uncommon, but I find it leads to misunderstandings when trying to
understand Rust (`'_`) lifetimes.  I will take care to use the term "lifetime"
only for those `'_` things.

In the example, `i` doesn't have a lifetime.  The diagram indicates where it
goes out of scope.

When the compiler does borrow check analysis, it doesn't assign `i` a
lifetime (one of those `'_` things).  It just knows that going out of
scope is a use of `i`, and checks that against any potential borrows
of `i`.

### [Explicit annotation](https://doc.rust-lang.org/rust-by-example/scope/lifetime/explicit.html)

What's `foo` here?  A function I guess?

Anyway, I think they're conflating concepts again.  I think they should introduce lifetime
bounds, aka outlives relationships, here instead of later.  In particular, the bound
```rust
Foo<'a, 'b, T>: 'x
```
holds if and only if all of these hold:
```rust
'a: 'x,
'b: 'x,
T: 'x,
```
In which case we say "`Foo<'a, 'b, 'T>` outlives `'x`".  Or perhaps "is valid for `'x`".

Note that outlives relationships are "greater or equal", not "strictly greater".

The wording on their examples here are also weird.  I don't think I've ever seen
anyone reason about function lifetimes by worrying about the functions "outliving"
anything.

What they were attempting to get at is this: Any lifetimes supplied to a function
that has a generic lifetime are chosen by the caller and must last at least
*just longer* than the function body.  In particular, this means that you can never
borrow a local variable for as long as a generic lifetime, because local variables
must be moved or go out of scope by the end of the function body.

More nits:
> A short lifetime cannot be coerced into a longer one.

It's not something you generally think about, but contravariant lifetimes -- which
can be coerced into a longer one (but not a shorter one) -- do exist.

> Because the lifetime is never constrained, it defaults to `'static`.

It doesn't matter what the unconstrained lifetime is, but it doesn't have a default.
But you can pretend it's `'static` if you want I suppose.

### [Functions](https://doc.rust-lang.org/rust-by-example/scope/lifetime/fn.html)

> Ignoring elision, function signatures with lifetimes have a few constraints:
>  - any reference must have an annotated lifetime.

Elision covers a majority of cases, so you should go read the rules.  People
generally only annotate lifetimes if they need to.

Here's the functions headers from their example when excercising elision.
```rust
// Didn't need any annotation
fn print_one(x: &i32) {

// Didn't need any annotation
fn add_one(x: &mut i32) {

// Didn't need any annotation
fn print_multi(x: &i32, y: &i32) {

// Finally we need an annotation... but for one lifetime, not two
fn pass_x<'a>(x: &'a i32, _: &i32) -> &'a i32 { x }
```

> - any reference being returned must have the same lifetime as an input or be static.

This part simply isn't true.
```rust
fn silly<'a>() -> &'a str {
    ""
}
```
Now, it's true this only compiles because the `""` can be a `&'static str`, but
that doesn't matter to the function signature.  And there are other
[less silly](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.leak) use cases.
```rust
pub fn leak<'a>(b: Self) -> &'a mut T
where
    A: 'a,
```

But the vast majority of the time, yes, an output lifetime will correspond to
an input lifetime, or be `'static`.

### [Methods](https://doc.rust-lang.org/rust-by-example/scope/lifetime/methods.html)

Again you would elide the lifetimes here in practice.

They mentioned it elsewhere, but should probably reiterate here...
- `self` is short for `self: Self`
- `&self` is short for `self: &Self`
- `&mut self` is short for `self: &mut Self`
- etc

### [Traits](https://doc.rust-lang.org/rust-by-example/scope/lifetime/trait.html)

The lifetime parameters of traits are *invariant*, meaning they cannot be
coerced to shorter or longer lifetimes.

### [Bounds](https://doc.rust-lang.org/rust-by-example/scope/lifetime/lifetime_bounds.html)

By "all references in `T`" they mean that bounds are recursive on their generic parameters,
like I noted before.
```rust
Foo<'a, 'b, T>: 'x
```
holds if and only if all of these hold:
```rust
'a: 'x,
'b: 'x,
T: 'x,
```
(Note that not all lifetime-carrying types actually contain references.)

Again you would elide the lifetimes in `print_ref`.  The `T: 'a` bound
is implied by the presence of `&T` in the inputs.  For reasons I don't want
to go into here, the function is actually more general if you elide the lifetimes.

You can't elide lifetimes in the struct definition, but you can elide bounds
implied by the fields.  So these two are equivalent:
```rust
struct Ref<'a, T: 'a>(&'a T);
struct Ref<'a, T>(&'a T);
```
Because, similar to in `print_ref`, the presence of `&'a T` implies the `T: 'a` bound.

[These actually do have a subtle difference when `T = dyn Trait`:](https://quinedot.github.io/rust-learning/dyn-elision-advanced.html#guiding-behavior-of-your-own-types)
```rust
struct Ref<'a, T: 'a + ?Sized>(&'a T);
struct Ref<'a, T: ?Sized>(&'a T);
```
...but it's one that almost no-one knows or needs to care about in practice, as far as I know.
(It also requires the `?Sized` implicit-bound-removal to be relevant.)

### [Coercion](https://doc.rust-lang.org/rust-by-example/scope/lifetime/lifetime_coercion.html)

Lifetimes in *covariant* positions can be coerced to shorter lifetimes.  That's the most common,
and includes the (outer) lifetime of references (shared and exclusive).

However, Rust also has *invariance* and *contravariance*.  You normally don't have to worry
about the latter.  But you will definitely run into invariance at some point.  Invariant
lifetimes cannot be coerced to a different lifetime.

[Here's some more on the topic.](https://quinedot.github.io/rust-learning/st-invariance.html)

### [Static](https://doc.rust-lang.org/rust-by-example/scope/lifetime/static_lifetime.html)

Lifetimes can't use keyword names, but to date, `'static` is the only special named lifetime
you actually have to deal with.  Well, and the wildcard `'_` lifetime I guess, if you want
to call that a name.

#### [Reference lifetime](https://doc.rust-lang.org/rust-by-example/scope/lifetime/static_lifetime.html#reference-lifetime)

Again, I wouldn't conflate constants with `static`s.

The `&'static str` case applies to anything else that can undergo [constant promotion,](https://doc.rust-lang.org/reference/destructors.html#constant-promotion)
like a `&i32` say.

```rust
fn example<T: 'static>(_: T) {}
fn main() {
    example(&0);
}
```

#### [Trait bound](https://doc.rust-lang.org/rust-by-example/scope/lifetime/static_lifetime.html#trait-bound)

The implied definition of "owned" is circular here.  Sometimes people use "owned" to mean "satisfies a `'static` bound".
But sometimes they use it to mean "responsible for cleaning up resources in a RAII sense".

I'd say `Vec<&str>` owns the references within, and the memory it allocated to store them.  It doesn't own
the `str` data that the references point to.  Yet it only satisfies a `'static` bound if it's a `Vec<&'static str>`.

Even if it's `Vec<&'static str>`, it doesn't own the `str` data that the references point to.
(Arguably nothing does, since it's borrowed forever.)

### [Elision](https://doc.rust-lang.org/rust-by-example/scope/lifetime/elision.html)

Yeah... this should have came *before* all those examples.  Or at least a big heads-up that
having unnecessary named lifetimes is very unidiomatic.

# [Traits](https://doc.rust-lang.org/rust-by-example/trait.html)

> A `trait` is a collection of methods defined for an unknown type: `Self`.

"unknown type" is odd phrasing.  The implementing type is sort of like a generic of the trait.
Trait do *not* have an implicit `Sized` bound, by the way.

Traits can have bounds that prevent implementing the trait for *any* type.

## [Derive](https://doc.rust-lang.org/rust-by-example/trait/derive.html)

It's worth noting that if your struct has any generic type parameters, the built-in
`derive` will put a bound on every type parameter that corresponds to the trait.
For example,
```rust
#[derive(Default)]
struct MyOption<T>(Option<T>);
```
will do something like
```rust
impl<T: Default> Default for MyOption<T> {
    fn default() -> Self {
        Self(Option::<T>::default())
    }
}
```
even though the `T: Default` is not required for `Option<T>` to implement `Default`.

If you don't need and don't want this restriction, you have to implement the trait yourself.

If you want to match `const`s of your type in patterns (like `match`), you currently have
to derive `PartialEq` and `Eq`.  This may change in the future.
([See here.](https://doc.rust-lang.org/core/marker/trait.StructuralPartialEq.html))

## [Returning Traits with `dyn`](https://doc.rust-lang.org/rust-by-example/trait/dyn.html)

This section reads like it was written when you could just write `Animal` instead of
`dyn Animal` for the trait object (edition 2015).  Back then, it was contextual
whether you meant the `dyn Animal` type or the `Animal` trait.

It needs a rewrite to consistently use `dyn Trait` when it means the trait object.

> This means all your functions have to return a concrete type

Eh, I don't agree with this.  I mean, *all* functions have to return a concrete type,
because Rust is statically typed.  Also, `dyn Trait + '_` *is* a concrete type.

The problem is that there's no way to return *unsized* types.  Like `[T]`, `dyn Trait`
is a dynamically sized type (DST) that does not implement `Sized`.  All functions have
to return a `Sized` type.

You're never "returning a trait", even when you box it up.  You're type erasing your
original value and returning a `Box<dyn Animal>` or whatever.  That's its own type.
Traits aren't types.

[See here](https://quinedot.github.io/rust-learning/dyn-trait-overview.html) for an
introduction to what `dyn Trait` is and isn't.

## [Operator Overloading](https://doc.rust-lang.org/rust-by-example/trait/ops.html)

I'll just note that any time you read that an operator is "equivalent to" calling
a method, it's not strictly true; there are subtle differences around type
inference and the like.

## [Drop](https://doc.rust-lang.org/rust-by-example/trait/drop.html)

At least today, `String` doesn't implement `Drop`.
```rust
// Fails today
fn witness<T: Drop>() {}
fn main() {
    witness::<String>();
}
```
As I mentioned before, types can have destructors even if they don't implement `Drop`.
If a type has a non-trivial destructor, it's an implementation detail as to whether
that's due to `Drop` or not.  Technically removing a `Drop` implementation is a
breaking change, but practically, it's not something anyone should be relying on.

[See also.](https://doc.rust-lang.org/std/mem/fn.needs_drop.html)

You can't implement `Drop` if you implement `Copy`.

## [Iterators](https://doc.rust-lang.org/rust-by-example/trait/iter.html)

> or automatically defined (as in arrays and ranges).

Not true but not important either really...

As mentioned before, it's `IntoIterator` that supplies the `into_iter` method.

## [`impl Trait`](https://doc.rust-lang.org/rust-by-example/trait/impl_trait.html)

This section should go over the capturing rules of return-position `impl Trait`,
how it differs inside and outside of traits, how that may change across editions,
the relation with `async fn`, and the future plans with type aliases.

OK fine, I'll do it here.

Return-position `impl Trait` (RPIT) is a form of thin, opaque aliasing.  You can
only return one concrete type from an RPIT function, so this doesn't work:
```rust
fn example(b: bool) -> impl std::fmt::Display {
    match b {
        false => 0,
        true => String::new(),
    }
}
```
There's no `dyn Trait`-like type erasure; the compiler still knows the underlying
concrete type on some level.  It just limits what consumers of the function can
know about it.  Among other things, this gives the function writer more flexibility --
they can change the concrete type used without breaking downstream.

Next, `async fn` are, roughly speaking, sugar for the following pattern:
```rust
// async fn foo<Input>(t: Input) -> OutputTy { /* ... */ }
fn foo<Input>(t: Input) -> impl Future<Output = OutputTy> {
    async move {
        _always_moves_everything = (&t,);
        /* ... */
    }
}
```
There are some subtle differences, which we'll return to soon.  But the part to
remember is that when we're talking about `async fn`, we're really talking about
RPIT -- an opaque return type.

With a normal function (without an opaque return type), and a healthy understanding
of lifetime elision, you can generally tell when the return type "captures" a generic
input of the function; that is, when the output type is also parameterized by the
generic input lifetime or type:
```rust
// "Captures" `T` but does not capture any lifetimes
fn example_1<T: Clone>(t: &T) -> Option<T> { None }

// "Captures" the lifetime but does not capture `T`
// You can tell due to the presence of `&`.  With less elision, the signature is:
// - `fn example_2<T: AsRef<str>>(t: &'_ T) -> &'_ str`, or
// - `fn example_2<'a, T: AsRef<str>>(t: &'a T) -> &'a str`
fn example_2<T: AsRef<str>>(t: &T) -> &str { "" }

// "Captures" both the lifetime and `T`
fn example_3<T: Debug>(t: &T) -> &T { t }
```

You can see how the types and/or lifetimes "flow into" the output type.

However, with RPIT, we don't currently have a great way to annotate generic lifetimes
and types on the return type.  So what do they capture?  As it turns out, it's
contextual.

- `async fn` everywhere and RPIT in traits (RPITIT (I know, ugh)) capture all generic inputs
- RPIT outside of traits *before edition 2024* captures generic types, but not generic lifetimes
- In edition 2024, [the plan is for all RPITs to capture lifetimes](https://github.com/rust-lang/rust/issues/117587)

Is there a way to deal with undercapturing RPITs in edition 2021 -- to make them capture the lifetimes?
There are ways, but they are clunky or imperfect.

The imperfect way is to slap `+ '_` on the end of the RPIT:
```rust
// Fails without the annotation                                         vvvv
fn example<T: ?Sized + AsRef<str>>(t: &T) -> impl Iterator<Item = char> + '_ {
    t.as_ref().chars()
}
```

It's imperfect as this actually imposes a bound on the opaque type -- and that bound applies to
*every generic type parameter*.

The clunky way is to find some other way to mention the lifetime:
```rust
// This trait doesn't mean anything, and we implement it for everything.
// It's just a way to mention a lifetime.
trait Captures<'a> {}
impl<T: ?Sized> Captures<'_> for T {}

fn example<T: ?Sized + AsRef<str>>(t: &T) -> impl Iterator<Item = char> + Captures<'_> {
    t.as_ref().chars()
}
```
There are also some subtle differences between capturing by mention only,
and capturing the concrete type containing the lifetime, which I won't go into here.

Alternatively you can wait for edition 2024, where *under*capturing will supposedly go away.

Ok then, but, is there a way to deal with *over*capturing?  There is not one yet on stable,
but it's planned.  The core idea is called `type` alias `impl Trait` (TAIT), and it allows
giving opaque types names by creating an alias -- and that alias can have parameters.

A TAIT captures whatever parameters you give it, but no more.
```rust
// Not available on stable yet
#![feature(type_alias_impl_trait)]
type MyIter<T: Clone> = impl Iterator<Item = (String, T)>;

// Captures `T`, but not the lifetime from the `&str`
fn example<T: Clone>(s: &str, t: T) -> MyIter<T> {
    // This is a silly (non-lazy) way to make an interator; for illustration only
    let vec: Vec<_> = s.split_whitespace().map(String::from).collect();
    vec.into_iter().map(move |s| (s, t.clone()))
}
```
Now, like non-opaque returning functions, the captures are "visible" again.

All of the RPIT and RPITIT and `async fn` versions can be described in terms of
TAITs.  The in-trait version of `TAIT` is a generic associated type (GAT) that
is given the "type" `impl Trait`.

## [Clone](https://doc.rust-lang.org/rust-by-example/trait/clone.html)

> When dealing with resources, the default behavior is to transfer them during assignments or function calls.

...unless the type implements `Copy`.

"Without resources" is a false distinction (or a cyclic defintion).  In order to implement
`Copy`, all your fields must recursively and structurally also implement `Copy`.  I wouldn't
say an `Option<i32>` is "without resources", but it implements `Copy`.  If you define a
unit struct and don't implement `Copy`, you could say it is "without resources", but it
still doesn't implement `Copy`.

Implementing `Copy` requires implementing `Clone`.

# [`macro_rules!`](https://doc.rust-lang.org/rust-by-example/macros.html)

I skipped this part :-)

# [Error handling](https://doc.rust-lang.org/rust-by-example/error.html)

> When there is a chance that things do go wrong and the caller has to deal with the problem, use `Result`.
> You can `unwrap` and `expect` them as well (please don't do that unless it's a test or quick prototype).

No good rule of thumb is as simple as "please don't do that".

https://blog.burntsushi.net/unwrap/

## [`panic`](https://doc.rust-lang.org/rust-by-example/error/panic.html)

Depending on the compilation, programs may abort on `panic`, instead of unwinding.

...oh, I see they correct themselves in the immediately following section.  I don't know why
they said it unwinds without qualifying the statement, then.

## [`Option` & `unwrap`](https://doc.rust-lang.org/rust-by-example/error/option_unwrap.html)

I only skimmed this part :-)

They might not have mentioned `ok_or_else` to turn an `Option` into a `Result`.

They might not have mentioned `let else`:
```rust
fn does_not_return_option(an_option: Option<Whatever>) {
    let Some(variable) = an_option else { return };
}
```

## [`Result`](https://doc.rust-lang.org/rust-by-example/error/result.html)

I only skimmed this part :-)

There could be some re-ordering here.  They "introduce `?`", but they already did that for `Option`.

They might not have mentioned `.ok()` to turn a `Result` into an `Option`.

## [Multiple error types](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types.html)

Good error handling depends on a lot of things (perhaps the most fundamental is, who
is your audience?  Are you an end-user binary or a library?).  If you care about good
error handling, you're probably better off finding some dedicated blog posts or other
resources on the topic.

Heads-up though, it's a large topic.

https://sabrinajewson.org/blog/errors

### [Other uses of `?`](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/reenter_question_mark.html)

> `?` was previously explained as either `unwrap` or `return Err(err)`. This is only mostly true. It actually means `unwrap` or `return Err(From::from(err))`.

[It's actually more complicated than that.](https://doc.rust-lang.org/std/ops/trait.Try.html)

But it acts mostly as if it did that, for `Result`.

# [Std library types](https://doc.rust-lang.org/rust-by-example/std.html)
## [Box, stack and heap](https://doc.rust-lang.org/rust-by-example/std/box.html)

It doesn't look like they cover how `*` is magical for `Box`.  You can move
non-`Copy` types out of a `Box` with `*` too.  For instance you can remove
the `derive(Copy)` from their example and it still compiles, even though it
contains this code:
```rust
let unboxed_point: Point = *boxed_point;
```

## [Vectors](https://doc.rust-lang.org/rust-by-example/std/vec.html)

> Like slices, their size is not known at compile time, but they can grow or shrink at any time.

The *length of the data they point to* is not known at compile time.  `Vec<_>` itself is `Sized`.

```
+---+---+---+---+---+---+---+---+
| Pointer       | Length        | &[T] (or &str, &Path, &OsStr, ...)
+---+---+---+---+---+---+---+---+
  |
  V
+---+---+---+---+---+---+---+---+
| D | A | T | A | . | . | . | ......    [T] (or str, Path, OsStr, ...)
+---+---+---+---+---+---+---+---+   
  ^
  |
+---+---+---+---+---+---+---+---+---+---+---+---+
| Pointer       | Length        | Capacity      | Vec<T> (or PathBuf, String, OsString, ...)
+---+---+---+---+---+---+---+---+---+---+---+---+
  ^
  |
+---+---+---+---+
| Pointer       | &Vec<T> (or &PathBuf, &String, &OsString, ...)
+---+---+---+---+
```

N.b. the order of pointer/length/capacity is unspecified and subject to change.

Incidentally, I don't think this book mentions that you should prefer `&[T]` over `&Vec<T>`
(et cetera) in your APIs.  There's not much you can do with a `&Vec<_>` that you can't do
with a `&[_]` (check capacity, I guess).  And as the diagram above demonstrates,
`&Vec<_>` introduces an extra layer of indirection.

(`&mut Vec<T>`, et cetera, is a different scenario.)

## [Option](https://doc.rust-lang.org/rust-by-example/std/option.html)

`Option` is for values that might not be present generally.  It has no special or intrinsic
relation to failures or `panic!`.  `None` doesn't inately mean something unfortunate is going on.

## [Result](https://doc.rust-lang.org/rust-by-example/std/result.html)

### [`?`](https://doc.rust-lang.org/rust-by-example/std/result/question_mark.html)

[`?` is not `Result` specific.](https://doc.rust-lang.org/core/ops/trait.Try.html#implementors)

## [`panic!`](https://doc.rust-lang.org/rust-by-example/std/panic.html)

Panics might abort instead of unwinding the stack.

There are ways to catch unwinding panics, which I don't think this guide covers.
(My notes here don't either.)

## [`HashMap`](https://doc.rust-lang.org/rust-by-example/std/hash.html)

> Like vectors, `HashMaps` are growable, but `HashMaps` can also shrink themselves when they have excess space. 

Uh, `Vec`s can do that too.  They both have `shrink*` methods. As far as I know, neither of the `std` data
structures sheds excess capacity unless you request it does so.

> or use `HashMap::new()` to get a HashMap with a default initial capacity (recommended).

`new` is documented to start with a capacity of 0.

Probably this section is conveying implementation details of the pre-hashbrown `std` `HashMap`.

### [Alternate/custom key types](https://doc.rust-lang.org/rust-by-example/std/hash/alt_key_types.html)

The mention of `int` and `uint` probably means this page hasn't been updated since before Rust 1.0.

### [`HashSet`](https://doc.rust-lang.org/rust-by-example/std/hash/hashset.html)

> If you insert a value that is already present in the `HashSet`, (i.e. the new value is equal to 
> the existing and they both have the same hash), then the new value will replace the old.

[Exactly wrong,](https://doc.rust-lang.org/std/collections/hash_set/struct.HashSet.html#method.insert)
the new value is dropped and the old one retained.

## [`Rc`](https://doc.rust-lang.org/rust-by-example/std/rc.html)

This is one of the `std` shared ownership types mentioned above.

> Reference count of an `Rc` increases by 1 whenever an `Rc` is cloned, and decreases by 1 whenever one cloned `Rc` is dropped out of the scope.

That's what happens to the *strong* count, yes...

> When an `Rc`'s reference count becomes zero (which means there are no remaining owners), both the `Rc` and the value are all dropped.

...but this is not correct (or at best very misleading).  When the strong count reaches 0, the shared value is dropped.
When the *weak* count reaches 0, *then* the shared backing memory is dropped.

## [`Arc`](https://doc.rust-lang.org/rust-by-example/std/arc.html)

Same drop behavior as `Rc`.

# Std misc
## [Threads](https://doc.rust-lang.org/rust-by-example/std_misc/threads.html)

Going to go out on a limb and guess they're not going to mention
[scoped threads,](https://doc.rust-lang.org/std/thread/fn.scope.html)
which can capture locals / need not be `'static`.

## [Path](https://doc.rust-lang.org/rust-by-example/std_misc/path.html)

The "flavors" of `Path` is an implementation detail.  Probably their note
at the bottom to check out the methods are what evolved into the `os`
module extension traits, [like these.](https://doc.rust-lang.org/std/os/unix/ffi/index.html#traits)

> A `Path` is immutable. The owned version of `Path` is `PathBuf`. 

No, a `Path` is not immutable.  You can have a `&mut Path`, no problem.
You could use that to get a `&mut OsStr` and then lowercase all the ASCII characters, for example.

A `Path` is unsized, like `str` and `[T]`.  It's not necessarily borrowed.  You can have a
(borrowed) `&Path` or `&mut Path`.  Or you can have a `Box<Path>`.

> The relation between `Path` and `PathBuf` is similar to that of `str` and `String`: a `PathBuf` can be mutated in-place, and can be dereferenced to a `Path`.

This is more accurate.

Everything I just said about mutability and ownership applies to `str` as well as `Path`.

> Note that a `Path` is not internally represented as an UTF-8 string, but instead is stored as an `OsString`

`PathBuf` corresponds to `OsString`.  `Path` corresponds to `OsStr`.

>  a `Path` can be freely converted to an `OsString` or `&OsStr`

Conversion from `&Path` to `OsString` (or `PathBuf`) isn't free, it allocates.

Maybe they meant infallibly.

## [File I/O](https://doc.rust-lang.org/rust-by-example/std_misc/file.html)

There's also a `create_new`.

### [`read_lines`](https://doc.rust-lang.org/rust-by-example/std_misc/file/read_lines.html)

> We also update `read_lines` to return an iterator instead of allocating new `String` objects in memory for each line.

`BufRead` allocates a new `String` per line too.

## [Filesystem operations](https://doc.rust-lang.org/rust-by-example/std_misc/fs.html)

(I'm skimming at this point but) these are awful implementations of commands like `cat`.
Files don't have to be UTF8 and reading the entire thing in first is inefficient, to
pick one example.

## [Program arguments](https://doc.rust-lang.org/rust-by-example/std_misc/arg.html)

Use `args_os` so you don't panic on non-UTF8 args.

# [Testing](https://doc.rust-lang.org/rust-by-example/testing.html)

I skipped this part :-)

# [Unsafe Operations](https://doc.rust-lang.org/rust-by-example/unsafe.html)

I only skimmed this part.

If you're going to venture into `unsafe` yourself, find a dedicated guide.
You should know it exists and what it means, but no introductory text is
going to give you enough information to use it soundly.
