# 6. Version Control with Git

## 📚 Quick Summary

Git is the industry standard for version control - essential for every DevOps engineer!

**What You'll Learn:**
- **Git Fundamentals**: Repository, commits, branches, merges
- **Branching Strategies**: GitFlow, GitHub Flow, trunk-based development
- **Collaboration**: Pull requests, code reviews, merge conflicts
- **Advanced Git**: Rebase, cherry-pick, stash, tags
- **GitHub/GitLab**: CI/CD integration, webhooks, automation
- **Best Practices**: Commit messages, .gitignore, security

**Why This Matters:**
- Version control is the foundation of DevOps
- Every CI/CD pipeline starts with Git
- Collaboration with team members
- Infrastructure as Code stored in Git
- Interview questions: 10% are Git-based

**Interview Reality:**
"How do you resolve merge conflicts?" = Daily DevOps task!

---

## 📖 Simple Explanation

**What is Version Control?**
Version control is like "Track Changes" for code, but 1000x more powerful:
- **Save Points**: Like video game checkpoints
- **Time Travel**: Go back to any previous version
- **Parallel Universes**: Work on features simultaneously (branches)
- **Collaboration**: Multiple people work without stepping on toes
- **Audit Trail**: Who changed what, when, and why

**Why Git?**
```
Before Git:
final_version.py
final_version_2.py
final_version_ACTUALLY_FINAL.py
final_version_ACTUALLY_FINAL_v2.py
😱

With Git:
project.py (all versions tracked automatically)
- Full history
- Branch for each feature
- Merge when ready
😊
```

**Real-World Analogy:**
```
Git = Google Docs with superpowers
- Every save is permanent
- Can see who wrote each line
- Can create alternate versions (branches)
- Can merge versions intelligently
```

---

