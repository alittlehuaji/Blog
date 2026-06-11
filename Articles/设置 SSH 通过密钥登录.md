## 设置 SSH 通过密钥登录

~~突然发现好久都没发文章了，今天就随便水一篇吧（）~~

如果你经常需要远程连接服务器，那你肯定离不开 SSH（全称Secure Shell）。不过，传统的密码登录方式其实有不少的麻烦，比如**容易被爆破**、**输密码时容易输错还麻烦**......
那么有没有更好的方法呢？当然！那就是使用密钥对登录。它相当于为你和服务器配了一把专属的数字钥匙串，登录时，系统会自动用这对钥匙验证身份，无需再输密码！这样不仅**省时省力**，安全性也**大大提升**！（你总不可能爆破我密钥吧？😂👉）

通常情况下，我们并不推荐直接在服务器上生成密钥对的操作。私钥（`id_ed25519` 或 `id_rsa`）是登录的凭证，若在服务器上生成私钥，需手动下载到本地，**过程中可能泄露私钥**（如通过不安全的传输方式）

演示环境：

- 服务器 `Debian GNU/Linux 12 (bookworm) x86_64`

（本篇文章将默认读者已经成功安装SSH并已经可以登录服务器。**以下在服务器上操作的步骤均使用root用户执行命令**）

### 正文步骤

1. 在**本地打开终端**并生成密钥对

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"  # 这里比较推荐ed25519算法
   # 或使用RSA：ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```

   - 按提示设置密钥存储路径（默认 `~/.ssh/id_ed25519`）和密码（设置密钥密码（Passphrase）可以增加安全性，但每次使用密钥需输入该密码，如果你追求免密登录可留空）
   - 如果你是 `Windows`用户的话，密钥将默认保存在 `C:\Users\User/.ssh/id_ed25519`（这里的User需要替换为你的计算机用户名）

     如果不出意外，那么你将会在 `~/.ssh/`目录下看到生成的密钥

     - `id_ed25519` 私钥，应当保密
     - `id_ed25519.pub` 公钥，用来验证私钥
2. 将公钥上传到服务器

   ```bash
   # 这里需要将server替换为服务器的IP地址
   scp ~/.ssh/id_ed25519.pub root@server:~/.ssh/
   ```

   如果你是 `Windows`用户的话

   ```cmd
   # 这里的User需要替换为你的计算机用户名
   scp "C:\Users\User\.ssh\id_ed25519.pub" root@server:~/.ssh/
   ```
3. 将公钥添加到 `authorized_keys`

   接下来**新建一个SSH会话**如果刚才不出意外的话，你的公钥就已经上传到 `~/.ssh`目录下了

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

   为了确保可以正确使用密钥连接到服务器，这里可能需要对 `sshd_config`进行一些修改。修改之前，记得先对此配置文件进行一次备份

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

   完成修改后，可以输入 `sshd -T`来测试配置文件是否正确，如果没有错误的话，`sshd`会从配置文件中读取并显示所有的默认值和已修改的值。如果存在错误，`sshd`会给出详细的提示，读者可以根据提示进行修改，如：

   ```bash
   # sshd -T
   /etc/ssh/sshd_config: line 14: Bad configuration option: xxxxxxx
   /etc/ssh/sshd_config: terminating, 1 bad configuration options
   ```

   如果没有问题，那么就可以重启 `sshd`了

   ```bash
   systemctl restart sshd
   ```

   重启后，**先别着急关闭当前的SSH会话**！这里强烈建议先试用密钥连接试一下是否可以正常连接到服务器，万一后面密钥和密码都不能登录上服务器的话，那就悲剧了
5. 测试连接

   接下来请打开 **新终端窗口** 测试密钥登录，确认成功后再关闭当前 SSH 会话

   ```bash
   ssh -i ~/.ssh/id_ed25519 root@server
   ```

   如果你是 `Windows`用户的话

   ```bash
   ssh -i "C:\Users\huajibenji\.ssh\id_ed25519" root@server
   ```

   如果登录成功，那么就可以进行下一步了
6. 编辑 `SSH config`

   接下来，可以通过配置 `SSH config`的方式来进一步简化登录步骤，一般情况下，`SSH config`一般都存放在 `~/.ssh`下（`Windows`在 `C:\Users\huajibenji\.ssh`目录下）

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

   如果你是 `Windows`用户的话

   ```
   Host myserver
     HostName example.com
     User user
     IdentityFile C:\Users\User\.ssh\id_ed25519
   ```

   - `Host` 别名或主机名
   - `HostName` 实际主机名/IP地址
   - `User` 用户
   - `IdentityFile` 密钥文件路径

   完成配置后，你就可以使用以下命令来连接服务器

   ```bash
   ssh myserver
   ```

