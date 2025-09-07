# Global states are evil and you probably shouldn't use them
I've been working on my JavaScript engine, [Bali](https://github.com/ferus-web/bali) for just over a year now. At the start of its life, the memory management was very questionable: I would allocate Nim-GC-allocated `MAtom`(s) (an atom is what Bali uses to refer to a JavaScript value), then store them in a `Table[uint, MAtom]`.

Suffice to say, Bali no longer does that eldritch witchcraft. It now just uses a vector of Boehm-allocated `JSValue` (which are `MAtom *`) that it calls "the valuespace". However, this article isn't about Bali's memory allocation - it's about the horrible globals-abuse Bali used to carry out.

# Bali and the Bane of Globals
Bali, prior to version 0.8.0, used to use a thread-local `HeapManager` (an API used by Bali to opaquely allocate memory either on the pre-allocated, fast-path 8MB bump allocator or slow-path GC) to allocate memory.

I'm assuming that's probably set off some alarm bells already. This essentially prevents Bali from:
- Letting two `Runtime`(s) co-exist on the same thread with guarantees of isolation
- Get initialized on thread A, then get transferred to thread B and executed over there

and it was horrible design anyways. So, in v0.8.0, I ended up refactoring the entire codebase to make all allocations explicitly specify which runtime instance's heap context they wish to allocate on. So, allocating a float `13.37` went from:

```nim
var runtime = ... # Initialize a `Runtime`

floating(13.37) # returns a `JSValue` with kind `Float`
```

to:
```nim
var runtime = ... # Initialize a `Runtime` here using `newRuntime()`

# ... code, you must also call `Runtime.run()` so that the heap manager for it is init'd
floating(runtime, 13.37) # Now, you must explicitly specify which instance's heap context you wish to allocate on
```

And that's pretty much it. I also ended up migrating the `console.log()` "delegate" system to be `Runtime`-bound as well as the exception handler callback that's called when the VM throws an exception.

# Closing Notes
With such a _relatively_ minor change, Bali's suddenly a lot more maintainable and sane. \
If you ever write something you aim to maintain for more than 5 minutes — **PLEASE, DO NOT USE GLOBAL VARIABLES!** (unless you're doing kernel development or something...)
