---
description: "Git recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-git with file paths and context included"
when-to-use: "Receives prepared arguments from task-git with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Version Control Recipes

**Tools:** git, gh, glab, delta, difftastic, tig, lazygit, gitui, git-lfs, git-crypt, git-extras, pre-commit, gitleaks

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Command |
|------|---------|
| Status | `git status` |
| Add files | `git add file.txt` or `git add .` |
| Commit | `git commit -m "message"` |
| Push | `git push` |
| Pull | `git pull` |
| Create branch | `git checkout -b branch-name` |
| Switch branch | `git checkout branch-name` |
| Merge | `git merge branch-name` |
| View log | `git log --oneline` |
| Create PR | `gh pr create` |
| Stash | `git stash` |
| Apply stash | `git stash pop` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| View diff (terminal) | delta | Syntax-highlighted, side-by-side |
| Structural diff | difftastic | Language-aware, ignores formatting |
| Browse history (TUI) | tig | Interactive git log viewer |
| Full git TUI | lazygit / gitui | Stage, commit, branch in TUI |
| GitHub operations | gh | PRs, issues, releases, actions |
| GitLab operations | glab | MRs, issues, pipelines |
| Track large files | git-lfs | Avoids repo bloat from binaries |
| Pre-commit checks | pre-commit | Automated hooks for code quality |
| Find leaked secrets | gitleaks | Scan history for credentials |
| Encrypt repo files | git-crypt | Transparent encryption |

---

## Basic Operations

### Status and Info
```bash
git status                            # Working tree status
git status -s                         # Short format
git status -b                         # Show branch info
git branch                            # List local branches
git branch -a                         # List all branches (including remote)
git branch -v                         # With last commit
git remote -v                         # Show remotes
```

### Staging
```bash
git add file.txt                      # Stage specific file
git add .                             # Stage all changes
git add -p                            # Interactive staging (by hunk)
git add -A                            # Stage all (including deletions)
git reset HEAD file.txt               # Unstage file
git restore --staged file.txt         # Unstage (newer syntax)
```

### Committing
```bash
git commit -m "message"               # Commit with message
git commit -am "message"              # Add tracked + commit
git commit --amend                    # Amend last commit
git commit --amend -m "new message"   # Amend message only
git commit --amend --no-edit          # Amend without changing message
git commit --allow-empty -m "msg"     # Empty commit (for triggers)
```

### Push/Pull
```bash
git push                              # Push to upstream
git push origin branch-name           # Push specific branch
git push -u origin branch-name        # Set upstream and push
git push --force-with-lease           # Safe force push
git pull                              # Fetch + merge
git pull --rebase                     # Fetch + rebase
git fetch                             # Fetch only (no merge)
git fetch --all                       # Fetch all remotes
```

---

## Branches

### Create and Switch
```bash
git checkout -b feature/new           # Create and switch
git switch -c feature/new             # Create and switch (newer)
git checkout branch-name              # Switch to existing
git switch branch-name                # Switch (newer syntax)
git branch feature/new                # Create without switching
git branch feature/new base-branch    # Create from specific branch
```

### Rename and Delete
```bash
git branch -m old-name new-name       # Rename branch
git branch -d branch-name             # Delete (safe - only if merged)
git branch -D branch-name             # Force delete
git push origin --delete branch-name  # Delete remote branch
git push origin :branch-name          # Delete remote (alternative)
```

### Track Remote
```bash
git checkout --track origin/branch    # Create local tracking branch
git branch -u origin/branch           # Set upstream for current branch
git branch --set-upstream-to=origin/branch
```

### Clean Up
```bash
# Delete merged branches (except main/master/develop)
git branch --merged | grep -vE "main|master|develop" | xargs git branch -d

# Prune deleted remote branches
git fetch --prune
git remote prune origin

# List branches merged into main
git branch --merged main
```

---

## History and Logs

