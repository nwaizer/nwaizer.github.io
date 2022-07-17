---
layout: post
title: Lessons learned in git workflow
tags: [git]
---

Here I want to leave a note as to how I can save my commit history as well as present a single commit for PR reason.

1. Fork the awesome repo and keep the master branch in sync.

2. create the feature "work" branch here I save all commit history from the tip I started the feature development

3. create another PR branch with the sum of all the commits. this branch is a squash of all the "work" branch

 ![initial](/assets/posts/git_workflow/initial_git_state.jpg)

This PR branch will merge --squash from the work branch.

I now want to solve this problem:

Since starting my feature branch, the original repository branch, has advanced and new content was submitted. In the drawing below the HEAD commit moved to commit B. Now my PR asks me tor rebase my PR branch.

My aim is to keep my added feature, which I wrote in my feature branch, but change the rest of the content that is in the rest of the repository. In other words I want to rebase the commit under my commit. Here is how to achieve this.

Then to keep this PR in sync with the updates in master we do.

git chekcout master
git fetch upstream
git rebase upstream/master
git checkout <feature_branch>
git rebase master

![master_advanced](/assets/posts/git_workflow/master_advanced.jpg)

Now to publish this to the fork repo in a new branch:

git push -u origin <feature_branch>
Now I add more commits to my work branch:

![add_local_commit](/assets/posts/git_workflow/adding_local_commit.jpg)

I keep adding more commits in my feature branch
When I want I merge --squash to the PR branch, these changes go on-top of the master from the original repository.
if for some reason if you have done git push -f , then you will have to do git push -f
And then again in case the master has moved on to commit K:

![master_advanced_again](/assets/posts/git_workflow/master_advanced_again.jpg)