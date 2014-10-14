title: "brew指定版本安装软件"
date: 2013-03-16 14:14
tags: 
- brew
categories:  
- 开发笔记
---

最近node.js升级到0.10版本了。周末今天看到yeoman好像有改变了，在主页上更改为`yo`、`brower`、`grunt`了，所以想升级下，发现执行`bower`时无法使用了。发生错误，找来找去，我猜估计是node.js折腾的，就打算降级node.js。

我的NodeJs是`bower`管理的，所以才有了这么一段记录，最后Google一大把，最后在这篇博客中找到了答案[brew install specific version of formula](https://coderwall.com/p/lqphzg)。我这边大致记录下我的过程：

{% codeblock brew version lang:bash %}
╭─yfyang@yfyang-mac 03/16/13  1:36PM /usr/local/share/npm/lib git:master ✗✗✗
╰─⚙ ─➤ brew versions node                                                                                                                        ✖ ✭
0.10.0   git checkout 687062f /usr/local/Library/Formula/node.rb
0.8.22   git checkout 3c4a714 /usr/local/Library/Formula/node.rb  # 下面一步可以指定这个 3c4a714
0.8.21   git checkout a3ef032 /usr/local/Library/Formula/node.rb
0.8.20   git checkout 9f95fff /usr/local/Library/Formula/node.rb
…
╭─yfyang@yfyang-mac 03/16/13  2:08PM /usr/local/share/npm/lib git:master ✗✗✗
╰─⚙ ─➤ git checkout 3c4a714 /usr/local/Library/Formula/node.rb
╭─yfyang@yfyang-mac 03/16/13  2:11PM /usr/local/share/npm/lib git:master ✗✗✗
╰─⚙ ─➤ rm -rf /Library/Caches/Homebrew/Formula/node.brewing
╭─yfyang@yfyang-mac 03/16/13  2:11PM /usr/local/share/npm/lib git:master ✗✗✗
╰─⚙ ─➤ brew install node
╭─yfyang@yfyang-mac 03/16/13  2:16PM /usr/local/share/npm/lib git:master ✗✗✗
╰─⚙ ─➤ npm install -g yo grunt-cli bower
{% endcodeblock %}

然后执行`yo`、`bower`都正常了。。

