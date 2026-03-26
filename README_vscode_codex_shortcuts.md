# VS Code Codex 快捷键记录

只记录当前实际使用的快捷键配置。

## 第一原则

- `Ctrl+L` 有选区时发给 Codex
- `Ctrl+L` 无选区时保留 VS Code 的选中当前行行为

## 已做配置

- 命令：`chatgpt.addToThread`
- 命令标题：`Add to Codex Thread`
- 文件：`C:\Users\chenk\AppData\Roaming\Code\User\keybindings.json`

配置：

```json
[
  {
    "key": "ctrl+l",
    "command": "chatgpt.addToThread",
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "ctrl+l",
    "command": "expandLineSelection",
    "when": "editorTextFocus && !editorHasSelection"
  }
]
```

## 最小复原步骤

1. 在 VS Code 打开 `keybindings.json`
2. 写入上面的配置
3. 保存后测试：
   - 选中文本按 `Ctrl+L`
   - 不选中文本按 `Ctrl+L`

## 图形界面设置方式

1. 在 VS Code 按 `Ctrl+Shift+P`
2. 搜索并打开 `Preferences: Open Keyboard Shortcuts`
3. 在快捷键界面搜索 `codex`
4. 找到 `Add to Codex Thread`
5. 直接用键盘录入想要的快捷键

说明：

- 这个方式适合先确认命令是否存在
- 如果要保留“有选区发给 Codex、无选区选当前行”这套分流行为，还是更适合直接改 `keybindings.json`