### View Log
```bash
git log                               # Full log
git log --oneline                     # Compact
git log --oneline -20                 # Last 20
git log --oneline --graph             # With graph
git log --oneline --graph --all       # All branches
git log --stat                        # With file stats
git log -p                            # With diffs
git log --author="name"               # By author
git log --since="2024-01-01"          # Since date
git log --until="2024-01-31"          # Until date
git log --grep="keyword"              # Search messages
git log -S "code"                     # Search for code changes
git log -- path/to/file               # File history
```

### Pretty Formats
```bash
git log --oneline --decorate          # With refs
git log --format="%h %s (%an, %ar)"   # Custom format
git log --pretty=format:"%h %ad | %s%d [%an]" --date=short
```

### Reflog
```bash
git reflog                            # All HEAD movements
git reflog show branch-name           # Branch-specific
```

---

## Diffs

### Working Tree
```bash
git diff                              # Unstaged changes
git diff --staged                     # Staged changes
git diff --cached                     # Same as --staged
git diff HEAD                         # All changes vs last commit
```

### Between Commits/Branches
```bash
git diff commit1 commit2              # Between commits
git diff main feature                 # Between branches
git diff main...feature               # Changes on feature since diverging
git diff HEAD~3                       # Last 3 commits
git diff @{yesterday}                 # Since yesterday
```

### Options
```bash
git diff --stat                       # Summary only
git diff --name-only                  # File names only
git diff --name-status                # Names with status (M/A/D)
git diff -w                           # Ignore whitespace
git diff --word-diff                  # Word-level diff
```

### With Better Tools
```bash
git diff | delta                      # Syntax highlighted
git diff --color-words                # Word-level coloring
```

---

## Merging

### Basic Merge
```bash
git merge feature-branch              # Merge into current
git merge --no-ff feature-branch      # No fast-forward (preserves history)
git merge --squash feature-branch     # Squash commits
git merge --abort                     # Abort in-progress merge
```

### Conflict Resolution
```bash
# After conflicts:
git status                            # See conflicted files
# Edit files to resolve
git add resolved-file.txt             # Mark as resolved
git commit                            # Complete merge

# Or abort
git merge --abort
```

### Merge Strategies
```bash
git merge -X theirs feature           # Prefer their changes
git merge -X ours feature             # Prefer our changes
```

---

## Rebasing

### Basic Rebase
```bash
git rebase main                       # Rebase onto main
git rebase --onto main feature~5 feature  # Rebase last 5 commits
git rebase --abort                    # Abort rebase
git rebase --continue                 # Continue after resolving
git rebase --skip                     # Skip current commit
```

### Interactive Rebase
```bash
git rebase -i HEAD~5                  # Last 5 commits
git rebase -i main                    # Since diverging from main
```

Interactive commands:
- `pick` - keep commit
- `reword` - change message
- `edit` - stop to amend
- `squash` - combine with previous
- `fixup` - combine, discard message
- `drop` - remove commit

### Autosquash
```bash
git commit --fixup=abc123             # Create fixup commit
git rebase -i --autosquash HEAD~5     # Auto-arrange fixups
```

---

## Stashing

### Basic Stash
```bash
git stash                             # Stash changes
git stash push -m "description"       # With message
git stash list                        # List stashes
git stash pop                         # Apply and remove
git stash apply                       # Apply without removing
git stash drop                        # Remove top stash
git stash drop stash@{2}              # Remove specific
git stash clear                       # Remove all stashes
```

### Advanced Stash
```bash
git stash -u                          # Include untracked
git stash -a                          # Include ignored
git stash -p                          # Interactive stash
git stash show                        # Show stash diff
git stash show -p stash@{0}           # Full diff
git stash branch new-branch           # Create branch from stash
```

---

## Undoing Changes

### Unstage
```bash
git restore --staged file.txt         # Unstage file
git reset HEAD file.txt               # Unstage (older syntax)
```

### Discard Working Changes
```bash
git restore file.txt                  # Discard changes
git checkout -- file.txt              # Discard (older syntax)
git restore .                         # Discard all
git checkout -- .                     # Discard all (older)
```

### Undo Commits
```bash
git reset --soft HEAD~1               # Undo commit, keep staged
git reset HEAD~1                      # Undo commit, keep unstaged
git reset --mixed HEAD~1              # Same as above
git reset --hard HEAD~1               # Undo commit, discard changes (DANGEROUS)
```

