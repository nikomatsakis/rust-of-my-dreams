# Well-combining combinators

## aka, "Maximally minimal" effects

"Effect systems" mean a lot of things to a lot of people. On the one hand, you have relatively simple annotations like Rust's `unsafe` or Java's `throws` annotations. These annotations don't have dynamic semantics, they just signal something that might happen in the body of the function as part of its signature, so that the caller can be aware of it. On the other extreme you have languages like Koka that use effects to mean "scoped handlers" which, combined with [Continuation Passing Style][CPS] allow for modeling a wide variety of language features (like exceptions and generators). 

I'm aiming for neither of these extremes. I'm interested in a very specific problem -- if you have ever tried to use `?` in an iterator, you'll realize you have to rewrite to use a `for` loop. Same with `await`. **This is the problem I would lke to solve.** I want you to be able to write combinator methods like `map` that work smoothly with async functions, [try functions](./try.md), and `const` fn (perhaps even `unsafe` fn?).

## Background: Monadic-like pairing

Rust has a number of "monadic" like effects, where a function gets a special "keyword" that comes in front and then a matching operation on the other side that "undoes" this keyword. Some o fthese monadic effects are not

* `async fn foo()` is paired with `.await`, as in `foo().await`
* `try fn foo()` ([proposed here](./try.md)) is paired with `?`, as in `foo()?`
* `async try fn foo()` ([proposed here](./async_try.md)) is paired with `.await?`, as in `foo()?`
* `unsafe fn foo()` is paired with `unsafe { foo() }` (not sure if this matters)

In addition there are "restrictions" -- that is, indications that a function does NOT have some effect that functions have by default:

* `const fn foo()` can be called in `const` contexts, unlike regular functions

To be clear, the analogy between these items is only "skin deep". They work in very different ways. `async fn` desugars to `-> impl Future`, for example, and `try fn` desugars to `Result`. But you can see that they at least **feel** similar -- and they have another similarity too. They all package up the return value in some kind of wrapper and then allow it to be unpacked. E.g., when you use the "pairing" operation (e.g., `await`, `?`), what you get back is the "regular return value" of the function. This is what I mean by them being "monadic".

## Credit where credit is due

Heavily influenced by [Yosh](https://github.com/yoshuawuyts) and [Oli](https://github.com/oli-obk/) and their work on [Keyword Generics](https://blog.rust-lang.org/inside-rust/2022/07/27/keyword-generics.html).



