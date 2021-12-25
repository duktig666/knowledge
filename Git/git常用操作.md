## git修改远程仓库地址

`git remote -v`     查看关联远程仓库地址

`git remote set-url origin 新的地址`      修改远程仓库地址

`git remote -v`     查看是否修改成功



## git clone 远程仓库的某个分支

`git clone -b 分支名 xxx.git`



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



## 中文乱码问题

```shell
# git status 乱码
git config --global core.quotepath false

#git commit 乱码
git config --global i18n.commitencoding utf-8

#git status 乱码
git config --global i18n.logoutputencoding utf-8
```

注意：如果是Linux系统，需要设置环境变量 `export LESSCHARSET=utf-8`

[Git for windows 中文乱码解决方案](https://www.cnblogs.com/ayseeing/p/4203679.html)



## git在.gitignore添加忽略文件不起作用

```shell
git rm -r --cached .
```

重新`add`和`commit`即可解决



## 多分支合并代码

对于多分支的代码库，将代码从一个分支转移到另一个分支是常见需求。

这时分两种情况。一种情况是，你需要另一个分支的所有代码变动，那么就采用合并（`git merge`）。另一种情况是，你只需要部分代码变动（某几个提交），这时可以采用 Cherry pick。

```sh
## 得到最新一次提交的commitHash（类似5b6612d84a7f401f4519b73f38348bc094ba5501）
git log -n1 --format=format:"%H"
## 可以查看所有的提交信息
git log
## Cherry pick 支持一次转移多个提交
git cherry-pick <HashA> <HashB>
```

参看：[git cherry-pick 教程（阮一峰的网络日志）](http://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)



## merge和rebase

比如`git pull`这个命令，我们经常会用，它默认是使用`merge`方式将远端别人的修改拉到本地；如果带上上参数`git pull -r`，就会使用`rebase`的方式将远端修改拉到本地。

这二者最直观的区别就是：`merge`方式合并的分支会有很多「分叉」，而`rebase`方式合并的分支就是一条直线。

**对于多人协作，`merge`方式并不好**。

那么问题来了，`rebase`是如何将两条不同的分支合并到同一条分支的呢：

![image-20211220094646114](https://cos.duktig.cn/typora/202112200947816.png)

**首先，找到这两条分支的最近公共祖先`LCA`，然后从`master`节点开始，重演`LCA`到`dev`几个`commit`的修改**，如果这些修改和`LCA`到`master`的`commit`有冲突，就会提示你手动解决冲突，最后的结果就是把`dev`的分支完全接到`master`上面。

假设你现在基于远程分支"origin"，创建一个叫"mywork"的分支。

但是与此同时，有些人也在"origin"分支上做了一些修改并且做了提交了. 这就意味着"origin"和"mywork"这两个分支各自"前进"了，它们之间"分叉"了。

![image-20211220110915080](https://cos.duktig.cn/typora/202112201109279.png)

在这里，你可以用"pull"命令把"origin"分支上的修改拉下来并且和你的修改合并； 结果看起来就像一个新的"合并的提交"(merge commit)：

![image-20211220111009074](https://cos.duktig.cn/typora/202112201110093.png)

但是，如果你想让"mywork"分支历史看起来像没有经过任何合并一样，你也许可以用 git rebase:

![image-20211220111043121](https://cos.duktig.cn/typora/202112201110178.png)

解决冲突：

- `git rebase  --continue`：这样git会继续应用(apply)余下的补丁。
- `git rebase  --abort`：在任何时候，你可以用--abort参数来终止rebase的行动，并且"mywork" 分支会回到rebase开始前的状态。

对于使用 git merge 来合并所看到的commit的顺序（从新到旧）是： C7 , C6 , C4, C5 , C3 ,C2,C1
对于使用 git rebase 来合并所看到的commit的顺序（从新到旧）是： C7 , C6‘,C5' ,C4,C3, C2,C1







