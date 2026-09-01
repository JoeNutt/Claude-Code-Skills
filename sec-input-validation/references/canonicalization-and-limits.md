# Canonicalization, File Handling, and Limits

## Why canonicalization keeps producing vulnerabilities

A validator and a consumer that disagree about what a string *means* is a hole, and every
layer in a modern stack does its own decoding: the proxy, the framework's router, the ORM,
the filesystem, the shell. Each transformation is a chance for the meaning to change after
the check.

The rule follows directly: **reduce to one canonical form, validate that form, and pass the
canonical form onward.** Never validate one representation and hand a different one to the
consumer.

### The many spellings of one file

All of these may reach `/etc/passwd` or escape a directory:

```
../../etc/passwd            ..\..\etc\passwd           ....//....//etc/passwd
%2e%2e%2f                   %252e%252e%252f            ..%c0%af..%c0%af
/var/www/../../etc/passwd   file.txt::$DATA            file.txt.
CON, NUL, AUX (Windows)     symlink -> /etc/passwd     ~/../../etc/passwd
```

Note two classes people miss: reserved device names on Windows, and trailing dots or spaces
which some filesystems strip *after* your check.

### Safe file handling

In order of preference:

1. **Accept an identifier, not a path.** Map it to a location server-side. Traversal becomes
   impossible rather than defended against.
2. **Generate the stored name yourself** — a UUID or content hash. Keep the user's filename
   as a display label, stored as data, never used to build a path.
3. **If a path is unavoidable:** reject separators and traversal sequences, resolve to an
   absolute canonical path, then verify the resolved path is under the permitted root by
   prefix comparison. Ensure the prefix comparison is on a path-segment boundary — `/data/x`
   must not match `/data/xyz`.

For uploads specifically:

- **Verify content, not extension or Content-Type.** Both are attacker-controlled. Parse the
  file with a real parser; if it doesn't parse as the declared type, reject it.
- **Re-encode where you can.** Decoding an image and re-encoding it strips embedded payloads
  and polyglots. Same for PDFs and documents where the tooling allows.
- **Serve from a separate origin**, with a fixed `Content-Type`, `Content-Disposition:
  attachment` where appropriate, and `X-Content-Type-Options: nosniff`. An SVG or HTML file
  served inline from your main origin is stored XSS.
- **Never place uploads inside a directory the server will execute**, and don't preserve an
  extension that maps to a handler.
- **Bound before decoding:** byte size, and image dimensions read from the header before the
  full decode. A 100KB PNG can declare 50,000×50,000 pixels.
- Store with least privilege: not executable, owned by a different account than the web
  process where possible.
- Scan where the file will be redistributed to other users.

## Per-format parser limits

Every parser needs limits set explicitly. Defaults are usually generous.

| Format | Set | Because |
|---|---|---|
| JSON | Max depth, max size, max keys | Deep nesting exhausts the stack |
| XML | **Disable external entities and DTDs**, cap depth and entity expansion | XXE reads local files and reaches internal services; billion laughs exhausts memory |
| YAML | Use the **safe** loader | Default loaders in several languages instantiate arbitrary objects |
| Archives | Decompressed size, entry count, path of each entry | Zip bombs; `../` in entry names ("zip slip") |
| Images | Dimensions before decode, byte cap, decode timeout | Decompression bombs |
| CSV | Field and row caps; escape leading `= + - @` on export | Formula injection when opened in a spreadsheet |
| Regex | Linear-time engine, or a timeout and input cap | Catastrophic backtracking |

**Never deserialize untrusted data into arbitrary types.** Language-native serialization
(pickle, Java serialization, PHP unserialize, .NET BinaryFormatter, unsafe YAML) is remote
code execution when the input is attacker-controlled. Use a data-only format, parse into a
known schema, and allowlist types explicitly if the framework supports it.

## Encoding traps

- **Decode exactly once**, then reject anything still encoded. Looping "until clean" makes
  your decoder more capable than the consumer's and creates the mismatch you were avoiding.
- **Normalize Unicode before validating and before comparing.** Apply the same form
  everywhere; a mismatch between storage-time and comparison-time forms is a bypass.
- **Reject overlong and invalid UTF-8** rather than substituting replacement characters —
  substitution changes the string after validation.
- **Null bytes** truncate strings in C-based layers. Reject them in any value reaching a
  filesystem, database, or native library.
- **Case-fold with the invariant/ordinal comparer** for security decisions. Locale-aware
  folding varies by culture and has produced real authentication bypasses.
- **Strip or reject bidirectional control characters** in identifiers and anything displayed;
  they let source and text render differently from what they contain.
- Homoglyph-check identity-bearing strings — usernames, org names, domains — or restrict
  them to a single script.

## Resource limits worth setting by default

Set these in the framework, once, rather than per endpoint:

- Max request body, header count and size, URL length.
- Read, write, and total request timeouts; and a timeout on every outbound call.
- Connection and thread pool caps; queue depth with a bounded queue.
- Per-IP, per-user and per-tenant rate limits, plus a global ceiling.
- Max response size where you fetch from elsewhere — an SSRF target can reply with terabytes.
- Query row limits and statement timeouts at the database.
- A hard cap on pagination size, independent of what the client requests.

Two rules make these effective. Enforce the limit **before** doing the work — checking after
decoding a zip bomb is checking after the damage. And **fail closed with a clear error**
rather than truncating, because a silently truncated input is an input you never validated.

## Where validation belongs

One barricade at the trust boundary, converting to domain types. Not a check scattered
through every layer. If interior code feels the need to re-validate, the boundary isn't
trusted — fix the boundary rather than adding a second one, since two validators with
different rules mean the looser one is your real contract.
