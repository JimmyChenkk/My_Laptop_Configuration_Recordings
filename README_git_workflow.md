# Git 工作流（GitHub + WSL + Windows + VS Code + Codex）

这份文档不只讲“怎么按按钮”，更想先把几个容易混掉的概念拆开：

- 本地仓库是什么
- 远端仓库是什么
- SSH 到底在做什么
- `WSL` 和 `Windows` 这两条线分别该怎么走

## 先说结论

你可以把 Git 先想成三层：

1. 你的项目文件夹
2. 这个文件夹里的 Git 历史，也就是 `.git`
3. GitHub 上的远端仓库

最关键的一句是：

**一个远端仓库，可以对应很多个本地仓库。**

比如同一个 GitHub 仓库，可以同时有这些本地副本：

- 旧电脑里的 `D:\repos\my-project`
- 移动硬盘里的 `E:\backup\my-project`
- 新电脑 Windows 里的 `C:\Users\chenk\Desktop\Workspace\my-project`
- 新电脑 WSL 里的 `~/workspace/my-project`

它们都可以指向同一个 GitHub 远端，比如：

```text
git@github.com:JimmyChenkk/<仓库名>.git
```

所以，换电脑并不会让 GitHub 上的仓库消失。真正要分情况看的是：

- 旧电脑里有没有“还没 push 上去”的提交
- 旧磁盘里的项目文件夹是否还保留 `.git`

## 四个核心概念

### 1. 项目文件夹

这就是你平时看到的那个目录，比如：

- Windows：`C:\Users\chenk\Desktop\Workspace\my-project`
- WSL：`~/workspace/my-project`

里面有源码、文档、配置文件等。

### 2. 本地仓库

当这个目录里存在 `.git` 文件夹时，它就不只是“普通文件夹”，而是一个 Git 本地仓库。

本地仓库保存的东西包括：

- 提交历史
- 分支信息
- 远端地址
- 当前工作区改动

所以“本地仓库”不是“这台电脑唯一的一份”。
**本地仓库可以有很多份，每个 clone 都是一份完整的本地仓库。**

### 3. 远端仓库

远端仓库通常就是 GitHub 上那个仓库页面。

它的作用是：

- 作为同步中心
- 给多台电脑共享同一套提交历史
- 给别人协作、开 PR、做备份

远端仓库不会自动知道你本地改了什么，必须靠：

- `git push` 把本地提交推上去
- `git pull` 或 `git fetch` 把远端变化拉下来

### 4. SSH

SSH 在这里不是“仓库本体”，也不是“仓库配对关系”。

你可以先把 SSH 理解成：

**一套“证明这台机器有权限访问 GitHub 仓库”的身份凭证。**

它通常包含两部分：

- 私钥：留在你的当前环境里，不能随便泄露
- 公钥：上传到 GitHub 账号

当你用下面这种地址和 GitHub 通信时：

```text
git@github.com:JimmyChenkk/<仓库名>.git
```

GitHub 会问：“你是谁？”
你的本地 SSH 私钥会参与认证，GitHub 用你账号里登记过的公钥去匹配。
匹配成功，才允许 `clone`、`push`、`pull`。

所以，SSH 的本质不是“把一个本地仓库绑定到一个远端仓库”。
而是：

- 这台机器
- 这个系统环境
- 这个 Git 进程

有没有权限访问 GitHub。

## 你现在最容易混的点

### 1. 仓库是不是“一个本地，一个远端”？

不是。

更准确地说是：

- 可以有 1 个远端仓库
- 可以有很多个本地仓库副本
- 每个本地仓库都可以连接到同一个远端

### 2. 换了电脑，原来的仓库是不是就没了？

不是。

如果你的代码都已经 `push` 到 GitHub 了，那么新电脑只需要重新 `clone`，就能拿回整套历史。

旧电脑的本地仓库仍然存在，只是它不再是你现在主要使用的那一份。

### 3. 旧电脑或移动硬盘里的本地仓库还有用吗？

有时有用，有时没用，关键看这两件事：

- 里面有没有没 push 的提交
- 里面有没有还没进 Git 的重要文件

如果都已经同步到 GitHub 了，那些旧本地仓库更多只是“历史副本”或备份。

### 4. SSH 是不是每个仓库都要单独配一次？

一般不是。

更常见的是：

- 一台 Windows 环境配一套 SSH key
- 一个 WSL 环境也配一套 SSH key
- 两套公钥都加到同一个 GitHub 账号里

之后这个环境里访问你账号有权限的仓库，都可以复用。

所以：

- `Windows` 可以有自己的 `~/.ssh` 或 `%USERPROFILE%\.ssh`
- `WSL` 也可以有自己的 `~/.ssh`
- 它们是两个环境，最好分别看待

## 先选路线：WSL 还是 Windows

