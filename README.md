# Maypop App Agent Skill

A portable Agent Skill for explaining, assessing, designing, and adapting applications for Maypop.

The skill gives an AI coding harness the platform model it needs to answer a deceptively important question correctly: **what is a Maypop app?** It distinguishes Maypop's static browser runtime from a conventional application server and explains how Maypop's SDK provides backend-as-a-service capabilities.

## What it covers

- The Maypop app, host, and platform-service layers.
- Host-owned authentication and desktop/mobile application chrome.
- Static bundle and sandboxed iframe constraints.
- Identity, permissions, KV, Drive, AI, agents, members, multiplayer, MCP, sharing, and notifications.
- Local Vite, Rsbuild, and Next.js sandbox setup for testing identity, KV, and Drive without deploying.
- App-owned data, audiences, versions, iterations, and remixes.
- Compatibility assessment for existing applications.
- Migration of common backend responsibilities to Maypop services.
- `maypop.toml`, static framework builds, and publishing concepts.
- CLI installation from verified release binaries or Cargo, authentication, `maypop init`, metadata application, publication, and optional profiles.
- The boundary between Maypop-native capabilities and a separately hosted backend.

## Install

Clone or download this repository, then copy or symlink the repository folder into your harness's skill directory under the name `maypop-app`:

```sh
git clone <repository-url> maypop-app
```

Skill discovery locations vary by harness. Point the harness at the directory containing `SKILL.md`; do not point it only at `references/`.

For repository-scoped harnesses that follow the common Agent Skills layout, place it at:

```text
<project>/.agents/skills/maypop-app/
```

For a user-wide installation, consult the harness's documentation for its personal skills directory and place the same folder there.

## Use

Invoke `maypop-app` explicitly if the harness supports named skill invocation, or ask a matching question such as:

- “What is a Maypop app?”
- “Does this project have a backend that Maypop can run?”
- “Assess this Next.js application for Maypop compatibility.”
- “Replace this app's authentication and database with Maypop services.”
- “Prepare this Vite application for Maypop publishing.”
- “Add the local Maypop sandbox to this Rsbuild app.”
- “Install the Maypop CLI for this machine.”
- “Authenticate the Maypop CLI and initialize this repository.”
- “Update this app's metadata from `maypop.toml` and publish a version.”

The skill intentionally avoids harness-specific tools and commands. For exact SDK implementation, it instructs the harness to inspect the installed `@basilica-digital/maypop-sdk` declarations instead of guessing from a possibly stale summary.

## Structure

```text
maypop-skill/
├── SKILL.md
├── README.md
├── LICENSE
└── references/
    ├── app-model.md
    ├── cli.md
    ├── host-environment.md
    └── sdk-and-publishing.md
```

`SKILL.md` contains routing and non-negotiable platform invariants. The references provide progressive detail only when a request needs it.

## Maintenance

Update the references when Maypop's product model changes. Keep exact method signatures and volatile limits out of the skill unless they are essential; the live SDK declarations remain authoritative.

## License

Apache-2.0