## Table of Contents
- [Git Fundamentals](#git-fundamentals)
- [Basic Git Commands](#basic-git-commands)
- [Branching and Merging](#branching-and-merging)
- [Branching Strategies](#branching-strategies)
- [Advanced Git Commands](#advanced-git-commands)
- [Resolving Conflicts](#resolving-conflicts)
- [GitHub and GitLab](#github-and-gitlab)
- [Git Hooks](#git-hooks)
- [Best Practices](#best-practices)
- [Common Scenarios](#common-scenarios)
- [Interview Questions](#interview-questions)

---

## Git Fundamentals

### 📖 Simple Explanation

**Core Concepts:**

**1. Repository (Repo)**
- Project folder tracked by Git
- Contains all files + complete history
- Local (your computer) or Remote (GitHub)

**2. Commit**
- Snapshot of your project at a point in time
- Permanent, immutable
- Has unique ID (SHA hash)

**3. Branch**
- Parallel version of code
- Allows isolated development
- Easy to create, merge, delete

**4. Remote**
- Version of repo hosted elsewhere (GitHub, GitLab)
- Team members share code here

---

### Git Workflow

```
Working Directory  →  Staging Area  →  Local Repository  →  Remote Repository
(your files)          (git add)        (git commit)         (git push)

Example:
1. Edit file.py          ← Working Directory
2. git add file.py       ← Staging Area (ready to commit)
3. git commit -m "msg"   ← Local Repository (saved locally)
4. git push origin main  ← Remote Repository (GitHub/GitLab)
```

---

### Three States of Git

```
┌─────────────────────────────────────────────────────────┐
│                    Git File States                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Modified          Staged              Committed         │
│  (Changed)   →    (git add)     →     (git commit)      │
│                                                          │
│  file.py          file.py              file.py          │
│  edited           ready to             saved in         │
│                   commit               history          │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Basic Git Commands

### Initial Setup

```bash
# Configure Git (first time setup)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Set default editor
git config --global core.editor "vim"

# View all config
git config --list

# View specific config
git config user.name
```

---

### Creating Repositories

```bash
# Initialize new repository
git init
# Creates .git/ directory (Git database)

# Clone existing repository
git clone https://github.com/user/repo.git

# Clone to specific directory
git clone https://github.com/user/repo.git my-project

# Clone specific branch
git clone -b develop https://github.com/user/repo.git
```

---

### Basic Commands

```bash
# Check status (most used command!)
git status
# Shows:
# - Modified files
# - Staged files
# - Untracked files
# - Current branch

# Add files to staging
git add file.py                    # single file
git add file1.py file2.py         # multiple files
git add .                         # all files in current directory
git add -A                        # all files in repository
git add *.py                      # all Python files

# Commit changes
git commit -m "Add user authentication"

# Add and commit in one step (tracked files only)
git commit -am "Update README"

# View commit history
git log                           # full history
git log --oneline                 # compact view
git log --graph --oneline         # visual branch graph
git log -5                        # last 5 commits
git log --author="John"           # commits by author
git log --since="2 weeks ago"     # recent commits

# Show changes
git diff                          # changes not staged
git diff --staged                 # changes staged
git diff HEAD                     # all changes
git diff branch1..branch2         # compare branches

# View specific commit
git show <commit-hash>
git show HEAD                     # last commit
git show HEAD~1                   # second last commit
```

---

### Undoing Changes

```bash
# Discard changes in working directory
git checkout -- file.py           # old way
git restore file.py               # new way (Git 2.23+)

# Unstage file (keep changes)
git reset HEAD file.py            # old way
git restore --staged file.py     # new way

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes) ⚠️ DANGEROUS
git reset --hard HEAD~1

# Revert a commit (creates new commit)
git revert <commit-hash>          # safer than reset

# Discard all local changes ⚠️ DANGEROUS
git reset --hard HEAD
git clean -fd                     # remove untracked files
```

---

## Branching and Merging

### 📖 Simple Explanation

**Branches = Parallel Universes**
```
main:     ●──●──●──●──────────●  (production code)
               \              /
feature:        ●──●──●──●──●    (new feature development)

You can:
- Work on feature without breaking main
- Switch between branches instantly
- Merge feature into main when ready
```

---

### Branch Commands

```bash
# List branches
git branch                        # local branches
git branch -r                     # remote branches
git branch -a                     # all branches

# Create branch
git branch feature-login

# Switch to branch
git checkout feature-login        # old way
git switch feature-login          # new way (Git 2.23+)

# Create and switch in one command
git checkout -b feature-login     # old way
git switch -c feature-login       # new way

# Rename branch
git branch -m old-name new-name

# Delete branch
git branch -d feature-login       # safe delete (merged only)
git branch -D feature-login       # force delete ⚠️

# Delete remote branch
git push origin --delete feature-login
```

---

### Merging Branches

```bash
# Merge branch into current branch
git checkout main
git merge feature-login

# Fast-forward merge (linear history)
# Happens when no divergent commits
main:     ●──●
               \
feature:        ●──●  →  main: ●──●──●──●

# Three-way merge (creates merge commit)
main:     ●──●──●────────●  (merge commit)
               \        /
feature:        ●──●──●

# Merge with commit message
git merge feature-login -m "Merge feature-login into main"

# Abort merge (if conflicts)
git merge --abort

# Merge strategies
git merge --no-ff feature-login   # always create merge commit
git merge --squash feature-login  # squash all commits into one
```

---

## Branching Strategies

### 📖 Simple Explanation

Different teams use different branching strategies based on their needs.

---

### 1. Git Flow (Traditional)

**Branches:**
- `main` - Production code
- `develop` - Integration branch
- `feature/*` - New features
- `release/*` - Release preparation
- `hotfix/*` - Emergency fixes

```
main:        ●────────●────────────●─────●
                       \          /       \
release:                ●────────●         \
                       /                    \
develop:    ●────●────●──●────●────●────●───●
                 \      /      \        /
feature-1:        ●────●        \      /
                                 \    /
feature-2:                        ●──●
```

**Workflow:**
```bash
# Start new feature
git checkout develop
git checkout -b feature/user-auth
# ... work on feature ...
git checkout develop
git merge feature/user-auth
git branch -d feature/user-auth

# Prepare release
git checkout develop
git checkout -b release/v1.2.0
# ... bug fixes, version bump ...
git checkout main
git merge release/v1.2.0
git tag v1.2.0
git checkout develop
git merge release/v1.2.0

# Hotfix
git checkout main
git checkout -b hotfix/critical-bug
# ... fix bug ...
git checkout main
git merge hotfix/critical-bug
git tag v1.2.1
git checkout develop
git merge hotfix/critical-bug
```

**When to Use:**
- Scheduled releases
- Multiple versions in production
- Large teams
- Enterprise applications

---

### 2. GitHub Flow (Simple)

**Branches:**
- `main` - Always deployable
- `feature/*` - All changes (features, fixes)

```
main:     ●──●────────●────────●──●
               \      /         \   \
feature-1:      ●────●           \   \
                                  \   \
feature-2:                         ●───●
```

**Workflow:**
```bash
# Create feature branch from main
git checkout main
git pull origin main
git checkout -b add-payment-gateway

# Work and commit
git add .
git commit -m "Add Stripe integration"

# Push to remote
git push origin add-payment-gateway

# Create Pull Request on GitHub
# After review and CI passes, merge to main
# Deploy main immediately
```

**When to Use:**
- Continuous deployment
- Small to medium teams
- Web applications
- Rapid iterations

---

### 3. Trunk-Based Development

**Strategy:**
- Everyone commits to `main` (trunk)
- Short-lived feature branches (< 1 day)
- Feature flags for incomplete features
- Continuous integration

```
main:  ●──●──●──●──●──●──●──●──●──●
            \  /  \  /  \  /
features:    ●●    ●●    ●●
         (short-lived branches)
```

**Workflow:**
```bash
# Small feature (commit directly)
git checkout main
git pull origin main
# ... make small change ...
git add .
git commit -m "Update button color"
git push origin main

# Larger feature (short branch)
git checkout -b quick-feature
# ... work for few hours ...
git add .
git commit -m "Add export feature"
git checkout main
git pull origin main
git merge quick-feature
git push origin main
git branch -d quick-feature

# Feature flags for incomplete work
if feature_flag_enabled('new_ui'):
    render_new_ui()
else:
    render_old_ui()
```

**When to Use:**
- Google, Facebook style
- Very high velocity teams
- Excellent CI/CD pipeline
- Strong testing culture

---

## Advanced Git Commands

### Rebasing

```bash
# Rebase current branch onto main
git checkout feature-branch
git rebase main

# Interactive rebase (edit history)
git rebase -i HEAD~3              # last 3 commits

# Interactive rebase options:
# pick   - use commit
# reword - change commit message
# edit   - edit commit
# squash - combine with previous commit
# drop   - remove commit

# Abort rebase
git rebase --abort

# Continue after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip
```

**Rebase vs Merge:**
```
Merge (preserves history):
main:     ●──●──●────────●
               \        /
feature:        ●──●──●

Rebase (linear history):
main:     ●──●──●
                 \
feature:          ●'──●'──●'  (commits replayed)
```

---

### Stashing

```bash
# Stash current changes
git stash
git stash save "Work in progress"

# List stashes
git stash list

# Apply most recent stash
git stash apply
git stash pop                     # apply and remove

# Apply specific stash
git stash apply stash@{2}

# View stash contents
git stash show
git stash show -p                 # show diff

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Stash including untracked files
git stash -u

# Create branch from stash
git stash branch new-branch-name
```

**When to Use Stash:**
```bash
# Scenario: Working on feature, urgent bug needs fixing
git stash                         # save current work
git checkout main
git checkout -b hotfix
# ... fix bug ...
git checkout feature-branch
git stash pop                     # restore work
```

---

### Cherry-pick

```bash
# Apply specific commit to current branch
git cherry-pick <commit-hash>

# Cherry-pick multiple commits
git cherry-pick <hash1> <hash2>

# Cherry-pick range
git cherry-pick <start-hash>..<end-hash>

# Cherry-pick without committing
git cherry-pick -n <commit-hash>

# Abort cherry-pick
git cherry-pick --abort
```

**Example:**
```
main:     ●──●──●
               \
develop:        ●──●(fix)──●

# Want to apply fix to main without other commits
git checkout main
git cherry-pick <fix-commit-hash>

main:     ●──●──●──●(fix)
```

---

### Tags

```bash
# List tags
git tag
git tag -l "v1.*"                 # list matching pattern

# Create lightweight tag
git tag v1.0.0

# Create annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.0 <commit-hash> -m "Message"

# View tag details
git show v1.0.0

# Push tags to remote
git push origin v1.0.0            # single tag
git push origin --tags            # all tags

# Delete tag
git tag -d v1.0.0                 # local
git push origin --delete v1.0.0  # remote

# Checkout tag
git checkout v1.0.0               # detached HEAD state
git checkout -b version1 v1.0.0   # create branch from tag
```

---

### Git Reflog (Recover Lost Commits)

```bash
# View reference log (all HEAD movements)
git reflog

# Output example:
# abc1234 HEAD@{0}: commit: Add feature
# def5678 HEAD@{1}: checkout: moving from main to feature
# ghi9012 HEAD@{2}: reset: moving to HEAD~1

# Recover lost commit
git checkout <commit-hash-from-reflog>
git checkout -b recovery-branch

# Undo accidental reset
git reset --hard HEAD@{1}
```

---

## Resolving Conflicts

### 📖 Simple Explanation

Conflicts happen when Git can't automatically merge changes.

**Example Conflict:**
```python
<<<<<<< HEAD (your changes)
def calculate_total(price, tax):
    return price + tax
=======
def calculate_total(price, tax_rate):
    return price * (1 + tax_rate)
>>>>>>> feature-branch (incoming changes)
```

---

### Conflict Resolution Workflow

```bash
# Start merge
git merge feature-branch
# Auto-merging file.py
# CONFLICT (content): Merge conflict in file.py
# Automatic merge failed; fix conflicts and then commit the result.

# Check which files have conflicts
git status

# Option 1: Manually resolve in editor
# Edit file, choose or combine changes
# Remove conflict markers (<<<<, ====, >>>>)

# Option 2: Use merge tool
git mergetool

# Option 3: Choose one version
git checkout --ours file.py       # keep your version
git checkout --theirs file.py     # keep their version

# After resolving conflicts
git add file.py
git commit                        # completes merge

# Or abort merge
git merge --abort
```

---

### Conflict Resolution Strategies

```bash
# Always use our version for specific files
git merge -X ours feature-branch

# Always use their version
git merge -X theirs feature-branch

# Find merge conflicts in large repositories
git diff --name-only --diff-filter=U

# View conflict in three-way diff
git diff --merge

# Show what changed in both branches
git log --merge
```

---

## GitHub and GitLab

### Remote Operations

```bash
# View remotes
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git

# Change remote URL
git remote set-url origin https://github.com/user/new-repo.git

# Remove remote
git remote remove origin

# Fetch from remote (download, don't merge)
git fetch origin

# Pull from remote (fetch + merge)
git pull origin main

# Pull with rebase (cleaner history)
git pull --rebase origin main

# Push to remote
git push origin main

# Push all branches
git push --all origin

# Push with upstream tracking
git push -u origin feature-branch
# After this, just use: git push

# Force push ⚠️ DANGEROUS (overwrites remote)
git push --force origin main
# Safer force push (checks remote hasn't changed)
git push --force-with-lease origin main
```

---

### Pull Requests / Merge Requests

**Creating a PR (Command Line + GitHub CLI):**
```bash
# Install GitHub CLI
# macOS: brew install gh
# Linux: see https://cli.github.com

# Authenticate
gh auth login

# Create PR
gh pr create --title "Add user authentication" --body "Implements login/logout"

# Create draft PR
gh pr create --draft

# View PRs
gh pr list

# View specific PR
gh pr view 123

# Checkout PR locally
gh pr checkout 123

# Review PR
gh pr review 123 --approve
gh pr review 123 --request-changes --body "Please add tests"

# Merge PR
gh pr merge 123
gh pr merge 123 --squash
gh pr merge 123 --rebase
```

---

### GitHub Actions Integration

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: |
          npm install
          npm test
      
      - name: Build
        run: npm run build
```

---

## Git Hooks

### 📖 Simple Explanation

Git hooks are scripts that run automatically at certain points in Git workflow.

---

### Common Hooks

**Pre-commit Hook** (`.git/hooks/pre-commit`):
```bash
#!/bin/bash
# Run linter before commit

echo "Running linter..."
npm run lint

if [ $? -ne 0 ]; then
    echo "❌ Linting failed. Commit aborted."
    exit 1
fi

echo "✓ Linting passed"
exit 0
```

**Commit-msg Hook** (`.git/hooks/commit-msg`):
```bash
#!/bin/bash
# Enforce commit message format

commit_msg=$(cat "$1")
pattern="^(feat|fix|docs|style|refactor|test|chore): .+"

if ! echo "$commit_msg" | grep -qE "$pattern"; then
    echo "❌ Invalid commit message format"
    echo "Format: <type>: <description>"
    echo "Types: feat, fix, docs, style, refactor, test, chore"
    exit 1
fi

exit 0
```

**Pre-push Hook** (`.git/hooks/pre-push`):
```bash
#!/bin/bash
# Run tests before push

echo "Running tests..."
npm test

if [ $? -ne 0 ]; then
    echo "❌ Tests failed. Push aborted."
    exit 1
fi

echo "✓ Tests passed"
exit 0
```

---

### Using Husky (Easier Hook Management)

```bash
# Install Husky
npm install --save-dev husky

# Initialize Husky
npx husky install

# Add pre-commit hook
npx husky add .husky/pre-commit "npm run lint"

# Add commit-msg hook
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit "$1"'
```

---

## Best Practices

### Commit Messages

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting, no code change
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Maintenance

**Examples:**
```bash
# Good commits
git commit -m "feat(auth): add JWT authentication"
git commit -m "fix(api): resolve timeout issue in user endpoint"
git commit -m "docs: update README with deployment instructions"

# Bad commits ❌
git commit -m "fixed stuff"
git commit -m "WIP"
git commit -m "asdfasdf"
git commit -m "changes"
```

**Detailed Commit:**
```
feat(payment): integrate Stripe payment gateway

- Add Stripe SDK dependency
- Implement payment processing service
- Add webhook handler for payment events
- Update user model with payment status

Closes #123
```

---

### .gitignore Best Practices

```bash
# .gitignore

# Dependencies
node_modules/
vendor/
*.egg-info/

# Environment variables
.env
.env.local
*.env

# Build outputs
dist/
build/
*.exe
*.dll

# IDE files
.vscode/
.idea/
*.swp
*.swo

# OS files
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Temporary files
*.tmp
*.temp
.cache/

# Secrets (NEVER commit!)
*.pem
*.key
secrets.yml
credentials.json
```

**Using global .gitignore:**
```bash
# Create global ignore file
touch ~/.gitignore_global

# Configure Git to use it
git config --global core.excludesfile ~/.gitignore_global

# Add OS-specific files
echo ".DS_Store" >> ~/.gitignore_global
echo "Thumbs.db" >> ~/.gitignore_global
```

---

### Security Best Practices

```bash
# ⚠️ NEVER commit:
- Passwords
- API keys
- Private keys
- Tokens
- Database credentials
- .env files

# If you accidentally committed secrets:

# 1. Remove from history (small file)
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch secrets.yml" \
  --prune-empty --tag-name-filter cat -- --all

# 2. Or use BFG Repo-Cleaner (faster)
bfg --delete-files secrets.yml
git reflog expire --expire=now --all
git gc --prune=now --aggressive

# 3. Force push (after backup!)
git push --force --all

# 4. IMPORTANT: Rotate the compromised secrets!
# Changed credentials in your Git history are still compromised
```

---

### Repository Management

```bash
# Keep repository clean
git gc                            # garbage collection
git prune                         # remove unreachable objects

# Check repository health
git fsck                          # file system check

# Optimize repository
git repack -a -d --depth=250 --window=250

# View repository size
git count-objects -vH

# Find large files
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | sed -n 's/^blob //p' \
  | sort --numeric-sort --key=2 \
  | tail -n 10
```

---

## Common Scenarios

### Scenario 1: Undo Last Commit

```bash
# Keep changes, undo commit
git reset --soft HEAD~1

# Discard changes and commit
git reset --hard HEAD~1

# Amend last commit (change message or add files)
git commit --amend
git commit --amend -m "New commit message"
```

---

### Scenario 2: Update Feature Branch with Main

```bash
# Option 1: Merge (preserves history)
git checkout feature-branch
git merge main

# Option 2: Rebase (linear history)
git checkout feature-branch
git rebase main
# If conflicts, resolve and:
git rebase --continue
```

---

### Scenario 3: Split Large Commit

```bash
# Reset last commit but keep changes
git reset HEAD~1

# Stage and commit in smaller chunks
git add file1.py
git commit -m "Part 1: Add database models"

git add file2.py
git commit -m "Part 2: Add API endpoints"
```

---

### Scenario 4: Move Commits to New Branch

```bash
# Accidentally committed to main, should be on feature branch
git branch feature-branch        # create branch with current commits
git reset --hard HEAD~3          # remove last 3 commits from main
git checkout feature-branch      # commits are now on feature branch
```

---

### Scenario 5: Sync Fork with Upstream

```bash
# Add upstream remote
git remote add upstream https://github.com/original/repo.git

# Fetch upstream changes
git fetch upstream

# Merge upstream changes
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

---

## Interview Questions

### Q1: Explain Git fetch vs Git pull
**Answer:**
```
git fetch:
- Downloads changes from remote
- Does NOT merge into current branch
- Safe, non-destructive

git pull:
- git fetch + git merge
- Downloads AND merges
- Can cause conflicts

Best practice: Use fetch to review, then merge manually
git fetch origin
git diff main origin/main
git merge origin/main
```

---

### Q2: What is Git rebase and when to use it?
**Answer:**
```
Rebase replays commits from one branch onto another.

Use rebase when:
- Keeping feature branch up-to-date with main
- Creating linear history
- Cleaning up commits before merge

DON'T rebase if:
- Branch is shared/public
- Commits already pushed to main

Interactive rebase for cleaning history:
git rebase -i HEAD~5
```

---

### Q3: How do you recover a deleted branch?
**Answer:**
```bash
# Find the commit
git reflog
# Look for: "checkout: moving from deleted-branch"

# Recreate branch
git checkout -b recovered-branch <commit-hash>

# Or if just deleted
git branch recovered-branch HEAD@{1}
```

---

### Q4: Explain merge conflict resolution
**Answer:**
```
Conflicts occur when same lines modified in different branches.

Resolution steps:
1. git merge feature-branch (conflict occurs)
2. git status (identify conflicted files)
3. Open files, resolve conflicts manually
   - Remove <<<, ===, >>> markers
   - Keep desired changes
4. git add resolved-file.py
5. git commit (complete merge)

Or use: git merge --abort to cancel
```

---

### Q5: What are Git submodules?
**Answer:**
```bash
Submodules = repositories inside repositories

# Add submodule
git submodule add https://github.com/user/library.git libs/library

# Clone repo with submodules
git clone --recursive https://github.com/user/project.git

# Update submodules
git submodule update --remote

# Alternative: Git subtree (easier)
git subtree add --prefix=libs/library https://github.com/user/library.git main
```

---

### Q6: How to handle large files in Git?
**Answer:**
```bash
# Use Git LFS (Large File Storage)
git lfs install

# Track large files
git lfs track "*.psd"
git lfs track "*.mp4"

# Add .gitattributes
git add .gitattributes

# Commit and push normally
git add large-file.mp4
git commit -m "Add video"
git push

# LFS stores pointers in repo, files on LFS server
```

---

### Q7: Explain detached HEAD state
**Answer:**
```
Detached HEAD = not on any branch

Occurs when:
- git checkout <commit-hash>
- git checkout <tag>

To fix:
git checkout main                    # return to branch
git checkout -b new-branch          # create branch from current state

Commits in detached HEAD are lost unless you create a branch!
```

---

### Q8: How to find who changed a specific line?
**Answer:**
```bash
# Git blame - show who last modified each line
git blame file.py

# Show specific lines
git blame -L 10,20 file.py

# Ignore whitespace changes
git blame -w file.py

# Show email instead of name
git blame -e file.py

# Find when line was deleted
git log -S "function_name" -- file.py
```

---

## Quick Reference

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email@example.com"

# Create
git init
git clone <url>

# Basic
git status
git add .
git commit -m "message"
git push
git pull

# Branching
git branch                      # list
git branch <name>              # create
git checkout <name>            # switch
git checkout -b <name>         # create + switch
git merge <branch>             # merge
git branch -d <name>           # delete

# History
git log
git log --oneline --graph
git diff
git show <commit>

# Undo
git restore <file>             # discard changes
git restore --staged <file>    # unstage
git reset --soft HEAD~1        # undo commit, keep changes
git reset --hard HEAD~1        # undo commit, discard changes

# Remote
git remote -v
git fetch
git pull
git push
git push -u origin <branch>

# Advanced
git stash                      # save work
git stash pop                  # restore work
git rebase <branch>            # rebase
git cherry-pick <commit>       # apply commit
git reflog                     # recover lost commits
```

---

## Summary

**Key Takeaways:**
1. Git is essential for all DevOps work
2. Commit often, push regularly
3. Write meaningful commit messages
4. Use branches for all changes
5. Never commit secrets
6. Learn to resolve conflicts
7. Understand rebase vs merge
8. Use .gitignore properly

**Next Steps:**
1. Practice Git commands daily
2. Learn branching strategy your team uses
3. Set up Git hooks for quality checks
4. Integrate Git with CI/CD pipelines
5. Master GitHub/GitLab features

**Remember:**
- Git is your time machine
- Branches are cheap, use them
- Commits tell a story, make it clear
- When in doubt, commit!

---

**Pro Tips:**
```bash
# Aliases make life easier
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg 'log --oneline --graph --all'
```

**Happy Coding! 🚀**

