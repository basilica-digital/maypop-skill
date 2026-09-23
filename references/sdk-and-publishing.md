# Maypop SDK and publishing

Use this reference for architecture, implementation planning, compatibility assessment, or publication. It is a capability map, not a substitute for the current typed SDK contract.

## SDK lifecycle

This reference targets Maypop SDK v1.2. The browser SDK is published as `@basilica-digital/maypop-sdk` and is also available through the hosted `/sdk/v1.js` script, depending on the project. Both forms expose the same `window.maypop` object.

Always wait for the host handshake:

```js
await maypop.ready();
```

Only then read identity, mode, permissions, theme, or other capabilities. The session is real in Studio preview as well as in a published app; preview writes are not disposable mocks.

This differs from the local framework host described below. Studio preview is attached to real Maypop services; the default sandbox uses development-only identity, audience, KV, Drive, MCP, sharing, and notification behavior on the developer's machine. Hybrid and connected modes can opt into authenticated services.

Exact types and signatures are defined by the current SDK declarations. Before implementing a capability, confirm that a package-based app uses v1.2, then inspect its installed `@basilica-digital/maypop-sdk` types, a locally provided SDK source, or the host's `/sdk/v1.d.ts`. Do not guess.

## Capability map

| Surface | Purpose | Important semantics |
| --- | --- | --- |
| `maypop.user`, `mode`, `permissions` | Viewer identity and access | Identity is pseudonymous and app-scoped; permissions can change at runtime |
| `maypop.kv` | JSON-serializable persistent records | One shared store per app; subscribe for rendered/live state |
| `maypop.drive` | Durable app files | Store file identity or content id, not temporary signed or object URLs |
| `maypop.ai` | Decisions (`decide`), chat and streaming, image, video, audio, and transcription calls | Uses platform access; capability and limits belong to the viewer/session |
| `maypop.agent` | Persistent or ephemeral conversational agents with tools | Give agents explicit tools over app data rather than implicit authority |
| `maypop.members()` | App roster and recent activity | Roster belongs to the app's whole audience, not one group |
| `maypop.apps` | Ask the host to open another known app | Does not enumerate apps; host may refuse |
| `maypop.multiplayer` | Ephemeral rooms and peer-to-peer sessions | Use KV for durable state; use multiplayer for session state or low-latency traffic |
| `maypop.mcp` | User-connected external services | Discover servers and tool schemas at runtime; never hardcode ids or credentials |
| `maypop.link` | Server-side URL preview/unfurling | Useful where iframe CORS prevents fetching arbitrary pages |
| `maypop.share` | Host share card and deep links | May require the app to be published and shared first |
| `maypop.notify` | Maypop and push notifications | Targets only people in the app's audience and is rate-limited |

## Authentication and authorization

Do not build a second login page, password flow, or Google/social authentication by default. The Maypop host owns account authentication and the SDK provides the current app-scoped viewer. The surrounding Maypop shell already exposes global account identity, so do not add a redundant account menu merely to show who is logged in.

Treat SDK roles as presentation hints. Gate mutation controls on current mode or scopes, handle permission failures, and let the Maypop backend enforce authorization. Listen for session or mode changes when a long-lived UI needs to react.

Anonymous link visitors can be read-only even when signing in would grant write access. When `maypop.signInRequired` indicates that an account would help, offer a contextual action and use the host sign-in flow exposed by the SDK; do not collect credentials inside the app. Do not prompt when signing in would leave the session read-only.

## Choosing storage

Use `maypop.kv` for JSON records, settings, votes, lists, scores, references, and other structured state. Use `maypop.drive` for images, audio, video, documents, and other byte content.

Do not replace shared or cross-device state with `localStorage`. Local browser storage remains appropriate only for disposable device-local UI preferences or drafts whose loss and lack of sharing are intentional.

For per-user data, use the app-scoped viewer id in the key design and provide a saved-data policy that restricts access as intended. Never assume an app-owned store is private to the writer merely because the key contains a user id.

## Choosing an AI call

Use `maypop.ai.decide()` when the answer is one of a fixed set: classify input, route a message to a screen, gate a destructive action, or verify and rank something a chat model produced. It asks a decision model closed questions (yes/no, pick one, rate on an ordered scale) about a `state` and resolves with typed answers and calibrated probabilities. Expect roughly 70–500 ms per call and a small fraction of a chat call's cost, so it can run on every interaction without an explicit user gesture. It never returns free text and does not stream; there is no `model` field because the platform pins the decision model. The upstream model is in preview, so new optional response fields may appear.

