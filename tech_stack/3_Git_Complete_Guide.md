# Git Complete Guide

## Table of Contents
1. [Introduction](#introduction)
2. [Git Basics](#git-basics)
3. [Local Repository Operations](#local-repository-operations)
4. [Branching & Merging](#branching--merging)
5. [Remote Repositories](#remote-repositories)
6. [Undoing Changes](#undoing-changes)
7. [Advanced Operations](#advanced-operations)
8. [Git Workflows](#git-workflows)
9. [Best Practices](#best-practices)
10. [Common Scenarios](#common-scenarios)

---

## Introduction

### What is Git?

Git is a distributed version control system (VCS) that tracks code changes and enables collaborative development. It is the most popular VCS for software projects.

### Key Characteristics

- **Distributed** - Complete project history in local repository
- **Version Control** - Track changes over time with full history
- **Branching** - Create isolated development lines
- **Merging** - Combine changes from different branches
- **Offline Work** - Work locally without internet connection
- **Fast** - Efficient handling of large projects
- **Flexible** - Supports various workflows and team structures

### Why Use Git?

1. **Collaboration** - Multiple developers work on same project
2. **History Tracking** - Complete record of all changes
3. **Branching** - Parallel development without conflicts
4. **Backup** - Distributed copies prevent data loss
5. **Code Review** - Pull requests enable peer review
6. **Integration** - Works with CI/CD pipelines
7. **Rollback** - Easy revert to previous versions

### Git vs Other VCS

| Feature | Git | Subversion | Mercurial |
|---|---|---|---|
| Distributed | ✅ | ❌ | ✅ |
| Speed | ✅ Fast | ❌ Slow | ✅ Fast |
| Branching | ✅ Lightweight | ❌ Heavy | ✅ Good |
| Learning Curve | Medium | Low | Medium |
| Community | ✅ Largest | Small | Small |

---

## Git Basics

### Installation

**Windows:**
```bash
# Download from https://git-scm.com/download/win
# Or use package manager
choco install git
```

**macOS:**
```bash
# Using Homebrew
brew install git

# Or Xcode command line tools
xcode-select --install
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install git
```

**Linux (Fedora):**
```bash
sudo yum install git
```

### Initial Configuration

Set up your Git identity:

```bash
# Configure globally (applies to all repositories)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure for specific repository (overrides global)
git config user.name "Your Name"
git config user.email "your.email@example.com"

# View configuration
git config --list
git config user.name
```

### Core Concepts

#### Repository
A directory containing your project files and Git metadata (.git folder).

```
my-project/
├── .git/           # Git metadata (hidden)
├── src/
├── README.md
└── .gitignore
```

#### Working Directory
The actual files on your disk that you're editing.

#### Staging Area (Index)
Intermediate area where you prepare changes before committing.

#### Commit
Snapshot of your project at a specific point in time.

#### Branch
Independent line of development.

#### Remote
External repository (like GitHub) for collaboration.

### Git Workflow States

```
Working Directory → Staging Area → Repository (Commits)
    ↓ git add           ↓ git commit      ↓ git push
  Modified Files    Staged Files      Committed Files
```

---

## Local Repository Operations

### Creating and Initializing

**Initialize a new repository:**
```bash
# Create new repository in current directory
git init

# Create new repository in specific directory
git init my-project
cd my-project
```

**Clone existing repository:**
```bash
# Clone into new directory
git clone https://github.com/user/repo.git

# Clone into current directory
git clone https://github.com/user/repo.git .

# Clone specific branch
git clone --branch develop https://github.com/user/repo.git

# Clone with depth (shallow clone)
git clone --depth 1 https://github.com/user/repo.git
```

### Checking Status

**View repository status:**
```bash
# Show status of working directory and staging area
git status

# Short status format
git status -s
# Output: M = modified, A = added, D = deleted, ?? = untracked
```

**View what changed:**
```bash
# Show changes in working directory (unstaged)
git diff

# Show changes in staging area
git diff --staged
# or
git diff --cached

# Show differences between commits
git diff commit1 commit2

# Show differences for specific file
git diff path/to/file.js
```

### Staging Changes

**Add files to staging area:**
```bash
# Add specific file
git add path/to/file.js

# Add all changes in current directory
git add .

# Add all changes in entire repository
git add -A

# Add changes interactively (choose hunks)
git add -p
# or
git add --patch

# Add all modified files (but not new files)
git add -u
```

### Committing Changes

**Create commits:**
```bash
# Commit with message
git commit -m "Add feature X"

# Commit with detailed message (opens editor)
git commit

# Commit with multiple paragraphs
git commit -m "Title" -m "Description line 1" -m "Description line 2"

# Commit all modified files (skip staging)
git commit -am "Fix bug in auth"

# Amend last commit (add forgotten changes)
git commit --amend

# Amend last commit without changing message
git commit --amend --no-edit

# Create empty commit (useful for triggering CI/CD)
git commit --allow-empty -m "Trigger deployment"
```

**Commit Message Best Practices:**
```
Imperative mood: "Add feature" not "Added feature"
No period at end
First line should be <= 50 characters
Blank line between title and body
Body wraps at 72 characters
Reference issues: "Fixes #123"

Example:
---
Add user authentication system

Implement JWT-based auth with refresh tokens.
Add login and register endpoints.
Secure password hashing with bcrypt.

Fixes #456
```

### Viewing History

**View commit history:**
```bash
# Show commit history
git log

# Show recent 5 commits
git log -5

# Show one-line format
git log --oneline

# Show with graph (for branches)
git log --graph --oneline --all

# Show commits affecting specific file
git log path/to/file.js

# Show commits by specific author
git log --author="John Doe"

# Show commits matching pattern
git log --grep="feature"

# Show commits after/before date
git log --since="2024-01-01" --until="2024-12-31"

# Show detailed changes
git log -p

# Show statistics
git log --stat
```

**View specific commit:**
```bash
# Show specific commit
git show commit-hash

# Show specific file in commit
git show commit-hash:path/to/file.js

# Show file history
git log -p path/to/file.js
```

### Tagging

**Create and manage tags:**
```bash
# Create lightweight tag
git tag v1.0.0

# Create annotated tag (recommended)
git tag -a v1.0.0 -m "Version 1.0.0 release"

# Show all tags
git tag

# Show specific tag
git show v1.0.0

# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0

# Push all tags
git push origin --tags

# Push specific tag
git push origin v1.0.0

# Checkout specific tag
git checkout v1.0.0
```

---

## Branching & Merging

### Understanding Branches

Branches are independent lines of development pointing to specific commits.

```
        feature-branch
               ↓
A → B → C → D → E
    ↓
main-branch
```

### Creating and Switching Branches

**Branch operations:**
```bash
# List local branches
git branch

# List all branches (local + remote)
git branch -a

# Create new branch (based on current branch)
git branch feature-x

# Create and switch to new branch
git checkout -b feature-x
# or (newer syntax)
git switch -c feature-x

# Switch to existing branch
git checkout feature-x
# or
git switch feature-x

# Rename branch
git branch -m old-name new-name

# Delete branch (safe - prevents losing commits)
git branch -d feature-x

# Force delete branch
git branch -D feature-x

# Show branch tracking info
git branch -vv
```

### Merging Branches

**Merge strategies:**
```bash
# Merge branch into current branch (creates merge commit)
git merge feature-x

# Merge with custom message
git merge feature-x -m "Merge feature-x into main"

# Merge without committing (dry run)
git merge --no-commit feature-x

# Abort merge
git merge --abort

# Merge with squash (combine commits)
git merge --squash feature-x

# Merge with rebase (linear history)
git rebase feature-x
```

### Merge Conflicts

**Handling conflicts:**
```bash
# Start merge
git merge feature-branch

# When conflicts occur, manually resolve in editor
# File will show:
# <<<<<<< HEAD
# Current branch changes
# =======
# Feature branch changes
# >>>>>>> feature-branch

# After resolving, stage changes
git add path/to/conflicted/file.js

# Complete merge
git commit

# Or abort merge
git merge --abort
```

### Rebasing

**Rebase operations:**
```bash
# Rebase current branch onto another
git rebase main

# Interactive rebase (modify commit history)
git rebase -i HEAD~3

# Continue after resolving conflicts
git rebase --continue

# Abort rebase
git rebase --abort

# Rebase with auto-squash
git rebase -i --autosquash
```

**Interactive Rebase Commands:**
```
pick   - Use commit
reword - Use commit but edit message
squash - Use commit but meld into previous
fixup  - Like squash but discard log message
drop   - Remove commit
```

### Fast-Forward vs Merge Commit

```
# Fast-forward (if no diverging commits)
git merge --ff feature-x    # Default

# Create merge commit (always)
git merge --no-ff feature-x # Recommended for features

# No fast-forward (newer)
git merge --ff-only feature-x
```

---

## Remote Repositories

### Remote Configuration

**Manage remote repositories:**
```bash
# List remote repositories
git remote

# Show remote details
git remote -v

# Add remote repository
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin upstream

# Show remote details
git remote show origin
```

### Fetching and Pulling

**Retrieve changes from remote:**
```bash
# Fetch updates without merging (safe)
git fetch

# Fetch specific remote
git fetch origin

# Fetch all remotes
git fetch --all

# Fetch and show deleted branches
git fetch --prune

# Pull = Fetch + Merge
git pull

# Pull with rebase (linear history)
git pull --rebase

# Pull specific branch
git pull origin main

# Dry run - show what would happen
git fetch --dry-run
```

### Pushing Changes

**Send commits to remote:**
```bash
# Push current branch
git push

# Push to specific remote and branch
git push origin main

# Push all branches
git push --all

# Push with all tags
git push --tags

# Push specific branch and set upstream
git push -u origin feature-x

# Force push (use with caution!)
git push --force
# Safer alternative:
git push --force-with-lease

# Delete remote branch
git push origin --delete feature-x
# or
git push origin :feature-x

# Push to different branch name
git push origin local-branch:remote-branch
```

### Tracking Branches

**Set upstream tracking:**
```bash
# Set upstream for current branch
git branch --set-upstream-to=origin/main

# Or during push
git push -u origin main

# View tracking relationships
git branch -vv

# Pull tracking branch
git pull  # Works because of tracking
```

---

## Undoing Changes

### Undoing Uncommitted Changes

**Discard changes in working directory:**
```bash
# Discard changes in specific file
git checkout -- path/to/file.js
# or (newer)
git restore path/to/file.js

# Discard all changes
git checkout -- .
# or
git restore .

# Discard staged changes (keep working changes)
git reset HEAD path/to/file.js
# or
git restore --staged path/to/file.js

# Discard both staged and working changes
git reset --hard HEAD

# Unstage all files
git reset HEAD
```

### Undoing Committed Changes

**Revert committed changes:**
```bash
# Create new commit that undoes changes
git revert commit-hash

# Revert multiple commits
git revert commit-hash1 commit-hash2

# Revert without committing
git revert --no-commit commit-hash

# Complete revert process
git revert --continue

# Abort revert
git revert --abort
```

### Resetting to Previous State

**Reset repository to specific commit:**
```bash
# Reset (keep changes in working directory)
git reset --soft commit-hash

# Reset and discard staged changes (keep working changes)
git reset --mixed commit-hash
# (default behavior)

# Reset and discard all changes
git reset --hard commit-hash

# Reset specific file
git reset --hard commit-hash -- path/to/file.js

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last 3 commits
git reset --hard HEAD~3
```

### Finding Lost Commits

**Recover lost commits:**
```bash
# Show reference logs (recent actions)
git reflog

# Restore from reflog
git checkout commit-hash

# Find and recover dangling commits
git fsck --lost-found
```

---

## Advanced Operations

### Cherry-picking

**Apply specific commits to current branch:**
```bash
# Apply single commit
git cherry-pick commit-hash

# Apply multiple commits
git cherry-pick commit-hash1 commit-hash2

# Apply range of commits
git cherry-pick commit-hash1..commit-hash2

# Cherry-pick without committing
git cherry-pick --no-commit commit-hash

# Continue after resolving conflicts
git cherry-pick --continue

# Abort cherry-pick
git cherry-pick --abort
```

### Stashing

**Temporarily save changes:**
```bash
# Save changes to stash
git stash

# Save with message
git stash save "WIP: feature X"

# List all stashes
git stash list

# Show stash contents
git stash show stash@{0}

# Apply stash (keep it)
git stash apply stash@{0}

# Pop stash (remove it)
git stash pop

# Delete specific stash
git stash drop stash@{0}

# Delete all stashes
git stash clear

# Create branch from stash
git stash branch feature-x stash@{0}

# Stash only staged changes
git stash --keep-index

# Stash untracked files
git stash -u
```

### Searching Code

**Find changes and commits:**
```bash
# Search for string in current files
git grep "search term"

# Search in specific files
git grep "term" -- "*.js"

# Show who added/removed lines
git blame path/to/file.js

# Show when line was changed
git blame -L 10,20 path/to/file.js

# Find commits that added/removed code
git log -S "search term"

# Find commits with specific pattern
git log -G "regex pattern"
```

### Bisecting

**Binary search for bug-introducing commit:**
```bash
# Start bisect
git bisect start

# Mark current commit as bad (has bug)
git bisect bad

# Mark old commit as good (no bug)
git bisect good commit-hash

# Test current commit
# Either mark as good or bad
git bisect good
# or
git bisect bad

# Git narrows down, repeat until found

# End bisect
git bisect reset
```

### Worktrees

**Work on multiple branches simultaneously:**
```bash
# Create new worktree
git worktree add path/to/worktree branch-name

# List worktrees
git worktree list

# Remove worktree
git worktree remove path/to/worktree

# Prune worktrees
git worktree prune
```

---

## Git Workflows

### Centralized Workflow

Everyone pushes to single branch (main/master).

```
Developer 1 → main ← Developer 2
Developer 3 → main ← Developer 4
```

**Steps:**
```bash
git pull origin main
# Make changes
git add .
git commit -m "message"
git push origin main
```

### Feature Branch Workflow

Each feature gets isolated branch.

```
main (stable)
  ├─ feature-auth (in development)
  ├─ feature-dashboard (in review)
  └─ hotfix-bug (in testing)
```

**Steps:**
```bash
# Create feature branch
git checkout -b feature-user-auth

# Make changes and commit
git add .
git commit -m "Add user authentication"

# Push feature branch
git push -u origin feature-user-auth

# Create pull request (on GitHub/GitLab)
# After review and approval:

# Merge to main
git checkout main
git pull origin main
git merge --no-ff feature-user-auth
git push origin main

# Delete feature branch
git branch -d feature-user-auth
git push origin --delete feature-user-auth
```

### Git Flow Workflow

Structured workflow with multiple branches for different purposes.

```
main (production)        → v1.0, v1.1, v1.2
  ↓
release (staging)       → release-1.0, release-1.1
  ↑ ↓
develop (integration)   → develop
  ↑ ↓
feature branches        → feature-*, hotfix-*, bugfix-*
```

**Main Branches:**
- **main** - Production-ready code only
- **develop** - Integration branch for features

**Supporting Branches:**
- **feature/*** - New features
- **release/*** - Prepare for production release
- **hotfix/*** - Emergency fixes to production

**Example:**
```bash
# Create feature
git checkout -b feature/user-dashboard develop

# Work and commit
git add .
git commit -m "Add dashboard UI"

# Finish feature (merge back to develop)
git checkout develop
git merge --no-ff feature/user-dashboard
git branch -d feature/user-dashboard

# When ready to release:
git checkout -b release/1.0 develop

# Version bumps and fixes
git checkout main
git merge --no-ff release/1.0
git tag -a v1.0 -m "Release v1.0"

# Merge back to develop
git checkout develop
git merge --no-ff release/1.0

# Delete release branch
git branch -d release/1.0
```

### GitHub Flow Workflow

Simplified flow with main and feature branches, requires PR review.

```
main (always deployable)
  ↑
Pull Request → Code Review → Merge
  ↑
feature-branch (in development)
```

**Steps:**
```bash
# Create feature branch
git checkout -b add-docker-support

# Commit and push
git add .
git commit -m "Add Docker support"
git push -u origin add-docker-support

# Create Pull Request on GitHub
# After review and approval

# Merge (via GitHub UI)
# Pull request is merged to main

# Delete feature branch
git branch -d add-docker-support
git push origin --delete add-docker-support
```

### Trunk-Based Development

Developers commit to main/trunk frequently with short-lived branches.

```
main (always stable)
├─ commit 1 (developer 1)
├─ commit 2 (developer 2)
├─ commit 3 (developer 1)
└─ commit 4 (developer 3)
```

**Characteristics:**
- Short-lived branches (1-2 days max)
- Frequent commits to main
- Requires good automated testing
- Continuous integration/deployment

---

## Best Practices

### Commit Best Practices

1. **Commit Early and Often**
   - Small, logical commits
   - Easy to review and understand
   - Easier to debug if issues arise

2. **Meaningful Commit Messages**
   ```
   ✅ Good: "Add user authentication with JWT"
   ❌ Bad: "updates" or "fixes stuff"
   ```

3. **Atomic Commits**
   - One logical change per commit
   - Changes work independently
   - Can be reverted cleanly

4. **Review Before Committing**
   ```bash
   git diff          # Review changes
   git add -p        # Stage specific hunks
   git commit        # Commit with message
   ```

### Branching Best Practices

1. **Branch Naming Conventions**
   ```
   feature/description    - New features
   bugfix/description     - Bug fixes
   hotfix/description     - Production fixes
   refactor/description   - Code refactoring
   docs/description       - Documentation
   ```

2. **Keep Branches Short-Lived**
   - Delete after merging
   - Reduces clutter
   - Fewer merge conflicts

3. **Branch Protection Rules**
   - Require pull request reviews
   - Require status checks to pass
   - Prevent force pushes
   - Dismiss stale reviews

### Collaboration Best Practices

1. **Use Pull Requests**
   - Code review
   - Discussion platform
   - CI/CD integration
   - Documentation

2. **Write Good PR Descriptions**
   ```markdown
   ## Description
   Brief description of changes

   ## Related Issues
   Fixes #123

   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Breaking change

   ## Testing
   Steps to test the changes

   ## Screenshots
   If applicable, add screenshots
   ```

3. **Communicate Clearly**
   - Descriptive commit messages
   - PR description and comments
   - Regular updates on progress
   - Ask questions when unclear

### Repository Management

1. **.gitignore Usage**
   ```
   # Dependencies
   node_modules/
   .venv/

   # Environment
   .env
   .env.local

   # Build outputs
   dist/
   build/

   # IDE
   .vscode/
   .idea/

   # System
   .DS_Store
   Thumbs.db
   ```

2. **Keep Repository Clean**
   - Delete merged branches
   - Archive old branches
   - Remove obsolete files
   - Maintain .gitignore

3. **Use .gitattributes**
   ```
   * text=auto
   *.js eol=lf
   *.md text
   *.png binary
   ```

### Security Best Practices

1. **Never Commit Secrets**
   ```bash
   # Use environment variables
   .env (add to .gitignore)

   # Check for secrets before committing
   git diff --cached
   ```

2. **Use SSH Keys**
   ```bash
   # Generate SSH key
   ssh-keygen -t ed25519 -C "email@example.com"

   # Use SSH URL instead of HTTPS
   git remote add origin git@github.com:user/repo.git
   ```

3. **Sign Commits**
   ```bash
   # Configure GPG signing
   git config --global user.signingkey KEY_ID

   # Sign commit
   git commit -S -m "message"

   # Sign future commits by default
   git config --global commit.gpgSign true
   ```

4. **Review Before Pushing**
   ```bash
   # Always review what you're pushing
   git log --oneline origin/main..HEAD
   git diff origin/main..HEAD
   ```

---

## Common Scenarios

### Scenario 1: Accidentally Committed to Wrong Branch

**Problem:** You made commits on main instead of feature branch.

**Solution:**
```bash
# Create feature branch from current position
git branch feature-x

# Reset main to previous state
git checkout main
git reset --hard HEAD~2  # Undo last 2 commits

# Switch to feature branch
git checkout feature-x
```

### Scenario 2: Need to Switch Branches with Uncommitted Changes

**Problem:** You have uncommitted changes but need to switch branches.

**Solution:**
```bash
# Option 1: Commit the changes
git add .
git commit -m "WIP"

# Option 2: Stash changes
git stash
git checkout other-branch
# Later restore
git stash pop

# Option 3: Use worktrees
git worktree add ../work-dir other-branch
```

### Scenario 3: Pushed to Wrong Remote/Branch

**Problem:** You pushed to wrong branch.

**Solution:**
```bash
# Undo the push (revert commits)
git revert HEAD~1..HEAD

# Or reset if not yet reviewed
git reset --hard origin/main
git push --force-with-lease

# Better: Use pull request to fix
```

### Scenario 4: Merge Conflict Resolution

**Problem:** Merge conflicts when merging branches.

**Solution:**
```bash
# Start merge
git merge feature-branch

# Find conflicted files
git status

# Edit files to resolve conflicts
# Search for: <<<<<<<, =======, >>>>>>>

# After resolving manually:
git add .
git commit -m "Resolve merge conflicts"

# Or use merge tool
git mergetool

# Or abort if unsure
git merge --abort
```

### Scenario 5: Lost Commits

**Problem:** Commits seem lost after reset or rebase.

**Solution:**
```bash
# Check reflog
git reflog

# Find lost commit
git reflog | grep "my commit message"

# Recover
git checkout lost-commit-hash

# Create branch to save it
git checkout -b recovery-branch

# Or reset to it
git reset --hard lost-commit-hash
```

### Scenario 6: Need to Undo Published Commits

**Problem:** Need to remove commits already pushed.

**Solution:**
```bash
# Option 1: Revert (safe, creates new commits)
git revert commit-hash
git push

# Option 2: Reset (force push - only if not shared)
git reset --hard HEAD~1
git push --force-with-lease

# Option 3: Use git reset + git push
git reset HEAD~1
git push --force-with-lease

# Best practice: Use revert for shared branches
```

### Scenario 7: Clean Up Old Branches

**Problem:** Many old branches clutter the repository.

**Solution:**
```bash
# Delete local merged branches
git branch --merged | grep -v main | xargs git branch -d

# Delete local unmerged branches
git branch --no-merged

# Delete remote merged branches
git push origin --delete-merged-refs

# Prune deleted remote branches
git fetch --prune
git remote prune origin
```

### Scenario 8: Partial Commit from File

**Problem:** File has multiple changes but you want to commit only some.

**Solution:**
```bash
# Interactive staging (choose hunks)
git add -p

# Options presented:
# y - stage hunk
# n - don't stage hunk
# s - split into smaller hunks
# q - quit

# Commit selected hunks
git commit -m "message"
```

### Scenario 9: Temporary Work in Progress

**Problem:** Need to pause work to switch context.

**Solution:**
```bash
# Option 1: Stash
git stash save "WIP: feature X"
git checkout other-branch
# Later resume
git checkout feature-branch
git stash pop

# Option 2: Commit as WIP
git add .
git commit -m "WIP: feature X"
# Later amend
git reset --soft HEAD~1
# Continue working...

# Option 3: Worktree
git worktree add ../temp-work other-branch
```

### Scenario 10: Synchronize Fork with Upstream

**Problem:** Your fork is behind the original repository.

**Solution:**
```bash
# Add upstream remote
git remote add upstream https://github.com/original/repo.git

# Fetch upstream changes
git fetch upstream

# Rebase local branch on upstream
git rebase upstream/main

# Push updated branch
git push --force-with-lease origin main

# Or merge (creates merge commit)
git merge upstream/main
git push origin main
```

---

## Useful Git Aliases

Create shortcuts for common commands:

```bash
# Configure aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
git config --global alias.amend 'commit --amend --no-edit'
git config --global alias.undo 'reset --soft HEAD~1'
git config --global alias.contrib 'shortlog -sn'

# Usage:
git st          # instead of git status
git co main     # instead of git checkout main
git ci -m "msg" # instead of git commit -m "msg"
```

---

## Git Tips and Tricks

### 1. Show Git Log as ASCII Tree
```bash
git log --graph --oneline --all --decorate
```

### 2. Find When a Bug Was Introduced
```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0
# Test and mark as good/bad until found
```

### 3. See What Changed Since Last Tag
```bash
git log v1.0..HEAD --oneline
git diff v1.0..HEAD
```

### 4. Format Patch for Email
```bash
git format-patch origin/main -o patches/
```

### 5. Apply Patches
```bash
git am patches/*.patch
```

### 6. See Untracked Files
```bash
git ls-files --others --exclude-standard
```

### 7. Clean Up Untracked Files
```bash
git clean -fd      # Remove untracked files and directories
git clean -fdx     # Also remove ignored files
```

### 8. Update All Submodules
```bash
git submodule update --init --recursive
```

---

## Summary

Git is a powerful distributed version control system that enables:
- **Complete project history** tracking
- **Collaborative development** with multiple developers
- **Branching and merging** for parallel work
- **Offline work** with local repositories
- **Flexibility** to support various workflows

Mastering Git is essential for modern software development. Key points:

1. **Understand the fundamental concepts** - repositories, commits, branches
2. **Choose appropriate workflow** - Feature branch, Git Flow, GitHub Flow
3. **Write good commit messages** - Clear and descriptive
4. **Collaborate effectively** - Use PRs, communicate clearly
5. **Practice regularly** - Familiarity comes with use
6. **Know how to undo mistakes** - Reset, revert, reflog
7. **Keep security in mind** - Protect secrets, use SSH/GPG

With Git proficiency, you'll work more efficiently, collaborate better, and maintain code quality across projects.