### SSH配置文件解释

为了方便理解，这里使用AI将整个SSH配置文件（`/etc/ssh/sshd_config`）注释了一遍，注释内容仅供参考。

```
# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# 该SSH服务端通过以下PATH环境变量编译
# This sshd was compiled with PATH=/usr/local/bin:/usr/bin:/bin:/usr/games

# OpenSSH默认策略：注释项表示默认值，取消注释将覆盖默认值
# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

# 包含其他配置文件目录
Include /etc/ssh/sshd_config.d/*.conf

# 默认监听端口（注释状态使用默认22端口）
#Port 22

# 地址族：any(IPv4/IPv6), inet(IPv4), inet6(IPv6)
#AddressFamily any

# 监听所有IPv4地址
#ListenAddress 0.0.0.0

# 监听所有IPv6地址
#ListenAddress ::

# 主机密钥文件路径
#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# 加密与密钥重协商设置
#RekeyLimit default none

# 日志设置
# 日志设施类型：AUTH/AUTHPRIV/DAEMON等
#SyslogFacility AUTH

# 日志级别：QUIET/FATAL/ERROR/INFO/VERBOSE等
#LogLevel INFO

### 认证设置 ###

# 登录宽限时间（超时断开）
#LoginGraceTime 2m

# 允许root登录：prohibit-password(仅密钥)/no(禁止)/yes(允许)
#PermitRootLogin prohibit-password

# 严格检查文件权限和所有权
#StrictModes yes

# 单次连接最大认证尝试次数
#MaxAuthTries 6

# 单个网络连接最大会话数
#MaxSessions 10

# 启用公钥认证
#PubkeyAuthentication yes

# 授权密钥文件路径
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2

# 授权主体文件路径
#AuthorizedPrincipalsFile none

# 外部授权密钥命令
#AuthorizedKeysCommand none

# 执行密钥命令的用户
#AuthorizedKeysCommandUser nobody

# 基于主机的认证
#HostbasedAuthentication no

# 是否忽略用户known_hosts文件
#IgnoreUserKnownHosts no

# 忽略.rhosts和.shosts文件
#IgnoreRhosts yes

# 是否允许空密码登录
#PermitEmptyPasswords no

# 禁用交互式键盘认证（安全增强）
KbdInteractiveAuthentication no

### Kerberos选项 ###
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

### GSSAPI选项 ###
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# 启用PAM认证模块（必需功能如密码认证）
UsePAM yes

# 允许SSH代理转发
#AllowAgentForwarding yes

# 允许TCP端口转发
#AllowTcpForwarding yes

# 网关端口：是否允许远程主机连接转发端口
#GatewayPorts no

# 启用X11图形界面转发
X11Forwarding yes

# X11显示偏移量
#X11DisplayOffset 10

# 绑定X11转发到本地回环地址
#X11UseLocalhost yes

# 是否分配TTY终端
#PermitTTY yes

# 禁用登录时显示系统信息（安全增强）
PrintMotd no

# 是否显示上次登录信息
#PrintLastLog yes

# 保持TCP连接活跃
#TCPKeepAlive yes

# 禁止加载用户环境变量
#PermitUserEnvironment no

# 压缩方式：delayed(延迟压缩)/yes/no
#Compression delayed

# 客户端活跃检测间隔（秒）
#ClientAliveInterval 0

# 客户端活跃检测最大次数
#ClientAliveCountMax 3

# 禁用DNS反向解析（加速连接）
UseDNS no

# 进程ID文件位置
#PidFile /run/sshd.pid

# 最大并发未认证连接数（10初始:30概率:100最大）
#MaxStartups 10:30:100

# 是否允许隧道设备
#PermitTunnel no

# 用户会话根目录限制
#ChrootDirectory none

# SSH版本附加信息
#VersionAddendum none

# 登录横幅文件路径
#Banner none

# 接受客户端传递的本地化环境变量
AcceptEnv LANG LC_*

# SFTP子系统配置
Subsystem       sftp    /usr/lib/openssh/sftp-server

### 用户匹配规则示例 ###
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server

### 以下为显式启用的关键安全设置 ###

# 禁用DNS解析（防止连接延迟）
UseDNS no

# 将日志记录到安全专用设施
SyslogFacility AUTHPRIV

# 允许root用户直接登录（需密钥认证）
PermitRootLogin yes

# 禁用密码认证（强制使用密钥登录）
PasswordAuthentication no
```
