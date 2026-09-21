# What a Maypop app is

## Short definition

A Maypop app is a static browser application that Maypop packages, publishes, and opens inside a sandboxed iframe. The app can be completely standalone, or it can opt into Maypop's browser SDK for authenticated identity, durable shared data, files, AI, integrations, collaboration, sharing, and notifications.

Maypop therefore combines two roles:

1. A static application deployment and versioning platform.
2. An opinionated backend as a service for apps running inside the Maypop host.

The second role does not turn the app into a conventional server application. The app still executes in the browser. Maypop operates the backend services and exposes them through the authenticated SDK session.

## The three layers

| Layer | Responsibility |
| --- | --- |
| App bundle | HTML, CSS, browser JavaScript, images, and other static assets |
| Maypop host | Sandboxed iframe, account authentication, session handshake, app identity chrome, theme, permissions, navigation, sharing, and app lifecycle |
| Maypop services | Identity, shared KV, file storage, AI, agents, integrations, notifications, app roster, and live sessions |

This distinction answers “does it have a backend?” precisely:

- A standalone Maypop app may have no backend behavior.
- An SDK-enabled Maypop app uses Maypop's managed backend services.
- An existing full-stack application does not bring its server into Maypop merely by publishing its frontend.

## Runtime and packaging

Studio apps are file workspaces with `index.html` as the entry point. Studio assembles local files into the iframe preview and can compile browser JSX or TSX during authoring.

CLI-managed apps may use Vite, Rsbuild, statically exported Next.js, or plain static files. Publishing uploads the generated static output. There is no long-running app-owned process after publication.

This excludes runtime-only features such as:

- Next.js route handlers, server actions, middleware, or SSR that needs a server.
- Direct Node filesystem or native module access.
- An app-owned SQL database connection.
- Server-held API keys or secrets.
- Background workers, cron jobs, and inbound webhooks hosted by the app.

Those features must be replaced with Maypop services, moved into browser-safe logic, or kept in a separately hosted backend.

## Why it is a BaaS

Maypop provisions platform capabilities around the app rather than asking the app author to operate infrastructure:

- The host supplies the current viewer's app-scoped identity and permissions.
- Every app can have a shared JSON key-value store.
- Every app can have durable file storage.
- AI and agent calls use the viewer's Maypop access instead of an app-owned provider key.
- Connected MCP services run through platform-held credentials; app code never receives those credentials.
- Notifications and share flows use the app's audience and Maypop shell.
- Live collaboration is available through KV synchronization and optional ephemeral multiplayer sessions.

“Database” in product explanations usually means the app's managed KV store, not a general relational database. Do not promise SQL, arbitrary queries, server functions, or background execution unless a current Maypop contract explicitly provides them.

## Identity and audience

The app is the unit of ownership. It owns its identity, versions, data, files, sessions, and release state.

Maypop owns account authentication and presents the viewer's account in the surrounding shell. Apps consume the resulting app-scoped identity; they do not normally implement login, signup, password recovery, Google authentication, or a second account menu. Anonymous link access is handled through the same host session model, with contextual host sign-in available through the SDK when it would grant more capability.

A group is one audience to which the app can be published. Directly shared users and link visitors can be other audiences. These entry paths do not create separate copies of the app or separate databases.

The current viewer identity is pseudonymous and scoped to the app. Its id is suitable for authorship and per-person keys inside that app, but it is not the person's global Maypop user id.

## Data semantics

App data is shared by default. Everyone with suitable access reaches the same app-owned KV store and Drive. Per-person data is modeled explicitly, commonly by including the current app-scoped user id in the key and declaring an appropriate saved-data policy.

Use KV subscriptions for rendered state that must remain current. One-shot reads can be stale while local data hydrates. Use multiplayer only when the feature needs ephemeral presence, session lifecycle, or low-latency messages; save anything that must survive the session to KV.

## Release model

Publishing creates immutable app versions. One live revision is selected for the app everywhere it appears. Publishing the app into another group changes its audience, not its code or release state.

Studio uses “iteration” for an authoring cycle in the draft workspace and “version” for a published artifact. Reverting an iteration creates another iteration; publishing releases the current state as a version.

An app may permit remixing. A remix becomes a new app with its own identity, data, release history, and author while retaining lineage to its source.

## Useful comparisons

| Maypop app | Not equivalent to |
| --- | --- |
| Static frontend hosted and versioned by Maypop | Container, VM, or general serverless runtime |
| Managed SDK capabilities | An app-owned backend process |
| App-owned shared data | One database per group or per viewer |
| Host-provided identity | A custom login page inside every app |
| Immutable published versions | Mutable deployment directory |
| Remix into a new app | Editing the source app in place |

## Recommended plain-language explanation

“A Maypop app is a web app that runs inside Maypop. Maypop deploys the static frontend and gives it an authenticated SDK for common backend needs such as shared data, files, AI, integrations, and notifications. You build the product UI and data model without operating a server, but Maypop is not hosting your own Node or database process. Server-only features must be redesigned around the SDK or remain in an external backend.”
