---
title: Proxy
hide_title: true
sidebar_position: 5
---

import useBaseUrl from "@docusaurus/useBaseUrl";

# Axon-Proxy

When it comes to providing a single RPC endpoint supported by multiple nodes, a reliable load balancer becomes necessary. While HTTP and WebSocket load balancers like NGINX (previously used in [Axon-devops](https://github.com/axonweb3/axon-devops)) have been widely used, we encountered challenges in implementing advanced JSON-RPC handling, such as rate limiting, caching, and sticky load balancing with them.

To address these limitations, we have built a new JSON-RPC load balancer Axon Proxy. By harnessing Rust's async ecosystem, Axon-Proxy offers robust support for complex use cases. Axon-Proxy will soon become the recommended JSON-RPC load balancer in Axon-devops; it is also a sensible choice for load balancing in other Ethereum JSON-RPC compatible blockchain nodes.

## Implementation

Building a JSON-RPC load balancer in Rust is straightforward using crates like tokio, axum, reqwest and serde_json. Axum is a popular web framework built on top of hyper and tokio. In axum HTTP and WebSocket handlers, we parse the JSON-RPC request, split it if it’s a batch, and handle each request, either by returning a rate-limiting response, cached response or by forwarding it to a backend node.

The tower crate has [load balancing](https://docs.rs/tower/latest/tower/balance/index.html) and [service discovery](https://docs.rs/tower/latest/tower/discover/index.html) functionalities and they are very good inspirations. But the service discovery mechanism doesn’t fit with how we want to implement health checking and failover, so we ended up not using tower load balancing.

### Rate Limiting

Axon-proxy supports rate limiting by IP (or IP and method name). Rate counters are stored in Redis so that they are shared among multiple instances of axon-proxy. Redis is still a lot slower compared to local memory, so it’s planned to add in-process caching of already rate-limited keys, similar to ingress-nginx memchached rate-limiter.

### Caching

Axon-proxy provides optional caching support for some methods, such as `eth_call` and `eth_estimateGas` for now. The cache key is the hash of the current tip block hash, the request method name and canonically serialized parameters. In-process request coalescing is also supported to prevent [dog-piling](https://en.wikipedia.org/wiki/Cache_stampede).

### Filter-ID Sticky Load Balancing

Filter-related methods, e.g. [`eth_getFilterChanges`](https://ethereum.org/en/developers/docs/apis/json-rpc/#eth_getfilterchanges), are stateful. In Axon, filter states are stored in the local memory of each node, so the load balancer must forward filter requests to the corresponding node. In Axon-proxy, a mapping between filter IDs and nodes is stored and filter requests are forwarded accordingly.

### Load Balancing, Health Checking, and Failover

Two load balancing methods are supported: p2c least requests and client IP consistent hashing.

When using [p2c](https://www.eecs.harvard.edu/~michaelm/postscripts/handbook2001.pdf) least requests, two nodes are randomly chosen, and the one with fewer outstanding requests is selected. This should distribute requests uniformly among the backend nodes.

When using client IP consistent hashing, requests from a certain IP are mapped to a certain backend node, as long as the backend nodes don't change. When they
do change, only a minimum fraction of IPs will be mapped to a different node. The specific hashing method we use is [weighed rendezvous hashing](https://www.snia.org/sites/default/files/SDC15_presentations/dist_sys/Jason_Resch_New_Consistent_Hashings_Rev.pdf). Rendezvous hashing has O(n) complexity where n is the number of nodes, but we don’t expect to have more than a few hundred nodes, so it’s fast enough. It’s also easy to implement and plays nicely with health checking.

Active health checking is supported: when enabled axon-proxy will periodically check whether the backend nodes are healthy. Unhealthy nodes are excluded in load balancing. We are also planning to enhance the health checking by also considering block syncing status: nodes that fall behind or haven’t caught up yet probably shouldn’t be selected (inspired by [dshackle](https://github.com/emeraldpay/dshackle)).

In case a node doesn’t respond timely but active health checking hasn’t detected the issue yet, the failover mechanism comes into play: a second node is attempted as long as it is considered safe.  Unlike an HTTP load balancer that only sees POST requests, not knowing which ones are [safe to retry](https://serverfault.com/questions/528653), Axon-proxy can examine the method name and retry all stateless or idempotent requests.

### Config Reloading

Most config options can be reloaded without restarting. Connection pools for backend nodes and Redis are kept across reloads. Load balancing and health checking status are not kept for now, which can be optimized in the future.

### Metrics

Various metrics are recorded in Axon-proxy and exposed in Prometheus text format. Aside from several histograms, the majority of these metrics are atomic variables, in which some are already available for load balancing, so that exposing them as metrics has no extra cost except for encoding them on scraping.

## Performance
Rust and axum both demonstrate remarkable performance (Refer to [this](https://www.techempower.com/benchmarks/#section=data-r21) web framework benchmark ranking). We have also implemented optimizations such as serde_json `&RawValue` to minimize data copies, byte slicing to reduce memory consumption, and vectorized I/O to further enhance efficiency.

To assess its performance, I have run a simple benchmark by calling a minimal ping/pong method through the proxy on a Ryzen 7950x desktop machine. The proxy is running with 12 threads. Rate limiting and request logging are disabled. The performance looks pretty good with the Req/Sec of 202357.66 and the max latency of only 3.94 ms:
```
$ rewrk -t4 -c64 -d 10s  --host http://127.0.0.1:8000 -m post -b '{"jsonrpc":"2.0","id":1,"method":"@ping"}'
Beginning round 1...
Benchmarking 64 connections @ http://127.0.0.1:8000 for 10 second(s)
  Latencies:
    Avg      Stdev    Min      Max
    0.32ms   0.06ms   0.08ms   3.94ms
  Requests:
    Total: 2023663 Req/Sec: 202357.66
  Transfer:
    Total: 413.00 MB Transfer Rate: 41.30 MB/Sec
```

There’s a performance caveat in a specific case: large batch requests take longer to process if the proxy has high network latency to the backend nodes. This is because each request in the batch needs at least an RTT (real-time text) to be handled. It’s currently not planned to change this because large batch requests should probably be limited in production. It’s recommended to deploy the proxy close to the backend nodes if this is a concern. In the future, we may add a more latency-aware load balancing method, which will also help with this issue if the backend nodes have different latency.

Check out Axon-proxy’s [GitHub repo](https://github.com/axonweb3/axon-proxy/) and try it out if you find it interesting or useful. Feel free to reach out to us if you find any problems, or have any questions or suggestions.

