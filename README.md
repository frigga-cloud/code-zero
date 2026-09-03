# code-zero

**Connect Claude Code to your organisation's whole codebase** — so it answers
questions about every repository you have, not just the one you happen to have
open.

```bash
npm i -g @frigga-cloud/code0
code0 init
```

That's the install. A browser opens, you approve with your Frigga account, and
you restart Claude Code. Nothing else to configure.

Install it **globally** (`-g`). A plain `npm i` drops it into the current
directory's `node_modules` and puts nothing on your `PATH`, so `code0` would only
work from that one folder — and `status`, `upgrade` and `uninstall` are commands
you will want later, from wherever you happen to be.

To try it without installing anything, `npx @frigga-cloud/code0 init` also works.

> This repository is for **issues and documentation**. The CLI's source is not
> public; the published package contains the compiled tool.

---

## Using it

Once installed, code0 works in two ways.

**Context arrives on its own.** A hook runs before every message and hands Claude
the code relevant to what you just asked. Nothing to type.

**Ask for the tools with `use code0`.** To have the agent actively go and search
your other repositories, end the prompt with `use code0`:

```
who calls createOrder? use code0
```

Without it the agent tends to answer from the repository you have open — the
right instinct for local questions, the wrong one for anything spanning
repositories: callers, dependents, ownership, "is this still used anywhere?".

## What it does

Claude Code can only read the repository you have checked out. code0 connects it
to two things beyond that:

**The tools.** An MCP connection registered for every project — search code, find
symbols and references, trace call hierarchies, and read files across *all* your
organisation's indexed repositories. Plus a narrow cloud lookup for finding a
live VPC or security group id while writing infrastructure code.

**The context.** A hook that runs before each message, finds the code relevant to
what you just asked, and hands it to Claude automatically — so you don't have to
know which repository to point it at.

The difference shows up on questions local search cannot answer:

> *Which services call this endpoint?*
> *Who owns the billing worker?*
> *Is anything still importing the old auth helper?*

## Commands

| | |
|---|---|
| `code0 init` | Authorise, install the hooks, register the MCP server |
| `code0 status` | Is it installed **and working**? |
| `code0 upgrade` | Refresh after a new release (no re-login) |
| `code0 uninstall` | Remove it (`--purge` also drops credentials) |

Run `init` again any time — it is safe to repeat and will re-authorise and
refresh everything.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- Node.js 20 or newer
- A Frigga account in your organisation, with at least one repository indexed

## Your uncommitted work stays yours

code0 indexes **committed** code. Before each request the hook runs
`git status --porcelain` and sends the **paths** of your modified files — never
their contents. code0 then withholds its own (now stale) copy of those files and
says so, while still answering about everything else.

That is what makes *"does my uncommitted change break something elsewhere?"* a
question with a real answer: your working tree is the truth for the file you are
editing, the index is the truth for everything that depends on it.

## What it installs

Everything code0 owns lives in `~/.claude/code0/` — the policy it injects, your
credentials (mode `0600`), and a copy of the hook program.

The one file outside that directory it touches is `~/.claude/settings.json`, to
register the hooks. It is edited in place, never regenerated: your own hooks,
permissions and settings are carried through untouched, and `uninstall` removes
exactly code0's entries and nothing else. If that file is ever unparseable, the
CLI refuses to write rather than overwrite settings it cannot read back.

It never creates, edits, backs up or deletes your `CLAUDE.md`.

## Authentication

OAuth 2.0 with PKCE over a loopback redirect. No long-lived API key is stored —
only a revocable refresh token, from which short-lived access tokens are minted.

On a machine with no browser (SSH, containers):

```bash
code0 init --no-browser
```

## If something looks wrong

```bash
code0 status
```

It performs a real token exchange rather than checking whether a file exists, so
it will tell you if the credential has expired.

The hook is built to fail quietly — a hook that errors would degrade *every*
message you send, so a network blip costs you some extra context and nothing
more. The one exception is an expired credential, which it reports, because
silently losing grounding for weeks is worse than a visible nudge.

## Support

Please [open an issue](https://github.com/frigga-cloud/code-zero/issues) and include:

- the output of `code0 status`
- your Claude Code version (`claude --version`)
- your operating system

## Licence

MIT — see [LICENSE](./LICENSE). Copyright © 2026 Frigga Cloud Pvt. Ltd, India.
