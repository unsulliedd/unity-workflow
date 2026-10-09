---
name: unity-dev-flow
description: Unity development workflow - task loop, risk tiers, verification, bug-fix loop, review gate, commits. Invoked by hand at the start of a Unity work session.
disable-model-invocation: true
argument-hint: "task description"
---

# Unity Dev Flow

Standing instructions for the rest of this session. Most important rules first.

## 1. Before any code

- Read the project's `CLAUDE.md`. For each module it lists, read `${CLAUDE_PLUGIN_ROOT}/modules/<module>.md`.
- Read project memory for the expected Unity MCP instance, project bug classes and reviewer suppressions.
- If the task touches more than one system, plan first and wait for approval. Work that spans sessions gets a written plan in the project's docs folder (see the project's `CLAUDE.md`). Do not re-derive or reopen what a written plan already decided.
- Assign a risk tier when the plan is approved and state it:

| Tier | Examples | Review |
| --- | --- | --- |
| Small | Renames, UI tweaks, one isolated method, config or data edits | User's diff read |
| Standard | Features, refactors, all bug fixes | User's diff read |
| High risk | Save and load, serialization formats, economy and currency, IAP, remote config keys and defaults, crossing a reusable-package boundary, plus the project's own list | Reviewer agent required |

## 2. Verify before reporting done

Run after every change, in order. Report what actually ran.

1. **Confirm the editor.** Before trusting any Unity MCP result, confirm MCP is attached to this project (`mcpforunity://instances` or `Application.dataPath`) and pin it with `set_active_instance`. An MCP server attached to another open project returns clean results that say nothing about this one.
2. **Compile.** Editor open and MCP confirmed: `refresh_unity`, then `read_console`. Otherwise the out-of-editor compile check below.
3. **Console.** Fix every error first. Report the warning count on touched files before and after.
4. **Tests.** Run the tests covering the touched code. Report which ran and their results.
5. **Device.** Touch input, performance, memory and store builds need a physical device. Say so; do not claim it.

**Out-of-editor compile check.** Copy `Assembly-CSharp.csproj` into the scratchpad; make every relative `Include` and `HintPath` absolute; drop `Compile` items whose file no longer exists; replace each `ProjectReference` with a `Reference` to `Library/ScriptAssemblies/<name>.dll`; add `Compile` items for new files; `dotnet build <copy> -nologo -v q -clp:ErrorsOnly "-p:OutputPath=<scratch>/bin/"`. Editor assemblies: same recipe on `Assembly-CSharp-Editor.csproj`, built second, referencing the runtime build just produced, not the stale DLL in `Library/`, each with its own output path. Nothing is written into the repo. About 40 seconds cold.

Batchmode test runs (`Unity -batchmode -projectPath . -runTests ...`) only work while the editor has the project closed.

## 3. Implement

- Smallest diff that does what the plan states. List anything the plan did not foresee instead of silently widening scope.
- Scene, prefab and Inspector wiring: unless project or user instructions settle it, ask whether the user wants to do it in the editor or have the Unity MCP `manage_*` tools do it. When the user does it, list the exact steps.
- Coding standards come from the `unity-coding` skill.

## 4. Bug fixes

1. Reproduce first. Pure logic (math, rules, balance and economy formulas, serialization round-trips, generators): a failing test. Scene-, timing- or lifecycle-dependent: exact repro steps plus logging that proves the cause. No fix before the cause is shown. No PlayMode harness for a one-off bug.
2. Fix the root cause. If it is out of scope, say so and fix the symptom only with the user's agreement.
3. Smallest diff. List every other behavior that code path affects.
4. Full verification sequence, not only the new test.
5. High risk: run the reviewer.
6. When a regression got through, record its class: generic Unity classes go to the reviewer agent's checklist in the plugin, project classes to project memory.

Tests go where the assemblies allow: code in `Assembly-CSharp` with no asmdef is tested from an `Editor` folder test.

## 5. Review (high-risk tier)

Run the `unity-reviewer` agent on the diff. Subagents do not see auto memory, so put the project's bug classes and reviewer suppressions from project memory into the prompt, together with the approved plan. A FAIL returns to step 3.

## 6. Hand over

Report: files changed, what was verified and how, remaining manual Unity steps (assets to create, references to wire), renamed serialized fields and how their values are kept, what the user should smoke-test. Then stop.

## 7. Commits — only when asked

- Commit only what compiles and what the user has smoke-tested in Unity. Keep a pure folder or namespace reorganization out of feature commits.
- Commit with an explicit pathspec (`git add <paths> && git commit -F - -- <paths>`) so the user's own staged changes are never swept in.
- Before committing an authored `.asset`, confirm it is saved on disk; Unity holds Inspector edits in memory until the project is saved.
- Message style: the repository's existing style, see the `unity-coding` skill.

## 8. Working habits

- Delegate long output that will not be referenced again (log digestion, test output, broad sweeps) to a subagent; `console-reader` for logs. Iterative work stays in the main session.
- A cheap search agent may locate code; its conclusions about how a subtle system behaves are not trusted until the main session reads the code.
- Parallel subagents multiply usage. Use them to remove noise, not by default.

## Task

$ARGUMENTS
