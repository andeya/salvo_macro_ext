# salvo_macro_ext

Unofficial extension macros for the [Salvo web framework](https://github.com/salvo-rs/salvo).

[![Crates.io](https://img.shields.io/crates/v/salvo_macro_ext)](https://crates.io/crates/salvo_macro_ext)
[![Documentation](https://shields.io/docsrs/salvo_macro_ext)](https://docs.rs/salvo_macro_ext)
[![License](https://img.shields.io/crates/l/salvo_macro_ext)](https://github.com/andeya/salvo_macro_ext?tab=MIT-1-ov-file)

## Features

-   Support converting multiple methods into handlers ([salvo issue #919](https://github.com/salvo-rs/salvo/issues/919))

## Example

```toml
[dependencies]
salvo_macro_ext = "0.1"
salvo = { version = "0.72", features = ["oapi"] }
tokio = "1"
```

### `#[module]`

`#[module]` is an attribute macro used to batch convert methods in an `impl` block into [`Salvo`'s `Handler`](https://github.com/salvo-rs/salvo).

```rust
use salvo::oapi::extract::*;
use salvo::prelude::*;
use salvo_macro_ext::module;
use std::sync::Arc;

#[tokio::main]
async fn main() {
    let service = Arc::new(Service::new(1));
    let router = Router::new()
        .push(Router::with_path("add1").get(service.add1()))
        .push(Router::with_path("add2").get(service.add2()))
        .push(Router::with_path("add3").get(Service::add3()));
    let acceptor = TcpListener::new("127.0.0.1:5800").bind().await;
    Server::new(acceptor).serve(router).await;
}

#[derive(Clone)]
pub struct Service {
    state: i64,
}

#[module]
impl Service {
    fn new(state: i64) -> Self {
        Self { state }
    }
    /// doc line 1
    /// doc line 2
    #[salvo_macro_ext::module(handler)]
    fn add1(&self, left: QueryParam<i64, true>, right: QueryParam<i64, true>) -> String {
        (self.state + *left + *right).to_string()
    }
    /// doc line 3
    /// doc line 4
    #[module(handler)]
    pub(crate) fn add2(
        self: ::std::sync::Arc<Self>,
        left: QueryParam<i64, true>,
        right: QueryParam<i64, true>,
    ) -> String {
        (self.state + *left + *right).to_string()
    }
    /// doc line 5
    /// doc line 6
    #[module(handler)]
    pub fn add3(left: QueryParam<i64, true>, right: QueryParam<i64, true>) -> String {
        (*left + *right).to_string()
    }
}
```

Sure, you can also replace `#[module(handler)]` with `#[module(endpoint(...))]`.

NOTE: If the receiver of a method is `&self`, you need to implement the `Clone` trait for the type.
