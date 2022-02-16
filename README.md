# lab02-kafka-producer-consumer

This lab is adapted from [here](https://github.com/SciSpike/kafka-lab).

## Introduction

This lab consists of two parts.
In the first part, we'll create a producer that can create messages in Kafka.
In the second part, we'll consume these messages.

Both the `Consumer` and `Producer` are written in Java.

Here are the links to the labs:

1. [Producer lab](producer.md)
1. [Consumer lab](consumer.md)

In this lab we'll use Java and Maven to create a typical Java based producer and consumer of Kafka messages.
Apart from working on the lab in IntelliJ and let the IDE handle everything, you have two additional options for running Maven and Java.
\Either, you install them natively or you can run the tools through a docker container.

### Native install

If you already have Java and Maven installed, make sure you have:

* Java 8 or higher
* Maven 3.X

With a successful installation, you should be able to run `mvn -v` from command line.
E.g., here is what that may look like (it may look slightly different on your machine based on the maven version and the operating system you use):

```bash
$ mvn -v
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: C:\Users\Ronny Seiger\Downloads\apache-maven-3.8.4-bin\apache-maven-3.8.4
Java version: 11.0.12, vendor: Oracle Corporation, runtime: C:\Program Files\Java\jdk-11.0.12
Default locale: de_DE, platform encoding: Cp1252
OS name: "windows 10", version: "10.0", arch: "amd64", family: "windows"
```

### Docker install of Maven and Java

If you decide to use docker, you would want to open a docker instance with all the tools in the root of your lab and keep it open during the lab.

For OSX and Linux, you can simply run:

```bash
$ cd DIR_WHERE_MY_LAB_IS
$ docker run -it --rm --name lab02 -v "$PWD":/usr/src/lab02 -w /usr/src/lab02 maven:3-jdk-8 bash
root@58b8ca1d738c:/usr/src/lesson#
```

If you are on Windows, you may have to replace the `$PWD` with the full path of your lab directory.

You are now running a `bash` shell inside your docker instance.
Your directory is mapped to `/usr/src/lab02` inside the docker instance.

You should be able to run the maven command to check that everything is working `mvn --version`.
The output should be:

```bash
root@58b8ca1d738c:/usr/src/lesson30# mvn --version
Apache Maven 3.8.4 (9b656c72d54e5bacbed989b64718c159fe39b537)
Maven home: /usr/share/maven
Java version: 11.0.14.1, vendor: Oracle Corporation, runtime: /usr/local/openjdk-1
Default locale: en, platform encoding: UTF-8
OS name: "linux", version: "5.10.60.1-microsoft-standard-wsl2", arch: "amd64", family: "unix"
root@43abc88b405f:/usr/src/lab02#
```

For simplicity of the lab instruction, we'll simply refer to the maven builds using the native install. However, you should be able to use the docker image above to build any of the labs.
Hence, if you select to use the docker approach, when we specify

```
$ mvn package
```

We assume that you are inside your docker container when you do.