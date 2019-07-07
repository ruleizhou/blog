---
title: gitbook
typora-root-url: gitbook
date: 2019-07-07 16:02:19
categories:
- tools
- others
tags:
- tools
---

​	GitBook时一个基于Node.js的命令行工具，支持Markdown和AsciiDoc两种语法格式，可以输出HTML、PDF、eBook等格式的电子书。故可以将GitBook定义为文档格式转换工具。

# 安装

* `install node.js`

  下载安装[node.js](<https://nodejs.org/en/download/>)。默认包含npm

  

* `install gitbook-cli`

  ```shell
  npm install -g gitbook-cli
  ```

  

* `install typora`

  [下载地址](https://typora.io/)

  

# 配置

## title 

------

设置书本的标题

```markdown
"title" : "Gitbook Use"
```



## author

------

作者的相关信息

```markdown
"author" : "rulei"
```



## description

------

本书的简单描述

```markdown
"description" : "书籍描述"
```



## language

------

Gitbook使用的语言

```markdown
en, ar, bn, cs, de, en, es, fa, fi, fr, he, it, ja, ko, no, pl, pt, ro, ru, sv, uk, vi, zh-hans, zh-tw
```

配置使用简体中文

```markdown
"language" : "zh-hans"
```



## gitbook

------

指定使用的gitbook版本

```markdown
"gitbook" : "3.2.2"
"gitbook" : ">=3.0.0"
```



## root

------

指定存放GitBook文件(除了book.json)的根目录

```markdown
"root" : "."
```



## links

------

在左侧导航栏添加链接信息

```markdown
"links" : {
    "sidebar" : {
        "Home" : "http://zhangjikai.com"
    }
}
```



## styles

------

自定义网页样式，默认情况下各generator对应的css文件

```markdown
"styles" : {
    "website": "styles/website.css",
    "ebook": "styles/ebook.css",
    "pdf": "styles/pdf.css",
    "mobi": "styles/mobi.css",
    "epub": "styles/epub.css"
}
```



## plugins

------

配置使用的插件

```markdown
"plugins": [
    "disqus"
]
```

添加新插件后需要运行如下命令来安装新的插件

```
gitbook install
```

Gitbook默认带有5个插件

- highlight
- search
- sharing
- font-settings
- livereload

如果去除这些自带插件，可以在插件名称前加 -

```markdown
"plugins": [
    "-search"
]
```



## pluginsConfig

------

配置插件的属性

```markdown
"pluginsConfig": {
    "fontsettings": {
        "theme": "sepia",
        "family": "serif",
        "size": 1
    }
}
```



## structure

------

指定Readme、Summary、Glossary和Languages对应的文件名，下面时这几个文件对应变量以及默认值：

| 变量                | 含义和默认值                               |
| ------------------- | ------------------------------------------ |
| structure.readme    | Readme file name (default to README.md)    |
| structure.summary   | Summary file name (default to SUMMARY.md)  |
| structure.glossary  | Glossary file name (defalt to GLOSSARY.md) |
| structure.languages | Languages file name (default to LANGS.md)  |



# 插件

​	记录一些实用的插件，如要指定插件版本可以实用plugin@0.3.1。因为有些插件可能不会随着GitBook版本的升级而升级，即有些插件不适合高版本的GitBook，所以需要指定了GitBook的版本。这里值列举了一部分插件

目前主要使用的插件有:

- Navigator
- Splitter
- Tbfed-pagefooter
- Expandable-chapters



## Anchor-navigation-ex

------

添加Toc到侧边悬浮导航以及回到顶部按钮, 需要注意一下两点:

- 本插件只会提取h[1-3]标签作为悬浮导航

- 只有按照一下顺序嵌套才会提取

  ```json
  # h1
  ## h2
  ### h3
  必须要以 h1 开始，直接写 h2 不会被提取
  ## h2
  ```

[插件地址](https://plugins.gitbook.com/plugin/anchor-navigation-ex)

```json
{
    "plugins": [
        "anchor-navigation-ex"
    ]
}
```



## Splitter

------

使侧边栏的宽度可以自由调节

[插件地址](https://plugins.gitbook.com/plugin/splitter)

```json
"plugins": [
    "splitter"
]
```



## Tbfed-pagefooter

------

为页面添加页脚

[插件地址](https://plugins.gitbook.com/plugin/tbfed-pagefooter)

```json
"plugins": [
   "tbfed-pagefooter"
],
"pluginsConfig": {
    "tbfed-pagefooter": {
        "copyright":"Copyright &copy zhangjikai.com 2017",
        "modify_label": "该文件修订时间：",
        "modify_format": "YYYY-MM-DD HH:mm:ss"
    }
}
```



## Expandable-chapters

------

使左侧的章节目录可以折叠

[插件地址](https://github.com/DomainDrivenArchitecture/gitbook-plugin-expandable-chapters)

```json
plugins: ["expandable-chapters"]
```



## Search Plus

------

支持中文搜索，需要将默认的search和lunr插件去掉

[插件地址](https://plugins.gitbook.com/plugin/search-plus)

```json
{
    "plugins": ["-lunr","-search","search-plus"]
}
```



## Prism

------

使用Prism.js为语法添加高亮显示，需要将**highlight**插件去掉。该插件自带的主题样式较少，可以在安装prism-themes插件，里面多提供了几种演示，具体样式可以参考[这里](https://github.com/PrismJS/prism-themes)，在设置样式时需要注意设置css文件名，而不是样式名。

[Prism插件地址](https://plugins.gitbook.com/plugin/prism)	[Prism-themes插件地址](https://plugins.gitbook.com/plugin/prism-themes)

```json
{
    "plugins": [
        "prism",
        "-highlight"
    ],
    "pluginsConfig": {
        "prism": {
            "css": [
                "prism-themes/themes/prism-base16-ateliersulphurpool.light.css"
            ]
        }
    }
}
```

如果需要修改背景、字体大小等，需要在**website.css**定义**pre[class*="language-"]**类来修改,示例如下:

```css
pre[class*="language-"] {
    border: none;
    background-color: #f7f7f7;
    font-size: 1em;
    line-height: 1.2em;
}
```



## Github

------

添加github图标

[插件地址](https://plugins.gitbook.com/plugin/github)

```json
"plugins": [
    "github"
],
"pluginsConfig": {
    "github": {
        "url": "https://github.com/zhangjikai"
    }
}
```



## Github Buttons

------

添加项目在github上的star,watch,fork情况

[插件地址](https://plugins.gitbook.com/plugin/github-buttons)

```json
{
    "plugins": [
        "github-buttons"
    ],
    "pluginsConfig": {
        "github-buttons": {
            "repo": "zhangjikai/gitbook-use",
            "types": [
                "star",
                "watch",
                "fork"
            ],
            "size": "small"
        }
    }
}
```



## Ace Plugin

------

[插件地址](https://plugins.gitbook.com/plugin/ace)

使GitBook支持ace. 默认情况下,line-height为1,会使代码显得比较挤,而作者好像没有提供修改行高的选项,如果需要修改行高,可以到node_module --> github-plugin-ace --> assets -->ace.js中加入下面代码(30行左右的位置)

```json
editor.container.style.lineHeight = 1.25;
editor.renderer.updateFontSize();
```

不过上面的做法有个问题就是, 每次使用gitbook install安装新的插件之后,代码又会重置为原来的样子, 另外可以在website.css中加入下面的css代码来指定字体大小

```css
.aceCode {
  font-size: 14px !important;
}
```

使用插件

```json
"plugins": [
    "ace"
]
```



## Emphasize

------

为文字添加底色

[插件地址](https://plugins.gitbook.com/plugin/emphasize)

```json
"plugins": [
    "emphasize"
]
```



## Include Codeblock

------

使用代码块的格式显示所包含文件的内容,该文件必须存在. 插件提供一些设置. 如果同时使用ace和本插件, 本插件要在ace插件前面加载.

[插件地址](https://plugins.gitbook.com/plugin/include-codeblock)

```json
{
    "plugins": [
        "include-codeblock"
    ],
    "pluginsConfig": {
        "include-codeblock": {
            "template": "ace",
            "unindent": "true",
            "theme": "monokai"
        }
    }
}
```



## Mermaid-gd3

------

支持渲染[Mermaid](https://github.com/knsv/mermaid)图表

[插件地址](https://plugins.gitbook.com/plugin/mermaid-gb3)

```json
"plugins": [
    "mermaid-gb3"
]
```



## Puml

------

使用PlantUML展示 uml 图

[插件地址](https://plugins.gitbook.com/plugin/puml)

[PlanUML地址](http://plantuml.com/)

```json
{
    "plugins": ["puml"]
}
```

使用示例

```json
{% plantuml %}
Class Stage
    Class Timeout {
        +constructor:function(cfg)
        +timeout:function(ctx)
        +overdue:function(ctx)
        +stage: Stage
    }
    Stage <|-- Timeout
{% endplantuml %}
```



## Chart

------

使用C3.js或者Highcharts绘制图形

[插件地址](https://plugins.gitbook.com/plugin/chart)

[C3.js](https://github.com/c3js/c3)

[highcharts](https://github.com/highcharts/highcharts)

```json
{
    "plugins": [ "chart" ],
    "pluginsConfig": {
        "chart": {
            "type": "c3"
        }
    }
}
```

type可以是C3 或者 highcharts 默认时C3



## Sharing-plus

------

分享当前网页, 比默认的sharing插件多了一些分享方式

[插件地址](https://plugins.gitbook.com/plugin/sharing-plus)

```json
 plugins: ["-sharing", "sharing-plus"]
```

配置:

```json
"pluginsConfig": {
    "sharing": {
       "douban": false,
       "facebook": false,
       "google": true,
       "hatenaBookmark": false,
       "instapaper": false,
       "line": true,
       "linkedin": true,
       "messenger": false,
       "pocket": false,
       "qq": false,
       "qzone": true,
       "stumbleupon": false,
       "twitter": false,
       "viber": false,
       "vk": false,
       "weibo": true,
       "whatsapp": false,
       "all": [
           "facebook", "google", "twitter",
           "weibo", "instapaper", "linkedin",
           "pocket", "stumbleupon"
       ]
   }
}
```



## Sectionx

------

将网页分块显示, 标签的tag 最好是使用b 标签, 如果使用h1-h6可能会和其他冲突

[插件地址](https://plugins.gitbook.com/plugin/sectionx)

```json
{
    "plugins": [
       "sectionx"
   ],
    "pluginsConfig": {
        "sectionx": {
          "tag": "b"
        }
      }
}
```



## Local Video

------

使用Video.js播放本地视频

[插件地址](https://plugins.gitbook.com/plugin/local-video)

```son
"plugins": [ "local-video" ]
```



## Simple-page-toc

------

自动生成本页的目录结构. 另外GitBook在处理重复的标题时有些问题,所以尽量不使用重复的标题

[插件地址](https://plugins.gitbook.com/plugin/simple-page-toc)

```json
{
    "plugins" : [
        "simple-page-toc"
    ],
    "pluginsConfig": {
        "simple-page-toc": {
            "maxDepth": 3,
            "skipFirstH1": true
        }
    }
}
```

使用方法: 在需要生成目录的地方加上<!--toc-->



## Anchors

------

添加Github风格的锚点样式

[插件地址](https://plugins.gitbook.com/plugin/anchors)

```json
"plugins" : [ "anchors" ]
```



## Edit Link

------

如果讲GitBook的源文件保存到github或者其他仓库上,使用该插件可以链接当当前页的源文件上.

[插件地址](https://plugins.gitbook.com/plugin/edit-link)

```json
"plugins": ["edit-link"],
"pluginsConfig": {
    "edit-link": {
        "base": "https://github.com/USER/REPO/edit/BRANCH",
        "label": "Edit This Page"
    }
}
```



# 命令

## 列出所有命令

```shell
gitbook help
```



## 输出帮助信息

```shell
gitbook --help
```



## 生成静态网页

```shell
gitbook build
```



## 生成静态网页并运行服务器

```shell
gitbook serve
```



## 生成指定gitbook的版本

```shell
gitbook build --gitbook=2.0.1
```



## 列出本地所有的gitbook版本

```shell
gitbook ls
```



## 列出远程可用的gitbook版本

```shell
gitbook ls-remote
```



## 安装对应的gitbook版本

```shell
gitbook fetch 标签/版本号
```



## 更新到gitbook的最新版本

```shell
gitbook update
```



## 卸载对应的gitbook版本

```shell
git book uninstall 2.0.1
```



## 指定log级别

```shell
gitbook build --log=debug
```



## 输出错误信息

```shell
gitbook build --debug
```



# 使用

## 初始化

```shell
gitbook init
```



## 环境配置

将需要用到的插件在book.json中进行配置:

```json
{
    "plugins": [
		"navigator",
		"splitter",
		"tbfed-pagefooter",
		"expandable-chapters"
	]
}
```

配置完book.json后,使用命令进行相关插件安装:

```shell
gitbook intall
```



## 构建

会在项目的目录下生成一个_book目录，存放静态站点的资源文件

```shell
gitbook build
```



## Debugging

可以获取更好的错误信息(使用堆栈跟踪)

```shell
gitbook build ./ --log=debug --debug
```



## 启动服务

运行一个web服务，通过http://localhost:4000 可以预览该书籍

```shell
gitbook serve
```



