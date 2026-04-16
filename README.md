# Vibecodr Security

Last updated: 2026-04-15

This folder is a public-facing security docs draft for Vibecodr.

The short version is simple: Vibecodr intentionally lets people run code. That means the main security question is not "does code run?" The real question is "where does it run, what can it touch, and which layer stays trusted?"

Vibecodr has two different execution surfaces. Vibes are client-side experiences that run in an isolated browser runtime. Pulses are server-side functions that run behind a trusted dispatch layer on Cloudflare. Both surfaces are designed so user-authored code can be expressive without quietly inheriting platform privileges.

## The model in one page

- Vibes run on a dedicated runtime domain that stays separate from the main app.
- Pulses run behind a trusted gatekeeper that checks identity, scope, quotas, and outbound rules before user code gets useful power.
- Secrets are encrypted at rest and are designed to stay out of user-code memory.
- Auth, moderation, rate limits, and incident telemetry live in trusted layers, not in user-authored code.
- Security-sensitive paths fail closed when the protection a path depends on is missing or unhealthy.

## Start here

- [Trust model](docs/trust-model.md)
- [Vibes runtime security](docs/vibes.md)
- [Pulses runtime security](docs/pulses.md)
- [Platform controls](docs/platform-controls.md)
- [Vulnerability reporting](SECURITY.md)

## If you want the deeper version

The pages above are the fast orientation layer. The docs below go further into the actual security surfaces and why they exist.

- [Edge and trust planes](docs/edge-and-trust-planes.md)
- [Browser runtime deep dive](docs/browser-runtime-layers.md)
- [Pulse execution, grants, and dispatch](docs/pulse-execution-and-dispatch.md)
- [Secrets, egress, and data handling](docs/secrets-egress-and-data-handling.md)
- [Auth, moderation, abuse controls, and telemetry](docs/auth-moderation-abuse-and-telemetry.md)

## What these docs are trying to do

These pages are meant to explain the real shape of Vibecodr's security posture in plain language. They are technical on purpose, but they are not written as an internal runbook. The goal is to help a user, researcher, or careful reviewer understand the trust boundaries without dumping source files, internal-only paths, or operational secrets.

## What these docs are not trying to claim

These docs do not claim that user-authored code becomes harmless. They do not claim that third-party resources on the public web are automatically trustworthy. They do not rely on obscurity. The core promise is narrower and more honest: untrusted code should stay visibly untrusted, and the privileged parts of the system should stay on the other side of explicit boundaries.

## Reporting something real

If you believe you found a security issue, email [founder@vibecodr.space](mailto:founder@vibecodr.space) and include the clearest reproduction path you can. The reporting guidelines live in [SECURITY.md](SECURITY.md).
