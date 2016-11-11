# merge-view

Ever been confused about the BASE, LOCAL and REMOTE sides of a merge or rebase conflict? Do they seem to flip in confusing ways? Read on to find out what's going on!

*Side Note: This is not an opinion piece on the virtues of merge and rebase - both have their place, and often in the same repository.*

This repository is set up with a simple example to demonstrate the difference. Screenshots
have been created using the popular merge tool Beyond Compare 4 Pro, but most merge tools
present similar views.

If you clone this repository, it will be all ready for you to try the merge or rebase.

It was created like this:

1. Commit A: `file.txt` is added in the initial commit on the master branch. `file.txt` contains `Cheese`.
2. Commit B: A new branch is created called `my-feature`. On this branch `file.txt` is modified to `I moved the cheese`.
3. Commit C: Back on master, `file.txt` is also modified to `Someone else moved the cheese`. This represents work done by another developer and added to `master` *after* our original developer had branched from `master`.

The changes in B and C are conflicting changes because the same line of "code" has been changed in each case.

At some point (and you do this at least daily don't you!!!) the developer will want to incorporate these latest changes into his branch. There are two main ways to do this, the merge and the rebase.

Now we can examine the often confusing difference in how merge and rebase conflicts are presented in popular tools.
We will explore two common scenarios - in both cases, playing the role of a developer who wishes to get changes from the `master` branch into his `my-feature` branch.
First, let's do it by merging:

## Merging

Here's the intended result of the merge:

![Merge Diagram][merge-diagram]

We will create new commit M which is a child of commits B and C. The `my-feature` branch is updated to point to this new commit. 

Let's merge with the following commands:

1. `git checkout my-feature`
2. `git merge master`

A conflict occurs and you see the something similar to the following (it may differ slightly depending on your settings):

    Auto-merging file.txt
    CONFLICT (content): Merge conflict in file.txt
    Recorded preimage for 'file.txt'
    Automatic merge failed; fix conflicts and then commit the result.

    On branch my-feature
    You have unmerged paths.
    (fix conflicts and run "git commit")

    Unmerged paths:
    (use "git add <file>..." to mark resolution)

            both modified:   file.txt

    no changes added to commit (use "git add" and/or "git commit -a")

When we resolve with `git mergetool` we see this:

![Merge Tool Screenshot][merge-tool]

We can see that:

- The LOCAL changes are the developer's change "I moved the cheese".
- The BASE is the original "Cheese"
- The REMOTE is the master's copy "Someone else moved the cheese".

[merge-diagram]: https://github.com/james-world/merge-view/raw/master/merge-diagram.png "Merge Diagram"
[merge-tool]: https://github.com/james-world/merge-view/raw/master/merge-tool.png "Merge Tool Screenshot"

This all seems quite logical, but now let's look at the alternative to merging.

## Rebasing

In a rebase, the commits on a branch are effectively "moved" on to a different parent commit. Typically this is a more recent commit from a source branch. The purpose is
to incorporate changes since the branch was created, and have all the branch work appear as though it came afterwards. This creates a cleaner more logical history, which
some people prefer.

The commits aren't really moved though - since commits are immutable and cannot be changed. Instead a new commit is created, and the branch is updated to point to this new commit. The old
commit is orphaned and eventually garbage collected.  

Here's the intended result of the rebase that will be performed.:

![Rebase Diagram][rebase-diagram]

We will create new commit R which is a child of commit C only. This commit is created by applying the work done by
the developer in the `my-feature` branch on top of master branch. Afterwards, it will be as though he branched from commit C
in the first place. The `my-feature` branch will be updated to point at the new commit. 

Let's rebase with the following commands:

1. `git checkout my-feature`
2. `git rebase master`

A conflict occurs and you see the something similar to the following (it may differ slightly depending on your settings):

    First, rewinding head to replay your work on top of it...
    Applying: I move the Cheese
    Using index info to reconstruct a base tree...
    M       file.txt
    Falling back to patching base and 3-way merge...
    Auto-merging file.txt
    CONFLICT (content): Merge conflict in file.txt
    error: Failed to merge in the changes.
    Patch failed at 0001 I move the Cheese
    The copy of the patch that failed is found in: .git/rebase-apply/patch

    When you have resolved this problem, run "git rebase --continue".
    If you prefer to skip this patch, run "git rebase --skip" instead.
    To check out the original branch and stop rebasing, run "git rebase --abort".

When we resolve with `git mergetool` we see this:

![Rebase Tool Screenshot][rebase-tool]

We can see that:

- The LOCAL change is the master's change "Someone else moved the cheese".
- The BASE is the original "Cheese"
- The REMOTE is the developer's change "I moved the cheese".

It's the reverse of the merge situation! Why is that? It's because of the way a rebase is applied - in fact the output above explains what is happening:

1. The target commit "C" is checked out. This is now the LOCAL branch and the start point for the rebase. (The "rewind".)
2. Git creates a patch for the source commit "B" as applied to commit "A" and then "replays" against commit "C" in order to create commit "R"

From the merge tool's perspective, this patch is the "remote" work coming in from another branch. This is why the LOCAL and REMOTE
sides in the mergetool are the opposite of what you would expect.

Hopefully this makes things a bit clearer, but feel free to clone the repo and try it yourself - especially if you use a different tool.

[rebase-diagram]: https://github.com/james-world/merge-view/raw/master/rebase-diagram.png "Rebase Diagram"
[rebase-tool]: https://github.com/james-world/merge-view/raw/master/rebase-tool.png "Rebase Tool Screenshot"


