# Do genericity

The idea of `do` is to allow a **single** "effect parameter". I don't believe we ever need more than one (famous last words? perhaps).

## Basic idea

You can write `do` in front of 

* a function/method
* a trait name, both in the definition

When you write `do` in front of a function, it means the effects of the function are dependent on some of its type parameters:

```rust
do fn compose<T, R>(
    input: T,
    op: impl do FnMut(T) -> R,
) -> R {
    op(input).do
}
```

When you call a `do` function, the effect parameter that gets used is dependent on context. If you are inside an async context, it becomes `async`. If you are inside a try context, it becomes `try`. If you are inside a normal function, it is normal. 

If you call a `do` function from inside a `do` context (as in the example above), you MUST use `.do` to "discharge" the unknown effect (again, as show in the example above). Not doing so will yield an error.

## `do` traits

You can write a `do` trait and, if you do so, it can have a mix of `do` functions and non-`do` functions:

```rust
impl do Iterator {
    type Item;

    do fn next(&mut self) -> Option<Self::Item>;

    fn map<F, R>(&mut self, op: F) -> Map<Self, F>
    where
        F: do FnMut(Self::Item) -> R
    {
        Map { iter: self, op }
    }
}
```

The functions identified as `do` are the ones which will have the effect identified by the trait when executed. You can leave off the `do` keyword (as in `map` above) to indicate a function that will not have the `do` effect -- in this case, because it simply creates a combinator, it doesn't actually **do** anything.

## Using a `do` trait

Here is an example of using the `do` trait from within an async function. The compiler figures out the closure can be compiled as async, so that doesn't have to be written explicitly.

```rust
async fn test(
    messages:  &[Messages]
) -> Vec<ProcessedMessage> {
    messages
        .iter()
        .map(|m| process(m).await)
        .collect()
}

async fn process(m: Message) -> ProcessedMessage {
    open_socket(m.addr).await
        .unwrap() // <-- ignore errors...
        .read_data().await
        .unwrap() // <-- ...we'll fix this later
}
```

Here is the same example of using `Iterator` but within a [try](./try.md) function.

```rust
try fn process_all(
    messages:  &[Messages]
) -> Vec<ProcessedMessage> {
    messages
        .iter()
        .map(|m| process(m)?)
        .collect()
}

try fn process(m: Message) -> ProcessedMessage {
    if an_error_occurs {
        throw Error;
    }
    result
}
```

Finally here is the same example, but with async try:

```rust
async try fn process_all(
    messages: &[Messages]
) -> Vec<ProcessedMessage> {
    messages
        .iter()
        .map(|m| process(m).await?)
        .collect()
}

async try fn process(m: Message) -> ProcessedMessage {
    open_socket(m.addr).await?
        .read_data().await? // <-- assume fallible
}
```

See the similarity?

## Implementing the map combinator

Here is how we implement the map combinator

```rust
struct Map<I, F> {
    iter: I,
    op: F,
}

impl<I, F, R> do Iterator for Map<I, F, R>
where
    I: do Iterator,
    F: do FnMut(I::Item) -> R,
{
    type Item = R;

    do fn next(&mut self) -> Option<R> {
        let o = self.iter.next().do?;
        Some((self.op)(o).do)
    }
}
```

## Desugaring this


