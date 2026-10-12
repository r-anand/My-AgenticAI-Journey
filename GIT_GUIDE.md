# Git and GitHub command guide

This guide covers common Git workflows on macOS and Windows, GitHub account setup, pull requests, and a practical commit-message convention.

## 1. Before you start

Install Git on your platform:

- **macOS:** install Apple's command-line tools with `xcode-select --install`, or install Git using Homebrew: `brew install git`.
- **Windows:** install [Git for Windows](https://git-scm.com/download/win). Use Git Bash or PowerShell; the Git commands in this guide work in either shell.

Check that Git is available:

```sh
git --version
```

Most commands below are the same in macOS Terminal, Windows PowerShell, and Git Bash. Commands explicitly marked **PowerShell** or **macOS** are shell-specific.

## 2. Configure your Git identity

Git commit identity and GitHub authentication are separate:

- `user.name` and `user.email` are recorded as the author of new commits.
- GitHub authentication controls whether you can fetch or push to a GitHub repository.
- The commit email should be verified on the GitHub account where you want commits attributed. You can use a GitHub-provided `noreply` address if you prefer not to expose your personal email.

### Set a default identity for one account

Set the default identity globally (for all repositories without a local override):

```sh
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
```

Inspect the configured values:

```sh
git config --global --get user.name
git config --global --get user.email
git config --global --list
```

### Use separate personal and organization identities

For many people, the simplest and safest approach is to set the identity **inside each repository**. First change into the repository, then set the appropriate author details:

```sh
git config user.name "Your Organization Name"
git config user.email "you@company.example"
```

For a personal repository, use your personal name and GitHub-verified personal email instead. Repository-local settings override global settings and do not affect other repositories.

Check which identity will be used in the current repository:

```sh
git config --show-origin --get user.name
git config --show-origin --get user.email
```

If you regularly work with both accounts, keep their repositories in clearly separated folders, for example `~/src/personal/` and `~/src/work/`, and consider setting repository-local identities immediately after cloning. This avoids accidentally authoring company work with your personal email or vice versa.

### Authenticate to GitHub

The GitHub CLI (`gh`) is optional but convenient for authentication and pull requests. Install it from [GitHub CLI](https://cli.github.com/).

Sign in and check the active account:

```sh
gh auth login
gh auth status
```

To use two GitHub accounts with the CLI, sign in to each account when prompted, then check or switch the active account:

```sh
gh auth status
gh auth switch --user YOUR_GITHUB_USERNAME
```

Use the organization account when working with repositories that require it. Organization membership, repository permissions, and any required SSO authorization must also be in place; switching accounts does not grant access by itself.

## 3. Clone a repository and check its status

Clone a repository, then enter its directory:

```sh
git clone https://github.com/OWNER/REPOSITORY.git
cd REPOSITORY
git status
```

To check or change the configured remote:

```sh
git remote -v
git remote add origin https://github.com/OWNER/REPOSITORY.git
git remote set-url origin https://github.com/OWNER/REPOSITORY.git
```

Use `git remote add` only if the remote is not already configured; otherwise, use `git remote set-url` to change it. `origin` is a common remote name, not a special keyword. Spell it exactly.

## 4. Everyday changes: inspect, stage, commit, push

Check changed files and review a diff:

```sh
git status
git diff
```

Stage one file, several files, or all changes:

```sh
git add path/to/file
git add path/to/file1 path/to/file2
git add .
```

Review what is staged, then commit:

```sh
git diff --staged
git commit -m "docs: add setup instructions"
```

Push the current branch to GitHub and set its upstream the first time:

```sh
git push -u origin HEAD
```

For later pushes from that branch:

```sh
git push
```

To update local references and download new commits without merging them:

```sh
git fetch origin
```

## 5. Create and use a branch

Start from the latest `main`, then create a descriptive branch:

```sh
git switch main
git pull --ff-only origin main
git switch -c feat/add-study-plan
```

Use `fix/`, `docs/`, or another purpose-based prefix when it better describes the change. Commit your work on that branch and push it:

```sh
git add .
git commit -m "feat: add study plan view"
git push -u origin HEAD
```

After the pull request has been merged, update `main` and remove the local feature branch:

```sh
git switch main
git pull --ff-only origin main
git branch -d feat/add-study-plan
```

Delete a remote branch only when appropriate and permitted:

```sh
git push origin --delete feat/add-study-plan
```

## 6. Add or delete files

Git tracks file changes, not empty directories. Create the file in your editor, then stage it:

```sh
git add path/to/new-file
```

Remove a tracked file from the repository and your working directory:

```sh
git rm path/to/file
git commit -m "chore: remove obsolete file"
```

Stop tracking a file but keep your local copy (for example, after adding it to `.gitignore`):

```sh
git rm --cached path/to/file
```

Add the appropriate ignore rule to `.gitignore`, then commit both changes. **Ignoring a file does not remove it from Git history.** If a secret was committed, revoke or rotate it immediately; removing the file in a later commit does not erase the secret from earlier history.

Undo selected, uncommitted changes carefully:

```sh
# Unstage a file, keeping its edits
git restore --staged path/to/file

# Discard edits to a tracked file (destructive)
git restore path/to/file
```

Review `git status` before using the destructive restore command.

## 7. Resolve merge conflicts

When Git reports a conflict, do not delete or reset work blindly. Check what is conflicted:

```sh
git status
```

Open each conflicted file. Resolve the sections marked with `<<<<<<<`, `=======`, and `>>>>>>>`, keeping the intended final content and removing the conflict markers. Then stage the resolved files.

For a merge:

```sh
git add path/to/resolved-file
git merge --continue
```

If the merge was started by `git pull` and you want to abandon that merge instead:

```sh
git merge --abort
```

For a rebase conflict, stage each resolved file and continue the rebase:

```sh
git add path/to/resolved-file
git rebase --continue
```

To abandon an in-progress rebase:

```sh
git rebase --abort
```

After resolving conflicts, run the project's tests, inspect `git status`, and push the updated branch. Avoid starting a merge and a rebase at the same time; use the operation Git says is in progress.

## 8. Open a pull request

The usual workflow is: create a branch, commit changes, push the branch, and open a pull request (PR) from that branch into the repository's base branch. A PR is for proposing and reviewing changes; it does not merge automatically.

### Using GitHub CLI

After authenticating with `gh`, push the branch and create the PR:

```sh
git push -u origin HEAD
gh pr create --base main --title "feat: add study plan" --body "Adds a study plan view."
```

To create a draft PR:

```sh
gh pr create --draft --base main --title "feat: add study plan" --body "Work in progress."
```

Common PR commands:

```sh
gh pr status
gh pr view
gh pr checkout 123
```

Replace `123` with the PR number.

### Without GitHub CLI

Push the feature branch, then open the repository on GitHub in a browser. GitHub will usually offer a **Compare & pull request** button. Select the correct base branch, review the diff, add a clear title and description, and create the PR.

Follow repository review requirements, respond to feedback, and wait for required approvals and checks before merging.

## 9. Conventional Commit messages

[Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) is a convention for writing clear, structured commit subjects. The common format is:

```text
<type>[optional scope]: <short description>
```

Common types:

| Type | Use |
| --- | --- |
| `feat` | Add a user-facing or application feature. |
| `fix` | Fix a bug. |
| `docs` | Add or update documentation. |
| `chore` | Maintenance that does not change application behavior, such as tooling or housekeeping. |
| `refactor` | Restructure code without changing its intended behavior. |
| `test` | Add or update tests. |
| `build` | Change build scripts or dependencies. |
| `ci` | Change continuous integration or delivery configuration. |
| `style` | Make formatting-only changes that do not affect behavior. |
| `perf` | Improve performance. |

Examples:

```text
feat(api): add exam search endpoint
fix: handle an empty study plan
docs: explain local setup on Windows
chore: update development dependencies
```

For a breaking change, append `!` to the type or scope and explain the change in the commit body or a `BREAKING CHANGE:` footer:

```text
feat(api)!: rename the exam search endpoint

BREAKING CHANGE: clients must use /api/v2/exams/search.
```

The specification requires `feat` for new features and `fix` for bug fixes. Other types are commonly used but do not have a special versioning meaning by themselves.

## 10. Quick checks and recovery pointers

```sh
git status
git log --oneline --decorate -10
git branch --all
git remote -v
git diff
git diff --staged
```

- If `git push` says the remote does not exist, check the spelling with `git remote -v`; a common remote name is `origin`.
- If a push is rejected because the remote branch has new commits, fetch and integrate the changes deliberately. Do not use `git push --force` as a routine fix.
- If the wrong author appears on the latest, not-yet-pushed commit, configure the correct identity before making more commits. Amending a commit rewrites its hash; coordinate before rewriting commits others may already have fetched.
- Never commit passwords, API keys, access tokens, private keys, or `.env` files.

## Learning references

### Web documentation

- [Git reference documentation](https://git-scm.com/docs)
- [Pro Git book: Git basics](https://git-scm.com/book/en/v2/Getting-Started-Git-Basics)
- [Pro Git book: Branching and merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [GitHub Docs: Set up Git](https://docs.github.com/en/get-started/git-basics/set-up-git)
- [GitHub Docs: Set your Git username](https://docs.github.com/en/get-started/git-basics/setting-your-username-in-git)
- [GitHub Docs: Set your commit email address](https://docs.github.com/en/account-and-profile/how-tos/email-preferences/setting-your-commit-email-address)
- [GitHub Docs: Ignore files](https://docs.github.com/en/get-started/git-basics/ignoring-files)
- [GitHub Docs: Pull requests](https://docs.github.com/en/pull-requests)
- [GitHub Docs: Resolve a merge conflict using the command line](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/resolving-a-merge-conflict-using-the-command-line)
- [GitHub CLI manual: `gh auth login`](https://cli.github.com/manual/gh_auth_login)
- [GitHub CLI manual: `gh auth switch`](https://cli.github.com/manual/gh_auth_switch)
- [GitHub CLI manual: `gh pr create`](https://cli.github.com/manual/gh_pr_create)
- [Conventional Commits 1.0.0 specification](https://www.conventionalcommits.org/en/v1.0.0/)

### Video learning

These links open YouTube learning results or the publisher's channel; check the video description and publication date to ensure the commands match your installed Git/GitHub CLI version.

- [GitHub's official YouTube channel](https://www.youtube.com/@GitHub)
- [YouTube search: GitHub Git basics, branching, and pull requests](https://www.youtube.com/results?search_query=GitHub+Git+basics+branches+pull+requests)
- [YouTube search: Git merge conflicts](https://www.youtube.com/results?search_query=Git+merge+conflicts+tutorial)
- [YouTube search: Conventional Commits](https://www.youtube.com/results?search_query=Conventional+Commits+feat+fix+chore+tutorial)
