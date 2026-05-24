---
title: Should we start shaming developers who don't use isolation?
slug: shame-devs-without-isolation
date: 2026-05-24
image: cover_extended.jpg
tags:
    - containers
---

## Intro

It seems we are seeing supply-chain attacks [every](https://safedep.io/mini-shai-hulud-strikes-again-314-npm-packages-compromised/) [other](https://opensourcemalware.com/npm/@bitwarden/cli) [day](https://www.stepsecurity.io/blog/nx-console-vs-code-extension-compromised) now. There are two main reasons for this:

1. Projects have **too many dependencies**. JS projects can easily reach 1000+ transitive dependencies.
2. Projects usually run **without any isolation** from the rest of the developer’s computer, allowing any attack to easily propagate.

Much has been written about the former. It may require the industry to adopt a different mindset, which is always hard. Instead, I want to talk about the latter, which mostly requires technological changes. By isolating projects from each other and from the host computer, you can drastically lower the “blast radius” of an infected dependency.

There are various tools available for it. [I made one myself](https://evertheylen.eu/p/probox-intro/) based on podman. Maybe you prefer stricter protection through the use of [virtual machines](https://www.qubes-os.org/) instead. Maybe you prefer running dev environments on [someone else’s computer](https://sprites.dev/) altogether. But at least there should be *some* barrier between `npm install` and `cat ~/.ssh/id_rsa`, no?

I would argue that we should start holding developers accountable. Some level of isolation should be expected. Failing to do so should earn them a similar amount of flak as those still writing SQL queries with string interpolation, or storing plaintext passwords. As an industry, we should do better!

## Symmetric keys are problematic

Okay, that was a spicy take. In practice there are still some hindrances, and I can see why some people don't bother with isolation. I’ve been using my own tool for a while now and I encounter plenty of problems, mostly when protecting secret keys.

I mentioned protecting your SSH keys. If your development environment doesn’t have access to them, how do you use `git` or `ssh`? In my own tool, I make it work by forwarding an `ssh-agent` socket and combining it with a tool like `ksshaskpass`. Crucially, SSH uses asymmetric keys so a potentially infected container never actually gets access to the full key.

Sadly most tools and APIs use symmetric keys or passwords. It is so common we skip the word “symmetric” altogether. Tools like `flyctl` (Fly.io) or `doctl` (DigitalOcean) all store some kind of symmetric secret in your home folder. It is very hard to protect those secrets without a deeper integration with the software trying to use them.

### What about HTTP APIs?

Both [Deno](https://deno.com/deploy/sandbox) and [Fly](https://github.com/superfly/tokenizer) have projects that “intercept” your HTTP API calls and replace placeholder secrets with the real (still symmetric) ones. I consider this one example of a “deeper integration”, as forcing the use of an HTTP proxy is not always trivial.

Personally I feel like asymmetric keys are a more elegant and universal solution. Sadly client-side certificates for HTTPS are not that easy to use (for example, standard `fetch` does not support it). I’ve only ever used [one API](https://benerail.com/solutions/moove_api/) that used client-side certificates, but it may be more common in industries like banking or healthcare.

## Call to action

**To individual developers:** please consider using *some* form of isolation so that the impact of installing or using malicious dependencies is limited. (Note: default settings for Docker are usually not sufficient!)

**To companies:** please consider using a system with asymmetric keys instead of symmetric API keys. Alternatively, give instructions on how to use your API with a project like Fly’s [tokenizer](https://github.com/superfly/tokenizer) or some system using short-lived tokens. Just don't assume your software will run in an environment that has access to symmetric keys.

**To you, dear reader:** I’d be very happy to hear your thoughts about this. Feel free to write a comment on Hacker News or reach out to me [via email](mailto:evertheylen@gmail.com).

