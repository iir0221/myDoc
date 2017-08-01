# 版本回退
## 1.已经提交的情况下
### 确保最新版本
```bash
svn update
```
### 查看版本信息
```bash
svn log
```
### 回退到相应版本
例如：从版本12回退到版本11
```bash
svn merge -r 12:11 .
```
### 查看当前版本和回退版本之前有无差别
例如
```bash
svn diff -r 11 .
```
### 提交新版本
```bash
svn ci -m “revert version to r11”
```
## 2.没有提交的情况下
### 取消单个文件
```bash
svn revert FULL_FILE_PATH
```
### 取消某个目录
```bash
svn revert -R PATH
```

## 3.冲突

提交时，报类似冲突：

! C ***/**/abc.php

\> local edit, incoming delete upon update

解决：
```bash
svn revert ***/**/abc.php
```

再尝试提交
