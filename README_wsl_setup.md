# WSL 重装记录

只记录这台机器已经采用或推荐采用的做法，重装时照做即可。
前半部分补一份官方建议的精简版，用来解释为什么这样安排。

## 第一原则

- 主工作区放在 WSL 内：`/home/chenk/workspace`
- Windows 盘只临时访问：`/mnt/c/...`
- 同一个项目尽量只用 WSL 这一套工具链
- 如果希望 WSL 和 Windows 主机尽量走同一套网络出口，不要默认相信 WSL2 NAT 会自动继承 Windows 的代理

## 官方建议（精简）

- 文件尽量放在和工具同一侧：用 Linux 命令行和 Linux 工具开发，就把项目放在 WSL 文件系统；用 PowerShell / CMD 和 Windows 工具开发，就把项目放在 Windows 文件系统。
- 官方明确不建议没有必要地跨 Windows / Linux 两边反复操作同一批项目文件；能互相访问，但性能会明显变差。
- 对这台机器来说，日常开发默认按 Linux 项目处理，所以项目根目录放 `~/workspace`，不要长期把项目放在 `/mnt/c/...`。
- WSL 访问 Windows 文件走 `/mnt/c/...`、`/mnt/d/...`；Windows 访问 WSL 文件走 `\\wsl$\Ubuntu\home\chenk\workspace\...`。
- 在 WSL 当前目录里执行 `explorer.exe .`，可以直接让 Windows 资源管理器打开这个 Linux 目录。
- 编辑器官方推荐直接用 Windows 上安装的 VS Code 配合 WSL 扩展；在项目目录执行 `code .`，或者在 VS Code 里用 `WSL: Connect to WSL` / `WSL: Reopen Folder in WSL`。
- 如果 VS Code 左下角显示 `WSL: Ubuntu`，就表示终端、扩展、调试都在 WSL 里运行，这就是推荐的工作方式。
- 命令可以互通：Windows 里用 `wsl <command>` 调 Linux；WSL 里用 `<tool>.exe` 调 Windows。这个能力适合辅助操作，不适合把同一个项目长期混着两边工具链跑。
- 如果启用下面的严格隔离配置，那么 `/mnt/c` 自动挂载和 WSL 调 Windows 命令这两种互通能力都会被主动关闭。

## 网络原则

- WSL2 默认常见的是 `NAT` 网络模式，它不是“自动和 Windows 主机完全同网”的同义词。
- 如果 Windows 上的代理工具只监听 `127.0.0.1:7890` 这类本机端口，WSL 在 `NAT` 模式下通常不能直接复用这个 `localhost` 代理。
- 所以会出现这种情况：
  - Windows 浏览器 / Windows VS Code 能正常访问目标服务
  - WSL 里的命令、WSL Remote 窗口里的扩展请求没有走到同一个代理
  - 最终出口 IP 或地区和 Windows 不一致
- 对 Codex / ChatGPT / OpenAI 登录来说，如果 WSL 那边的请求没有走到和 Windows 一样的代理，可能出现：

```text
Token exchange failed: token endpoint returned status 403 Forbidden:
Country, region, or territory not supported
```

- 这类报错更像“出口地区不对”，不是“WSL 完全没网”。

## 已做配置

### 1. 基础环境

- 已执行：`wsl --install`
- 发行版：`Ubuntu`
- Linux 用户：`chenk`
- 工作目录：`~/workspace`

### 2. 基础工具

```bash
sudo apt update
sudo apt install -y git curl zip unzip build-essential
```

### 3. 防手滑 alias

文件：`~/.bashrc`

```bash
alias rm='rm -I --preserve-root'
alias cp='cp -i'
alias mv='mv -i'
```

生效：

```bash
source ~/.bashrc
```

### 4. 推荐网络同步方案

如果目标是“尽量让 WSL 和 Windows 主机走同一套网络出口”，推荐先用这一套。

这个文件在 **Windows 主机**，不是在 WSL 里。

正确位置是：

```text
C:\Users\chenk\.wslconfig
```

也就是：

- Windows 路径：`%UserProfile%\.wslconfig`
- 不是 WSL 里的 `~/.wslconfig`
- 也不是 WSL 里的 `/etc/wsl.conf`

可以理解成：

- `.wslconfig`：Windows 主机侧的 WSL 全局配置文件
- `/etc/wsl.conf`：某个 Linux 发行版内部的配置文件

如果 `C:\Users\chenk\.wslconfig` 还不存在，就需要你在 Windows 里手动新建。

最简单做法：

