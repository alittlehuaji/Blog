## 设置 SSH 通过密钥登录

~~突然发现好久都没发文章了，今天就随便水一篇吧（）~~

如果你经常需要远程连接服务器，那你肯定离不开 SSH（全称 Secure Shell）。不过，传统的密码登录方式其实有不少的麻烦，比如**容易被爆破**、**输密码时容易输错还麻烦**......
那么有没有更好的方法呢？当然！那就是使用密钥对登录。它相当于为你和服务器配了一把专属的数字钥匙串，登录时，系统会自动用这对钥匙验证身份，无需再输密码！这样不仅**省时省力**，安全性也**大大提升**！（你总不可能爆破我密钥吧？😂👉）

通常情况下，我们并不推荐直接在服务器上生成密钥对。私钥（`id_ed25519` 或 `id_rsa`）是登录凭证，如果在服务器上生成私钥，还需要手动下载到本地，**过程中可能泄露私钥**（如通过不安全的传输方式）

演示环境：

- `Debian GNU/Linux 12 (bookworm) x86_64`

（本篇文章将默认读者已经成功安装 SSH，并且已经可以登录服务器。**以下在服务器上操作的步骤均使用 root 用户执行命令**）

### 正文步骤

1. 在**本地打开终端**并生成密钥对

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"  # 这里比较推荐ed25519算法
   # 或使用RSA：ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   - 按提示设置密钥存储路径（默认 `~/.ssh/id_ed25519`）和密钥密码（Passphrase）。设置密钥密码可以增加安全性，但每次使用密钥时需要输入该密码；如果你追求免密登录，也可以留空
   - 如果你是 `Windows` 用户的话，密钥将默认保存在 `C:\Users\User\.ssh\id_ed25519`（这里的 `User` 需要替换为你的计算机用户名）

     如果不出意外，那么你将会在 `~/.ssh/`目录下看到生成的密钥

     - `id_ed25519` 私钥，应当保密
     - `id_ed25519.pub` 公钥，用来验证私钥
2. 将公钥上传到服务器

   如果本地是 `Linux`/`macOS`，并且已经安装 `ssh-copy-id`，可以直接使用下面的命令自动安装公钥：

   ```bash
   # 这里需要将server替换为服务器的IP地址
   ssh-copy-id -i ~/.ssh/id_ed25519.pub root@server
   ```

   如果没有 `ssh-copy-id`，也可以手动上传。上传前可以先确保服务器上的 `~/.ssh` 目录存在：

   ```bash
   ssh root@server "mkdir -p ~/.ssh && chmod 700 ~/.ssh"
   ```

   ```bash
   # 这里需要将server替换为服务器的IP地址
   scp ~/.ssh/id_ed25519.pub root@server:~/.ssh/
   ```

   如果你是 `Windows` 用户的话

   ```cmd
   # 这里的User需要替换为你的计算机用户名
   scp "C:\Users\User\.ssh\id_ed25519.pub" root@server:~/.ssh/
   ```

