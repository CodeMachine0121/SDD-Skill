---
name: commit
description: >
  The commit convention for this workflow — English Conventional Commits, one
  commit per completed TDD cycle or app integration, and no SDD identifiers
  (test-case ids, AC numbers, document names) in the message. Use it standalone
  to stage and commit the current work, or as the convention that /tdd and
  /implement follow for every commit they make.
  Triggered by: "commit", "/commit", "commit this", "write a commit message",
  and by every commit made inside /tdd and /implement.
---

## The four rules

Non-negotiable, and they apply to every commit made in this workflow:

1. **Conventional Commits format** — `type(scope): summary`.
2. **One commit per unit of work** — one completed TDD cycle, or one app integration. Never two, never half of one.
3. **No SDD identifiers** — no test-case id, AC/scenario number, PRD/ARCH filename, or feature-folder date.
4. **English, always** — regardless of the language of the conversation, the spec, or the existing git history.

---

## 1 · Format

```
type(scope): summary

[optional body — why, not what]

[optional footer]
```

| type | Use for |
|---|---|
| `feat` | New behavior visible to a user of the module |
| `fix` | Corrected behavior that was wrong |
| `refactor` | Structure changed, behavior identical |
| `test` | Tests added or changed on their own (rare here — a cycle's test ships with its implementation) |
| `perf` | Faster or lighter, behavior identical |
| `docs` | Documentation, including `.sdd/` documents |
| `build` / `ci` | Build scripts, dependencies, pipeline |
| `chore` | Wiring, config, and everything with no behavioral story |

- **scope** — the module or component the change lives in, in `UL-MAP.md` vocabulary, lowercase (`feat(cart): …`). Omit the parentheses entirely when the change genuinely spans the codebase; do not invent a scope to fill the slot.
- **summary** — imperative mood ("add", not "added"/"adds"), lowercase first word, no trailing period, ≤ 72 characters. It describes the **behavior**, so it should read as the completion of "This commit will …".
- **body** — optional, only when the *why* is not obvious: the constraint that forced this shape, the alternative rejected. Never a restatement of the diff.
- **footer** — `BREAKING CHANGE: …` for an incompatible change (or `type(scope)!: summary`); issue refs if the project uses them.

---

## 2 · One commit per unit of work

| Unit | What goes in it |
|---|---|
| **A completed TDD cycle** | The test **and** its implementation together, plus the refactor done in that cycle. They are one behavior; splitting them produces a commit that is red on its own. |
| **An app integration** | Wiring a component into the composition root / DI container / route table / config. **Its own commit**, separate from the behavior cycles it wires up. |
| **A standalone refactor** | Structure-only moves not belonging to any one cycle. |

Rules that follow from this:

- **Stage only that unit's files.** Never `git add -A` over a dirty tree; name the paths.
- **Never batch cycles.** Three finished cycles are three commits, in the order they were done.
- **Every commit is green and self-contained** — the suite passes at that commit, and it builds without the next one.
- Nothing temporary ships: no debug output, no commented-out code, and no deliberately broken behavior left over from a falsification check.

---

## 3 · No SDD identifiers

The commit log is read by people who have never opened `.sdd/`. Internal numbering means nothing to them, and it rots the moment a document is renumbered.

Banned in the subject, body, and footer: test-case ids (`TC-07`), acceptance-criteria or scenario numbers (`AC-3`, `Scenario 2`), document filenames (`PRD.md`, `ARCH.md`, `BRIEF.md`), and the feature-folder date/slug (`2026-08-25-checkout-discount`).

| ❌ | ✅ |
|---|---|
| `feat: implement TC-04` | `feat(cart): apply member discount to subtotal` |
| `feat(order): AC-2 and AC-3 done` | two commits: `feat(order): reject checkout when cart is empty` / `feat(order): cap discount at order subtotal` |
| `test: add tests per PRD.md section 3` | `feat(pricing): round tax to two decimals` |
| `chore: 2026-08-25-checkout-discount wiring` | `chore(checkout): register discount calculator in DI` |

Describe what the software now does. If the summary cannot be written without a document reference, the commit is too big or the behavior is not yet understood.

---

## 4 · English, always

Write every commit message in English even when the conversation, the PRD, the UL Map, and the entire existing history are in another language. The log is a machine-readable, greppable, tool-parsed artifact shared across contributors and agents; one language keeps `git log --grep`, changelog generation, and review tooling working.

> This **replaces** any "match the existing history's language and prefix style" instinct. Detection is not part of this convention — the format above is the project standard, applied from the first commit onward. Domain nouns keep their `UL-MAP.md` spelling inside an otherwise-English sentence.

---

## Standalone use — `/commit`

When invoked directly on a dirty tree:

1. **Look before staging** — `git status --short`, `git diff`, `git diff --staged`, and `git log --oneline -10` for context. Read what actually changed; never commit a diff you have not seen.
2. **Group into units of work** per the table above. More than one unit → propose the split to the user, then commit them one at a time in dependency order.
3. **Verify green** — run the tests covering the unit (or the suite, if it is fast) before committing. If the tree is red, say so and stop; do not commit red to "save progress".
4. **Stage by path** — only that unit's files. Leave everything else untouched; never revert or stash the user's unrelated work.
5. **Commit** with a message built from the rules above.
6. **Report** — `git log --oneline -5`, plus anything deliberately left uncommitted and why.

Do **not** push, tag, or open a PR unless the user asks. If a pre-commit hook rewrites or rejects the commit, surface its output rather than retrying blindly.

---

## Checklist

- [ ] `type(scope): summary`, imperative, lowercase, ≤ 72 chars, no trailing period
- [ ] English
- [ ] Exactly one unit of work — a cycle (test + implementation) or an app integration
- [ ] Only that unit's files staged
- [ ] No test-case id, AC number, document name, or feature-folder date anywhere in the message
- [ ] Tests green at this commit; nothing temporary or deliberately broken included
