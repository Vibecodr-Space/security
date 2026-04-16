# Secrets, Egress, and Data Handling

Last updated: 2026-04-15

This is the part of the security story that answers a practical question every builder eventually asks: when my code talks to other systems, what keeps that from becoming a leak surface?

## Secrets are protected in two different moments

Good secret handling is not only about storage. It is also about runtime exposure.

Vibecodr protects secrets at rest by encrypting them. But it also tries to protect them in motion by designing the common secret-usage path so the raw secret does not have to become an ordinary variable inside user-authored code.

That distinction matters. A secret can be encrypted perfectly in a database and still be easy to leak later if the platform hands the plaintext value back into the runtime without restraint.

## Write-only management is intentional

User-facing secret management is designed so saved values are not casually re-exposed later.

The platform returns metadata that is useful for managing a secret, such as whether a value exists or when it was updated, while avoiding a pattern where the original secret value keeps reappearing in ordinary browser or API flows after it has already been stored.

## Opaque token references reduce runtime exposure

One of Vibecodr's stronger design choices is that the user runtime can work with opaque secret references rather than raw values.

In that model, the runtime gets something it can place into a request flow, but that placeholder is not the underlying secret itself. The trusted side resolves the placeholder and performs the real injection when the outbound request is prepared. That keeps the runtime usable while making it much harder for a function to accidentally print, serialize, or forward the true credential.

## Secret use can be constrained by destination

A secret is not equally safe everywhere it can be sent.

That is why Vibecodr supports the idea of host restrictions around secret use. If a secret is intended for one provider, the safer design is to bind it to the destinations where it is supposed to travel rather than assume the runtime should be able to send it to any host on the internet. This is a defense-in-depth layer against exfiltration and simple misuse.

## Egress is treated as a first-class control surface

Backend runtimes often get into trouble through outbound traffic long before they break through an inbound auth layer.

Vibecodr therefore treats outbound fetch as something that needs its own policy layer. The outbound path is not just "make an HTTP request and hope." It is a place where the platform can say no to self-invocation, no to internal infrastructure targets, no to metadata-style or localhost-style abuse paths, and no to redirect tricks that start at an allowed host and end somewhere privileged.

## Redirects matter as much as first-hop targets

One of the easiest ways to weaken a host blocklist is to validate only the first URL and then blindly follow redirects.

Vibecodr accounts for that by treating redirect handling as part of the security boundary. Each hop matters. A request that begins on a harmless hostname but redirects into an internal or blocked destination should still be treated as unsafe.

## Observability is part of egress control

The platform also treats outbound traffic as something that should remain attributable.

Requests are tagged and measured so operators can tell which runtime path reached for which host and can investigate suspicious behavior without relying on guesswork. This is useful not only for incident response, but also for understanding whether protective rules are working as intended.

## File and data handling stay scoped

The same pattern shows up in data handling more broadly. Vibecodr tries to keep storage and file access attached to the user, Pulse, capsule, or artifact that actually owns the data rather than treating raw object access as a generic capability.

On file-serving paths, the platform adds another layer by serving scriptable content types with restrictive headers so that content storage does not quietly become a second execution environment. On shared-reference paths, tokenized access and scoped ownership checks help keep download and sharing behavior attached to the right object and the right caller.

## The practical outcome

Put together, these choices mean a Pulse can do useful external work without being handed an unrestricted network pipe and a pile of plaintext credentials.

Secrets stay encrypted at rest. Runtime use favors server-side injection. Host restrictions can narrow where a secret may travel. Outbound traffic is checked against platform policy, including redirect targets. File and object access stay scoped and policy-aware. That is the difference between "backend code can call APIs" and "backend code can do anything with everything."
