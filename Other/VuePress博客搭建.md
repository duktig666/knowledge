

## 本地搭建

直接基于主题 [vuepress-theme-reco](https://vuepress-theme-reco.recoluan.com/) 进行搭建。

### 执行初始化命令

方式一：npx

```sh
npx @vuepress-reco/theme-cli init 
npx @vuepress-reco/theme-cli init [文件夹名字]
```

方式一：npm

```sh
npm install @vuepress-reco/theme-cli -g theme-cli init
npm install @vuepress-reco/theme-cli -g theme-cli init [文件夹名字]
```

注意：`init` 后可以直接加文件夹根目录，避免初始化时需要再次生成目录（不创建目录执行，会发生些报错）。

![image-20220118213937581](https://cos.duktig.cn/typora/202201182139231.png)

上图这个步骤，`What style do you want your home page to be?` 有三个选项，分别是：

- blog（**推荐**）
- doc（和blog几乎一致，只不过 `init` 执行中输入的信息未应用，存在一些问题）
- 2.x（目前还不成熟）

*三个版本测试的时间为：2022.01.18*

### 安装依赖

```sh
cd [文件夹名字]
npm install
```

### 本地运行

```sh
npm run dev
```

## `.vuepress/config.js` 配置

### 修改静态文件输出目录

在 vuepress-theme-reco 主题中，`"dest"` 的值为`public`；在 vuepress 官方中，`"dest"` 的值为`.vuepress/dist`。

这个值意为，执行 `npm run build` 后，静态资源目录生成的位置，**会影响到后边GitHub Actions的配置**。

这里我们将`"dest"` 的值为`.vuepress/dist`。

## 部署上线

### 方式一：GitHub Actions 自动部署

**主分支存放代码，`gh-pages` 用于展示内容。**

#### 设置 Secrets

后面部署的 Action 需要有操作你的仓库的权限，因此提前设置好 GitHub personal access（个人访问令牌）。

生成教程可以看 GitHub 官方的帮助文档：[创建用于命令行的个人访问令牌 (opens new window)](https://help.github.com/cn/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)。

授予权限的时候只给 `repo` 权限即可。

![1W3GRA.png](https://s2.ax1x.com/2020/02/08/1W3GRA.png)

令牌名字一定要叫：`ACCESS_TOKEN`，这是后面的 Action 需要用的。

![1W35i4.png](https://s2.ax1x.com/2020/02/08/1W35i4.png)

#### 编写 workflow 文件

> 持续集成一次运行的过程，就是一个 workflow（工作流程）。

创建`.github/workflows/main.yml`文件，内容如下：

```yaml
name: Deploy GitHub Pages

# 触发条件：在 push 到 master 分支后
on:
  push:
    branches:
      - main

# 任务
jobs:
  build-and-deploy:
    # 服务器环境：最新版 Ubuntu
    runs-on: ubuntu-latest
    steps:
      # 拉取代码
      - name: Checkout
        uses: actions/checkout@v2
        with:
          persist-credentials: false

      # 生成静态文件
      - name: Build
        run: npm install && npm run build

      # 部署到 GitHub Pages
      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@releases/v3
        with:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: gh-pages
          FOLDER: .vuepress/dist # 静态资源目录
```

需要了解 workflow 的基本语法可以查看[官方帮助 (opens new window)](https://help.github.com/cn/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)，也可以参考[阮一峰老师的 GitHub Actions 入门](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

具体参看文章：[使用 GitHub Actions 自动部署博客](https://vuepress-theme-reco.recoluan.com/views/other/github-actions.html)

#### 设置GitHub Pages

![image-20220118233652096](https://cos.duktig.cn/typora/202201182336115.png)

### 方式二：两个仓库

#### 新建仓库一： `USERNAME.github.io` （不用克隆到本地）

`USERNAME` 必须是你 Github 的账号名称，这个仓库用来展示博客。

#### 新建仓库二：随便起一个名字，比如：vuepressBlog （克隆到本地）

这个项目是用来开发博客的，以后只需要改这个项目就够了。

创建部署脚本：

```sh
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run build

# 进入生成的文件夹
cd docs/.vuepress/dist

# 如果是发布到自定义域名
# echo 'www.yourwebsite.com' > CNAME

git init
git add -A
git commit -m 'deploy'

# 如果你想要部署到 https://USERNAME.github.io
git push -f git@github.com:USERNAME/USERNAME.github.io.git master

# 如果发布到 https://USERNAME.github.io/<REPO>  REPO=github上的项目
# git push -f git@github.com:USERNAME/<REPO>.git master:gh-pages

cd -
```

修改仓库二中的 deploy.sh 发布脚本，把文件中的 USERNAME 改成 Github 账号名。

这样仓库二和仓库一就建立了关联。

简单说二者的关系是：仓库一负责显示网站内容，我们不需要改动它；日常开发和新增内容，都在仓库二中，并通过 npm run deploy 命令，将代码发布到仓库一。

#### 在 package.json 文件夹中添加发布命令

```json
"scripts": {
  "deploy": "bash deploy.sh"
}
```

#### 运行发布命令

```sh
npm run deploy
```

## 添加自定义域名

在仓库一 USERNAME.github.io 中找到 Settings > Custom domain 把 **域名** 添加进去即可。

具体参看文章：[手把手教你使用 VuePress 搭建个人博客](https://www.cnblogs.com/softidea/p/10084946.html) （文中含有部署相关）



## 参看

- [vuepress-theme-reco](https://vuepress-theme-reco.recoluan.com/)
- [手把手教你使用 VuePress 搭建个人博客](https://www.cnblogs.com/softidea/p/10084946.html) （文中含有部署相关）
- [使用 GitHub Actions 自动部署博客](https://vuepress-theme-reco.recoluan.com/views/other/github-actions.html)
- [从hexo到vuepress](https://www.mhynet.cn/blog-hexo-to-vuepress/#hexo)