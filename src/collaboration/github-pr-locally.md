# Dealing with Github Pull Requests locally

## Table of Contents

  * [Pull Requests are simple refs](#pull-requests-are-simple-refs)
  * [Dealing with Pull Requests locally](#dealing-with-pull-requests-locally)
    * [The ad-hoc way](#the-ad-hoc-way)
    * [The set it up once forget about it way per project](#the-set-it-up-once-forget-about-it-way-per-project)
    * [The set it up once forget about it way system wide](#the-set-it-up-once-forget-about-it-way-system-wide)
  * [Conclusion](#conclusion)


As an open-source contributor, one will eventually need to do code contributions or code reviews.
As of today, GitHub is the platform where most open-source projects live. It's model is based on Pull Requests one fills against the main repository.

One common pratice is to review others' contributions or integrate others' contributions within one's own. In order to do so, one needs to locally retrieve the content of those pull requests that live on GitHub servers locally. While not hard - as its all git based - it is not obvious at first how one should proceed.

This article is here to clarify that.

## Pull Requests are simple refs

When one locally creates a new branch, one creates a new (humanly-referenceable) named reference (refs) to a commit object (its SHA1).

```
#> ls -l .git/refs/heads
total 4
-rw-rw-r--. 1 spredzy spredzy 41 Oct 14 15:27 devel

#> git checkout -b mynewbranch
Switched to a new branch 'mynewbranch'

#> ls -l .git/refs/heads
total 8
-rw-rw-r--. 1 spredzy spredzy 41 Oct 14 15:27 devel
-rw-rw-r--. 1 spredzy spredzy 41 Oct 14 15:47 mynewbranch

#> cat .git/refs/heads/mynewbranch
500f056b4ec37bd987c41db7e7a043526acd5d02

$> git cat-file -t 500f056b4ec37bd987c41db7e7a043526acd5d02
commit

#> git cat-file -p 500f056b4ec37bd987c41db7e7a043526acd5d02
tree dac0f7288e6eb1f4f79fc4c6ce755dff0cebc554
parent 2fe2a12440b9b912180373d0b7679ed20b30c334
author Martin Nečas <necas.marty@gmail.com> 1571054233 +0200
committer ansibot <ansibot@users.noreply.github.com> 1571054233 -0400

ovirt_vm correct graphical port (#63379)
```

When one opens a pull request against a project in GitHub, this is exactly what GitHub does in the background.
It creates a new refs to the HEAD commit that has been pushed. This reference is stored under `refs/pull/<PULL-ID>/head`.
This is a simple refs, there is nothing magic about it, and you can interact with it the same way you interact with any other refs.



## Dealing with Pull Requests locally

For the following example, we will assume that we are working with the `ansible/ansible` git repository on GitHub located at https://github.com/ansible/ansible.
We are putting ourselves in the shoes of a contributor that does both code contributions and code reviews. As a good practice we have forked the `ansible/ansible` repo, to our own namespace. (ie. `Spredzy/ansible`).

The setup to highlight this article will go as follow:

```
#> git clone git@github.com:Spredzy/ansible.git
#> cd ansible
#> git remote -v
origin  git@github.com:Spredzy/ansible.git (fetch)
origin  git@github.com:Spredzy/ansible.git (push)
#> git remote add upstream https://github.com/ansible/ansible.git
#> git remote -v
origin  git@github.com:Spredzy/ansible.git (fetch)
origin  git@github.com:Spredzy/ansible.git (push)
upstream        https://github.com/ansible/ansible (fetch)
upstream        https://github.com/ansible/ansible (push)
```

So we end up with our repository configured with two remotes. Namely, `origin` our fork, and `upstream` the actual `ansible/ansible` repository (where the bulk of the pull requests are filled against). If you are not familiar with what a `remote` is please check this [article](src/terminology/remotes.md)


### The ad-hoc way

As stated in [Pull Requests are simple refs](#pull-requests-are-simple-refs), GitHub pull requests **are** simple refs. Ultimately, this means that their content can be fetched just as with any other commit/branch with the following syntax:

```
#> git fetch <remote> <src-ref>:<local-ref>
```

In our case, it means that to retrieve PR#4242 (https://github.com/ansible/ansible/pull/42424), one would do:

```
#> git fetch upstream refs/pull/42424/head:localcopy42424
remote: Enumerating objects: 72, done.
remote: Counting objects: 100% (72/72), done.
remote: Total 97 (delta 72), reused 72 (delta 72), pack-reused 25
Unpacking objects: 100% (97/97), done.
From https://github.com/ansible/ansible
 * [new ref]               refs/pull/42424/head -> localcopy42424

#> git checkout localcopy42424
Switched to branch 'localcopy42424'
```

Your working directory has now the content of the PR#42424 - One is free to review it on its favorite `$EDITOR`, run tests or simply deploy it to see if it does what it is supposed to.


### The set it up once forget about it way per project

While the process above works properly, if one is often pulling pull requests for review, it can become cumbersome to run this command for every new pull request one wants to review.

One could leverage her `.git/config` file to be able to fetch all the pull requests references available. Note, it says pull requests references and not pull requests content. Only the references will be pulled and sync with your local git database, the content won't until you actually checkout the new reference locally.


To do so, one should edit its configuration file (via cli or by editing `.git/config` file directly), as follow:

```
#> git config --add remote.upstream.fetch '+refs/pull/*/head:refs/remotes/upstream/pr/*'
#> git fetch upstream
[...]
 * [new ref]               refs/pull/6867/head  -> upstream/pr/6867
 * [new ref]               refs/pull/6869/head  -> upstream/pr/6869
 * [new ref]               refs/pull/687/head   -> upstream/pr/687
 * [new ref]               refs/pull/6870/head  -> upstream/pr/6870
 * [new ref]               refs/pull/6871/head  -> upstream/pr/6871
 * [new ref]               refs/pull/6872/head  -> upstream/pr/6872
 * [new ref]               refs/pull/6873/head  -> upstream/pr/6873
 * [new ref]               refs/pull/6876/head  -> upstream/pr/6876
 * [new ref]               refs/pull/6877/head  -> upstream/pr/6877
 * [new ref]               refs/pull/6878/head  -> upstream/pr/6878
 * [new ref]               refs/pull/6880/head  -> upstream/pr/6880
 * [new ref]               refs/pull/6883/head  -> upstream/pr/6883
[...]
```

Now, one can simply run the following command to locally checkout a pull request

```
#> git checkout -b localcopy42424 upstream/pr/42424
```

To fetch the latest pull requests references, simply run `git fetch upstream` and the references will be synced with your local database.

### The set it up once forget about it way system wide

Here we take a convention over configuration approach. Let's assume, when one is contributing to a GitHub "upstream" project, one **always**:

  * Forks the project to her own namespace
  * Clones the namespace fork locally (ie. `origin`)
  * Adds the upstream project as a remote (ie. `upstream`)

Now, we have seen that if one runs the following command, it will, at the project level (ie. `$PROJ/.git/config`) enable one to `git fetch` all pull requests locally

```
#> git config --add remote.upstream.fetch '+refs/pull/*/head:refs/remotes/upstream/pr/*'
```

Now, if one is consistent about her approach when contributing to GitHub "upstream" projects, one can run the following command and enable it system wide (ie. `~/.gitconfig`)

```
#> git config --global --add remote.upstream.fetch '+refs/pull/*/head:refs/remotes/upstream/pr/*'
```

And this will be true for **all** the projects that will follow the convention, there will be no need of a per-project configuration.


## Conclusion

While having the Pull Requests available online is fine, the fact to have them locally presents many advantages.

  * Their content becomes available offline, meaning the review and testing can be done while offline
  * If one wants to run a set of test pre-merging check configured in GitHub wouldn't be testing, it becomes easy to do so
  * Use of habituals tools for review `$GIT_EDITOR`, `$GIT_PAGER`, ... make it a productivity boost
