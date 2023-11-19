# Mkdocs

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

前面是安装 mkdocs，它基于 Python，用 pip 安装即可。教程有。

**本地预览网页的一些命令**

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

**本地的项目结构**

    mkdocs.yml    # The configuration file.
    docs/
        index.md  # The documentation homepage.
        ...       # Other markdown pages, images and other files.

**部署**

部署到 git-pages 上，这一步的坑特别的多，官网的教程也未详细提及。
只是简单的提到 `mkdocs gh-deploy` 使用这个命令就可以将静态网页发布，但是我操作后仍然不行。
好在是找到了解决的办法：<https://blog.arisa.moe/blog/2022/220407-github-pages/#github-actions>

首先要部署一个 github 的 workflow，我直接复制了上面链接里的代码。

然后是链接里提到的，要将 github-pages 的 source 切换到 `gh-deploy` 的支线上才能成功发布。
这是因为 `mkdocs gh-deploy` 这个命令它将网页 push 到 github 上，但并不是 push 到主分支，而是 push 到 `gh-deploy` 支线。

做完上面这两步，终于是可以成功发布了。

以后每次写新文件的时候，记得修改一下 `mkdocs.yml` 文件。

但是不得不说，mkdocs 比 Jekyll 方便了太多，省去了很多复杂的步骤，非常适合小白。