# Git & GitHub Cheatsheet

> Complete guide to version control with Git and collaboration with GitHub. From basics to advanced workflows.

## Overview

- Purpose: Quick reference for Git commands, workflows, and GitHub features
- Audience: Beginner to Advanced
- Scope: Local Git operations, branching strategies, remote collaboration, GitHub-specific features (PRs, Actions, etc.). Excludes GitLab/Bitbucket-specific features.
- Updated: 2025-11-09

---

## Git Basics

### 1. Repository Setup

Short explanation:

- What it is: Initialize or clone a Git repository
- Why it matters: First step to start tracking changes
- Mental model: Creating a photo album (init) or copying someone else's album (clone)

```shell
# Create new repository in current directory
git init

# Create new repository in specific directory
git init my-project

# Clone existing repository
git clone https://github.com/user/repo.git

# Clone to specific directory
git clone https://github.com/user/repo.git my-folder

# Clone specific branch
git clone -b develop https://github.com/user/repo.git

# Shallow clone (faster, less history)
git clone --depth 1 https://github.com/user/repo.git
```

Common pitfalls:

- Running `git init` in the wrong directory (creates .git in wrong place)
- Cloning without checking disk space for large repos

---

### 2. Configuration

Short explanation:

- What it is: Set user identity and preferences
- When to use: First-time setup, per-project configuration
- Mental model: Setting your signature on documents

```shell
# Set global username (for all repos)
git config --global user.name "Your Name"

# Set global email
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "code --wait"  # VS Code
git config --global core.editor "vim"          # Vim

# View all config
git config --list

# View specific setting
git config user.name

# Set config for current repo only (no --global)
git config user.email "work@company.com"

# Useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
git config --global alias.lg "log --oneline --graph --all"
```

---

### 3. Basic Workflow

Short explanation:

- What it is: Track changes, stage files, commit snapshots
- Mental model: Take photos (stage), save album (commit)

```shell
# Check repository status
git status

# Short status (compact output)
git status -s

# Add specific file to staging
git add file.txt

# Add all files in current directory
git add .

# Add all changes (including deletions)
git add -A

# Add interactively (choose hunks)
git add -p

# Remove file from staging (keep changes)
git reset file.txt

# Remove file from staging and discard changes
git checkout -- file.txt

# Commit staged changes
git commit -m "Add feature X"

# Commit with detailed message (opens editor)
git commit

# Add and commit in one step (tracked files only)
git commit -am "Fix bug Y"

# Amend last commit (change message or add files)
git add forgotten-file.txt
git commit --amend -m "Updated commit message"

# Amend without changing message
git commit --amend --no-edit
```

---

### 4. Viewing History

Short explanation:

- What it is: Review commit history and changes
- When to use: Understand project evolution, find bugs, review changes

```shell
# View commit history
git log

# Compact one-line format
git log --oneline

# Show last N commits
git log -5

# Graph view with branches
git log --oneline --graph --all

# Show commits by author
git log --author="John"

# Show commits in date range
git log --since="2 weeks ago"
git log --after="2024-01-01" --before="2024-12-31"

# Show commits affecting specific file
git log -- file.txt

# Show commits with diffs
git log -p

# Show file changes in each commit
git log --stat

# Pretty format
git log --pretty=format:"%h - %an, %ar : %s"

# Search commit messages
git log --grep="bug fix"

# Show reflog (all reference updates, including resets)
git reflog
```

---

### 5. Viewing Changes

Short explanation:

- What it is: Compare files between commits, branches, or working directory
- Mental model: Side-by-side comparison

```shell
# Show unstaged changes
git diff

# Show staged changes
git diff --staged
git diff --cached

# Show changes between commits
git diff commit1 commit2

# Show changes in specific file
git diff file.txt

# Show changes between branches
git diff main..feature

# Show what changed in a commit
git show commit-hash

# Show files changed in commit
git show --name-only commit-hash

# Word-level diff (not line-level)
git diff --word-diff
```

---

## Branching & Merging

### 6. Branch Management

Short explanation:

- What it is: Create parallel versions of your code
- Why it matters: Isolate features, experiment safely, collaborate
- Mental model: Parallel universes - changes in one don't affect others

