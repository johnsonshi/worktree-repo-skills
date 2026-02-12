---
name: macos-linux-worktree-repo-operations
description: "macOS and Linux guidance for working in bare Git repositories with worktree branch operations. Covers worktree branch management (creating, checking out, removing, and listing worktree branches), and GitHub PR/issue workflows via gh CLI. Use this skill whenever performing git operations, creating or switching worktree branches, checking out code into worktree branches, navigating the repository structure, viewing or creating PRs, reviewing PR comments/diffs, or working with GitHub issues on macOS or Linux. Triggers on any git workflow task including: worktree branch creation, worktree branch checkout, worktree management, committing, pushing, or any operation that assumes a working tree. Also triggers on GitHub tasks including: viewing PRs, listing PRs/issues, creating PRs, reviewing PR diffs or comments, and any gh CLI usage. Also use when the user asks about the repo layout or how to work with worktree branches."
---

# macOS / Linux Worktree Bare Repo Operations

> **Note**: The `AGENTS.md` file at the repository root describes the overall repo structure and purpose. This skill provides the detailed worktree branch operations guide that `AGENTS.md` references.

## Prerequisites — Confirm You Are in a Bare Repo

This skill only applies to bare Git repositories using worktrees. **Not everyone on the team uses this workflow.** Before following any instructions in this skill, verify you are in a bare repo:

```bash
git config --get core.bare
```

- If the output is `true` → you are in a **bare repo**. This skill applies — continue reading.
- If the output is `false` (or the command runs from a normal clone) → you are in a **standard clone**. Use normal git commands (`git checkout`, `git switch`, etc.) as usual. **Do not use this skill.**

## Environment

- **OS**: macOS or Linux.
- **Shell**: The Bash tool in Claude Code runs under the default shell (zsh on macOS, bash on most Linux distributions). Use standard Unix paths (e.g., `/home/<username>/...` on Linux, `/Users/<username>/...` on macOS).
- **Paths**: All tools (Bash, Read, Edit, Glob) use the same standard Unix path format. There are no dual-path conventions on macOS or Linux.
- **Repo type**: Bare clone (`--bare`). There is no default working tree.
- **Repo path**: `/<home-dir>/<username>/<path-to-repo>/<repo-name>.git` (where `<home-dir>` is `/Users` on macOS or `/home` on Linux)

## Repo Layout

The bare repo root contains only Git internals and the `branches/` directory for worktree checkouts. It does **not** contain a working tree — all source code, skills, project files, and agent configuration files (`AGENTS.md`, `CLAUDE.md`) live inside the worktree checkouts under `branches/`.

```
<repo-name>.git/                        # Bare repo root (no working tree here)
├── branches/                           # All worktree checkouts live here
│   ├── main/                           # worktree for the main branch
│   │   ├── .github/
│   │   │   └── skills/                 # Skills live inside worktrees
│   │   │       └── <skill-name>/
│   │   │           └── SKILL.md
│   │   ├── AGENTS.md                   # Agent instructions containing repository purpose and structure (checked into the repo)
│   │   ├── CLAUDE.md -> AGENTS.md      # Symlink: CLAUDE.md points to AGENTS.md
│   │   └── ...                         # All other repo files (source code, docs, etc.)
│   ├── path/to/<branch-name-1>/        # one worktree directory per checked-out branch
│   └── path/to/<branch-name-2>/        # ...
├── config
├── HEAD
├── hooks/
├── objects/
├── packed-refs
├── refs/
└── worktrees/                          # Git-managed metadata for active worktrees, not the actual worktree branches
```

