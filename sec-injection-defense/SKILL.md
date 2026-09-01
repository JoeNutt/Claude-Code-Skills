---
name: sec-injection-defense
description: This skill should be used when user-controlled data is placed into a query, command, document, template, URL, or any other interpreted context - SQL and NoSQL queries, shell commands, HTML output, LDAP filters, XML, file paths, redirects, or server-side fetches. It covers parameterization, context-correct output encoding, and the modern injection classes including SSRF, XXE, template injection and unsafe deserialization. Trigger phrases include "SQL query", "database query", "run this command", "shell out", "render this", "display user input", "build this URL", "fetch this URL", "template", "escape this", "XSS", "SQL injection", "concatenate".
---

# Injection Defense

Injection happens when data is placed somewhere an **interpreter** reads it, and the
interpreter can't tell your structure from the attacker's. The fix is never better escaping
in a string concatenation — it is keeping data out of the code channel entirely.

**The rule: separate code from data structurally. Where that is impossible, encode for the
exact destination context.**

## Structural separation, by interpreter

| Interpreter | Do | Never |
|---|---|---|
| SQL / NoSQL | Parameterized queries, bound variables | String concatenation or interpolation, however "escaped" |
| Shell / OS | Call the program directly with an argument **array**; no shell | Build a command string and pass it to a shell |
| HTML / DOM | Contextual output encoding; safe DOM APIs (`textContent`) | `innerHTML` with user data; string-built markup |
| LDAP | Parameterized filter API, or strict allowlist + filter escaping | Concatenating into a filter |
| XML | Disable DTDs and external entities; build with a DOM API | String-built XML; default parser settings |
| Templates | Pass data as **context**; auto-escaping on | User data in the template *source* |
| File paths | Ids mapped server-side; generated names | Paths built from user input |
| Redirects | Allowlist of permitted destinations | Redirecting to a user-supplied URL |
| Server-side fetch | Allowlist of hosts | Fetching a user-supplied URL |
| Deserialization | Data-only formats; schema parsing | Native deserialization of untrusted bytes |

If a library offers no parameterized form, that is a strong signal to find a different
library before writing your own escaping.

### SQL: what parameterization does and does not cover

Bound parameters protect **values**. They cannot parameterize identifiers — table names,
column names, `ORDER BY` targets, `ASC`/`DESC`, `LIMIT` in some drivers. For those, map the
user's input through an **allowlist** to a known-safe literal:

```
# The user picks a key; your code chooses the SQL. Their string never reaches the query.
SORT_COLUMNS = { "date": "created_at", "name": "display_name" }
column = SORT_COLUMNS.get(requested) or raise BadRequest
```

Also note: an ORM is not automatically safe. Raw-query escape hatches, `where` clauses built
from strings, and `LIKE` patterns with unescaped wildcards all reintroduce injection. NoSQL
is injectable too — a JSON body where a string was expected can smuggle an operator object,
so validate types before querying.

### Shell: don't

Prefer a native library over invoking a program at all. Where you must, pass an argument
array to a direct exec — never a single string through a shell. Do not attempt to quote or
escape user data into a command line: quoting rules differ per shell and platform, and the
metacharacter set is larger than anyone remembers.

Environment variables, the working directory, and `PATH` are also inputs — set them
explicitly rather than inheriting.

### HTML: encoding is context-dependent

There is no single "HTML escape". The correct transformation depends on where the value
lands, and using the wrong one is equivalent to using none:

- **Element text** — HTML-entity encode.
- **Attribute value** — attribute-encode *and* quote the attribute.
- **Inside `<script>`** — JavaScript string encoding; better, don't. Pass data via a JSON
  block or a data attribute and read it from the DOM.
- **URL parameter** — URL-encode; and validate the scheme, since `javascript:` and `data:`
  in an `href` execute.
- **CSS context** — CSS-encode; avoid entirely.

Prefer a framework with contextual auto-escaping and keep it on. Treat every explicit
"raw"/"unsafe"/"trusted HTML" call as a finding requiring justification. Where users must
submit rich text, run it through a maintained HTML sanitizer with an allowlist policy —
never a regex.

Defense in depth: a strict Content-Security-Policy, `nosniff`, and `HttpOnly` on session
cookies limit the damage when something slips through. They are second lines, not the fix.

## The modern classes the book predates

Flagged because they postdate the source material and are now among the most exploited.

**SSRF** — your server fetches an attacker-supplied URL and becomes a proxy into your
network, including cloud metadata endpoints. Validate against an **allowlist of
destinations**; blocking internal ranges fails to DNS rebinding, redirects, alternate IP
encodings and IPv6. Disable redirect-following or re-validate each hop, cap the response
size, set timeouts, and give the fetcher its own restricted egress path.

**Unsafe deserialization** — native deserializers instantiate arbitrary types and are remote
code execution on untrusted input. Use data-only formats, parse into a known schema, and
allowlist types where supported.

**XXE** — XML external entities read local files and reach internal services. Disable DTDs
and external entity resolution explicitly; several parsers still enable them by default.

**Server-side template injection** — user data in the *template source* rather than the
template *context* yields code execution in most engines. Templates are code: they come from
your repository, never from a request or a database row.

**Log injection** — unescaped newlines let an attacker forge log entries, and untrusted data
in a format string has produced RCE. Log structured fields, never a concatenated message.

**Header and response splitting** — CR/LF in a header value or redirect target. Reject
control characters in anything reaching a header.

## Reviewing for injection

Find every place data reaches an interpreter and check the mechanism, not the escaping:

1. Grep for string concatenation and interpolation into queries, commands, markup, paths.
2. Check every raw/unsafe/trusted escape hatch and every dynamic identifier.
3. Trace whether the value is attacker-reachable — including via a database row written
   earlier (stored injection) and via a header or cookie.
4. Confirm the defence is structural. "It's escaped" is a finding until you can name the
   exact function and confirm it matches the destination context.

Per-context escaping tables, safe and unsafe patterns per interpreter, and a review checklist
are in `references/injection-by-context.md`.