Use `maypop.ai.chat()` or `maypop.ai.stream()` for generated text, and `maypop.agent` when the assistant needs tools over app data or a persistent conversation. Do not ask a chat model for a JSON label, boolean, or score; that is a decision. Pair the two: decide first, then generate.

Tune each probability threshold to the cost of that mistake rather than 0.5: act on a low probability when gating something irreversible, and require a high one before auto-applying a label the user cannot easily undo. When a `choice` answer's top two `probabilities` are close, show both or ask instead of picking silently.

`decide` is gated by the same `ai:use` scope and limits as `chat`, so it fails with the same `maypop/ai-limit`, `maypop/sign-in-required`, and `maypop/forbidden` errors. The pure local sandbox never grants `ai:use`; test decisions in hybrid mode with the `ai` remote capability.

## Develop with the local host

The `@basilica-digital/maypop-sdk` package includes local host integrations for Vite, Rsbuild, and Next.js. They let an app use the normal SDK handshake and exercise identity, members, KV, Drive, agents, multiplayer, MCP, sharing, and notification inspection through the framework's ordinary development server. CLI authentication, `maypop init`, and deployment are not required for the default local loop.

Inspect the project's manifest and lockfile first. If the SDK is absent or older than v1.2, install or update it with the project's existing package manager so the manifest and lockfile stay in sync. Keep it as an application dependency when browser code imports it. For example, the current v1.2 release is:

```sh
pnpm add @basilica-digital/maypop-sdk@1.2.0
```

Use the equivalent `npm`, Yarn, or Bun command when that is what the project already uses. Do not introduce a second package manager merely to add the SDK.

Application code does not need a sandbox branch:

```ts
import { maypop } from "@basilica-digital/maypop-sdk";

await maypop.ready();
const records = await maypop.kv.list({ prefix: "record/" });
```

Configure the matching development host.

### Vite

```ts
import { maypop } from "@basilica-digital/maypop-sdk/vite";
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [maypop()],
});
```

### Rsbuild

```ts
import { defineConfig } from "@rsbuild/core";
import { maypop } from "@basilica-digital/maypop-sdk/rsbuild";

export default defineConfig({
  plugins: [maypop()],
});
```

### Next.js

```ts
import { withMaypop } from "@basilica-digital/maypop-sdk/next";

export default withMaypop({
  output: "export",
});
```

`withMaypop` adds its proxy only during `next dev`; production builds retain the static-export configuration. The Vite and Rsbuild plugins likewise apply only to their development servers. Run the framework's normal development command and open its normal URL. The local Maypop host wraps the app and preserves routed deep links.

The default sandbox viewer is a synthetic signed-in admin named `Developer`. Local state persists across server restarts in:

- `.maypop/config.json` for the generated app and viewer identity.
- `.maypop/kv.json` for KV data.
- `.maypop/drive/` for Drive metadata and file bytes.
- `.maypop/notifications.json` for captured notification sends.

Ignore this generated state. If the project also commits `.maypop/kv-policy.json`, do not ignore the entire directory; use selective entries such as:

```gitignore
.maypop/.lock
.maypop/config.json
.maypop/dev.json
.maypop/kv.json
.maypop/drive/
.maypop/notifications.json
```

Vite and Rsbuild accept sandbox options directly. Next.js accepts them as the second `withMaypop` argument:

```ts
maypop({ username: "Alice", dataDirectory: ".maypop/alice" });
withMaypop(nextConfig, { username: "Alice", dataDirectory: ".maypop/alice" });
```

Use separate data directories to test isolated local identities or datasets. One directory cannot be written by two development servers at once.

The local host provides configurable identity and audience fixtures, KV, Drive, mock MCP tools, a local Share card, and captured notifications. Open `/_maypop/notifications` to inspect whom each `maypop.notify()` call addressed, its content, and its deep link. Captured calls never prove real delivery.

Use `.maypop/dev.json` in sandbox mode to exercise read-only, anonymous, and signed-in flows without application-only branches:

```json
{
  "mode": "sandbox",
  "viewer": {
    "username": "Guest",
    "role": "reader",
    "anonymous": true,
    "signInGrantsWrite": true,
    "signedInUsername": "Alice",
    "signedInRole": "writer"
  },
  "members": [
    { "username": "Bob", "role": "editor", "connected": true }
  ],
  "guestCount": 2,
  "strictStorage": true
}
```

