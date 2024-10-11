---
layout: post
title: Git Tips for Beginners
date: 2013-10-17 22:02:11.000000000 -07:00
redirect_from:
  - /blog/git-tips-for-beginners-interested-in-open-source
---

As I'm nearing the end of the Google Summer of Code (GSoC) program with the
[Tahoe-LAFS community](https://tahoe-lafs.org/trac/tahoe-lafs), I can say it has
been an amazing learning experience. I learned a lot and was able to solve some
important bugs along the way.

As with any learning experience, there were plenty of frustrations along the
way. Sadly, most of these frustration were caused by git and not the problems I
was trying to solve. Before Summer of Code, I had only used git for personal
projects and my knowledge was limited. The learning curve for git is extremely
high, and it can be discouraging to spend half of your time fighting with git
instead of hacking on your project. Additionally, there are some conventions
beginners are not aware of when they start working on an open source project.
Here are some of the things I wish I knew before I started hacking on
Tahoe-LAFS.

### Clone with SSH

If you fork a repository on Github, clone the repository with the `ssh` link.
Otherwise, you will have to authenticate with Github each time you push a
branch. For information on authenticating with Github via ssh keys, check out
[Github's detailed guide](https://help.github.com/articles/generating-ssh-keys).

### Never Work on Master

One of the most important rules to remember is: never commit to the master
branch. Master is pristine and can only be touched after multiple people have
signed off on the code. Since we cannot commit code to master, all work must be
done on a branch, which can be thought of as another copy of the code base. In
order to create a branch based on master, use:

    git branch <your_branch_name> master

Or if you are currently on master:

    git branch <your_branch_name>

Or if you want to create a branch and check it out in the same command:

    git checkout -b <your_branch_name>

### Updating Master to Reflect Upstream

Since git is a decentralized version control system, there is no obvious way to
sync your master branch with the project's master branch. To do this, we first
need to add a new remote to the project's main repository.

    git remote add upstream <link_to_main_repo>

Now we need to retrieve information about this new remote repository.

    git fetch upstream

Finally, we can update the local master branch to reflect upstream's master
branch.

    git checkout master
    git merge upstream/master

Now our master branch is identical to the master branch on upstream.

**Edit:** In general, you shouldn't have to worry about writing your patch on
the latest version of upstream. Anyone who can commit to the project's master
branch should be able to merge your branch if it is based on a recent version of
upstream. Thanks to [Jed Brown](http://59a2.org/research/) for suggesting this
change.

### Fixing Commits

Oh no! After you committed that awesome patch, you noticed that you made a
spelling mistake in your code. Even worse, there is also a spelling mistake in
your commit message. Thankfully, both of these errors can easily be fixed. To
fix the commit message, use:

    git commit --amend

This will bring up the previous commit message. Fix the spelling mistake, save,
and exit to recommit the patch.

Now to fix the spelling error in the patch. Fix the spelling error and make an
additional commit. Don't worry about the commit message. Then type
`git rebase -i HEAD~2`:

    pick 40e4369 My sweet bug-busting commit
    pick ec48bcf My spelling fix
    ...
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell

`HEAD~2` means we want to rebase the two commits before our current position.
Rebase is a powerful tool, and a proper explanation is outside the scope of this
post. But, for our fix we want to use `fixup` on `ec48bcf`.

    pick 40e4369 My sweet bug-busting commit
    f    ec48bcf My spelling fix

Save and exit. Now a simple mistake is not breaking an otherwise useful commit.
It's important to note, that if the branch has already been pushed to Github,
you will have to do a forced push to sync the branch.

    git push -f origin <my_branch>

As a rule of thumb, if someone else is using your branch in any capacity, you
should not force push the branch. Instead of rebasing to combine commits, leave
the separate commit as is.

### Check for Trailing Whitespace

This doesn't have to do with git, but it is still worth knowing. Before making
any commits, it's good practice to run `git diff` to ensure you aren't
accidently committing any debug code. Additionally, check for any newlines,
tabs, or spaces you may have added by accident, especially trailing whitespace.
While trailing whitespace isn't the end of the world,
[it annoys a lot of people](http://programmers.stackexchange.com/questions/121555/why-is-trailing-whitespace-a-big-deal).
If you use Sublime Text, I recommend the
[Trailing Spaces](https://github.com/SublimeText/TrailingSpaces) plugin.

Thanks for reading!

_9/10/13: The initial version of this post refered to upstream as trunk._
