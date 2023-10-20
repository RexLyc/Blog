---
title: "博客搭建之路"
date: 2021-08-10T23:14:17+08:00
categories:
- 计算机科学与技术
- 博客搭建
tags:
- 滚动更新
- 网站建设
thumbnailImagePosition: left
thumbnailImage: /images/thumbnail/lycStamp.png
---
&emsp;&emsp;这篇博文就是想分享一下从零开始的博客搭建之路。让有想法的朋友们也能一样记录自己的生活，让网络见证自己的成长。
<!--more-->
&emsp;&emsp;知识分享现在已经完全不是什么新鲜的内容了。但我一直不太喜欢寄人篱下的感觉。比如知乎、公众号等等。毕竟作为自己的专业，肯定是希望能在自己的网站上（合理合法地）做点有趣有意义的东西。
## 概述
- 主要准备
    - 一点点数量的人民币
    - 灵活的大脑和勤劳的双手
        - 前置知识：Markdown语法、Linux bash使用、亿点点计算机网络知识
- 工具选择
    - 网站生成工具：其实制作自己的网站，最快的方法就是做静态网站。现在已经有很多Markdown文件到静态网页的生成器。经典的如Hexo、Hugo。本网站目前实用的就是Hugo。这类生成器还有一个优点就是可以替换主题。内容基本不变的前提下，可以通过更换主题来变换网站的样式。非常方便。
        - 有能力的同学可以尝试自己制作主题模板。如果更厉害的话，可以自己尝试用前端写几个页面出来。（当然如果你有这个能力，应该下面的也不用看了哈哈）。
    - 服务器硬件：对于绝大多数朋友，用国内的服务器商就够用了。大品牌如阿里云、腾讯云、电信云这些厂商，都会定期开展活动。新用户可以白菜价尝尝鲜。如果你已经有服务器，直接用就行啦。
## 我的
- 基础工具准备
    - nginx（服务器软件）、MobaXterm（用于访问远程服务器的本地终端）、VSCode（文本编辑器）、Git（代码同步和版本管理）、Hugo-v0.87
- 剁手阶段
    - 购买服务器硬件，作为静态网站，最便宜的就够了。1核2G绰绰有余。
    - 购买域名
    - 购买SSL证书
- 内容准备
    - 在剁手之前，建议先用Hugo在本地生成一个网页，这样购买之后第一时间就可以将网站部署上去（hugo工程的public文件夹就是部署用的网站文件）
    - Hugo生成的网站支持的最好的内容是：文字、（动态静态）图片。扩展后可以使用HTML语法，支持任何内容类型。
- 部署
    - 在服务器硬件提供商、域名代理商、证书代理商的商店等页面中：
        - 将域名和ip绑定 <!-- 查一下绑定时候的几个什么A的区别 -->
        - 下载SSL证书
        - 获取服务器硬件的远程登录SSH名称和密码
    - 登录到远程服务器：
        - 安装nginx
        - 上传网站内容（建议用git同步，省事方便）
        - 上传SSL证书 <!-- 贴一下ssl证书包含的内容 -->
        - 修改nginx配置，指定SSL证书位置、指定网站位置 <!-- 贴一下自己的配置 -->
        - 启动nginx
            ```bash
            # 启动
            nginx
            # 关闭
            nginx -s stop
            # nginx 重新加载配置
            nginx -s reload
            ```
- 备案
    - 这是整个搭建过程中最慢的步骤，注意一定要在网站有内容可以展示的时候再去备案，否则会以网站未完成为理由被拒绝。
    - 域名备案：备案可以选择不同省的省管局，建议选一个快一点的，这个和你是哪个省的人没啥关系。这一步内容填写正确即可。
    - 公安备案：需要去公安信息管理系统中备案，需要一个手持身份证的拍照，唯一离谱的地方是不让自拍（不能是镜像）。
- 扩展内容
    1. 安全性：由于https协议（安全的超文本传输协议）具有明显优势，因此不建议还使用普通的http协议来发布网站。
