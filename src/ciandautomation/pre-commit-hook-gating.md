# Do not commit code that won't pass CI - Leverage pre-commit hook

## Table of Contents

  * [Git Hooks](#git-hooks)
    * [Vanilla approach](#vanilla-approach)
    * [For team approach](#for-team-approach)
  * [Pre-commit hook the gate keeper's gate keeper](#pre-commit-hook-the-gate-keepers-gate-keeper)


## Git Hooks

A hook is a mechanism that allows one to trigger custom actions when an event occurs.

Git hooks take the form of scripts that are invoked at a specific moment of the git flow. with the name of those scripts being  "hardcoded".
The full list can be found [here](https://git-scm.com/docs/githooks).

As long as they are executable, those scripts can be written in any language one wants.
By default, when cloning any repo, Git provides meaningfull samples of hooks. They all ends with `.sample`, one can simply remove this string from the filename and the hook will be enabled.

```
± % ll .git/hooks
total 48K
-rwxrwxr-x. 1 spredzy spredzy  482 Oct 14 14:16 applypatch-msg.sample
-rwxrwxr-x. 1 spredzy spredzy  900 Oct 14 14:16 commit-msg.sample
-rwxrwxr-x. 1 spredzy spredzy 3.3K Oct 14 14:16 fsmonitor-watchman.sample
-rwxrwxr-x. 1 spredzy spredzy  193 Oct 14 14:16 post-update.sample
-rwxrwxr-x. 1 spredzy spredzy  428 Oct 14 14:16 pre-applypatch.sample
-rwxrwxr-x. 1 spredzy spredzy 1.7K Oct 14 14:16 pre-commit.sample
-rwxrwxr-x. 1 spredzy spredzy 1.5K Oct 14 14:16 prepare-commit-msg.sample
-rwxrwxr-x. 1 spredzy spredzy 1.4K Oct 14 14:16 pre-push.sample
-rwxrwxr-x. 1 spredzy spredzy 4.8K Oct 14 14:16 pre-rebase.sample
-rwxrwxr-x. 1 spredzy spredzy  548 Oct 14 14:16 pre-receive.sample
-rwxrwxr-x. 1 spredzy spredzy 3.6K Oct 14 14:16 update.sample
```

**Note**: Please refer to the [githooks](https://git-scm.com/docs/githooks) documentation to know the parameters that are passed to each of those hooks.


### Vanilla approach

The vanilla approach of dealing with hooks, is - as shown above - to have one's hooks live in `$PROJ/.git/hooks`

In any of your git projects, simply run `cp .git/hooks/pre-commit.sample .git/hooks/pre-commit` and try commiting a file that contains trailing whitespace, one would end up with:

```
spredzy@bordeaux ~/Projects/github.com/Spredzy/productive-git [hooks *]
± % cp .git/hooks/pre-commit.sample .git/hooks/pre-commit

spredzy@bordeaux ~/Projects/github.com/Spredzy/productive-git [hooks *]
± % echo "test whitespace    " > test

spredzy@bordeaux ~/Projects/github.com/Spredzy/productive-git [hooks *]
± % git add test

spredzy@bordeaux ~/Projects/github.com/Spredzy/productive-git [hooks *]
± % git commit -m "Adding a new test file"
test:1: trailing whitespace.
+test whitespace****
```

Meaning the pre-commit hook is working and enabled and won't allow one to commit anyfile that contains trailing whitespaces - that was a fast and easy win.
Whitespaces being only one example of what could be done here. We can extend it so we can make sure all the YAML file we commit are valid, same things with JSON, all kind of linting actions.

While this was easy and fast this approach is not suitable for teams.
The content of the `.git/` is not version controlled as the git repo itself. Meaning that all your team members will have to manually enable and consistently update the content of the hooks - one can see how this is not going to work.


### For team approach


Luckily Git developers have thought about this use case and made it easy to leverage hooks, in a version controlled way and as a team.

Starting with Git 2.9, rather than having the hooks living on each team members' computers at `$PROJ/.git/hooks/pre-commit`, a team can have the hooks they rely upon be part of the project they are working on.
For example in `$PROJ/hooks/pre-commit`. This way this file is treated as any other file of the project and is updated constantly.

Team members only have to specify where hooks live on a per-repo basis by configuring `core.hooksPath`. If it is decided for the hooks files to live in `$PROJ/hooks` then running `git config core.hooksPath hooks` will ensure hooks are properly detected and run.


## Pre-commit hook the gate keeper's gate keeper

Most projects are now moving into a gating system approach meaning that uppon contribution reception, the contribution is ran through a certain level of testing in order to be merged. If the contribution does not pass the testing then it is rejected and a messsage let the contributor knows what failed. Else upon code-review approval the contribution is merged. Making the branch constantly in a good shape.

This level of testing is determined on a per-project basis. Some will want only linting, other linting and unit testing, other might want to test every commit with their full integration test suite.

While the testing will happen in-fine before the commit is merged on the platform the contribution is pushed to (GitHub, GitLab, email, ...) - this is not a reason to not do one's due dilligence **before** sending the contribution - and the best way to not forget about doing it - is the *pre-commit git hooks*.

The idea is to run locally (within reason of test suite execution time), for every commit created, the tests that are run in the CI and ensure they are passing. What is the point of pushing code, that ultimately will be rejected because it doesn't pass CI ?

From a collaboration/quality point of view, it forces to have the scripts that will be run in CI available within the repo and runnable from every contributor workstations.

From now on, one knows that her commit will *always* pass CI, and that only code-review will be the cause of rejection - not the trailing whitespace line 455 that one missed, or the wrong identation.
