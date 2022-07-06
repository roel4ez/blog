---
title: Using `git cherry-pick` to optimize your PRs
author: Roel Fauconnier
author_gh_user: roel4ez
read_time: 5min
publish_date: 06-july-2022
tags:
    - github
    - git
    - git-101
    - branching
    - pull-requests
---

One of the main requests that OSS repository owners have when submitting PRs is
that you use a small number of commits that fixes one specific issue. This is a 
*good thing*™️, as it makes it easier for the maintainer to review the PR and make
sure it can be merged without issues.

But what happens when the feedback on your PR is: "Please separate this into two
PRs". Great, now what? 🤔

No worries, this is where `git cherry-pick` comes to the rescue. The [official 
documentation](https://git-scm.com/docs/git-cherry-pick) describes it as "Apply
the changes introduced by some existing commits", but how do we use it in
practice?

Let's take the following situation for example, where the ask is to make two PRs:
one that contains C1, and one that contains everything starting from C2.
How do we get from this:

```mermaid
gitGraph
    commit
    branch feature/my-bugfix
    checkout feature/my-bugfix
    commit id: "C1"
    commit id: "C2"
    commit id: "C3"
    commit id: "C4"
    commit id: "C5"
    commit id: "C6"
    checkout main
    merge feature/my-bugfix
```

to this:
    
```mermaid
gitGraph
    commit
    branch feature/my-bugfix-1
    checkout feature/my-bugfix-1
    commit id: "C1"
    checkout main
    branch feature/my-bugfix-2
    checkout feature/my-bugfix-2
    commit id: "C2"
    commit id: "C3"
    commit id: "C4"
    commit id: "C5"
    commit id: "C6"
    checkout main
    merge feature/my-bugfix-1
    merge feature/my-bugfix-2
```

Do you have to do everything again now? No, you can use `git cherry-pick` to
setup your branches and create two new PRs:

```bash
$ [main]> 
    git checkout -b feature/my-bugfix-1
    git cherry-pick C1
    git push --set-upstream origin fix/my-bugfix-1

    git checkout main
    git checkout -b feature/my-bugfix-2
    git cherry-pick C2..C6
    git push --set-upstream origin fix/my-bugfix-2

    git push origin --delete feature/my-bugfix  # delete the old branch remotely
    git branch -d feature/my-bugfix             # delete the old branch locally
```

Let's take it step by step. First, we are creating a new branch
`feature/my-bugfix-1` and cherry-picking the commit `C1` into it. Then
we do the same for `feature/my-bugfix-2`, but instead of cherry-picking each
commit, we can specify the range `C2..C6` when executing the pick.
Finally, we remove the old branch `feature/my-bugfix`, since we won't be needing
that anymore.

!!! note "Git Commit Hashes"  
    `C1`, `C2`, etc.. are git commit hashes.
    You can find them when looking at your git history with for example `git log`, or
    via the GitHub web interface. It is basically a pointer to where git can find the
    bit of code that is part of the commit. It's a `SHA-1` hash that contains the
    commit message, the author, the date, the files that were changed, as well as the
    parent commit hash.  
    **TIP** The hash is actually a `40` character string, but git accepts
    the first `7` characters as a valid hash, which makes it easier to use.

