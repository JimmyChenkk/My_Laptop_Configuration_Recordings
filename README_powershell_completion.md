# PowerShell 补全记录

记录这台 Windows 机器上 PowerShell 输入补全相关的实际行为，避免以后再次误判成“补全坏了”。

## 结论

PowerShell 里看到的灰色提示，不等于 `Tab` 会像 Linux Bash 那样直接把整段内容补全进去。

这里其实是两套机制：

- `Tab`：传统补全，主要用于命令、参数、路径候选的循环补全
- 灰色提示：`PSReadLine` 的预测建议，通常来自历史命令

所以现在这台机器上的行为是正常的，不是配置错误。

## 这台机器上的实际情况

- PowerShell 版本：`7.6.0`
- `PSReadLine` 版本：`2.4.5`
- 当前没有单独的个人 profile 文件

当前按键绑定的核心逻辑是：

- `Tab`：`TabCompleteNext`
- `Shift+Tab`：`TabCompletePrevious`
- `Ctrl+Space`：`MenuComplete`
- `RightArrow`：在行尾时可以接受灰色预测

## 为什么会感觉和 Linux 不一样

在 Bash 里，很多时候直觉是：

- 输入一部分
- 按 `Tab`
- 直接补全

但在 PowerShell 里：

- 灰色那段更像“预测建议”
- `Tab` 更偏向“在补全候选里切换”
- 所以看得到灰色内容，不代表按 `Tab` 就一定会把它吃进去

## 实际怎么用

### 1. 接受灰色预测

如果后面已经出现灰色建议，直接按：

- `RightArrow`

就可以接受它。

这次已经实际验证过：`RightArrow` 就是最直接的接受方式。

### 2. 用传统补全

如果是命令、路径、参数这类传统补全，优先用：

- `Tab`

例如：

```powershell
cd Doc
git chec
```

这类更接近传统补全场景。

### 3. 看补全菜单

如果想显式看候选项，用：

- `Ctrl+Space`

## 最短记忆版

- 灰色提示：按 `RightArrow`
- 传统补全：按 `Tab`
- 看补全菜单：按 `Ctrl+Space`

## 这次排查的最终结论

不是 PowerShell 补全坏了，而是：

- 我把灰色预测当成了 Linux 风格的 `Tab` 补全
- 实际上 PowerShell 默认把这两件事分开了
- 这台机器上按 `RightArrow` 接受灰色建议即可