不是所有仓库都必须放在 WSL，也不是所有仓库都应该放在 Windows。

### 推荐用 WSL 的情况

- 项目主要跑在 Linux 环境
- 你会用 Python / Node.js / Rust / Go / Docker / C/C++ 等 Linux 工具链
- 你希望 Git、依赖管理器、运行时、Codex 都在一套 Linux 环境里统一起来

### 推荐用 Windows 的情况

- 项目主要依赖 Windows 程序
- 你主要处理 Office、LaTeX、一些 Windows 原生 GUI 工具
- 这个仓库本身更像“个人配置 / 文档 / 笔记 / 脚本集合”
- 你当前并不需要 Linux 工具链

### 最重要的实践

**同一个仓库，尽量只选一边作为主工作区。**

也就是说：

- 仓库如果主要放在 `WSL`，就用 `WSL Git` 操作它
- 仓库如果主要放在 `Windows`，就用 `Windows Git` 操作它
- 不要一会儿用 Windows Git，一会儿用 WSL Git，轮流操作同一份开发中的仓库

## 路线 A：WSL 工作流

这一条最适合“开发型项目”。

### 第一原则

- 仓库目录放在 WSL 内：`~/workspace/<repo>`
- 不要长期把开发中的仓库放在 `/mnt/c/...`
- 所有 Git 命令都在 WSL 终端里跑
- VS Code 以 WSL 方式打开项目，左下角看到 `WSL: Ubuntu`
- Codex 也跟着这个仓库一起跑在 WSL 窗口里

### 1. 在 WSL 安装工具

```bash
sudo apt update
sudo apt install -y git openssh-client
```

### 2. 配 Git 身份

```bash
git config --global user.name "JimmyChenkk"
git config --global user.email "chenkunlong101@gmail.com"
git config --global init.defaultBranch main
```

### 3. 在 WSL 生成 SSH key

```bash
ssh-keygen -t ed25519 -C "chenkunlong101@gmail.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```

然后：

1. 复制输出的公钥
2. 打开 GitHub
3. 进入 `Settings` -> `SSH and GPG keys` -> `New SSH key`
4. 粘贴公钥并保存

测试：

```bash
ssh -T git@github.com
```

### 4. 在 WSL clone 仓库

```bash
cd ~/workspace
git clone git@github.com:JimmyChenkk/<仓库名>.git
cd <仓库名>
pwd
```

你应该看到类似：

```text
/home/chenk/workspace/<仓库名>
```

### 5. 用 VS Code 以 WSL 方式打开

```bash
code .
```

检查：

- 左下角显示 `WSL: Ubuntu`
- VS Code 新终端里的 `pwd` 仍然是 `/home/...`
- Git 命令在这个 WSL 终端里执行

## 路线 B：Windows 工作流

这一条更适合“Windows 工具链型项目”或“配置 / 文档仓库”。

### 第一原则

- 仓库目录放在 Windows 路径下，例如：`C:\Users\chenk\Desktop\Workspace\<repo>`
- Git 命令统一在 Windows PowerShell 或 Windows 终端里跑
- VS Code 正常以 Windows 窗口打开仓库
- Codex 也跟着这个 Windows 仓库窗口一起工作

### 1. 安装 Windows Git

推荐装 `Git for Windows`。装好后在 PowerShell 里确认：

```powershell
git --version
ssh -V
```

### 2. 配 Git 身份

```powershell
git config --global user.name "JimmyChenkk"
git config --global user.email "chenkunlong101@gmail.com"
git config --global init.defaultBranch main
```

### 3. 在 Windows 生成 SSH key

```powershell
ssh-keygen -t ed25519 -C "chenkunlong101@gmail.com"
Set-Service ssh-agent -StartupType Manual
Start-Service ssh-agent
ssh-add $HOME\.ssh\id_ed25519
Get-Content $HOME\.ssh\id_ed25519.pub
```

然后同样把公钥加到 GitHub：

1. 打开 GitHub
2. 进入 `Settings` -> `SSH and GPG keys` -> `New SSH key`
3. 粘贴公钥并保存

测试：

```powershell
ssh -T git@github.com
```

### 4. Windows 上 `ssh-agent` 可能看起来“没生效”的原因

在 Windows 上，常见会同时存在两套 SSH：

- Windows 自带 OpenSSH：`C:\Windows\System32\OpenSSH\ssh.exe`
- Git for Windows 自带 SSH：例如 `C:\Program Files\Git\usr\bin\ssh.exe`

如果你是用 Windows 的 `ssh-agent` 和 `ssh-add` 加载私钥，但 `git push` 实际调用的是 Git 自带那套 SSH，就可能出现这种现象：

- `ssh-add -l` 能看到密钥
- `git push` 还是继续要求输入 `passphrase`

一个明显信号是提示里出现了这种路径风格：

