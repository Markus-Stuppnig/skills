---
name: "conventional-commits"
description: "Write standardized Conventional Commit messages following the v1.0.0 specification. Covers structure, atomic commits, the \"why not what\" principle, and common examples."
version: 2
created: "2026-07-13"
updated: "2026-07-13"
---
## When to Use
Use this skill whenever you need to write a commit message that follows the Conventional Commits specification (https://www.conventionalcommits.org/en/v1.0.0/). Suitable for any Git-based project to ensure consistent, machine-readable commit history.

## Procedure
1. **Review changes** – Run `git status` and `git diff` (or `git diff --cached`) to understand what changed and why.
2. **Think atomic** – Each commit should be a single, self-contained logical change. If you find multiple unrelated changes, split them into separate commits.
3. **Write 'why', not 'what'** – The code already shows *what* changed. Describe *why* the change was made, the context, or the motivation. Avoid obvious descriptions like 'fix bug' or 'update code'.
4. **Choose the type** from the allowed list: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
5. **Add an optional scope** in parentheses (e.g., `feat(parser):`) to indicate the module or area affected.
6. **Write a short description** in imperative mood (e.g., 'add', 'fix', 'remove', not 'added', 'fixed', 'removed'). Keep it under 50 characters if possible.
7. **Add a body** (optional) for deeper context: why this change was necessary, trade-offs considered, alternative approaches rejected.
8. **Add a footer** (optional) for breaking changes (`BREAKING CHANGE: ...`) or issue references (`Closes #123`, `Refs #456`).
9. **Construct the final message** and commit with `git commit -m "type(scope): description"` (add `-m` for body/footer if needed).

## Pitfalls
- Do not describe what the code does – the diff already shows that. Focus on the motivation, context, or effect of the change.
- Do not lump unrelated changes into one commit. If a change touches two concerns, split them.
- Do not use past tense in the description ('added', 'fixed', 'removed'). Use imperative mood ('add', 'fix', 'remove').
- Keep the first line (subject) concise. Long subjects are truncated in many Git tools.
- Do not capitalize the first letter after the type/scope prefix ('fix(ui): correct' not 'fix(ui): Correct').
- Do not add a period at the end of the subject line.

## Verification
1. The commit message follows the pattern: `type(scope): description`
2. The type is one of the allowed values: feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert
3. The description is in imperative mood and explains *why*, not *what*
4. Breaking changes are marked with `BREAKING CHANGE:` or `!` after the type/scope
5. Each commit represents exactly one atomic logical change
6. Total message passes `git log --oneline` readability: the subject alone should be meaningful

## Examples
| Type | Scope | Description | Why (motivation) |
|------|-------|-------------|-----------------|
| `feat` | `auth` | add OAuth2 login flow | replaces password-based auth for SSO compliance |
| `fix` | `parser` | handle empty input gracefully | previously caused a silent crash on EOF |
| `docs` | | document API rate limits | users kept hitting 429 without knowing the cap |
| `refactor` | `db` | extract query builder into own module | prepares for multi-database support |
| `perf` | `cache` | replace LRU with LFU eviction | access pattern is frequency- not recency-based |
| `test` | `api` | add contract tests for /users endpoint | caught 3 regressions in the last sprint |
| `chore` | | bump go to 1.22 | required by security patch CVE-2025-... |
| `feat` | | send email on registration | `!` or footer `BREAKING CHANGE: email service is now required` |
| `style` | `ui` | format with Prettier 3.0 | aligns with team-wide formatting rule |
| `ci` | | pin GitHub Actions runners to ubuntu-24.04 | ubuntu-latest broke our build twice this month |

### Bad examples (what *not* to do)

| ❌ Wrong | ✅ Right | Reason |
|----------|----------|--------|
| `fix: fixed a bug` | `fix(api): validate user ID before DB lookup` | "fixed a bug" says *what*, not *why*; past tense |
| `feat: add new feature` | `feat(export): add CSV export` | "new feature" is vague |
| `refactor: clean up code` | `refactor(auth): extract token validation into middleware` | "clean up" doesn't explain the benefit |
| `chore: update deps` | `chore: bump lodash to 4.17.21` | closes security advisory GHSA-... |