> **Note**: `branches/` and `worktrees/` do not exist in a freshly cloned bare repo. They are created automatically when you add your first worktree (see [Initial Setup](#initial-setup) below).

## Reading and Editing Files in Worktrees

All source code, skills, and project files live inside worktree checkouts under `branches/`, not at the bare repo root. Use the full worktree path when reading or editing files:

```
# Example: read a skill from the main worktree with the Read tool
/<home-dir>/<username>/<path-to-repo>/<repo-name>.git/branches/main/.github/skills/<skill-name>/SKILL.md

# Example: read a file from a feature branch worktree
/<home-dir>/<username>/<path-to-repo>/<repo-name>.git/branches/feature/<branch-name>/path/to/file
```

Do **not** try to read project files directly from the bare repo root — they only exist inside worktree checkouts.

## Worktree Commands

All `git worktree` commands must be run from the bare repo root.

```bash
REPO=/<home-dir>/<username>/<path-to-repo>/<repo-name>.git
```

### Initial Setup

When working with a freshly cloned bare repo, the `branches/` directory does not exist yet. You must create it and set up a worktree for the default branch first:

```bash
# 1. Create the branches directory
cd $REPO && mkdir -p branches

# 2. Fetch latest refs
cd $REPO && git fetch origin

# 3. Create a worktree for the default branch (usually main, sometimes master)
#    Check which one exists:
cd $REPO && git show-ref --verify refs/heads/main 2>/dev/null && echo "main" || echo "master"

#    Then create the worktree for whichever default branch the project uses:
cd $REPO && git worktree add branches/main main
#    or, if the project uses master:
cd $REPO && git worktree add branches/master master
```

Once you have the default branch worktree, all new feature branches should be created from it.

### Create a new feature branch

Always create new branches from the default branch worktree (not from a bare ref). This ensures your new branch starts from the latest local state of main:

```bash
# First, make sure main is up to date
cd $REPO/branches/main && git pull origin main

# Then create the new branch worktree from the bare repo root
cd $REPO && git worktree add branches/<new-branch-name> -b <new-branch-name> main
```

For feature branches with a path prefix (e.g., `feature/my-feature`), the parent directories are created automatically by `git worktree add`:

```bash
cd $REPO && git worktree add branches/feature/<new-branch-name> -b feature/<new-branch-name> main
```

### Check out an existing remote branch

```bash
cd $REPO && git fetch origin <branch-name> && git worktree add branches/<branch-name> <branch-name>
```

> If the local branch does not exist, Git will auto-create it from the matching remote-tracking branch (`origin/<branch-name>`).

### Remove a worktree

```bash
cd $REPO && git worktree remove branches/<branch-name>
```

### List active worktrees

```bash
cd $REPO && git worktree list
```

### Push from a worktree

Git push works from inside the worktree directory:

```bash
cd $REPO/branches/<branch-name>
git push origin <branch-name>
```

## Working with PRs and Issues (gh CLI)

The `gh` CLI works in a bare repo as long as you run it from inside a worktree directory (e.g., `branches/main/`). Always `cd` into a worktree before running `gh` commands.

```bash
# All gh commands below assume you first cd into a worktree:
cd $REPO/branches/main
```

### View a PR

```bash
gh pr view <number>
```

### List open PRs

```bash
gh pr list
```

### View PR diff or files changed

```bash
gh pr diff <number>
gh pr view <number> --json files --jq '.files[].path'
```

### View all PR comments

PRs have three types of comments. To get the full picture, fetch all three:

```bash
# 1. Conversation comments (top-level discussion on the PR)
gh pr view <number> --json comments --jq '.comments[]'

# 2. Review summaries (review bodies + approval/changes-requested state)
gh pr view <number> --json reviews --jq '.reviews[]'

# 3. Inline review comments (file-level / line-level feedback)
#    Readable one-liner format:
gh api repos/{owner}/{repo}/pulls/<number>/comments \
  --jq '.[] | "[\(.user.login)] \(.path):\(.original_line // .line) — \(.body | split("\n") | first)"'
#    Full JSON (verbose):
gh api repos/{owner}/{repo}/pulls/<number>/comments
```

To fetch everything at once:

```bash
gh pr view <number> --json comments,reviews --jq '.comments[], .reviews[]'
gh api repos/{owner}/{repo}/pulls/<number>/comments --jq '.[] | "[\(.user.login)] \(.path):\(.original_line // .line) — \(.body | split("\n") | first)"'
```

### View unresolved PR comments

The REST API does not expose thread resolution status. Use the **GraphQL API** to filter for unresolved threads only:

```bash
# One-liner summary of unresolved threads (first comment in each thread)
gh api graphql -f query='
{
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: <number>) {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 10) {
            nodes {
              author { login }
              body
              path
              line
            }
          }
        }
      }
    }
  }
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false) | .comments.nodes[0] | "[\(.author.login)] \(.path):\(.line) — \(.body | split("\n") | first)"'
```

For full thread details (all replies in each unresolved thread):

```bash
gh api graphql -f query='
{
  repository(owner: "{owner}", name: "{repo}") {
    pullRequest(number: <number>) {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          comments(first: 10) {
            nodes {
              author { login }
              body
              path
              line
            }
          }
        }
      }
    }
  }
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false) | {id: .id, path: .comments.nodes[0].path, line: .comments.nodes[0].line, comments: [.comments.nodes[] | {user: .author.login, body: .body}]}'
```

### Respond to, add, and resolve PR comments

**Add a top-level conversation comment on a PR:**

```bash
gh pr comment <number> --body "Your comment here"
```

**Reply to an inline review thread (requires the thread's GraphQL node ID):**

```bash
gh api graphql -f query='
mutation {
  addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<thread-id>",
    body: "Your reply here"
  }) {
    comment { id body }
  }
}'
```

**Resolve a review thread:**

```bash
gh api graphql -f query='
mutation {
  resolveReviewThread(input: {
    threadId: "<thread-id>"
  }) {
    thread { isResolved }
  }
}'
```

**Unresolve a review thread:**

```bash
gh api graphql -f query='
mutation {
  unresolveReviewThread(input: {
    threadId: "<thread-id>"
  }) {
    thread { isResolved }
  }
}'
```

**Reply and resolve multiple threads at once** (batch mutations):

```bash
gh api graphql -f query='
mutation {
  reply1: addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<thread-id-1>",
    body: "Done in commit abc123."
  }) { comment { id } }
  resolve1: resolveReviewThread(input: { threadId: "<thread-id-1>" }) { thread { isResolved } }

  reply2: addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "<thread-id-2>",
    body: "Fixed."
  }) { comment { id } }
  resolve2: resolveReviewThread(input: { threadId: "<thread-id-2>" }) { thread { isResolved } }
}'
```

To get thread IDs, use the [View unresolved PR comments](#view-unresolved-pr-comments) query and look for the `id` field on each `reviewThread` node.

### Change PR draft status

```bash
# Mark a PR as ready (remove draft)
gh pr ready <number>

# Convert a PR back to draft
gh pr ready <number> --undo
```

### View an issue

```bash
gh issue view <number>
```

### List open issues

```bash
gh issue list
```

### Create a PR for the current worktree branch

```bash
cd $REPO/branches/<branch-name>
gh pr create --title "title" --body "description"
```

### Key notes

- Always run `gh` from inside a worktree directory (e.g., `branches/main/`), not from the bare repo root. The bare root has no working tree so `gh` may fail to detect the repo.
- `gh pr view` and `gh issue view` are the quickest way to get PR/issue details — no need to open a browser.
- For richer JSON output, use `--json` with field names: `gh pr view <number> --json title,body,state,reviews,comments`.

## Important Rules

- **Always use standard Unix paths** — e.g., `/Users/<username>/...` on macOS or `/home/<username>/...` on Linux. All tools (Bash, Read, Edit, Glob) use the same path format on macOS and Linux.
- Never run commands that assume a default working tree (e.g., `git checkout`, `git switch`). Always use `git worktree` to manage branches.
- All branch working directories live under `branches/`. Do not create worktrees elsewhere.
- If `branches/` does not exist yet, create it with `mkdir -p branches` before adding any worktrees.
- Always ensure a worktree for the default branch (main or master) exists before creating new feature branches. New branches should be created from the default branch.
- When the user says "switch to branch X", add it as a worktree under `branches/X` (or confirm it already exists there).
