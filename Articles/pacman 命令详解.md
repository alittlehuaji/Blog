## 前言

为了保证写作效率，本文在资料整理以及写作阶段使用了生成式人工智能来协助。如有谬误，欢迎来[Github issue](https://github.com/alittlehuaji/Blog/issues)或者博客评论区留言指出

阅读指北：

- 操作指南主要分为两个部分，分别是“常用操作”和“选项详解”。一般情况下去看“常用操作”部分就已经足够了
- 本文篇幅较长，请善用“大纲”以及“Ctrl+F”来完成快速跳转和查找

> 博客网页在某些地方可能存在布局问题导致难以阅读的情况。如有必要，可以前往 [Github 仓库](https://github.com/alittlehuaji/Blog)阅读 Markdown 原文

----

演示环境

- `pacman 7.1.0`
- `Arch Linux`

## pacman 基本结构

pacman 命令大体是：

```bash
pacman <操作> [选项] [软件包]
```

常见操作：

| 操作                | 含义                             |
| ------------------- | -------------------------------- |
| `-S` / `--sync`     | 从同步仓库安装、搜索、升级软件包 |
| `-Q` / `--query`    | 查询本地已安装软件包             |
| `-R` / `--remove`   | 删除软件包                       |
| `-U` / `--upgrade`  | 安装本地软件包文件               |
| `-F` / `--files`    | 查询某个文件属于仓库里的哪个包   |
| `-D` / `--database` | 修改本地数据库标记               |
| `-T` / `--deptest`  | 检查依赖是否满足                 |

pacman 的选项是可以组合的，就拿下面这个命令来举例：

```bash
sudo pacman -Syu
```

这里：

- `-S`：操作模式，表示sync，用于同步软件包数据库
- `-y`：刷新软件包数据库
- `-u`：升级系统中所有可以升级的软件包

所以`-Syu`并不是一个神秘的命令，而是：

```bash
sudo pacman -S -y -u
```

的简写

这里也可以使用*长选项*

```bash
sudo pacman --sync --refresh --sysupgrade
```

如果你注意力惊人，***你会发现第一个大写的字母通常表示“操作模式”，后面的小写字母是这个操作模式下的选项***

所以说

- `操作模式`决定了 pacman 应该干什么
- `操作选项`是对这个操作模式的补充说明

### 实践

查看 pacman 可用的操作：

```bash
pacman -h
```

查看某个操作的可用选项：

```bash
pacman -操作h
```

比如我想查看`-Q`操作的可用选项就可以使用：

```bash
pacman -Qh
```

## 常用命令速查表

| 命令                               | 目的                 |
| ---------------------------------- | -------------------- |
| `sudo pacman -Syu`                 | 更新系统             |
| `sudo pacman -S 包名`              | 安装软件             |
| `sudo pacman -Syu 包名`            | 更新系统并安装软件   |
| `sudo pacman -R 包名`              | 删除软件             |
| `sudo pacman -Rs 包名`             | 删除软件和无用依赖   |
| `sudo pacman -Rns 包名`            | 更彻底删除           |
| `pacman -Ss 关键词`                | 搜索仓库软件         |
| `pacman -Qs 关键词`                | 搜索已安装软件       |
| `pacman -Si 包名`                  | 查看仓库包信息       |
| `pacman -Qi 包名`                  | 查看已安装包信息     |
| `pacman -Ql 包名`                  | 查看包安装了哪些文件 |
| `pacman -Qo 文件路径`              | 查询文件属于哪个包   |
| `pacman -Qdt`                      | 查询孤儿包           |
| `sudo pacman -Rns $(pacman -Qdtq)` | 删除孤儿包           |
| `sudo pacman -U ./xxx.pkg.tar.zst` | 安装本地包           |
| `sudo pacman -Sc`                  | 清理旧缓存           |

## 操作选项详解

### `-S` / `--sync`

这个应该是最常用的操作了。`pacman -S`的核心意思是：从软件仓库同步、安装、升级软件包

#### 常用操作

##### 安装软件包

```bash
sudo pacman -S firefox
```

安装`firefox`

##### 升级系统

```bash
sudo pacman -Syu
```

刷新数据库并升级整个系统

##### 搜索远程仓库

```bash
pacman -Ss neovim
```

搜索远程仓库里`neovim`相关的软件包

##### 查看远程仓库某个包的详细信息

```bash
pacman -Si neovim
```

查看远程仓库中`neovim`的详细信息

##### 安装包，如果最近就跳过

```bash
sudo pacman -S --needed git base-devel
```

安装`git`和`base-devel`，如果已经是最新的就跳过

##### 只下载但不安装

```bash
sudo pacman -Sw linux
```

仅下载`linux`包，不安装

##### 清理缓存

```bash
sudo pacman -Sc
```

清理旧版本软件包缓存

##### 升级系统但忽略某个包

```bash
sudo pacman -Syu --ignore linux
```

升级系统，但暂时不升级`linux`内核包

> 注意：Arch 系的发行版一般不推荐单独升级某个或某些软件包，这样做可能会损坏系统

#### 选项解释

##### 基本同步/安装选项

| 选项                     | 作用                                                         | 例子                                                         |
| ------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-y`, `--refresh`        | 刷新软件包数据库，也就是从镜像源下载最新仓库数据库。`-yy` 会强制刷新，即使本地的数据库已经是最新 | `sudo pacman -Sy` 只刷新数据库；升级系统时更推荐：`sudo pacman -Syu` |
| `-u`, `--sysupgrade`     | 升级所有已安装的软件包。通常和 `-y` 一起用。`-uu` 允许降级软件包，例如镜像源版本低于本地版本时 | `sudo pacman -Syu` 升级整个系统；`sudo pacman -Syuu` 允许降级并升级系统 |
| `-s`, `--search <regex>` | 在远程软件仓库中搜索软件包，支持正则表达式                   | `pacman -Ss firefox`                                         |
| `-i`, `--info`           | 查看远程仓库中的软件包信息。`-ii` 显示更详细信息             | `pacman -Si firefox`；`pacman -Sii linux`                    |
| `-l`, `--list <repo>`    | 查看某个软件仓库里的软件包列表                               | `pacman -Sl extra`                                           |
| `-g`, `--groups`         | 查看软件包组。指定组名时列出该组中的包；`-gg` 列出所有组及其包 | `pacman -Sg gnome`；`pacman -Sgg`                            |
| `-w`, `--downloadonly`   | 只下载软件包，不安装或升级                                   | `sudo pacman -Sw firefox`                                    |
| `-p`, `--print`          | 只打印将要处理的目标，不执行安装/升级。常用于预览            | `pacman -Sp firefox`                                         |
| `-q`, `--quiet`          | 安静输出，减少信息。常和搜索、列表搭配，只显示包名           | `pacman -Ssq firefox`                                        |

##### 安装行为控制

| 选项                               | 作用                                                         | 例子                                                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `--needed`                         | 如果目标软件包已经是最新版本，就跳过，不重新安装。比较适合脚本里批量装软件 | `sudo pacman -S --needed git base-devel neovim`              |
| `--asdeps`                         | 把安装的软件包标记为“作为依赖安装”。但如果没有包依赖它，可能会被 `pacman -Qdt` 识别为孤儿包 | `sudo pacman -S --asdeps cmake`                              |
| `--asexplicit`                     | 把安装的软件包标记为“用户显式安装”。这样它不会被当成孤儿依赖清理 | `sudo pacman -S --asexplicit firefox`                        |
| `-d`, `--nodeps`                   | 跳过依赖检查。`-d` 跳过依赖版本检查，`-dd` 跳过所有依赖检查。**这个操作非常容易搞坏系统** | `sudo pacman -Sd somepkg`                                    |
| `--assume-installed <软件包=版本>` | 临时假装某个包已经安装，用来满足依赖关系。不会真的安装那个包。慎用 | `sudo pacman -S somepkg --assume-installed 'java-runtime=21'` |
| `--dbonly`                         | 只修改 pacman 数据库，不实际解包安装文件。常用于修复数据库状态，普通用户不该随便用 | `sudo pacman -S --dbonly somepkg`                            |
| `--noscriptlet`                    | 不执行软件包自带的安装/升级脚本，例如 `post_install`、`post_upgrade`。危险，可能导致软件配置不完整 | `sudo pacman -S somepkg --noscriptlet`                       |
| `--overwrite <glob>`               | 允许覆盖已经存在且冲突的文件。`glob` 是匹配模式，建议加引号防止 shell 展开。危险，先确认冲突原因再用 | `sudo pacman -S somepkg --overwrite '/usr/bin/foo'`          |

##### 缓存清理

| 选项                | 作用                                                         | 例子                                                |
| ------------------- | ------------------------------------------------------------ | --------------------------------------------------- |
| `-c`, `--clean`     | 清理软件包缓存。`-Sc` 删除旧版本缓存，保留当前已安装版本；`-Scc` 清除所有缓存，比较激进。 | `sudo pacman -Sc`；彻底清理：`sudo pacman -Scc`     |
| `--cachedir <目录>` | 指定额外或替代的软件包缓存目录。适合临时用别的磁盘存包。     | `sudo pacman -Syu --cachedir /mnt/cache/pacman/pkg` |

##### 忽略升级

| 选项                       | 作用                                                       | 例子                                     |
| -------------------------- | ---------------------------------------------------------- | ---------------------------------------- |
| `--ignore <软件包>`        | 系统升级时忽略指定软件包，不升级它。可以多次使用或逗号分隔 | `sudo pacman -Syu --ignore linux,nvidia` |
| `--ignoregroup <软件包组>` | 系统升级时忽略某个软件包组                                 | `sudo pacman -Syu --ignoregroup gnome`   |

### `-Q` / `--query`

这个操作主要用于查询本机已经安装的软件包

#### 常用操作

##### 查看已经安装的包

```bash
pacman -Q
```

查看所有已经安装的包

> 如果只想看包名，可以用`-Qq`

##### 查看本地包详细信息

```bash
pacman -Qi firefox
```

查看本地已经安装的`firefox`的详细信息

> 如果嫌不够详细可以使用`-Qii`

##### 搜索本地已安装包

```bash
pacman -Qs vim
```

搜索本地已经安装的软件包关键词

> 这里和`-Ss`的区别是一个是搜索远程仓库，一个是搜索本地已经安装的软件包

##### 查看本地包安装到系统的文件

```bash
pacman -Ql vim
```

查看本地`vim`包安装到系统里的所有文件，类似这样

```bash
$ pacman -Ql bash
bash /etc/
bash /etc/bash.bash_logout
bash /etc/bash.bashrc
```

> 如果你只想看文件的路径而不显示包名的话，可以使用`-Qql`

##### 查询某个文件属于哪个本地包

```bash
pacman -Qo /usr/bin/bash
```

查询`/usr/bin/bash`这个文件属于哪个包

> 这个命令还蛮有用的，比如你不知道某个命令是哪个包提供的
>
> ``` bash
> $ pacman -Qo /usr/bin/bash
> /usr/bin/bash 由 bash 5.3.15-1 所拥有
> ```

##### 查看本地手动安装的包

```bash
pacman -Qe
```

查看手动安装的软件包

> 如果你只想看包名的话可以使用`-Qqe`
>
> 这个操作经常用于备份软件包列表，比如你可以这样做
>
> ```bash
> pacman -Qqe > pkglist.txt
> ```
>
> 后续需要恢复的时候
>
> ```bash
> sudo pacman -S --needed - < pkglist.txt
> ```

##### 查看本地作为依赖安装的包

```bash
pacman -Qd
```

查看作为依赖安装的软件包

> 只看包名可以使用`-Qqd`

##### 列出孤儿依赖包

```bash
pacman -Qdt
```

列出不再被需要的依赖包（孤儿包）

含义：

- `-d`：只看作为依赖安装的包
- `-t`：只看不再被其他包依赖的包

> 只显示包名可以用`-Qdtq`
>
> 如果你想一次性删掉这些孤儿包，你可以用
>
> ```bash
> sudo pacman -Rns $(pacman -Qdtq)
> ```

##### 查看外部包

```bash
pacman -Qm
```

查看外来包，也就是不在当前同步数据库里的包

常见情况：

- Aur 安装的包
- 自己用`pacman -U`安装的本地包
- 曾经在仓库中但现在已经被移除的软件包
- 第三方仓库关闭后遗留的包

> 只显示包名可以用`-Qqm`

##### 查看通过远程仓库安装的包

```bash
pacman -Qn
```

`-n`表示 native，也就是在当前同步数据中的包

> 只显示包名可以用`-Qqn`

##### 查看可升级的包

```bash
pacman -Qu
```

列出有新版可升级的已安装的包

##### 检查软件包文件是否有缺失

```bash
pacman -Qk bash
```

检查本地`bash`包是否有文件缺失

> 如果你想让他更严格点检查可以用`-Qkk`
>
> 不过有时候不过 `-Qkk` 可能会报告一些配置文件被修改，这不一定是错误，因为很多 `/etc` 下配置本来就会被用户修改

##### 查询本地软件包文件信息

假如你下载了一个包文件

```bash
cider-v4.0.0-linux-x64.pkg.tar.zst
```

你想查看它的信息

```bash
pacman -Qip cider-v4.0.0-linux-x64.pkg.tar.zst
```

查看这个包会安装哪些文件

```bash
pacman -Qlp cider-v4.0.0-linux-x64.pkg.tar.zst
```

这里的`-p`很重要，表示查询的是包文件，而不是本地数据库

##### 查看软件包组

```bash
pacman -Qg gnome
```

查看`gnome`组中已安装的包

> 只显示包名可以用`-Qqg`

#### 选项解释

##### 基本查询与信息查看

| 选项                         | 作用                                                         | 例子                                         |
| ---------------------------- | ------------------------------------------------------------ | -------------------------------------------- |
| `-i`, `--info`               | 查看已安装软件包的详细信息；使用 `-ii` 会额外显示备份文件相关信息 | `pacman -Qi pacman`；`pacman -Qii pacman`    |
| `-l`, `--list`               | 列出指定已安装软件包包含的文件                               | `pacman -Ql pacman`                          |
| `-c`, `--changelog`          | 查看某个已安装软件包的更新日志；是否有内容取决于该包是否提供 changelog | `pacman -Qc linux`                           |
| `-g`, `--groups`             | 查看软件包组信息；可列出某个组里的包，或查看某个包所属的组   | `pacman -Qg base-devel`；`pacman -Qg pacman` |
| `-k`, `--check`              | 检查已安装软件包的文件是否存在；`-kk` 会进一步检查文件属性，如权限、大小、修改时间等 | `pacman -Qk pacman`；`pacman -Qkk pacman`    |
| `-o <file>`, `--owns <file>` | 查询某个文件属于哪个已安装软件包                             | `pacman -Qo /usr/bin/pacman`                 |

##### 按安装原因和来源过滤

| 选项                 | 作用                                                         | 例子                                        |
| -------------------- | ------------------------------------------------------------ | ------------------------------------------- |
| `-d`, `--deps`       | 只列出作为依赖关系安装的软件包；常和 `-t` 搭配查孤儿包       | `pacman -Qd`；`pacman -Qdt`                 |
| `-e`, `--explicit`   | 只列出明确手动安装的软件包                                   | `pacman -Qe`                                |
| `-m`, `--foreign`    | 只列出不在同步数据库中的已安装软件包，常见于 `AUR` 包或本地安装包 | `pacman -Qm`                                |
| `-n`, `--native`     | 只列出存在于同步数据库中的已安装软件包，也就是通常来自官方仓库的包 | `pacman -Qn`                                |
| `-t`, `--unrequired` | 列出不被任何软件包需要的软件包；`-tt` 会忽略可选依赖，只看强制依赖关系 | `pacman -Qt`；`pacman -Qdt`；`pacman -Qdtt` |
| `-u`, `--upgrades`   | 列出所有可升级的软件包；结果依赖本地同步数据库是否已刷新     | `pacman -Qu`                                |

##### 搜索、安静输出与详细输出

| 选项                             | 作用                                                   | 例子                                                         |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| `-s <regex>`, `--search <regex>` | 在已安装的本地软件包中按正则表达式搜索名称和描述       | `pacman -Qs firefox`；`pacman -Qs '^linux'`                  |
| `-q`, `--quiet`                  | 查询或搜索时减少输出信息，通常只显示包名，适合脚本使用 | `pacman -Qq`；`pacman -Qdtq`；`pacman -Qeq`                  |
| `-v`, `--verbose`                | 显示更详细的信息，适合排查问题                         | `pacman -Qv`；`pacman -Qiv pacman`                           |
| `--color <when>`                 | 控制彩色输出，常见值有 `auto`、`always`、`never`       | `pacman -Qs firefox --color auto`；`pacman -Q --color never` |

##### 查询本地包文件

| 选项                               | 作用                                                         | 例子                                                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-p <package>`, `--file <package>` | 从一个软件包文件中查询信息，而不是从已安装数据库中查询；常用于查看 `.pkg.tar.zst` 包文件 | `pacman -Qip ./example-1.0-1-x86_64.pkg.tar.zst`；`pacman -Qlp ./example-1.0-1-x86_64.pkg.tar.zst` |

### `-R` / `--remove`

这个操作一般用来卸载已安装的软件包

#### 常用操作

##### 卸载软件包

```bash
sudo pacman -R firefox
```

卸载`firefox`，但不会自动删除它安装时带来的依赖包

##### 卸载软件包并删除依赖

```bash
sudo pacman -Rs firefox
```

卸载`firefox`，并删除只被`firefox`需要的依赖包

##### 卸载软件包、清理依赖并删除配置备份

```bash
sudo pacman -Rns firefox
```

卸载`firefox`的同时删除不再需要的依赖，并且不保留`.pacsave`这类配置备份文件

##### 删除孤儿包

```bash
sudo pacman -Rns $(pacman -Qdtq)
```

删除`pacman -Qdtq`所列出的包，并删除无用的依赖和配置备份

##### 预览将要删除哪些包

```bash
pacman -Rsp firefox
```

打印卸载`firefox`时会涉及包

##### 删除软件包时跳过依赖检查

```bash
sudo pacman -Rdd proken-pkg
```

删除`proken-pkg`时跳过所有依赖检查，通常用于修复系统

> 注意：`-c`、`-dd`、`--dbonly`、`--noscriptlet`都是比较危险的选项，不应该随便使用

#### 选项解释

##### 基本删除行为

| 选项                | 作用                                                         | 例子                                                         |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-c`, `--cascade`   | 级联删除：删除指定软件包，以及所有依赖于它的软件包。**风险很高，可能一次删掉大量软件包，使用前一定要仔细确认列表** | `sudo pacman -Rc icu`；更安全地先预览：`pacman -Rcp icu`     |
| `-n`, `--nosave`    | 删除配置文件，不保留 `.pacsave` 备份；适合想彻底清理某个软件包配置的情况 | `sudo pacman -Rn firefox`；常用组合：`sudo pacman -Rns firefox` |
| `-s`, `--recursive` | 递归删除不再被其他软件包需要的依赖；`-ss` 会连“单独指定安装”的依赖也纳入删除范围，清理更彻底但也更容易误删你手动装过的包 | `sudo pacman -Rs firefox`；更激进：`sudo pacman -Rss firefox` |
| `-u`, `--unneeded`  | 只删除不被其他软件包需要的目标包，常用于避免删掉仍被依赖的软件包 | `sudo pacman -Ru package-name`；组合使用：`sudo pacman -Rsu firefox` |

##### 依赖相关

| 选项                               | 作用                                                         | 例子                                                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-d`, `--nodeps`                   | 跳过依赖关系检查；`-d` 只跳过依赖版本检查，`-dd` 跳过所有依赖检查。**可能导致系统里留下坏掉的依赖关系，除非你知道自己在修什么，否则不建议用** | `sudo pacman -Rd package-name`；强制跳过全部检查：`sudo pacman -Rdd package-name` |
| `--assume-installed <软件包=版本>` | 临时假装某个 `软件包=版本` 已安装，用它来满足依赖要求；主要用于处理特殊依赖场景，不会真的安装这个包 | `sudo pacman -R package-name --assume-installed 'foo=1.0'`   |

##### 输出与显示

| 选项                      | 作用                                                         | 例子                                                         |
| ------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-p`, `--print`           | 只打印将要处理的目标，而不真正执行删除操作；适合在危险操作前预览 | `pacman -Rp firefox`；预览递归删除：`pacman -Rsp firefox`    |
| `--print-format <字符串>` | 配合 `--print` 使用，指定打印目标时的格式                    | `pacman -Rp firefox --print-format '%n %v'`                  |
| `-v`, `--verbose`         | 显示更详细的信息，适合排查卸载过程中的问题                   | `sudo pacman -Rv firefox`                                    |
| `--color <when>`          | 控制彩色输出，`<when>` 常见值有 `auto`、`always`、`never`    | `sudo pacman -R firefox --color auto`；`sudo pacman -R firefox --color never` |
| `--noprogressbar`         | 下载文件时不显示进度条；对普通 `-R` 卸载通常没什么影响，因为卸载一般不需要下载 | `sudo pacman -R firefox --noprogressbar`                     |

### `-U` / `--upgrade`

这个操作用来安装或者升级本地的软件包文件，也常用来从缓存里手动降级软件包。

简单来说就是：不从仓库里按名字安装包，而是直接指定一个`.pkg.tar.zst`包文件来安装

#### 常用操作

##### 安装本地软件包文件

```bash
sudo pacman -U ./cider-v4.0.0-linux-x64.pkg.tar.zst
```

安装当前目录下的`cider-v4.0.0-linux-x64.pkg.tar.zst`软件包文件

##### 从缓存中降级软件包

```bash
sudo pacman -U /var/cache/pacman/pkg/niri-26.04-1-x86_64.pkg.tar.zst
```

把`niri`安装回本地缓存里的`26.04-1`版本

> 这个操作常用于新版本软件包出现问题时临时回退

##### 下载远程软件包并安装

```bash
sudo pacman -U https://example.com/packages/niri-26.04-1-x86_64.pkg.tar.zst
```

从指定`URL`下载并安装`niri-26.04-1`

> 如果你只想下载但不安装的话可以使用`-Uw`

#### 选项解释

##### 基本安装/升级选项

| 选项                   | 作用                                                         | 例子                                                         |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-w`, `--downloadonly` | 只下载软件包，不安装或升级。主要用于`URL`形式的软件包        | `sudo pacman -Uw https://example.com/packages/example-1.0-1-x86_64.pkg.tar.zst` |
| `--needed`             | 如果目标软件包已经安装且版本已经是最新，就不重新安装，适合批量安装本地包时避免重复操作 | `sudo pacman -U --needed ./*.pkg.tar.zst`                    |
| `--asdeps`             | 把软件包标记为“作为依赖安装”，以后如果没有其他包需要它，可能会被`pacman -Qtdq`识别为孤儿包 | `sudo pacman -U --asdeps ./libfoo-1.0-1-x86_64.pkg.tar.zst`  |
| `--asexplicit`         | 把软件包标记为“显式安装”，表示这是用户主动安装的软件包，不会因为没人依赖它就被当作孤儿包 | `sudo pacman -U --asexplicit ./firefox-120.0-1-x86_64.pkg.tar.zst` |

##### 依赖检查与安装脚本控制

| 选项                               | 作用                                                         | 例子                                                         |
| ---------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `-d`, `--nodeps`                   | 跳过依赖关系的版本检查；使用`-dd`会跳过所有依赖检查。**风险很高**，可能装出一个无法正常运行的软件包，甚至破坏系统依赖关系 | `sudo pacman -Ud ./example-1.0-1-x86_64.pkg.tar.zst`；`sudo pacman -Udd ./example-1.0-1-x86_64.pkg.tar.zst` |
| `--assume-installed <软件包=版本>` | 临时假装某个`软件包=版本`已经安装，用它来满足依赖要求。**只影响本次操作的依赖判断，不是真的安装了这个包** | `sudo pacman -U ./example-1.0-1-x86_64.pkg.tar.zst --assume-installed 'foo=1.0'` |
| `--dbonly`                         | 只修改`pacman`数据库记录，不真正安装、升级或删除软件包文件。**非常危险**，会让数据库状态和实际文件状态不一致，一般只用于修复特殊问题 | `sudo pacman -U --dbonly ./example-1.0-1-x86_64.pkg.tar.zst` |
| `--noscriptlet`                    | 不执行软件包自带的安装脚本，比如安装后钩子、用户创建、缓存刷新等可能不会执行。**可能导致软件包配置不完整** | `sudo pacman -U --noscriptlet ./example-1.0-1-x86_64.pkg.tar.zst` |
| `--overwrite <glob>`               | 允许覆盖与其他软件包冲突的文件，`<glob>`可以指定文件匹配模式，可多次使用。**风险很高**，可能覆盖其他包拥有的文件 | `sudo pacman -U ./example-1.0-1-x86_64.pkg.tar.zst --overwrite '/usr/bin/example'`；`sudo pacman -U ./example.pkg.tar.zst --overwrite '*'` |

### `-D` / `--database`

`-D`一般用于直接操作`pacman`的本地数据库，最常见的用法是更改软件包的安装原因

#### 常用操作

##### 把软件包标记为依赖安装

```bash
sudo pacman -D --asdeps firefox
```

把`firefox`标记为“作为依赖安装”的软件包

> 注意：`pacman -D`是直接改本地数据库的操作，它不会检查你这样做是否合理，乱用`--asdeps`可能导致重要软件被`pacman -Qdt`识别成孤儿包

##### 把软件包标记为手动安装

```bash
sudo pacman -D --asexplicit firefox
```

把`firefox`标记为“手动安装”的软件包，这样做可以避免它被当成孤儿包清理

##### 检查本地软件包数据库

```bash
pacman -Dk
```

一般用于检查本地`pacman`数据库的有效性

> 如果你想让它检查更严格一点可以使用`-Dkk`

#### 选项解释

##### 数据库检查

| 选项            | 作用                                                         | 例子                        |
| --------------- | ------------------------------------------------------------ | --------------------------- |
| `-k`, `--check` | 检查本地数据库有效性；使用`-kk`会进行更严格的检查，并结合同步数据库一起检查 | `pacman -Dk`；`pacman -Dkk` |

##### 安装原因标记

| 选项           | 作用                                                         | 例子                                  |
| -------------- | ------------------------------------------------------------ | ------------------------------------- |
| `--asdeps`     | 把指定软件包标记为“非明确指定安装”，也就是作为依赖安装。**如果乱标记，之后可能被当成孤儿包清理掉** | `sudo pacman -D --asdeps firefox`     |
| `--asexplicit` | 把指定软件包标记为“明确指定安装”，也就是用户手动安装。常用于防止重要软件被识别为孤儿包 | `sudo pacman -D --asexplicit firefox` |

### `-T` / `--deptest`

`-T`一般用于检查指定的依赖是否已经满足

#### 常用操作

##### 检查某些依赖是否已安装

```bash
pacman -T bash glibc
```

检查`bash`和`glibc`这两个依赖是否已经满足，缺哪个就输出哪个

##### 检查带版本要求的依赖

```bash
pacman -T 'python>=3.11'
```

检查系统里是否有满足`python>=3.11`要求的`python`包

##### 在脚本中检查缺失依赖

```bash
missing=$(pacman -T git base-devel)
```

把缺失的`git`和`base-devel`依赖保存到`missing`变量里，方便脚本后续处理

##### 检查另一个系统根目录里的依赖

```bash
sudo pacman -T bash glibc --root /mnt
```

检查`/mnt`这个系统根目录里是否满足`bash`和`glibc`依赖

#### 选项解释

##### 依赖检查与输出信息

| 选项              | 作用                                                   | 例子                        |
| ----------------- | ------------------------------------------------------ | --------------------------- |
| `-v`, `--verbose` | 显示更详细的信息，适合排查为什么某个依赖判断不符合预期 | `pacman -Tv 'python>=3.11'` |

## 通用选项

以下选项是所有操作（`-S`、`-Q`、`-R`、`-U`、`-D`、`-T`）共有的，因此单独列出

### 路径、根目录与配置文件

| 选项                    | 作用                                                         | 例子                                                  |
| ----------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| `-r`, `--root <路径>`   | 指定另一个安装根目录，让 pacman 以该路径作为系统根目录操作   | `sudo pacman -Syu --root /mnt`                        |
| `-b`, `--dbpath <路径>` | 指定 pacman 数据库位置，默认 `/var/lib/pacman`               | `sudo pacman -Syu --dbpath /mnt/var/lib/pacman`       |
| `--config <路径>`       | 使用指定的 pacman 配置文件，默认 `/etc/pacman.conf`          | `sudo pacman -Syu --config /tmp/pacman.conf`          |
| `--gpgdir <路径>`       | 指定 GnuPG 密钥目录，默认 `/etc/pacman.d/gnupg`             | `sudo pacman -Syu --gpgdir /etc/pacman.d/gnupg`       |
| `--hookdir <目录>`      | 指定 pacman hook 钩子目录，可多次使用                        | `sudo pacman -Syu --hookdir /etc/pacman.d/hooks`      |
| `--logfile <路径>`      | 指定 pacman 日志文件路径，默认 `/var/log/pacman.log`         | `sudo pacman -Syu --logfile /tmp/pacman.log`          |
| `--cachedir <目录>`     | 指定软件包缓存目录，默认 `/var/cache/pacman/pkg`             | `sudo pacman -Syu --cachedir /mnt/cache/pacman/pkg`   |
| `--sysroot <路径>`      | 指定替代根文件系统路径，让 pacman 将该目录视为系统根目录 `/` | `sudo pacman -Syu --sysroot /mnt`                     |

### 输出与调试

| 选项              | 作用                                                         | 例子                                          |
| ----------------- | ------------------------------------------------------------ | --------------------------------------------- |
| `-v`, `--verbose` | 显示更详细的信息，例如仓库、下载、依赖解析等                 | `sudo pacman -Syu -v`                         |
| `--color <when>`  | 控制彩色输出。常见值：`auto`、`always`、`never`              | `pacman -Ss linux --color=always`             |
| `--noprogressbar` | 下载时不显示进度条，适合脚本、日志、CI 环境                  | `sudo pacman -Syu --noprogressbar`            |
| `--print-format <字符串>` | 配合 `-p` 使用，指定打印格式。常用占位符：`%n` 包名、`%v` 版本、`%r` 仓库、`%l` 下载地址、`%s` 大小 | `pacman -Sp firefox --print-format '%r/%n %v'` |
| `--debug`         | 输出调试信息，一般用于排查 pacman 问题                       | `sudo pacman -Syu --debug 2> pacman-debug.log` |

### 行为确认

| 选项          | 作用                                                         | 例子                               |
| ------------- | ------------------------------------------------------------ | ---------------------------------- |
| `--confirm`   | 总是询问确认，用于覆盖配置里的 `NoConfirm` 行为              | `sudo pacman -S firefox --confirm` |
| `--noconfirm` | 不询问确认，自动回答默认选项。脚本里常见，但日常操作不建议盲用 | `sudo pacman -Syu --noconfirm`     |

### 下载与沙盒

| 选项                           | 作用                                                         | 例子                                            |
| ------------------------------ | ------------------------------------------------------------ | ----------------------------------------------- |
| `--disable-download-timeout`   | 禁用较严格的下载超时限制。网络慢或镜像源响应慢时可以用       | `sudo pacman -Syu --disable-download-timeout`   |
| `--disable-sandbox`            | 禁用 pacman 的沙盒安全机制。一般在容器、特殊环境中遇到兼容性问题时才使用 | `sudo pacman -Syu --disable-sandbox`            |
| `--disable-sandbox-filesystem` | 只禁用下载沙盒的文件系统隔离部分                             | `sudo pacman -Syu --disable-sandbox-filesystem` |
| `--disable-sandbox-syscalls`   | 只禁用下载沙盒的系统调用限制部分                             | `sudo pacman -Syu --disable-sandbox-syscalls`   |

### 架构

| 选项            | 作用                                       | 例子                             |
| --------------- | ------------------------------------------ | -------------------------------- |
| `--arch <架构>` | 指定目标架构，而不是使用配置文件中的架构   | `sudo pacman -Syu --arch x86_64` |

## 实际使用场景

### 系统急救：chroot 修复

当系统无法正常启动时，可以从 Live USB 启动，挂载分区后用 pacman 修复：

```bash
# 挂载根分区
mount /dev/sda2 /mnt

# chroot 进入系统
arch-chroot /mnt

# 用 pacman 重新安装出问题的包
pacman -S linux linux-headers

# 或者回滚某个包到缓存中的旧版本
pacman -U /var/cache/pacman/pkg/linux-6.x.x-1-x86_64.pkg.tar.zst
```

### 查找某个命令属于哪个包

```bash
# 查找 /usr/bin/vim 属于哪个包
pacman -Qo /usr/bin/vim

# 查找一个尚未安装的文件属于哪个包（需要安装 pkgfile）
pkgfile vim
```

### 批量安装软件

```bash
# 从文件读取包名批量安装
pacman -S --needed - < pkglist.txt

# 或一行命令
pacman -S --needed firefox neovim docker git base-devel
```

### 清理系统垃圾

```bash
# 查看哪些包是孤儿包（无被依赖）
pacman -Qdt

# 删除孤儿包
sudo pacman -Rns $(pacman -Qdtq)

# 清理旧版本缓存，只保留当前安装版本
sudo pacman -Sc

# 彻底清空缓存（不推荐，除非空间紧张）
sudo pacman -Scc
```

### 处理密钥过期问题

```bash
# 初始化 pacman 密钥
sudo pacman-key --init

# 从 Arch Linux 密钥服务器获取密钥
sudo pacman-key --populate archlinux

# 手动刷新密钥
sudo pacman-key --refresh-keys
```

### 解决文件冲突

```bash
# 遇到 "exists in filesystem" 错误时
# 先查看是哪个包拥有这个文件
pacman -Qo /path/to/conflict/file

# 然后强制覆盖安装
sudo pacman -S 包名 --overwrite /path/to/conflict/file
```

### 使用 pacman 搜索未安装的包

```bash
# 搜索仓库中包含关键词的包
pacman -Ss "keyword"

# 只搜索包名，不搜索描述
pacman -Ss --regex "^keyword"
```

