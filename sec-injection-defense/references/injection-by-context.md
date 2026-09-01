# Injection by Context — patterns and review checklist

Language-neutral. Translate to the target platform's idiom, and prefer its maintained
library over anything hand-rolled here.

---

## SQL

```
# UNSAFE - concatenation
query("SELECT * FROM users WHERE email = '" + email + "'")

# UNSAFE - the same hole with extra steps
query("SELECT * FROM users WHERE email = '" + escape(email) + "'")

# SAFE - the value never enters the code channel
query("SELECT * FROM users WHERE email = ?", [email])
```

Escaping loses to charset tricks, quoting-context mistakes, and numeric contexts where no
quotes are present at all. Bind instead.

**Identifiers can't be bound.** Allowlist them:

```
SORT = { "date": "created_at", "name": "display_name" }
col = SORT.get(requested) or raise BadRequest      # user string never reaches SQL
sql = "SELECT ... ORDER BY " + col + " " + ("ASC" if asc else "DESC")
```

Other cases worth checking:

- `LIKE` — escape `%` and `_` in user input, or the query walks the whole table.
- `IN (...)` — generate one placeholder per element; cap the count.
- Stored procedures are not automatically safe; they can concatenate internally.
- ORMs: audit `raw`, `expr`, string-built `where`, and any query-fragment API.
- NoSQL: validate that a field is a *string* before it becomes a query operand — a JSON
  object where a string was expected smuggles operators (`{"$ne": null}`).
- Set a statement timeout and a row limit; injection that fails still shouldn't cost the DB.

## OS commands

```
# UNSAFE - shell parses the string; ; | & $() ` newline all work
system("convert " + filename + " out.png")

# SAFE - direct exec, argument array, no shell
exec("convert", [filename, "out.png"], shell=False)

# SAFER - don't shell out at all
image_library.convert(source, dest)
```

- Never pass user data through a shell, quoted or not.
- A leading `-` in a filename becomes an option. Use `--` where the program supports it, or
  prefix relative paths with `./`.
- Set `PATH`, working directory, and environment explicitly; don't inherit.
- Apply a timeout and bound the output.

## HTML — encode for the destination, not "for HTML"

| Where the value lands | Transformation | Notes |
|---|---|---|
| Element text | HTML entity encode | `textContent`, never `innerHTML` |
| Quoted attribute | Attribute encode | Attribute must be quoted |
| Unquoted attribute | — | Never do this; no encoding is sufficient |
| `href` / `src` | URL encode **and** validate scheme | Block `javascript:`, `data:`, `vbscript:` |
| Inside `<script>` | JS string encode | Prefer a JSON block read from the DOM |
| Inside `<style>` | CSS encode | Prefer avoiding entirely |
| Event handler attribute | — | Never; attach listeners in code |

```
# UNSAFE
el.innerHTML = "<div title='" + name + "'>" + name + "</div>"

# SAFE
el.textContent = name                    # text context
el.setAttribute("title", name)           # attribute context, encoded by the API
```

Rich text from users: a maintained sanitizer with an allowlist policy, applied server-side.
Never a regex — HTML is not a regular language and every hand-rolled filter has been bypassed.

Defense in depth, not substitutes: a strict CSP without `unsafe-inline`, `HttpOnly` and
`Secure` and `SameSite` on session cookies, `X-Content-Type-Options: nosniff`.

## LDAP

Escape per RFC rules for the position — filter values and distinguished names have different
escaping — or better, use a parameterized filter API. Allowlist where the value is a known
set. `*` in an unescaped filter turns an authentication check into a wildcard match.

## XML

```
# Explicitly disable, on every parser you construct
disallow DTDs
disallow external general entities
disallow external parameter entities
disable entity expansion / set a low expansion limit
```

Defaults are unsafe in several widely used parsers. Set this at construction, not globally
and hopefully. Bound depth and total expanded size. XPath built from user strings is
injectable too — parameterize or allowlist.

## Templates

```
# UNSAFE - user data is part of the template SOURCE -> code execution
render_string("Hello " + user.name)

# SAFE - user data is CONTEXT
render_template("greeting.html", { "name": user.name })
```

Templates are code. They belong in the repository, not in a request, a database row, or an
admin-editable field. Keep auto-escaping on; audit every `|safe`, `|raw`, `{{{ }}}`, or
equivalent.

## Redirects and forwards

```
# UNSAFE - open redirect, used for phishing and token theft
redirect(request.params["next"])

# SAFE - allowlist, or relative-only with validation
ALLOWED = { "dashboard": "/dashboard", "settings": "/settings" }
redirect(ALLOWED.get(request.params["next"], "/"))
```

If arbitrary relative paths must be supported: reject anything with a scheme, a `//` prefix,
a backslash, or control characters, and resolve before comparing.

## Server-side fetch (SSRF)

```
# UNSAFE
fetch(request.params["url"])

# SAFE
host = parse(url).host
if host not in ALLOWED_HOSTS: reject
fetch(url, follow_redirects=False, timeout=5s, max_bytes=1_000_000)
```

Why blocklists fail: DNS rebinding, redirects to internal addresses, alternate IP encodings
(decimal, octal, IPv6-mapped), `localhost` aliases, and cloud metadata endpoints. Allowlist
destinations. Where a broad fetch is a genuine product requirement, isolate it — a separate
service with its own egress rules, no credentials, and no access to internal networks.

## Deserialization

Never native-deserialize untrusted bytes. Prefer JSON or another data-only format parsed
into a declared schema. Where a framework must deserialize types, allowlist them explicitly.
Treat "we only deserialize our own data" as false the moment that data crosses a boundary or
is stored somewhere another component can write.

## Logs

```
# UNSAFE - forged entries via newlines; format-string risk
log("login failed for " + username)

# SAFE - structured fields, escaped by the logger
log.warn("login_failed", { "username": username })
```

Never place untrusted data in a format string. Never log secrets, tokens, full card numbers,
or personal data.

---

## Review checklist

1. Locate every interpreter boundary: queries, exec/spawn, markup, templates, paths,
   headers, redirects, outbound fetches, deserializers, loggers.
2. At each, is the defence **structural** (binding, argument arrays, safe DOM APIs, schema
   parsing)? If it is escaping, name the exact function and confirm it matches the context.
3. Check every escape hatch: `raw`, `unsafe`, `trusted`, `eval`, `exec`, string-built queries.
4. Check dynamic identifiers — table, column, sort direction, class name, template name.
5. Trace reachability, including **stored** injection: data written earlier, then rendered
   or executed later without encoding.
6. Confirm parser hardening is explicit: XML entities off, safe YAML loader, JSON depth caps.
7. Confirm the second lines exist where they should: CSP, cookie flags, nosniff, egress
   restrictions.
8. Check the tests: is there a case proving the injection is blocked, not just that the happy
   path works?
