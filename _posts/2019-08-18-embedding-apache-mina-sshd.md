---
layout: post
title: Embedding the Apache MINA sshd server
date: 2019-08-18 12:00:57 -0500
categories: [dev, testing]
tags: [java, ssh]
featured-img: mina.png
---

<img src="/assets/images/mina.png" align="right">A simple example of embedding the latest version of the [Apache MINA pure Java SSHD server](http://mina.apache.org/sshd-project/index.html) in an application.
<!--more-->

The [Apache MINA SSHD](http://mina.apache.org/sshd-project/index.html) library is a great way to provide SSHD client or server functionality in your application. Unfortunately, it is not especially well documented and recent refactoring in the project has left most of the few usage examples you can find obsolete. [This repo](https://github.com/keathmilligan/sshdtest) provides a very simple server example to help you get started.

This example was built using version 2.3.0 of apache-sshd running on JDK 12.

# Build Configuration

The apache-sshd includes [optional socket libraries](https://github.com/apache/mina-sshd/blob/master/docs/dependencies.md#nio2-default-socket-factory-replacements) (MINA Core and Netty) that can be used in place of the default standard NIO2 sockets. These need to be excluded in your Gradle build, otherwise, you will get a runtime error complaining about having more than one IoServiceFactoryFactory enabled:

```
Exception in thread "main" java.lang.IllegalStateException: Multiple (2) registered IoServiceFactoryFactory instances detected. Please use -Dorg.apache.sshd.common.io.IoServiceFactoryFactory=...factory class.. to select one or remove the extra providers from the classpath
```

You should also exclude 'org.slf4j:slf4j-jdk14' since we'll be setting up our own logging.

#### build.gradle:
```groovy
dependencies {
    implementation 'org.apache.logging.log4j:log4j-api:2.11.1'
    implementation 'org.apache.logging.log4j:log4j-core:2.11.1'
    implementation 'org.apache.logging.log4j:log4j-slf4j-impl:2.11.1'
    implementation('org.apache.sshd:apache-sshd:2.3.0') {
        exclude group: 'org.slf4j', module: 'slf4j-jdk14'
        exclude group: 'org.apache.sshd', module: 'sshd-netty'
        exclude group: 'org.apache.sshd', module: 'sshd-mina'
    }
    testCompile group: 'junit', name: 'junit', version: '4.12'
}
```

# Server Setup

In this example, we're going to configure a very basic server with default configuration settings:

#### Main.java:
```java
        var sshServer = SshServer.setUpDefaultServer();
        sshServer.setHost("127.0.0.1");
        sshServer.setPort(2222);
        sshServer.setKeyPairProvider(new SimpleGeneratorHostKeyProvider());
        sshServer.setPasswordAuthenticator((username, password, session) -> {
            return true;
        });
        sshServer.setShellFactory(new ProcessShellFactory("/bin/sh", "-i", "-l"));
```

A few things to note:
* The `setHost` method sets the host that the server will bind to. If you omit this line, it will bind to the default address '0.0.0.0' which would accept connections from any interface. There is _no security_ in this example, so you sohuld not bind to an externally-accessible interface until some authentication mechanism (password, public key, etc.) is added.
* `setport` sets the bind port to 2222. The default for ssh would normally be 22, but using a number >1024 allows you to run the application without root.
* `setKeyPairProvider` sets up the host key generator. in tis case, a simple implementation that generates a new key each time you run the server.
* The call to `setPasswordAuthenticator` configures a "stub" password authenticator for this example. This stub simply returns `true` no matter what the user enters. Obviously, you will need to replace this with a real authentication scheme.
* Finally, `setShellFactory` is used to configure a shell factory that spawns `/bin/sh`. This should work on most Linux/Unix (inlcuding MacOS) systems, but you will need to change this on Windows.

# Run the Server

Run the server from your IDE by executing the Main class, or from the command-line by running

```
./gradlew run
```

You should see something like:

```
2019-Aug-18 13:05:51 INFO  [] [main] net.keathmilligan.sshdtest.Main - sshd example
2019-Aug-18 13:05:52 INFO  [] [main] org.apache.sshd.common.util.security.bouncycastle.BouncyCastleSecurityProviderRegistrar - getOrCreateProvider(BC) created instance of org.bouncycastle.jce.provider.BouncyCastleProvider
2019-Aug-18 13:05:52 INFO  [] [main] org.apache.sshd.common.util.security.eddsa.EdDSASecurityProviderRegistrar - getOrCreateProvider(EdDSA) created instance of net.i2p.crypto.eddsa.EdDSASecurityProvider
2019-Aug-18 13:05:52 INFO  [] [main] net.keathmilligan.sshdtest.Main - starting server
2019-Aug-18 13:05:52 INFO  [] [main] org.apache.sshd.common.io.DefaultIoServiceFactoryFactory - No detected/configured IoServiceFactoryFactory using Nio2ServiceFactoryFactory

Enter command ("quit" to exit, "?" for help):
```

You may see some warnings due to illegal reflective access and/or other deprecation issues in some of the dependencies, but these should not prevent the example from working.

At this point the server is ready to accept connections. You can confirm this by running `netstat -an -p tcp`:

```
tcp4       0      0  127.0.0.1.2222         *.*                    LISTEN
```

# Test

In another terminal window, connect to your test server with:

```
ssh localhost -p 2222 -o UserKnownHostsFile=/dev/null
```

The options specify the non-standard port number that is being used for testing purposes and also directs ssh to not update your "known_hosts" file since the server's key will get re-generated every time it is launched. If you did not include this option, the client would issue a security warning and refuse to connect.

When prompted for a password, just hit enter and a shell session will be started. You may notice some issues with characters either not being echoed or being doubled since there is no line-discipline configuration happening.

Type Ctrl-D to exit.

# Stop the Server

In the server window, enter "q" or "quit" to stop the server and exit the app.

### Source Code

<https://github.com/keathmilligan/sshdtest>
