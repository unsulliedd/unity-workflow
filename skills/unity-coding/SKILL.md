---
name: unity-coding
description: Unity C# coding standards. Use when writing, reviewing, or refactoring Unity C# code. Apply whenever generating or modifying any .cs file in a Unity project.
argument-hint: "file-path or task description"
paths: ["**/*.cs"]
---

# Unity C# Coding Standards

Apply these standards to all code written or reviewed in a Unity project.

**Project additions.** If the project's `CLAUDE.md` lists modules, read each one from `${CLAUDE_PLUGIN_ROOT}/modules/<module>.md` before writing code. Modules add stack-specific rules (async conventions, editor GUI limits). The project's `CLAUDE.md` also holds its folder layout, logging facade, namespace rules and known exceptions. Project and user instructions override the defaults below.

---

## Null Checks

**Unity objects must never use `?.` or `??`.**
Unity overrides the `==` operator on `UnityEngine.Object`. The null-conditional operators bypass this override and produce incorrect behavior on destroyed objects.

```csharp
// WRONG
myComponent?.DoSomething();
var val = myObject ?? fallback;

// CORRECT
if (myComponent != null) myComponent.DoSomething();
if (myObject != null) { ... } else { ... }
```

**Pure C# objects:** `?.` and `??` are fine.

**Constructor-assigned variables do not need null checks.** If a dependency is assigned in a constructor or via `[SerializeField]` and validated once in `Awake`, callers trust it is valid. Do not add null guards on every usage.

---

## Find & GetComponent

**Never use Find methods:**
- `GameObject.Find()`
- `FindObjectOfType<T>()`
- `FindObjectsOfType<T>()`
- `transform.Find()`

These are expensive, string-based, and fragile. Use dependency injection, `[SerializeField]` references, or event channels instead.

**Never call `GetComponent<T>()` in `Update`, `FixedUpdate`, or any hot path.**
Cache component references in `Awake` or `OnEnable`.

**Prefer `TryGetComponent` over `GetComponent` + null check:**
```csharp
// PREFERRED
if (TryGetComponent(out Rigidbody rb))
{
    // use rb
}

// AVOID
var rb = GetComponent<Rigidbody>();
if (rb != null) { ... }
```

**Never compare `tag` with `==` — use `CompareTag`:**
```csharp
// WRONG — allocates a string
if (other.tag == "Enemy") { }

// CORRECT
if (other.CompareTag("Enemy")) { }
```

---

## LINQ

**LINQ is not allowed in runtime hot paths** (any method called frequently: `Update`, `FixedUpdate`, async loops, per-frame logic, per-entity logic).

LINQ is acceptable in:
- Editor-only code (`#if UNITY_EDITOR` or an `Editor` folder)
- One-time initialization (called once at startup or scene load)
- Lightweight, infrequent operations — **if unsure, ask before using it**

```csharp
// WRONG — allocates in Update
var enemies = unitTracker.GetAll().Where(u => u.IsAlive).ToList();

// CORRECT — use pre-allocated lists and manual loops
for (int i = 0; i < _enemies.Count; i++)
{
    if (_enemies[i].IsAlive) { ... }
}
```

---

## MonoBehaviour Usage

**Minimize MonoBehaviour.** A MonoBehaviour is only justified when you need:
- Unity lifecycle hooks (`Awake`, `OnEnable`, `OnDisable`, `OnDestroy`)
- Scene-attached components (`[SerializeField]` fields, Gizmos)
- Physics callbacks (`OnTriggerEnter`, `OnCollisionEnter`)

All logic, data processing, and state management belongs in **plain C# classes** (handlers, services, strategies). Follow the composition pattern the project already uses.

Async work accepts and forwards a `CancellationToken` so it stops when its owner is destroyed. The project's modules define which async library to use.

---

## Inspector Fields

Use `[SerializeField] private` instead of `public` for inspector-exposed fields. Keeps the public API surface clean.

```csharp
// WRONG
public float speed = 5f;

// CORRECT
[SerializeField] private float _speed = 5f;
```

**Renaming a serialized field loses its value** on every existing scene, prefab and asset. By default, keep the value with `[FormerlySerializedAs]`, unless the project or user instructions say renamed fields are re-wired by hand. Either way, list the rename in the hand-over.

```csharp
[FormerlySerializedAs("_settings")]
[SerializeField] private GameConfig _config;
```

---

## Magic Numbers

No inline literals for gameplay values. Use `const`, `static readonly`, or ScriptableObject config fields.

```csharp
// WRONG
if (health < 0.2f) { ... }

// CORRECT
private const float LowHealthThreshold = 0.2f;
if (health < LowHealthThreshold) { ... }
```

---

## Naming Conventions

Microsoft C# conventions. Apply to new code; do not propagate legacy `SCREAMING_SNAKE_CASE` constants found in older files.

