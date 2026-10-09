---
name: project-setup
description: Sets up a Unity project for the Unity dev flow - writes the shared project CLAUDE.md with neutral facts and stores per-machine values in project memory. Invoked by hand once per project.
disable-model-invocation: true
---

# Unity Project Setup

Creates the project overlay for the Unity dev flow. Discover everything that can be discovered; ask only for what cannot.

The repo `CLAUDE.md` is committed and loads into every teammate's sessions on every turn. It holds **neutral project facts only**: no personal preferences, no workflow rules, no commit habits. Keep it under about 80 lines.

## 1. Discover

- **Unity version:** `ProjectSettings/ProjectVersion.txt`.
- **Render pipeline:** `Packages/manifest.json` (`com.unity.render-pipelines.universal` / `.high-definition`, else built-in).
- **Platforms:** build targets and platform identifiers in `ProjectSettings/ProjectSettings.asset` and `ProjectSettings/EditorBuildSettings.asset`, signing files, platform-specific folders under `Assets/Plugins`.
- **Assemblies:** every `.asmdef` outside third-party folders; note which code compiles into `Assembly-CSharp`.
- **Tests:** files containing `[Test]` or `[UnityTest]`, and their assemblies.
- **Folder map:** top-level folders under `Assets/` that hold project code, configuration assets, UI, editor tools, reusable packages and third-party code.
- **Docs:** the project's documentation folder, if any (`Docs/`, `Documentation/`, ...) — design documents, plans, known-issue lists.
- **Modules** (from `${CLAUDE_PLUGIN_ROOT}/modules/`):
  - `unitask-dotween` — UniTask and DOTween are both present (packages or `Assets/`).
  - `editor-tooling` — the project is an editor extension or Asset Store editor tool.
- **Conventions:** namespace patterns and outliers, the logging facade if the project wraps `Debug.Log`, in-house frameworks and their rules, architecture patterns (read a few representative files per area).
- **Release tags:** `git tag --sort=-creatordate` — the naming pattern.
- **Existing `CLAUDE.md`:** if one exists, read it. It is merged into, never overwritten.

## 2. Ask, in one message

Only what discovery could not settle:

- Status of each target platform (shipping, planned, not started).
- Project systems to add to the high-risk tier.
- Expected Unity MCP instance name and port on this machine.
- Project bug classes and reviewer suppressions not already in project memory.
- Any discovered fact that looked ambiguous.

## 3. Write the repo `CLAUDE.md`

Sections, each short:

```markdown
# <Project>

## Project
- Unity <version>, <render pipeline>. Platforms: <platform — status>, ...
- Modules: <module>, <module>

## Layout
- <folder> — <what lives there>
...

## Assemblies and tests
- <which assemblies exist, where code without an asmdef compiles>
- Tests: <where, how to run>
- Compile check without the editor: <how>

## Conventions
- <namespace rules and exceptions not to copy or "fix">
- <logging facade, in-house framework rules>
- <architecture patterns the project follows>

## Docs
- <where plans and design documents live, and how they are organized>

## High-risk areas
- <project systems>

## Releases
- Tags: <release tag pattern and branch, e.g. v1.4.0 on main>
```

Show the draft and wait for approval before writing it. Do not commit it; tell the user it changes teammates' sessions and is worth mentioning to them.

## 4. Write project memory

One memory file per fact, following the memory conventions already in context:

- `reference` — Unity MCP instance name and port.
- `project` — project bug classes and reviewer suppressions (one file, a list).

Before writing, check existing memory: update an entry that covers the same fact instead of duplicating it, and point out entries that the new `CLAUDE.md` or the plugin now covers, so the user can decide whether to remove them.

## 5. Report

List what was written where, which modules apply, and which questions are still open.
