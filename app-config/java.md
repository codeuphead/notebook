# JAVA

## 安装

```bash
wget --no-cookie --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.tar.gz

export JAVA_HOME=
export JRE_HOME=$JAVA_HOME/jre
export CLASS_PATH=.:$JAVA_HOME/lib:$JRE_HOME/lib

PATH=$JAVA_HOME\bin : $PATH
```

## 参数

```bash
-Xms512m  设置JVM促使内存为512m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

-Xmx512m ，设置JVM最大可用内存为512M。

-Xmn200m：设置年轻代大小为200M。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。

-Xss128k：

设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。

-Xloggc:file
 与-verbose:gc功能类似，只是将每次GC事件的相关情况记录到一个文件中，文件的位置最好在本地，以避免网络的潜在问题。
 若与verbose命令同时出现在命令行中，则以-Xloggc为准。
-Xprof

 跟踪正运行的程序，并将跟踪数据在标准输出输出；适合于开发环境调试。
```

## 非 Stable 参数

用-XX 作为前缀的参数列表在 jvm 中可能是不健壮的，SUN 也不推荐使用，后续可能会在没有通知的情况下就直接取消了；但是由于这些参数中的确有很多是对我们很有用的，比如我们经常会见到的-XX:PermSize、-XX:MaxPermSize 等等；

## FatJar

```groovy
buildscript {
  repositories {
    jcenter()
  }
  dependencies {
    classpath 'com.github.jengelman.gradle.plugins:shadow:2.0.4'
  }
}

plugins {
  id 'com.github.johnrengelman.shadow' version '2.0.4'
  id 'java'
}


def mainClassName = "com.xadslyf.cal.Main"

jar {
  manifest {
    attributes "Main-Class": "$mainClassName"
  }
}

tasks.withType(JavaCompile) {
  options.encoding = 'UTF-8'
}
```
