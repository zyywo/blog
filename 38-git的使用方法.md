<!--markdown-->`git checkout bugfix`：切换到分支bugfix

`git branch bugfix`：创建分支bugfix

`git branch --list`：列出所有分支

`git branch -vv`：列出分支详细信息

把当前分支与远程分支建立映射关系：
```
git branch -u origin/branch-a
或
git branch --set-upstream-to origin/addFile
```

`git branch --unset-upstream`：撤销本地分支与远程分支的映射关系

`git branch -D bugfix`：删除分支bugfix

### 修改 git log 输出内容中的时区
```bash
//默认git的日志时区是 UTC +0000
//中国的时区是 UTC +8000
    
git log --date=local //临时生效
//或者
git config --global log.date local //全局生效
//查看
git config --list
```	