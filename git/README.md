# [技术备忘录](../README.md) | Git
## 目录
  1. [如何在本地新建分支？](#create-branch)
  2. [如何把本地新建的分支推送到远程仓库？](#push-new-local-branch-to-remote)
  3. [解决Git在Linux 乱码问题](#git-linux-encoding)
  
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
### 3. 解决Git在Linux 乱码问题<a name="git-linux-encoding"></a>[↑](#top)
在Linux如果要提交的文件名是中文的，默认git commit的时候就会把中文显示为一串数字如下：
```
 create mode 100644 "\346\265\213\350\257\225"
```
这个时候只需要添加相应的配置即可显示正常的中文，执行以下命令即可
```
$ git config --global core.quotepath false
```
或者手动更改配置文件~/.gitconfig，编辑添加如下内容即可：
```
[core]
    quotepath = false
```