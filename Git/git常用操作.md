## 远程分支改变，如何同步本地？

```shell
git branch -m master main
git fetch origin
git branch -u origin/main main
git remote set-head origin -a
```



## git强制提交本地代码到远程

```shell
git init
git remote add origin 远程仓库地址
git add . 
git commit -m “添加注释信息"
#强制提交
git push -u origin master -f 
```



## 版本回退

```shell
#查看历史提交日志
git log  
git log --pretty=oneline
#查看历史命令
git reflog
#回退指定版本
git reset --hard commit_id
##HEAD代表当前版本，一个^代表向上回退一个版本
git reset --hard HEAD^
```

