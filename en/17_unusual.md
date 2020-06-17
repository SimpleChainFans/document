### An error was reported when calling the contract

Possible problems:
The common problem is that the contract execution fails. When the contract execution fails, Simplechain prompts that the cause of the error is generally not very intuitive and may not be related to the error.

- 1.Insufficient tokens and balances in the contract account;
- 2.Whether the current operating account has permissions;
- 3.Contract execution failed.

### spring boot applications use web3j

You can directly use the spring boot dependency package that web3j-spring-boot-starter depends on without repeatedly depending on the spring boot package.

    <dependency>
    <groupId>org.web3j</groupId>
    <artifactId>web3j-spring-boot-starter</artifactId>
    <version>1.6.0</version>
    </dependency>

### Does the spring boot application use web3J dependency to report an error

The gradle I used in the demo depends on web3j, and there is no problem when the function is completed. In the formal project, the code that maven depends on web3j package is reported as the same, but no specific problem can be found.

If you use maven to rely on web3j 3.5.0, an error is reported. If you use web3j 3.6.0, the same error is reported. The error message is as follows:

    at java.net.URLClassLoader.findClass(URLClassLoader.java:382) ~[na:1.8.0_191]
    at java.lang.ClassLoader.loadClass(ClassLoader.java:424) ~[na:1.8.0_191]
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349) ~[na:1.8.0_191]
    at java.lang.ClassLoader.loadClass(ClassLoader.java:357) ~[na:1.8.0_191]
    at org.web3j.crypto.Sign.<clinit>(Sign.java:34) ~[crypto-3.5.0.jar:na]
    at org.web3j.crypto.ECKeyPair.create(ECKeyPair.java:68) ~[crypto-3.5.0.jar:na]
    at org.web3j.crypto.Credentials.create(Credentials.java:36) ~[crypto-3.5.0.jar:na]

Solution:

After troubleshooting, the reason is that the download of some files is incomplete when maven downloads the web3j dependency, and the ec package does not exist,
Delete the downloaded web3j dependency package in the maven local repository, download the maven tool, and use the command to clear and install dependencies in the project directory.

    ->mvn clean
    ->mvn install

Open Idea and refresh the project to compile and run normally.

### we3j reports an error when "import ./safeERC20.sol" is used in the file when compiling the. sol file. The import file cannot be found.

The solution is to write the import contract or libary to the current file.