# Worktree Repo Skills for Claude Code

[Claude Code](https://github.com/anthropics/claude-code) skills that teach AI assistants how to work with Git worktree repositories. Each skill is tailored for a specific operating system, handling the unique path conventions and shell environments of that platform.

## Available Skills

| Skill | Directory | OS | Key Details |
| --- | --- | --- | --- |
| **windows-worktree-repo-operations** | [`skills/windows-worktree-repo-operations/`](skills/windows-worktree-repo-operations/) | Windows | Git Bash (Unix paths) for Bash tool, Windows paths for Read/Edit/Glob tools |
| **macos-linux-worktree-repo-operations** | [`skills/macos-linux-worktree-repo-operations/`](skills/macos-linux-worktree-repo-operations/) | macOS / Linux | Standard Unix paths for all tools |

## What do these skills teach?

Bare Git repositories with worktrees offer a powerful workflow for managing multiple branches simultaneously, but they come with unique challenges. These skills teach Claude Code to:

- **Manage worktrees**: Create, remove, list, and work with multiple branch checkouts under `branches/`
- **Navigate bare repo structure**: Understand where worktrees, skills, and project files live
- **Handle OS-specific path conventions**: Unix paths vs Windows paths (Windows skill), or standard Unix paths everywhere (macOS/Linux skill)
- **Work with GitHub**: View, create, and manage PRs and issues using the `gh` CLI
- **Handle PR comments**: Fetch, reply to, and resolve review comments using GraphQL batch mutations

## Why use bare repos with worktrees?

1. **Multiple simultaneous checkouts** — work on several branches at once without stashing or switching
2. **Clean separation** — each branch lives in its own directory under `branches/`
3. **Atomic operations** — push, test, and manage branches independently
4. **No more `git checkout`** — all branch management happens through `git worktree`

## Installation

### Option 1: Using skills.sh (recommended)

Install the skill for your operating system using [skills.sh](https://github.com/anthropics/skills):

**Windows:**

```bash
npx skills add johnsonshi/worktree-repo-skills --skill windows-worktree-repo-operations
```

**macOS / Linux:**

```bash
npx skills add johnsonshi/worktree-repo-skills --skill macos-linux-worktree-repo-operations
```

This will automatically install the skill to `.claude/skills/<skill-name>/`.

### Option 2: Direct installation

Copy the appropriate `SKILL.md` to your repository's `.claude/skills/` directory:

**Windows:**

```bash
mkdir -p .claude/skills/windows-worktree-repo-operations
curl -o .claude/skills/windows-worktree-repo-operations/SKILL.md \
  https://raw.githubusercontent.com/johnsonshi/worktree-repo-skills/master/skills/windows-worktree-repo-operations/SKILL.md
```

**macOS / Linux:**

```bash
mkdir -p .claude/skills/macos-linux-worktree-repo-operations
curl -o .claude/skills/macos-linux-worktree-repo-operations/SKILL.md \
  https://raw.githubusercontent.com/johnsonshi/worktree-repo-skills/master/skills/macos-linux-worktree-repo-operations/SKILL.md
```

### Option 3: Clone this repository

```bash
git clone https://github.com/johnsonshi/worktree-repo-skills.git

# Windows:
mkdir -p .claude/skills/windows-worktree-repo-operations
cp worktree-repo-skills/skills/windows-worktree-repo-operations/SKILL.md \
   .claude/skills/windows-worktree-repo-operations/

# macOS / Linux:
mkdir -p .claude/skills/macos-linux-worktree-repo-operations
cp worktree-repo-skills/skills/macos-linux-worktree-repo-operations/SKILL.md \
   .claude/skills/macos-linux-worktree-repo-operations/
```

## Prerequisites

- **Git** (Git for Windows with Git Bash on Windows; standard git on macOS/Linux)
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

```text
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

## What do the skills cover?

Both skills share the same core capabilities, with OS-specific adaptations:

### Worktree management

- Initial setup of a bare repo with `branches/` directory
- Creating new feature branches from the default branch
- Checking out existing remote branches as worktrees
- Removing, listing, and pushing from worktrees

### GitHub PR workflows

- Viewing PRs, diffs, and files changed
- Fetching all three types of PR comments (conversation, review summaries, inline)
- Querying unresolved review threads via GraphQL
- Replying to and resolving review threads (including batch operations)
- Managing PR draft status
- Creating PRs and working with issues

### OS-specific differences

| Feature | Windows Skill | macOS/Linux Skill |
| --- | --- | --- |
| Bash tool paths | Unix-style (`/c/Users/...`) via Git Bash | Standard Unix (`/Users/...` or `/home/...`) |
| Read/Edit/Glob paths | Windows-style (`C:\Users\...`) | Standard Unix (same as Bash) |
| Shell | Git Bash | zsh (macOS) / bash (Linux) |
| PowerShell fallback | `powershell.exe -Command "..."` | N/A |

## Example usage in Claude Code

Once installed, Claude Code will automatically use the skill when:

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

## Customization

To customize a skill for your repository, edit the following placeholders in `SKILL.md`:

- `<username>` → your system username
- `<path-to-repo>` → path to your bare repo
- `<repo-name>` → your repository name
- `{owner}` and `{repo}` → GitHub org/username and repo name (for `gh` commands)

Or leave them as-is — Claude Code can infer these from context.

## Contributing

Contributions welcome! If you have improvements or find issues, please open a PR or issue.

## License

MIT License - see [LICENSE](LICENSE) for details.

## Related resources

- [Claude Code documentation](https://github.com/anthropics/claude-code)
- [Git worktree documentation](https://git-scm.com/docs/git-worktree)
- [GitHub CLI documentation](https://cli.github.com/manual/)
- [skills.sh — Claude Code skill manager](https://github.com/anthropics/skills)

## Authors

Originally developed by Johnson Shi for the Azure Container Registry team. Generalized and open-sourced for the wider community.
