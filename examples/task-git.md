---
description: "Version control - messed up my commit, accidentally deleted changes, can't push to remote, merge went wrong, lost my work, Git operations, branching, merging, and pull requests (git, gh, glab)"
when_to_use: "Use when: messed up a commit and need to fix it, accidentally deleted changes or lost work, can't push to remote, merge went wrong or has conflicts, need to undo or revert changes, committing, branching, merging, viewing history, creating pull requests, managing GitHub issues, stashing changes. Triggers: messed up commit, accidentally deleted, can't push, merge went wrong, lost my work, undo commit, fix last commit, wrong branch, git, commit, branch, merge, pull request, gh, github, gitlab, diff, history, rebase, stash, cherry-pick, bisect, revert changes, view changes, git blame, merge conflict, cherry pick, worktree, submodule, patch, squash, sparse checkout, large files in history, git log, amend, interactive rebase, git hooks, tag."
when-to-use: "Use when: messed up a commit and need to fix it, accidentally deleted changes or lost work, can't push to remote, merge went wrong or has conflicts, need to undo or revert changes, committing, branching, merging, viewing history, creating pull requests, managing GitHub issues, stashing changes. Triggers: messed up commit, accidentally deleted, can't push, merge went wrong, lost my work, undo commit, fix last commit, wrong branch, git, commit, branch, merge, pull request, gh, github, gitlab, diff, history, rebase, stash, cherry-pick, bisect, revert changes, view changes, git blame, merge conflict, cherry pick, worktree, submodule, patch, squash, sparse checkout, large files in history, git log, amend, interactive rebase, git hooks, tag."
argument-hint: "[describe the git operation]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Git Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the git recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What git operation is needed?
- Commit, branch, merge, rebase, stash, view history, create PR, resolve conflict

### 2. Scope
What is involved?
- Specific files or all changes
- Specific branch or current branch
- Specific commits (by hash or reference)
- Local only or remote involved

### 3. Context from Conversation
- Branch names discussed
- Commit messages or hashes mentioned
- Files modified earlier
- Errors encountered

### 4. Safety Considerations
- Is this a destructive operation? (force push, reset, rebase published)
- Does it need user confirmation?

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Basic version control | git | Core operations (commit, branch, merge) |
| GitHub PRs/issues | gh | GitHub CLI for remote ops |
| GitLab operations | glab | GitLab CLI equivalent |
| Better diffs | delta / difftastic | Syntax-highlighted diff output |
| Interactive history | tig / lazygit | Terminal UI for git log |
| Large file tracking | git-lfs | Track binaries without bloating repo |
| Secret scanning | gitleaks | Find leaked credentials |
| Pre-commit hooks | pre-commit | Automated code checks |
| Encrypt repo files | git-crypt | Transparent encryption in repo |

---

## Argument Format

Construct a structured argument string:

```
<operation> - scope: <what>, options: <flags>
```

**Include only relevant components.**

### Examples

**Commit:**
```
commit staged changes - message: "Add user authentication module"
```

**Branch operations:**
```
create branch feature/login from main - and switch to it
```

**View history:**
```
show commit history - last 20 commits, with graph, oneline format
```

**Diff:**
```
show diff between main and feature/api - files only
```

**Stash:**
```
stash current changes - with message "WIP: debugging auth"
```

**Cherry-pick:**
```
cherry-pick commit abc123 - from feature branch to current
```

**Rebase:**
```
rebase current branch onto main - interactive, last 5 commits
```

**Pull request:**
```
create pull request - from feature/login to main, title: "Add login functionality"
```

**Undo:**
```
undo last commit - keep changes staged
```

**Clean up:**
```
delete merged branches - local only, exclude main and develop
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-git-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Operation is destructive (reset --hard, force push)
- Rebase involves published commits
- Commit message not provided
- Target branch unclear

**Use Read tool first if:**
- Need to check current branch status
- Need to verify file changes before commit
