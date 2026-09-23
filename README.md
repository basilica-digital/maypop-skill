# Maypop App Agent Skill

A portable Agent Skill for explaining, assessing, designing, and adapting applications for Maypop.

The skill gives an AI coding harness the platform model it needs to answer a deceptively important question correctly: **what is a Maypop app?** It distinguishes Maypop's static browser runtime from a conventional application server and explains how Maypop's SDK provides backend-as-a-service capabilities.

## What it covers

- The Maypop app, host, and platform-service layers.
- Host-owned authentication and desktop/mobile application chrome.
- Static bundle and sandboxed iframe constraints.
- Identity, permissions, KV, Drive, AI, agents, members, multiplayer, MCP, sharing, and notifications.
- Maypop SDK v1.2, including local Vite, Rsbuild, and Next.js sandbox setup for testing identity, KV, and Drive without deploying.
- App-owned data, audiences, versions, iterations, and remixes.
- Compatibility assessment for existing applications.
- Migration of common backend responsibilities to Maypop services.
- `maypop.toml`, static framework builds, and publishing concepts.
- CLI installation from verified release binaries or Cargo, status-first authentication, `maypop init`, metadata application, publication, and optional profiles.
- Authenticated image, audio, and video generation through `maypop ai` when a harness has no native media tools.
- The boundary between Maypop-native capabilities and a separately hosted backend.

## Install

The recommended installer is the community-maintained
[`skills` CLI](https://github.com/vercel-labs/skills). It discovers the
`maypop-app` skill at this repository's root and can keep the installation up
to date.

Install it globally for Codex:

```sh
npx skills add basilica-digital/maypop-skill \
  --skill maypop-app \
  --global \
  --agent codex
```

Install the same managed copy for both Codex and Claude Code:

```sh
npx skills add basilica-digital/maypop-skill \
  --skill maypop-app \
  --global \
  --agent codex \
  --agent claude-code
```

Update a global installation later with:

```sh
npx skills update maypop-app --global
```

Use `npx skills list` to inspect installed skills. In Codex, run `/skills` or
type `$maypop-app` to verify discovery. In Claude Code, invoke
`/maypop-app`. Restart the harness if a new or updated skill does not appear.

### Project-scoped installation

Omit `--global` to install into the current project. Select one or more target
agents as needed:

```sh
npx skills add basilica-digital/maypop-skill \
  --skill maypop-app \
  --agent codex
```

### Pinned manual installation

To pin an exact release without the third-party installer, clone the tag into
the personal skill directory for the harness. For Codex:

```sh
mkdir -p ~/.agents/skills
git clone --branch v1.5.0 --depth 1 \
  https://github.com/basilica-digital/maypop-skill.git \
  ~/.agents/skills/maypop-app
```

For Claude Code, use `~/.claude/skills/maypop-app` instead. Manually cloned
copies are updated with Git rather than `npx skills update`.

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
- “Generate an app hero image with Maypop AI.”
- “Create an audio loop for this game when no audio tool is available.”
- “Update this app's metadata from `maypop.toml` and publish a version.”

The skill intentionally avoids harness-specific tools and commands. It targets `@basilica-digital/maypop-sdk` v1.2 and instructs the harness to inspect the installed declarations instead of guessing from a possibly stale summary.

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
