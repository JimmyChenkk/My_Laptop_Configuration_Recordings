# TeX 环境重装记录

只记录已经跑通的最小配置。

## 第一原则

- 中文文档统一用 `xelatex`
- 编辑器统一用 VS Code + `LaTeX Workshop`
- 编译产物统一放 `out/`

## 已做配置

### 1. TeX Live

- 版本：`TeX Live 2026`
- 安装目录：`C:\texlive\2026`
- 需要加入 `PATH`：

```text
C:\texlive\2026\bin\windows
```

- 已确认可用命令：
  - `latexmk`
  - `xelatex`
  - `xdvipdfmx`
  - `pdftoppm`
- 当前终端验证结果：

```powershell
where.exe xelatex
C:\texlive\2026\bin\windows\xelatex.exe

where.exe latexmk
C:\texlive\2026\bin\windows\latexmk.exe
```

- 补充说明：
  - 用 PowerShell 把 `C:\texlive\2026\bin\windows` 写入用户 `Path`，与在“高级系统设置 -> 环境变量”里手动添加到“用户变量 Path”，本质上是一样的
  - 两者最终效果相同，都是让当前用户可以直接调用 `xelatex`、`latexmk`
  - 这里修改的是“用户级环境变量”，一般不需要管理员权限
  - 设置完成后如果命令暂时找不到，完全关闭并重新打开 PowerShell、CMD、VS Code 即可

- 这次使用的 PowerShell 写法：

```powershell
$tex = 'C:\texlive\2026\bin\windows'
$userPath = [Environment]::GetEnvironmentVariable('Path', 'User')
if ($userPath -notlike "*$tex*") {
  $newPath = if ([string]::IsNullOrWhiteSpace($userPath)) { $tex } else { $userPath.TrimEnd(';') + ';' + $tex }
  [Environment]::SetEnvironmentVariable('Path', $newPath, 'User')
}
```

### 2. 已确认可用宏包/文档类

- `ctexart.cls`
- `ctexbeamer.cls`
- `beamer.cls`
- `xeCJK.sty`

### 3. VS Code

- 扩展：`james-yu.latex-workshop`
- 用户设置文件：`C:\Users\chenk\AppData\Roaming\Code\User\settings.json`

最小设置：

```json
{
  "latex-workshop.latex.autoBuild.run": "onSave",
  "latex-workshop.view.pdf.viewer": "tab",
  "latex-workshop.latex.outDir": "out",
  "latex-workshop.synctex.afterBuild.enabled": true,
  "latex-workshop.synctex.synctexjs.enabled": true,
  "latex-workshop.latex.tools": [
    {
      "name": "xelatex",
      "command": "xelatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "-output-directory=out",
        "%DOC%"
      ]
    },
    {
      "name": "bibtex",
      "command": "bibtex",
      "args": [
        "out/%DOCFILE%"
      ]
    }
  ],
  "latex-workshop.latex.recipes": [
    {
      "name": "XeLaTeX -> BibTeX -> XeLaTeX x2",
      "tools": [
        "xelatex",
        "bibtex",
        "xelatex",
        "xelatex"
      ]
    }
  ]
}
```

## 最小复原步骤

1. 安装 `TeX Live`
2. 把 `C:\texlive\2026\bin\windows` 加入环境变量
3. 安装 VS Code
4. 安装扩展 `LaTeX Workshop`
5. 把上面的最小设置写入 `settings.json`
6. 编译时优先用：

```powershell
latexmk -xelatex -interaction=nonstopmode -halt-on-error your_file.tex
```
