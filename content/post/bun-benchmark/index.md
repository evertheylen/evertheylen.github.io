---
title: "Node vs Bun: which is the fastest backend?"
slug: node-vs-bun
date: 2024-10-04
image: cover2.webp
tags:
    - node
    - bun
    - benchmark
---

In this article I'll be comparing the performance of [Bun](https://bun.sh/) vs [Node](https://nodejs.org/). In particular (as Bun is many things), I will focus on their performance as the runtime for your server-side JavaScript code. With Node being the default choice for many, Bun has to prove it is worthy of your attention. It makes strong claims:

- There’s a benchmark right on the homepage, claiming almost *5 times* more requests per second than Node! [^1].
- In the [documentation](https://bun.sh/docs/cli/run#performance), they claim that *“In most cases, the startup and running performance is faster than V8, the engine used by Node.js and Chromium-based browsers.”*

[^1]: I believe there is a chance Bun is kinda cheating here, with a different version of React than what Node uses. See [this benchmark](https://medium.com/deno-the-complete-reference/node-js-vs-deno-vs-bun-server-side-rendering-performance-comparison-f80a5abc766f).

Part of what makes this an interesting comparison is that Node uses the [V8 engine](https://v8.dev/) (which also powers Chromium-based browsers), while Bun uses [JavascriptCore](https://developer.apple.com/documentation/javascriptcore) (which powers Safari).



## Benchmark setup

I will be running the benchmarks from this excellent GitHub project: [github.com/nDmitry/web-benchmarks](https://github.com/nDmitry/web-benchmarks). The README says:

> […] on each request it simply fetches a 100 fake users from the \[Postgres\] database, creates a class instance/structure for each row converting a datetime object to an ISO string and encrypting one of the fields with Caesar cypher, serializes resulting array to JSON and responds with this payload.
>

I think it’s a reasonable setup for benchmarks like this. The repository also has instructions for running Python and Go benchmarks, if you’re interested in that. (Spoiler: Go is about 2x faster than Node, and Node is *maybe* 10% faster than async CPython on a good day.)

I adjusted a few things which you can find in [my fork](https://github.com/evertheylen/web-benchmarks). Each benchmark will run with a fixed amount of processes (eg. node-8 will be using 8 processes). I'm using `hey` to do 50000 requests. All benchmarks run on an AMD 5900X processor (12 physical cores, 24 threads) with 32GB of memory.

<details>
<summary>Software versions</summary>

    - OS: Arch Linux, kernel 6.9.7-arch1-1
    - Node: v22.9.0
    - Bun: 1.0.28
    - hey: 0.1.4-6
</details>



## Results

In short: Node has a very slight performance advantage, but it is almost negligible.

{{< figure src="combined_100.svg" title="Results for 100 parallel connections">}}

I tried many variations of the test, to see if I could find a scenario where the differences would be larger. In particular, I tested different amounts of parallel connections. The chart above is for 100 connections in parallel (`hey -c 100`), but the same pattern holds for 200, 800, even up to 2000 connections. At such high connection counts, RPS actually drops for both.



### Running without Postgres

A rather drastic thing you could do is remove the Postgres connection from the benchmark. I did this by hardcoding the result you’d usually get from Postgres. This makes it almost a “hello world” benchmark, but it still has the Caesar cipher and some `JSON.stringify` going on. The result:

{{< figure src="combined_200_nopg.svg" title="Results for 200 parallel connections, without Postgres">}}

Here, Bun clearly takes the win. In fact, I tried the Go server and it hits (only) about 100k RPS. Bun beats Go! However, I’d argue this is almost meaningless. If your server is this simple, run it in the client.



## Discussion

It's a boring result, but an expected one: in real-world usecases, your servers will likely **not** get any performance boost out of Bun. Bun's extraordinary claims only hold up for hello-world-style benchmarks.


### Multiprocessing differences

To run a Node server with multiple processes, I used the `cluster` package, but this is [not implemented yet in Bun](https://bun.sh/guides/http/cluster). Instead I have to `spawn` different Bun processes that should each run `serve({reusePort: true, ...})`, as described by the [excellent Bun guides](https://bun.sh/guides/http/cluster).

The end result works similarly to Node’s `cluster` module, but Node’s solution is a lot nicer to use. It’s easier to check for `cluster.isPrimary` and use `cluster.fork()` than to maintain multiple entrypoints. Node also [claims](https://nodejs.org/api/cluster.html#how-it-works) “*some built-in smarts to avoid overloading a worker process*”, which Bun may lack. On the other hand, inter-process communication seems easy on both platforms should you need it.


### Impact of frameworks

I chose not to use any server frameworks/libraries to focus on the runtime itself. The original repository did test a few popular Node frameworks, which shows a sizable performance impact. From fast to slow, it goes koa > express > hapi. In any case, not using a framework is always the fastest.

In my own (separate) tests, using a library like hapi did make Node substantially slower than Bun. Bun’s native `serve` API feels more modern and functional, so maybe you won't need a framework at all. If you have to choose between Node+hapi and just Bun, Bun will win. But as a more realistic example, Node+koa vs Bun+Hono is (probably) going to be quite close.


### Startup time

Bun also claims a much faster startup time. I have no reason to doubt that:

```
$ hyperfine\
  "node -e 'require(\"http\").createServer(() => {}).listen(8000).close()'"\
  "bun -e '(await import(\"bun\")).serve({fetch: () => {}, port: 8000, development: false}).stop()'"

Benchmark 1: node -e 'require("http").createServer(() => {}).listen(8000).close()'
  Time (mean ± σ):      18.6 ms ±   1.7 ms    [User: 12.7 ms, System: 6.1 ms]
  Range (min … max):    17.4 ms …  29.8 ms    142 runs

Benchmark 2: bun -e '(await import("bun")).serve({fetch: () => {}, port: 8000, development: false}).stop()'
  Time (mean ± σ):      11.7 ms ±   0.5 ms    [User: 4.3 ms, System: 9.5 ms]
  Range (min … max):    11.1 ms …  14.5 ms    253 runs

Summary
  bun -e '(await import("bun")).serve({fetch: () => {}, port: 8000, development: false}).stop()' ran
    1.59 ± 0.15 times faster than node -e 'require("http").createServer(() => {}).listen(8000).close()'
```

The extra 7ms is nice for tooling, but for servers I honestly don’t think it matters much.


### Where is Deno?

I like that Deno tries something new with an [interesting permission system](https://docs.deno.com/runtime/fundamentals/security/). But from a performance perspective, they use V8 just like Node, so I thought it was less interesting. Feel free to [send a PR](https://github.com/evertheylen/web-benchmarks) with a Deno server and I’ll include the results.

