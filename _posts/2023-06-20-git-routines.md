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
{% highlight terminal %}
git status
git diff
git diff <filename>
git add --all # Stage all files
git add <filename>	# Stage a file, ready to commit
git reset # Unstage all files
git reset <filename> # Unstage one file
git restore <filename> # Discard changes of the file

git commit -m "Post update" 
git push -u origin <your-branch> 
{% endhighlight %}

<a href="#">Git Tools - Branch</a>
{% highlight terminal %}
git checkout -b <your-branch> # new local branch
git branch -d <your-branch>   # delete the branch
{% endhighlight %}

<a href="https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging">Git Tools - Interactive staging</a> 

We can stage patches under interactive staging mode.
{% highlight terminal %}
git add -i # Interactive Staging
git add -p # or --patch to start partial staging
git reset -p
{% endhighlight %}


<a href="https://www.cloudbees.com/blog/git-undo-commit">Git Tools - Time Travel</a>

Better undoing mistakes before commit:
{% highlight terminal %}
git stash
git checkout -- <filename> # Discard file changes permanently
git reset --hard # Discard all changes permanently
{% endhighlight %}

Undo Committed changes:
{% highlight terminal %}
git reflog # To check the commit id.
git reset --hard <commit-id> # Discard all changes permanently

git rm -rf --cached . # remove the cache
git reset --hard HEAD # reset to the latest commit of the current branch
{% endhighlight %}

Undoing changes in a shared repo:
{% highlight terminal %}
git revert <commit-id> # A new commit that reverts the changes of commit-id.
{% endhighlight %}


<a href="https://git-scm.com/book/en/v2/Git-Tools-Submodules">Git Tools - Submodules</a>
{% highlight terminal %}
git clone --recurse-submodules -j8  target_git@github.com
{% endhighlight %}

If we cloned the repo without recursively pull submodules, try the following: 
{% highlight terminal %}
cd <repo-dir>
git submodule update --init --recursive
{% endhighlight %}

To individually update a specific submodule:
{% highlight terminal %}
cd <submodule-dir>
git pull --recurse-submodules
{% endhighlight %}

<a href="https://linuxhint.com/pull-git-submodules-after-cloning-project-from-github/">Submodule can be added</a> to your current repo by:
{% highlight terminal %}
cd <repo-dir>
git submodule add <git_url_to_the_submodule> <submodule_folder_name>
{% endhighlight %}

<a href="https://stackoverflow.com/questions/11168141/find-which-commit-is-currently-checked-out-in-git">Git Tools - commit information</a> 

variations:
{% highlight terminal %}
git show [--outline -s]
git reflog 
git log -1 [--outline] # show the last commit info
git status
{% endhighlight %}

Launch gitk graphic display:
{% highlight terminal %}
git bisect visualize
git bisect view  # shorter, means same thing
{% endhighlight %}

<a href="https://www.codeblocq.com/2016/02/Stash-your-changes-before-switching-branch/">Git Tools - Stash</a>:
{% highlight terminal %}
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
{% endhighlight %}

<a href="https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings?platform=linux">Git Tools - Eol configuration</a>

Global settings for line endings:

{% highlight terminal %}
git config --global core.autocrlf input
# Configure Git to ensure line endings in files you checkout are correct for Linux

git config --get core.autocrlf # Check the value
{% endhighlight %}

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