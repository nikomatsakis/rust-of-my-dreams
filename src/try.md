# Try fns and try blocks

## TL;DR

Permit writing...

```rust
try {
    ...  something()? ...
    ... throw error; ...
    regular_return
}
```

...which will (by default) return a `Result<T, std::failure::Failure>` (where failure is the [single failure type](./fail))
The `Failure`
The `throw` keyword will cause the result
Note that no `Ok` is required.