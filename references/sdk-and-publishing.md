# Maypop SDK and publishing

Use this reference for architecture, implementation planning, compatibility assessment, or publication. It is a capability map, not a substitute for the current typed SDK contract.

## SDK lifecycle

Maypop's browser SDK is published as `@basilica-digital/maypop-sdk` and is also available through the hosted browser script, depending on the project. Both forms expose the same `window.maypop` object.

Always wait for the host handshake:

```js
await maypop.ready();
```

Only then read identity, mode, permissions, theme, or other capabilities. The session is real in Studio preview as well as in a published app; preview writes are not disposable mocks.

This differs from the local framework sandbox described below. Studio preview is attached to real Maypop services; the local sandbox uses development-only identity, KV, and Drive data on the developer's machine.

Exact types and signatures are defined by the current SDK declarations. Before implementing a capability, inspect the installed `@basilica-digital/maypop-sdk` types, a locally provided SDK source, or the host's `/sdk/v1.d.ts`. Do not guess.

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

Do not build a second login page, password flow, or Google/social authentication by default. The Maypop host owns account authentication and the SDK provides the current app-scoped viewer. The surrounding Maypop shell already exposes global account identity, so do not add a redundant account menu merely to show who is logged in.

Treat SDK roles as presentation hints. Gate mutation controls on current mode or scopes, handle permission failures, and let the Maypop backend enforce authorization. Listen for session or mode changes when a long-lived UI needs to react.

Anonymous link visitors can be read-only even when signing in would grant write access. When `maypop.signInRequired` indicates that an account would help, offer a contextual action and use the host sign-in flow exposed by the SDK; do not collect credentials inside the app. Do not prompt when signing in would leave the session read-only.

## Choosing storage

Use `maypop.kv` for JSON records, settings, votes, lists, scores, references, and other structured state. Use `maypop.drive` for images, audio, video, documents, and other byte content.

Do not replace shared or cross-device state with `localStorage`. Local browser storage remains appropriate only for disposable device-local UI preferences or drafts whose loss and lack of sharing are intentional.

For per-user data, use the app-scoped viewer id in the key design and provide a saved-data policy that restricts access as intended. Never assume an app-owned store is private to the writer merely because the key contains a user id.

## Develop with the local sandbox

The `@basilica-digital/maypop-sdk` package includes local host integrations for Vite, Rsbuild, and Next.js. They let an app use the normal SDK handshake and exercise identity, KV, and Drive through the framework's ordinary development server. CLI authentication, `maypop init`, and deployment are not required for this local loop.

Install `@basilica-digital/maypop-sdk` from npm with the project's package manager. Keep it as an application dependency when browser code imports it:

```sh
pnpm add @basilica-digital/maypop-sdk
```

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

Ignore this generated state. If the project also commits `.maypop/kv-policy.json`, do not ignore the entire directory; use selective entries such as:

```gitignore
.maypop/.lock
.maypop/config.json
.maypop/kv.json
.maypop/drive/
```

Vite and Rsbuild accept sandbox options directly. Next.js accepts them as the second `withMaypop` argument:

```ts
maypop({ username: "Alice", dataDirectory: ".maypop/alice" });
withMaypop(nextConfig, { username: "Alice", dataDirectory: ".maypop/alice" });
```

Use separate data directories to test isolated local identities or datasets. One directory cannot be written by two development servers at once.

The local host currently provides only identity, KV, and Drive scopes. Other capabilities, including AI, agents, integrations, members, multiplayer, notifications, sharing, and cross-app actions, fail with `maypop/unsupported`. Treat that as a supported degradation path, not proof that those features work in production. Test permission-sensitive and unsupported capabilities in an attached Maypop environment before release.

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

- The normal development server loads the app through the local Maypop sandbox when framework bindings are configured.
- Local identity, KV, and Drive behavior survives a development-server restart when persistence matters.
- The configured build creates the declared static entry file.
- Direct navigation and asset paths work from the published routing model.
- No required feature calls an unavailable local server route.
- SDK initialization waits for readiness and handles read-only sessions.
- Shared data and file behavior use the intended app-wide or per-user policy.
- Optional AI, multiplayer, integration, share, and notification features fail gracefully when unavailable.
- The app works at iframe dimensions and does not assume the top-level browser window.
