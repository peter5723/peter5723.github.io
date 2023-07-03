---
title: github pages搭建
author: Peter
date: 2023-07-03
category: Jekyll
layout: post
---
## 不知道取啥题目，试一下2级标题大小

今天（应该说是昨天）折腾了巨久怎么样基于github pages和Jekyll，搭建一个网页，现在终于基本上差不多弄好了，虽然是很简陋的gitbook的形式，不过也足够用了。其实我现在还是不知道Jekyll是什么，但是它能用2333.

现在想整理一下应该怎么用，以免以后忘记了。

一、新建一个github仓库，仓库的名字是`username.github.io`。

二、从网上找一个模板，复制到自己的仓库里面。

三、`index.md` `readme.md`是主页内容。

四、`_pages`文件夹里放网页

这个开始的格式挺重要的
```
	---
	title: github pages搭建
	author: Peter
	date: 2023-07-03
	category: Jekyll
	layout: post
	---
```

title就是网页的名称
	
别的没啥用，但是要保持这样的格式不然会报错
	
五、`_posts`文件夹就是用来放博客了，名称要求必须是`xxxx-xx-xx-name.md`，里面的开头的注释也必须是这样的:
```
---
title: 一定要这样写
author: Peter
date: 2021-08-10
category: Jekyll
layout: post
---
```
下面的内容用markdown写就可以了。

六、网页的本地测试具体看这个网站[Testing your GitHub Pages site locally with Jekyll - GitHub Docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) 

按照上面的内容装了一大堆东西以后，以后要本地测试，就只需要：
在网页文件夹里打开git bash
然后`bundle install`
然后`bundle exec jekyll serve`
就可以在`http://localhost:4000`查看网页了。

七、网页URL的配置在`_config.yml`文件中，记得配置

八、以后每次写完博客，`git push`上去就可以更新了，非常的方便，而且可以在obsidian里写博客，挺好的。

九、这不是教程，只是一个整理，看不懂不关我的事。
还不知道怎么插图片，估计挺麻烦的，可以不插。

十、参考的教程：
[Setting up a GitHub Pages site with Jekyll - GitHub Docs](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)

[Jekyll Gitbook Theme · Jekyll Gitbook (sighingnow.github.io)](https://sighingnow.github.io/jekyll-gitbook/)
