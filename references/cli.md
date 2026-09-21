# Maypop CLI

Read this reference when a request involves the `maypop` command, authentication, project initialization, `maypop.toml`, app metadata, profiles, or publication.

The CLI is a Rust Clap application. Prefer its self-documenting command tree over recalled syntax:

```sh
maypop --help
maypop auth --help
maypop init --help
maypop publish --help
maypop app --help
maypop profile --help
```

The public workflow consists of `auth`, `init`, `publish`, `info`, `app apply`, `status`, and `profile`. Do not teach hidden or maintainer-only commands as normal app workflows.

## Default workflow

Most people using one Maypop account and the production service do not need to think about profiles or API URLs:

```sh
maypop auth
cd my-app
maypop init
git add .
git commit -m "feat: create Maypop app"
maypop publish
```

This sequence has real side effects:

- `auth` opens a browser approval flow and saves a credential.
- `init` creates an unpublished Maypop app and changes local Git configuration.
- `publish` pushes committed source and creates an immutable published version.

Explain the effects and obtain authorization before running these commands for a user.

## Authenticate

```sh
maypop auth
```

The CLI starts a device authorization, opens Maypop in the browser, waits for the user to approve the device, and saves the resulting credential in an owner-only profiles file. Maypop's existing account authentication handles the browser side; the CLI never asks for a password or Google credential in the terminal.

Useful options:

```sh
maypop auth --no-browser
maypop auth --device-name "Work laptop"
```

`--no-browser` prints the approval URL and code without launching a browser. The current CLI flow issues a 90-day credential that can be revoked from Maypop account settings. If an old credential predates Git access or has expired or been revoked, authenticate again.

Check connectivity and the selected identity without changing the app:

```sh
maypop status
```

## Initialize a project

Run `init` inside the app directory, or pass a directory explicitly:

```sh
maypop init
maypop init ./my-app --name "My app" --description "What it does" --visibility private
```

Available initial visibility values are `private`, `unlisted`, and `public`; the default is `private`. The app name defaults to the directory name.

`maypop init`:

1. Initializes a Git repository when necessary.
2. Creates an unpublished app identity on Maypop.
3. Records the app id and API URL in repository-local Git configuration.
4. Configures the Maypop Git service as `origin` with a repository-local credential helper.
5. Creates or validates `maypop.toml` and detects the framework adapter.

It refuses to continue if the repository is already attached to a Maypop app or already has an `origin` remote. Never delete or overwrite an existing remote automatically. If the user wants to retain a GitHub remote, inspect it first and offer to rename it deliberately, for example:

```sh
git remote -v
git remote rename origin github
maypop init
```

After initialization, Maypop remains `origin`, because `maypop publish` verifies and pushes that remote. A separate `github` remote can still be used for GitHub. Renaming remotes changes local repository configuration, so obtain permission first.

Initialization does not publish a version. Commit the files normally before publishing.

## Understand `maypop.toml`

The configuration has two independent concerns:

```toml
[app]
name = "My app"
description = "What this app does"
visibility = "private"
link_access = "request"
allow_remixing = true
tags = []
# thumbnail = "assets/thumbnail.png"

[build]
framework = "vite"
```

### `[app]`: remote app metadata

- `name`: non-empty app name.
- `description`: listing or detail description.
- `visibility`: `private`, `unlisted`, or `public`.
- `link_access`: `request`, `view`, or `use`; the backend validates it together with visibility.
- `allow_remixing`: whether others may create a new app derived from this one.
- `tags`: string list.
- `thumbnail`: repository-relative image path; it must remain inside the repository.

`maypop init` starts private and unlisted apps at `link_access = "request"`; public apps start at `link_access = "view"`. Not every reach pair is valid—for example, a publicly listed app cannot use `request` link access.

Editing `[app]` does not change Maypop by itself, and `maypop publish` does not apply it implicitly. Apply the fields explicitly:

```sh
maypop app apply
```

Only fields present in `[app]` are sent; omitted fields remain unchanged. If a thumbnail is configured, `app apply` uploads it before updating the app. This command mutates remote app metadata, so review the file and obtain authorization first.

Inspect the connected remote app without changing it:

```sh
maypop info
```

### `[build]`: local static build

- `framework`: `auto`, `vite`, `next`, `rsbuild`, or `static`.
- `command`: optional argv array such as `["pnpm", "run", "build"]`, not a shell string.
- `output`: optional repository-relative build directory.
- `entry`: optional repository-relative entry file, defaulting to `index.html`.

Adapter defaults:

| Framework | Build | Output | Routing |
| --- | --- | --- | --- |
| Vite | Package manager's build script | `dist/` | SPA fallback |
| Rsbuild | Package manager's build script | `dist/` | SPA fallback |
| Next.js | Package manager's build script | `out/` | Static route files |
| Static | No build command | Repository root | SPA fallback |

Next.js must use `output: "export"`; a `.next` server build is not publishable. Build output and entry paths must stay inside the repository, and the entry file must exist.

Package-manager detection uses `packageManager` in `package.json` first, then lockfiles, and otherwise falls back to npm. Override the build fields only when the adapter defaults do not match the project.

## Publish a version

```sh
maypop publish
maypop publish --changelog "Add shared collections"
```

Publication operates on the exact committed Git `HEAD`. It:

1. Verifies the repository is connected to the expected Maypop app and `origin` is the matching Maypop Git remote.
2. Refuses a dirty worktree; commit or stash first.
3. Builds using `[build]`.
4. Refuses a build that modifies tracked files.
5. Pushes `HEAD` to the Maypop Git remote's `main` branch.
6. Uploads and validates the static build output.
7. Records the source commit and bundle as the next immutable app version.

Do not commit, stash, change remotes, or publish merely to get past a failed precondition without the user's approval. Fix the underlying build or configuration, then retry. Remember that publishing code and applying `[app]` metadata are separate operations.

## Profiles are optional

For one account on production, use plain `maypop auth`; it creates the `default` profile against the production API. Do not introduce profile ceremony into the basic workflow.

Profiles become useful for multiple accounts or environments:

```sh
maypop --profile local --url http://localhost:3000 auth
maypop --profile dev --url https://api.dev.maypop.ai auth
maypop --profile prod --url https://api.app.maypop.ai auth

maypop profile list
maypop profile set-default prod
```

Select a profile for one command with `--profile NAME` or `MAYPOP_PROFILE`. Global Clap options may appear before or after the subcommand, but use one consistent style in examples.

Each initialized repository stores its API URL locally. Commands such as `info`, `app apply`, and `publish` normally locate a saved credential for that URL even when another profile is the default. Specify `--profile` when multiple accounts share the same API endpoint or when the intended identity must be explicit. An explicitly selected profile must match the repository's API URL.

Other one-process overrides are:

- `--url` or `MAYPOP_URL` for the API origin.
- `--token` or `MAYPOP_TOKEN` for an access token.
- `MAYPOP_CONFIG_DIR` for the credentials directory.

Do not print, commit, or place access tokens in `maypop.toml` or Git remotes. The repository credential helper reads the saved profile; the token is not embedded in `.git/config`.

## Diagnose before changing state

Use read-only commands first:

```sh
maypop status
maypop profile list
maypop info
git status --short
git remote -v
```

Then explain the intended change. `auth`, `init`, `app apply`, and `publish` require explicit user authorization because they create credentials, alter repository configuration, or change remote Maypop state.