```shell
# List all branches
git branch

# List all branches (including remote)
git branch -a

# Create new branch
git branch feature-login

# Switch to branch
git checkout feature-login

# Create and switch in one command
git checkout -b feature-signup

# Switch branch (modern syntax)
git switch feature-login

# Create and switch (modern syntax)
git switch -c feature-logout

# Rename current branch
git branch -m new-name

# Rename other branch
git branch -m old-name new-name

# Delete branch (safe - prevents deleting unmerged)
git branch -d feature-old

# Force delete branch
git branch -D feature-experimental

# Delete remote branch
git push origin --delete feature-old

# Show branches with last commit
git branch -v

# Show merged branches
git branch --merged

# Show unmerged branches
git branch --no-merged
```

---

### 7. Merging

Short explanation:

- What it is: Combine changes from different branches
- When to use: Integrate feature branches into main, sync branches
- Gotchas: Merge conflicts require manual resolution

```shell
# Merge branch into current branch
git checkout main
git merge feature-login

# Merge with commit message
git merge feature-login -m "Merge login feature"

# Fast-forward merge (if possible)
git merge --ff-only feature-login

# Always create merge commit (no fast-forward)
git merge --no-ff feature-login

# Abort merge (if conflicts)
git merge --abort

# Continue merge after resolving conflicts
git add resolved-file.txt
git commit

# Merge strategies
git merge -s recursive feature-branch    # Default
git merge -s ours feature-branch         # Keep our changes on conflict

# View merge conflicts
git status
git diff --name-only --diff-filter=U  # List conflicted files

# Use tool to resolve conflicts
git mergetool
```

---

### 8. Rebasing

Short explanation:

- What it is: Move or combine commits to create linear history
- Why it matters: Cleaner history, easier to review
- Mental model: Cut and paste commits onto different base

```shell
# Rebase current branch onto main
git checkout feature-branch
git rebase main

# Interactive rebase (edit, squash, reorder commits)
git rebase -i HEAD~3  # Last 3 commits

# Continue rebase after resolving conflicts
git add resolved-file.txt
git rebase --continue

# Skip current commit during rebase
git rebase --skip

# Abort rebase
git rebase --abort

# Rebase onto specific commit
git rebase commit-hash

# Preserve merge commits
git rebase --preserve-merges main
```

Interactive rebase options:

- `pick` - keep commit as-is
- `reword` - change commit message
- `edit` - stop to amend commit
- `squash` - merge with previous commit
- `fixup` - like squash but discard message
- `drop` - remove commit

Common pitfalls:

- Never rebase commits pushed to shared branches (rewrites history)
- Rebase conflicts can be complex with many commits

---

### 9. Stashing

Short explanation:

- What it is: Temporarily save uncommitted changes
- When to use: Switch branches without committing, pull updates cleanly
- Mental model: Put work in drawer, come back later

```shell
# Stash current changes
git stash

# Stash with message
git stash save "WIP: login form"

# Stash including untracked files
git stash -u

# Stash including ignored files
git stash -a

# List all stashes
git stash list

# Show stash contents
git stash show
git stash show -p  # Show full diff

# Apply most recent stash (keep in stash list)
git stash apply

# Apply specific stash
git stash apply stash@{2}

# Apply and remove from stash list
git stash pop

# Remove specific stash
git stash drop stash@{1}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch feature-branch stash@{0}
```

---

## Working with Remotes

### 10. Remote Repositories

Short explanation:

- What it is: Connect to repositories hosted on GitHub, GitLab, etc.
- Mental model: Cloud backup and collaboration hub

```shell
# Show remote repositories
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git

# Add SSH remote
git remote add origin git@github.com:user/repo.git

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Rename remote
git remote rename origin upstream

# Remove remote
git remote remove origin

# Show remote details
git remote show origin

# Fetch remote branches
git fetch origin

# Fetch all remotes
git fetch --all

# Prune deleted remote branches
git fetch --prune
git remote prune origin
```

---

### 11. Push & Pull

Short explanation:

- What it is: Send commits to remote (push) or get commits from remote (pull)
- When to use: Share your work, sync with team changes

```shell
# Push to remote
git push origin main

# Push and set upstream (first push)
git push -u origin feature-branch

# Push all branches
git push --all

# Push tags
git push --tags

# Force push (dangerous - overwrites remote)
git push --force

# Safer force push (rejects if remote has new commits)
git push --force-with-lease

# Delete remote branch
git push origin --delete feature-old

# Pull changes (fetch + merge)
git pull origin main

# Pull with rebase instead of merge
git pull --rebase origin main

# Pull from upstream (for forks)
git pull upstream main

# Fetch without merging
git fetch origin
git merge origin/main
```

