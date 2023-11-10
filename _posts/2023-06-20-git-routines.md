---
layout: post
title:  Git Routine
date:   2023-06-20 10:00:00
description: Routine git commands
tags: notes code
categories: note-posts
---
#### Useful git commands

##### Command cheatsheet
```bash
git clone --recurse-submodules -j8 git@github.com:example/target-repo.git
git status
git diff
git diff <filename>
git add --all # Stage all files
git add <filename>	# Stage a file, ready to commit
git reset # Unstage all files
git reset <filename> # Unstage one file
git reset HEAD <submodule-name> # Reset the submodule head
git restore <filename> # Discard changes of the file

git commit -m "Post update" 
git push -u origin <your-branch> 
```

Discard local changes and force to pull the remote branch:
```bash
# Really the ideal way to do this is to not use pull at all, but instead fetch and reset:
git fetch origin <master-branch>
# force the state of the working directory to a state matching that of a particular commit. 
git reset --hard FETCH_HEAD
# removes files which are not tracked by git
# the -df flags tell it to remove directories (-d) and actually do the removal (-f)
git clean -df
```

<a href="#">Git Tools - Branch</a>
```bash
git checkout -b <your-branch> # new local branch
git branch -d <your-branch>   # delete the branch
```

<a href="https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging">Git Tools - Interactive staging</a> 

We can stage patches under interactive staging mode.
```bash
git add -i # Interactive Staging
git add -p # or --patch to start partial staging
git reset -p
```

<a href="https://git-scm.com/docs/git-merge">Git Tools - Merge</a>
Caution: before merging, make sure there is no non-trivial uncommitted changes.

```bash
# Example:
#       A---B---C topic
#      /
# D---E---F---G master
```

```bash
git checkout master
git merge topic

# Result:
#       A---B---C topic
#      /         \
# D---E---F---G---H master
```

Better practice:

```bash
git merge --no-commit [-s ort] topic
# --no-commit: permform merge but stop before creating a merge commit.
# -s: merge strategy option. By default, its `ort`.

git commit -m "msg"
# -m: message about the merge
```

What should we do if we run into conflicts:

1. Abort the merge to start over again?
	
	```bash
	# Abort the merge process and *try* to reconstruct the pre-merge state:
	git merge --abort
	```

2. Resolve the conflicts with tools

	Use your favorate editor to resolve the changes.

	`HEAD`: current branch head

	`MERGED_HEAD`: other branch head

	```bash
	# Option 1: use graphical mergetool
	git mergetool

	# Option 2: highlight diff
	git diff

	git log --merge -p <path> # show diffs first for the HEAD version and then MERGED_HEAD version

	git show :1:<filename> # show the common ancestor
	git show :2:<filename> # show the HEAD version of the file
	git show :3:<filename> # show the MERGED_HEAD version

	```


<a href="https://www.cloudbees.com/blog/git-undo-commit">Git Tools - Time Travel</a>

Better undoing mistakes before commit:
```bash
git stash
git checkout -- <filename> # Discard file changes permanently
git reset --hard # Discard all changes permanently
```

Undo `git add` files:

Concepts:
- Working directory: where you make changes to your code;
- Staging area: intermedia step where you prepare changes for commit.

```bash
# option 1:
git reset <file> # remove a single file from staging area, i.e. undo git add
git reset # remove all files from staging area, changes are still in working directory

# option 2:
git rm --cached <file> # remove a single file from the staging area

# Caution:
git rm <file> # remove the changes from both working directory and staging area.
```

Undo Committed changes:
```bash
git reflog # To check the commit id.
git reset --hard <commit-id> # Discard all changes permanently

git rm -rf --cached . # remove the cache
git reset --hard HEAD # reset to the latest commit of the current branch
```

Undoing changes in a shared repo:
```bash
git revert <commit-id> # A new commit that reverts the changes of commit-id.
```

<a href="https://git-scm.com/book/en/v2/Git-Tools-Submodules">Git Tools - Submodules</a>
```bash
git clone --recurse-submodules -j8  target_git@github.com
```

If we cloned the repo without recursively pull submodules, try the following: 
```bash
cd <repo-dir>
git submodule update --init --recursive
```

To individually update a specific submodule:
```bash
cd <submodule-dir>
git pull --recurse-submodules
```

<a href="https://linuxhint.com/pull-git-submodules-after-cloning-project-from-github/">Submodule can be added</a> to your current repo by:
```bash
cd <repo-dir>
git submodule add <git_url_to_the_submodule> <submodule_folder_name>
```

<a href="https://stackoverflow.com/questions/11168141/find-which-commit-is-currently-checked-out-in-git">Git Tools - commit information</a> 

variations:
```bash
git show [--outline -s]
git reflog 
git log -1 [--outline] # show the last commit info
git status
```

Launch gitk graphic display:
```bash
git bisect visualize
git bisect view  # shorter, means same thing
```

<a href="https://www.codeblocq.com/2016/02/Stash-your-changes-before-switching-branch/">Git Tools - Stash</a>:
```bash
git stash save "changes on the current-branch"
git stash pop # unstash the changes of the top stash
git stash list # list the stash stack
git stash pop "stash@{1}" # unstash a specific stash
git stash apply # apply the top stash
git stash drop # drop the top stash
git stash show # Show the files in the most recent stash
git stash show -p # show the changes in the most recent stash
git stash show -p stash@{1} # show the changes of the specified stash
git stash show -p 1 # simplified version of the above comment
```

<a href="#">Git Tools - working tree</a>
TODO...

<a href="https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings?platform=linux">Git Tools - Eol configuration</a>

Global settings for line endings:

```bash
git config --global core.autocrlf input
# Configure Git to ensure line endings in files you checkout are correct for Linux

git config --get core.autocrlf # Check the value
```

Per-repository settings:

You can configure a `.gitattributes` file to manage how Git reads line endings in a specific repository. The `.gitattributes` file must be created in the root of the repository and committed like any other file.

An example of the `.gitattributes` file content:
```
# Set the default behavior, in case people don't have core.autocrlf set.
* text=auto

# Explicitly declare text files you want to always be normalized and converted
# to native line endings on checkout.
*.c text
*.h text

# Declare files that will always have CRLF line endings on checkout.
*.sln text eol=crlf

# Denote all files that are truly binary and should not be modified.
*.png binary
*.jpg binary
```

Reference:
<a href="https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration">Customizing your git configuration</a>