<p align="center">
  <a href="https://obris.ai">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="./.assets/obris-logo-light.svg">
      <source media="(prefers-color-scheme: light)" srcset="./.assets/obris-logo-dark.svg">
      <img src="./.assets/obris-logo-dark.svg" alt="Obris" width="200">
    </picture>
  </a>
</p>

<p align="center">
  Use Obris in the Codex CLI.
</p>

<p align="center">
  <a href="https://github.com/obris-dev/obris-codex-plugin/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache%202.0-blue.svg" alt="License"></a>
</p>

The plugin's goal is to sync your Obris knowledge to disk through the
`obris` CLI, where Codex can read your topics as files. That's faster
than calling a server every time, and it leaves Codex more room to
focus on what you actually asked. The bundled MCP server keeps your
knowledge reachable when Codex doesn't have filesystem access.

Provides:

- **MCP server**: List topics, read items, and save knowledge from any
  Codex CLI session.
- **`@obris-sync` skill**: Sync a topic to a local directory (pulls
  remote items, pushes local edits, bulk-uploads via `--add-all`,
  manages exclude/include rules per directory).
- **`@obris-add` skill**: Push a single local file into a synced topic.
- **`@obris-link` skill**: Relink a renamed file to its remote item.
- **`@obris-unlink` skill**: Stop syncing a file without deleting either side.
- **`@obris-auth` skill**: Set up Obris CLI authentication.

## Common workflows

- **Bootstrap a new directory into a topic.** Find or create the topic,
  then `obris sync -p <dir> -t <id> --add-all`. Subdirs become subtopics;
  files in `.env`, `.ssh/`, `.aws/`, `.git/` etc. are skipped by default.
- **Pull a topic down to inspect its files.** `obris sync -p <dir> -t <id>`
  with no `--add-all`. Pulls every item; doesn't upload local files.
- **Keep a working directory synced.** Just `obris sync -p <dir>` with no
  flags. It picks up local edits and remote changes, surfaces conflicts,
  and reports new untracked files without uploading them.
- **Override a default exclusion** (e.g., a `.env.example` you actually do
  want synced). `obris sync include .env.example -p <dir>`. Last-call-wins;
  the inverse is `obris sync exclude`.

## Install the plugin

In a terminal, add the plugin's marketplace and install it:

```bash
codex plugin marketplace add obris-dev/obris-codex-plugin
```

Then open the in-CLI plugin browser to confirm it's installed:

```bash
codex /plugins
```

For more on what `codex plugin marketplace add` does, see OpenAI's
[Add a marketplace from the CLI](https://developers.openai.com/codex/plugins/build#add-a-marketplace-from-the-cli)
docs.

Plugin install is currently a Codex CLI feature. The Codex desktop app
and the IDE extensions don't surface plugin management yet. If you use
Codex through the desktop app or an IDE extension, connect Obris over
MCP directly via an MCP connector instead.

## Enable the MCP server (one-time auth)

Installing the plugin registers Obris's MCP server with Codex, but Codex
is not yet signed in to your Obris account. The first time you ask Codex
to do something that hits an Obris tool, it will prompt you to sign in.

1. In a Codex CLI session, ask anything that needs Obris. For example,
   "list my Obris topics."
2. Codex prompts you to sign in to Obris and surfaces a sign-in URL.
   Open it in your browser and authorize Codex to access your Obris
   account.
3. Return to the session. The original request continues, and every
   future Obris tool call uses your account automatically.

Sign-in is per device. A second CLI workstation walks through the same
one-time flow.

## CLI authentication

The sync, add, and link skills shell out to the bundled `obris` CLI. The
first time the agent runs an `obris` command, the
[`@obris-auth`](skills/auth/SKILL.md) skill walks you through a two-step
browser sign-in. No API keys to copy.

The MCP server works without ever invoking the CLI. The CLI is only
involved for the disk-level sync commands.

## Invoking skills

Codex picks up skills automatically when their description matches what
you're asking for. To invoke one explicitly, type `@` in a Codex session
and select the skill by name (`@obris-sync`, `@obris-add`,
`@obris-link`, `@obris-auth`).

## Links

- [Obris](https://obris.ai)
- [Plugin documentation](https://docs.obris.ai/plugins/codex)
- [Obris CLI on PyPI](https://pypi.org/project/obris-cli/)
- [CLI documentation](https://docs.obris.ai/cli)
- [Codex plugin docs (OpenAI)](https://developers.openai.com/codex/plugins/build)
- Using Claude instead? See the [Obris Claude Plugin](https://docs.obris.ai/plugins/claude).
