# 常用
## 资料
官方文档
https://git-scm.com/doc

中文书籍
https://git-scm.com/book/zh/v2
## state
查看工作区状态：
```bash
git status
```
查看修改内容：
```bash
git diff 
```
## add commit
add了多个文件，想取消其中的某个
```bash
git reset HEAD filename
```
取消对文件的修改
```bash
git checkout --filename
```

## stash
存储未提交的工作：
```bash
git stash
```
查看存储的工作：
```bash
git stash list
```
删除存储的工作：
```bash
git stash clear
```
取出存储的工作：
```bash
git stash pop
```
## 查看branch
查看所有的分支：
```bash
git branch -a
```
查看远程所有分支：
```bash
git branch -r
```
查看本地分支和远程分支的跟踪关联关系：
```bash
git branch -vv
```
## 远程branch
根据远程branch2.1.29新建本地branch2.1.29：
```bash
git checkout -b 2.1.29 origin/2.1.29
```
更新远程branch2.1.29到本地branch2.1.29:
```bash
git pull origin 2.1.29:2.1.29
```
推送本地branch2.1.29的更新到远程branch2.1.29
```bash
git push <远程主机名> <本地分支名>:<远程分支名>
```

键入 git branch -vv查看本地分支和远程分支的跟踪关联关系

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略

```bash
git push origin
```
若推送失败，则尝试删除tag
```bash
git tag -d 2.1.29
```
## log
格式化输出日志：
```bash
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative
```
根据关键信息（bug号等）查看历史提交，获取commit id
```bash
git log --grep 1234
```

获取commit id，根据commit id查看该提交有哪些change
```bash
git diff 12344556667788～ 12344556667788
```
或者
```bash
git diff --name-only  4f749789c4d9bf6cf9~ 4f749789c4d9bf6cf9
```
查看完整提交信息：
```bash
git show --stat commit id
```
## 版本回退
```bash
git reset --hard commit id
```
## backport
```bash
git cherry-pick
```
在master上提交的代码，backport到2.1.29：

1.获取commit id

2.切换到2.1.29

3.git cherry-pick -n commit id
## 概念
### 工作区和暂存区：

新建文件夹learngit，并通过命令git init命令创建仓库后，会生成隐藏目录.git

则文件夹learngit为工作区

文件夹.git为版本库









