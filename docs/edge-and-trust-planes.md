# Edge and Trust Planes

Last updated: 2026-04-15

If you want to understand Vibecodr deeply, start here. The product makes more sense when you stop imagining it as one app and start imagining it as several neighboring planes with different jobs and different trust levels.

The easiest way to say it is this: the same system that lets people publish code also has to prevent that code from inheriting the platform's authority. Vibecodr does that by splitting responsibilities across distinct browser and edge surfaces instead of pretending one layer can safely do everything.

## The public app plane

The main public app plane is where people browse, sign in, manage profiles, publish work, and interact with content. This plane owns the ordinary product contract. It is where user identity is verified, where write paths are checked, and where most public APIs live.

That does not mean every request in this plane gets the same power. It means this is the place where the platform decides whether a caller has the right to ask for something in the first place.

## The public runtime plane

Vibecodr also has a dedicated runtime host family for public execution surfaces. This matters because a social coding product cannot safely treat "viewer-facing app shell" and "creator-authored runtime" as the same origin or the same trust layer.

The runtime host family exists to serve execution-oriented browser surfaces. It is intentionally kept separate from the main app's authenticated state. That separation is what lets Vibecodr allow rich client-side code without making the main site and the runtime one merged trust domain.

## The dispatch plane

On the server-side path, the equivalent separation happens through dispatch.

Dispatch is the layer that decides which Pulse is being invoked, whether the request has the right grant, whether plan and quota rules allow the operation, whether the execution target is disabled or quarantined, and what narrowly-scoped context should be passed forward. In other words, dispatch is not just a router. It is the point where user-authored backend code meets platform-owned policy.

## The outbound control plane

Once user backend code is allowed to run, there is still another boundary: egress.

Vibecodr does not treat outbound fetch as a harmless detail. It has a dedicated outbound control plane that can block self-invocation, platform abuse, and suspicious redirect chains before a request leaves the edge. That gives the product one more place to say, "this code is allowed to call the public web, but it is not allowed to turn that into a side door into Vibecodr itself."

## The internal service plane

There are also operations that should never live on a public hostname at all. Internal grants, administrative operations, secret-handling paths, billing webhooks, and certain runtime support paths live on service-only surfaces protected by internal signing rules rather than by public browser access patterns.

This is important because it keeps the most privileged flows from being reachable just because a public route exists elsewhere in the product.

## Why the separation matters

Each plane has a different job. The public app plane is where users interact. The public runtime plane is where untrusted browser code executes. Dispatch is where user backend code is admitted under policy. Outbound control is where external network calls are fenced. Internal service surfaces are where privileged operations stay off the public internet path.

Security in Vibecodr comes from keeping those jobs explicit. A lot of platform failures happen when one surface quietly starts borrowing assumptions from another. Vibecodr's architecture is designed to resist that drift.

## The practical takeaway

When Vibecodr says it is permissive, it does not mean every layer gets relaxed. It means the product tries to give creators expressive runtime surfaces while keeping identity, authority, secrets, moderation, and privileged operations in planes that are meant to own them.

That is the throughline for everything else in this doc set.
