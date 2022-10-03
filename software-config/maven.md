# Maven

环境变量

```bash
M2_HOME=C:\Program Files\Apache Software Foundation\apache-maven-3.2.5

Path 中设置 %M2%
```

setting.xml 中添加镜像 `<mirrors>` 节点下添加

```xml
<mirror>
  <id>alimaven</id>
  <name>aliyun maven</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
  <mirrorOf>central</mirrorOf>
</mirror>
```

| 目录                           | 目的                             |
| ------------------------------ | -------------------------------- |
| \${basedir}                    | 存放 pom.xml 和所有的子目录      |
| \${basedir}/src/main/java      | 项目的 java 源代码               |
| \${basedir}/src/main/resources | 项目的资源，比如说 property 文件 |
| \${basedir}/src/test/java      | 项目的测试类，比如说 JUnit 代码  |
| \${basedir}/src/test/resources | 测试使用的资源                   |
