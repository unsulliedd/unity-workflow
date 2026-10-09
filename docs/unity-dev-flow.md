# Unity + Claude Code Development Workflow

The design behind the `unity-workflow` plugin. This document is the human-readable source of truth. It is not loaded into any session. The compact files that are loaded (skills, agents, hooks) are derived from it, so a change starts here.

## How the configuration is layered

Rules are placed by who they apply to and how reliably they must apply, not by topic.

| Layer | Contents | Location | Applies to |
| --- | --- | --- | --- |
| Plugin defaults | Coding standards, task loop, verification, review gate, guards | This plugin | Everyone who installs it |
| Personal, always on | Your own hard rules and overrides | `~/.claude/CLAUDE.md` | You, every project, every turn |
| Personal, on demand | Your house style (commit format, naming variants, planning habits) | A personal skill in `~/.claude/skills/` | You, when relevant |
| Project, shared | Neutral project facts | Repo `CLAUDE.md` | Everyone who opens the repo |
| Project, personal | Per-machine values, project bug classes, reviewer suppressions | Project memory, `.claude/settings.local.json` | You, this project |

Consequences:

- The plugin states defaults and says where they can be overridden. Project and user instructions win.
- A rule that must hold even when no skill is invoked goes in a `CLAUDE.md` or a hook, never only in a skill.
- Nothing personal goes into a file committed to a project repo. Committed `CLAUDE.md` and `.claude/settings.json` change every teammate's sessions.

## Principles

1. Claude verifies its own work (compile, console, tests) before reporting a change as done, and reports what was actually run.
2. Any change that touches more than one system is planned first. You approve the plan before code is written, and you read the diff before commit.
3. The main session holds decisions and edits. High-volume reading (log digestion, codebase sweeps) goes to subagents.
4. One task per session. Start a new session between unrelated tasks.
5. Stable rules live in files, not in prompts you retype.

## Guards

Enforced by the plugin's hook in every session, whether or not a skill is invoked:

- Hand-written or hand-edited `.meta` files are blocked. Unity generates them; hand-made GUIDs cause import problems. Deleting a `.meta` together with its asset is unaffected.
- In Unity projects, edits to `Library/`, `Temp/`, `Logs/`, `obj/` and the generated root `.csproj`/`.sln` files are blocked.
- `git push` asks for confirmation (an ask, not a deny, so an explicit request still works).

## Task loop

| Step | What happens | Gate |
| --- | --- | --- |
| 1. Explore | Locate the code involved. A subagent may do the sweep, but the main session reads the files it will change. | |
| 2. Plan | Plan mode for small and standard work. A written plan in the project's docs folder for work that spans sessions. Assign a risk tier. | You approve the plan |
| 3. Implement | Smallest diff that does what the plan states. List anything the plan did not foresee instead of silently widening scope. | |
| 4. Verify | The verification sequence below. A failure returns to step 3. | Clean compile and console |
| 5. Review | Run the reviewer agent. High-risk tier only. A FAIL returns to step 3. | PASS |
| 6. Hand over | Files changed, what was verified, which manual Unity steps remain, what to smoke-test. | You smoke-test and read the diff |
| 7. Commit | Only when asked. | |

Scene, prefab and Inspector wiring: unless project or user instructions settle it, Claude asks whether you want to do it in the editor or have the Unity MCP tools do it.

## Risk tiers

Assign the tier when the plan is approved. The tier decides the effort spent and whether review is required. Treat the routing as a hypothesis: after a few weeks, check which tier the regressions came from, and move task types up or down.

| Tier | Examples | Writes | Review |
| --- | --- | --- | --- |
| Small | Renames, UI tweaks, one isolated method, config or data edits | Main session, or a cheaper model for purely mechanical edits | Your diff read |
| Standard | Features, refactors, all bug fixes | Main session, strongest model | Your diff read |
| High risk | Save and load, serialization formats, economy and currency, IAP, remote config keys and defaults, anything that crosses a reusable-package boundary | Main session, strongest model | Reviewer agent required |

The project's `CLAUDE.md` extends the high-risk list with its own systems.

## Verification

Run after every change, in this order.

1. **Confirm the editor.** Before trusting any Unity MCP result, confirm MCP is attached to this project: check the instance list or `Application.dataPath`, and pin it with `set_active_instance`. An MCP server attached to another open project returns clean results that say nothing about this one.
2. **Compile.** With the editor open and MCP confirmed: `refresh_unity`, then `read_console`. With the editor closed or MCP unreliable: the out-of-editor compile check (below).
3. **Console.** Fix every error before anything else. Report the warning count on touched files before and after.
4. **Tests.** Run the tests that cover the touched code and report which ran and their results, not only that they passed.
5. **Device.** Touch input, performance, memory and store builds need a physical device. Claude cannot do this step and says so.

