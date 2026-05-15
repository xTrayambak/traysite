+++
title = "Writing a new package manager for Nim"
description = "An introduction to the Neo package manager"
date = 2025-10-06
+++

I've been working on my JavaScript engine, [Bali](https://github.com/ferus-web/bali) for over a year at this point.

Through all of its releases, the codebase has slowly grone from a thousand lines of Nim to over ~14K lines of Nim. However, that isn't the main point here.

Ever since Bali became a fairly "heavyweight" Nim project, the limitations of Nim's tooling began to show up.

# Here comes the pain
## Some Context
I'm running NixOS. As such, I use Nix shells all the time to ensure that I have all the dependencies to compile my projects.

## Compiling Balde
Balde ("**BAL**i **DE**bugger") is the main command-line interface provided by Bali. It acts as a script runner, read-eval-print-loop (REPL) session runner, profiling tool and debugging instrumentation tool for various subsystems in the engine. It is about ~500 lines of Nim.

When I work on Bali, I use Balde to test all the changes I make, unless they're very specific (in which case, I write a Nim-based test case in `tests/`).

Here's where the problem comes in: Nimble is slow. **REALLY SLOW.** For context, here's how much time Nimble takes to compile Balde after I enter a fresh Nix shell and run `nimble build balde`.

![Yikes](https://files.catbox.moe/c70tmz.jpg)

To put it lightly, that's not very bearable. In all honesty, I'm assuming using a Nix shell just exacerbates the problem, because Nimble was not designed with Nix's reproducability in mind. As such, its caches get invalidated every time you exit and re-enter a Nix shell.

### Is Nim a slow language?
No, not really. Well-written Nim can compete with C, C++ and Rust code. It's just that Nimble spends a painful amount of time doing the most inexplicable things, which makes it unacceptably slow.

If `strace` is attached onto Nimble, we can observe that it opens a large number of files, reads their contents, closes them, and so on.
![Nimble committing tomfooleries](https://files.catbox.moe/n072jz.jpg)

# Neo, a newer approach
![Neo goes vroom vroom](https://files.catbox.moe/2z1qei.jpg)

I started work on [Neo](https://github.com/xTrayambak/neo) in February of this year, but I've not been able to dedicate much time to it. Recently, due to my vexations with Nimble, I decided to pick it up again and I've managed to make a surprising amount of progress. Oh, and it's much faster.

Now, I would say that Neo is ready for very simple Nim projects to use it. It can interoperate with Nimble-based packages (to a reasonable extent) and also relies on the Nimble package index.

It's obviously much leaner than Nimble (25K LoC vs ~1.8K LoC) and obviously much faster as well. It isn't as advanced as Nimble is, but still works fairly well for my own use cases.

I won't throw any rocks at Nimble, mostly because I'm sitting in a glass house myself. **Neo is not replacing Nimble, yet.** But I do wish to make it a proper competitor to it, and maybe even get Nimble replaced by it one day. Nimble (or as it was called back in the day, Babel) is a relatively old codebase, and it shows.

Making any fixes to it or adding new features is a total headache, from personal experience. It's like adding more pieces to a jenga tower except the base is made with ice cubes. Neo is much nicer to extend, and that's objectively true. (because it's a new codebase!)

The main thing Neo's missing at this point is a good dependency resolver. Right now, it uses a naive rules-based solver that _mostly_ works (emphasis on mostly), but it'd be nice to replace it with something like Nimble's SAT solver, preferrably without making it as slow as Nimble.

## I want to try out Neo
Whew, that's nice. Unfortunately, there's no `neo migrate` command as of yet, but you can simply run `neo init` to open a nice, Nimble-like wizard to setup a project. Afterwards, the flow is fairly straightforward:

- `neo add <package / URL / forge alias>` to add a package
- `neo install <optional/path/to/package>` to install a package's binaries or libraries
- `neo build` to build a package's binaries
- `neo search <name>` to find a package
- `neo info <name>` to get extended information on a package

![neo info](https://files.catbox.moe/le2lqt.jpg)

A lot is remaining, and a lot will be done in the next few months. Hopefully, we can get something better than Nimble soon. :^)
