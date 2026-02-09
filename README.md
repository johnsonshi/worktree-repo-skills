# Windows Bare Repo Worktree Skill for Claude Code

A [Claude Code skill](https://github.com/anthropics/claude-code) that provides comprehensive guidance for working with bare Git repositories using worktrees on Windows. This skill helps AI assistants navigate the unique challenges of bare repos on Windows, including Git Bash path conventions, worktree management, and GitHub PR workflows.

## What is this skill for?

Bare Git repositories with worktrees offer a powerful workflow for managing multiple branches simultaneously, but they come with unique challenges—especially on Windows where Claude Code's Bash tool runs under Git Bash (requiring Unix-style paths) while other tools expect Windows paths. This skill teaches Claude Code to:

- **Handle dual path conventions**: Unix paths (`/c/Users/...`) for Git Bash, Windows paths (`C:\Users\...`) for file operations
- **Manage worktrees**: Create, remove, list, and work with multiple branch checkouts
- **Navigate bare repo structure**: Understand where worktrees, hooks, and skills live
- **Work with GitHub**: View, create, and manage PRs and issues using the `gh` CLI
- **Handle PR comments**: Fetch, reply to, and resolve review comments using GraphQL

## Why use bare repos with worktrees?

Bare repos + worktrees enable:

1. **Multiple simultaneous checkouts**: Work on several branches at once without stashing or switching
2. **Efficient CI/CD**: Check out specific branches without full clones
3. **Clean separation**: Each branch lives in its own directory under `branches/`
4. **Atomic operations**: Push, test, and manage branches independently

## Installation

### Option 1: Using skills.sh (recommended)

The easiest way to install this skill is using [skills.sh](https://github.com/anthropics/skills):

```bash
npx skills add johnsonshi/windows-bare-repo-worktree-skill --skill windows-bare-repo-worktree
```

This will automatically install the skill to `.claude/skills/windows-bare-repo-worktree/`.

### Option 2: Direct installation

Copy `SKILL.md` to your repository's `.claude/skills/windows-bare-repo-worktree/` directory:

```bash
mkdir -p .claude/skills/windows-bare-repo-worktree
curl -o .claude/skills/windows-bare-repo-worktree/SKILL.md \
  https://raw.githubusercontent.com/johnsonshi/windows-bare-repo-worktree-skill/main/skills/windows-bare-repo-worktree/SKILL.md
```

### Option 3: Clone this repository

```bash
git clone https://github.com/johnsonshi/windows-bare-repo-worktree-skill.git
mkdir -p .claude/skills/windows-bare-repo-worktree
cp windows-bare-repo-worktree-skill/skills/windows-bare-repo-worktree/SKILL.md .claude/skills/windows-bare-repo-worktree/
```

### Option 4: Add as a Git submodule

```bash
git submodule add https://github.com/johnsonshi/windows-bare-repo-worktree-skill.git .claude/skills/windows-bare-repo-worktree
# Note: With this approach, the skill will be at .claude/skills/windows-bare-repo-worktree/skills/windows-bare-repo-worktree/SKILL.md
# You may want to create a symlink or adjust your setup accordingly
```

## Prerequisites

- **Windows** (not WSL)
- **Git Bash** (comes with Git for Windows)
- **Claude Code** with Bash tool support
- **GitHub CLI** (`gh`) for PR/issue management (optional but recommended)

## Setting up a bare repo with worktrees

If you don't already have a bare repo, here's how to set one up:

```bash
# Clone as bare repo
git clone --bare https://github.com/your-org/your-repo.git your-repo.git
cd your-repo.git

# Create branches directory for worktrees
mkdir branches

# Check out main branch
git worktree add branches/main main

# Check out other branches as needed
git worktree add branches/feature-x feature-x
```

Your structure will look like:

```
your-repo.git/
├── branches/
│   ├── main/           # Full working tree for main
│   └── feature-x/      # Full working tree for feature-x
├── config
├── HEAD
├── hooks/
├── objects/
├── refs/
└── worktrees/
```

## What does this skill teach Claude Code?

### 1. Path conventions

Claude Code's Bash tool runs under Git Bash, which requires Unix-style paths:

```bash
# ✅ Correct (Git Bash / Unix)
cd /c/Users/username/repos/my-repo.git

# ❌ Wrong (Windows)
cd C:\Users\username\repos\my-repo.git
```

But Read/Edit/Glob tools expect Windows paths:

```bash
# ✅ Correct (Read/Edit/Glob tools)
Read C:\Users\username\repos\my-repo.git\README.md

# ❌ Wrong (Read/Edit/Glob tools)
Read /c/Users/username/repos/my-repo.git/README.md
```

### 2. Worktree management

**Check out an existing branch:**

```bash
cd /c/Users/username/repos/my-repo.git
git fetch origin feature-branch
git worktree add branches/feature-branch feature-branch
```

**Create a new branch:**

```bash
git worktree add branches/new-feature -b new-feature main
```

**Remove a worktree:**

```bash
git worktree remove branches/old-feature
```

### 3. GitHub PR workflows

**View unresolved PR comments** (requires `gh` CLI):

```bash
cd /c/Users/username/repos/my-repo.git/branches/main
gh api graphql -f query='
{
  repository(owner: "your-org", name: "your-repo") {
    pullRequest(number: 42) {
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
}' --jq '.data.repository.pullRequest.reviewThreads.nodes[] | select(.isResolved == false)'
```

**Reply to and resolve comments** (batch operation):

```bash
gh api graphql -f query='
mutation {
  reply1: addPullRequestReviewThreadReply(input: {
    pullRequestReviewThreadId: "PRRT_abc123",
    body: "Fixed in commit xyz789."
  }) { comment { id } }
  resolve1: resolveReviewThread(input: { threadId: "PRRT_abc123" }) { thread { isResolved } }
}'
```

### 4. Important rules

- **Never use `git checkout` or `git switch`** — always use `git worktree` commands
- **All worktrees live under `branches/`** — don't create them elsewhere
- **Run `gh` commands from inside worktrees** — the bare repo root has no working tree, so `gh` may fail
- **Use Unix paths in Bash, Windows paths everywhere else**

## Customization

To customize this skill for your repository, edit the following placeholders in `SKILL.md`:

- `<username>` → your Windows username
- `<path-to-repo>` → path to your bare repo
- `<repo-name>` → your repository name
- `{owner}` and `{repo}` → GitHub org/username and repo name (for `gh` commands)

Or leave them as-is — Claude Code can infer these from context.

## Example usage in Claude Code

Once installed, Claude Code will automatically use this skill when:

1. **Creating or switching branches:**
   - User: "Check out the feature-x branch"
   - Claude: Creates worktree at `branches/feature-x/`

2. **Viewing PR comments:**
   - User: "Show me unresolved comments on PR #42"
   - Claude: Runs GraphQL query to fetch unresolved threads

3. **Managing worktrees:**
   - User: "List all active branches"
   - Claude: Runs `git worktree list`

4. **Pushing changes:**
   - User: "Push this branch to remote"
   - Claude: `cd`s into the worktree and runs `git push`

## Advanced workflows

### Multi-branch development

```bash
# Work on frontend and backend simultaneously
git worktree add branches/frontend-feature frontend-feature
git worktree add branches/backend-feature backend-feature

# Make changes in both
cd branches/frontend-feature && git add . && git commit -m "Update UI"
cd ../backend-feature && git add . && git commit -m "Update API"

# Push both
cd branches/frontend-feature && git push origin frontend-feature
cd ../backend-feature && git push origin backend-feature
```

### Automated PR review workflows

Claude Code can:
1. Fetch all unresolved PR comments
2. Address each comment with code changes
3. Reply to threads with commit references
4. Resolve all threads in a single batch operation
5. Mark PR as ready for review

See the [skills/windows-bare-repo-worktree/SKILL.md](skills/windows-bare-repo-worktree/SKILL.md) for complete examples.

## Troubleshooting

### "fatal: not a git repository"

You're likely in the bare repo root. `gh` commands must run from inside a worktree:

```bash
cd branches/main  # or any other worktree
gh pr list
```

### "No such file or directory" (Bash)

You're using Windows paths in Git Bash. Use Unix-style paths:

```bash
# ❌ Wrong
cd C:\Users\username\repos\my-repo.git

# ✅ Correct
cd /c/Users/username/repos/my-repo.git
```

### "Cannot read file" (Read tool)

You're using Unix paths with Read/Edit/Glob tools. Use Windows-style paths:

```bash
# ❌ Wrong
Read /c/Users/username/repos/my-repo.git/README.md

# ✅ Correct
Read C:\Users\username\repos\my-repo.git\README.md
```

## Contributing

Contributions welcome! This skill was developed through real-world use managing Azure Container Registry PM workflows. If you have improvements or find issues, please open a PR or issue.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related resources

- [Claude Code documentation](https://github.com/anthropics/claude-code)
- [Git worktree documentation](https://git-scm.com/docs/git-worktree)
- [GitHub CLI documentation](https://cli.github.com/manual/)
- [Git Bash path conventions](https://stackoverflow.com/questions/13701218/windows-path-in-mingw-bash)

## Authors

Originally developed by Johnson Shi for the Azure Container Registry team. Generalized and open-sourced for the wider community.