### Revert (safe, creates new commit)
```bash
git revert abc123                     # Revert specific commit
git revert HEAD                       # Revert last commit
git revert HEAD~3..HEAD               # Revert last 3 commits
git revert --no-commit abc123         # Revert without committing
```

### Recover
```bash
git reflog                            # Find lost commits
git checkout abc123                   # Check out lost commit
git branch recovery abc123            # Create branch at lost commit
```

---

## Cherry-Pick

```bash
git cherry-pick abc123                # Apply specific commit
git cherry-pick abc123 def456         # Multiple commits
git cherry-pick abc123..def456        # Range of commits
git cherry-pick --no-commit abc123    # Apply without committing
git cherry-pick --abort               # Abort in progress
git cherry-pick --continue            # Continue after resolving
```

---

## Bisect

```bash
git bisect start                      # Start bisect
git bisect bad                        # Mark current as bad
git bisect good abc123                # Mark known good commit
# Git checks out middle commit
git bisect good                       # Mark as good
git bisect bad                        # Mark as bad
# Repeat until found
git bisect reset                      # End bisect

# Automated
git bisect run ./test-script.sh       # Auto-run test
```

---

## Tags

```bash
git tag v1.0.0                        # Lightweight tag
git tag -a v1.0.0 -m "Version 1.0.0"  # Annotated tag
git tag -a v1.0.0 abc123              # Tag specific commit
git tag -l "v1.*"                     # List matching tags
git push origin v1.0.0                # Push specific tag
git push origin --tags                # Push all tags
git tag -d v1.0.0                     # Delete local tag
git push origin :refs/tags/v1.0.0     # Delete remote tag
```

---

## GitHub CLI (gh)

### Pull Requests
```bash
gh pr create                          # Interactive PR creation
gh pr create --title "Title" --body "Body"
gh pr create --fill                   # Auto-fill from commits
gh pr list                            # List PRs
gh pr status                          # PR status for current branch
gh pr view 123                        # View PR details
gh pr checkout 123                    # Checkout PR branch
gh pr merge 123                       # Merge PR
gh pr merge 123 --squash              # Squash merge
gh pr close 123                       # Close PR
gh pr review 123 --approve            # Approve PR
gh pr review 123 --request-changes    # Request changes
```

### Issues
```bash
gh issue create                       # Create issue
gh issue create --title "Bug" --body "Description"
gh issue list                         # List issues
gh issue view 123                     # View issue
gh issue close 123                    # Close issue
gh issue reopen 123                   # Reopen issue
gh issue comment 123 --body "Comment" # Add comment
```

### Repository
```bash
gh repo clone owner/repo              # Clone
gh repo create                        # Create new repo
gh repo fork                          # Fork current repo
gh repo view                          # View repo info
gh repo view --web                    # Open in browser
```

### Workflows
```bash
gh run list                           # List workflow runs
gh run view                           # View latest run
gh run watch                          # Watch run progress
gh workflow list                      # List workflows
gh workflow run workflow.yml          # Trigger workflow
```

### Releases
```bash
gh release create v1.0.0              # Create release
gh release create v1.0.0 --notes "Notes"
gh release list                       # List releases
gh release download v1.0.0            # Download release assets
```

---

## Configuration

### User Config
```bash
git config --global user.name "Name"
git config --global user.email "email@example.com"
git config --global core.editor "code --wait"
git config --global init.defaultBranch main
```

### Aliases
```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --decorate"
```

### View Config
```bash
git config --list                     # All config
git config --global --list            # Global only
git config user.name                  # Specific value
```

---

## Common Workflows

### Feature Branch
```bash
git checkout main
git pull
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "Add feature"
git push -u origin feature/new-feature
gh pr create
```

### Sync Fork
```bash
git remote add upstream https://github.com/original/repo.git
git fetch upstream
git checkout main
git merge upstream/main
git push
```

### Squash Commits Before PR
```bash
git rebase -i main                    # Interactive rebase
# Change 'pick' to 'squash' for commits to combine
# Edit commit message
git push --force-with-lease
```

