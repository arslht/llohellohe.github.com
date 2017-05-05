#### 创建ZooKeeper Session


	ZooKeeper(
	 String connectString,
	 int sessionTimeout,
	 Watcher watcher
	)
	
sessionTimeout单位为毫秒，

Watcher 是个接口，通过这个接口可以收到session建立或者断开的事件，

同样地，也能监视ZK数据的变化。

当连接成功后，会获得SyncConnected的通知，

如果连接断开，则会收到DISCONNECT通知。

##### 实现Watcher

一个对象的构造函数没有完成前不要调用这个对象的其他方法。

##### telnet
通过telnet zkserver，可以获取些基本信息。

如
	telnet localhost 2181 
	
	stat 可以查看zk的基本信息，包括有哪些客户端连接。
	
	dump 查看连接的过期时间
	
#### Create


	String create(String path,byte[]data,ACL,CreateMode)

正确创建后，返回节点的path,否则抛异常。

CreateMode是节点类型的枚举。

#### Stat

	 byte[] getData(String path,boolean watch,Stat stat)
	 
Stat 非必须，如果有的话，会将节点的信息复制到这个对象。

返回节点的数据信息。

如果watch为true的话，一旦后续数据发生变化，那么在创建session时的watch对象将收到通知。

同步版本的master[实现1](https://github.com/llohellohe/zookeeper/blob/master/src/main/java/yangqi/zookeeper/example/masterworker/Master.java)
[实现2](https://github.com/arslht/zookeeper/blob/master/src/main/javaApi/Master.java)


#### 异步操作
ZK的操作都提供了异步操作版本，有了异步版本后，可以消除部分while循环了。

比如create的异步操作，

	void create(String path, byte[] data,
	        List<ACL> acl,
	        CreateMode createMode,
	        AsyncCallback.StringCallback cb, //提供回调方法的对象
	        Object ctx) //用户指定上下文信息（回调方法调用是传入的对象实例）

前四个参数和同步操作相同，多了个callback和用于上下文传递的ctx。

其中Callback（回调函数）有多种类型，比如StringCallback和DataCallback。

StringCallback有个接口方法：

	 public void processResult(int rc, String path, Object ctx, String name);
	 
rc为返回的状态码，通过状态码可以判断操作是否成功。

ctx即用于传递的上下文对象。

##### 设置元数据

使用异步API设置元数据：三个目录/tasks、/assign和workers

zkCli.sh –server 10.77.20.23:2181

用户在客户端可以通过 telnet 或 nc 向 ZooKeeper 提交相应的命令（四字命令）

#### Master-Worker实例
[同步操作版本的master](https://github.com/llohellohe/zookeeper/blob/master/src/main/java/yangqi/zookeeper/example/masterworker/Master.java)

[异步操作版本的master](https://github.com/llohellohe/zookeeper/blob/master/src/main/java/yangqi/zookeeper/example/masterworker/AsynMaster.java)

#### 注册从节点

zk会严格维护执行顺序。多线程下，当回调函数中包括重试逻辑的代码时，容易发生错误：当遇到ConnectionLossException异常而补发一个请求时，新建立的请求可能排序在其他线程中的请求之后，而实际上其他线程中的请求应该在原来请求之后

#### 任务队列化

/tasks 使用有序节点

#### 管理客户端

通过getData和getChildren方法来获得主从系统的运行状态。
