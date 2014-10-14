title: "red5 API 学习记录"
date: 2013-04-28 13:13
tags: red5
categories:  
- 开发笔记
---

1. `IScope`接口

	它定义了Red5中的作用域对象。维护了一个由一组客户端连接组成的上下文状态。通过作用域对象，我们可以很好的实现一个分级访问、区域对象的共享功能。
	它包含如下方法：
	
	* boolean addChildScope(IBasicScope scope)
	> 描述添加一个子作用域，返回是否添加成功。
	* boolean connection(IConnection conn)
	> 添加一个连接对象，返回是否添加成功。
	* boolean connect(IConnection conn,Object[] params)
	> 添加一个连接对象，并传入相应的参数对象。返回是否添加成功
	* boolean createChildScope(String name)
	> 通过一个字符串名称创建一个子作用域对象，返回是否创建成功
	* void disconnection(IConnection conn)
	> 通过从该作用域对象的连接对象列表中删除一个指定的连接对象，这样的操作会将所有提供该连接对象的客户端和本作用域断开连接。
	* IBasicScope getBasicScope(String type,String name)
	> 获得一个子作用域对象.
	* IteratorString getBasicScopeNames(String type)
	> 获得所有指定类型的子作用域.
	* SetIClient getClients()
	> 返回当前作用域对象中所有子作用域对象的范型集合.
	* IteratorIConnection getConnections()
	> 获得本作用域所有连接对象的范型迭代器.
	* IContext getContext()
	> 返回本作用域上下文环境.
	* String getContextPath()
	> 返回上下文路径.
	* IScopeHandler getHandler()
	> 获得该作用域对象的控制器对象.
	* IScope getScope(String name)
	> 通过名称获得作用域对象.
	* IteratorString getScopeNames()
	> 获得所有子作用域的名称迭代器.
	* boolean hasChildScope(String name)
	> 判断当前作用域是否有指定名称的子作用域.
	* boolean hasChildScope(String type, String name)
	> 通过指定类型和名称判断当前作用域是否有指定名称的子作用域.
	* boolean hasHandler()
	> 当前作用域是否存在控制器
	* SetIConnection lookupConnections(IClient client)
	> 通过客户端对象，查找连接对象
	* void removeChildScope(IBasicScope scope)
	> 删除指定的子作用域

2. 
