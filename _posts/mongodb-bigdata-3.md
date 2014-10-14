title: "大数据之mongodb:实时分析"
date: 2013-02-04 09:16
tags: mongodb
categories:  
- NoSQL
---
**TODO**

接上：[环境搭建](/blog/mongodb-bigdata-2)

现在环境搭建OK后，我们来做个应用来验证下mongoDB的数据处理和存储能力。

# 应用需求

这里写个《小说分析》,要完成如下：

1. 从txt小说中导入小说的内容，并按照每行的方式将小说的内容读取到mongodb中；
2. 分析导入的小说中，每篇小说中的出现的最多的人名；
3. 有个请求地址，可以将分析的结果实时显示在页面上。

# 模型设计

这里的模型有两个

1. 小说：现实内小说的抽象，包含书名，总行数，创建时间；
2. 小说行内容：小说的每行信息，包含书的信息，行内容，创建时间。

> 在 mongoDB的设计中，最好不用使用类似关系数据库的表和表之间的关联的方式，在mongodb中称为引用，就是文档之间不要采用引用的方式来完成映射，因为面向文档的nosql实现本来就是无模式。比如在这里，小说的行内容包含了书的内容，那么我们可以将小说存储到行内容中。
> 另外，由于这里是做技术验证，不表示这么设计小说和行内容的方式是正确的。

# 数据导入程序

这里 我使用 [golang](http://golang.org) 来作为开发语言，Java虽然是我最熟悉的，但是那个搞个demo出来，所需要的工作太大了，不如这种动态脚本语言方便。它的安装和入门可以参考[这里]
(https://github.com/astaxie/build-web-application-with-golang/blob/master/01.0.md).

mongodb的go驱动，在[这里](http://labix.org/mgo)获取。也可以通过 `go get launchpad.net/mgo`直接安装。

程度代码[Fork](https://github.com/yfyang/Story-Figures/fork)

# 数据分析程序

<!-- TODO 待完成 -->

# WEB应用

<!-- TODO 待完成 -->
