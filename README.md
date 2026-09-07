# microapps

Small tools. One Bun file each.

No bundler, no dependency installation, no shared runtime to untangle. Copy a file, run it with [Bun](https://bun.sh), and keep the tools you find useful.

| App | What it does | Requirements |
| --- | --- | --- |
| [Runner status](apps/runner-status.ts) | A GitHub-style dashboard for a shared self-hosted runner pool: assignments, running steps, durations, and queued jobs across repositories. | Bun and authenticated GitHub CLI |

## Runner status

```sh
gh auth login
RUNNER_ORG=your-org RUNNER_REPOS=your-org/api,your-org/web bun apps/runner-status.ts
```

Open **http://127.0.0.1:4545**.

The GitHub account needs permission to list the organization's runners and read Actions in each repository. Authentication stays in your existing `gh` configuration; the app does not store credentials or send them to the browser.

| Variable | Default | Purpose |
| --- | --- | --- |
| `RUNNER_ORG` | Required | GitHub organization owning the runners. |
| `RUNNER_REPOS` | Required | Comma-separated `owner/repo` names to inspect for jobs. |
| `RUNNER_NAME_PREFIX` | Empty, all runners | Optionally limit the displayed runner pool by name prefix. |
| `PORT` | `4545` | Local HTTP port. |
| `RUNNER_PUBLIC_HOST` | Unset | One allowed hostname for a private reverse proxy. Does not change the loopback bind address. |

### What you see

- Runner status and assignments across the configured repositories.
- Current steps, expandable individually or with **Expand all**.
- Running jobs and queued jobs, with repository and text filters.
- Links to the original GitHub jobs and workflow runs.
- Visible errors and retained last-known data when a source cannot refresh.

Running durations use GitHub's job and step start timestamps. Queued rows show time since the workflow was created, **not** time spent executing or a guaranteed queue position. Matching labels do not account for all GitHub scheduling constraints.

Job discovery covers only `RUNNER_REPOS`. A busy runner working elsewhere is shown with an unknown assignment rather than marked idle.

### Polling and access

GitHub is polled every 45 seconds with at most four concurrent requests. Browser tabs read one shared cache every 15 seconds. Each refresh costs approximately `1 + 5 × repositories + active workflow runs` API requests, plus pagination. Be mindful of the GitHub account's shared API quota when adding many repositories. The service continues polling while running, even without an open tab.

The app binds to `127.0.0.1`. It has no built-in authentication: keep it local, or put it behind private/authenticated access such as Tailscale Serve. Do not publish its runner and job data to the internet unintentionally. When using a reverse proxy, set `RUNNER_PUBLIC_HOST` to its exact hostname.

For a persistent service, run the same command under launchd on macOS or systemd on Linux, with the environment variables above. The service user must have its own working `gh` login and `bun`/`gh` on `PATH`. No credentials belong in this repository or in service command lines.

### Checks

```sh
bun apps/runner-status.ts --help
bun apps/runner-status.ts --test
# Or run all registered app checks:
bun run test
```

The embedded checks exercise pagination, runner-label matching, job filtering, run deduplication, concurrency, and browser-script syntax without GitHub credentials or network access.

## Adding a tool

Put it in `apps/<name>.ts`, add it to the table, and document its configuration. Keep application code, UI, and assets in that single file. Prefer Bun's built-in APIs and no external dependencies. Add a credential-free `--test` entry point and register it in `package.json`.

Never commit credentials, local configuration, logs, or live API snapshots.