| Element | Convention | Example |
|---|---|---|
| Class / struct / enum / interface | PascalCase, interface prefixed `I` | `SpawnManager`, `IDamageable` |
| Method (any access level) | PascalCase | `private void RecalculateStats()` |
| Property | PascalCase | `public bool IsActive { get; }` |
| Private / protected field | `_camelCase` | `private float _moveSpeed;` |
| `[SerializeField]` field | `_camelCase` | `[SerializeField] private Canvas _overlayCanvas;` |
| Public field (avoid — see Inspector Fields) | PascalCase if unavoidable | `public float Speed;` |
| Constant (`const`, `static readonly`) | PascalCase | `private const float SpawnTweenDuration = 0.25f;` |
| Local variable / parameter | camelCase | `int activeSlotCount` |
| Enum member | PascalCase, no prefix | `enum StepType { Click, DragDrop }` |
| Boolean member | PascalCase, prefixed `Is`/`Has`/`Can`/`Should` | `IsActive`, `HasTarget` |
| Async method | PascalCase, suffixed `Async` | `private async UniTaskVoid LoadLevelAsync()` |
| Event / UnityEvent field | PascalCase, named for the action | `OnClicked`, `OnDragged` |

**Acronyms:** two-letter acronyms stay fully capitalized (`UI`, `ID`, `IO`); three-or-more-letter acronyms are PascalCase like a normal word (`Json`, `Http`, not `JSON`, `HTTP`).

**One file, one primary type** — the file name matches the public type it declares (`SpawnManager.cs` declares `SpawnManager`).

```csharp
// WRONG
private const string CHARACTER_CARD_INDEX = "CharacterCardIndex";
public float Health_Points;
private float MoveSpeed;

// CORRECT
private const string CharacterCardIndex = "CharacterCardIndex";
public float HealthPoints;
private float _moveSpeed;
```

---

## Namespaces & Folder Organization

Namespaces mirror the folder path, even where no `.asmdef` makes them an assembly boundary. Check the project's `CLAUDE.md` for which assemblies exist and for any segments the project drops or renames (organizational folders such as `Scripts` or `Runtime` are common candidates).

```
<Root>/Systems/Inventory/Components/ItemSlot.cs     → <Root>.Systems.Inventory.Components
```

**When creating a new file:** match the namespace of the closest existing sibling file in the same folder first. Only fall back to deriving it from the path if the folder is empty. Known exceptions listed in the project's `CLAUDE.md` are not copied into new code and not "fixed" in passing — fixing them is a separate, deliberate refactor.

---

## Comments & Documentation

**XML doc comments (`///`) are required on every `public` and `protected` method and property.** Use the full block, not a one-line summary:

```csharp
/// <summary>
/// Applies damage after armor reduction.
/// </summary>
public float TakeDamage(float amount) { ... }

/// <summary>
/// Recalculates derived stats when a base stat or modifier changes.
/// </summary>
protected virtual void RecalculateStats() { ... }
```

**Private methods get no XML doc.** Use a plain `//` line above the method only if the logic is genuinely non-obvious (a hidden constraint, a subtle invariant, a workaround for a specific bug). Do not comment what the code already clearly says, and do not add a comment to every private method by default.

```csharp
// Clamped to avoid a negative remainder on the last wave when enemyCount < waveSize.
private int GetRemainingEnemies() { ... }

private void ResetCooldown() { ... } // self-explanatory — no comment needed
```

**Comments are technical documentation, not conversation.** Write them as if for a stranger reading the code with no project history:
- Never use first person (`I`, `we`, `let's`) or talk to a reader as "you."
- Never reference the current task, a chat session, a request, an author, a date, or "why we changed this now." Explain what the code does and any non-obvious constraint — not the history of how it got there.
- Never use change-marker comments (`// new`, `// added`, `// changed`, `// updated`, `// fix for X`). Version control tracks history, not comments.
- Never cite compiler diagnostics (`CS0229`, "is a compile error", "would not compile") in XML docs or inline `//` comments. Explain the design reason the member exists, not the error it avoids.

```csharp
// WRONG — conversational / session-referential
// I added this check because we talked about the null bug earlier
if (target == null) return;

// CORRECT — states the invariant, nothing else
// Target can be destroyed mid-tween; bail instead of animating a dead reference.
if (target == null) return;
```

**No commented-out code.** Delete it; version control keeps the history.

**Permitted tags:** `TODO`, `FIXME`, `HACK`, `NOTE`, `OPTIMIZE` — used sparingly, stated as a fact about the code, not a note to a teammate.

---

## Commit Messages

**Follow the repository's existing style.** Read `git log --oneline -20` (and a few full messages) before writing one, and match its subject format, tense, prefixes and body layout. Project or user instructions override this.

Without an established style:
- Subject: imperative, sentence case, under about 72 characters.
- Body: what changed and the non-obvious reason, as short bullets when there is more than one point.

**Describe the state the commit lands, not a diff against a prior version — unless that prior version actually exists in `git log`.** For a file with no history yet, phrasing like "move X", "rename Y", "instead of the old W" implies a contrast that isn't in the repo. Say what the code *is* and *does*.

**Same voice as code comments** (see Comments & Documentation above): no first person, no reference to a task, chat session or request.

---

## Task

$ARGUMENTS
