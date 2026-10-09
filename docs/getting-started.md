# Getting Started

A short guide to using the `unity-workflow` plugin with Claude Code in a Unity project.

## Requirements

- Claude Code (CLI, desktop app or IDE extension).
- Node.js on `PATH`, for the guard hook.
- Optional: [MCP for Unity](https://github.com/CoplayDev/unity-mcp) so Claude can refresh, read the console and run tests in the editor; the .NET SDK for compile checks while the editor is closed.

## 1. Install

```bash
claude plugin marketplace add unsulliedd/unity-workflow
claude plugin install unity-workflow@unsulliedd-skills
```

Start a new session afterwards. Skills appear under `/unity-workflow:`.

## 2. Set up each project once

Open Claude Code in the Unity project root and run:

```
/unity-workflow:project-setup
```

It reads the project (Unity version, platforms, assemblies, tests, folders, conventions), asks only what it cannot find, and shows a `CLAUDE.md` draft for approval. Commit that file: it gives every teammate's Claude sessions the same project facts. Per-machine details (Unity MCP instance and port, known bug classes) go to your personal project memory instead.

## 3. Daily use

| You want to | Do this |
| --- | --- |
| Write or change C# | Nothing. `unity-coding` loads automatically for `.cs` work. |
| Do a feature, refactor or bug fix | `/unity-workflow:unity-dev-flow <task>`. Claude plans, waits for your approval, implements, verifies (compile, console, tests), and hands over the manual Unity steps. |
| Get a second opinion on risky code | High-risk work (save data, serialization, economy, IAP, remote config) runs the `unity-reviewer` agent inside the dev flow. Outside it, ask: "run unity-reviewer on the diff". |
| Make sense of a long Unity log | Ask: "use console-reader on `<log file>`". |
| Write release notes and a QA plan | `/unity-workflow:qa-changelog [from] [to]`. Defaults to the latest release tag up to `develop`. |

Claude commits only when you ask.

## 4. Built-in guards

These hold in every session, whether or not a skill is used:

- Hand-written `.meta` files are blocked. Unity generates them.
- In Unity projects, edits to `Library/`, `Temp/`, `Logs/`, `obj/` and the generated root `.csproj`/`.sln` files are blocked.
- `git push` asks for your confirmation every time.

## 5. Make it yours

Every plugin rule is a default. Override it at the level it belongs to:

| Where | For | Example |
| --- | --- | --- |
| `~/.claude/CLAUDE.md` | Your rules in every project, always on | "Never use `[FormerlySerializedAs]`; I re-wire references by hand." |
| `~/.claude/skills/<name>/SKILL.md` | Your longer house style, loaded when relevant | Your commit message format, namespace conventions |
| The project's `CLAUDE.md` | Facts and conventions the whole team shares | Logging facade, in-house framework rules |

Keep personal preferences out of the project's `CLAUDE.md`; it loads into every teammate's sessions.

## 6. Update or remove

```bash
claude plugin marketplace update unsulliedd-skills
claude plugin update unity-workflow@unsulliedd-skills
```

```bash
claude plugin uninstall unity-workflow@unsulliedd-skills
claude plugin marketplace remove unsulliedd-skills
```

Switching between a local clone and GitHub needs `marketplace remove` first; otherwise `marketplace add` fails with "source doesn't match its extraKnownMarketplaces entry".

## For maintainers

1. Edit in a local clone. To test before pushing: `claude plugin marketplace remove unsulliedd-skills`, then `claude plugin marketplace add <path-to-clone>` and install; edits apply on the next session or after `/reload-plugins`.
2. Run `claude plugin validate .`.
3. Bump `version` in `.claude-plugin/plugin.json` and add an entry to `CHANGELOG.md`. Without a version bump, existing installs stay on the old version.
4. Push, then switch back to the GitHub source and update.

Why each rule exists, and which layer it belongs to: [unity-dev-flow.md](unity-dev-flow.md).
