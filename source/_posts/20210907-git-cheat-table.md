---
title: Git 筆記
date: 2021-09-07
categories: develop
tags: [git, develop]
---

## 前言
----------

Git 常用指令

```
git status
git branch -avv
git remote show origin
```


<!--more-->

## 內容
----------

### Pull, fetch and push 

```
git pull
git add .
git commit -m <COMMIT_MESSAGE>
git push
```

### Show current repository status
```
git status
```

Sample Result (No changed files):
```
On branch master
Your branch is up to date with 'origin/master'.

nothing to commit, working tree clean 
```

Sample Result (With uncommitted changes)
```
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   source/.obsidian/workspace

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        source/_posts/20210907-git-cheat-table.md

no changes added to commit (use "git add" and/or "git commit -a")
```

### Show all branch with upstream
Using command with following.
```
git branch -avv
```   

Sample Result
```
* master                                                            944c6b6 [origin/master] Update
  remotes/origin/HEAD                                               -> origin/master
  remotes/origin/dependabot/npm_and_yarn/hexo-renderer-marked-4.1.0 33e72a0 Bump hexo-renderer-marked from 4.0.0 to 4.1.0
  remotes/origin/gh-pages                                           a0c1163 Deploy NatLee/Blog to github.com/NatLee/Blog.git:gh-pages
  remotes/origin/master                                             944c6b6 Update
```


```
* master            944c6b6    [origin/master]   Update
    ↑                  ↑             ↑             ↑
Current branch      Revision      Upstream       Commit message

'Upstream branches' are local branches that have a direct relationship to a remote branch.
```

### Show remote informations

```
git remote show origin
```

Sample result:
```
* remote origin
  Fetch URL: https://github.com/NatLee/Blog.git
  Push  URL: https://github.com/NatLee/Blog.git
  HEAD branch: master
  Remote branches:
    dependabot/npm_and_yarn/hexo-renderer-marked-4.1.0 tracked
    gh-pages                                           tracked
    master                                             tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```


