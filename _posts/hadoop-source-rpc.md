title: "hadoop-NameNode RPC.getServer() 分析"
date: 2013-03-18 10:04
tags: hadoop
categories: 
- Hadoop
---

接上周的[NameNode分析](/blog/hadoop-hdfs-source-1/)留下的问题，继续往下分析。

这次打算去分析去了解下NameNode的RPC通信方面的实现，在网络上也有很多人分析了不少，比如[hadoop rpc机制 && 将avro引入hadoop rpc机制初探](http://www.tbdata.org/archives/1413)、[](http://blog.csdn.net/historyasamirror/article/details/6281689)、[Hadoop基于Protocol Buffer的RPC实现代码分析-Server端](http://yanbohappy.sinaapp.com/?p=110)、[HDFS之RPC分析](http://yuntai.1kapp.com/?p=707)、[ 源码级强力分析hadoop的RPC机制 ](http://weixiaolu.iteye.com/blog/1504898)。最后一篇对我的理解很有帮助，谢谢博主。此篇记录也只是将个人的理解记录在此。

# RPC协议
做JavaEE应用，单纯的Spring WEB应用多了，很多底层的内容都忘记了，比如网络通信方面。这里也补习下**RPC协议**。

RPC协议一种远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。采用的是C/S模式。请求程序就是客户端，而服务提供程序就是服务器端。

这里有篇[Java常见分布式协议比较-RPC](http://xiaoqing.me/2012/12/25/protocols-rpc/)

# Hadoop 的 RPC
了解了RPC后，我们接着上篇说说的PPC协议在Hadoop如何实现并起到什么作用

在 Hadoop的NameNode中，是通过如下代码创建RPC服务器的。
    
    this.serviceRpcServer = RPC.getServer(this, dnSocketAddr.getHostName(), 
              dnSocketAddr.getPort(), serviceHandlerCount,
              false, conf, namesystem.getDelegationTokenSecretManager());

PRC类是Hadoop实现的一个简单的RPC机制。

## RPC类简要分析

整个类结构如下所示

 {% img /images/blog/hadoop-rpc-uml.png  Hadoop-RPC %}

在下方的5个属于`RPC`的内部类。`PRC`的构造函数为私有的。

1. Invocation 内部类

    这个类的作用是用来定义方法调用的相关信息，包括方法名称和参数信息。它实现了 `Writable`这个接口，这个接口的作用是：在`DataInput`和`DataOutput`的基础上实现了一个简单的、高效的、系列化协议的序列化对象。

2. ClientCache 内部类

    使用客户端调用的套接字工厂的散列作为键来缓存其客户端对象。内部维护了一个键值对`clients`

        private Map<SocketFactory, Client> clients = new HashMap<SocketFactory, Client>();

3. Invoker 内部类

    代理实例的调用处理程序实现，反向代理调用类；也就是反向代理接口的实现。应该就是Hadoop执行相关任务或者方法的执行器。其内部实现了`invoke`方法。内部维持了`Client.Connection`-客户端连接以及`Client`客户端信息。
    
    	// 远程客户端连接
    	private Client.ConnectionId remoteId;
    	// 客户端连接
    	private Client client;
    	// 判断当前连接是否已经关闭
    	private boolean isClosed = false;
    	
    invoke方法为反向代理执行的方法，比较重要的代码如下
    
    	 ObjectWritable value = (ObjectWritable) client.call(new Invocation(method, args), remoteId);
    	 
    这段代码并没有执行和其他的反向代理常规的方法`method.invoke(ac,arg)`,因为在Hadoop中，是将数据发送给服务器端，服务器将处理的结果返回给客户端，所以这里的`invoke()`方法必然需要进行网络通信。那么上面那段代码的作用就可以看出来了。
    
    在上面我们可以看的出来，`Invocation`类封装了方法名和参数信息，这里将`Invocation`交给`client`的call方法来处理。

4. VersionMismatch 内部类

    一个异常类，用于当与RPC协议版本不匹配时抛出的异常，属于IO异常系列。

5. Server 内部类

    实现了IPC服务的RPC服务器，提供了Call队列。它继承于`org.apache.hadoop.ipc.Server`。`ipc.Server` 是一个抽象的RPC服务。
    
6. `ipc.Server`的主要结构
    
    * Server.Listener
    		
    	RPC服务的监听器，用来接收RPC客户端的连接请求和数据，其中数据封装成Call后Push到Call队列中。
    	
    	是在启动RPC服务端的时候的一个监听并接收连接请求数据的线程类。它绑定了服务器端Socket地址的Socket通道，并且包含一个使用这个Socket通道注册的选择器Selector，以及一个Reader线程。Reader用来读取用户请求。
    	
    * Server.Hander
    
    	RPC Server的Call处理者，和Server.Listener通过Call队列交互。
    	
    * Server.Responder
    	
    	RPC Server的响应者。Server.Handler按照异步非阻塞的方式向RPC Client发送响应，如果有未发送出的数据，交由Server.Responder来完成。
    	
    * Server.Connection
    	
    	RPC Server的连接，数据接收和解析数据。
    	
    * Server.Call
    
    带有客户端的Call信息，作为呼叫队列的处理对象存在。
    
{% codeblock Call.java lang:java %}
	private static class Call {
		// 客户端Call标识
		private int id;
		// 传递的参数
		private Writable param;
		// 客户端的连接
		private Connection connection;
		// 接收 响应时间
		private long timestamp;
		// 当前客户端Call的响应信息
    		private ByteBuffer response; 
{% endcodeblock %}

### RPC.getServer方法

开头我们说过，在上篇文章中，作为namenode的RPC服务端对象，都是通过`RPC.getServer`来构造出来的。通过对源代码的了解，可以看得出getServer()是一个创建Server对象的工厂方法，但创建的却是RPC.Server类的对象。
	
	/** Construct a server for a protocol implementation instance listening on a
	  * port and address, with a secret manager. */
	public static Server getServer(final Object instance, final String bindAddress, final int port,  final int numHandlers, final boolean verbose, Configuration conf, SecretManager<? extends TokenIdentifier> secretManager)  throws IOException {
    	return new Server(instance, conf, bindAddress, port, numHandlers, verbose, secretManager);
    }

## 在[源码级强力分析hadoop的RPC机制 ](http://weixiaolu.iteye.com/blog/1504898) 提出的问题

1. 客户端和服务端的连接是怎样建立的？
2. 客户端是怎样给服务端发送数据的？
3. 客户端是怎样获取服务端的返回数据的？

博主也给出了答案，这边只是记录下来，如果有兴趣，可以到原作者那查看。

### ipc.Client 类
整个类结构如下所示

{% img /images/blog/ipc-Clinet-uml.png  Hadoop-ipc-Client %}

内部类解释

* Call ：用于封装Invocation对象，作为VO，写到服务端，同时也用于存储从服务端返回的数据，同时也作为请求队列；
* Connection ：处理远程连接的线程类，每个远程连接都有一个Socket的地址，通过这个连接类对队列中的连接进行处理；
* ConnectionId ：唯一确定一个连接，包含远程连接的地址和客户端Token；
* ParallelResults：并行处理结果集；
* ParallelCall：用于并行调用的实现；
* PingInputStream：用于处理远程连接超时连接时，在没有出现故障异常时，进行重试连接，知道至少有字节被读出。

#### 问题1，客户端和服务端的连接如何建立的？

`Call#call`方法
{% codeblock Call#Call方法 lang:java %}
public Writable call(Writable param, ConnectionId remoteId)
                       throws InterruptedException, IOException {
    // 对请求参数解析封装为Call对象，用于向服务器发送
    Call call = new Call(param);
    // 根据remoteId来获取一个远程连接
    Connection connection = getConnection(remoteId, call);
    try {
        // 想服务器发送Call对象
        connection.sendParam(call);
    } catch (RejectedExecutionException e) {
      throw new IOException("connection has been closed", e);
    } catch (InterruptedException e) {
      Thread.currentThread().interrupt();
      LOG.warn("interrupted waiting to send params to server", e);
      throw new IOException(e);
    }
    boolean interrupted = false;
    synchronized (call) {
      while (!call.done) {
        try {
            // 等待结果返回，call#callComplete中的notify来唤醒线程
            call.wait();
        } catch (InterruptedException ie) {
          // 因中断异常而终止，设置标志interrupted为true
          interrupted = true;
        }
      }
      if (interrupted) {
        Thread.currentThread().interrupt();
      }
      if (call.error != null) {
        if (call.error instanceof RemoteException) {
          call.error.fillInStackTrace();
          throw call.error;
        } else {
          throw wrapException(connection.getRemoteAddress(), call.error);
        }
      } else {
          //返回结果数据
          return call.value;
      }
    }
}
{% endcodeblock %}

从上面的 `call`方法可以看出，获取一个远程服务器端连接是`getConnection`方法来获取的。

{% codeblock Client#getConnection lang:java %}

private Connection getConnection(ConnectionId remoteId,
                                  Call call)
                                  throws IOException, InterruptedException {
   if (!running.get()) {
     // 如果当前的客户端已经停止，那么中断当前的连接
     throw new IOException("The client is stopped");
   }
   Connection connection;
   do {
     synchronized (connections) {
        // 如果当前的connections连接池中有对应的连接，就不进行初始化创建了。
        connection = connections.get(remoteId);
        if (connection == null) {
            // 如果连接池中没有，则创建一个连接
            connection = new Connection(remoteId);
            // 向连接池中存放remoteId信息及当前创建的连接，注意这个时候并没有进行连接
            connections.put(remoteId, connection);
        }
     }
   } while (!connection.addCall(call));// 将Call对象加入call队列中。
   // 正式建立与服务器的连接
   connection.setupIOstreams();
   return connection;
}
{% endcodeblock %}

继续跟踪方法链的话，可以在 `connection.setupIOstreams`-> `connection.setupConnection`的方法找到，在`setupConnection`建立连接了。

{% codeblock Connection#setupConnection lang:java %}
private synchronized void setupConnection() throws IOException {
    short ioFailures = 0;
    short timeoutFailures = 0;
    while (true) {
        try {
            // 熟悉的Socket 连接
            this.socket = socketFactory.createSocket();
            this.socket.setTcpNoDelay(tcpNoDelay);
            // 如果启动用了用户组安全认证，则将将套接字绑定到指定的主机的主要客户端的名称，主要通过匹配，以确保服务器地址的客户端连接到主机名。
            if (UserGroupInformation.isSecurityEnabled()) {
                KerberosInfo krbInfo = remoteId.getProtocol().getAnnotation(KerberosInfo.class);
                if (krbInfo != null && krbInfo.clientPrincipal() != null) {
                String host = SecurityUtil.getHostFromPrincipal(remoteId.getTicket().getUserName());
                InetAddress localAddr = NetUtils.getLocalInetAddress(host);
                if (localAddr != null) {
                    this.socket.bind(new InetSocketAddress(localAddr, 0));
                }
            }
        }
        // 连接超时为20s
        NetUtils.connect(this.socket, remoteId.getAddress(), 20000);
        if (rpcTimeout > 0) {
            pingInterval = rpcTimeout;
        }
        this.socket.setSoTimeout(pingInterval);
        return;
    } catch (SocketTimeoutException toe) {
        // 最多重新连接 45 次
        // 总共有20s*45 = 15 分钟的重试时间。
        handleConnectionFailure(timeoutFailures++, 45, toe);
    } catch (IOException ie) {
        handleConnectionFailure(ioFailures++, maxRetries, ie);
    }
    }
}
{% endcodeblock %}

#### 问题2，客户端是怎么样将数据发送到服务端的？
`Client.Connection`类中有个`sendParam`方法，从注释来看，就是用来将数据发送到服务器端的。

{% codeblock Connection#sendParam lang:java %}
public void sendParam(final Call call) throws InterruptedException {
    if (shouldCloseConnection.get()) {
        // 如果当前连接已经关闭，则什么都不做，返回
        return;
    }
    // 锁定发送参数的锁定标记，以保证队列执行
    synchronized (sendParamsLock) {
        // 异步处理
        Future senderFuture = SEND_PARAMS_EXECUTOR.submit(new Runnable() {
            @Override
            public void run() {
                DataOutputBuffer d = null;
                try {
                    // 当前连接的远程服务器的输出
                    synchronized (Connection.this.out) {
                        if (shouldCloseConnection.get()) {
                            return;
                        }
                        if (LOG.isDebugEnabled()) {
                            LOG.debug(getName() + " sending #" + call.id);
                        }
                        // 创建一个缓冲区
                        d = new DataOutputBuffer();
                        d.writeInt(call.id);
                        call.param.write(d);
                        byte[] data = d.getData();
                        int dataLength = d.getLength();
                        // 首先写出数据的长度
                        out.writeInt(dataLength); 
                        // 向服务端写数据
                        out.write(data, 0, dataLength);
                        out.flush();
                    }
                } catch (IOException e) {
                    markClosed(e);
                } finally {
                    IOUtils.closeStream(d);
                }
            }
        });
        try {
            senderFuture.get();
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof RuntimeException) {
                throw (RuntimeException) cause;
            } else {
                throw new RuntimeException("checked exception made it here", cause);
            }
        }
    }
} 
{% endcodeblock %}

与写Socket类似

#### 问题3，客户端是怎么样得到服务器端返回的数据的？

同样通过调用方法链的方式，`Client.Connection#run`->`Client.Connection#receiveResponse`，`receiveResponse`方法解析了服务器端返回的数据。

{% codeblock Client.Connection#receiveResponse lang:java %}
private void receiveResponse() {
    if (shouldCloseConnection.get()) {
        return;
    }
    touch();
    try {
        // 阻塞读取id
        int id = in.readInt();
        if (LOG.isDebugEnabled()){
            LOG.debug(getName() + " got value #" + id);
        }
        // 在calls队列中找到发送时的那个对象 
        Call call = calls.get(id);
        // 阻塞读取call对象的状态 
        int state = in.readInt(); 
        if (state == Status.SUCCESS.state) {
            Writable value = ReflectionUtils.newInstance(valueClass, conf);
            // 读取数据 
            value.readFields(in);
            // 将读取到的值赋给call对象，同时唤醒Client等待线程
            call.setValue(value);
            // 删除已经处理的call队列
            calls.remove(id);
        } else if (state == Status.ERROR.state) {
            call.setException(new RemoteException(WritableUtils.readString(in),WritableUtils.readString(in)));
            calls.remove(id);
        } else if (state == Status.FATAL.state) {
            // Close the connection
            markClosed(new RemoteException(WritableUtils.readString(in), WritableUtils.readString(in)));
        }
    } catch (IOException e) {
        markClosed(e);
    }
}
{% endcodeblock %}


