---
title: Git Command
description: 'about git common command'
publishDate: 2025-11-09 13:13:12
tag:
    - git
    - shell
---

# Record common git commands

```bash
# check the current status of the warehouse and see the files that have been changed
git status

# show the commit history
git log

# initialize local git repository
git init

# add remote repository
git remote add origin https://xxx.com/xxx/xxx.git

# roll back all content to the previous version (commit message)
git reset -soft HEAD^ 

# undo all unsubmitted modifications in the workspace
git reset -hard HEAD^ 

# clone remote repository
git clone https://xxx.com/xxx/xxx.git

# add all changes to the staging area
git add .

# commit all changes from the staging area to local repository
git commit -m "feat: xxx"
 
# get updates from the remote repository
git pull origin branch-name

# push on remote repository
git push origin main

# create a new git branch
git checkout -b branchname

# show all branch
git branch

# delete special branch
git branch -d branchname
```
