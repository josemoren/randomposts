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

### Internals of conflicts

Let's imagine we have the following commits:

<img width="319" alt="graph1" src="https://user-images.githubusercontent.com/33334531/83038178-f3622980-a03c-11ea-8f56-e6c12bd74471.png">

`B` and `C` have `A` as their parent. In all three we have the file `sample.txt`. Both B and C have modified the same line in the text file.

Now we do a rebase of `branch1` onto `master`. A conflict arises.

{% highlight Shell Session %}
$ git status

rebase in progress; onto aa39bd4
You are currently rebasing branch 'branch1' on 'aa39bd4'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git restore --staged <file>..." to unstage)
  (use "git add <file>..." to mark resolution)
	both modified:   sample.txt
  
{% endhighlight %}

This output tells us that we have a `rebase in progress; onto aa39bd4`. `aa39bd4` is the commit on the master branch which is the target of the changes.

The section `Unmerged paths:` shows the files that have been modified in both sides of the rebase and that require a manual resolution. In this case the file `sample.txt`.

{% highlight Shell Session %}
$ git diff # This is the diff between B (ours) and C (theirs)

diff --cc sample.txt
index 017dd08,ddcd36d..0000000
--- a/sample.txt
+++ b/sample.txt
@@@ -1,1 -1,1 +1,5 @@@
++<<<<<<< ours
 +sample from master
++=======
+ sample from branch
++>>>>>>> theirs

{% endhighlight %}

{% highlight Shell Session %}
➜ git show head sample.txt # This is the diff between B and A

commit aa39bd494895f8ffc26ae4d8bfa288b4cbe75b66 (HEAD, master)
Author: Jose M <jj@mo.com>
Date:   Wed May 27 15:32:58 2020 +0200

    from master

diff --git a/sample.txt b/sample.txt
index 466e7c1..017dd08 100644
--- a/sample.txt
+++ b/sample.txt
@@ -1 +1 @@
-sample
+sample from master

{% endhighlight %}

{% highlight Shell Session %}
➜ git show rebase_head sample.txt # This is the diff between C and A

commit f569b8c51faf116289d5de844e6dfa90bcfee956 (branch1)
Author: Jose M <jj@mo.com>
Date:   Wed May 27 15:32:26 2020 +0200

    texto

diff --git a/sample.txt b/sample.txt
index 466e7c1..ddcd36d 100644
--- a/sample.txt
+++ b/sample.txt
@@ -1 +1 @@
-sample
+sample from branch

{% endhighlight %}

But where does this list of unmerged paths come from? It is stored in the `index`.


{% highlight Shell Session %}

 ➜ git ls-files -s
100644 9773bff9f5b3fd28bb79d4fd4eb0a40639e380a0 0	hola.txt
100644 466e7c193eb3f9b7d20810115c08f6a0ee2209b5 1	sample.txt
100644 017dd0869e506ff7dfef3b1c9d0a5ed5eaf39900 2	sample.txt
100644 ddcd36d751de2b23dc9771b9728e0defa6dbe7a6 3	sample.txt

{% endhighlight %}

The ls-files command here shows the content of our index.
Files without conflicts have a 0 after the hash of the blob. In our example `hola.txt`. 
Files with a conflict have 3 blobs present marked with 1, 2 and 3.

- 1: this is the base of the conflict (file as a blob in A).
- 2: this is the ours version of the file (file as a blob in B).
- 3: this is the theris version of the file (file as a blob in C).

{% highlight Shell Session %}

 ➜ git show 466e7c193eb3f9b7d20810115c08f6a0ee2209b5

sample

 ➜ git show 017dd0869e506ff7dfef3b1c9d0a5ed5eaf39900

sample from master

 ➜ git show ddcd36d751de2b23dc9771b9728e0defa6dbe7a6

sample from branch

{% endhighlight %}