### Undo Published Commit (Safe)
```bash
git revert abc123
git push
```

---

## Enhanced Diffing

### delta
```bash
# Configure delta as default pager (one-time setup)
git config --global core.pager delta
git config --global interactive.diffFilter "delta --color-only"
git config --global delta.navigate true
git config --global delta.line-numbers true
git config --global delta.side-by-side true

# Side-by-side diff with delta
git diff | delta --side-by-side

# Delta with syntax highlighting theme
git diff | delta --syntax-theme "Dracula"

# Delta with line numbers
git log -p | delta --line-numbers
```

### difftastic (structural diff)
```bash
# Use difftastic for git diff
GIT_EXTERNAL_DIFF=difft git diff

# Configure as default difftool
git config --global diff.external difft

# Compare two files structurally
difft file1.py file2.py

# Diff with color output mode
difft --color=always file1.js file2.js | less -R
```

---

## Interactive Git Tools

### tig
```bash
# Browse log (interactive)
tig

# Browse specific branch
tig branch-name

# Interactive blame
tig blame file.txt

# Browse stash
tig stash

# Show status view
tig status
```

### lazygit / gitui
```bash
# Launch lazygit (terminal UI for git)
lazygit

# Launch gitui (fast terminal UI)
gitui

# lazygit in specific repo
lazygit -p /path/to/repo
```

---

## Git LFS (Large File Storage)

```bash
# Install LFS hooks in repo
git lfs install

# Track file patterns
git lfs track "*.psd"
git lfs track "*.zip"
git lfs track "models/**"

# List tracked patterns
cat .gitattributes

# List LFS-managed files
git lfs ls-files

# Pull LFS objects
git lfs pull

# Migrate existing files to LFS
git lfs migrate import --include="*.psd" --everything

# Check LFS storage usage
git lfs env
```

---

## git-crypt (Transparent Encryption)

```bash
# Initialize git-crypt in repo
git-crypt init

# Add GPG user who can decrypt
git-crypt add-gpg-user USER_ID

# Specify files to encrypt (.gitattributes)
# secrets/** filter=git-crypt diff=git-crypt
echo "secrets/** filter=git-crypt diff=git-crypt" >> .gitattributes

# Check encryption status
git-crypt status

# Lock repo (encrypt files on disk)
git-crypt lock

# Unlock repo (decrypt files on disk)
git-crypt unlock
```

---

## Pre-commit Hooks & Security Scanning

### pre-commit
```bash
# Install pre-commit framework
pip install pre-commit

# Install hooks from .pre-commit-config.yaml
pre-commit install

# Run all hooks on all files
pre-commit run --all-files

# Run specific hook
pre-commit run black --all-files

# Update hook versions
pre-commit autoupdate

# Skip hooks for one commit (not recommended)
git commit --no-verify -m "message"

# Sample .pre-commit-config.yaml
# repos:
#   - repo: https://github.com/pre-commit/pre-commit-hooks
#     rev: v4.5.0
#     hooks: [{id: trailing-whitespace}, {id: end-of-file-fixer}]
#   - repo: https://github.com/astral-sh/ruff-pre-commit
#     rev: v0.4.0
#     hooks: [{id: ruff, args: [--fix]}, {id: ruff-format}]
```

### gitleaks (Secret Detection)
```bash
# Scan repo for secrets
gitleaks detect

# Scan with verbose output
gitleaks detect -v

# Scan specific commit range
gitleaks detect --log-opts="HEAD~10..HEAD"

# Scan staged changes (pre-commit)
gitleaks protect --staged

# Generate report
gitleaks detect --report-format json --report-path report.json
```

---

## git-extras

```bash
# Repository summary (commits, authors, age)
git summary

# Show effort stats per file (commits and active days)
git effort

# Generate changelog
git changelog

# Delete all merged branches
git delete-merged-branches

# Show what you worked on today
git standup

# Show who contributed to a file
git authors

# Create branch and switch
git create-branch feature/new

# Count commits per author
git count --all
```

### Worktrees