---

## Advanced Git

### 12. Cherry-Pick

Short explanation:

- What it is: Copy specific commits from one branch to another
- When to use: Apply hotfix to multiple branches, selectively merge commits
- Mental model: Pick specific cherries from one basket to another

```shell
# Apply commit to current branch
git cherry-pick commit-hash

# Cherry-pick multiple commits
git cherry-pick commit1 commit2 commit3

# Cherry-pick range
git cherry-pick commit1^..commit2

# Cherry-pick without committing
git cherry-pick -n commit-hash

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort

# Sign off cherry-picked commit
git cherry-pick -s commit-hash
```

---

### 13. Reset & Revert

Short explanation:

- What it is: Undo commits (reset = rewrite history, revert = new commit)
- When to use: Fix mistakes, undo changes
- Gotchas: Reset is destructive, revert is safe for shared branches

```shell
# Reset to previous commit (keep changes staged)
git reset --soft HEAD~1

# Reset to previous commit (keep changes unstaged)
git reset HEAD~1
git reset --mixed HEAD~1  # Same as above

# Reset to previous commit (discard all changes)
git reset --hard HEAD~1

# Reset to specific commit
git reset --hard commit-hash

# Undo reset (use reflog)
git reflog
git reset --hard HEAD@{2}

# Revert commit (creates new commit)
git revert commit-hash

# Revert merge commit
git revert -m 1 merge-commit-hash

# Revert without committing
git revert -n commit-hash

# Revert range
git revert commit1..commit2
```

Reset levels:

- `--soft`: Move HEAD, keep staged and working changes
- `--mixed`: Move HEAD, unstage changes, keep working changes
- `--hard`: Move HEAD, discard all changes (DANGEROUS)

---

### 14. Tags

Short explanation:

- What it is: Mark specific commits (usually releases)
- When to use: Version releases, important milestones

```shell
# List tags
git tag

# Create lightweight tag
git tag v1.0.0

# Create annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v0.9.0 commit-hash -m "Beta release"

# Show tag details
git show v1.0.0

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push --tags

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Checkout tag (detached HEAD)
git checkout v1.0.0

# Create branch from tag
git checkout -b hotfix-1.0.1 v1.0.0
```

---

### 15. Reflog & Recovery

Short explanation:

- What it is: Reference log of all HEAD movements
- Why it matters: Recover lost commits, undo mistakes
- Mental model: Time machine for Git history

```shell
# Show reflog
git reflog

# Show reflog for specific branch
git reflog show feature-branch

# Recover lost commit
git reflog
git checkout commit-hash
git checkout -b recovered-branch

# Recover deleted branch
git reflog
git checkout -b recovered-branch HEAD@{2}

# Reset to previous state
git reset --hard HEAD@{1}

# Expire old reflog entries (cleanup)
git reflog expire --expire=30.days --all
```

---

### 16. Git Hooks

Short explanation:

- What it is: Scripts that run automatically on Git events
- When to use: Enforce coding standards, run tests, automate tasks
- Mental model: Event listeners for Git actions

```shell
# Hooks location
# .git/hooks/

# Common hooks (remove .sample to activate)
# pre-commit      - Before commit
# commit-msg      - Validate commit message
# pre-push        - Before push
# post-merge      - After merge
# post-checkout   - After checkout
```

Example pre-commit hook (`.git/hooks/pre-commit`):

```bash
#!/bin/sh
# Run tests before commit
npm test
if [ $? -ne 0 ]; then
  echo "Tests failed. Commit aborted."
  exit 1
fi
```

Example commit-msg hook (validate commit format):

```bash
#!/bin/sh
# Enforce conventional commits
commit_msg=$(cat "$1")
pattern="^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{1,50}"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
  echo "Invalid commit message format."
  echo "Use: type(scope): description"
  exit 1
fi
```

---

## GitHub-Specific Features

### 17. Pull Requests (PRs)

Short explanation:

- What it is: Request to merge changes from one branch to another
- Why it matters: Code review, collaboration, quality control
- Mental model: Proposal for changes with discussion

```shell
# Create PR (via GitHub CLI)
gh pr create --title "Add login feature" --body "Implements user authentication"

# Create PR for current branch
gh pr create

# List PRs
gh pr list

# View PR
gh pr view 123

# Checkout PR locally
gh pr checkout 123

# Review PR
gh pr review 123 --approve
gh pr review 123 --request-changes --body "Please fix X"

# Merge PR
gh pr merge 123

# Merge with squash
gh pr merge 123 --squash

# Merge with rebase
gh pr merge 123 --rebase

# Close PR
gh pr close 123
```

