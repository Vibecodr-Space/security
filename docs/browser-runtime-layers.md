# Browser Runtime Deep Dive

Last updated: 2026-04-15

The browser runtime is where Vibecodr's philosophy gets tested most directly. The platform intentionally allows creator-authored code to execute in the viewer's browser. That means the job is not to deny execution. The job is to keep execution in the right place and to keep privilege from leaking across the wrong boundary.

## Layer 1: origin and sandbox separation

The first and most important browser defense is origin separation.

Public Vibe code runs on a dedicated runtime host rather than on the main authenticated app origin. That matters because the browser treats origin as one of its strongest native security boundaries. Cookies, storage, and a long list of privileged assumptions collapse too easily when an app starts mixing execution surfaces on the same origin.

Vibecodr treats cross-origin runtime hosting as a release gate, not a nice extra. If a runtime surface is supposed to run in the isolated mode, the safe contract is to keep it on the dedicated runtime host. The platform should not quietly slide back into "just run it on the main app origin" because a deployment or host configuration is wrong.

## Layer 2: separate the wrapper, the control shell, and the runtime

Another subtle but important boundary is role separation inside the browser itself.

Public Vibe playback is not just one iframe and a prayer. There is an outer policy and embed layer, there is a control shell that owns the product-facing behavior around the runtime, and there is the runtime frame that is actually allowed to execute creator-authored code. That split helps keep the runtime from quietly becoming the place where sizing logic, UX control, browser capability approval, and other host-side responsibilities pile up.

The control shell should stay predictable. The runtime should stay untrusted. Once those roles blur, browser isolation gets much harder to reason about.

## Layer 3: the bridge is a privileged API

Cross-origin separation is necessary, but it is not enough.

If the parent window would blindly accept any message from a child runtime, then the runtime could still ask the parent to do privileged work on its behalf. That is why the message bridge is treated as a genuine API boundary. It is not just a convenience channel.

The host side validates the sender, the exact trusted origin, the message shape, and the session context before allowing host-mediated actions. Public runtime hosts are not supposed to treat broad embed ancestry or generic wildcard messaging as proof that a parent should be trusted.

## Layer 4: sensitive browser powers stay host-mediated

A Vibe can be interactive without receiving a blank check to the browser.

Downloads, popups, clipboard actions, and device-style permissions are treated as mediated capabilities rather than something every runtime automatically gets in full. This is one of the places where Vibecodr tries to be permissive and safe at the same time. The runtime can ask. The host layer decides whether the capability should actually be granted on that surface.

This also matters because some browser permissions are effectively document-scoped. If a privilege changes the meaning of the iframe sandbox or `allow` policy, the right way to apply it is often to rearm or reload the document cleanly rather than pretend the browser will safely reinterpret the old document in place.

## Layer 5: persistent footholds are treated differently from one-page code

There is a major difference between "this page runs code right now" and "this code can keep controlling future browsing on the same origin."

Vibecodr treats persistent footholds such as service workers as a materially higher-risk category and blocks them on runtime surfaces. That choice is important because a service worker can outlive the page that created it and influence later navigations. A runtime that is meant to stay sandboxed should not quietly acquire that kind of reach.

## Layer 6: network policy distinguishes openness from trust

Vibes are intentionally allowed to talk to the public web. That is part of what makes them useful and expressive.

But Vibecodr is careful not to confuse "ordinary web egress is allowed" with "everything reachable over HTTPS is trusted." Runtime documents use explicit CSP rules. Passive media loading is treated differently from executable script trust. Requests to Vibecodr infrastructure and sibling runtime hosts are fenced separately from normal public traffic. In other words, the runtime is open enough to build real things, but not so open that it can treat the platform's own control surfaces like ordinary third-party websites.

## Layer 7: dependency trust is treated like a real security decision

Every time a platform lets untrusted runtime code load more code, it is making a trust decision whether it admits that or not.

Vibecodr tries to keep those decisions explicit. Direct script fallback is not something that should expand casually until it becomes an invisible allow-everything policy. Reviewed executable origins and deterministic first-party dependency delivery are stronger contracts than a runtime that silently says yes to whatever remote script URL happens to appear.

That matters because an executable dependency host is not just a media source. It is effectively part of the code-signing story for the runtime.

## Layer 8: file safety and runtime safety work together

Not every risky browser path comes through the main runtime bootstrap.

Files such as HTML, SVG, XML, and related formats can become execution surfaces if they are served loosely. Vibecodr therefore protects user file serving as a separate layer. Restrictive CSP, `nosniff`, and content-type-aware handling reduce the chance that content meant to be viewed as content turns into an accidental bypass around the intended runtime path.

## What this adds up to

The browser runtime is not one defense. It is a stack of them.

Origin separation keeps the runtime out of the main app's trust plane. Wrapper and control-shell separation keep runtime responsibilities from drifting upward. Bridge validation keeps host mediation from becoming a blind proxy. Capability gating keeps powerful browser APIs from being silently pre-authorized. Service-worker blocking prevents persistent footholds. CSP and network policy keep web openness from turning into platform trust. Dependency review keeps executable trust deliberate. Safe file serving protects the edges.

Put together, those layers are what let Vibecodr host real browser code without pretending that real browser code is harmless.
