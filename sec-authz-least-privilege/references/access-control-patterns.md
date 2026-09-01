# Access Control Patterns

## Choosing a model

| Model | Grants by | Fits | Watch |
|---|---|---|---|
| **RBAC** — role-based | Named role | Stable, coarse org structures | Role explosion; roles that accrete permissions and are never trimmed |
| **ABAC** — attribute-based | Attributes of actor, object, environment | Fine-grained, contextual rules | Hard to audit "who can reach this?"; needs tooling |
| **ReBAC** — relationship-based | Relationship to the object (owner, member, editor) | Sharing, collaboration, hierarchies | Deep graph traversal cost; cycles |
| **Capability** | Possession of an unforgeable token | Delegation, links, service-to-service | Revocation; leakage via logs, referrers, history |

Most systems end up RBAC for coarse rights plus ownership/relationship checks for objects.
Start there. Whatever the model, the implementation rule is the same: **one policy layer,
consulted everywhere, deny by default.**

## A policy layer

Centralize the decision so it can be found, tested, and audited:

```
policy.can(actor, action, object) -> allow | deny
```

- One place to read, one place to test, one place to fix.
- Log every denial with actor, action, object, and reason. Denials are your earliest signal
  of an attack in progress.
- Make the default `deny`, including for an unknown action or an object type nobody wrote a
  rule for.
- Return the same response for "not found" and "not permitted" where existence itself is
  sensitive — otherwise the error message enumerates your data.

**Make omission impossible, not merely discouraged.** The strongest version fails closed at
the framework level: routes require a declared policy, and a route without one refuses to
register. A convention that authors must remember will eventually be forgotten, and the
forgetting is silent.

## Multi-tenant isolation

Ranked strongest to weakest:

1. **Separate databases or schemas per tenant.** Strongest isolation, heaviest operationally.
2. **Row-level security in the database**, driven by a session variable set at connection
   checkout. The database enforces it even if application code forgets.
3. **A repository layer that requires a tenant** — no raw query access; the tenant predicate
   is applied centrally and cannot be omitted by a caller.
4. **A tenant predicate in every query, by convention.** Weakest. One missed `WHERE` is a
   cross-tenant breach, and it will pass every test written by someone with one tenant.

Whichever you pick: test with **at least two tenants**, and assert that tenant B cannot read,
update, or delete tenant A's objects. A single-tenant fixture cannot catch the bug this
entire section exists to prevent.

Also check the paths that bypass the query layer: caches keyed without a tenant, search
indexes, exports, background jobs, webhooks, and admin tools.

## Tokens and sessions

- **Sessions:** rotate the identifier on privilege change and at login (prevents fixation);
  set `HttpOnly`, `Secure`, `SameSite`; enforce absolute and idle timeouts; invalidate
  server-side on logout — clearing a cookie is not revocation.
- **JWTs and signed tokens:** verify signature, issuer, audience and expiry; pin the expected
  algorithm and reject `none`; never let the token select its own key. Remember they are
  **not revocable** by default — keep lifetimes short and hold a revocation list for
  sensitive operations.
- **API keys:** scope them to actions and resources, identify the caller, allow rotation
  without downtime, and support revocation. Store only a hash of the key.
- **Capability URLs** (unguessable links): treat as bearer credentials — they leak through
  referrers, logs, browser history, and shared screenshots. Expire them; don't use them alone
  for sensitive data.
- **Never trust client-asserted identity.** A user id, role, tenant, price, or quantity in a
  request body, header, or cookie is input. Derive identity from the verified session or a
  signature you can check.

## Least privilege in practice

**Process:**
- Non-root service account; separate account per service.
- Elevate only if required, at startup, and drop irreversibly before handling requests.
- Containers: non-root `USER`, read-only root filesystem, `cap_drop: ALL`, no `--privileged`,
  no host network or mounts unless genuinely needed.

**Database:**
- Runtime account: DML only, on the tables it uses. No DDL, no superuser.
- Separate migration account, used only by migrations.
- Read-only replica account for reporting and analytics.

**Cloud IAM:**
- Scope to named resources and actions. `Action: "*"` or `Resource: "*"` is a finding.
- Prefer workload identity over long-lived static keys.
- Short-lived, automatically rotated credentials.
- Separate roles per environment; production credentials must not exist in a dev account.
- Audit for privilege granted "temporarily" during an incident and never revoked — this is
  where wildcard roles come from.

**Secrets:** a secrets manager, not environment files in the repository. Rotate on a
schedule and on staff departure. Scope each secret to the one service that needs it. See
`sec-crypto-secrets`.

## Common broken patterns

| Pattern | Why it breaks |
|---|---|
| Check role, not ownership | Any user reads any object of that type (IDOR / BOLA) |
| Authorize in the controller only | Service, job, GraphQL resolver, or admin path bypasses it |
| Hide the button | The endpoint is still reachable; the UI is not a control |
| Unguessable id as the control | Ids leak via logs, referrers, exports, support tickets |
| Check at step 1 of a wizard | Steps 2–5 are unprotected |
| Trust a client-supplied role or tenant | Attacker sets their own |
| Cache the permission decision | Revocation doesn't take effect |
| Allow on policy-lookup error | An induced error becomes a bypass |
| Different responses for missing vs forbidden | Enumerates records |
| Tests assert success only | The denial path was never exercised |