```text
/c/Users/chenk/.ssh/id_ed25519
```

这通常说明 Git 正在走 Git for Windows / MSYS 风格的 SSH，而不是 Windows OpenSSH。

### 5. 让 Git 明确使用 Windows OpenSSH

如果你希望 `ssh-agent` 真正接管 `git push`，可以明确指定：

```powershell
git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
```

然后重新打开 PowerShell，再检查：

```powershell
ssh-add -l
git push
```

如果 `git push` 不再要求输入 `passphrase`，就说明已经打通。

### 6. 在 Windows clone 仓库

```powershell
cd C:\Users\chenk\Desktop\Workspace
git clone git@github.com:JimmyChenkk/<仓库名>.git
cd .\<仓库名>
pwd
```

你应该看到类似：

```text
Path
----
C:\Users\chenk\Desktop\Workspace\<仓库名>
```

### 7. 用 VS Code 打开

```powershell
code .
```

检查：

- 左下角不要显示 `WSL: Ubuntu`
- VS Code 终端里 `pwd` / `Get-Location` 还是 Windows 路径
- Git 命令在 Windows 终端里执行

## 共同流程：从 GitHub 建仓库到第一次 push

如果你现在本地还没有代码，最稳的流程是：

1. GitHub 网页建仓库
2. 复制 SSH 地址
3. 在你选定的环境里 `clone`
4. 用 VS Code 打开
5. 开始写文件
6. 第一次提交并推送

常用命令：

```bash
git status
git add .
git commit -m "chore: bootstrap project"
git push -u origin main
```

如果你用的是 Windows，把终端换成 PowerShell 即可，命令本身一样。

## 已有本地目录时怎么接到 GitHub

这里要先判断，你手上的“本地目录”到底是哪一种。

### 情况 A：目录里已经有 `.git`

这说明它本来就是一个 Git 仓库。

先看它现在连到哪里：

```bash
git remote -v
git branch -vv
git status
```

你可能看到几种情况：

- 已经连着 GitHub 远端：那就继续用，不需要重新 `git init`
- 没有远端：那就 `git remote add origin ...`
- 远端地址不对：那就改成新的 SSH 地址

例如：

```bash
git remote remove origin
git remote add origin git@github.com:JimmyChenkk/<仓库名>.git
```

如果你收到：

```text
error: remote origin already exists.
```

说明这个仓库已经记录过一个 `origin` 了。这时不要再 `add`，而是改地址：

```bash
git remote set-url origin git@github.com:JimmyChenkk/<仓库名>.git
```

### 情况 B：目录里没有 `.git`

这说明它只是“普通文件夹”，还不是 Git 仓库。

这时才需要先在本地初始化：

```bash
git init
git branch -M main
git add .
git commit -m "chore: initial import"
```

上面这几步都只发生在本地，不要求 GitHub 先有仓库。

但如果你接下来想 `push` 到 GitHub，就还需要一个已经存在的远端仓库。最常见做法是先去 GitHub 网页建一个空仓库，然后再把它接上：

```bash
git remote add origin git@github.com:JimmyChenkk/<仓库名>.git
git push -u origin main
```

也就是说：

- `git init`：只是在本地把普通文件夹变成 Git 仓库
- `git commit`：只是在本地生成提交历史
- `git remote add origin ...`：告诉本地仓库将来要连到哪个远端
- `git push`：把本地提交真正推到远端，所以这一步要求远端仓库已经存在

### 情况 C：旧移动硬盘上有老仓库

如果那个目录保留了 `.git`，它依然是一个完整的本地仓库。

你可以：

- 直接把整个目录拷到新电脑继续用
- 或者更干净一点：确认远端是最新的后，在新电脑重新 `clone`

如果你怀疑旧仓库里还有没 push 的内容，先在旧仓库里检查：

```bash
git status
git log --oneline --decorate --graph -20
git remote -v
```

## 第一次 push 常见情况

### 1. `Repository not found`

这通常表示：

- GitHub 上这个仓库还不存在
- 或者仓库名 / 大小写写错了
- 或者你当前 GitHub 账号没有访问权限

如果你是新建仓库后第一次推送，最常见原因就是 GitHub 网页端还没先建这个仓库。

### 2. `This repository moved. Please use the new location`

这表示仓库曾经改过名字，或 GitHub 记录的规范地址已经变化。

例如你原来用的是：

```text
git@github.com:JimmyChenkk/my_laptop_configuration_recordings.git
```

但 GitHub 提示应改成：

```text
git@github.com:JimmyChenkk/My_Laptop_Configuration_Recordings.git
```

这种情况下，GitHub 往往仍会先帮你重定向，所以这次 `push` 可能还是成功的。

但更干净的做法是把本地远端地址也改正：

