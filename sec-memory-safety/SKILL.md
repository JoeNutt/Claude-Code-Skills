---
name: sec-memory-safety
description: This skill should be used when writing or reviewing code in a memory-unsafe language - C, C++, Objective-C, or unsafe blocks and foreign-function interfaces in Rust, Go, C#, Python, Java and similar. It covers buffer overruns, integer overflow leading to undersized allocations, use-after-free, format string flaws, and the bounds and lifetime discipline that prevents them. Trigger phrases include "buffer", "memcpy", "malloc", "pointer", "unsafe block", "FFI", "cgo", "JNI", "C extension", "strcpy", "array bounds", "segfault", "memory corruption", "parsing binary data".
---

# Memory Safety

Memory-safety defects are the highest-severity class in the languages that permit them,
because they usually mean arbitrary code execution rather than data disclosure. They remain
a leading source of critical CVEs.

## Scope

This applies wherever memory is manually managed:

- C, C++, Objective-C, assembly.
- `unsafe` blocks in Rust; `unsafe` in C#; `unsafe`/`uintptr` in Go.
- Any foreign-function interface: cgo, JNI, P/Invoke, ctypes, native Node addons, Python C
  extensions.
- Parsers handling attacker-controlled binary data — the highest-risk code in any codebase.

**In a memory-safe language with no unsafe code or FFI, this skill does not apply.** Say so
rather than inventing concerns; the risks there are the injection and authorization classes
covered by the other `sec-*` skills.

## The first question: does this need to be unsafe?

Most memory-safety bugs are in code that had no need to manage memory manually.

- Use the language's safe abstractions — bounded strings, vectors, slices, spans, smart
  pointers — over raw pointers and manual allocation.
- Use a memory-safe language for new components handling untrusted input, especially parsers.
  This is the single highest-value decision available here.
- Where `unsafe` is genuinely required, make the block as small as possible, wrap it in a
  safe interface, and document the invariants the caller must uphold for it to stay sound.

## Buffer overruns

Writing past the end of an allocation corrupts adjacent memory — and in the classic case,
the return address.

- **Never use unbounded copies.** The `strcpy`/`strcat`/`sprintf`/`gets` family has no
  concept of a destination size. Use the length-bounded equivalents, and check their return
  values — several truncate silently, which is a different bug rather than a fix.
- **Track sizes with buffers**, and prefer types that carry their own length over a pointer
  and a separately-passed size that can drift apart.
- **Off-by-one is the common case.** Account for the null terminator; be exact about whether
  a size is bytes, elements, or characters, and whether a length includes the terminator.
- **Validate index and length before every access**, including loop bounds computed from
  input. A length field in a file or packet is attacker-controlled.
- **Check every allocation for failure** before using the result.
- Stack buffers with attacker-controlled sizes are the highest-risk form; prefer heap
  allocation with an explicit cap, and never use variable-length stack allocation on
  untrusted sizes.

## Integer issues

Integer bugs are how buffer overruns are usually reached, which makes them worth their own
attention.

```
len = read_u32(input)          # attacker controls this
buf = malloc(len + 1)          # len = 0xFFFFFFFF wraps to 0 -> tiny allocation
memcpy(buf, src, len)          # writes 4GB into it
```

- **Check for overflow before arithmetic** used in a size or index — especially `+ 1`,
  multiplication for element counts, and any sum of two input-derived values. Use the
  platform's checked-arithmetic helpers.
- **Signed/unsigned confusion:** a negative length compared as signed passes `len < max`, then
  converts to an enormous unsigned value.
- **Truncation on narrowing** (`size_t` to `int`, 64-bit to 32-bit) discards the high bits and
  the check you performed on the wider type.
- Be explicit about integer widths when parsing binary formats; don't rely on the platform's
  `int`.

## Lifetime and pointers

- **Use-after-free and double-free** — set pointers to null after freeing, or better, use
  ownership types that make the mistake unrepresentable. Watch for a pointer retained by a
  second structure after the first frees it.
- **Uninitialized memory** — read before write discloses whatever was there, including keys.
  Initialize on allocation.
- **Zero secrets before freeing**, using a function the compiler is not permitted to optimize
  away.
- **Null checks** on every allocation and every pointer returned from an API that can fail.
- **Match allocator and deallocator** — `malloc`/`free`, `new`/`delete`, `new[]`/`delete[]`.
- Beware pointers into buffers that may be reallocated; a container's growth invalidates
  them.

## Format strings and other classics

- **Never pass user data as a format string.** `printf(user_input)` reads and writes memory
  through `%s` and `%n`. Always `printf("%s", user_input)`. Same for logging APIs.
- **Bound every loop** driven by input; never trust an embedded length or a terminator that
  might be absent.
- **Check return values** of every function that can fail — ignored failures leave
  uninitialized or stale data in play.
- Beware time-of-check/time-of-use races on files. Open first, then operate on the handle.

## Tooling

Reasoning is not enough here; use the tools, because they find what review misses:

- **Compiler warnings at maximum, treated as errors.** Enable the platform's hardening flags:
  stack protectors, FORTIFY, ASLR/PIE, non-executable stack, control-flow integrity.
- **Sanitizers in CI** — address, undefined-behavior, memory, and thread sanitizers.
- **Static analysis** for the language.
- **Fuzz every parser that touches untrusted input.** This is the highest-yield security
  testing available for memory-unsafe code, and it finds cases no reviewer would construct.
- Test at boundaries: zero length, maximum length, off-by-one either side, and integer
  extremes.

## Reviewing

1. Every copy, concatenation, and index: is the destination size known and checked?
2. Every arithmetic operation producing a size or index: can it overflow, wrap, or truncate?
3. Every allocation: checked for failure, and matched with the right deallocator?
4. Every input-derived length: validated against the actual buffer before use?
5. Every pointer: can it outlive what it points to?
6. Every format string: is it a literal?
7. Every `unsafe` block: is it minimal, wrapped, and are its invariants documented and
   actually upheld by all callers?
