---
name: sec-input-validation
description: This skill should be used when handling data from any external source - request parameters, form fields, headers, cookies, uploaded files, webhooks, imported data, filenames and paths, or API payloads. It covers allowlist validation, canonicalization order, path traversal, Unicode and encoding attacks, and resource limits that prevent denial of service. Trigger phrases include "validate this input", "sanitize", "file upload", "user-supplied", "parse this", "handle this request", "filename", "file path", "untrusted data", "rate limit", "request size", "is this safe to accept".
---

# Input Validation

**All input is evil until proven otherwise.** Not just form fields: headers, cookies, query
strings, path segments, uploaded file contents *and* names, webhook payloads, imported files,
API responses from services you don't control, environment variables, database rows written
by something else, and anything sent by your own client code.

Client-side validation is a usability feature. The attacker doesn't use your client.

## Allowlist, never denylist

Define what is **valid** and reject everything else. Do not enumerate what is dangerous — the
list of bad inputs is unbounded, encodings multiply it, and you will be outnumbered.

```
BAD:   reject if input contains "<script"     # ..., <SCRIPT, <scr\0ipt, %3Cscript, <svg onload
GOOD:  accept only /^[A-Za-z0-9 '-]{1,60}$/   # everything else rejected
```

For each field, decide and enforce: type, length or range, format, and character set. Then
**reject** — do not "clean". Stripping bad characters produces new inputs you didn't
anticipate (removing `../` from `....//` yields `../`) and hides attacks from your logs.
Sanitize only where the data must be rendered rather than refused, and do it by encoding for
the destination, not by deleting characters. See `sec-injection-defense`.

## Canonicalize first, then validate

The single most common validation bypass is checking a string before reducing it to its one
true form. The same resource has endlessly many spellings.

**Order: decode → canonicalize → validate → use.** Never validate a form you have not
canonicalized, and never re-decode after validating.

Decode exactly once, and reject input that is still encoded afterwards rather than looping —
`%252e%252e%252f` decodes to `%2e%2e%2f` decodes to `../`, and a validator that decodes once
while the consumer decodes twice is a hole.

### Paths

Path traversal is the classic case. `../`, `..\`, absolute paths, symlinks, UNC paths,
`%2e%2e%2f`, Unicode-encoded separators, trailing dots and spaces, and case differences on
case-insensitive filesystems all reach the same file.

**Never build a path from user input.** Preferred, in order:

1. Don't accept a path at all — accept an id, look the path up server-side.
2. Generate the filename yourself; store the user's name as a display label only.
3. If you must: resolve to an absolute canonical path, then verify it is *inside* the
   permitted directory by prefix — after resolution, never before.

Verify by comparing resolved paths, not by string matching the input, and remember a
symlink inside the directory can still point outside it.

### URLs and hostnames

Same rule, and the stakes are higher because a validated-looking URL fetched server-side is
SSRF. Parse with a real parser, then check the resolved host against an allowlist. Watch for
credentials in the authority (`http://allowed@evil/`), alternate IP encodings, redirects,
DNS rebinding, and cloud metadata addresses. Prefer an allowlist of destinations over any
attempt to block internal ranges.

## Unicode and encoding

Text is not bytes, and equality is not obvious.

- **Normalize before comparing** (NFC/NFKC as appropriate). Distinct code point sequences
  render identically; without normalization, two "equal" strings compare unequal — and
  worse, an unequal pair compares equal after some later normalization you didn't control.
- **Normalize before validating, not after.** A normalization step applied downstream can
  reintroduce a character your validator rejected.
- Reject **overlong encodings** and invalid UTF-8 outright rather than repairing them.
- Beware **homoglyphs and bidirectional overrides** in anything shown to a human or used for
  identity — usernames, domains, and filenames especially.
- Measure length in the unit that matters: bytes for a buffer, code points for a limit,
  grapheme clusters for anything user-facing.
- Case-folding is locale-dependent. Use invariant/ordinal comparison for security decisions;
  a locale-aware `toLowerCase` has changed the meaning of identifiers before.

## Bound every resource

Unbounded input is a denial of service waiting for a bad day. Every one of these needs an
explicit limit, enforced **before** the expensive work:

- Request body size, header size and count, URL and field length.
- Array and collection lengths; pagination page size (cap it — never trust the client's).
- Upload size, and for archives the **decompressed** size and entry count (zip bombs).
- Image dimensions checked before decoding, not after.
- Nesting depth for JSON, XML, and any recursive parser.
- Timeouts on every network call, parse, and query. No unbounded wait, anywhere.
- Concurrency and rate limits per client, per tenant, and globally.

Watch for algorithmic complexity attacks: catastrophic regex backtracking on crafted input,
hash collisions, and accidental O(n²) over an attacker-sized collection. A regex over
untrusted input should be linear-time or time-bounded.

**Fail closed.** When a limit is hit, reject with a clear error. Don't truncate silently —
truncation turns a rejected input into an accepted, different one.

## Validate at the boundary, once

This is the barricade from `cc2-defensive-design`, and the two skills agree: validate
completely at the trust boundary, convert to domain types there, and let interior code trust
its inputs. Scattered re-validation is not defense in depth — it is two validators that will
eventually disagree, with the looser one silently defining the contract.

Detail on canonicalization traps, file upload handling, and per-format parser limits is in
`references/canonicalization-and-limits.md`.