```bash
git remote set-url origin git@github.com:JimmyChenkk/My_Laptop_Configuration_Recordings.git
git remote -v
```

### 3. `branch 'main' set up to track 'origin/main'`

这表示第一次推送成功，而且本地 `main` 已经开始跟踪远端 `origin/main`。

后面再执行普通的：

```bash
git push
git pull
```

通常就够了。

## 换电脑时，到底哪些东西要迁移

### 必须重新配置的

- Git 本身
- SSH key
- Git `user.name` / `user.email`
- VS Code / 扩展 / 终端环境

### 不一定要迁移的

- 旧电脑上的本地仓库副本

如果远端 GitHub 已经是最新的，新电脑直接 `clone` 通常更干净。

### 一定要先确认的风险

- 旧电脑有没有没 push 的提交
- 旧电脑有没有只存在本地、还没纳入 Git 的重要文件
- 移动硬盘里的仓库是不是保留了 `.git`

## SSH 的最小直觉版解释

你可以把它想成“门禁卡”：

- GitHub 是门
- 远端仓库是门里的房间
- 你的 SSH 私钥是你手里的门禁卡
- GitHub 账号里登记的公钥，是系统里保存的门禁卡备案

当你 `git push` 时，不是把“仓库”拿去认证，而是这台机器上的 Git 在说：

“我是 chenk 这边的某个已授权环境，请允许我访问 GitHub 上的这个仓库。”

所以：

- 仓库可以复制很多份
- 但每个环境都要有自己的访问凭证，或者能使用已有凭证
- 新电脑换了以后，最常见的事不是“仓库没了”，而是“这台新机器还没配 SSH key”

### passphrase 和 ssh-agent 的关系

- `passphrase` 可以理解成“本地私钥自己的密码”
- 它保护的是私钥文件，不是 GitHub 账号密码
- `ssh-agent` 的作用是缓存“已经解锁过的私钥”，避免你每次 `push` 都重新输入 `passphrase`

如果你没有给私钥设置 `passphrase`，那通常就不太需要 `ssh-agent`。

如果你设置了 `passphrase`，那 `ssh-agent` 的价值会明显变大。

## Codex 配合方式

### 如果仓库在 WSL

- 用 WSL 窗口打开
- 让 Codex 在这个 WSL 窗口里读代码、跑命令、改文件
- 不要切回 Windows 窗口去操作同一份仓库

### 如果仓库在 Windows

- 用 Windows 窗口打开
- 让 Codex 在这个 Windows 窗口里工作
- 不要同时再去 WSL 里操作同一个仓库目录

## 常见误区

- 把同一个开发中的仓库同时放在 `Windows` 和 `WSL` 两边各改一份，却不清楚谁是主副本
- 用 Windows 的 Git 和 WSL 的 Git 轮流操作同一个仓库
- 以为 SSH 是“每个仓库绑定一次”
- 以为换电脑后 GitHub 上的仓库会跟着消失
- 以为只拷走工作文件、不拷 `.git` 也等于完整迁移仓库
- 在 Windows 上用了 `ssh-agent`，却没注意 Git 实际调用的是另一套 SSH
- 让 Codex 改完就直接提交，不先看 `git diff`
- 把 `.env`、密钥、令牌、数据库文件直接提交进仓库

## 最小命令速查

```bash
# 看当前仓库状态
git status

# 看远端地址
git remote -v

# 看当前分支和跟踪关系
git branch -vv

# 看最近提交
git log --oneline --decorate --graph -20

# 更新主分支
git switch main
git pull

# 开新分支
git switch -c feat/<topic>

# 提交
git add .
git commit -m "feat: describe change"

# 推送
git push -u origin feat/<topic>
```

## 一句话版本

先分清：**仓库可以有很多个本地副本，但远端 GitHub 仓库通常只有一个主同步点；SSH 不是仓库本体，而是这台机器访问 GitHub 的身份凭证。**

在此基础上再选路线：

- 开发型项目：优先 `WSL`
- Windows 工具链型项目、文档仓库、配置仓库：可以直接走 `Windows`

选好以后，一个仓库尽量只在一套环境里作为主工作区长期维护。

## 官方参考

- GitHub Docs: [Creating a new repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)
- GitHub Docs: [Cloning a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)
- GitHub Docs: [Connecting to GitHub with SSH](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- GitHub Docs: [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
- GitHub Docs: [Adding a new SSH key to your GitHub account](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)
- GitHub Docs: [About remote repositories](https://docs.github.com/en/get-started/git-basics/about-remote-repositories)
- Microsoft Learn: [Set up a WSL development environment](https://learn.microsoft.com/en-us/windows/wsl/setup/environment)
- Microsoft Learn: [Working across Windows and Linux file systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- VS Code: [Developing in WSL](https://code.visualstudio.com/docs/remote/wsl)