## Markdown实用
- 添加右上角页内引用[<sup>1</sup>](#参考资料)，让文档和参考引用看起来更整洁：
    ```md
    [<sup>1</sup>](#参考资料)
    ```
## Hugo进阶
- 添加内嵌html支持，需要修改toml，令markdown引擎支持内嵌html（开启unsafe模式）
    ```toml
    [markup]
        [markup.goldmark] # 需要hugo版本高于0.64（大概），在使用goldmark之后
            [markup.goldmark.renderer]     
            unsafe = true
    ```
- Latex支持
    - 第一步：添加一个单独的html于主题文件夹partials下，并引入所需的js库，css内容。
    - 第二步：去head.html中利用插值表达式引用该html。
    - 第三步：最好是在markdown开头的Front Matter部分添加math: true。也可以在toml中全局启用MathJax。
    - 参考：[使用MathJax在Hugo的Markdown中绘制公式](https://note.qidong.name/2018/03/hugo-mathjax/)
- 使用其他人的主题，可以考虑使用bootcdn对js、css等进行加速，但注意只能使用主题中限定的版本。
- 标记草稿
    - Front Matter（就是markdown开头），添加标记
        ```md
        <!-- 作为草稿 -->
        draft: true
        <!-- 非草稿 -->
        draft: false
        ```
- 标记置顶
    - Front Matter添加权重，越大越优先
        ```md
        <!-- 权重 -->
        weight: 1000
        ```
- 添加侧边导航目录功能：
    - 基本原理就是使用Hugo从0.64（大概）开始提供的toc（table of content）变量，可以直接生成一个目录，剩余的事情就是配置一个css来美化。注意默认是从二级目录##开始记录。
        ```html
        <!-- 支持平滑移动 -->
        <style>
        * { scroll-behavior: smooth; }
        </style>
        <!-- 插入一个页面目录 -->
        <div>{{ .TableOfContent }}</div>
        ```
- 添加搜索功能：
    - 思路都是生成一个索引文件，然后进行搜索匹配，难点在于如何优雅的把搜索按钮添加到主题中（参考目录、归档页面的结构）
    - 最终选择用hugo本省生成json索引，然后使用fuse.js进行模糊搜索的方案，修改思路和内容如下
        1. /config.toml，添加:
            ```toml
            # HTML和RSS默认会生成，这里主要是通知Hugo，也有json文件需要生成
            [outputs]
                home = ["HTML","RSS","JSON"]
            ```
        1. themes/你的主题/layouts/_default/index.json，新增：
            ```json
            // 该文件就是生成的json文件的模板，指示Hugo生成的索引格式和内容
            {{- $.Scratch.Add "index" slice -}}
            {{- range .Site.RegularPages -}}
                {{- $.Scratch.Add "index" (dict "title" .Title "tags" .Params.tags 
                "categories" .Params.categories "contents" .Plain "permalink" .Permalink 
                "date" .Params.date "thumbnailImageUrl" .Params.thumbnailImage) -}}
            {{- end -}}
            {{- $.Scratch.Get "index" | jsonify -}}
            ```
        1. themes/你的主题/layouts/partials/search.html，新增：
            ```html
            <!-- 根据你自己的喜好，添加一个展示搜索结果的页面 -->
            <!-- 一定要有一个可供修改的入口元素 -->
            <div id="search-results">
                <!-- ...  -->
            </div>
            ```
        1. themes/你的主题/layouts/partials/sidebar.html，修改：
            ```html
            <!-- 根据你自己的喜好，把搜索页面和搜索入口添加到原有页面中 -->
            <!-- 这里我选择添加到导航栏 -->
           <nav>
                <div>
                    <input type="search" style="width: 100%;appearance: none"
                    id="searchInput" class="docsearch-input"
                    placeholder="搜索" />
                </div>
            </nav>
            {{partial "mysearch.html" .}}
            <!-- 引入依赖库，和具体实现search.js -->
            <script src="/scripts/jquery360/jquery.min.js"></script>
            <!-- fuse也可以直接从cdn获取 -->
            <script src="/js/fuse.min.js"></script>
            <script src="/js/search.js"></script>
            ```
        1. themes/你的主题/static/js/search.js，新增：
            ```js
            // 页面基础逻辑，略
            // ...

            // 使用Fuse模糊搜索
            function executeSearch(searchQuery){
                // index会生成在根目录下
                $.getJSON( "/index.json", function( data ) {
                    var pages = data;
                    var fuse = new Fuse(pages, fuseOptions);
                    var result = fuse.search(searchQuery);
                    populateResults(result,searchQuery);
                });
            }

            // 对搜索结果进行展示处理
            function populateResults(result,searchQuery){
                var html='';
                $.each(result,function(key,value){

                    html += '<div class="media">';
                    if (value.item.thumbnailImageUrl) {
                    html += '<div class="media-left">';
                    html += '<a class="link-unstyled" href="' + permalink + '">';
                    html += '<img class="media-image" ' +
                        'src="' + value.item.thumbnailImageUrl + '" ' +
                        'width="90" height="90"/>';
                    html += '</a>';
                    html += '</div>';
                    }
                    // 更多内容，略
                    // ...
                });
                // console.log(html)
                $('#search-results').html(html)
            }
            ```
    - 参考：
        - [几种搜索方式的官方指南](https://gohugo.io/tools/search/)
        - [hugo-elasticsearch](https://blog.travismclarke.com/project/hugo-elasticsearch/)
        - [使用lunr生成搜索](https://blog.csdn.net/weixin_33143629/article/details/118322638)
- 使用ShortCode引用本站文章
    - ShortCode是Hugo提供的类似于函数的一类语法糖。允许用户自定义一个ShortCode，并在markdown文本中调用自己的ShortCode
    - Hugo内置的ShortCode中，```ref```可以自动计算出渲染后的md路径，更方便站点内部相互引用，使用```relref```甚至可以指定到章节。
    > 参考：[Content Management: Shortcodes](https://gohugo.io/content-management/shortcodes/)
- 如何自定义模板
    - 其实我也想知道
## CDN加速
&emsp;&emsp;当我们的网站初步搭建好之后，会发现稍微有点图片，有点公用的js、css、fonts资源就会变得很慢，这时候我们就需要使用CDN来进行加速。服务器硬件提供商一般就都提供CDN服务，如果你希望快一些，可以考虑购买一些套餐，其实价格并不会很贵（如果你的网站数据访问量不大的话）。

&emsp;&emsp;这里有一些配置比较令人疑惑，要耐心的根据官方教程进行配置。
<!-- 贴一下自己的配置，尤其是回源那里 -->

## Wiki类网站参考
&emsp;&emsp;看看其他的知识分享类网站的建设逻辑，可以向博客中添加有用的部分。
- 链接：[Confluence](https://www.atlassian.com/software/confluence)、[MediaWiki](https://www.mediawiki.org/)、[DokuWiki](https://www.dokuwiki.org/dokuwiki)、[BlueSpice](https://bluespice.com/)

## 参考资料
1. [emoji编码查询网站](https://www.webfx.com/tools/emoji-cheat-sheet/)
1. [如何创建自己的hugo主题](https://www.jianshu.com/p/0b9aecff290c)
1. [Hugo官方文档](https://gohugo.io/documentation/)
1. [Hugo中文帮助文档](https://hugo.aiaide.com/)
<!-- nginx原理 -->
<!-- dns解析原理 -->
<!-- https原理 -->
<!-- CDN原理和术语 -->