When `signInGrantsWrite` is true, the normal SDK sign-in request transitions the local fixture and reloads the app. Local KV writes always validate production size limits; `strictStorage` additionally applies the committed `.maypop/kv-policy.json` rules. Without it, sandbox storage remains permissive except for the viewer's read or write scope.

Mock MCP servers live in developer-local `.maypop/mcp.json`. Define each server's id, name, mock URL, and tools plus an optional static `result`; calls without one echo their arguments. This is for deterministic local integration testing, not emulating real external services.

The local Share card clearly labels its URL as unpublished. Notification deep links can be reopened inside the local host from the inspector. These simulators validate app behavior but not Maypop delivery or host-shell presentation.

### Authenticated and connected development

`.maypop/dev.json` is a developer-local opt-in. Never put credentials in it and do not commit it: its mode can use a developer's AI allowance or real app data.

Hybrid mode keeps KV and Drive local while forwarding selected capabilities through a valid CLI profile. Confirm the selected profile with `maypop status`; run `maypop auth` only when that check reports no valid authentication:

```json
{
  "mode": "hybrid",
  "profile": "dev",
  "remoteCapabilities": ["ai", "members", "link", "mcp", "multiplayer"],
  "notifications": "inspect"
}
```

`ai` covers chat, streaming, decisions, images, video, audio, transcription, and agent model calls and consumes the selected account's real allowance. Agent execution stays in the local page while its model calls use the authenticated AI capability. `members` reads the real app roster. `link` performs server-side URL unfurling. `mcp` exposes the app's real linked integrations; connect and link them with `maypop mcp connect` and `maypop mcp link`. `multiplayer` uses the real ephemeral room service and also exposes the real member roster needed for peer display. KV, Drive, sharing, and notifications remain local. Do not configure real `mcp` and `.maypop/mcp.json` fixtures together in hybrid mode.

Connected mode skips local KV/Drive initialization and sends the app API surface to the real app while preserving the local host handshake:

```json
{
  "mode": "connected",
  "profile": "dev",
  "notifications": "inspect"
}
```

The repository must already be connected by `maypop init`, and the selected profile must have access to the app. Confirm that profile with `maypop status` before starting connected development; authenticate only if the check fails. Omit `profile` to use normal CLI selection, including `MAYPOP_PROFILE` and repository API URL matching. The Node development host reads the owner-only CLI profile store, mints an app-scoped session itself, and never exposes the long-lived CLI credential or refresh token to the iframe. Connected mode forwards linked MCP discovery and tool calls with the rest of the app API without another capability flag.

Real notification delivery requires connected mode plus `"notifications": "live"`. Use `"disabled"` to remove notification permission. The safe default in every mode is `"inspect"`.

Capabilities that require navigation in the surrounding Maypop product shell, such as `maypop.apps.open()` and opening group settings, remain unavailable locally. Treat that as supported degradation and verify those interactions in Maypop before release.

The local sandbox is development tooling, not a security boundary. Bind the development server only to a trusted interface unless the project is safe to expose.

## Build compatibility

Maypop publishes static output. Current CLI adapters include Vite, Rsbuild, Next.js static export, and plain static files. Read [cli.md](cli.md) for authentication, initialization, `maypop.toml`, metadata application, profiles, and publication commands.

For Next.js, the production configuration must use static export:

```ts
const nextConfig = {
  output: "export",
};
```

The expected output is static files, normally under `out/`. A `.next` server build is not publishable as a Maypop app. Route handlers, server actions, SSR, middleware, and other server-runtime features must be removed, replaced, or hosted elsewhere.

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

- The normal development server loads the app through the local Maypop host when framework bindings are configured.
- Local identity, KV, and Drive behavior survives a development-server restart when persistence matters.
- The configured build creates the declared static entry file.
- Direct navigation and asset paths work from the published routing model.
- No required feature calls an unavailable local server route.
- SDK initialization waits for readiness and handles read-only sessions.
- Shared data and file behavior use the intended app-wide or per-user policy.
- Optional AI, agent, multiplayer, integration, share, and notification features work in the intended sandbox, hybrid, or connected mode and fail gracefully elsewhere.
- The app works at iframe dimensions and does not assume the top-level browser window.