**Out-of-editor compile check.** Unity generates SDK-style `.csproj` files. Copy `Assembly-CSharp.csproj` into the scratchpad, make every relative path absolute, replace each `ProjectReference` with a `Reference` to the prebuilt DLL in `Library/ScriptAssemblies/`, add `Compile` items for new files Unity has not seen yet, then `dotnet build` it with an output path in the scratchpad. Nothing is written into the repo. Editor assemblies are built second, against the runtime build just produced, not the stale DLL in `Library/`. Expect about 40 seconds cold.

Batchmode test runs (`Unity -batchmode -projectPath . -runTests ...`) work only while the editor has the project closed.

## Testing policy

- Pure logic (math, rules, balance and economy formulas, serialization round-trips, generators): write a failing test first, then fix.
- Scene-, timing- or lifecycle-dependent behavior: write exact repro steps and add logging that proves the cause, then fix. Do not build a PlayMode test harness for a one-off bug.
- Tests go where the project's assemblies allow. Code in `Assembly-CSharp` with no asmdef is tested from an `Editor` folder test.

## Bug fixes

1. Reproduce: a red test or exact repro steps. No fix before the cause is shown.
2. Fix the root cause, not the symptom. If the root cause is out of scope, say so and fix the symptom only with your agreement.
3. Smallest diff. List every other behavior that code path affects.
4. Run the full verification sequence, not only the new test.
5. High risk: run the reviewer.
6. When a regression gets through, record its class: generic Unity classes go into the reviewer agent, project classes into project memory.

## Review

The reviewer agent reviews a diff it did not write and assumes it contains a mistake. It ends with PASS or FAIL and findings ranked must fix, should fix, consider.

Generic Unity bug classes it checks:

- Event or delegate subscription without a matching unsubscribe, especially `OnEnable`/`OnDisable` pairs and static events.
- Awake, OnEnable and Start ordering assumptions across objects.
- Renamed or retyped serialized fields, and how their values are kept.
- New save fields that load as their default value for existing players. Check what an old save produces.
- Allocations in per-frame paths (`Update`, `LateUpdate`, per-unit loops).
- Pooled objects: state not reset on reuse, registration tied to spawn rather than to the pool's own events.
- Async work that outlives its owner (destroyed object, unloaded scene).
- Behavior beyond what the approved plan states.

Project bug classes and **suppressions** (things the project has confirmed are not bugs, such as a manager that is guaranteed non-null) live in project memory. Subagents do not see auto memory, so the main session puts them into the reviewer's prompt together with the approved plan. The reviewer has no memory of its own: `memory: project` would store it inside the repo. It does load `CLAUDE.md`, so personal and project overrides reach it.

## Coding standards

The `unity-coding` skill holds the rules with examples and loads automatically for `.cs` work. Each rule is a default that project or user instructions may override:

- No `?.` or `??` on `UnityEngine.Object`. No repeated null guards on dependencies validated once.
- No `Find` methods. No `GetComponent` in hot paths. Prefer `TryGetComponent`. Compare tags with `CompareTag`.
- No LINQ in runtime hot paths.
- MonoBehaviour only where lifecycle hooks, serialized scene references or physics callbacks are needed.
- `[SerializeField] private` instead of public fields. A renamed serialized field keeps its value with `[FormerlySerializedAs]` by default, and the rename is always reported.
- No magic numbers for gameplay values.
- Microsoft C# naming. Namespaces mirror the folder path; a new file matches its nearest sibling.
- XML docs on public and protected members. Comments are technical documentation, not conversation. No commented-out code.
- Commit messages follow the repository's existing style.

Project-specific facts (folder layout, logging facade, namespace rules and exceptions, in-house framework rules, architecture patterns, assemblies) belong in the project's `CLAUDE.md`, not in the skill.

## Modules

Rules that apply only when a project uses a given third-party stack. A module holds both coding and process rules, and both skills read it. The project's `CLAUDE.md` lists which modules apply.

- **`unitask-dotween`:** UniTask instead of coroutines with cancellation tokens; DOTween recycling with awaited tweens; CS4014 on unawaited tween calls; killed tweens completing awaits silently.
- **`editor-tooling`:** IMGUI cannot render emoji; editor code stays out of player builds.