```bash
# Create a worktree for a branch (work on multiple branches simultaneously)
git worktree add ../feature-branch feature     # Checkout feature branch in sibling dir
git worktree add ../hotfix -b hotfix/urgent    # Create new branch in worktree
git worktree list                              # Show all worktrees
git worktree remove ../feature-branch          # Remove worktree
git worktree prune                             # Clean up stale worktree references
```

### Submodules

```bash
# Add submodule
git submodule add https://github.com/user/lib.git libs/lib
git submodule add -b main https://github.com/user/lib.git libs/lib  # Track specific branch

# Initialize and update
git submodule update --init --recursive        # Clone all submodules after fresh clone
git clone --recurse-submodules URL             # Clone with submodules in one step

# Update submodules to latest
git submodule foreach git pull origin main     # Pull latest in each submodule
git submodule update --remote                  # Update to latest remote commit

# Remove submodule
git rm libs/lib && rm -rf .git/modules/libs/lib
```

### Blame and Annotation

```bash
git blame file.txt                             # Show who changed each line
git blame -L 10,20 file.txt                    # Blame specific line range
git blame -w file.txt                          # Ignore whitespace changes
git blame -C file.txt                          # Detect lines moved from other files
git blame -C -C file.txt                       # Also detect copies across commits
git log -p -S "function_name" -- file.txt      # Track when a string was added/removed
```

### Patches

```bash
# Create patches
git format-patch -3 HEAD                       # Create patch files for last 3 commits
git format-patch main..feature                 # Patches for all commits on feature branch
git diff > changes.patch                       # Unstaged changes as patch
git diff --staged > staged.patch               # Staged changes as patch

# Apply patches
git apply changes.patch                        # Apply diff-style patch
git apply --check changes.patch                # Dry run — check if patch applies cleanly
git am *.patch                                 # Apply format-patch patches (preserves commits)
git am --abort                                 # Abort failed patch apply
```

### Clean, Grep, and Archive

```bash
# git clean — remove untracked files
git clean -n                                   # Dry run — show what would be removed
git clean -fd                                  # Remove untracked files and directories
git clean -fdx                                 # Also remove ignored files (node_modules, etc.)
git clean -fdX                                 # Remove ONLY ignored files

# git grep — search tracked files
git grep "pattern"                             # Search all tracked files
git grep "pattern" -- '*.py'                   # Search only Python files
git grep -n "TODO" -- '*.js'                   # With line numbers
git grep -c "import" -- '*.py'                 # Count matches per file
git grep "pattern" HEAD~5                      # Search in older commit

# git archive — export repository
git archive --format=tar HEAD | gzip > export.tar.gz   # Export HEAD as tarball
git archive --format=zip HEAD -o export.zip            # Export as zip
git archive HEAD -- src/ docs/ > partial.tar           # Export specific directories
```

### Sparse Checkout

```bash
# Initialize sparse checkout (only checkout specific directories)
git sparse-checkout init --cone                # Enable cone mode (directory-based)
git sparse-checkout set src/ docs/             # Only checkout src/ and docs/
git sparse-checkout add tests/                 # Add another directory
git sparse-checkout list                       # Show current sparse paths
git sparse-checkout disable                    # Disable, checkout everything
```

### Combined Workflows

```bash
# Changelog generation
git log --oneline --since="2024-01-01" | grep -iE "fix|feat" | awk '{$1=""; print "-"$0}'

# Find large files in history
git rev-list --objects --all | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | awk '/^blob/{print $3,$4}' | sort -rn | head -20

# PR data extraction (GitHub CLI)
gh pr list --json title,author,createdAt --limit 50 | jq -r '.[] | "\(.author.login): \(.title)"'

# Authors by contribution to a file
git log --format='%aN' -- path/to/file | sort | uniq -c | sort -rn

# Most frequently changed files
git log --pretty=format: --name-only | sort | uniq -c | sort -rn | head -20

# Branch cleanup — delete merged local branches
git branch --merged main | grep -v 'main\|master' | xargs git branch -d
```

---

## Installation

```bash
# Git is usually pre-installed

# GitHub CLI
sudo apt install gh
gh auth login

# Better diff
sudo apt install git-delta

# Interactive tools
sudo apt install tig
```
