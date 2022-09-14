---
title: 玩转Github Pages
author: Lin Han
date: 2019-08-08 11:33:00 +0800
categories:
  - Tool
  - Github
tags:
  - Blog
  - Jekyll
  - Github
pin: true
---
![github-page](/assets/img/post/Tool/github-page.png)

[Github Pages](https://pages.github.com/)是Github的一个免费静态网页托管服务。可以用来搭个人博客，项目简介网页，组织官网（比如给实验室做个官网）等。本文将详细介绍Pages的使用方法（非常简单，最快10分钟就可以起一个个人博客）。开始前先进一段简介，也可以直接跳到做[项目主页](#项目主页)和[个人博客](#个人博客)的具体步骤。

## Pages简介

Pages的优点是不需要配服务器，数据库这些环境，简单稳定，而且免费。这使得Pages很适合做个人博客，项目主页，企业官网这一类纯展示性质（可能也不产生收益 :joy:）的站点。其缺点是只能托管静态网页，意思是对于每个用户网站展示的内容都是一样的。这不意味着Pages里不能有任何动态的元素，比如可以结合Issue实现[博客评论](#博客评论)。但是要做功能复杂的网站或者很多的数据处理，大概还是有一个服务器后台会更方便。

Pages在项目每次更新后会做两件事：
- 把项目指定分支，指定文件夹中的内容用Jekyll转成网站
- 托管这个网站

不过Jekyll只会转markdown文件，所有的网页都会原样保留，所以直接上传网站只是白嫖一下托管也是可以的。这种方案可以用Jekyll以外的静态网站生成工具，比如[hexo](https://hexo.io/)和[hugo](https://gohugo.io/)，过程和用Jekyll本地构建后直接上传网站相同。

总结起来用Pages主要有三种姿势
1. 上传markdown，直接让Pages用Jekyll构建成网站托管
2. 上传markdown，用Github Action构建成网站托管
3. 直接上传网页，让Pages托管

第一种方式最简单也很方便，但是Pages的Jekyll只提供了[13个](https://pages.github.com/themes/)为项目主页设计的单页主题，所以通常也只有项目主页用第一种方式。个人博客和组织官网用第二种的居多，直接Fork主题项目就行，够灵活而且不用本地配环境，Action脚本通常Jekyll主题都会带。直接上传网页灵活性最高，但是稍微麻烦一些。除非是自己直接写的网页否则感觉第二种方法就够用了。


## Github
[//]: # (TODO:创建项目)
这段面向之前完全没用过Github和git版本管理工具的纯小白，简单说说如何注册Github，创建项目和git是什么。Github是一个代码托管平台，开发者将代码放在这个平台上面，方便一个项目的多个开发者同步代码，同时也有助于开源项目的传播。注册需要一个邮箱，访问[Github注册](https://github.com/signup)页面，按提示操作就行。

![github-signup](/assets/img/post/Tool/github-signup.png)

Github上的项目可以自己新建也可以复制别人的，就做博客来说直接复制主题项目比较多。Fork的意思就是将别人的一个项目复制一份，存到自己的Github账户下。在每个项目的右上角都有一个Fork按钮

![fork button](/assets/img/post/Tool/fork-button.png)

点击之后就会跳转到一个自己的新项目，项目的内容和名称都和原来的项目一样，只不过这个项目是属于自己的，可以进行修改。

如果是要创建自己的项目，点主页左边一个绿色的[创建新项目](https://github.com/new)（New）按钮就行。

![new project](/assets/img/post/Tool/new-project.png)

![image](https://user-images.githubusercontent.com/29757093/152296633-62157681-e911-4fb8-82e6-ced17e103dff.png)

一般建议添加一个README，空项目是没法直接pull的，需要本地先创建之后推上去稍微麻烦一点。

git是一个代码版本管理工具，简单来说可以让你在写代码过程中创建一些存档点。如果只有一个开发者那么这些存档点就是一条直线，一个比一个新，像下面图里的红线。如果有多个开发者同时对项目做修改，存档点可以有一些平行的路径，像下面图里的蓝线和黄线。就做个人博客来说不需要了解很多的git知识，操作也不需要命令行，有很多带界面的git软件比如[Github Desktop](https://desktop.github.com/)。做博客最简单的方法是fork一个主题项目，将项目pull到本地，进行修改，之后再push回github的服务器上。后面用到的时候回说具体怎么操作。

![git](/assets/img/post/Tool/git.png)

## 项目主页

从最简单的开始，先介绍怎么做项目主页，也就是怎么用Pages直接把markdown变成一个能访问的网站。首先打开一个Github项目，点进Settings->Pages页面。一个没开启Pages的项目应该是这样的

![no-page](/assets/img/post/Tool/no-page.png)

Source这里选markdown所在的Branch

![source](/assets/img/post/Tool/source.png)

后面的文件夹选项是指定在这个Branch下去哪找markdown文件。如果只在根目录下有一个README.md的话保持默认/(root)就行，如果文档是分成多个文件可以都放到docs文件夹下，选择/docs。

![root](/assets/img/post/Tool/root.png)

保存。

![saved](/assets/img/post/Tool/saved.png)

稍等一会访问上面这个网址就能看到网页了。Pages用一个Enviroment进行静态网站生成和托管，查看进展的方法是首先回到项目主页

![project-home](/assets/img/post/Tool/project-home.png)

在右下角可以看到一个Enviroments，点进那个github-pages

![pages-environment](/assets/img/post/Tool/pages-environment.png)

就可以看到部署的执行情况，刷新几次就会看到一条新的部署记录，点view deployment就会跳转到网页了。

![no theme](/assets/img/post/Tool/no-theme.png)

默认部署没有主题看起来大概比较素，可以在Settings -> Pages -> Theme Chooser -> Choose a theme这里选一个主题。

![choose theme](/assets/img/post/Tool/choose-theme.png)

点击之后会看到主题的预览，点Select Theme选中一个主题。

![theme](/assets/img/post/Tool/theme.png)

这个时候Pages会在代码里创建一个_config.yml文件，记下你选的主题。内容类似这个

![config yml](/assets/img/post/Tool/config-yml.png)

等一会Pages更新之后就能看到带主题的网页了。


Pages给了12个主题，做项目主页不建议用这12个之外的。Pages的文档中描述了[怎么使用外部主题](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/adding-a-theme-to-your-github-pages-site-using-jekyll)但是Pages的Jekyll环境比较简单，很多主题都会缺gem依赖，而且部署过程貌似也没有详细的log，出问题了大都跟下面一样只有一句构建失败，不太好debug。

![build-fail](/assets/img/post/Tool/build-fail.png)

想用第三方主题建议用Action创建一个环境编译。

<!--
觉得都不好看可以从Jekyll官网列的一些[主题网站](https://jekyllrb.com/docs/themes/#pick-up-a-theme)里找一个主题。切换的主题需要在项目根目录下创建一个 _config.yml ，在里面写
```yaml
theme: [选择的主题名字]
``` -->

[//]: # (TODO:怎么判断有没有gem，怎么找名字，有什么限制)


## 个人博客

下面就到了这篇的重头戏，用Pages部署个人博客。过程非常简单，选一个主题，用Jekyll或者类似工具（如[hexo](https://hexo.io/)，[Octpress](http://octopress.org/)）把markdown构建成html，之后推到Github项目中就完成了（个人只用过Jekyll所以下文也是针对Jekyll写的，但用其他工具的流程也是一样的）。构建的这一步通常用Github Action，非常方便，搭建的过程10分钟就能搞定。本地构建麻烦一些，但是灵活度高，下文将分别介绍。

### 主题

如果是前端大佬大可以用html，css，js，Liquid这些语言自己搞一个主题，不过Jekyll有很丰富的主题生态可以即fork即用。选择主题的指导原则是好看和好用。好看纯看个人审美，没什么好说的。好用是希望主题的功能尽可能丰富。一些常见的功能包括：
- 博客内搜索
- 自动生成sitemap：便于搜索引擎收录，搜索引擎大概会是博客最主要的流量来源
- 文章标签和分类：博客首页一般都按发表顺序展示文章，内容多了之后标签和分类会比较实用
- 文章目录：长文用的上
- 发表时间，修改时间：发表时间一般都是有的，修改时间要看git记录不是所有主题都有
- 深色模式
- 代码块
- 数学公式
- ...

这些功能主题就算没有也都可以自己加，不过带的话可以省很多麻烦。此外主题一般都是Github项目，质量高的通常Star和Fork数比较多，更新频率高。好的主题一般教程详细，部署过程中也不容易出问题。

本文以我正用的[chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)为例，更多的主题可以逛逛下面这些网站

- [Github 上的 #jekyll-theme](https://github.com/topics/jekyll-theme)
- [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
- [jekyllthemes.org](http://jekyllthemes.org/)
- [jekyllthemes.io](https://jekyllthemes.io/)
- [jekyll-themes.com](https://jekyll-themes.com/)

推荐Fork主题，优点是用Github最近出的Fetch Upstream跟着上游更新主题比较方便。

![fetch-upstream](/assets/img/post/Tool/fetch-upstream.png)

Fork项目之后需要按照 用户名.github.io 格式改项目名，比如我Github用户名是linhandev，那项目名就是 linhandev.github.io。点Settings第一个设置就是。

![rename](/assets/img/post/Tool/rename.png)

如果名字冲突要删之前的博客千万千万记得保存内容，之前丢过好几波写好的文章。。。

看一看主题的 README 对使用方法的介绍，一般都会仔细的写Fork之后需要进行哪些操作，比如chirpy会需要运行一个脚本。看文档虽然无聊，但绝对比出问题之后一通debug节省时间。

Jekyll的主要设置都在根目录下的 _config.yml 文件里，比如网站标题 title ,副标题 tagline，时区 timezone，头图 avatar，和一些社交媒体配置之类的。走一波这个文件，一般都有注释每个设置是啥，把想改的改一改。

### Action构建

[//]: # (TODO:写示例action)
Action写起来稍微复杂，不过大多数主题要么自带用Action构建的脚本，要么可以直接用Pages的环境构建。写这部分的目的主要在于方便在博客构建失败的时候Debug，以后可能会更新这个Action具体怎么写。如果主题没提供这个脚本，自己对Action也不熟悉的话，那本地构建会更简单。

Github Action会在项目有一些动作之后触发，比如push，开Issue之类。Action的配置在项目的 .github/workflows 文件夹下。首先一定**看一下主题README.md里有关开始使用的部分**，一些主题会有一个初始化的脚本，之前用Chirpy的时候就因为没仔细看文档走了一堆弯路。

每次push更新之后Action会自动执行，在项目的代码页面commit的左边会有一个图标，就是下图红框的位置

![action-status](/assets/img/post/Tool/action-status.png)

Action执行过程中是一个黄点，执行成功是一个绿色的对号，如果看到一个红叉就是执行出问题了。执行成功之后点代码左上角的下拉菜单看看是不是多出一个branch

![branch](/assets/img/post/Tool/branch.png)

修改Settings->Pages里的设置，将Source改成这个branch

![change-branch](/assets/img/post/Tool/change-branch.png)

如果commit左边图表是红叉可以看一下Action执行的log来排查具体是什么问题。

- 点项目上面的Action能看到所有的执行

![actions](/assets/img/post/Tool/actions.png)

- 点进其中的一次执行是这样

![action-run](/assets/img/post/Tool/action-run.png)

- 点在那个continus-delivery位置的按钮（不一定叫一个名字）可以看到具体的执行log。

![acton-log](/assets/img/post/Tool/acton-log.png)

展开出错的部分就可以看到具体是什么问题了。

第一次执行Action需要创建环境，下一堆依赖应该会比较慢，可能要几分钟。chirpy的Action脚本会保存ruby的环境，以后每次执行大概只需要半分钟。Action执行成功后Pages还需要几分钟才会更新，只要Action执行成功了这次发布就没什么问题，等着就行。

### 本地构建

使用Action构建很方便，主要的缺点是每次push之后需要等上几分钟才能看到效果，本地跑Jekyll可以实时编译，保存之后刷新网页就能看到更新。直接推网页Pages更新的速度也稍快一些。

<!-- 此外本地构建更灵活，比如自己之前写脚本实现构建两个Jekyll网站，一个是博客，一个是简历，构建之后将简历的html塞进博客里，这种操作在Action里也可以实现但是比较复杂。 -->

#### 安装环境
Jekyll用Ruby编写，本地运行需要装Ruby和Ruby的包管理工具RubyGem，这里记Arch Linux的安装步骤，其他系统可以参考官方[安装文档](https://jekyllrb.com/docs/installation/)
```shell
sudo pacman -S ruby base-devel
```
安装的过程中遇到个小问题，pacman下清华源几个文件一直失败。解决的方法是上清华源的网站，直接下载对应的安装包文件之后 `pacmsn -U` 安装。装好ruby之后换源，之后装jekyll，bundle

```shell
# 添加清华源并移除默认源
gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/
gem sources -l # 列出所有源，应该只有TUNA一个
gem install jekyll bundler # 装包
bundle # 让bundle安装jekyll的依赖
jekyll # 测试安装是否正确
# 头两行输出应该是
# A subcommand is required.
# jekyll 4.2.1 -- Jekyll is a blog-aware, static site generator in Ruby
```
如果上面最后一行输出的是找不到 jekyll 命令，那应该是可执行文件路径里没有gem中的bin文件夹，仔细看看`gem install`命令的输出应该针对这个问题有提示，把bin的路径添加到PATH里就行。
```shell
gem environment # 找输出里的 GEM PATHS 部分
export PATH=$PATH:[上面的gem path] # 比如Arch上是 /usr/lib/ruby/gems/3.0.0
jekyll # 试试改的对不对
# 如果能找到命令了就把这行写到 ~/.bashrc 里，这样打开一个新命令行依旧有效
echo 'export $PATH=$PATH:[上面的gem path]' >> ~/.bashrc # 必需单引号，双引号变量会替换成值
```
到这编译Jekyll的环境应该就配置好了，下一步进行构建和push。

#### 构建和push
首先把Github上的项目clone到本地，创建新项目或者Fork主题，项目名要求是 Github用户名.github.io。前面已经说过了。如果创建新项目添加一个空的Readme方便后面clone。完成后把项目 clone 到本地。
```shell
git clone https://github.com/[username]/[username].github.io
cd [username].github.io # 进到项目里
```
如果是新项目将主题的所有文件放到项目里。应该有_config.yml，index.html，_post之类的一堆文件和文件夹。

这个时候就可以本地构建了
```shell
jekyll build # 构建结果在_site文件夹里
# 或者
jekyl b # b是简写，效果是一样的
```
此外还可以跑一个本地的网站展示博客，每次markdown文件保存后网站都会自动重新构建，刷新页面就能看到修改
```shell
jekyll serve # 之后访问 Server address: 那个网址就能看到
# 或者
jekyll s # 简写
```
构建成功之后给构建出来的网页文件单独创建一个branch
```shell
git checkout # 查看一下当前分支叫什么，一会还要回来
git branch gh-page # 创建 gh-page branch
git checkout gh-page # 切换到新branch
git rm -r * # checkout会把原来分支的所有文件都带过来，删掉它们
git commit -m "clean up"
git push --set-upstream origin gh-page # 推到 Github上
git checkout main # 返回之前的branch
```
分支创建完成了，build并推到 Github 上。下面的脚本有删除的代码，一定确定好自己在哪个branch上，否则可能误删markdown文件。
```shell
# 在主分支构建
git checkout main
jekyll build # md转html

# 将 _site 中生成的html网站放到 gh-page 分支的根目录里
git checkout gh-page
mv _site .site
rm -rf *
mv .site/* .
rm -rf .site
ls # 应该有404.html，main.html这样的文件

# 在 gh-page 分支 push
git add *
git commit -m "Update Blog"
git push

# 在 main 分支 push
git checkout main
rm -rf _site
git add *
git commit -m "Update Jekyll Blog"
git push
```
到这复杂的部分基本就完成了，最后跟前面一样改一下Pages的Source。到 Github 项目的Settings->Pages里，将 Source 设成 gh-page，/root，保存。

![page](/assets/img/post/Tool/Jekyll-Github-Page/page.png)

等几分钟网站应该就可以访问了。

![my-blog](/assets/img/post/Tool/my-blog.png)

### 博客评论

你可能希望大家在看完博客之后给点反馈。文章评论不是一个静态的功能所以光靠Github Page实现不了。我了解到有两种实现方案

- 用自己搭建的或第三方的评论服务
- 用Github API实现

最开始我是用的第三方评论服务[hyvor](https://talk.hyvor.com/)，当时试运营是免费的但是现在已经收费了。这种服务套餐的量一般都很大，比如hyvor起步的$5/月套餐就有10w page view。估计我有生之年博客都不会有这么大的流量。/笑哭

[//]: # (TODO:添加 gitalk https://github.com/gitalk/gitalk)

Github的服务依旧更加良心。 /笑哭 目前发现了两个基于Github Issue给文章添加评论的项目

- [utteranc](https://utteranc.es/)
- [gitalk](https://github.com/gitalk/gitalk)

utteranc看起来配置简单一点，gitalk的使用方法以后添加。

utteranc会给每篇文章创建一个Issue，用户对文章的评论会保存为Issue下的回复。评论的时候会需要用户用Github账户登陆，之后授权utteranc。

安装过程很简单

- 访问[这个网址](https://github.com/apps/utterances)，在Gtihub上安装utterances app

![utterancec-app](/assets/img/post/Tool/utterancec-app.png)

可以只给博客一个项目的权限

![utterances-config](/assets/img/post/Tool/utterances-config.png)

- 安装完成后会打开[utteranc网站](https://utteranc.es/)，按照引导进行设置，先写项目名

![repo-name](/assets/img/post/Tool/repo-name.png)

- 选择每篇文章Issue的格式

![issue-format](/assets/img/post/Tool/issue-format.png)

这里要注意utteranc会按照配置的格式去找文章对应的Issue，基本上是根据文章的标题或者markdown文件名。如果修改了文章的标题或者md文件名也需要修改对应Issue的名字，否则就找不到任何评论。个人感觉设第一个是最好的，文件名估计不会有文章名改的多。

- 之后可以填一个utteranc创建Issue的标签，作用不大

![issue-label](/assets/img/post/Tool/issue-label.png)

- 最后选择一个和主题视觉协调的配色

![utteranc-theme](/assets/img/post/Tool/utteranc-theme.png)

下面能看到一段代码，把它插入到文章模板里正文下面的位置就可以用了。Chirpy的文章模板是在 `_layouts/post.html`，别的主题也都类似。最后的效果

![utteranc-finish](/assets/img/post/Tool/utteranc-finish.png)

## 本地编辑环境

markdown不是所见即所得，而且插入图片一般有点费劲。本地环境可以很好的解决这些问题，简化写作流程。个人很喜欢Atom，在另一篇文章中记了[如何配置Atom写markdown][cab3ae05]，可以参考。

[cab3ae05]: https://linhandev.github.io/posts/Atom/ "Atom编辑器配置"

## 搜索引擎收录
一般个人博客的流量都不大，Chirpy主题的作者甚至不推荐大家开文章浏览量功能，怕打击博主的创作热情 /笑哭 不过如果你的博客有一些高质量的内容，搜索引擎是可以带来一些流量的。

搜索引擎优化是一门学问，但如果只是很佛系的告诉搜索引擎：“我有个网站，你爬不爬看着办”，那只要添加robots.txt，之后提交一个sitemap就行。

sitemap顾名思义就是网站的地图，告诉爬虫我这个网站都有些什么页面可以访问。大多数主题都会带这个功能，访问 [github id].github.io/sitemap.xml ，如果不是404那就是有sitemap的。比如我的长这样

![sitemap](/assets/img/post/Tool/sitemap.png)

如果你的主题不带sitemap也可以自己添加

[//]: # (TODO: 怎么添加sitemap)

robots.txt是用来告诉搜索引擎网站上哪些网页该爬，哪些网页不该爬的。可以访问 [github id].github.io/robots.txt 看看自己有没有这个文件。如果没有的话，在项目根目录中加上一个robots.txt文件就行，一个最简单的写法是

```text
User-agent: *

Disallow: /norobots/

Sitemap: https://[github id].github.io/sitemap.xml
```
{: file='robots.txt'}

这个文件告诉搜索引擎所有类型的爬虫都可以爬这个网站，除了 /norobots/ 路径下的所有页面都可以爬取和网站的sitemap在哪。

robots.txt和sitemap俱全就可以向搜索引擎提交网站了。这里需要用到各大搜索引擎的站长工具，webmaster或者search console。下面是一些常用的搜索引擎站长工具链接。

- [百度](https://ziyuan.baidu.com/site/index)
- [Bing](https://www.bing.com/webmasters/about)
- [Google](https://search.google.com/search-console/about)
- [搜狗](https://zhanzhang.sogou.com/)
- [头条](https://zhanzhang.toutiao.com/)
- [神马](https://zhanzhang.sm.cn/)
- [360](https://zhanzhang.so.com/)

提交的过程类似：
1. 注册帐号
2. 证明网站是你的
3. 提交sitemap

证明这一步通常有多种方法，对于Pages来说最方便的应该是添加文件。只需要将文件下载下来扔到项目根目录下，之后把修改推上Github就行。注意这个文件要放在项目目录而不是构建出来的网页目录，否则重新构建会把这个文件抹掉。

第二种常用的方法是在网站header中加入meta tag，这种方法的优点是添加多个搜索引擎的话项目的根目录整洁一些（如果这可以算优点的话）。tag的内容是一串随机字符。比如我的bing搜索验证tag是

```html
<meta name="msvalidate.01" content="D657F7EB150CC9886DA47F92C4D34ED6" />
```

在Chirpy主题中网站的header部分在 _includes/head.html定义，如果你的主题目录结构不一样可以在项目里搜一下head开头的文件。找到之后这行代码放到head.html的head部分就行了。

验证身份后找带sitemap的tab，点进去输入sitemap网址就大功告成了。就我这个网站添加多个搜索引擎之后不多的流量来看，bing国内版似乎是对新网站最友好的，大概提交sitemap 3 4天之后就能看到来自cn.bing.com的流量。Google也有一两个，其他国内的搜索引擎大多数连爬都懒得爬。


## 流量统计
[//]: # (TODO: 补全这块)


## 自定义域名

Pages的默认网址是固定的，比如个人博客都是 用户名.github.io，项目网站都是 用户名.github.io/项目名 。但是Pages也支持自定义域名。

要用自定义域名非常明显首先要自己有一个域名，这个在各大云厂商都能买到，大概的过程是
- 到[百度](https://cloud.baidu.com/)，[阿里](https://alibabacloud.com)，[腾讯](https://cloud.tencent.com/)这样的云服务上注册一个帐号
- 实名认证
- 找到域名产品
- 选一个下单
就可以了。价格跟很多因素有关，比如域名长度，顶级域名是什么，买多久之类的，大概每年几十。如果不想花钱也可以到[Freenom](https://www.freenom.com/)申一个免费的。
[//]: # (TODO:freenom怎么申请)

在域名解析的后台将这个域名CNAME解析到 username.github.io，之后在项目的 Settings -> Pages 里面填写自己的域名。这个时候 gh-page branch里会多一个CNAME文件，本地注意pull一下否则会冲突。在main分支的根目录里也加入这个文件。之后就可以使用自己的域名访问了。

之前从自定义域名转换回 Github 给的免费域名过程中遇到一点麻烦，输入免费域名总是直接跳转到之前解析的自己的域名，之后页面显示Github的404。这种情况清一下浏览器的cache就好了。

## 结语

到这基本上Github Pages的基本用法就介绍完了。这篇文章还会不断更新进行更正和丰富。如果使用过程中有任何问题也欢迎在下方评论留言。

[//]: # (TODO: 表单 https://formspree.io/plans)
