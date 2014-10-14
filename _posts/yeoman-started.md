title: "yeoman入门"
date: 2013-01-28 16:59
tags: yeoman
categories:  
- 开发笔记
---

一直在前端上面，不知道该如何做很好的管理；比如包，比如类似Maven的骨架一样建立一个工程，每次都是需要找到各个依赖（jQuery或者其他组件）。直到前段时间，看到了一个Yeoman的项目，后来在[InfoQ-Yeoman](http://www.infoq.com/cn/news/2013/01/yeoman)又进一步了解了[Yeoman](http://yeoman.io)的作用。终于忍不住安装学习下。

InfoQ上的介绍，说Yeoman是一个前端开发工作流，的确如此。具体可以看InfoQ的介绍。另外，我的入门过程基本照它来的[Yeoman at your service](http://www.adobe.com/devnet/html5/articles/yeoman-at-your-service.html)

# Yeoman安装

> windows下没有试过安装，我这里是在os x 下直接进行的。

1. 执行命令：
    
        curl -L get.yeoman.io | sh
        
    如果提示没有安装`Compass`、`OptiPNG`、`PhantomJS`、`Yeoman`的话，则按照提示进行相关的安装即可。比如，我这里的提示：
    
    {% img /images/blog/yeoman_init.jpg %}
    
2. 校验以及查阅帮助

        $ yeoman
        $ yeoman init --help
        $ yeoman help

# Yeoman 快速入门

这里简单对[Yeoman Quick Start](http://yeoman.io/gettingstarted.html) 简单介绍。

{% codeblock Yeoman Quick Start Command Line. lang:bash %}
yeoman init      # 使用默认的模板骨架创建初始化一个新的项目，最基本的应用骨架(Bootstrap,Boilerplate等)
yeoman build     # 构建并进行优化处理（比如压缩等），并生成dist目录。
yeoman server    # 启动预览服务器，生成的站点在目录temp下；
yeoman test      # 运行Mocha测试用例
yeoman install   # 安装到本地（感觉和Maven的mvn install一样）
yeoman uninstall # 卸载
yeoman update    # 更新某个资源包到最新版本
yeoman list      # 查看当前项目使用的资源包列表
yeoman search    # 根据资源包名称搜索
yeoman lookup    # 在某个特定的包中进行搜索

 #其他常用
yeoman init quickstart # 跳过交互式问题，直接创建最基本的应用

{% endcodeblock %}

# Yeoman Sample

创建一个Sample例子，通过进入一个空的文件夹，执行`yeoman init`命令，进行骨架初始化，过程中会提示下载一些必要的文件。

{% img /images/blog/yeoman_init.jpg %}

执行的代码

{% codeblock Yeoman Smaple. lang:bash %}
cd ~/Sites
mkdir yeoman-sample
cd yeoman-sample
yeoman init
{% endcodeblock %}

## 运行例子

    yeoman server

这时，yeoman会启动浏览器打开[localhost:3501](http://localhost:3501)。同时，我们如果修改源文件的话，yeoman会自动刷新浏览器，免去我们去切换浏览器在进行刷新的操作了。

## 单元测试

yeoman的单元测试默认的情况下使用的是[Mocha](http://visionmedia.github.com/mocha/),不大了解前端的测试，搜了下资料，应该不错。

    yeoman test

会进行如下输出

    
{% codeblock yeoman test. lang:bash %}
╭─yfyang@macbook 01/28/13  9:23PM ~/Sites/yeoman-sample
╰─λ  yeoman test
Running "server:phantom" (server) task

Starting static web server on port 3501
  - /Users/yfyang/Sites/yeoman-sample/test
I'll also watch your files for changes, recompile if neccessary and live reload the page.
Hit Ctrl+C to quit.

Running "clean" task

Running "coffee:compile" (coffee) task
Unable to compile; no valid source files were found.

Running "compass:dist" (compass) task
directory temp/styles/ 
   create temp/styles/main.css 

Running "mocha:all" (mocha) task
Testing index.html.OK
>> 1 assertions passed (0s)

Done, without errors.
{% endcodeblock %}

总体来说，和InfoQ上作者介绍的那样，Yeoman不愧为前端利器，值得推荐。
