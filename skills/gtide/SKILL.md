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
| Copy commits from another branch (1.12.0+) | `gtide pick <sha>`, `gtide pick <from>..<to>` (oldest first; skips what is already there; conflicts: `gtide continue` / `gtide abort`) |
| Bring files from another ref, no merge (1.12.0+) | `gtide restore --from origin/staging -f path -f dir/` (staged; then `gtide save`; asks before overwriting local edits) |
| Take back a commit, even pushed (1.12.0+) | `gtide revert <sha>` (new commit; `undo` is for unpushed ones) |
| Refresh remote branches only (1.12.0+) | `gtide fetch [remote]` (touches no local branch or file) |
| Put work aside | `gtide stash -m "note"`, then `gtide stash pop` |
| Get remote changes of this branch | `gtide pull` |
| Get base branch changes | `gtide update` |
| Publish | `gtide push` |
| Pull/Merge Request | `gtide pr` (`--draft`, `--title`, `--body`, `--base`) |
| Review someone's PR | `gtide review <number>` (`--diff` for the patch) |
| List PRs | `gtide pr list`, `gtide pr view <n>`, `gtide pr status` |
| Review a PR without checkout (gtide 1.11.0+, GitHub) | `gtide pr diff <n>` (`--file`), `gtide pr comments <n>`, `gtide pr commit <n> <sha>`; waiting on you: `gtide pr list --reviewer me` |
| Take part in a review (1.11.0+) | `gtide pr comment <n> --body "…"` (`--file f --line N` for a line, `--reply-to <id>`), `gtide pr approve <n>`, `gtide pr request-changes <n> --body "…"` (not on your own PR) |
| Change reviewers, assignees, labels (1.11.0+) | `gtide pr options` lists them, then `gtide pr edit <n> --add-reviewer a --add-label bug --remove-assignee b` |
| Commit log | `gtide history` (`--me`, `--since 7d`, `-f <file>`, `--type feat`, `<from>..<to>`; `-f <file> --patch` shows what each commit changed) |
| Branch overview / cleanup | `gtide branch`, `gtide branch --merged`, `gtide branch delete <name>` |
| Compare two branches (Git 2.40+) | `gtide history --compare <from> <to>`: what `<from>` has that `<to>` lacks, and which commits would conflict (exit 6 when one does; `--json` still prints) |
| Rename branch | `gtide branch rename <old> <new>` |
| Tags | `gtide tag`, `gtide tag create v1.2.0 -m "Release 1.2.0" --push` |
| New project | `mkdir app && cd app && gtide init --yes --remote <url>` |
| Clone | `gtide clone <url>` |
| Remote | `gtide remote` (show), `gtide remote add <url>`, `gtide remote set-url <url>` |
| Merge/rebase conflict | edit files, `git add`, then `gtide continue`; give up with `gtide abort` |
| Something odd | `gtide doctor` |

Run `gtide <command> --help` when unsure: the help has examples.

## Output as data: `--json` (gtide 1.10.0+)

Every command takes `--json`. Use it instead of parsing text, for reads and for actions.

Rules that always hold:
- stdout is only the JSON document. Progress, notes and warnings go to stderr.
- Failure is the exit code (`0` ok, `1` general, `2` usage, `3` config, `4` policy, `5` external, `6` unsafe repo) with stdout empty. The message and hint are on stderr: read them, do not retry blindly.
- Exceptions that still print a document on a non-zero exit: `doctor --json` (exit 1) and `config validate --json` (exit 3).
- Lists are `[]`, never `null`. New fields may appear; ignore the ones you do not know.
- Pass `--yes` with `--json` when a command would otherwise ask; there is nobody to answer a prompt.

Reads: `status`, `diff` (`--patch`), `branch`, `history` (`--patch`, `-f <file>`), `history show`, `tag`, `stash list`, `pr list`, `pr view`, `pr status`, `pr diff`, `pr comments`, `pr commit`, `pr options`, `config show`, `config validate`, `remote show`, `doctor`, `security`, `secrets scan`.

Actions (`save`, `amend`, `undo`, `pick`, `revert`, `restore`, `fetch`, `push`, `pull`, `update`, `create`, `change`, `pr`, `review`, `continue`, `abort`, `tag create|push|delete`, `branch delete|rename`, `pr comment|approve|request-changes|edit`, `stash`, `stash pop|drop`, `remote add|set-url`, `init`, `clone`) print one envelope:

```json
{"ok": true, "action": "save", "branch": "feature/x", "head": "abc1234", "details": {}}
```

`details` depends on the action (commit hash and files for `save`, the request URL for `pr`, and so on). Read `branch` and `head` to confirm the state after the action instead of running `status` again.

```bash
gtide status --json                              # branch, upstream/base distance, files with flags
gtide save -m "fix(api): handle empty token" -f api/token.go --json
gtide pr view 42 --json                          # state, review, files, commits of a request
gtide history -f src/login.go --patch --json     # who changed a file, and what they changed
```

Older gtide: reads on `status`, `diff`, `branch`, `history`, `tag`, `stash list` need 1.6.0; the rest needs 1.10.0. If `--json` is rejected, say which version is installed (`gtide version`) and fall back to the text output. The shapes are in the gtide repo's `docs/json.md`.

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
| commit already on the remote (undo/amend) | history is published. Make a new commit with `gtide save`, or take it back with `gtide revert <sha>` |
| pick refused: message or secret | the source commit breaks the rules; fix it on its own branch, do not work around it |
| conflicts while integrating base | resolve files, `git add`, `gtide continue`; do not guess at conflict resolution, ask the user if the intent is unclear |
| invalid commit message | fix to Conventional Commits |
| secret detected | remove it, rotate it if it was ever committed or pushed, tell the user |
| no remote | `gtide remote add <url>` |
| tag refused | tags: semver names only, protected branch only, HEAD must be on the remote, never moved |

## Tags and releases

`gtide tag create vX.Y.Z --push` can start a release build in CI. **Always confirm with the user first.** Pick the version by SemVer from the commits since the last tag: `gtide history <last-tag>..HEAD --summary` / `--type feat`.

## Not covered by gtide

Use plain `git` (read-only first, and ask before anything destructive) only for what has no command: `git log --graph`, `git blame`, `git bisect`, `git merge-base`, `git reset --hard`, `git rebase -i`, submodules, worktrees. Before gtide 1.12.0 that list also had `cherry-pick`, `revert`, `checkout <ref> -- <files>` and a bare `fetch`; with an older gtide, use git for those.

Everything gtide does cover goes through gtide, **reads included**: `gtide status` not `git status`, `gtide diff`, `gtide history` (`--me`, `--since`, `-f <file>`, `<from>..<to>`), `gtide branch`, `gtide save -f <files>` instead of `git add` + `git commit`. The safety rules live there.

`git reset --hard`, `git clean -fd` and `git checkout -- .` destroy work: never run them without explicit user approval.

## Config

`.tide.yml` (shared, committed, **per branch**): base branch, protected branches, remote, provider, `update.strategy` (merge or rebase). `secret.tide.yml` (local, git-ignored): provider token reference and secret patterns. Inspect with `gtide config`. Tokens: `GITHUB_TOKEN` / `GITLAB_TOKEN`.

Output is English. Hints after an error name the exact next command: follow them.
