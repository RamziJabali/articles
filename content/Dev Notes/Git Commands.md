---
title: Git Commands
date: 2020-03-05
tags:
  - git
  - article
---
Essential Git commands to help with coding 
Git = Is a version control unit system, keeps control over versions.
Repo - Folder + data related to it's history. 
Clone - download a repo and begin tracking it.
Branch - A variant of the redo.
Fetch - download most recent version of the branch.
Merge - taker Changes from branch and apply it to another branch.

```bash
git config user.name
git config user.email
```
See your username & email

```bash
git config --global user.name "{{your_username}}"
git config --global user.email "{{your_email}}"
```
Set your new email or name

```
git init
```
Utility : To initialize a git repository for a new or existing project.
How to : git init in the root of your project directory.

```
git clone
```
Utility : To copy a git repository from remote source, also sets the remote to original source so that you can pull again.
How to : git clone <:clone git url:>

```
git status
```
Utility : To check the status of files you’ve changed in your working directory, i.e, what all has changed since your last commit.
How to : git status in your working directory. lists out all the files that have been changed.

```
git add
```
Utility : adds changes to stage/index in your working directory.
How to : git add.

```
git commit
```

Utility : commits your changes and sets it to new commit object for your remote.
How to : git commit -m”sweet little commit message”

```
git push
git pull
```
Utility : Push or Pull your changes to remote. If you have added and committed your changes and you want to push them.
Or if your remote has updated and you want those latest changes.
How to : git pull <:remote:> <:branch:> and git push <:remote:> <:branch:>

```
git branch
```

Utility : Lists out all the branches.
How to : git branch or git branch -a to list all the remote branches as well.

```
git checkout
```
Utility : Switch to different branches
How to : git checkout <:branch:> or **_git checkout -b <:branch:> if you want to create and switch to a new branch.

```
git stash
```
Utility : Save changes that you don’t want to commit immediately.
How to : git stash in your working directory. git stash apply if you want to bring your saved changes back.

```
git merge
```
Utility : Merge two branches you were working on.
How to : Switch to branch you want to merge everything in. git merge <:branch_you_want_to_merge:>

```
git reset
```

Utility : You know when you commit changes that are not complete, this sets your index to the latest commit 
that you want to work on with.
How to : git reset <:mode:> <:COMMIT:>

```
git remote
```

Utility : To check what remote/source you have or add a new remote.
How to : git remote to check and list. And git remote add <:remote_url:>

Merge process
      Commit 
          Push 
              Pull Request 
                    not Merged    Merged
