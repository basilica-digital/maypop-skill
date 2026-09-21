# Maypop SDK and publishing

Use this reference for architecture, implementation planning, compatibility assessment, or publication. It is a capability map, not a substitute for the current typed SDK contract.

## SDK lifecycle

Maypop's browser SDK is available through the `maypop-sdk` package or the hosted browser script, depending on the project. Both forms expose the same `window.maypop` object.

Always wait for the host handshake:

```js
await maypop.ready();
```

Only then read identity, mode, permissions, theme, or other capabilities. The session is real in Studio preview as well as in a published app; preview writes are not disposable mocks.

Exact types and signatures are defined by the current SDK declarations. Before implementing a capability, inspect the installed `maypop-sdk` types, a locally provided SDK source, or the host's `/sdk/v1.d.ts`. Do not guess.

## Capability map

| Surface | Purpose | Important semantics |
| --- | --- | --- |
| `maypop.user`, `mode`, `permissions` | Viewer identity and access | Identity is pseudonymous and app-scoped; permissions can change at runtime |
| `maypop.kv` | JSON-serializable persistent records | One shared store per app; subscribe for rendered/live state |
| `maypop.drive` | Durable app files | Store file identity or content id, not temporary signed or object URLs |
| `maypop.ai` | Model, image, video, audio, and transcription calls | Uses platform access; capability and limits belong to the viewer/session |
| `maypop.agent` | Persistent or ephemeral conversational agents with tools | Give agents explicit tools over app data rather than implicit authority |
| `maypop.members()` | App roster and recent activity | Roster belongs to the app's whole audience, not one group |
| `maypop.apps` | Ask the host to open another known app | Does not enumerate apps; host may refuse |
| `maypop.multiplayer` | Ephemeral rooms and peer-to-peer sessions | Use KV for durable state; use multiplayer for session state or low-latency traffic |
| `maypop.mcp` | User-connected external services | Discover servers and tool schemas at runtime; never hardcode ids or credentials |
| `maypop.link` | Server-side URL preview/unfurling | Useful where iframe CORS prevents fetching arbitrary pages |
| `maypop.share` | Host share card and deep links | May require the app to be published and shared first |
| `maypop.notify` | Maypop and push notifications | Targets only people in the app's audience and is rate-limited |

## Authentication and authorization

Do not build a second login page by default. The Maypop host owns sign-in and the SDK provides the current viewer.

Treat SDK roles as presentation hints. Gate mutation controls on current mode or scopes, handle permission failures, and let the Maypop backend enforce authorization. Listen for session or mode changes when a long-lived UI needs to react.

Anonymous link visitors can be read-only even when signing in would grant write access. Use the host sign-in flow exposed by the SDK; do not collect credentials inside the app.

## Choosing storage

Use `maypop.kv` for JSON records, settings, votes, lists, scores, references, and other structured state. Use `maypop.drive` for images, audio, video, documents, and other byte content.

Do not replace shared or cross-device state with `localStorage`. Local browser storage remains appropriate only for disposable device-local UI preferences or drafts whose loss and lack of sharing are intentional.

For per-user data, use the app-scoped viewer id in the key design and provide a saved-data policy that restricts access as intended. Never assume an app-owned store is private to the writer merely because the key contains a user id.

## Publishing with the CLI

The project configuration is `maypop.toml`:

```toml
[app]
name = "My app"
description = "What this app does"
visibility = "private"
link_access = "request"
allow_remixing = true
tags = []

[build]
framework = "vite"
```

The app table describes metadata. Applying metadata and publishing code are separate operations. The build table selects how the CLI produces static output.

Current adapters include Vite, Rsbuild, Next.js static export, and plain static files. Custom build command, output directory, and entry file can override adapter defaults.

For Next.js, the production configuration must use static export:

```ts
const nextConfig = {
  output: "export",
};
```

The expected output is static files, normally under `out/`. A `.next` server build is not publishable as a Maypop app. Route handlers, server actions, SSR, middleware, and other server-runtime features must be removed, replaced, or hosted elsewhere.

The normal CLI lifecycle is conceptually:

1. Authenticate the CLI.
2. Initialize the project and app identity.
3. Commit a clean Git revision.
4. Build static output.
5. Publish it as the next immutable app version.

Do not run remote or state-changing steps without the user's authorization.

## Compatibility decision

Classify a project using these outcomes:

### Already compatible

The production output is static, browser-only, and contains no required server behavior. It may publish directly after build verification.

### Convertible to Maypop-native

The UI is browser-compatible, and server responsibilities map cleanly to Maypop identity, KV, Drive, AI, agents, multiplayer, MCP, sharing, or notifications. Explain the data-model and permission changes before editing.

### Requires a separate backend or product redesign

The app depends on arbitrary SQL queries, trusted secret-bearing operations, background jobs, inbound webhooks, custom server libraries, or server rendering that cannot become static. Keep those services external or change the product behavior; `maypop.toml` alone does not solve them.

## Verification

At minimum, verify:

- The configured build creates the declared static entry file.
- Direct navigation and asset paths work from the published routing model.
- No required feature calls an unavailable local server route.
- SDK initialization waits for readiness and handles read-only sessions.
- Shared data and file behavior use the intended app-wide or per-user policy.
- Optional AI, multiplayer, integration, share, and notification features fail gracefully when unavailable.
- The app works at iframe dimensions and does not assume the top-level browser window.