1. 在 Windows 里按 `Win + R`
2. 输入 `%UserProfile%`
3. 回车后会打开 `C:\Users\chenk`
4. 在这个目录下新建一个文件，名字必须是 `.wslconfig`
5. 用记事本或 VS Code 打开它
6. 写入：

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
autoProxy=true
firewall=true
```

你也可以直接在 Windows PowerShell 里打开这个文件：

```powershell
notepad $HOME\.wslconfig
```

如果文件不存在，记事本会提示是否创建，选“是”即可。

改完后在 Windows 执行：

```powershell
wsl --shutdown
```

这一步的含义是：

- 关闭当前所有 WSL 发行版
- 让新的 `.wslconfig` 在下次启动 WSL 时生效

`shutdown` 之后怎么重新启动：

下面任意一种都可以：

- 在开始菜单打开 `Ubuntu`
- 在 Windows Terminal 里打开 `Ubuntu` 标签页
- 在 PowerShell 里执行：

```powershell
wsl
```

如果你想直接进默认用户目录，这样最简单：

```powershell
wsl
```

如果你想进某个具体发行版，也可以：

```powershell
wsl -d Ubuntu
```

启动后可以在 WSL 里检查：

```bash
ip addr
cat /etc/resolv.conf
env | grep -i proxy
```

说明：

- `networkingMode=mirrored`：尽量让 WSL 网络行为更接近 Windows 主机
- `dnsTunneling=true`：减少 DNS 解析和 VPN / 代理环境下的奇怪问题
- `autoProxy=true`：让 WSL 尽量继承 Windows 的代理信息
- `firewall=true`：继续让 Windows 防火墙参与管理

补充：

- 如果你的 `wsl` 版本较旧，先执行 `wsl --update`
- 这套方案是“最接近一致”的推荐做法，不代表任何代理软件都会 100% 自动同步

### 5. 代理软件侧的要求

如果你用的是 Clash、v2rayN、Nekoray 之类本地代理工具，除了上面的 `.wslconfig`，通常还要满足至少一条：

- 开启 `TUN mode`
- 开启 `System Proxy`
- 开启 `Allow LAN` / `允许局域网连接`

理解方式：

- 如果代理只是一个 Windows 本机 `localhost` 端口，WSL 经常接不上
- 如果代理工作在系统层、TUN 层，或者允许局域网访问，WSL 才更容易和 Windows 走同一个出口

如果你的目标真的是“尽量完全一致”，优先级通常是：

1. `mirrored networking + autoProxy`
2. 代理软件开启 `TUN mode` 或至少 `Allow LAN`
3. 只有在前两者不够时，再手动给 WSL 配 `http_proxy`

### 6. NAT 模式下的兜底方案

如果暂时还在 `NAT` 模式，或者你的代理没有被 `autoProxy` 自动同步，可以手动把 WSL 指到 Windows 主机代理。

先在 WSL 里找 Windows 主机地址：

```bash
awk '/nameserver/ {print $2; exit}' /etc/resolv.conf
```

很多机器上会返回一个类似 `172.29.x.1` 的地址。

然后假设你的 Windows 代理端口是 `7890`，在 WSL 里临时设置：

```bash
HOST_IP=$(awk '/nameserver/ {print $2; exit}' /etc/resolv.conf)
export http_proxy=http://$HOST_IP:7890
export https_proxy=http://$HOST_IP:7890
export all_proxy=socks5://$HOST_IP:7890
```

如果你要长期使用，再写入 `~/.bashrc`。

前提：

- Windows 侧代理工具必须开启 `Allow LAN`
- 否则即使端口号对了，WSL 也连不上

### 7. 可选严格隔离

文件：`/etc/wsl.conf`

```ini
[automount]
enabled=false

