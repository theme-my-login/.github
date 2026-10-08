# Contributing

## No AI-attribution in commits or PRs

Don't include an AI-assistant attribution trailer (e.g. a `Co-Authored-By:` line naming an AI tool) or a "Generated with ..." footer in any commit or PR in this repo.

## Commits and PR titles

This repo uses [Conventional Commits](https://www.conventionalcommits.org/) for commit messages. PRs merge into `master` as real merge commits, not squashed, so every individual commit lands on `master` verbatim and is what release automation actually scans.

Format: `type: subject` (lowercase after the colon, no trailing period, no ticket-tracker prefix — reference a GitHub issue in the body with `Fixes #N` if one applies). This applies per-commit: squash together commits that are really one change split across saves before opening a PR (see Squashing below), but every commit that survives still needs its own correct type — CI lints each commit individually.

**PR titles are plain English, not Conventional Commits format** (e.g. `Fix favorites list pagination on multisite`, not `fix: ...`). GitHub embeds the PR title in the merge commit it creates, and a conventional-format title there gets double-counted by release automation as a duplicate commit.

Allowed types:

| Type | Use for |
|---|---|
| `feat` | User-facing functionality |
| `fix` | User-facing bug fix |
| `chore` | Dev tooling, dependency bumps, anything with no user-facing effect |
| `ci` | CI/pipeline changes |
| `docs` | Documentation only |
| `build` | Build process/tooling |
| `perf` | Performance improvement |
| `refactor` | Code change with no behavior change |
| `test` | Test-only changes |

`feat`/`fix`/`perf`, and anything marked breaking, are the only types that ship in the changelog and trigger a version bump. The rest are invisible to users by design.

**`fix` is reserved for user-facing bug fixes.** A change that fixes a bug in internal tooling, scripts, CI, or build config is `chore`/`refactor`/`test`/`ci`, not `fix`, even though it "fixes" something — check the file path (anything under `.github/workflows/`, CI config, build tooling) against this before typing the type.

**A distribution-listing-only change is always `chore`, even when user-facing** (e.g. correcting the `Tags`/`License` fields or `Tested up to` in a WordPress.org `readme.txt`, where this repo has one). That file deploys independently of the plugin/extension build, so it doesn't need a version bump to reach users.

**Branch names are conventionally prefixed with the same type**, e.g. `fix/nonce-check-on-favorites-widget`, `feat/passwordless-login`, `chore/bump-cssnano`. This one is a readability nicety, not a rule: nothing reads the branch name, a branch carrying commits of more than one type has no single correct prefix, and the branch is deleted on merge anyway. **Don't request changes on a pull request over it** — renaming a branch closes its pull request rather than retargeting it, so the correction costs a replacement PR and another round of CI for no functional gain.

### Squashing

Before opening a PR, squash commits that are really incremental edits to the same change (fixup-style saves) into one. Leave commits separate when a PR genuinely contains multiple distinct logical changes — that's a per-PR judgment call, not something CI enforces or a repo-wide setting forces.

### Major version bumps

A major bump isn't manual — it's the same commit-driven mechanism as `feat`/`fix`, just marked as breaking. Either:

- Add `!` after the type/scope: `feat!: drop support for PHP 7.4`
- Or add a `BREAKING CHANGE:` footer to the commit body (any type):

```
fix!: remove deprecated helper in favor of its replacement

BREAKING CHANGE: the deprecated helper has been removed; use its replacement instead.
```

To force a specific version regardless of what the commits would otherwise compute (e.g. a deliberate version jump), add a `Release-As: X.Y.Z` footer to any commit.

### Commit subject wording

For any type that ships in the changelog (`feat`/`fix`/`perf`, or anything marked breaking), the subject itself has to read as something a user would actually see — describe the user-facing effect, not the internal mechanism. If the subject contains a function name, option key, filename, or shortcode/hook name, that's the tell to rewrite it around what breaks, works, or is now safe.

`fix: avoid double-firing tml_activate_favorites on multisite` fails this bar even though it's accurate; `fix: stop the favorites list from occasionally not appearing on network sites` clears it.

If this repo has its own changelog-drafting script (check for `bin/draft-changelog.php` before relying on this — most repos don't have one), you can keep the subject technical and add a `Release-Note:` trailer with the user-facing wording instead:

```
fix: avoid double-firing login_redirect during password reset

Release-Note: Prevent site crash on Bluehost during password reset.
```

Without that script, there's no override — the subject alone is what ships to users, so it has to clear the bar on its own.

With that script, `Release-Note: none` leaves a commit out of the changelog entirely while it still bumps the version. Use it for a real user-facing change that isn't worth announcing, such as a promotional notice. Keep the honest type (`feat`, not `chore`) rather than retyping the commit to hide it.

If a change is purely internal (refactor, test coverage, tooling), it should be `chore`/`refactor`/`test`, not a reworded `fix`.

## PR mechanics

- All changes require a PR to `master` — no direct commits, even temporarily. Create the type-prefixed branch *before* the first commit, not retrofitted after.
- Write PR descriptions in first person, describing what changed and why — not third-person narration about the author.
