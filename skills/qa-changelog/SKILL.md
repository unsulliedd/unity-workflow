---
name: qa-changelog
description: Generates a milestone changelog from git history - a short player-facing "What's new" for the stores and a QA verification document grouped by feature area. Use when asked for a changelog, release notes or a QA test plan between releases.
disable-model-invocation: true
argument-hint: "[from-ref] [to-ref]"
---

# Milestone Changelog

Builds one document for a range of commits: player-facing release notes plus a QA verification plan. Both describe the **latest state** of the game at the end of the range.

## 1. Resolve the range

- `from` = `$0` if given, otherwise the most recent release tag (`git tag --sort=-creatordate`; the project's `CLAUDE.md` may name the tag pattern).
- `to` = `$1` if given, otherwise the development branch head (`develop` if it exists, else the current branch). If there is no release tag, ask for the range.
- Count the commits with `git rev-list --count --no-merges <from>..<to>`.
- State the range and count, and wait for confirmation before reading.

## 2. Read the history

- `git log --no-merges --reverse --format="%h %ad %an%n%s%n%b%n---" --date=short <from>..<to>`. Commit messages are the primary source; include every author.
- More than about 150 commits: read in chunks of 100 and keep running notes, without stopping between chunks.
- Open a commit (`git show --stat <hash>`, then the relevant diff) only when its message does not make the player- or tester-visible effect clear.

## 3. Reduce to the latest state

- Group commits by feature area (for example Combat, Inventory, Save System, UI and Localization).
- A later commit that refines, changes or fixes an earlier one replaces it; describe only the final behavior.
- Drop anything added and removed or reverted inside the range.
- Drop what a player or tester cannot observe: refactors, cleanups, folder moves, editor-only tooling, internal logging.
- A fix for a bug that never shipped (introduced and fixed inside the range) is not a fix; it is part of the feature.

Show the outline (areas, items per area, items dropped and why in one line each) and wait for confirmation before writing.

## 4. Write the document

Path: `<docs folder>/Changelog/<to-tag or from..to>.md`, using the project's documentation folder (default `Docs/`). If the file exists, update it rather than replacing it.

**No code in either section:** no file names, class names, namespaces, method names or code symbols. Describe what the player does and what happens.

```markdown
# <Milestone or range> — Changelog

Range: `<from>`..`<to>` (<N> commits, <first date> – <last date>)

## What's new

<Player-facing, plain language. Short bullets, most exciting first. No internal names, no "bug fixes and improvements" filler unless nothing else qualifies. If the project ships to stores, fit the strictest target's release-notes limit (for example Google Play: 500 characters).>

## QA verification

### <Feature area>

**<Item>**
- What to expect: <player action or trigger, and the visible result>
- QA check:
  1. <step>
  2. <step>
  3. <expected result>

## Needs clarification

- <items whose visible effect could not be determined from history, with the commit hash>
```

## 5. Finish

Report the file path and the character count of "What's new". Do not commit; ask whether to commit the document.
