---
layout:     post
title:      "Git 學習筆記 - 指令相關"
date:       2015-04-04
author:     "Jesse"
catalog:    false
tags:
    - git
---

```
git init
```
>Initialized empty Git repository

<hr>

```
git status
```
>see what the current state of our project

<hr>

```
git add octocat.txt
```
>To tell Git to start tracking changes made to octocat.txt, we first need to add it to the staging area by using git add.

<hr>

```
git pull origin master
```
>We can check for changes on our GitHub repository and pull down any new changes by running:
check github 資料是否有異動

<hr>

```
git diff HEAD
```
>we want the diff of our most recent commit, which we can refer to using the HEAD pointer.

<hr>

```
git diff --staged
```
>run git diff with the --staged option to see the changes you just staged.

<hr>

```
git reset octofamily/octodog.txt
```
>You can unstage files by using the git reset command. Go ahead and remove octofamily/octodog.txt

<hr>

```
git checkout -- octocat.txt
```
>Files can be changed back to how they were at the last commit by using the command

<hr>

```
git branch branchAA
```
>We want to remove all these pesky octocats, so let's create a branch called branchAA, where we'll do all the work

<hr>

```
git checkout branchAA
```
>You can switch branches using this command.

延伸用法 :
git checkout `-b` branchAA
to checkout and create a branch at the same time.

<hr>

```
git rm '*.txt'
```
>You can finally remove all those pesky octocats by using the git rm command which will not only remove the actual files from disk, but will also stage the removal of the files for us.

延伸用法 :
git rm `-r` folder_of_cats
This will recursively remove all folders and files from the given directory.

<hr>

```
git branch -d branchAA
```
>You can use git branch -d (branch name) to delete a branch.

延伸用法 :
You can either add the --force (`-f`) option or use `-D` which combines -d -f together into one command.

#### 相關資源：

1.中文教學 : http://dylandy.github.io/Easy-Git-Tutorial/

2.Code School : https://try.github.io/

3.好文 : http://gogojimmy.net/2012/01/17/how-to-use-git-1-git-basic/