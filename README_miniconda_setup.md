# Miniconda 安装与建环境记录

只记录在 Windows 和 WSL 中都容易复用的最小做法。

## 第一原则

- Windows 项目用 Windows 里的 Miniconda
- WSL 项目用 WSL 里的 Miniconda
- 不要把同一个 Conda 环境跨 Windows 和 WSL 混着用
- 先装 Miniconda，再单独创建项目环境，不要长期把包装在 `base` 里

## 什么时候装哪一边

- 如果你主要在 `PowerShell`、`CMD`、Windows 版 VS Code、Windows Python 工具链里工作，就装 Windows 版
- 如果你主要在 `WSL`、Linux 命令行、`code .` 打开的 WSL 窗口里工作，就装 WSL 版
- 两边都要开发时，可以两边都装，但它们是两套独立环境

## Windows 安装

### 1. 下载

官方安装包：

```text
https://repo.anaconda.com/miniconda/Miniconda3-latest-Windows-x86_64.exe
```

如果你的机器是 ARM，再改用对应的 Windows ARM 安装包。

### 2. 安装建议

- 正常双击安装即可
- 一般选 `Just Me`
- 安装目录可用默认值，也可以用例如 `C:\Users\chenk\miniconda3`
- 不建议勾选“把 Conda 自动加入系统 PATH”的旧式选项
- 安装完成后，优先用 `Anaconda Prompt` 或自己执行 `conda init powershell`

### 3. PowerShell 初始化

如果你想在 PowerShell 里直接使用 `conda`，打开一个已经能运行 `conda` 的终端，执行：

```powershell
conda init powershell
```

然后完全关闭并重新打开 PowerShell。

### 4. 验证

```powershell
conda --version
conda info
```

如果命令还找不到，通常是终端还没重开，或者初始化还没生效。

## WSL 安装

### 1. 下载安装脚本

在 WSL 里执行：

```bash
cd ~
curl -LO https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

如果你的 WSL 发行版是 ARM 架构，再改用对应的 Linux ARM 安装脚本。

### 2. 运行安装

```bash
bash Miniconda3-latest-Linux-x86_64.sh
```

安装过程中：

- 阅读协议后输入 `yes`
- 安装目录可直接用默认值，一般是 `~/miniconda3`
- 当提示是否执行初始化时，选 `yes`

### 3. 让 shell 生效

如果刚装完当前终端还没生效，可以执行：

```bash
source ~/.bashrc
```

如果你用的是 `zsh`，则执行：

```bash
source ~/.zshrc
```

### 4. 验证

```bash
conda --version
conda info
```

如果提示 `conda: command not found`，通常是 shell 初始化还没加载；也可以临时执行：

```bash
source ~/miniconda3/bin/activate
```

## 新建环境

下面这套命令在 Windows PowerShell 和 WSL Bash 中都可以直接用。

### 1. 创建环境

例如新建一个 `py311` 环境：

```bash
conda create -n py311 python=3.11 -y
```

### 2. 激活环境

```bash
conda activate py311
```

激活后，终端前面通常会出现 `(py311)`。

### 3. 安装常用包

例如：

```bash
conda install jupyter numpy pandas matplotlib -y
```

如果某些包更适合用 `pip`，先激活环境，再执行：

```bash
pip install package_name
```

### 4. 查看已有环境

```bash
conda env list
```

### 5. 退出当前环境

```bash
conda deactivate
```

## 常见问题记录

### `CondaToSNonInteractiveError`

在 WSL 里执行下面命令时：

```bash
conda create -n tep_env python=3.10 -y
```

如果出现：

```text
CondaToSNonInteractiveError: Terms of Service have not been accepted for the following channels.
```

通常表示当前 Conda 还没有接受 `defaults` 对应源的服务条款，常见是这两个 channel：

- `https://repo.anaconda.com/pkgs/main`
- `https://repo.anaconda.com/pkgs/r`

这次实际可用的处理命令是：

```bash
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main
conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r
```

接受后重新执行：

```bash
conda create -n tep_env python=3.10 -y
```

本次已经成功创建，环境路径显示为：

```text
/home/chenk/miniconda3/envs/tep_env
```

后续验证命令：

```bash
conda env list
conda activate tep_env
```

判断成功的标志：

- `conda env list` 里能看到 `tep_env`
- 激活后命令行前缀变成 `(tep_env)`

## 常用维护命令

更新 Conda：

```bash
conda update -n base -c defaults conda -y
```

删除环境：

```bash
conda remove -n py311 --all -y
```

导出环境：

```bash
conda env export -n py311 > environment.yml
```

从文件重建环境：

```bash
conda env create -f environment.yml
```

## 推荐做法

- 每个项目单独建一个环境
- 环境名尽量直观，例如 `py311`、`ml`、`latex-tools`
- Windows 里的环境不要拿到 WSL 里复用，反过来也一样
- 如果项目已经有 `environment.yml`，优先按文件创建环境
- 不确定包装在哪个环境时，先执行 `conda info --envs`
