---
name: auth-implementation-patterns
description: Implements authentication and authorization safely. Use when adding login, sessions, JWT, OAuth2/SSO, or access control (roles, permissions, resource ownership). Use when reviewing or debugging auth code.
---

# Authentication & Authorization Patterns

## Overview

Authentication (who are you) and authorization (what may you do) are the highest-stakes code in most applications — mistakes are silently exploitable. This skill encodes the decisions: which strategy fits, where credentials live, and what must be checked on every request. For framework-specific implementation, follow `source-driven-development` and fetch current official docs — auth libraries change fast, and stale patterns are dangerous. For general input validation, headers, and secrets handling, see `security-and-hardening`.

## When to Use

- Implementing login, registration, logout, or password reset
- Securing REST/GraphQL APIs or adding access control (RBAC, permissions, ownership)
- Adding OAuth2/social login or SSO
- Reviewing, migrating, or debugging auth code

## Choose an Authentication Strategy

| Strategy | Choose when | Key properties |
|---|---|---|
| **Server-side sessions** | Default for first-party web apps | Stateful; instant revocation; needs a session store; CSRF protection required |
| **JWT access + refresh tokens** | Multiple API consumers, mobile clients, or scaling without a shared session store | Stateless access token; revocation only works via a server-side refresh-token store |
| **OAuth2 / OIDC** | Social login, enterprise SSO, delegated identity | You are the client; after the callback you still create your own local user record and session/tokens |

Don't mix strategies ad hoc. Pick one primary mechanism; OAuth login should feed into it, not exist beside it.

## Non-Negotiable Rules

**Passwords**
- Hash with bcrypt (cost ≥ 12), scrypt, or argon2id. Never store or log plaintext; never use fast hashes (MD5/SHA-*).
- Enforce a minimum length (12+) and check against breached-password lists; composition rules matter less than length.
- On login failure, return one generic "invalid credentials" error — don't reveal whether the email exists.

**Session and token storage**
- Cookies carrying sessions or tokens: `httpOnly`, `secure`, `sameSite` (lax or strict), sensible `maxAge`.
- Never put tokens in `localStorage`/`sessionStorage` (XSS-readable) or in URLs (leaks via logs, referrers, history) — including OAuth callback redirects.
- Access tokens are short-lived (15–30 min). Anything longer must be revocable server-side.

**JWT specifics**
- Verify signature, expiry, and (for third-party tokens) issuer + audience on every request. Pin the algorithm; reject `alg: none`.
- Keep payloads minimal — no PII, no secrets; JWTs are readable by anyone who holds them.
- Refresh tokens: store hashed in the database, expire them, rotate on use, and delete on logout (single device) or by user id (all devices). This is your only revocation lever.

**OAuth2 / OIDC**
- Always use `state` (CSRF) and PKCE. Use the authorization-code flow; never the implicit flow.
- Validate the redirect URI against an exact allowlist.
- Deliver resulting tokens to the frontend via an httpOnly cookie or a one-time server exchange — not as a query parameter.

**Endpoints**
- Rate-limit login, registration, refresh, and password-reset endpoints aggressively (e.g., 5–10 attempts / 15 min).
- Password-reset tokens: single-use, short expiry (≤ 1 hour), hashed at rest, invalidated on password change.
- Invalidate sessions/refresh tokens on password change; consider re-authentication for sensitive actions.

## Choose an Authorization Model

| Model | Choose when | Shape |
|---|---|---|
| **Roles (RBAC)** | Few, stable permission tiers (user/moderator/admin) | Role hierarchy; middleware checks `hasRole` |
| **Permissions** | Actions vary independently of tiers, or roles keep sprouting exceptions | Map roles → permission strings (`read:users`); middleware checks permissions, not roles |
| **Resource ownership** | Users act on their own data | Load the resource, compare `resource.ownerId` to the authenticated user (admins may bypass) |

Most real systems combine them: RBAC or permissions for capabilities, plus an ownership check on every resource access.

```typescript
// The check every protected mutation needs — authn is not authz
const task = await taskService.findById(req.params.id);
if (!task) return res.status(404).json({ error: { code: 'NOT_FOUND' } });
if (task.ownerId !== req.user.id && req.user.role !== 'admin') {
  return res.status(403).json({ error: { code: 'FORBIDDEN' } });
}
```

**Authorization rules:**
- Check authorization on **every** protected endpoint, server-side. Client-side checks are UX, not security.
- Missing ownership checks (IDOR) are the most common real-world auth bug — test that user A cannot read or mutate user B's resources.
- Return 401 for "not authenticated", 403 for "authenticated but not allowed", and 404 when revealing existence would leak information.
- Apply least privilege to service accounts and API keys; scope them narrowly.

## Red Flags

- Tokens in localStorage, URLs, or logs
- JWT accepted without signature/expiry verification, or with unpinned algorithm
- Endpoints that check authentication but not authorization (IDOR)
- Password hashing with fast hashes, or plaintext anywhere
- No rate limiting on login/reset endpoints
- OAuth flow missing `state`/PKCE, or redirect URI not allowlisted
- Refresh tokens that can't be revoked server-side
- Auth code written from memory instead of current official docs

## Verification

After implementing auth-related code:

- [ ] Passwords hashed with bcrypt/scrypt/argon2id; no plaintext in code, DB, or logs
- [ ] Session/token cookies are httpOnly + secure + sameSite; nothing sensitive in localStorage or URLs
- [ ] Every protected endpoint checks both authentication and authorization (including ownership)
- [ ] Tested: user A cannot access user B's resources; expired/tampered tokens are rejected
- [ ] Refresh tokens (if used) are hashed at rest, rotated, and revocable
- [ ] Rate limiting active on auth endpoints; reset tokens single-use and expiring
- [ ] Framework patterns verified against current official docs (`source-driven-development`)
