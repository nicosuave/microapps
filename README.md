# microapps

Small tools. One Bun file each.

No bundler, no dependency installation, no shared runtime to untangle. Copy a file, run it with [Bun](https://bun.sh), and keep the tools you find useful.

| App | What it does | Requirements |
| --- | --- | --- |
| [Runner status](apps/runner-status) | A GitHub-style dashboard for a shared self-hosted runner pool: assignments, running steps, durations, and queued jobs across repositories. | Bun and authenticated GitHub CLI |

## Adding a tool

Give it an `apps/<name>/` folder with a `README.md` and a single `main.ts`, then add it to the table. Keep application code, UI, and assets in that single file. Prefer Bun's built-in APIs and no external dependencies. Add a credential-free `--test` entry point and register it in `package.json`.

Never commit credentials, local configuration, logs, or live API snapshots.
