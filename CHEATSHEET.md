# Git cheat sheet

## Activate the virtual environment
```bash
source .venv/bin/activate
```

## Check repository status
```bash
git status
```

## Stage and commit changes
```bash
git add <files>
git commit -m "your message"
```

## Fetch official updates
```bash
git fetch upstream
```
Why: downloads the latest changes from the official repo without modifying your current branch yet.

## Rebase onto the latest official changes
```bash
git rebase upstream/main
```
Why: applies your local commits on top of the newest official course changes and keeps your history clean.

## Push to your own fork
```bash
git push origin main
```
Why: uploads your current branch to your own GitHub fork.

## Force-push after a rebase
```bash
git push --force-with-lease origin main
```
Why: use this when `git rebase` rewrote your commit history and a normal push is rejected.