3. 将公钥添加到 `authorized_keys`

   如果你使用的是 `ssh-copy-id`，这一步可以跳过

   接下来**新建一个 SSH 会话**。如果刚才不出意外的话，你的公钥就已经上传到 `~/.ssh` 目录下了

   ```bash
   # ls ~/.ssh
   authorized_keys  id_ed25519.pub
   ```

   安装公钥

   ```bash
   cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
   ```

   > 执行这一步之前可能需要先确认 `sshd_config`中是否设置了 `AuthorizedKeysFile     .ssh/authorized_keys`
   >
   > 此选项一般默认存在
   >

   设置权限

   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/authorized_keys
   ```

4. 修改 `SSH`服务端配置文件

   为了确保可以正确使用密钥连接到服务器，这里可能需要对 `sshd_config` 进行一些修改。修改之前，记得先对此配置文件进行一次备份

   > **注意：修改 SSH 配置前，不要关闭当前 SSH 会话。** 先保留一个已经登录的窗口，等新终端确认密钥登录成功后，再关闭旧会话。这样即使配置写错，也还有机会回滚。

   ```bash
   cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
   # 使用编辑器修改配置项
   vim /etc/ssh/sshd_config
   ```

   需要修改的配置项

   ```
   # 公钥验证 yes(启用)/no(关闭)
   PubkeyAuthentication yes
   # 密码验证
   PasswordAuthentication no
   # 交互式认证
   KbdInteractiveAuthentication no
   # root登录 prohibit-password(仅密钥)/no(禁止)/yes(允许)
   PermitRootLogin prohibit-password
   ```

   完成修改后，可以先输入 `sshd -t` 来检查配置文件语法是否正确。如果没有输出，通常就表示语法检查通过

   如果需要查看最终生效的配置，也可以使用 `sshd -T`。如果存在错误，`sshd` 会给出详细的提示，读者可以根据提示进行修改，如：

   ```bash
   # sshd -t
   /etc/ssh/sshd_config: line 14: Bad configuration option: xxxxxxx
   /etc/ssh/sshd_config: terminating, 1 bad configuration options
   ```

   如果没有问题，那么就可以重启 SSH 服务了。`Debian` 下服务名通常是 `ssh`：

   ```bash
   systemctl restart ssh
   ```

   如果你的发行版使用的是 `sshd` 服务名，可以使用：

   ```bash
   systemctl restart sshd
   ```

   重启后，**先别着急关闭当前的 SSH 会话**！这里强烈建议先使用新终端测试密钥登录，确认可以正常连接到服务器后，再关闭旧会话。万一后面密钥和密码都不能登录上服务器的话，那就悲剧了
5. 测试连接

   接下来请打开 **新终端窗口** 测试密钥登录，确认成功后再关闭当前 SSH 会话

   ```bash
   ssh -i ~/.ssh/id_ed25519 root@server
   ```

   如果你是 `Windows` 用户的话

   ```bash
   ssh -i "C:\Users\User\.ssh\id_ed25519" root@server
   ```

   如果登录成功，那么就可以进行下一步了
6. 编辑 `SSH config`

   接下来，可以通过配置 `SSH config` 的方式来进一步简化登录步骤。一般情况下，`SSH config` 都存放在 `~/.ssh` 下（`Windows` 在 `C:\Users\User\.ssh` 目录下）

   ```bash
   vim ~/.ssh/config
   ```

   如果不存在，可以先创建一个

   ```bash
   touch ~/.ssh/config
   ```

   这里提供一份示例配置供读者参考

   ```
   Host myserver
     HostName example.com
     User user
     IdentityFile ~/.ssh/id_ed25519
   ```

   如果你是 `Windows` 用户的话

   ```
   Host myserver
     HostName example.com
     User user
       IdentityFile C:/Users/User/.ssh/id_ed25519
   ```

   - `Host` 别名或主机名
   - `HostName` 实际主机名/IP地址
   - `User` 用户
   - `IdentityFile` 密钥文件路径

   完成配置后，你就可以使用以下命令来连接服务器

   ```bash
   ssh myserver
   ```

### SSH 配置文件解释

为了方便理解，这里挑几个和“密钥登录”关系比较大的配置项简单解释一下。`sshd_config` 里面有很多选项，通常不建议直接照抄整份配置，只需要按自己的需求修改关键项即可

```
# 启用公钥认证
PubkeyAuthentication yes

# 禁用密码认证，强制使用密钥登录
PasswordAuthentication no

# 禁用键盘交互式认证，避免仍然可以通过交互方式输入密码
KbdInteractiveAuthentication no

# root 登录策略：
# prohibit-password 表示允许 root 使用密钥登录，但禁止 root 使用密码登录
# no 表示完全禁止 root 直接登录，更安全，但需要先准备普通用户和 sudo
PermitRootLogin prohibit-password

# 授权公钥文件路径，默认通常就是这个
AuthorizedKeysFile .ssh/authorized_keys

# 严格检查用户家目录、.ssh 目录和 authorized_keys 的权限
StrictModes yes
```

其中需要特别注意的是 `PermitRootLogin`：

- 如果你和本文一样直接使用 `root` 用户登录，可以设置为 `prohibit-password`，这样 `root` 只能通过密钥登录，不能通过密码登录
- 如果你已经创建了普通用户，并且配置好了 `sudo`，更推荐设置为 `no`，彻底禁止 `root` 直接登录

最后再提醒一次：关闭密码登录前，一定要先开一个新终端测试密钥登录是否成功，确认没问题后再关闭旧的 SSH 会话