In-house frameworks are not modules: their rules go in each project's `CLAUDE.md`, where every teammate gets them.

## Git and commits

- Commit only when asked, and only what compiles and has been smoke-tested in Unity. Keep pure folder or namespace reorganizations out of feature commits.
- Commit with an explicit pathspec so changes you staged yourself are never swept in.
- Before committing an authored `.asset`, confirm it is saved on disk. Unity holds Inspector edits in memory until the project is saved.
- Scenes and prefabs merge through UnityYAMLMerge, configured in `.gitattributes` and git config.

## Models, effort and subagents

- The session model is the strongest one at your configured effort. A cheaper model is for small-tier mechanical edits only.
- Delegate to a subagent when the output is long and will not be referenced again: log digestion, test-run output, broad codebase sweeps. Stay in the main session for iterative work where phases share context.
- A fast, cheap search agent is fine for locating code. Its conclusions about how a subtle system behaves are not trusted until the main session reads the code.
- Every subagent call counts toward the same usage limits. Parallel subagents multiply cost.
- Agent frontmatter uses model aliases (`opus`, `sonnet`, `haiku`), not full IDs, so it follows new releases.

**Advisor tool (experimental).** `advisorModel` pairs the main model with a stronger model that Claude consults at decision points: before committing to an approach, on a recurring error, before declaring done. The main model still writes every edit, and Claude decides when to consult.

- Not a default. Most Unity work needs the strongest model on every turn, which an advisor does not provide.
- Worth trying for the small tier: a Sonnet session with an Opus advisor instead of plain Sonnet. From the CLI, start it with `--advisor opus` so the setting does not persist; in the desktop app, use `/advisor opus` and `/advisor off` when done.
- Not for high risk: the reviewer agent is the explicit, gated second opinion there.
- Subagents inherit the advisor. Set globally, it gives `console-reader` an Opus advisor and `unity-reviewer` a second Opus.
- `/advisor <model>` saves itself as the default in user settings.
- It is a user setting; the plugin cannot carry it.

## Project overlay

Each project carries two pieces. The `project-setup` skill creates both: it discovers what it can and asks only for what it cannot find.

**Repo `CLAUDE.md`** (shared, neutral facts only, kept short):

- Unity version, render pipeline, target platforms and their status.
- Modules that apply.
- Folder map: gameplay code, configuration data, UI, editor tools, tests, reusable packages.
- Assemblies, how to compile-check and how to run tests.
- Conventions: namespace rules and exceptions, logging facade, in-house framework rules, architecture patterns.
- Where plans and design documents live.
- Project additions to the high-risk list.
- Release tag pattern.

**Project memory** (personal):

- Expected Unity MCP instance name and port.
- Project bug classes and reviewer suppressions.
- Tier adjustments for this project.

## Pre-release checklist

Claude can prepare and check the first five items. The rest need a person. Platforms marked as not yet started in the project's `CLAUDE.md` are skipped.

- [ ] Console shows zero errors and no new warnings since the last release.
- [ ] Test suites pass, with the run reported.
- [ ] Diff of balance, economy and remote-config values since the last release is reviewed.
- [ ] Version and build numbers are bumped per platform.
- [ ] Release builds complete for each shipping platform.
- [ ] Smoke test on a physical device per shipping platform.
- [ ] Performance and memory checked on the lowest-spec target device.
- [ ] Release is tagged in git after store submission.

## Files

Paths are relative to the repository root, which is both the marketplace and the `unity-workflow` plugin.

| Section of this document | File |
| --- | --- |
| Guards | `hooks/hooks.json`, `hooks/guard.js` (Node, so it runs the same under Git Bash and PowerShell) |
| Principles, Task loop, Risk tiers, Verification, Testing policy, Bug fixes, Git and commits, Models | `skills/unity-dev-flow/SKILL.md`, compact, most important rules first, invoked by hand |
| Coding standards | `skills/unity-coding/SKILL.md`, loaded automatically for `.cs` work |
| Modules | `modules/*.md`, read by both skills when the project lists them |
| Review | `agents/unity-reviewer.md` |
| Log digestion | `agents/console-reader.md` |
| Project overlay | `skills/project-setup/SKILL.md` |
| (separate) | `skills/qa-changelog/SKILL.md`, invoked by hand |

## Roadmap

- A handoff skill that writes the state of the current work so a new chat can continue it without losing context.
