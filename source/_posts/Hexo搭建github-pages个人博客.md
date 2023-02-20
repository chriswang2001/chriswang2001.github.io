---
title: Hexo搭建github_pages个人博客
date: 2023-02-19 23:16:12
tags:
---

## 安装前提

- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)

## Github pages

GitHub Pages 是一项静态站点托管服务，它直接从 GitHub 上的仓库获取 HTML、CSS 和 JavaScript 文件，（可选）通过构建过程运行文件，然后发布网站。你可以在 GitHub 的 github.io 域或自己的自定义域上托管站点。(省下了自己买服务器的钱，非常适合于搭建个人主页或者博客)

1. 新建github存储库，存储库必须命名为 \<username\>.github.io（username就是你的用户名）

    ![username](https://docs.github.com/assets/cb-34195/images/help/pages/create-repository-name-pages.png)

1. 克隆仓库，创建index.html，写入hello world（可选）

    ``` bash
    git clone https://github.com/username/username.github.io
    cd username.github.io
    echo "Hello World" > index.html
    ```

1. 推送到github后，访问 \<username>.github.io , 查看github pages是否搭建成功（可选）

    ``` bash
    git add --all
    git commit -m "Initial commit"
    git push -u origin main
    ```

上述操作成功搭建了github pages。但是自己编写网页文件来搭建一个好看易用的网站是一件很耗时耗力的事情。好在我们可以通过许多优秀的博客框架（例如hexo、wordpress）来规避掉其中繁琐的操作，而专注于内容本身。

## Hexo

1. 通过npm安装hexo

    ``` bash
    npm install -g hexo-cli
    ```

1. 初始化文件夹（\<folder>即github pages仓库，需要先删除之前用于测试的index.html）

    ``` bash
    hexo init <folder>
    cd <folder> 
    npm install
    ```

1. 测试hexo是否初始化成功

    ``` bash
    hexo generate & hexo server
    ```

    执行上述命令后访问 http://localhost:4000

1. 在储存库中建立 .github/workflows/pages.yml，并填入以下内容

    ``` yml
    .github/workflows/pages.yml
    name: Pages

    on:
    push:
        branches:
        - main # default branch

    jobs:
    pages:
        runs-on: ubuntu-latest
        permissions:
        contents: write
        steps:
        - uses: actions/checkout@v2
        - name: Use Node.js 16.x
            uses: actions/setup-node@v2
            with:
            node-version: "16"
        - name: Cache NPM dependencies
            uses: actions/cache@v2
            with:
            path: node_modules
            key: ${{ runner.OS }}-npm-cache
            restore-keys: |
                ${{ runner.OS }}-npm-cache
        - name: Install Dependencies
            run: npm install
        - name: Build
            run: npm run build
        - name: Deploy
            uses: peaceiris/actions-gh-pages@v3
            with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./public
    ```

    node-version: "16" 中的16请替换为你本地的版本（可使用下述命令查询）

    ``` bash
    node --version
    ```

1. 在储存库中前往 Settings > Actions > General，设置github action的工作权限

    ![20230220104350](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220104350.png)

1. 将 main 分支 push 到 GitHub（注意有些存储库默认分支为master）

    ``` bash
    git push -u origin main
    ```

1. 前往github action页面查看部署工作进度。当部署作业完成后，产生的页面会放在储存库中的 gh-pages 分支。

    ![20230220104631](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220104631.png)

1. 在储存库中前往 Settings > Pages > Source，并将 branch 改为 gh-pages。

    ![20230220104710](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220104710.png)

1. 前往 username.github.io 查看网站。

### Hexo使用

#### 创建新推文

``` bash
hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

#### 启动hexo服务器（此时可以本地访问）

``` bash
hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

#### 生成静态文件

``` bash
hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

#### 主题设置

hexo生态拥有丰富的主题配置，More info: [Themes](https://hexo.io/themes/)

## 自定义域名

github pages支持自定义域名，意味着你可以通过你拥有的任意域名（而非默认的 \<uesrname>.github.io）来访问你自己的github pages。

如果你还没有自己的域名，可前往各大云厂商（例如阿里云、腾讯云）购买，本文将以腾讯云为示例。

1. 在腾讯云域名注册下找到解析按钮

    ![20230220110136](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220110136.png)

1. 创建A记录指向 gitbub pages服务器
185.199.108.153 和 CNAME记录指向你自己的github.io地址（如需ipv6, 再创建一个AAAA记录指向2606:50c0:8000::153）

    ![![20230220110608](httpsimage-1305582579.cos.ap-chengdu.myqcloud.com20230220110608.png)](https://image-1305582579.cos.ap-chengdu.myqcloud.com/![20230220110608](httpsimage-1305582579.cos.ap-chengdu.myqcloud.com20230220110608.png).png)

1. 在储存库中前往 Settings > Pages > Custom domain，改为你自己的域名

    ![20230220110940](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220110940.png)

1. 上一个步骤会在gh-pages分支中创建一个CNAME文件，但这个文件会在我们每次提交部署的时候被hexo覆盖。所以我们需要将这个文件放在main分支下的source文件夹里面。

    ![20230220111321](https://image-1305582579.cos.ap-chengdu.myqcloud.com/20230220111321.png)

1. 前往你的域名查看网站。
