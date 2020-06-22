### 调用合约时报错

可能的问题：
一般的问题是合约执行失败导致，合约执行失败时Simplechain提示报错原因一般不是太直观，可能与错误无关。

- 1.合约账户中代币和余额不足；
- 2.当前操作账户是否有权限；
- 3.合约执行失败。

### spring boot应用程序使用web3j

可以直接使用web3j-spring-boot-starter 其中依赖的spring boot依赖包，不用重复依赖spring boot包

    <dependency>
    <groupId>org.web3j</groupId>
    <artifactId>web3j-spring-boot-starter</artifactId>
    <version>1.6.0</version>
    </dependency>

### spring boot应用程序中是用web3J依赖报错？？

我在demo中使用的gradle依赖web3j，功能完成没有问题，在正式项目中使用maven依赖web3j包相同的代码却报错，一直找不到具体问题。
使用maven依赖web3j 3.5.0后报错，换成web3j 3.6.0还是报同样的错误，报错信息如下：

    at java.net.URLClassLoader.findClass(URLClassLoader.java:382) ~[na:1.8.0_191]
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_191]
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349) ~[na:1.8.0_191]
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_191]
    at org.web3j.crypto.Sign.<clinit>(Sign.java:34) ~[crypto-3.5.0.jar:na]
    at org.web3j.crypto.ECKeyPair.create(ECKeyPair.java:68) ~[crypto-3.5.0.jar:na]
    at org.web3j.crypto.Credentials.create(Credentials.java:36) ~[crypto-3.5.0.jar:na]

解决办法：
经排查原因是因为maven下载web3j依赖时下载的不完整导致有些文件没有下载完整，ec包不存在，
删除maven本地仓库中已下载的web3j依赖包，然后下载maven工具，在项目目录中使用命令清除、安装依赖

    ->mvn clean
    ->mvn install

打开Idea,刷新项目就可以正常编译运行了。

### we3j编译 .sol文件时文件中使用 “import ./safeERC20.sol” 时编译会报错，找不到import文件。

解决办法是把import的contract或libary 写到当前文件中。