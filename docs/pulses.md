# Pulses Runtime Security

Last updated: 2026-04-15

Pulses are Vibecodr's server-side execution surface. They are designed to be useful backend helpers without becoming a path into platform trust.

If you want the fuller execution-path version, read [Pulse execution, grants, and dispatch](pulse-execution-and-dispatch.md) and [Secrets, egress, and data handling](secrets-egress-and-data-handling.md).

## The mental model

If a Vibe is a protected browser room, a Pulse is a private workshop behind a staffed gate.

Your code can do real work in that workshop. It can answer requests, talk to external services, and use scoped platform capabilities. But it still reaches those capabilities through platform-owned controls rather than by inheriting the internals of Vibecodr itself.

## Dispatch stays trusted

The most important part of the Pulse model is the dispatch layer in front of user code.

That layer checks identity, scope, plan rules, quotas, and execution policy before forwarding the request into the Pulse runtime. It is the place where Vibecodr keeps decisions that should remain platform-owned. User-authored code is downstream from those checks, not in charge of them.

## Execution uses short-lived scoped grants

Pulse execution is not just "someone has a logged-in session, so let the function run."

Vibecodr uses short-lived grant tokens that are scoped to a specific Pulse and request context. That keeps execution authorization narrower than a general platform session and reduces the amount of reusable authority floating around the system.

Private surfaces can be narrowed further. Public or link-shareable behavior does not automatically mean all callers get the same access to the same backend power.

## User code does not become the platform

Pulse code runs as user-authored backend logic, but it is still treated as untrusted relative to the platform.

That means a Pulse is not supposed to get broad access to Vibecodr's control-plane databases, internal admin surfaces, or infrastructure credentials. It gets the context and capabilities the platform deliberately passes through, not an open door into the rest of the system.

## Secrets are designed to stay out of user-code memory

One of the strongest parts of the Pulse model is how secrets are handled.

Secrets are encrypted at rest and managed through a pattern that avoids handing raw secret values back to the user's function. In the common case, a Pulse asks the platform to use a secret for a specific outbound request, and the trusted layer injects that secret server-side. This matters because the safest secret is not just encrypted in storage. It is also absent from the ordinary runtime variables user code can log, forward, or accidentally expose.

Vibecodr also supports write-only secret management patterns in user-facing surfaces so the platform does not keep re-exposing plaintext values after they are saved.

## Outbound requests are treated as a policy boundary

Pulses need to call external APIs, but outbound network access is one of the easiest places for backend code to become dangerous.

That is why Vibecodr treats egress as a guarded surface. Internal infrastructure targets are blocked. Hostile endpoints such as localhost-style or metadata-style destinations are blocked. Redirects are re-validated instead of blindly followed into whatever host appears next. Observability tags help attribute requests to the Pulse that made them. The point is not to stop useful HTTP calls. The point is to keep "can fetch the web" from turning into "can reach privileged network surfaces."

## Storage stays scoped

Pulses can use platform storage, but that access is intentionally scoped.

The platform keeps storage ownership attached to the correct user or Pulse context so one creator's backend does not drift into another creator's data. Some storage features vary by plan or product surface, but the underlying rule stays the same: storage access should be explicitly scoped, not inferred from a shared runtime.

## Quotas and abuse controls are part of the security model

Rate limits, execution budgets, and concurrency caps are not just billing features. They are also part of how the system protects itself and its users from runaway behavior.

When a Pulse repeatedly fails, overruns its limits, or behaves like an abuse source, Vibecodr can slow it down, deny the request, or quarantine the path instead of letting the problem spread. Security-critical paths are designed to fail closed when the enforcement they depend on is unavailable.

## What a Pulse can and cannot do

A Pulse can run backend logic, call approved external services, use scoped storage, and act as a real programmable helper.

A Pulse should not be able to read the platform's own secrets, wander through internal-only services, or turn a creator function into a generic operator console. The whole design is there to keep that line bright.
