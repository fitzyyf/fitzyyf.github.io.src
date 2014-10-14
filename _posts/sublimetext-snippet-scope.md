title: "sublime text2 Snippet Scope的查询方法"
date: 2013-01-29 11:52
tags: tool
categories:  
- 开发笔记
---

问题答案来源: [stackoverflow](http://stackoverflow.com/questions/13063033/sublime-text-2-name-of-scope-selector-for-objective-c)

最近开始折腾[Octopress](http://octopress.org/docs/)来建立自己的日志博客，在用markdown时，octopress的一些tag指令写起来比较麻烦，作为懒人，就想怎么懒的吧。这种指令级别的有很多可以方法可以选择了，比如snippet,macro等。

我使用的工具是 sublime text 2，所以就使用snippet的方式，建立好snippet，如下

{% codeblock code snippet lang:xml%}
<snippet>
	<content><![CDATA[
{% codeblock ${1:title} ${2:lang:bash} ${3:http://js} ${4:link text} %}
code snippet
{% endcodeblock %}
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<tabTrigger>codeblock</tabTrigger>
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<scope>source.markdown</scope>
</snippet>
{% endcodeblock %}

这样的话，在md文件中无法找到找到这个snippet。google，发现了问题答案，如开头所示

修改如下

{% codeblock code snippet lang:xml %}
<snippet>
	<content><![CDATA[
{% codeblock ${1:title} ${2:lang:bash} ${3:http://js} ${4:link text} %}
code snippet
{% endcodeblock %}
]]></content>
	<!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
	<tabTrigger>codeblock</tabTrigger>
	<!-- Optional: Set a scope to limit where the snippet will trigger -->
	<scope>text.html.markdown</scope>
</snippet>

{% endcodeblock %}


这样就OK了。
