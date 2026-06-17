## 引言

CoreProtect 是一款为 Minecraft 插件服务器设计的日志记录插件，能够记录玩家对大多数方块的操作，例如破坏、放置、点击等。本文将介绍如何从源码构建该插件

演示环境：

- `Arch Linux`
- `Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)`
- `openjdk version "26.0.1" 2026-04-21`


在开始之前，先确认环境

```bash
java -version
```

Maven

```bash
mvn -version
```

## 步骤

### 克隆源代码

首先，克隆 CoreProtect 的源代码：

```bash
git clone https://github.com/PlayPro/CoreProtect.git
```

### 修改 `pom.xml`

找到 `pom.xml` 文件中的`<properties>`部分 ，修改项目分支

``` xml
<project.branch>master</project.branch>
```

### 构建

在CoreProtect项目根目录运行此命令进行构建：

```bash
mvn clean install
```

### 5\. 完成构建

如果一切顺利，你将在终端看到类似以下的日志输出：

```log

    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time:  35.659 s
    [INFO] Finished at: 2026-06-17T22:46:10+08:00
    [INFO] ------------------------------------------------------------------------
```

`BUILD SUCCESS` 表示构建成功。此时你可以在 `target` 目录下找到生成的插件文件，通常应使用非 `original-` 前缀的那个 JAR，例如 `CoreProtect-24.0.jar`

## 常见问题

### 类文件具有错误的版本

如果运行`mvn clean install`后，遇到了类似报错：

```log
......
[ERROR]   错误的类文件: /home/huajibenji/.m2/repository/io/papermc/paper/paper-api/26.1.2.build.9-alpha/paper-api-26.1.2.build.9-alpha.jar(/org/bukkit/block/data/type/Stairs.class)
[ERROR]     类文件具有错误的版本 69.0, 应为 65.0
[ERROR]     请删除该文件或确保该文件位于正确的类路径子目录中。
[ERROR] -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
```

这通常表示当前 JDK 版本无法读取依赖中的 class 文件

例如，`paper-api-26.1.2.build.9-alpha` 中的部分类是用 Java 25 编译的，如果你使用 Java 21 构建，就会出现 69.0, 应为 65.0 的错误

常见版本对应关系：

- 65.0 = Java 21
- 69.0 = Java 25
- 70.0 = Java 26

解决方法：

- 升级构建用的 JDK 到 Java 25 或更高版本

> 如果你使用的是 Arch Linux，并且安装了多个 Java。那么你可以使用`archlinux-java`命令来切换 Java

修改 Java 版本后，建议用`mvn -version`命令确认 Maven 实际使用的 JDK。如：

```bash
mvn -version
Apache Maven 3.9.16 (2bdd9fddda4b155ebf8000e807eb73fd829a51d5)
Maven home: /usr/share/java/maven
Java version: 26.0.1, vendor: Arch Linux, runtime: /usr/lib/jvm/java-26-openjdk
Default locale: zh_CN, platform encoding: UTF-8
OS name: "linux", version: "7.0.11-1-lily", arch: "amd64", family: "unix"
```

