# Pulse Execution, Grants, and Dispatch

Last updated: 2026-04-15

Pulses are server-side functions, but they are not meant to be tiny copies of Vibecodr's own backend. They run behind a deliberate execution path that keeps trusted platform decisions in trusted layers.

## The admission path matters

The first thing to understand about Pulses is that execution is staged.

A caller does not simply hit a Pulse and inherit backend power. Identity is established upstream. A short-lived grant is issued for the specific execution lane. Dispatch resolves the target, checks whether the request is allowed, applies plan and quota rules, and only then forwards the request into the user-authored runtime.

This is important because it means the platform still owns the admission decision even when the code that eventually runs belongs to a creator.

## Short-lived grants keep authority narrow

One of the biggest differences between "ordinary logged-in user" and "allowed to execute this server-side function right now" is scope.

Vibecodr uses short-lived grant tokens scoped to Pulse execution so it does not have to rely on a broad session token alone. That narrows the authority carried into the runtime path and reduces the chance that a reusable credential becomes a catch-all backend pass.

In plain language, the execution ticket is smaller than the account itself. That is almost always the safer design.

## Dispatch is not just plumbing

Dispatch is where the trusted side of the system decides whether a Pulse is allowed to run and under what conditions.

That includes checks such as which Pulse is being targeted, whether the caller has the right to invoke it, whether the Pulse is private or limited to the owner, whether the user has the right plan for the requested capability, whether concurrency or budget limits have been exceeded, and whether the Pulse is disabled or quarantined. This layer is not optional scaffolding. It is the control surface that keeps user-authored backend code from becoming the authority that admits itself.

## The runtime still stays untrusted

Once a Pulse starts executing, it can do useful work. It can handle data, call APIs, and use scoped platform capabilities. Even so, it is still not supposed to become part of Vibecodr's own trusted backend.

That means the Pulse runtime should not be able to read platform-wide secrets, call privileged internal services freely, or treat the platform's internal database and operations surfaces as if they were ordinary application libraries. The user runtime receives a carefully chosen environment, not the keys to the building.

## Storage is capability-based, not ambient

Pulses can use storage, but the storage model is scoped rather than ambient.

KV access is tied to the Pulse context. Pro-tier SQL access is provisioned as a per-user native D1 binding rather than as a single shared database that hopes row filtering is always applied correctly. That matters because infrastructure-level separation is a much cleaner trust story than "all tenants live in the same place and every query must remember to behave."

## Missing prerequisites are supposed to stop the path

Another important part of the execution model is what happens when the expected boundary is missing.

If a request depends on a grant, a native binding, a quota check, or a security-sensitive guardrail, the safer behavior is to deny the path rather than improvise. Vibecodr leans toward fail-closed behavior on those paths because backend execution is exactly where casual fallbacks tend to become privilege leaks.

## Why this design exists

The simplest way to describe the Pulse design is this: creator code gets real backend power, but it gets that power through a staffed checkpoint.

That checkpoint is what makes it possible to support useful server-side code without turning the user runtime into the platform runtime. The whole stack depends on keeping that distinction explicit.
