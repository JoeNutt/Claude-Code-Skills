---
name: sec-crypto-secrets
description: This skill should be used when handling passwords, API keys, tokens, connection strings, certificates or encryption keys, when encrypting or hashing anything, when generating random values used for security, or when deciding how sensitive data is stored and transmitted. It covers algorithm selection, password storage, key management, secret handling, and the failure modes that make cryptography go wrong. Trigger phrases include "encrypt this", "hash this", "store passwords", "API key", "secret", "credentials", "token", "random", "generate a key", "TLS", "certificate", "connection string", "is this secure to store".
---

# Cryptography and Secret Data

Cryptography fails through **misuse**, not broken maths. The primitives are fine; the bugs
are in choosing the wrong one, using it in the wrong mode, reusing a nonce, or protecting the
key with the thing the key was supposed to protect.

## The rules

1. **Never invent your own algorithm or protocol.** Not a cipher, not a hash, not an
   authentication handshake, not "encryption" by XOR or base64. Base64 and hex are encodings,
   not protection.
2. **Never implement a standard primitive yourself.** Use the platform's vetted library.
   Hand-rolled implementations leak through timing and edge cases.
3. **Prefer a high-level library** that picks the primitives for you (an authenticated
   "secretbox"-style API, a maintained TLS stack, a password-hashing library) over composing
   primitives yourself.
4. **Encryption is not authentication.** Ciphertext without integrity can be tampered with.
   Use authenticated encryption (AEAD), or encrypt-then-MAC. Never MAC-then-encrypt.
5. **Encryption is not authorization.** Encrypted data still needs an access check.
6. **The key is the secret**, and a key stored beside the data it protects protects nothing.
7. **Compare secrets in constant time.** A byte-by-byte early-exit comparison of tokens,
   MACs, or password hashes leaks the value through timing.

## Choosing primitives

Named for the decision, not to be memorized — check current guidance when it matters.

| Need | Use | Never |
|---|---|---|
| Password storage | Argon2id; else scrypt or bcrypt | Any plain hash, salted or not |
| Data at rest / in transit | AEAD: AES-GCM, ChaCha20-Poly1305 | ECB mode; unauthenticated CBC; DES, 3DES, RC4 |
| Hashing / integrity | SHA-256, SHA-3, BLAKE2 | MD5, SHA-1 |
| Message authentication | HMAC-SHA-256, or the AEAD's own tag | Hand-built `hash(key + message)` |
| Signatures | Ed25519, ECDSA P-256, RSA-PSS ≥3072 | RSA PKCS#1 v1.5 for new work; small keys |
| Key exchange | X25519, ECDH | Custom handshakes |
| Random for security | The OS/platform **CSPRNG** | `rand()`, `Math.random()`, `Random`, time-seeded PRNGs |
| Key derivation from a password | Argon2id / PBKDF2 with high iterations | Using the password directly as a key |
| Key derivation from a key | HKDF | Truncating or reusing a key for two purposes |
| Transport | TLS 1.3 (or 1.2), certificates verified | Disabling verification "for now"; plaintext fallback |

**Passwords are not encrypted, they are hashed** — with a slow, memory-hard, per-password
salted function, so a stolen database is expensive to attack. Store the parameters with the
hash and raise the cost factor over time. Never encrypt a password (encryption is reversible)
and never invent a "salt and SHA-256" scheme.

**Randomness:** anything security-bearing — tokens, session ids, nonces, IVs, salts, reset
links, keys — comes from the cryptographic generator. A general-purpose PRNG is predictable
from a few outputs. Never seed from the clock or the process id.

**Nonces and IVs:** never reuse one with the same key. AES-GCM nonce reuse destroys
confidentiality *and* forgeability in one step. Generate randomly, or use a counter you can
prove never repeats across restarts and instances.

## Key and secret management

Secrets are what actually leaks. The failure is almost never the algorithm.

- **Never commit secrets.** Not in source, config, test fixtures, CI files, container images,
  infrastructure code, or comments. Assume any secret ever committed is public: rotate it
  rather than deleting the commit. Git history and image layers are forever.
- **Use a secrets manager** — the platform's vault or KMS. Environment variables are
  acceptable at the edge; a `.env` in the repository is not.
- **Separate keys by purpose and environment.** One key, one job. Production keys never exist
  in development, and a developer laptop is not a place production credentials live.
- **Rotate** on a schedule, after any suspected exposure, and when someone with access leaves.
  Design for rotation from the start — support two valid keys during a changeover, or the
  rotation will never actually happen.
- **Keep secrets out of:** logs, error messages, URLs and query strings, analytics, crash
  reports, client-side code and mobile binaries, and support tickets. Redact where values are
  formatted, not where they're read.
- **Minimize lifetime in memory** where the platform allows; prefer short-lived credentials
  over careful handling of long-lived ones.
- **Client-side secrets do not exist.** An API key in a mobile app, SPA, or browser extension
  is published. Design so it doesn't need one, or proxy through a server.

## Sensitive data

- **Classify before protecting.** Know which fields are credentials, personal data, financial
  or health data — the handling follows from the classification.
- **Minimize.** Data not collected cannot leak. Data deleted on schedule cannot leak later.
  Storing less is the cheapest control available.
- **In transit:** TLS everywhere, including internal service-to-service traffic. Verify
  certificates; never ship a disabled-verification path, even behind a flag.
- **At rest:** encrypt sensitive fields, with keys held elsewhere. Extend it to backups,
  exports, caches, search indexes, and derived stores — the copies are where breaches happen.
- **Don't log it, don't put it in a URL**, and keep it out of error messages and stack traces.
- **Tokenize where you can:** store a reference rather than the value — a payment token
  rather than a card number.

## Reviewing

1. Any custom crypto, or a primitive composed by hand? Both are findings.
2. Algorithms against the table above — and MD5/SHA-1 used for anything security-bearing.
3. Passwords: slow, salted, memory-hard? Parameters stored and current?
4. Randomness: CSPRNG everywhere security depends on unpredictability?
5. Encryption authenticated? Nonce/IV unique per use?
6. Where does the key live, and what protects it?
7. Secrets in the repository, history, images, or CI config?
8. Rotation possible without downtime, and actually scheduled?
9. Secrets reachable in logs, URLs, errors, or client-side code?
10. Constant-time comparison for tokens and MACs?
11. TLS verification enabled everywhere, with no plaintext fallback?

Detail on password parameters, key hierarchies, rotation mechanics, and specific failure
modes is in `references/crypto-choices.md`.
