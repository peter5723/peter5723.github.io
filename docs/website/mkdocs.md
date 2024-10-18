# Mkdocs

The website is built with Mkdocs.

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

The theme is [Material for Mkdocs](https://squidfunk.github.io/mkdocs-material/).

## 安装
下面简单说下流程：

首先是安装 mkdocs，它基于 Python，用 pip 安装即可。教程有。由于我们使用 Material，看 Material 的部署即可。

若显示找不到 `mkdocs`，运行时尝试在前面加上 `python -m` 选项。(但是注意，一般是路径的问题，好好看看，之前在另外一台电脑的问题是，mkdocs 没有装在对应 anaconda 的环境中却装在了另一个地方)

### 本地预览网页的一些命令

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

### 本地的项目结构

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

### 部署

部署到 git-pages 上，这一步的坑特别的多，官网的教程也未详细提及。
只是简单的提到 `mkdocs gh-deploy` 使用这个命令就可以将静态网页发布，但是我操作后仍然不行。
好在是找到了解决的办法：<https://blog.arisa.moe/blog/2022/220407-github-pages/#github-actions>

首先要部署一个 github 的 workflow，我直接复制了上面链接里的代码。

然后是链接里提到的，要将 github-pages 的 source 切换到 `gh-deploy` 的支线上才能成功发布。
这是因为 `mkdocs gh-deploy` 这个命令它将网页 push 到 github 上，但并不是 push 到主分支，而是 push 到 `gh-deploy` 支线。

大致上的原理就是 `mkdocs gh-deploy` 将网页代码 push 到支线，然后我们设置的工作流开始运行，根据支线上的代码生成静态网页并发布。

做完上面这两步，终于是可以成功发布了。

### 日常用法

环境配完，以后增添新东西就很简单了：

1. 写新东西或者修改。记得在 `mkdocs.yml` 文件中调整文档的结构
2. 本地预览： `mkdocs serve`
3. `git commit -m ...` `git push` 提交上传你的修改。
4. `mkdocs gh-deploy` 发布成网页。

怎么插图片？和 markdown 语法一样。注意博客的相对路径是以 `docs` 文件夹为根目录的，所以多媒体文件也放入 `docs` 文件夹中。下面插入了一张图片。
![CPU](/media/image/CPUwww.png)


### 结尾？

但是不得不说，mkdocs 比 Jekyll 方便了太多，省去了很多复杂的步骤，非常适合小白。难怪看到很多人的博客都是用 mkdocs 做的。相见恨晚啊。前面用 Jekyll 踩了太多坑了。

一直都想做一个自己的网站或者说是博客之类的。一个人总有想说些什么的欲望，但是找不到合适的平台。现有的博客网站参差不齐。后来听说了 gitpages 很好用就想办法搭了一个。gitpages 的官方推荐的是使用 Jekyll 来生成静态网页。但是这样生成的网页，博客只能按照时间顺序排列，要想整理归类的话就非常的麻烦。看到 cc98 上有一个人是从语雀转到了 mkdocs， 没想到大名鼎鼎的[CS自学指南](https://csdiy.wiki/)也是使用 mkdocs 的。
今天才试了一下，好像宝藏一样。不仅很轻松地就可以管理博客的结构，而且换主题也很轻松，很好地解决了我的需求。mkdocs 用 markdown 就可以生成网页。还有一点问题是我用 Matlab 做笔记，用了很多实时脚本，不知道怎么导出。幸好今天发现了，最新版本的 Matlab 可以将实时脚本导出成 markdown 文件。我就把 Matlab 卸载了重装一遍，总算是解决掉笔记的问题啦。

## 其它

介绍一些杂七杂八的问题、插件、主题、导航栏等等吧。

插件要用 pip 安装，然后加到 yml 文件中。

### 数学公式
数学公式的插件，参考这个链接即可:

<https://hucorz.github.io/myDoc/%E5%85%B6%E4%BB%96/Mkdocs/#markdown>

<https://wdk-docs.github.io/mkdocs-material-docs/reference/mathjax/#configuration>

### 显示修改日期
显示修改日期插件:

<https://timvink.github.io/mkdocs-git-revision-date-localized-plugin/>

### 中文字体：

就是想试试别的字体了，这个字体是霞鹜文楷（能用的中文字体不多啊2333）。官网是 <https://lxgw.github.io/>.

配置很简单：在 `docs` 文件夹下建一个 `stylesheets` 文件夹，用来放 css 文件。建一个 `extra.css` 文件，里面复制
```yaml
body {
      font-family: "LXGW WenKai", sans-serif;
    }
```

然后在 `mkdocs.yml` 文件中加上
```yaml
    extra_css:
  - stylesheets/extra.css
  - https://cdn.jsdelivr.net/npm/lxgw-wenkai-webfont@1.1.0/style.css
```
就可以成功改变字体了。

2024年10月18日更新：今天开始改成用思源宋体了。用 Google Font 上带有的字体很简单：在 yml 文件的 `theme` 下指定 `font` 项就可以了：

```yaml
font:
    text: Noto Serif SC
    code: Roboto Mono
```

### highlight 和 删除线

<https://squidfunk.github.io/mkdocs-material/reference/formatting/#sub-and-superscripts>

### 下标
<https://facelessuser.github.io/pymdown-extensions/extensions/tilde/>

### 添加提示信息等

<https://facelessuser.github.io/pymdown-extensions/extensions/details/>

## 快速 markdown 表格

怎么插入 markdown 表格？可以在这个网站快速生成：<https://www.tablesgenerator.com/markdown_tables>

### 别人的博客
可以看看别人的博客怎么搭建:

<https://wcowin.work/>

<https://blog.arisa.moe/>

<https://note.tonycrane.cc/>
