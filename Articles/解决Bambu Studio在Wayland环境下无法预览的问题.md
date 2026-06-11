# 解决Bambu Studio在Wayland环境下无法预览的问题

## 引言

如果你也是 ArchLinux的忠实用户，并且正好拥有一台拓竹3D打印机，那么你大概率经历过这种绝望：好不容易在 Wayland 环境下跑通了所有依赖，打开Bambu Studio准备切片，却发现模型预览区像黑洞一样深邃（什么都看不见😅）

在 GNU/Linux + Wayland + NVIDIA 这套 “折腾三剑客”的组合中，3D软件的预览失败几乎是必须经历的洗礼。虽然 Github Issue 上有很多关于 XWayland 的临时解决方案，但是针对于**双显卡的笔记本用户**，其实有一个更加优雅、更加底层的暴力解法。不需要切回 X11,也不需要折腾复杂的补丁，只需要一行环境变量，就可以让模型重新出现在屏幕上

## 解决方案

演示环境

- `Archlinux`
- `Niri 25.11 (Wayland)`
- `GPU 1 NVIDIA GeForce RTX 4060 Max-Q / Mobile`
- `GPU 2 Intel Iris Xe Graphics`
- `Bambu Studio 2.5.0.66 (AUR bambu-studio-bin)`

> 本文方法主要适用于
>
> 1. **NVIDIA 独显 + Intel /AMD 核显** 的双显卡用户
> 2. **Wayland会话环境**

首先使用自定义环境变量来运行BambuStudio

```bash
env __GLX_VENDOR_LIBRARY_NAME=mesa __EGL_VENDOR_LIBRARY_FILENAMES=/usr/share/glvnd/egl_vendor.d/50_mesa.json MESA_LOADER_DRIVER_OVERRIDE=zink GALLIUM_DRIVER=zink WEBKIT_DISABLE_DMABUF_RENDERER=1 /opt/bambustudio-bin/AppRun "$@"
```

命令解释

| **组成部分**                               | **详细作用**                                                                                                    |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| **`env`**                                | 这是一个用于设置环境变量并执行程序的工具。它确保接下来的变量仅对这个特定程序生效，而不会影响全局系统。                |
| **`__GLX_VENDOR_LIBRARY_NAME=mesa`**     | 强制**GLX**（OpenGL 与 X 窗口系统的接口）使用 Mesa 开源驱动，而不是 NVIDIA 或其他闭源驱动。                     |
| **`__EGL_VENDOR_LIBRARY_FILENAMES=...`** | 指定**EGL** 加载器的配置文件路径，强制它指向 Mesa 的配置文件。这通常用于解决多显卡或驱动冲突问题。              |
| **`MESA_LOADER_DRIVER_OVERRIDE=zink`**   | **核心参数。** 告诉 Mesa 加载器：忽略显卡本身的硬件驱动，改用名为 **Zink** 的驱动。                       |
| **`GALLIUM_DRIVER=zink`**                | 与上一条类似，进一步确保 Mesa 的 Gallium3D 架构使用**Zink** 后端。                                              |
| **`WEBKIT_DISABLE_DMABUF_RENDERER=1`**   | 禁用 WebKit（浏览器内核）的 DMA-BUF 渲染。Bambu Studio 的内嵌页面经常因为这个设置导致黑屏或闪烁，禁用它能提高兼容性。 |
| **`"/opt/bambustudio-bin/AppRun"`**      | Bambu Studio 实际的可执行文件路径。                                                                                   |
| **`"$@"`**                               | Shell 占位符，代表你传递给这个脚本的所有后续参数（例如你打开特定的 `.3mf` 文件）。                                  |

虽然在终端输入命令很酷，但是每次需要使用的时候都需要打开终端。这显然不太符合“优雅”的定义，那么接下来我们可以通过修改 `.desktop` 文件来让系统每次启动 BambuStudio 的时候都可以自动带上这个参数

### 第一步：拷贝配置文件

为了防止在系统更新的时候把配置覆盖掉，这里可以将全局配置拷贝到用户目录下

```bash
mkdir -p ~/.local/share/applications/
cp /usr/share/applications/BambuStudio.desktop ~/.local/share/applications/
```

### 第二步：编辑 Exec 启动项

```bash
vim ~/.local/share/applications/BambuStudio.desktop
```

找到 `Exec=`这一行，将后面改成如下内容

```bash
env __GLX_VENDOR_LIBRARY_NAME=mesa __EGL_VENDOR_LIBRARY_FILENAMES=/usr/share/glvnd/egl_vendor.d/50_mesa.json MESA_LOADER_DRIVER_OVERRIDE=zink GALLIUM_DRIVER=zink WEBKIT_DISABLE_DMABUF_RENDERER=1 /opt/bambustudio-bin/AppRun %F
```

> 这里需要注意的是，在末尾需要将原有的 `"$@"`改为 `%F`，因为 `"$@"`在 `.desktop`文件中是无效的
> `%F`表示传递**多个**本地文件路径

完成编辑后使用命令 `wq`保存并退出

### 第三步：保存并加载配置

接下来你可以使用桌面环境重新加载一下配置

```bash
update-desktop-database ~/.local/share/applications/
```

现在从系统的应用启动器启动BambuStudio就不会出现预览黑屏的情况了

## 参考来源

https://github.com/bambulab/BambuStudio/issues/2595

https://github.com/bambulab/BambuStudio/issues/6066#issuecomment-2728152079

https://gemini.google.com/
