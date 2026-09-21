# Maypop host environment

Read this reference when designing or reviewing an app's interface. These are ownership boundaries and useful defaults, not fixed layout rules.

## The app runs inside a product shell

Maypop renders the app inside a sandboxed iframe surrounded by Maypop-owned chrome. The exact controls and responsive layout can evolve, so reason from their responsibilities instead of copying current pixels or positions.

The host commonly provides:

- App identity, including icon, title, and author or provenance.
- Global actions such as Share, app details, editing, remixing, and overflow actions when available to the viewer.
- Navigation back to Maypop surfaces and between apps.
- The viewer's Maypop account identity and account-level controls.
- Mobile safe-area handling around host navigation.

On wider screens, this currently appears as Maypop navigation around the app plus an app title bar above it. On mobile, persistent navigation collapses and the app is commonly presented under a compact in-flow bar with back, app identity, and trailing actions. The iframe receives the remaining app surface in both cases.

Do not query, style, obscure, or depend on the host DOM. The SDK and the iframe viewport are the app's contracts; the shell's element arrangement is not.

## Authentication belongs to Maypop

Maypop owns user authentication. A signed-in viewer arrives with a host session, and the app receives an app-scoped pseudonymous identity through `maypop.user` after `await maypop.ready()`.

Do not add these by default:

- Login or signup pages.
- Password collection or password recovery.
- “Continue with Google,” OAuth buttons, or another parallel social-login flow.
- A global account avatar/menu whose only purpose is to show who is signed in.

Anonymous share-link visitors can still exist. That does not transfer authentication ownership to the app. If signing in would unlock a capability, use `maypop.signInRequired` to present a contextual action such as “Sign in to save,” and call `maypop.signIn()` so the host handles the flow. If signing in would not change the viewer's permissions, keep the interface read-only instead of prompting.

Authentication to an external service is a different concern. Prefer an MCP integration connected through Maypop, where app code does not receive credentials. Add a separate third-party authorization flow only when the product explicitly requires it and the current Maypop integration model cannot provide it.

## Avoid duplicate global chrome by default

For a product-style app, begin with useful product content rather than recreating a masthead Maypop already supplies. Usually omit:

- A second app title, icon, and author row at the top of the iframe.
- A generic global Share button that duplicates Maypop's Share control.
- A global back button that returns to Maypop rather than navigating within the app.
- A signed-in-user chip, avatar menu, or account settings entry that duplicates the shell.
- Edit, remix, app details, or publication controls owned by Maypop.

This is a default, not a prohibition. Repetition is justified when the element is part of the app's content or task:

- A document, story, dashboard, or game can have its own meaningful heading.
- A branded website or marketing page can use its brand and hero title.
- Internal tabs, breadcrumbs, or back buttons can navigate the app's own information architecture.
- An object-specific action such as “Share result” or “Share this list” can call `maypop.share` with a deep path; label it specifically so it does not read as a duplicate global app-share button.
- Member avatars and profiles belong in the app when authorship, presence, assignment, or collaboration needs them—not merely to restate the current login.

Ask whether the element helps someone operate the app's content. If it only repeats a fact or action already supplied by the shell, leave it to Maypop.

## Design for the iframe, not the outer window

- Respond to the app iframe's actual width and height. Do not infer layout from the top-level window, the presence of a desktop sidebar, or a touch-device check.
- Expect the app to appear docked, expanded, fullscreen, and on narrow mobile surfaces.
- Keep all app functionality reachable when the viewport changes. Use internal scrolling deliberately and avoid placing controls under fixed app-owned overlays.
- Do not reserve space for Maypop's current header or navigation. The host lays out the iframe below or beside its own chrome.
- Follow the host theme through the SDK-provided theme state or mirrored document theme attributes when the app supports both modes.

## Review questions

Before considering an interface complete, ask:

1. Does it work without its own account system?
2. Does it avoid repeating Maypop's app identity and global actions without losing necessary content hierarchy?
3. Are any in-app back, share, profile, or settings controls clearly scoped to app content?
4. Does anonymous read-only access degrade gracefully and use host sign-in only when it would help?
5. Does the layout respond to its own iframe on desktop and mobile without relying on shell geometry?
