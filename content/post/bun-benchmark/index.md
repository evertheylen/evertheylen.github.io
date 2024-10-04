---
title: "Bun vs Node: which is the fastest backend?"
date: 2024-10-04
image: cover2.webp
tags:
    - node
    - bun
    - benchmark
---

In this article I'll be comparing the performance of [Bun](https://bun.sh/) vs [Node](https://nodejs.org/). In particular (as bun is many things), I will focus on their performance as the runtime for your server-side JavaScript code. With Node being the default choice for many, Bun has to prove it is worthy of your attention. It makes strong claims:

- There’s a benchmark right on the homepage, claiming almost *5 times* more requests per second than node! [^1].
- In the [documentation](https://bun.sh/docs/cli/run#performance), they claim that *“In most cases, the startup and running performance is faster than V8, the engine used by Node.js and Chromium-based browsers.”*

[^1]: I believe there is a chance Bun is kinda cheating here, with a different version of React than what Node uses. See [this benchmark](https://medium.com/deno-the-complete-reference/node-js-vs-deno-vs-bun-server-side-rendering-performance-comparison-f80a5abc766f).

Part of what makes this an interesting comparison is that Node uses the [V8 engine](https://v8.dev/) (which also powers Chromium-based browsers), while Bun uses [JavascriptCore](https://developer.apple.com/documentation/javascriptcore) (which powers Safari).



## Benchmark setup

I will be running the benchmarks from this excellent GitHub project: [github.com/nDmitry/web-benchmarks](https://github.com/nDmitry/web-benchmarks). The README says:

> […] on each request it simply fetches a 100 fake users from the database, creates a class instance/structure for each row converting a datetime object to an ISO string and encrypting one of the fields with Caesar cypher, serializes resulting array to JSON and responds with this payload.
>

I think it’s a reasonable setup for benchmarks like this. The repository also has instructions for running Python and Go benchmarks, if you’re interested in that. (Spoiler: Go is about 2x faster than Node, and Node is *maybe* 10% faster than async CPython on a good day.)

I adjusted a few things which you can find in [my fork](https://github.com/evertheylen/web-benchmarks). In particular, I:

- used `hey` instead of `ab` for their better CSV export,
- adjusted the number of processes to something I could set explicitly,
- added python scripts to run the tests and collect the results,
- configured `hey` to run 50000 requests.

I am running all benchmarks on a AMD 5900X processor (12 physical cores, 24 threads) with 32GB of memory. Software versions:

- OS: Arch Linux, kernel 6.9.7-arch1-1
- Node: v22.9.0
- Bun: 1.0.28
- hey: 0.1.4-6



## Results

In short: Node has a very slight performance advantage, but it is almost negligible.

{{< figure src="combined_100.svg" title="Results for 100 parallel connections">}}

I tried many variations of the test, to see if I could find a scenario where the differences would be larger. If anything, Node seems to handle high concurrency better. The number after the dash denotes the amount of processes, so node-8 will be using 8 processes. The chart above is for 100 connections in parallel (`hey -c 100`). Here it is for 2000 connections:

{{< figure src="combined_2000.svg" title="Results for 2000 parallel connections">}}

Note that both Bun and Node lose performance, but Node does it a bit more gracefully. On the other hand, node-1 has some crazy outliers when it comes to response time.

A rather drastic thing you could do is remove the postgres connection from the benchmark. I did this by hardcoding the result you’d usually get from postgres. This makes it almost a “hello world” benchmark, but it still has the caesar cipher and some `JSON.stringify` going on. The result:

{{< figure src="combined_200_nopg.svg" title="Results for 200 parallel connections, without postgres">}}

Here, Bun clearly takes the win. In fact, I tried the Go server and it hits (only) about 100k RPS. Bun beats Go! However, I’d argue this is almost meaningless. If your server is this simple, run it in the client.


### Conclusion

- In real-world usecases, your servers will likely **not** get any performance boost out of Bun. At scale, they may even lose some performance, but you should do your own benchmarks.
- In hello-world-style benchmarks, Bun clearly wins.



## Caveats

### Multiple processes

To run a Node server with multiple processes, I used the `cluster` package, but this is [not implemented yet in Bun](https://bun.sh/guides/http/cluster). Instead we have to `spawn` different Bun processes that should each run `serve({reusePort: true, ...})`, as described by the [excellent Bun guides](https://bun.sh/guides/http/cluster).

The end result works similarly to Node’s `cluster` module, but Node’s solution is a lot nicer. It’s easier to check for `cluster.isPrimary` and use `cluster.fork()` than to maintain multiple entrypoints. Node also [claims](https://nodejs.org/api/cluster.html#how-it-works) “*some built-in smarts to avoid overloading a worker process*”, which Bun may lack. On the other hand, inter-process communication seems easy on both platforms should you need it.

### Frameworks

I chose not to use any server frameworks/libraries to focus on the runtime itself. The original repository did test a few popular Node frameworks, which shows a sizable performance impact. From fast to slow, it goes koa > express > hapi. In any case, not using any framework is always the fastest.

In my own (separate) tests, using a library like hapi did make Node substantially slower than Bun. Bun’s native `serve` API feels a bit more modern and functional. If you have to choose between Node+hapi and just Bun, Bun will win. But as a more realistic example, Node+koa vs Bun+Hono is (probably) going to be quite close.

### Startup time

Bun also claims a much faster startup time. I have no reason to doubt that. It’s nice for tooling, but for servers I don’t think it matters much. Please [let me know](mailto:evertheylen@gmail.com) if you have an application that sees sizable traffic but still scales to zero.

### Where is Deno?

I like that Deno tries something new with an [interesting permission system](https://docs.deno.com/runtime/fundamentals/security/). But from a performance perspective, they use V8 just like Node, so I thought it was less interesting. Feel free to [send a PR](https://github.com/evertheylen/web-benchmarks) with a Deno server and I’ll include the results.