PR workflow (manual):

```shell
# 1. Create feature branch
git checkout -b feature/login

# 2. Make changes and commit
git add .
git commit -m "Add login functionality"

# 3. Push to remote
git push -u origin feature/login

# 4. Create PR on GitHub (web interface or gh CLI)
# 5. Address review comments
# 6. Merge PR (web interface or gh CLI)

# 7. Update local main
git checkout main
git pull origin main

# 8. Delete feature branch
git branch -d feature/login
git push origin --delete feature/login
```

---

### 18. Issues

Short explanation:

- What it is: Track bugs, features, tasks
- When to use: Project planning, bug tracking, collaboration

```shell
# Create issue (GitHub CLI)
gh issue create --title "Fix login bug" --body "Users can't login"

# List issues
gh issue list

# View issue
gh issue view 42

# Close issue
gh issue close 42

# Reopen issue
gh issue reopen 42

# Link PR to issue (in PR description or commit message)
# "Fixes #42" or "Closes #42" - auto-closes issue when PR merges
```

---

### 19. GitHub Actions (CI/CD)

Short explanation:

- What it is: Automate workflows (testing, deployment, etc.)
- Mental model: Robots that run tasks when events happen

Example workflow (`.github/workflows/test.yml`):

```yaml
name: Run Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Run linter
        run: npm run lint
```

```shell
# Trigger workflow manually
gh workflow run test.yml

# List workflows
gh workflow list

# View workflow runs
gh run list

# View specific run
gh run view 12345
```

---

## Common Workflows

### 20. Feature Branch Workflow

```shell
# 1. Update main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/new-feature

# 3. Work on feature
git add .
git commit -m "Add feature X"

# 4. Keep branch updated
git checkout main
git pull origin main
git checkout feature/new-feature
git merge main  # or: git rebase main

# 5. Push and create PR
git push -u origin feature/new-feature
gh pr create

# 6. After PR merged, cleanup
git checkout main
git pull origin main
git branch -d feature/new-feature
```

---

### 21. Hotfix Workflow

```shell
# 1. Create hotfix branch from main
git checkout main
git checkout -b hotfix/critical-bug

# 2. Fix and commit
git add .
git commit -m "Fix critical security bug"

# 3. Merge to main
git checkout main
git merge hotfix/critical-bug
git push origin main

# 4. Merge to develop (if using)
git checkout develop
git merge hotfix/critical-bug
git push origin develop

# 5. Tag release
git tag -a v1.0.1 -m "Hotfix release"
git push --tags

# 6. Delete hotfix branch
git branch -d hotfix/critical-bug
```

---

### 22. Forking Workflow

```shell
# 1. Fork repo on GitHub (web interface)

# 2. Clone your fork
git clone https://github.com/your-username/repo.git
cd repo

# 3. Add upstream remote
git remote add upstream https://github.com/original-owner/repo.git

# 4. Create feature branch
git checkout -b feature/contribution

# 5. Make changes and commit
git add .
git commit -m "Add feature"

# 6. Push to your fork
git push origin feature/contribution

# 7. Create PR on GitHub (your fork -> upstream)

# 8. Sync with upstream
git checkout main
git fetch upstream
git merge upstream/main
git push origin main
```

---

## Commit Conventions

### 23. Conventional Commits

Format: `type(scope): description`

```shell
# Types
feat: New feature
fix: Bug fix
docs: Documentation only
style: Code style (formatting, no logic change)
refactor: Code restructuring
test: Add/update tests
chore: Maintenance (dependencies, config)
perf: Performance improvement
ci: CI/CD changes

# Examples
git commit -m "feat(auth): add login with OAuth"
git commit -m "fix(api): handle null response"
git commit -m "docs: update README installation steps"
git commit -m "refactor(user): simplify validation logic"
git commit -m "chore(deps): upgrade react to v18"

# With breaking change
git commit -m "feat(api)!: change response format

BREAKING CHANGE: Response now returns { data, meta } instead of array"
```

---

## .gitignore Patterns

```txt
# Dependencies
node_modules/
vendor/
*.egg-info/

# Build outputs
dist/
build/
*.js.map
*.d.ts

# Environment
.env
.env.local
.env.*.local

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*

# Testing
coverage/
.nyc_output/

# Temporary
*.tmp
*.temp
.cache/

# Specific files
config/secrets.json
```

