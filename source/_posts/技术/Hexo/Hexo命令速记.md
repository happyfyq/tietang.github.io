---
title: Hexo命令速记
thumbnail: http://7xiovs.com1.z0.glb.clouddn.com/DPP_0004.JPG
date: 2016-02-21 22:31:12
categories: 
	- 技术
	- Hexo
tags:
	- Hexo
---

## 简写

```
hexo n "我的博客" == hexo new "我的博客" #新建文章
hexo p == hexo publish
hexo g == hexo generate#生成
hexo s == hexo server #启动服务预览
hexo d == hexo deploy#部署

```

## 服务器

```
hexo server #Hexo 会监视文件变动并自动更新，您无须重启服务器。
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP

hexo clean #清除缓存 网页正常情况下可以忽略此条命令
hexo g #生成静态网页
hexo d #开始部署
hexo d -g #部署前先生成今天网页
```

## 监视文件变动

```
hexo generate #使用 Hexo 生成静态文件快速而且简单
hexo generate --watch #监视文件变动
```
## 完成后部署

```
两个命令的作用是相同的
hexo generate --deploy
hexo deploy --generate
hexo deploy -g
hexo server -g
```

## 模版

```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub

hexo new [layout] <title>
hexo new photo "My Gallery"
hexo new "Hello World" --lang tw
```

变量 | 描述
--- | ---
layout | 布局
title	| 标题
date	| 文件建立日期



```
layout: post
date: 2014-03-03 19:07:43
comments: true
categories: Blog
tags: [Hexo]
keywords: Hexo, Blog
description: 生命在于折腾，又把博客折腾到Hexo了。给Hexo点赞。
```

常用指令
---

## 安装升级

```
npm install hexo -g #安装  
npm update hexo -g #升级  
```

## init

``` bash
$ hexo init [folder]
```

新建一个网站。如果没有设置 `folder` ，Hexo 默认在目前的文件夹建立网站。

## new

``` bash
$ hexo new [layout] <title>
$ hexo n [layout] <title>
```

新建一篇文章。如果没有设置 `layout` 的话，默认使用 [_config.yml](configuration.html) 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。



## generate

``` bash
$ hexo generate
```

生成静态文件。

选项 | 描述
--- | ---
`-d`, `--deploy` | 文件生成后立即部署网站
`-w`, `--watch` | 监视文件变动

## publish

``` bash
$ hexo publish [layout] <filename>
```

发表草稿。

## server

``` bash
$ hexo server
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`。

选项 | 描述
--- | ---
`-p`, `--port` | 重设端口
`-s`, `--static` | 只使用静态文件
`-l`, `--log` | 启动日记记录，使用覆盖记录格式

## deploy

``` bash
$ hexo deploy
```

部署网站。

参数 | 描述
--- | ---
`-g`, `--generate` | 部署之前预先生成静态文件

## render

``` bash
$ hexo render <file1> [file2] ...
```

渲染文件。

参数 | 描述
--- | ---
`-o`, `--output` | 设置输出路径

## migrate

``` bash
$ hexo migrate <type>
```

从其他博客系统 [迁移内容](migration.html)。

## clean

``` bash
$ hexo clean
```

清除缓存文件 (`db.json`) 和已生成的静态文件 (`public`)。

## list

``` bash
$ hexo list <type>
```

列出网站资料。

## version

``` bash
$ hexo version
```

显示 Hexo 版本。

## 选项

### 安全模式

``` bash
$ hexo --safe
```

在安全模式下，不会载入插件和脚本。当您在安装新插件遭遇问题时，可以尝试以安全模式重新执行。

### 调试模式

``` bash
$ hexo --debug
```

在终端中显示调试信息并记录到 `debug.log`。当您碰到问题时，可以尝试用调试模式重新执行一次，并 [提交调试信息到 GitHub](https://github.com/hexojs/hexo/issues/new)。

### 简洁模式

``` bash
$ hexo --silent
```

隐藏终端信息。

### 自定义配置文件的路径

``` bash
$ hexo --config custom.yml
```

自定义配置文件的路径，执行后将不再使用 `_config.yml`。

### 显示草稿

``` bash
$ hexo --draft
```

显示 `source/_drafts` 文件夹中的草稿文章。

### 自定义 CWD

``` bash
$ hexo --cwd /path/to/cwd
```

自定义当前工作目录（Current working directory）的路径。




