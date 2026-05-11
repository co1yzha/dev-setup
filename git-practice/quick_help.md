## Git Quick Help (daily use)

### Configure once (global)
```bash
# Identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Quality of life
git config --global init.defaultBranch main
git config --global pull.ff only                 # prevent accidental merge commits on pull
git config --global push.autoSetupRemote true    # auto set upstream on first push
git config --global core.editor "code --wait"    # or your favourite editor
```

### Start work
```bash
# Clone or initialise a repo
git clone <url>  # or:  git init -b main

# Create a feature branch from latest main
git fetch origin
git switch main
git pull --ff-only
git switch -c feature/short-topic
```

### Status, stage, commit
```bash
git status
git add <path>          # stage specific files
git add -p              # stage hunks interactively
git commit -m "feat: concise, imperative summary"

# Amend last commit (e.g., add forgotten file)
git add missed.file
git commit --amend --no-edit
```

### Share work (push / track)
```bash
git push -u origin feature/short-topic   # first push (sets upstream)
git push                                 # subsequent pushes
```

### Keep your branch up to date
Rebase (clean history, recommended for feature branches):
```bash
git fetch origin
git switch feature/short-topic
git rebase origin/main
# resolve conflicts -> edit files -> git add <resolved files>
git rebase --continue    # or: --abort to cancel
git push --force-with-lease
```

Merge (preserves merge commits):
```bash
git fetch origin
git switch feature/short-topic
git merge origin/main
git push
```

### Inspect changes
```bash
git diff                 # working tree vs index
git diff --staged       # index vs HEAD
git log --oneline --graph --decorate --all
git show <commit>
```

### Stash work-in-progress
```bash
git stash -u -m "wip: context"
git stash list
git stash pop            # apply latest and drop
```

### Undo and fix
```bash
# Unstage changes (keep in working tree)
git restore --staged <path>

# Discard local changes to a file
git restore --source=HEAD -- <path>

# Move HEAD back but keep changes staged (soft reset)
git reset --soft HEAD~1

# Hard reset last commit (destructive)
git reset --hard HEAD~1

# Create a new commit that reverts a bad commit (safe on shared branches)
git revert <commit>
```

### Resolve conflicts (common loop)
```bash
# after rebase/merge stops with conflicts
git status
# edit conflicted files, keep desired changes
git add <resolved files>
git rebase --continue    # or: git merge --continue
```

### Tags (releases)
```bash
git tag -a v1.2.3 -m "v1.2.3"
git push origin v1.2.3
git tag                 # list
```

### Clean up
```bash
git fetch --prune                     # remove deleted remote refs
git branch --merged main              # see merged local branches
git branch -d feature/short-topic     # delete local (safe)
git push origin --delete feature/short-topic
```

### Useful extras
```bash
git blame <file>          # who last changed each line
git grep "pattern"        # search tracked files
git cherry-pick <commit>  # copy a commit onto current branch
git worktree add ../repo-main main   # another checkout without switching

# Investigate regressions
git bisect start
git bisect bad
git bisect good <good_commit_or_tag>
# test -> mark good/bad iteratively -> git bisect reset
```

### Conventional commit message hints
- feat: new user-facing feature
- fix: bug fix
- docs: documentation only
- refactor: code change that neither fixes a bug nor adds a feature
- perf: performance improvement
- test: adding or correcting tests
- chore: build/infra/maintenance

Tip: keep subject ≤ 72 chars, imperative mood, and add details in body if needed.

