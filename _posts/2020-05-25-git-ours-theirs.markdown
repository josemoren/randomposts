---
layout: post
title:  "git checkout ours and theirs"
date:   2020-05-25 18:53:25 +0200
categories: git
---

The documentation referring to [ours and theirs](https://git-scm.com/docs/git-checkout#Documentation/git-checkout.txt---ours) in git is not the easiest to understand.


But in essence it seems to come down to a couple of things:
- **Ours** refers to the target of the changes
- **Theirs** refers to the changes being applied on top


### Merge

When doing a merge the target is the branch we are merging into. The changes being applied are the ones in the branch we are merging from. For example:

{% highlight Shell Session %}
$ git checkout master
master$ git merge feature1
{% endhighlight %}

<img width="255" alt="merge_sample1" src="https://user-images.githubusercontent.com/33334531/82841253-b0d20d00-9ed5-11ea-80e7-548d3fe5ab13.png">

&nbsp;
&nbsp;
Let's imagine a conflict arises in this merge. The following checkout would get the file `path/to/file` from `feature1`

{% highlight Shell Session %}
$ git checkout --theirs path/to/file
{% endhighlight %}


### Rebase

When doing a rebase the target is the branch we are rebasing onto.

{% highlight Shell Session %}
$ git checkout feature1
feature1$ git rebase master
{% endhighlight %}

<img width="250" alt="rebase sample" src="https://user-images.githubusercontent.com/33334531/82841426-49688d00-9ed6-11ea-94a6-20de4fb168ad.png">

So, if the current branch is `feature1` and we apply a rebase on master all the changes in `feature1` that are not in `master` will be applied on top of master. Therefore the "target" is really the master branch, even if the current branch is `feature1`.

{% highlight Shell Session %}
$ git checkout --theirs path/to/file
{% endhighlight %}

This checkout would get the file `path/to/file` from `feature1` again. Even if the current branch is different.
