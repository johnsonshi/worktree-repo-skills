---
name: windows-bare-repo-worktree
description: "Windows-specific guidance for working in bare Git repositories with worktrees. Covers Git Bash path requirements, worktree management, and GitHub PR/issue workflows via gh CLI. Use this skill whenever performing git operations, creating or switching branches, checking out code, navigating the repository structure, viewing or creating PRs, reviewing PR comments/diffs, or working with GitHub issues on Windows. Triggers on any git workflow task including: branch creation, branch checkout, worktree management, committing, pushing, or any operation that assumes a working tree. Also triggers on GitHub tasks including: viewing PRs, listing PRs/issues, creating PRs, reviewing PR diffs or comments, and any gh CLI usage. Also use when the user asks about the repo layout or how to work with branches."
---

# Windows Bare Repo Worktree Operations

## Environment

- **OS**: Windows (not WSL).
- **Bash tool**: The Bash tool in Claude Code runs under **Git Bash**, not PowerShell. You **must** use Unix-style paths with forward slashes (e.g., `/c/Users/...`), not Windows backslash paths (e.g., `C:\Users\...`). Windows-style paths will fail in the Bash tool.
- **Repo type**: Bare clone (`--bare`). There is no default working tree.
- **Repo path (Git Bash)**: `/c/Users/<username>/<path-to-repo>/<repo-name>.git`
- **Repo path (Windows)**: `C:\Users\<username>\<path-to-repo>\<repo-name>.git` (use only for Read/Edit/Glob tools, which accept Windows paths)

## Repo Layout

```
<repo-name>.git/
├── .claude/
│   └── skills/                    # Custom skills live here (plain Markdown)
│       └── <skill-name>/
│           └── SKILL.md
├── branches/                      # All worktree checkouts live here
│   ├── main/                      # worktree for the main branch
│   ├── <branch-name-1>/           # one worktree directory per checked-out branch
│   └── <branch-name-2>/           # ...
├── CLAUDE.md
├── config
├── HEAD
├── hooks/
├── objects/
├── packed-refs
├── refs/
└── worktrees/
```

## Reading and Editing Skills

Skills are plain Markdown files at `.claude/skills/<skill-name>/SKILL.md` in the bare repo root. Just read them directly — do **not** try to extract, unzip, or decompress anything. Use Windows-style paths for the Read/Edit tools:

```
# Example: read a skill with the Read tool
C:\Users\<username>\<path-to-repo>\<repo-name>.git\.claude\skills\<skill-name>\SKILL.md
```

## Worktree Commands

All `git worktree` commands must be run from the bare repo root.

```bash
REPO=/c/Users/<username>/<path-to-repo>/<repo-name>.git
```

### Check out an existing remote branch

```bash
cd $REPO && git fetch origin <branch-name> && git worktree add branches/<branch-name> <branch-name>
```

### Create a new branch from main

```bash
cd $REPO && git worktree add branches/<new-branch-name> -b <new-branch-name> main
```

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
cd /c/Users/<username>/<path-to-repo>/<repo-name>.git/branches/<branch-name>
git push origin <branch-name>
```

## Working with PRs and Issues (gh CLI)

The `gh` CLI works in a bare repo as long as you run it from inside a worktree directory (e.g., `branches/main/`). Always `cd` into a worktree before running `gh` commands.

```bash
# All gh commands below assume you first cd into a worktree:
cd /c/Users/<username>/<path-to-repo>/<repo-name>.git/branches/main
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
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false) | {path: .comments.nodes[0].path, line: .comments.nodes[0].line, comments: [.comments.nodes[] | {user: .author.login, body: .body}]}'
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
cd /c/Users/<username>/<path-to-repo>/<repo-name>.git/branches/<branch-name>
gh pr create --title "title" --body "description"
```

### Key notes

- Always run `gh` from inside a worktree directory (e.g., `branches/main/`), not from the bare repo root. The bare root has no working tree so `gh` may fail to detect the repo.
- `gh pr view` and `gh issue view` are the quickest way to get PR/issue details — no need to open a browser.
- For richer JSON output, use `--json` with field names: `gh pr view <number> --json title,body,state,reviews,comments`.

## Important Rules

- **Always use Git Bash (Unix-style) paths in the Bash tool** — e.g., `/c/Users/<username>/...`, never `C:\Users\<username>\...`. Windows backslash paths will fail in Git Bash.
- **Use Windows paths only for Read/Edit/Glob tools**, which expect Windows-style paths (e.g., `C:\Users\<username>\...`).
- Never run commands that assume a default working tree (e.g., `git checkout`, `git switch`). Always use `git worktree` to manage branches.
- All branch working directories live under `branches/`. Do not create worktrees elsewhere.
- When the user says "switch to branch X", add it as a worktree under `branches/X` (or confirm it already exists there).
- Use `powershell.exe -Command "..."` when PowerShell-specific commands are needed (e.g., `New-Item`, `Remove-Item`), since the Bash tool runs under Git Bash.
