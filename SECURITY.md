# Security Policy

Last updated: 2026-04-15

If you believe you found a security issue in Vibecodr, please report it privately to [founder@vibecodr.space](mailto:founder@vibecodr.space).

Please include as much of the following as you can:

- a short description of the issue
- the exact URL, route, or product surface involved
- step-by-step reproduction instructions
- whether the issue requires authentication
- what kind of account or role was used
- timestamps, screenshots, logs, or captured requests if they help prove the issue
- your assessment of impact

## Please keep testing safe

Good-faith security research is welcome, but please keep it narrow, non-destructive, and evidence-driven.

That means:

- use the least invasive proof that establishes the issue
- avoid deleting data, changing configuration, rotating secrets, or degrading service for real users
- do not use phishing, credential stuffing, password spraying, MFA fatigue, or sustained brute force
- do not run noisy automation that risks lockouts or production instability
- if you appear to reach a privileged surface, gather the smallest safe proof and stop

## In scope

The following are generally in scope when they are reachable from public internet surfaces or through ordinary user-facing product flows:

- public web routes and APIs
- authenticated user flows
- auth, session, and authorization behavior
- browser security boundaries
- runtime isolation behavior
- privilege checks on moderator or admin surfaces that appear reachable from user-visible behavior

## Out of scope unless explicitly authorized

- social engineering
- physical attacks
- denial-of-service testing
- destructive testing against production data
- attacks on third-party vendors that do not depend on a Vibecodr exposure or misconfiguration

## Coordinated disclosure

Please do not publicly disclose a vulnerability before Vibecodr has had a reasonable chance to investigate and address it.

Vibecodr will review reports as quickly as possible, ask follow-up questions when needed, and work to confirm impact before shipping a fix. Clear, reproducible reports help the most.
