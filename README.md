# Unity Workflow

Welcome to **Unity Workflow**—a curated collection of modular AI Agent Skills designed for automated workflows, code quality enforcement, and project-wide development standardizations.

---

## 📌 Repository Overview

This repository contains reusable **Agent Skills** adhering to the open AI Agent Skill Specification (`SKILL.md`). Each skill provides domain-specific instructions, strict behavioral guidelines, execution templates, and coding standards that are automatically discovered and loaded by compatible AI coding assistants (such as Antigravity, Claude, Cursor, and custom agent tools).

### Skill File Structure
Each skill resides in its own folder within `skills/` and contains a `SKILL.md` file structured with:
- **YAML Frontmatter**: Contains required metadata (`name`, `description`) used by agents for trigger-matching.
- **Instruction Body**: Markdown payload defining domain rules, coding standards, output templates, and step-by-step procedures.

---

## 🛠️ Skills Summary & Index Table

Below is the complete index of skills available in this repository, including direct pointer links to their specification files.

| Skill Name | Path & Specification | Trigger / Purpose Description | Domain / Category |
| :--- | :--- | :--- | :--- |
| **`unity-coding`** | [skills/unity-coding/SKILL.md](skills/unity-coding/SKILL.md) | Unity C# coding standards: null safety, Find/GetComponent, LINQ in hot paths, inspector fields, naming, namespaces, comments, commit messages. Loads automatically for `.cs` work. | Unity C# Development |
| **`unity-dev-flow`** | [skills/unity-dev-flow/SKILL.md](skills/unity-dev-flow/SKILL.md) | Development workflow: planning, risk tiers, verification, bug-fix loop, review gate, commits. Invoked by hand. | Unity Workflow |
| **`project-setup`** | [skills/project-setup/SKILL.md](skills/project-setup/SKILL.md) | Writes a Unity project's shared `CLAUDE.md` (neutral facts) and per-machine project memory. Invoked by hand once per project. | Unity Workflow |
| **`qa-changelog`** | [skills/qa-changelog/SKILL.md](skills/qa-changelog/SKILL.md) | Milestone changelog between release tags: player-facing "What's new" plus a QA verification plan by feature area. Invoked by hand. | Documentation & QA |

Stack-specific rules live in [`modules/`](modules) (`unitask-dotween`, `editor-tooling`). A project's `CLAUDE.md` lists which modules apply, and the skills read them from there.

The workflow behind these files, and the reasoning for where each rule lives, is in [`docs/unity-dev-flow.md`](docs/unity-dev-flow.md). Change that document first, then the generated files.

---

## 🧩 Claude Code Plugin

The repository is also a Claude Code marketplace holding one plugin, `unity-workflow`, which bundles the skills above with:

* **Agents**: `unity-reviewer` (adversarial review of high-risk diffs, PASS/FAIL) and `console-reader` (digests long Unity logs).
* **Hooks** ([`hooks/guard.js`](hooks/guard.js), needs Node): blocks hand-written `.meta` files and edits to `Library/`, `Temp/`, `Logs/`, `obj/` and the generated root `.csproj`/`.sln` files in Unity projects; asks for confirmation before any `git push`.

Install:

```bash
claude plugin marketplace add unsulliedd/unity-workflow
claude plugin install unity-workflow@unsulliedd-skills
```

Plugin skills are namespaced, for example `/unity-workflow:unity-dev-flow`. Run `/unity-workflow:project-setup` once per project to write its `CLAUDE.md`.

**Defaults, not mandates.** Every rule is a default. Your `~/.claude/CLAUDE.md`, a personal skill, or the project's `CLAUDE.md` override it, for example to forbid `[FormerlySerializedAs]` or to define your own commit message format. The reasoning behind each rule and where overrides belong is in [`docs/unity-dev-flow.md`](docs/unity-dev-flow.md).

The plugin version lives in `.claude-plugin/plugin.json`; installs stay on a version until it is bumped. Release notes are in [`CHANGELOG.md`](CHANGELOG.md).

---

## 🚀 How to Use & Integrate Skills

### Auto-Discovery Paths
Compatible AI agents automatically discover and load skills placed in customization root directories:
* **Project Scope**: Copy the target skill directory into `.agents/skills/<skill-name>/` within your project repository.
* **Global Scope**: Copy to `~/.gemini/config/skills/<skill-name>/` (or `C:\Users\<User>\.gemini\config\skills\<skill-name>\` on Windows).

### Creating a New Skill
To add a new skill to this repository:
1. Create a subfolder under `skills/<skill-name>/`.
2. Add a `SKILL.md` file with standard YAML frontmatter:
   ```yaml
   ---
   name: your-skill-name
   description: Clear trigger description detailing when the agent should apply this skill.
   ---
   ```
3. Write the markdown instructions, guidelines, and output templates below the frontmatter block.
4. Update the index table and summary sections in [`README.md`](README.md).
