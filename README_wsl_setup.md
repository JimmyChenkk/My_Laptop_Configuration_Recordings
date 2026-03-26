# WSL 重装记录

只记录这台机器已经采用的做法，重装时照做即可。
前半部分补一份官方建议的精简版，用来解释为什么这样安排。

## 第一原则

- 主工作区放在 WSL 内：`/home/chenk/workspace`
- Windows 盘只临时访问：`/mnt/c/...`
- 同一个项目尽量只用 WSL 这一套工具链

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

### 4. 可选严格隔离

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

### 5. VS Code 使用方式

- 项目目录：`\\wsl$\Ubuntu\home\chenk\workspace\...`
- 推荐：Windows 上的 VS Code + Remote WSL

## 最小复原步骤

1. 执行 `wsl --install`
2. 安装 `Ubuntu`，创建用户 `chenk`
3. 在 Ubuntu 执行 `mkdir -p ~/workspace`
4. 安装基础工具
5. 把 alias 写入 `~/.bashrc`
6. 如果需要更强隔离，再写入 `/etc/wsl.conf` 并执行 `wsl --shutdown`
7. 以后项目统一放 `~/workspace`，编辑器用 VS Code 打开 WSL 项目

## 官方参考

- Microsoft Learn: [Set up a WSL development environment](https://learn.microsoft.com/en-us/windows/wsl/setup/environment)
- Microsoft Learn: [Working across Windows and Linux file systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- VS Code: [Developing in WSL](https://code.visualstudio.com/docs/remote/wsl)
