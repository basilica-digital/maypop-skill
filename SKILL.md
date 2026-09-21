---
name: maypop-app
description: Explain, assess, design, develop, adapt, or publish web applications for Maypop's static runtime, platform SDK, local framework sandbox, and `maypop` CLI. Use when a user asks what a Maypop app is, whether an existing project is compatible, how Maypop's BaaS works, how to test SDK capabilities locally, how to use `maypop.toml`, or how to authenticate, initialize, configure, or publish with the CLI. Do not use for unrelated flower questions or for development of Maypop's own platform internals.
license: MIT
---

# Maypop apps

Help users understand and build for Maypop without confusing it with a general-purpose application server.

## Start with the platform model

For conceptual questions, lead with this definition in language appropriate to the user:

> A Maypop app is a browser application packaged as static files and run by Maypop in a sandboxed iframe. Maypop hosts and versions the frontend, while optional backend-like features come from the `maypop` browser SDK. It is not a runtime for an app-owned Node, Python, Rust, or database server.

Read [references/app-model.md](references/app-model.md) before explaining the model, assessing compatibility, or proposing architecture.

Do not call every Maypop app “full stack.” A standalone static app may use no SDK. An attached app may use Maypop's backend-as-a-service capabilities. Say which case applies.

## Route the request

- For an explanation, distinguish app code, the Maypop host, and Maypop platform services. Correct misconceptions directly.
- For compatibility assessment, inspect the supplied project and classify it as already compatible, convertible, or dependent on a separate backend.
- For interface or product design, read [references/host-environment.md](references/host-environment.md) so the app complements Maypop's authentication and surrounding chrome instead of duplicating them.
- For architecture, implementation, or local SDK testing, also read [references/sdk-and-publishing.md](references/sdk-and-publishing.md). Prefer the `maypop-sdk` framework sandbox for Vite, Rsbuild, and Next.js projects that need to exercise identity, KV, or Drive before deployment.
- For CLI authentication, initialization, `maypop.toml`, metadata, profiles, or publishing, read [references/cli.md](references/cli.md). Use the installed command's `--help` output as the final authority on available flags.
- For exact SDK code, inspect the current `maypop-sdk` declarations or the host's `/sdk/v1.d.ts` contract before writing code. Never infer method names or signatures from this skill's summary.
- For publication, inspect `maypop.toml`, framework configuration, Git state, and build output. Authentication, initialization, metadata application, and publication change local or remote state; do not run them without explicit authorization.

## Assess an existing application

Separate the application into browser code and server dependencies. Look for server routes, server actions, middleware, Node built-ins, filesystem access, databases, secrets, background jobs, webhooks, and server-render-only behavior.

Map requirements deliberately:

| Existing requirement | Maypop-native direction |
| --- | --- |
| Login and user identity | Host session and `maypop.user`; never add a second login system by default |
| Shared structured state | `maypop.kv` and an explicit saved-data access policy |
| User or app files | `maypop.drive` |
| Model calls or an in-app assistant | `maypop.ai` or `maypop.agent` |
| App audience and recent presence | `maypop.members()` |
| Ephemeral live sessions | `maypop.multiplayer` |
| External connected services | `maypop.mcp`, discovered at runtime |
| App notifications | `maypop.notify` |
| Arbitrary server computation, SQL, secrets, jobs, or webhooks | Redesign for Maypop services or retain a separately hosted backend |

Do not claim a project is compatible merely because it contains `maypop.toml`. Confirm its production build is static and that no required feature depends on a server process.

For a framework project, report:

1. What can publish unchanged.
2. What can move to the Maypop SDK.
3. What needs redesign or an external backend.
4. The smallest verification needed to prove the result.

## Preserve Maypop invariants

- The app owns its identity, data, files, versions, sessions, and release state. A group is an audience, not an owner or a separate deployment.
- Maypop owns account authentication. Do not add an app-level login, signup, password, or social-auth flow by default; use the SDK's host sign-in path when an anonymous visitor needs an account.
- Maypop's shell already presents global app identity, authoring, sharing, navigation, and account controls. Product apps should avoid duplicating that chrome, while still using headings and app-specific actions when they help the content.
- The app has one shared data space wherever it is opened. Per-person records require keys and policies scoped to the app-specific `maypop.user.id`.
- Await `maypop.ready()` before using the SDK. Treat identities as app-scoped and pseudonymous.
- Gate mutation UI on live permissions or mode. Server enforcement remains authoritative.
- Use durable Maypop storage instead of `localStorage` for shared or cross-device state.
- Use persistent KV for durable collaboration and multiplayer only for ephemeral presence or low-latency sessions.
- Feature-detect optional integrations and degrade gracefully.
- Treat Studio iterations as editing history and published versions as released artifacts; do not use the terms interchangeably.

## Communicate uncertainty precisely

The platform can evolve. Stable concepts belong in explanations; exact limits, methods, supported adapters, and CLI behavior require the current local SDK or Maypop source. If that source is unavailable, state the boundary and avoid fabricating details.