[interop]
enabled=false
appendWindowsPath=false
```

修改后在 Windows 执行：

```powershell
wsl --shutdown
```

说明：

- 开启后默认不挂载 `/mnt/c`
- 开启后不能直接从 WSL 调 Windows 命令

### 8. VS Code 使用方式

- 项目目录：`\\wsl$\Ubuntu\home\chenk\workspace\...`
- 推荐：Windows 上的 VS Code + Remote WSL

### 9. Codex 登录 403 的判断与排查

如果你在 WSL 里 `code .` 打开 VS Code，然后遇到：

```text
Sign-in could not be completed
Token exchange failed: token endpoint returned status 403 Forbidden:
Country, region, or territory not supported
```

优先判断为：

- WSL 请求的出口地区和 Windows 主机不一致
- 不一定是“WSL 完全没网”

推荐排查顺序：

1. 先在 Windows 本机开的 VS Code 里试一次登录
2. 再在 WSL 里执行：

```bash
env | grep -i proxy
curl https://ifconfig.me
```

3. 在 Windows PowerShell 里执行：

```powershell
curl.exe https://ifconfig.me
```

4. 对比两边 IP

如果 Windows 和 WSL 的出口 IP 不一样，就说明它们没有走同一个网络出口。

补充说明：

- 在 WSL Remote 窗口里，扩展和相关命令的网络请求可能来自 WSL 侧
- 所以“Windows 本机网页正常”并不自动等于“WSL Remote 里的扩展也正常”

### 10. Codex 登录端口冲突：`Port 127.0.0.1:1455 is already in use`

如果你把 `.wslconfig` 配好之后，不再报 `403 unsupported region`，但改成报：

```text
Sign-in failed: failed to start login server:
Port 127.0.0.1:1455 is already in use
```

这通常说明：

- 网络路径已经比之前前进了一步
- 现在卡住的是本地登录回调端口冲突
- 这和“地区不支持”不是同一个问题

可以先这样理解：

- Codex 登录时会临时在本地起一个小的回调服务
- 它想绑定 `127.0.0.1:1455`
- 但这个端口当时已经被别的进程占着，或者刚被旧会话释放、还处在 `TIME_WAIT`

这台机器上已经验证有效的处理顺序：

1. 完全退出所有 VS Code 窗口
2. 确认任务管理器里尽量没有残留 `Code.exe`
3. 在 Windows PowerShell 执行：

```powershell
wsl --shutdown
```

4. 等待 `30` 到 `90` 秒
5. 重新打开一个新的 WSL 终端
6. 再进项目目录执行：

```bash
code .
```

7. 只保留这一个 VS Code WSL 窗口，再尝试登录

本机这次的实际结果：

- 只做 `wsl --shutdown` 还不够稳定
- 先把所有 VS Code 窗口彻底关掉，再执行 `wsl --shutdown`
- 然后重新打开 WSL、重新 `code .`
- 端口冲突问题就解除了，Codex 登录恢复正常

为什么这套顺序有效：

- 旧的 VS Code / WSL Remote 会话可能还占着登录回调端口，或者刚释放端口、仍处在 `TIME_WAIT`
- 只重启 WSL，不一定能带走所有残留的 VS Code 登录进程
- 先完全退出 VS Code，再 `wsl --shutdown`，更像一次真正的“会话清场”

为什么要等一会儿：

- 如果端口只是卡在 `TIME_WAIT`，并不一定有真的监听进程
- 这种情况下，等一小会儿再重开，常常就恢复了

### 11. 如果还是冲突，怎么查是谁占了端口

先在 Windows PowerShell 检查：

```powershell
netstat -ano | findstr :1455
```

常见两种情况：

- 看到 `LISTENING`：说明真有进程正在占用这个端口
- 只看到 `TIME_WAIT`：说明更像是旧连接刚释放，先完全退出 VS Code 并稍等再试

如果确实看到 `LISTENING`，记下最后一列 `PID`，再查它是谁：

```powershell
tasklist /FI "PID eq <PID>"
```

如果你怀疑冲突发生在 WSL 里，也可以在 WSL 里检查：

```bash
ss -ltnp | grep ':1455'
```

如果有输出，再根据对应的 `pid` 结束那个进程。

### 12. 实际使用建议

- 同一时间不要开很多个 WSL Remote 的 VS Code 窗口去重复发起登录
- 如果已经登录成功，后面一般不需要频繁重新走登录流程
- 如果又遇到 `1455` 冲突，优先做“完全退出 VS Code + `wsl --shutdown` + 等几十秒再重开”
- 如果重新出现 `403 unsupported region`，那就回到上一节，按网络出口问题继续排查

## 最小复原步骤

1. 执行 `wsl --install`
2. 安装 `Ubuntu`，创建用户 `chenk`
3. 在 Ubuntu 执行 `mkdir -p ~/workspace`
4. 安装基础工具
5. 把 alias 写入 `~/.bashrc`
6. 在 Windows 的 `%UserProfile%\.wslconfig` 写入推荐网络同步配置
7. 如果使用本地代理软件，开启 `TUN mode` 或 `Allow LAN`
8. 执行 `wsl --shutdown`
9. 以后项目统一放 `~/workspace`，编辑器用 VS Code 打开 WSL 项目
10. 如果需要更强隔离，再写入 `/etc/wsl.conf` 并再次执行 `wsl --shutdown`

## 官方参考

- Microsoft Learn: [Set up a WSL development environment](https://learn.microsoft.com/en-us/windows/wsl/setup/environment)
- Microsoft Learn: [Working across Windows and Linux file systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- VS Code: [Developing in WSL](https://code.visualstudio.com/docs/remote/wsl)



