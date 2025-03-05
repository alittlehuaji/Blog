## CoreProtect 构建教程

> CoreProtect 是专门为 Minecraft 插件服务器设计的记录插件，它可以记录玩家对几乎所有方块的操作（包括但不限于破坏、防止以及点击等操作）接下来这篇文章将会介绍如何构建此插件

演示环境：

- `Windows 11 24H2`
- `Apache Maven 3.9.9`
- `OpenJDK 21.0.3`
- `git version 2.43.0.windows.1`

在开始之前，确保你的系统已安装 Java 21、Maven 和 Git Bash。你可以通过以下命令来确认 Java 的版本以及你是否正确安装了 Maven：

```bash
java -version
```

```bash
mvn --version
```

如果你还没安装 Maven，请参考 [如何在 Windows 上安装 Maven](https://phoenixnap.com/kb/install-maven-windows)。


### 1. 克隆源代码

首先，克隆 CoreProtect 的源代码：

```bash
git clone https://github.com/PlayPro/CoreProtect.git
```


### 2. 修改 `pom.xml`

在克隆下来的项目目录中，找到 `pom.xml` 文件，并进行以下修改：

- **第 7 行：** 设置分支为 `master`。

  ```xml
  <project.branch>master</project.branch>
  ```

- **第 5 行：** 设置插件的版本（保持默认也可以）。

  ```xml
  <version>22.4</version>
  ```


### 3. 修改 `build.gradle`

接着找到 `build.gradle` 文件，按如下修改：

- **第 10 行：** 设置项目版本为 `22.4`，与 `pom.xml` 中一致。

  ```gradle
  String projectVersion = '22.4'
  ```

- **第 11 行：** 设置分支为 `master`。

  ```gradle
  String projectBranch = 'master'
  ```


### 4. 构建项目

打开终端，切换到 CoreProtect 项目的目录，运行以下命令进行构建：

```bash
mvn clean install
```


### 5. 完成构建

如果一切顺利，你将在终端看到类似以下的日志输出：

```
[INFO]
[INFO] BUILD SUCCESS
[INFO]
[INFO] Total time:  13.201 s
[INFO] Finished at: 2024-10-06T15:20:20+08:00
[INFO]
```

`BUILD SUCCESS` 表示构建成功。 你可以在 `target` 目录下找到 `CoreProtect-xx.x.jar` 文件（不要使用 `original-CoreProtect-xx.x.jar`）。将该文件放入服务器的 `plugins` 目录，重启后即可安装插件。

