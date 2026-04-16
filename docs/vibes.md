# Vibes Runtime Security

Last updated: 2026-04-15

Vibes are client-side experiences. They are allowed to run real browser code, so the safety model depends on isolation rather than on pretending the code will never execute.

If you want the layer-by-layer version instead of the overview, read [Browser runtime deep dive](browser-runtime-layers.md).

## The mental model

Think of a public Vibe as running inside a dedicated viewing room. You can watch what happens in that room, but the room is not supposed to become part of the building's control panel.

In practice, Vibecodr separates the human-facing app from the runtime that executes creator code. Public embeds and player surfaces hand off to a dedicated runtime host on a separate domain so the code that powers a Vibe does not share origin state with the main app.

## Origin separation comes first

The most important defense for Vibes is origin separation.

Creator code runs on a dedicated runtime domain instead of on the main `vibecodr.space` application origin. That means browser features that are scoped to origin, such as cookies and local storage, do not automatically collapse into the main app's trust plane.

Vibecodr also keeps the runtime host execution-oriented. The runtime side is there to run the Vibe, not to act like a cookie-bearing extension of the first-party app.

## There is a control plane and an execution plane

Vibecodr does not treat the runtime iframe as the whole product surface. Public Vibe playback is split on purpose.

One layer handles the outer page and embed policy. Another layer handles player behavior and capability mediation. The runtime frame is the layer that is actually allowed to execute creator-authored code. Keeping those roles separate makes it much harder for a browser execution surface to quietly inherit responsibilities that belong to a trusted shell.

## Messaging is treated like an API boundary

Cross-origin separation helps, but it is not enough on its own. If the parent window would blindly obey any `postMessage` request from a child runtime, then the child could still ask the parent to do privileged things.

That is why Vibecodr's runtime bridge is treated as a real trust boundary. The host side validates where messages come from, which window sent them, whether the message shape is expected, and whether the current session is allowed to use that bridge. The runtime is not supposed to gain broad power just because it can speak to its parent.

## Sensitive browser capabilities stay host-mediated

A Vibe is allowed to be interactive, but that does not mean it receives a blank check to use powerful browser features.

Capabilities such as downloads, popups, clipboard access, and device-style permissions are mediated by the host layer rather than pre-authorized everywhere by default. That design keeps the runtime feeling flexible without turning it into a privileged browser app.

## Network freedom is broad, but not boundary-free

Vibes are intentionally allowed to make ordinary browser requests to the public web. That is part of being a creative runtime instead of a toy.

Even so, Vibecodr still keeps structural boundaries in place. Runtime documents are served with explicit CSP rules. Script and worker behavior are controlled separately from passive media loading. Vibecodr infrastructure and sibling runtime hosts are not treated like ordinary public destinations from inside the runtime. The result is a runtime that can talk to the open web without quietly becoming a shortcut into the platform's own control-plane surfaces.

## Persistence footholds are treated as higher risk

One of the clearest security lines in the browser is the difference between "code runs in this page right now" and "code can keep controlling future page loads."

Vibecodr treats that second category as materially riskier. Persistent footholds such as service workers are blocked for runtime surfaces. That matters because a service worker can outlive the page that created it and influence later browsing behavior on the same origin.

## File serving uses defense in depth

Vibecodr also protects the edges around Vibes, not just the runtime shell itself.

User-uploaded files that could contain executable markup are served with restrictive headers so a browser does not casually execute embedded script when a file is viewed outside the intended runtime path. That reduces the odds that a file which is meant to be handled as content turns into an unexpected execution surface.

## What a Vibe can and cannot do

A Vibe can run creator-authored browser code, load approved web resources, and build real interactive experiences.

A Vibe should not be able to read Vibecodr's authenticated app state, inherit the platform's cookies, register long-lived browser footholds on privileged origins, or use broad embed policy as if it were proof of bridge trust. Those are exactly the boundaries the runtime architecture is designed to hold.
