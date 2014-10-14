title: "django csrf 记录"
date: 2013-04-08 09:44
tags:
- django
categories: 
- Web开发
---

最近在看Python的Django WEB框架时，在处理其POST请求中，会出现如下`Forbidden（403）`问题，提示如下：

	Reason given for failure:
		CSRF token missing or incorrect.

不好意思，在这之间不知道啥叫CSRF，用`Google`得到如下结果：

1. 什么是`CSRF`

> CSRF是Cross Site Request Forgery的缩写，称其为“跨站请求伪造”。
> 常与XSS想提并论，但它与XSS非常不同，并且攻击方式几乎相左。
> XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。
> 与XSS攻击相比，CSRF攻击往往不大流行（因此对其进行防范的资源也相当稀少）和难以防范，所以有时候被认为比XSS更具危险性。

因为HTTP请求默认都会带上Cookie，所以攻击者可以在自己的网站上创建一个隐藏的表单并且用JS偷偷提交，而这个偷偷提交的请求是自动带有被攻击站点有效Cookie的。但是如果在Cookie里加一个每次登录都改变的csrf，并且在表单里加一个hidden域，提交时验证两者是否匹配，那么以上攻击就没法实现了，因为恶意网站无法读取你的Cookie（因为浏览器的同源策略），所以无法获得Cookie里的CSRF Token，无法伪造出csrf，POST就会失败，这样就不会产生安全问题。*来自[idndx](https://idndx.com)*

2.  Django的处理办法

我看的是最新的Django版本，为`1.5.1`，只需要在其视图方法中增加`@csrc_exempt`这个`decorators`即可解决。
