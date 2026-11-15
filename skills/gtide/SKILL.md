---
name: gtide
description: Drive Git through Git Tide (`gtide`) instead of raw git. Use whenever the task involves git in a repo that has a `.tide.yml`, or the user mentions gtide, Git Tide, branches, commits, pushing, pull requests, merge requests, tags, undoing a commit, stashing, or starting/cloning a project. Gives the shortest safe command for each everyday Git task and the rules gtide enforces.
---

# Git Tide (`gtide`) for AI agents

`gtide` wraps Git with guard rails: protected branches, Conventional Commits, secret scan, no silent data loss. **Prefer it over raw `git` for everything it covers.** Fall back to `git` only for what it does not cover (see the end).

## Ground rules

1. **Look first.** Run `gtide status` before changing anything. Local only, no network.
2. **Never bypass.** No `--no-verify`, no `git push --force`, no editing `.git/hooks`. If gtide refuses, read the message and the hint, then fix the cause. Refusals are policy, not bugs.
3. **One logical change per commit.** Message is English Conventional Commits: `type(scope): subject`.
4. **Never work on a protected branch** (usually `main`, `release/*`). Create a branch.
5. **Ask the user before anything outward-facing or hard to undo:** `push --force`, `tag ... --push`, `tag delete --remote`, `branch delete --force`, `pr`. These also prompt; do not pass `--yes` unless the user said to.
6. **Never print or commit secrets.** `secret.tide.yml` and `.env` stay local. `gtide save` runs a secret scan; if it trips, remove the secret, do not skip the check.

## Everyday flow (copy this)

```bash
gtide status                                   # where am I, what changed
gtide create feature/short-name                # branch from fresh base
# ...edit...
gtide save -m "feat(scope): what it does" -f path/a -f path/b
gtide update                                   # bring base branch in (when behind)
gtide push                                     # publish, sets upstream
gtide pr --draft                               # open PR/MR (needs user OK)
```

`save` without `-f` stages **every** change. Prefer `-f` with explicit paths so unrelated edits stay out.

## Task → command

| Task | Command |
|---|---|
| See state | `gtide status` |
| See changes | `gtide diff` (per file), `--patch` (full), `--staged` (what would be committed), `-f <path>` |
| Start a branch | `gtide create <name>` (`--local` offline, `--track` for an existing remote branch) |
| Switch branch | `gtide change <name>` (partial name ok, `-` = previous, `--carry` to take dirty files) |
| Commit all | `gtide save -m "type: msg"` |
| Commit some | `gtide save -m "type(scope): msg" -f file -f dir/` |
| Commit what is already staged | `gtide commit -m "type: msg"` |
| Fix last commit (not pushed) | `gtide amend -f forgotten.go` or `gtide amend -m "new msg"` |
| Take last commit back (not pushed) | `gtide undo` (changes stay staged) |
| Put work aside | `gtide stash -m "note"`, then `gtide stash pop` |
| Get remote changes of this branch | `gtide pull` |
| Get base branch changes | `gtide update` |
| Publish | `gtide push` |
| Pull/Merge Request | `gtide pr` (`--draft`, `--title`, `--body`, `--base`) |
| Review someone's PR | `gtide review <number>` (`--diff` for the patch) |
| List PRs | `gtide pr list`, `gtide pr view <n>`, `gtide pr status` |
| Commit log | `gtide history` (`--me`, `--since 7d`, `-f <file>`, `--type feat`, `<from>..<to>`; `-f <file> --patch` shows what each commit changed) |
| Branch overview / cleanup | `gtide branch`, `gtide branch --merged`, `gtide branch delete <name>` |
| Rename branch | `gtide branch rename <old> <new>` |
| Tags | `gtide tag`, `gtide tag create v1.2.0 -m "Release 1.2.0" --push` |
| New project | `mkdir app && cd app && gtide init --yes --remote <url>` |
| Clone | `gtide clone <url>` |
| Remote | `gtide remote` (show), `gtide remote add <url>`, `gtide remote set-url <url>` |
| Merge/rebase conflict | edit files, `git add`, then `gtide continue`; give up with `gtide abort` |
| Something odd | `gtide doctor` |

Run `gtide <command> --help` when unsure: the help has examples.

## Reading state as data (gtide 1.6.0+)

When you need to decide something from the repo state, prefer `--json` over parsing the table: stdout is only JSON, notes go to stderr, an empty result is `[]` or an object. Available on `status`, `diff` (`--patch` adds the patch), `branch`, `history` (`--patch` adds what each commit changed, with `-f` only that file), `history show`, `tag` and `stash list`. Failure is the exit code, with the message on stderr. The shapes are in the gtide repo's `docs/json.md`.

```bash
gtide status --json                       # branch, upstream/base distance, files with staged/unstaged/conflict flags
gtide history -f src/login.go --patch --json   # who changed a file, and what each commit changed in it
```

## Commit messages

Format `type(scope): subject`. Types: `feat fix docs style refactor perf test build ci chore revert`.
Imperative, lowercase after the colon, no trailing period, aim ≤ 50 chars (limit 72). Use `--body` for the *why* when it is not obvious. Alternatively `--type fix --scope api -m "handle empty token"`.

Never add AI attribution lines (`Co-Authored-By`, "Generated with ...") unless the user explicitly asks.

## When gtide refuses (common cases)

| Message says | Do |
|---|---|
| protected branch | `gtide create <name>`, redo the work there. Exception: the very first commit/push of a brand new repo is allowed |
| uncommitted changes would be at risk | `gtide save` them or `gtide stash`, re-run |
| a merge/rebase is in progress | resolve, `git add`, `gtide continue` (or `gtide abort`) |
| commit already on the remote (undo/amend) | history is published. Make a new commit with `gtide save` instead |
| conflicts while integrating base | resolve files, `git add`, `gtide continue`; do not guess at conflict resolution, ask the user if the intent is unclear |
| invalid commit message | fix to Conventional Commits |
| secret detected | remove it, rotate it if it was ever committed or pushed, tell the user |
| no remote | `gtide remote add <url>` |
| tag refused | tags: semver names only, protected branch only, HEAD must be on the remote, never moved |

## Tags and releases

`gtide tag create vX.Y.Z --push` can start a release build in CI. **Always confirm with the user first.** Pick the version by SemVer from the commits since the last tag: `gtide history <last-tag>..HEAD --summary` / `--type feat`.

## Not covered by gtide

Use plain `git` (read-only first, and ask before anything destructive) for: `git log --graph`, `git blame`, `git bisect`, `git cherry-pick`, `git revert`, `git reset --hard`, `git rebase -i`, submodules, worktrees. `git reset --hard`, `git clean -fd`, `git checkout -- .` destroy work: never run them without explicit user approval.

## Config

`.tide.yml` (shared, committed, **per branch**): base branch, protected branches, remote, provider, `update.strategy` (merge or rebase). `secret.tide.yml` (local, git-ignored): provider token reference and secret patterns. Inspect with `gtide config`. Tokens: `GITHUB_TOKEN` / `GITLAB_TOKEN`.

Output is English. Hints after an error name the exact next command: follow them.
