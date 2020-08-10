---
layout: post
title: Reverting multiple merged commits
---

Ok, let's talk a little bit about git based on an issue I had recently.

# Context of the issue #
For this specific project, the system's servers are managed by Chef, and Chef cookbooks versions running on the servers are controlled by environment files. Normally production would be running whatever its environment file says. So if a cookbook version is updated, production won't pick it up until its environment file is updated to do so. 
So I was asked to perform some changes on a cookbook, I did them, created a pull request, it was approved and I merged those changes which meant a new version of the cookbook was created. I repeated the process due to an additional change, so a second new cookbook version was created. The cookbook went from, let's say, version 0.0.1 to 0.0.3.

Then I updated a non-prod environment file to point to the cookbook version 0.0.3 and asked the customer to validate the changes and report back when I could point production to the same updated cookbook version. 

But, it turned out, the customer didn't want to pursue the changes anymore and asked to roll everything back to version 0.0.1, yeii!

# Actual fix - Long answer #
So, this was the output of ```git log```:

```bash
commit 8003cfd9f00e26a8d377c3c91811ac7d6f1017e2
Merge: a2652fe 62d5441

    Merged in branch XXX (Bumped cookbook to version 0.0.3)

commit 62d54412fa1493024cca5bd3bd39be63e8785426 (origin/branch, branch)

    Branch: commit XXX

commit afda38d89c6d78f5b1d6d07940ca026ca8a48ff8

    Branch: commit XXX

commit 81ecd3e2305a2888d43b629eca3e62f8d503f51e

    Branch: commit XXX

commit 902fd55d512f5dcb9b0187f09d1fa348abcd225a

    Branch: commit XXX

commit a2652fe7665275d9bb7b7ceb3ca043f24d2eee37
Merge: 702f590 4ae05d1

    Merged in branch XXX (Bumped cookbook to version 0.0.2)

commit 4ae05d1a700748d8c920bd9cc40a06b2fd95f488
    
    Branch: commit XXX
```
I had two merges done and commits between them of course. The best approach was to revert those commits. So I did the following:

* Revert the first merged commit:

```git revert --no-commit 8003cfd9f00e26a8d377c3c91811ac7d6f1017e2 -m 1```

*Note: This automatically removed the changes entered by the commits it merged*

I added the following flags to the command:
```bash
--no-commit: Usually git revert automatically creates some commits with commit log messages stating which commits were reverted. This flag applies the changes necessary to revert the
           named commits to your working tree and the index, but does not make the commits. In addition, when this option is used, your index does not have to match the HEAD commit.
           The revert is done against the beginning state of your index.

           This is useful when reverting more than one commits effect to your index in a row.

-m parent-number
           Usually you cannot revert a merge because you do not know which side of the merge should be considered the mainline. This option specifies the parent number (starting from
           1) of the mainline and allows revert to reverse the change relative to the specified parent.

           Reverting a merge commit declares that you will never want the tree changes brought in by the merge. As a result, later merges will only bring in tree changes introduced by
           commits that are not ancestors of the previously reverted merge. This may or may not be what you want.
```
* Revert the second merged commit

Once I reverted the first merged commit with parent number 1, the second merged commit became the parent number 1 in the history. So, in order to remove the second merged commit I needed to run the following:

```git revert --no-commit a2652fe7665275d9bb7b7ceb3ca043f24d2eee37 -m 1```

After that I validated I had my local to the state I wanted it which was cookbook version 0.0.1 and proceeded to commit the changes with a few additional change, I actually bumped the cookbook version to 0.0.4 but kept the cookbook state as it was in the beginning, version 0.0.1. (This move has to do with chef cookbook versioning which I don't get into details in here).

# Actual fix - Short answer #

1. Run ```git log``` to check commit hashes
2. Identify the hashes of the merged commits
3. Revert the first merged commit with ```git revert --no-commit 8003cfd9f00e26a8d377c3c91811ac7d6f1017e2 -m 1```
4. Revert the second merged commit with ```git revert --no-commit a2652fe7665275d9bb7b7ceb3ca043f24d2eee37 -m 1```
5. Commit the changes

## Conclusion ##
First of all, to be honest I don't think git documentation or even git commands CLI help are that clear. Secondly, the whole purpose of this post was only to show how to fix the mess I got into with these merges and commits and not about chef cookbook versioning. Mentioning that was only to give a little bit of context about how everything started.

## References ##
[How to revert multuple git commits](https://stackoverflow.com/questions/1463340/how-to-revert-multiple-git-commits)

[Reverting a series of pushed merges and commits in git](https://stackoverflow.com/questions/18082830/reverting-a-series-of-pushed-merges-and-commits-in-git-without-rewriting-histor)