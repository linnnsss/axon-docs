---
title: Procedural Macros in Instrumentation
hide_title: true
sidebar_position: 3
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Procedural Macros in Instrumentation
As addressed in [this](https://docs.axonweb3.io/devops/monitoring-platform) article, in a distributed system, monitoring platform holds an immense significance in providing essential system feedback. As a peer-to-peer network, Axon relies on its real-time monitoring platform that contains logging, metrics, and tracing functionalities. Achieving these capabilities requires the addition of the instrumentation to the existing system.

This article first explores several existing instrumentation methods, then delves into the rationale behind Axon's adoption of procedural macros.

**Prometheus API**

Directly call the Prometheus API. This method requires adding code into the function, as shown below:
```rust
fn eth_sendRawTransaction(&self, tx_bytes: Vec<u8>) -> Result<H256, Error> {
    // Instrumentation: Increment the counter for the number of times this function is called.
    self.tx_send_total.with_label_values(&["call"]).inc();
    // function logic...
}
```
However, this approach is inconvenient and will modify the existing structure of the function.

**Aspect-Oriented Programming**

Aspect-Oriented Programming (AOP), is a programming paradigm that aims to modularize and manages cross-cutting concerns (e.g., logging, security, caching, etc.) that spread across multiple classes or modules. AOP separates these concerns from the core business logic and encapsulates them into reusable modules, known as "aspects.” However, AOP is currently only applicable to Java, not yet to Rust.

**LLVM IR**

LLVM IR (Low-Level Virtual Machine Intermediate Representation) is a low-level, platform-independent programming language that serves as an intermediate step during the compilation process.

Using LLVM IR in instrumentation basically involves the following steps:

1. Before compiling Rust code into LLVM IR, parse the Abstract Syntax Tree (AST) to find the functions that need instrumentation.
2. Generate corresponding instrumentation code for these functions.
3. Insert operations for instrumentation (e.g., inc) before entering and exiting these functions' IR instructions.
4. Re-package the code in Rust and compile it to obtain an executable file with the added instrumentation code.

Although the original code remains unaffected, instrumentation-related code has to be regenerated after each update, resulting in a cumbersome process.

**Procedural Macros**

Compared to C/C++ macros, Rust's procedural macros offer a powerful meta-programming model. Here are the pros and cons:

Pros:

- Highly flexible, suitable for metrics and traces.
- Zero runtime cost, since the compiler generates actual code during compilation.
- Good encapsulation and ease to use.

Cons: 

- Steep learning curve
- Limited portability, not easy to adapt across different environments
- Difficult to debug, needing [cargo-expand](https://docs.rs/crate/cargo-expand/0.1.3/source/README.md) command to support.

## Implement Procedural Macros in Axon

Axon leverages procedural macros in its instrumentation process to benefit from the above advantages. The following sections detail the specific implementations for metrics and trace tracking.

### Metrics

Let's first look at an example. Compared to using Prometheus, which requires adding metric code inside the send_transaction function, procedural macros only need to be added outside the targeted function, making the code cleaner and simpler.

```rust
#[metrics_rpc("net_listening")]
fn listening(&self) -> Result<bool, Error> {
    Ok(false)
}
```

### Procedural Macros: Code Example

The logic basically involves reporting different situations to Prometheus based on the function's execution, as shown below:

```rust
#[proc_macro_attribute]
pub fn metrics_rpc(attr: TokenStream, func: TokenStream) -> TokenStream {
    rpc_expand::expand_rpc_metrics(attr, func)
}

pub fn expand_rpc_metrics(attr: TokenStream, func: TokenStream) -> TokenStream {
       ...
       let func_block_wrapper = 
        quote! {
            let inst = common_apm::Instant::now();

            log::debug!(#debug, #fn_name, #( #args, )*);

            let ret: #func_ret_ty = #func_block;

            if ret.is_err() {
								// Report failure to Prometheus.
                common_apm::metrics::api::API_REQUEST_RESULT_COUNTER_VEC_STATIC
                    .#func_ident
                    .failure
                    .inc();
                return ret;
            }
            // Report success to Prometheus.
            common_apm::metrics::api::API_REQUEST_RESULT_COUNTER_VEC_STATIC
                .#func_ident
                .success
                .inc();
            common_apm::metrics::api::API_REQUEST_TIME_HISTOGRAM_STATIC
                .#func_ident
                .observe(common_apm::metrics::duration_to_sec(inst.elapsed()));

            ret
        };

    ...
}
```

### Macro Expansion: Code Example

The expanded macro parallels the procedural macros in a one-to-one manner, as shown below:

```rust
fn listening(&self) -> Result<bool, Error> {
    let inst = std::time::Instant::now();
    let ret: Result<bool, Error> = { Ok(false) };

    if ret.is_err() {
        common_apm::metrics::api::API_REQUEST_RESULT_COUNTER_VEC_STATIC
            .net_listening
            .failure
            .inc();
        return ret;
    }

    common_apm::metrics::api::API_REQUEST_RESULT_COUNTER_VEC_STATIC
        .net_listening
        .success
        .inc();
    common_apm::metrics::api::API_REQUEST_TIME_HISTOGRAM_STATIC
        .net_listening
        .observe(common_apm::metrics::duration_to_sec(common_apm::elapsed(inst)));
    ret
}
```

## Tracing

### Use Example

```rust
#[trace_span(kind = "trace")]
fn version(&self, ctx: Context) -> String {
    "0.1.0".to_string()
}
```
### Procedural Macros: Code Example
Since trace requires tracking the entire execution, trace_span has its own specific logic (*italicized* in the code below):
- Context must be added into function’s parameter list, and Context must be transferred during function call.
- The procedural macros must check whether the Context exists. If the Context is absent, procedural macros have to generate a new root context; if the Context exists, procedural macros should add the logic for the current function to the existing Context.

```rust
#[proc_macro_attribute]
pub fn trace_span(attr: TokenStream, func: TokenStream) -> TokenStream {
    trace_expand::expand_trace_span(attr, func)
}

pub fn expand_trace_span(attr: TokenStream, func: TokenStream) -> TokenStream {
    ...
    quote! {
        #[allow(unused_variables, clippy::type_complexity)]
        #func_vis #func_sig {
            use common_apm::tracing::{LogField, SpanContext, Tag, TRACER};

            let mut span_tags: Vec<Tag> = Vec::new();
            #(#span_tag_stmts)*

            let mut span_logs: Vec<LogField> = Vec::new();
            #(#span_log_stmts)*

            let mut span = if let Some(parent_ctx) = ctx.get::<Option<SpanContext>>("parent_span_ctx") {
                if parent_ctx.is_some() {
                    TRACER.load().child_of_span(#trace_name, parent_ctx.clone().unwrap(), span_tags)
                } else {
                    TRACER.load().span(#trace_name, span_tags)
                }
            } else {
                TRACER.load().span(#trace_name, span_tags)
            };

            let ctx = match span.as_mut() {
                Some(span) => {
                    span.log(|log| {
                        for span_log in span_logs.into_iter() {
                            log.field(span_log);
                        }
                    });
                    ctx.with_value("parent_span_ctx", span.context().cloned())
                },
                None => ctx,
            };

            #func_block_wrapper
        }
    }.into()
}
```

## Macro Expansion of `async_wait`

As Rust's standard library does not support `async_wait yet`, Axon uses the `async_wait` procedural macro in the asynchronous code. Functions that already include `async_wait may` also require instrumentation. This means that both `metrics_rpc` and `trace_span` procedural macros need to expand the code once again based on the already expanded code by `async_wait`. This step involves certain technical details explicated below. Here I use `send_transaction` as the example:

**Original Function**

```rust
#[async_trait]
pub trait Rpc {
    async fn send_transaction(&self, tx: SignedTransaction) -> Result<Hash, Error>;
}
```

**Macro Expansion of `async_wait`**

```rust
fn send_transaction<'life0, 'async_trait>(
    &'life0 self,
    tx: SignedTransaction,
) -> ::core::pin::Pin<
    Box<
        dyn ::core::future::Future<Output = Result<Hash, Error>>
            + ::core::marker::Send
            + 'async_trait,
    >,
>
where
    'life0: 'async_trait,
    Self: 'async_trait;
```
We can observe the following changes in the original function as the result of `async_wait`:

- `send_transaction` is no longer an `async` function, which is irrelevant to the expansion of metric and trace macros.
- The return value is modified into a highly complex `Pin<Box<dyn future::Future<Output = Result<Hash, Error>>>>`. This change requires metric and trace procedural macros to execute the different logics for different return types.

## Trace Procedural Macros

Take a look at the trace_span function and we can observe its extensive logic that handles various return types (related code *italicized*). For this, Axon has specifically released the fut-ret crate.

```rust
pub fn expand_trace_span(attr: TokenStream, func: TokenStream) -> TokenStream {
    ...
    let func_return = PinBoxFutRet::parse(func_output);
    ...
    let func_block_wrapper = if func_return.is_ret_pin_box_fut() && func_return.is_fut_ret_result()
    {
        let ret_ty = func_return.return_type();
        ...
    } else if func_return.is_ret_pin_box_fut() && !func_return.is_fut_ret_result() {
        quote! {
            Box::pin(async move {
                let _ = span;
                #func_block.await
            })
        }
    } else if !func_return.is_ret_pin_box_fut() && is_return_result(func_output) {
        ...
    } else {
        quote! { #func_block }
    };
    ...
}
```

