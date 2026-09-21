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

The repository root is the skill directory. Install the tagged release into the
personal skills directory for your harness.

### Codex

```sh
mkdir -p ~/.agents/skills
git clone --branch v1.0.0 --depth 1 \
  https://github.com/basilica-digital/maypop-skill.git \
  ~/.agents/skills/maypop-app
```

OpenAI's `$skill-installer` can also install the repository for personal use.
Run `/skills` or type `$maypop-app` to verify discovery. Codex normally detects
new skills automatically; restart it if the skill does not appear.

### Claude Code

```sh
mkdir -p ~/.claude/skills
git clone --branch v1.0.0 --depth 1 \
  https://github.com/basilica-digital/maypop-skill.git \
  ~/.claude/skills/maypop-app
```

Invoke it explicitly with `/maypop-app`, or let Claude select it when a request
matches its description. Restart Claude Code if the skills directory did not
exist when the session started.

### Project-scoped installation

To share the skill with a repository instead, place the same folder at the
harness-specific project path and commit it or add it as a Git submodule:

```text
Codex:       <project>/.agents/skills/maypop-app/
Claude Code: <project>/.claude/skills/maypop-app/
```

Other Agent Skills-compatible harnesses use their own discovery paths. Point
the harness at the directory containing `SKILL.md`, not only at `references/`.

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
