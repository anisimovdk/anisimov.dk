---
layout: post
title: "Git Cookbook"
categories: git
toc: true
comments: true
---

This cookbook contains some useful git command.

## Reset local repository to remote

```bash
git fetch origin
git reset --hard origin/master
```

## Skip SSL check

For specified repo

```bash
git -c http.sslVerify=false clone https://example.com/path/to/git
```

Disable SSL check globally

```bash
git config --global http.sslVerify "false"
```

Disable SSL check for existing repo

```ini
# add following code to reponame/.git/config
[http]
  sslVerify = false
```

## Show short git log

Show short git log (last 10 commits)

```bash
git log -n 10 --pretty=oneline --abbrev-commit
```

## Create patch

Create patch from last 10 commits

```bash
git format-patch -10 HEAD --stdout > 0001-last-10-commits.patch
```

## Apply patch

Check patch before applying.
Also you can stripe paths from `/one/two/three/` to `two/three` by adding `-p2` param

```bash
git apply --check -p2 /path/to/0001-name.patch
```

Applay patch

```bash
git apply -p2 /path/to/0001-name.patch
```

## Clone remote branch to local

```bash
git checkout -b <branch> --track <remote>/<branch>
```

## Remove pruned branches

```bash
git fetch -p && for branch in `git branch -vv | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
```

## Rewrite remote commit history

```bash
git rebase -i HEAD~3 # rewrite history for last free commits
git push --force # force push to remote to rewrite remote history
```

## Update local branch from remote master

Old git way:

```bash
git fetch origin            # Updates origin/master
git rebase origin/master    # Rebases current branch onto origin/master
```

Modern git way:

```bash
git pull --rebase origin master
```
