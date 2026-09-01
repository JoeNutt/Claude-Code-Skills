---
name: sec-authz-least-privilege
description: This skill should be used when deciding who may do what - adding an endpoint or action that needs a permission check, designing roles and permissions, handling multi-tenant data isolation, choosing what privileges a process, container, service account or token runs with, or reviewing code for missing authorization. It covers deny-by-default authorization, object-level checks, privilege separation, and least privilege at runtime. Trigger phrases include "permissions", "who can access", "authorization", "roles", "admin only", "multi-tenant", "can this user", "service account", "run as root", "API key scope", "IDOR", "access control".
---

# Authorization and Least Privilege

Broken access control is the most exploited vulnerability class in production software. It is
also the least likely to be caught by a scanner, because a missing check looks exactly like
code that works.

Two separate questions, and conflating them is itself a common bug:

- **Authentication** — who is this? (`sec-crypto-secrets` covers credential handling.)
- **Authorization** — is this actor allowed to do this, to *this object*, right now?

## Deny by default

Every action requires an explicit grant. The default answer is no.

- Authorize in one place — a middleware, a policy layer, a decorator — so a new endpoint is
  protected by construction rather than by the author remembering. **A new route with no
  policy must fail closed**, not fall through to public.
- Never rely on the UI hiding an action, or on a URL being unguessable. Both are
  client-side, and neither is a control.
- Check on **every** request. A permission verified at login is stale by the next call;
  roles change, sessions outlive them.
- Fail closed on error: if the policy lookup throws, deny. An authorization check that
  proceeds on exception is a bypass.

## Authorize the object, not just the action

The most common serious flaw: a check that the caller is *logged in*, or holds a *role*, but
never that this particular record is theirs.

```
# BROKEN - authenticated, but any user reads any invoice
invoice = db.get(request.params["id"])
return invoice

# CORRECT - the actor and the object are checked together
invoice = db.get(request.params["id"])
if not policy.can_read(current_user, invoice): deny()
return invoice

# BETTER - scope the query so the object can't be fetched at all
invoice = db.get(id=request.params["id"], owner=current_user.id) or deny()
```

Prefer the scoped query. A filter in the data access layer cannot be forgotten by a later
`if`, and it fails safe when someone adds a new read path.

The same applies to **function**-level access: an admin endpoint reachable by a normal user
who simply knows the path. Both are ranked at the top of the current OWASP Top 10 as object-
and function-level authorization failures.

**Multi-tenancy:** the tenant predicate belongs in the data layer, not in each query.
Enforce it where it cannot be omitted — a scoped connection, a session variable with
row-level security, or a repository that requires a tenant. Every hand-written query is a
chance to forget it, and the failure is silent and total.

## Least privilege at runtime

Give every actor — human, process, service, token — the minimum rights for the minimum time.

- **Processes:** don't run as root or administrator. Drop privileges immediately if elevation
  is needed only at startup, and never regain them. Containers: non-root user, read-only
  filesystem, dropped capabilities, no privileged mode.
- **Database accounts:** the application account does not need schema rights. Separate
  accounts for migrations and for runtime; read-only where reads are all that happen.
- **Service accounts and cloud roles:** scope to specific resources and actions, not
  wildcards. A role granting `*` on `*` is the single most damaging misconfiguration
  available, and it is usually a temporary fix nobody removed.
- **Tokens and API keys:** narrow scopes, short lifetimes, bound to an audience. Prefer
  short-lived credentials that rotate automatically over long-lived secrets.
- **File and network access:** minimum permissions; expose the minimum surface; bind to
  localhost unless remote access is required.
- **Separate duties:** the component that requests a sensitive action shouldn't be the one
  that approves it.

Ask of every privilege: what is the blast radius if this component is fully compromised?
That is the real question least privilege answers.

## Elevation boundaries

Where privilege changes, put a boundary and treat everything below it as untrusted:

- Re-authenticate for sensitive actions — password change, MFA reset, payment details,
  destructive operations — even within a valid session.
- Never let the client assert its own identity or role. A user id, tenant id, role, or price
  in a request body, header, cookie, or JWT claim your service didn't sign is input, not fact.
- Verify tokens properly: signature, issuer, audience, expiry, and algorithm. Reject `none`
  and never let the token choose its own verification key.
- Re-check authorization after any redirect, workflow step, or state transition. Multi-step
  flows are routinely protected at step one and open at step three.

## Reviewing for missing authorization

Scanners don't find these. Read for them:

1. List every route, action, and message handler. For each: authenticated? authorized? is
   the **object** checked, or only the action?
2. Find every read or write by id from user input — each is an object-level check to verify.
3. Confirm the tenant predicate is enforced structurally, not per query.
4. Look for role checks that only guard the UI, and for endpoints protected by obscurity.
5. Check the error paths: does a thrown policy lookup deny, or continue?
6. Check for privilege that was widened temporarily — a wildcard role, a debug bypass, a
   `TODO: tighten this` — and never narrowed.
7. Confirm there are tests asserting **denial**, not just success. A test suite that only
   proves the owner can read proves nothing about the stranger.

Patterns for policy layers, multi-tenant enforcement, and token scoping are in
`references/access-control-patterns.md`.
