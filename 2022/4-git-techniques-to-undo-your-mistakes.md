Oops! You just commited a secret API key! Or maybe you want to revert the changes of an earlier commit. Whatever it may be, this blog will highlight some common VCS mistakes and how to *Git* things fixed.

**Note**: the vast majority of these tips apply only **before** you have pushed your changes to a remote repository. Rewriting the history of a remote repo is scary and should be avoided unless you know what you're doing! If you really want to replace a remote with local state, push with the `--force` flag.

## 1. Unstage

Say you want to remove something from the staging area without modifying the file itself.

- Check what files are currently in the staging area: `git diff --name-only --cached`
- Unstage a specific file: `git reset HEAD <file>`
- Or unstage everything: `git reset HEAD`
  
## 2. Uncommit

- Remove your last commit from the history without changing the working tree (your files): `git reset HEAD^`
- Do the same, but keep files in the staging area: `git reset --soft HEAD^`
- Remove the commit **and** throw away **all** changes since the previous commit (be careful with this one): `git reset --hard HEAD^`

## 3. Revert a Commit

Some commit in the history has become problematic, so create a new "revert commit" to undo changes:

- Find the `ref` of the commit you want to revert: `git log`
- Create a new revert commit: `git revert <ref>`

Keeps the original commit intact, and is safe to use for commits that have been pushed to a remote repo.

## 4. Amend a Commit

- You made a typo in a commit message, amend it without changing files: `git commit --amend -m "New message"`
- Change the files of a commit without changing the message: `git commit --amend --no-edit`

## 5. Discard

You have modified files, and want to throw everything out and go back to a previous commit.

### Option A - Stash your Changes (Recommended)

- Stash all your current changes (can also be used for a specific file): `git stash push`
- Also stash untracked files for a clean repo: `git stash --all`
- You can restore your stashed changes whenever you want: `git stash pop`

### Option B - Hard Reset (Not Recommended)

- Reset your working tree to the state of a commit (**changes will be lost**): `git reset --hard <commit>`
- Remove untracked files for a clean repo (`-x` for ignored files as well): `git clean -fd`
- If you need to recover changes, use `git reflog show HEAD` and reset back to the `ref`, though I find `stash` easier to work with.

## 6. Rebase

The swiss army knife of git. Can basically do steps 2-4 all in one go. Re-write your history by editing, moving and merging commits. Again, make sure to only use this for commits that have yet to be pushed to a remote repo, else you might get some angry coworkers.

- Interactively rebase everything that hasn't been pushed to a remote: `git rebase -i`
- Or rebase the last `x` commits: `git rebase -i HEAD~<x>`

Interactive rebase reference:

```md
Commands:
p, pick = use commit
r, reword = use commit, but edit the commit message
e, edit = use commit, but stop for amending
s, squash = use commit, but meld into previous commit
f, fixup = like "squash", but discard this commit's log message
x, exec = run command (the rest of the line) using shell
d, drop = remove commit
```

This is a very powerful tool - I'd recommend doing some [further reading](https://cityofaustin.github.io/ctm-dev-workflow/rebase.html).

## Bonus - .gitignore

Create a `.gitignore` to specify certain files that you don't want added to the staging area (importantly, environment variables and API keys). See [this](https://github.com/github/gitignore) list of templates for different projects.

## What to do Next

- Set some of these commands as an alias so you don't need to remember everything. See my [`.gitconfig`](https://github.com/Tim-W-James/.dotfiles/blob/main/.gitconfig#L10).
- If you use VSCode, install some Git related extensions. For example, I like to use [Checkpoints](https://marketplace.visualstudio.com/items?itemName=micnil.vscode-checkpoints) to create temporary local WIP commits to save my work as I go. For a complete list, see my [recommended VSCode extensions](https://dev.to/timwjames/the-ultimate-list-of-vscode-extensions-57ih#git).
- Install some CLI tools. For example, I like to use [`diff-so-fancy`](https://github.com/so-fancy/diff-so-fancy) for viewing diffs. For a complete list, see my [recommended CLI tools](https://dev.to/timwjames/command-line-tools-for-productive-developers-pph#git).

Feel free to comment with any additional tips!
