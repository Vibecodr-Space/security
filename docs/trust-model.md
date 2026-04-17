# Vibecodr Trust Model

Last updated: 2026-04-15

The easiest way to understand Vibecodr security is to stop thinking of the product as one giant app. It is closer to a set of neighboring rooms with different rules.

Some rooms are trusted. Those are the places that verify identity, decide who can do what, store secrets, apply moderation, and keep audit history. Some rooms are intentionally untrusted. Those are the places where creator-authored code is allowed to run. The product works only if those rooms stay distinct.

## Two execution surfaces

Vibecodr has two primary execution surfaces.

Vibes are browser-side experiences. They run in the viewer's browser inside an isolated runtime. A vibe is allowed to execute HTML, CSS, and JavaScript, but that code is not supposed to inherit access to Vibecodr's app state, cookies, internal APIs, or other creators' runtime storage.

Pulses are server-side functions. They run on Cloudflare's edge, but not inside the same trust zone as Vibecodr's control-plane services. A Pulse can do useful backend work, yet it still has to go through platform-owned gates for identity, quotas, secret use, outbound policy, and storage scope.

## The main idea

Vibecodr does not try to make creator code inert. It tries to keep creator code in the right place.

For Vibes, that means browser isolation, cross-origin separation, strict host mediation for sensitive capabilities, and a message bridge that treats the parent-child boundary as a privileged API.

For Pulses, that means a trusted dispatch layer in front of user code, short-lived grants, scoped storage access, guarded outbound traffic, and a secret model that avoids handing raw secret values back to the user's function.

## What stays trusted

The main application surface stays trusted. So do the services that verify auth, issue short-lived grants, store encrypted secrets, enforce moderation, apply quotas, and collect operational telemetry.

Those trusted layers are where Vibecodr decides whether a caller is allowed to act, whether a piece of content should remain public, whether a request has exceeded its limits, and whether a secret can be used for a specific outbound call.

## What stays untrusted

Creator-authored runtime code stays untrusted, even when it is allowed to do real work.

That includes code running inside a Vibe and code running inside a Pulse. Both can be powerful. Neither should quietly become a platform operator shell. Security in Vibecodr depends on keeping that distinction legible.

## Why this matters for users

This model is what lets Vibecodr stay permissive without becoming careless.

Creators can publish weird things. Viewers can open them without turning the main site into the same trust surface. Pulses can call external services and hold useful credentials without those credentials becoming ordinary variables that user code can print, forward, or accidentally leak.

## What Vibecodr does not promise

Vibecodr does not promise that code from the public web is safe. It does not promise that every third-party dependency is benign. It does not promise that user-authored code cannot fail, fingerprint a browser, or behave badly inside its own allowed sandbox.

The promise is narrower and more important: Vibecodr is built to do everything in its power to let untrusted code be viewed, played with, and shared in a way that stays safe, contained, and meaningfully separated from the trust of the platform itself.