Common patterns:

```shell
# Ignore file
file.txt

# Ignore directory
folder/

# Ignore pattern
*.log

# Negate (don't ignore)
!important.log

# Ignore in specific directory
src/*.class

# Ignore in all directories
**/*.class
```

---

## Troubleshooting

### 24. Common Issues

**Merge conflicts:**

```shell
# 1. Identify conflicts
git status

# 2. Open conflicted files, look for:
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> branch-name

# 3. Resolve manually, then:
git add resolved-file.txt
git commit
```

**Undo last commit (keep changes):**

```shell
git reset --soft HEAD~1
```

**Undo changes to file:**

```shell
# Unstaged file
git checkout -- file.txt

# Staged file
git reset file.txt
git checkout -- file.txt
```

**Fix wrong commit message:**

```shell
git commit --amend -m "Correct message"
```

**Recover deleted branch:**

```shell
git reflog
git checkout -b recovered-branch HEAD@{2}
```

**Remove sensitive file from history:**

```shell
# Use BFG Repo-Cleaner or git filter-branch
git filter-branch --tree-filter 'rm -f passwords.txt' HEAD
```

**Large files blocking push:**

```shell
# Use Git LFS
git lfs install
git lfs track "*.psd"
git add .gitattributes
```

**Detached HEAD:**

```shell
# Save work
git checkout -b temp-branch

# Or discard
git checkout main
```

---

## Quick Reference

Essential Git commands:

1. `git status` - Check current state
2. `git add .` - Stage all changes
3. `git commit -m "message"` - Commit changes
4. `git push` - Send to remote
5. `git pull` - Get from remote
6. `git checkout -b branch` - Create and switch branch
7. `git merge branch` - Merge branch
8. `git log --oneline` - View history
9. `git diff` - View changes
10. `git stash` - Save work temporarily

---

## Performance Tips

1. **Shallow clone** for large repos: `git clone --depth 1`
2. **Use .gitignore** - don't track unnecessary files
3. **Garbage collection**: `git gc` (runs automatically)
4. **Prune old branches**: `git fetch --prune`
5. **Use SSH** instead of HTTPS for faster authentication
6. **Git LFS** for large binary files
7. **Sparse checkout** for monorepos
8. **Repack repository**: `git repack -a -d --depth=250 --window=250`

---

## Security & Safety

- **Never commit secrets** - use `.env` and `.gitignore`
- **Use SSH keys** - more secure than passwords
- **Sign commits** with GPG for verification
- **Review .gitignore** before first commit
- **Use branch protection** on main branches (GitHub settings)
- **Require PR reviews** before merging
- **Enable 2FA** on GitHub account
- **Regularly update dependencies** (Dependabot)
- **Scan for secrets** - use tools like git-secrets, gitleaks

---

## Git Aliases

Add to `~/.gitconfig`:

```ini
[alias]
  st = status
  co = checkout
  br = branch
  ci = commit
  cm = commit -m
  ca = commit --amend
  cp = cherry-pick
  df = diff
  dc = diff --cached
  lg = log --oneline --graph --all --decorate
  ls = log --pretty=format:"%C(yellow)%h%Cred%d\\ %Creset%s%Cblue\\ [%cn]" --decorate
  unstage = reset HEAD --
  last = log -1 HEAD
  visual = log --graph --oneline --all
  undo = reset --soft HEAD~1
```

---

## Next Steps

- **Practice daily** - use Git for all projects
- **Learn GitHub Flow** - understand PR-based workflow
- **Explore Git internals** - understand how Git works under the hood
- **Master branching strategies** - GitFlow, GitHub Flow, trunk-based
- **Automate with hooks** - enforce standards, run tests
- **Contribute to open source** - practice forking workflow
- **Learn advanced features** - submodules, worktrees, bisect

---

## Resources

- Official docs: [Git Documentation](https://git-scm.com/doc)
- Interactive learning: [Learn Git Branching](https://learngitbranching.js.org/)
- GitHub guides: [GitHub Skills](https://skills.github.com/)
- Cheat sheet: [GitHub Git Cheat Sheet](https://training.github.com/downloads/github-git-cheat-sheet/)
- GitHub CLI: [gh documentation](https://cli.github.com/manual/)
- Advanced: [Pro Git Book](https://git-scm.com/book/en/v2) (free)
