# Changelog

Versions of the `unity-workflow` plugin. The version lives in `.claude-plugin/plugin.json`. Installs from GitHub stay on a version until it changes, so every release bumps it: patch for wording fixes, minor for new skills, agents or rules, major for changes that alter existing behavior.

## 1.0.0 — 2026-10-09

- `unity-coding` skill: Unity C# coding standards as overridable defaults.
- `unity-dev-flow` skill: task loop, risk tiers, verification, bug-fix loop, review gate, commits.
- `project-setup` skill: writes a project's shared `CLAUDE.md` and per-machine project memory.
- `qa-changelog` skill: milestone changelog between release tags with a player-facing "What's new" and a QA verification plan.
- Modules: `unitask-dotween`, `editor-tooling`.
- Agents: `unity-reviewer`, `console-reader`.
- Hooks: block hand-written `.meta` files and edits to Unity-generated folders and files; ask before `git push`.
