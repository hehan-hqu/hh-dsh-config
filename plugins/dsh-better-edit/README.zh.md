# dsh-better-edit 指引

每个 agent preset 在这里都有自己的指引目录：`<preset>/<section>.md`。首次启动时
插件会为随附的 preset（`standard`、`code`、`minimal`、`cordis`）写入编译内置的
默认内容；已有文件绝不被覆盖，因此你的编辑会保留。

每个文件对应一个工具片段：

- `read.md` -> `tool:read`
- `edit.md` -> `tool:edit`
- `undo_last_edit.md` -> `tool:undo_last_edit`

## 自定义

编辑你想修改的 preset 与片段的 `<section>.md` 文件即可——文件正文就是该片段呈
现给模型的文本。文件在 agent 的 session-start 时读取一次，因此修改只影响新会话。

可以用可选的 YAML front-matter 栅栏把片段放到组装后系统提示中的自定义位置：

    ---
    order: 150
    ---

    <片段文本>

没有 front-matter 的文件保持默认顺序。格式错误的栅栏（缺少收尾 `---`、非整数
`order`、未知键）会让整个文件退化为纯文本。

## 回退

这里没有对应目录的 preset、或缺失某个片段文件时，会回退到插件包内的编译内置默认
值。要自定义没有种子目录的 preset，把一个种子目录复制成它的名字即可。
