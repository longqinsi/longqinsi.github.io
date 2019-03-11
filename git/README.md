# [技术备忘录](../README.md) | Git
## 目录
  1. [如何在本地新建分支？](#create-branch)
  2. [如何把本地新建的分支推送到远程仓库？](#push-new-local-branch-to-remote)

  
## 问题
### 1.如何在本地新建分支？<a name="create-branch"></a>[↑](#top) 
下列命令在本地创建分支 new_feature_branch，并切换到该分支。
```bash
git checkout -b new_feature_branch
```
### 2.如何把本地新建的分支推送到远程仓库？<a name="push-new-local-branch-to-remote"></a>[↑](#top) 
下列命令把本地新建的分支 new_feature_branch 推送到远程仓库 origin
```bash
git push -u origin new_feature_branch
```