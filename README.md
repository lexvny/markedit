# MarkEdit

MarkEdit 是一个实验性的、零构建步骤的单文件 Markdown 编辑器。打开 `markdown-editor-v1.html` 即可使用：页面默认显示渲染结果，点击任意内容块后切换为该块的 Markdown 源码编辑态。

> 当前仓库对应 v1 原型。README 以实际代码行为为准，不代表一个已经具备文件管理能力的完整桌面编辑器。

## 当前状态

- 入口文件：`markdown-editor-v1.html`
- 技术形态：HTML、CSS、JavaScript 和 Markdown 解析器全部内联
- Markdown 引擎：内联的 `marked.js 15.0.12`
- 构建与安装：不需要
- 外部网络请求：不需要
- 自动化测试：暂无
- 数据持久化：暂无；刷新或关闭页面会丢失本次编辑内容

## 已实现功能

- 渲染态与块级源码编辑态一体化
- 点击块进入编辑，失焦或按 `Esc` 提交当前编辑并重新渲染
- 普通 `Enter` 在当前 textarea 中插入换行
- `Ctrl+Enter` 或 `Cmd+Enter` 提交当前块，并在其后创建一个新段落
- 空块中按 `Enter` 也会创建新段落
- textarea 根据内容自动增高
- 支持 1–6 级 ATX 标题、段落、引用、无序/有序列表、围栏代码块和分隔线的基础块解析
- 行内 Markdown 与段落渲染交由 `marked.js` 处理，包括常见的强调、链接、行内代码等语法
- 代码块内容通过文本转义后渲染

## 快速开始

无需安装依赖或启动服务器，直接用现代浏览器打开：

```text
markdown-editor-v1.html
```

也可以在仓库目录启动任意静态文件服务器。例如已安装 Python 时：

```powershell
python -m http.server 8000
```

然后访问 `http://localhost:8000/markdown-editor-v1.html`。

## 使用方式

| 操作 | 当前行为 |
| --- | --- |
| 点击渲染后的内容块 | 进入该块的 Markdown 源码编辑态 |
| 点击其他块 | 提交当前块，然后编辑目标块 |
| 编辑框失焦 | 提交当前块并重新渲染 |
| `Esc` | 提交当前块并退出编辑态 |
| `Enter` | 在当前块内插入换行 |
| `Ctrl+Enter` / `Cmd+Enter` | 提交当前块，并在后方创建新段落 |
| 空块中按 `Enter` | 创建新段落 |

页面首次打开时加载内置示例内容。当前没有打开本地 Markdown 文件、保存文件或恢复上次会话的能力。

## 实现概览

MarkEdit 将文档维护为一个全局块数组：

```js
blocks: Array<{
  type: string,
  content: string,
  lines: string[],
  level: number,
  lang: string
}>
```

同一时刻最多有一个块处于编辑态，由 `activeIdx` 标识。

主要代码职责如下：

- `init()`：加载内置内容，初始化块数组并渲染编辑器
- `parseBlocks()`：把 Markdown 字符串拆分为内部块结构
- `renderInline()`、`renderBlock()`：将块内容转换为 HTML
- `createBlockEl()`、`refreshDisplay()`：创建块 DOM 并执行全量重渲染
- `enterEdit()`、`exitEdit()`：管理渲染态与 textarea 编辑态的切换
- `handleEditKeydown()`、`finishBlockAndCreateNext()`：处理编辑快捷键和新段落创建
- `rebuildDoc()`：尝试从块数组重建 Markdown；目前尚未接入保存或导出流程
- `insertBlock()`、`mergeBlocks()`：基础块操作；`mergeBlocks()` 目前没有接入交互流程

### 数据流

```text
初始 Markdown
    ↓ parseBlocks()
blocks（内存中的唯一文档状态）
    ↓ refreshDisplay()
渲染后的块 DOM
    ↓ 点击
textarea 编辑态
    ↓ 失焦 / Esc / Ctrl(Cmd)+Enter
重新解析当前块并全量渲染
```

## 项目结构

```text
markedit/
├─ markdown-editor-v1.html   # 应用、样式和内联 marked.js
├─ README.md                 # 当前项目说明与维护约定
└─ .gitignore
```

## 开发与维护

### 修改原则

1. **以实际交互为准。** 修改键盘行为时，同时更新：
   - `handleEditKeydown()` 附近的实现与注释
   - `DEFAULT_MD` 中面向用户的操作提示
   - 本 README 的“使用方式”表格
2. **保持解析与重建尽量对称。** 新增块类型时，至少检查 `parseBlocks()`、`renderBlock()` 和 `rebuildDoc()`。
3. **不要直接修改内联依赖。** HTML 中标有 `DO NOT EDIT` 的区域是压缩后的 `marked.js`；升级时应整体替换，并同步更新版本号和许可证信息。
4. **避免无意引入网络依赖。** 单文件、离线可运行是当前原型最明确的约束。
5. **谨慎处理全量重渲染。** `refreshDisplay()` 会重建全部块 DOM；新增选区、滚动位置、撤销栈等功能时，需要先处理状态恢复。
6. **提交前检查文档漂移。** README 只描述已经存在的能力；计划中的功能应放在 issue 或独立提案中，而不是写成现有功能。

### 最小人工回归检查

当前没有测试框架。每次修改后，至少在浏览器中完成以下检查：

1. 直接打开 HTML，页面无控制台错误，示例文档正常渲染。
2. 点击标题、段落、列表、引用、代码块和分隔线，均能进入编辑态。
3. 修改标题级别后退出编辑，级别和内容正确更新。
4. 普通 `Enter` 能在当前块内换行。
5. `Ctrl+Enter`（macOS 为 `Cmd+Enter`）能创建并聚焦新段落。
6. `Esc`、失焦和点击其他块不会重复提交或丢失刚输入的内容。
7. 编辑多行内容后，块拆分和重新渲染结果符合预期。
8. 刷新页面后确认只恢复内置示例，以免误以为已实现持久化。

如果继续扩大功能范围，建议先为 `parseBlocks()`、`exitEdit()` 和快捷键行为补充可自动运行的测试，再进行结构性重构。

## 已知限制

- 不支持打开、保存、导入或导出 Markdown 文件
- 不支持浏览器本地持久化，刷新页面会重置内容
- 不支持撤销/重做、工具栏、搜索、字数统计或多文档管理
- 自定义块解析器只覆盖基础语法，不是完整 CommonMark/GFM 块解析器
- 空行不会作为独立块保留，复杂列表、嵌套结构和混合块可能被简化
- 围栏代码块解析会记录语言标识，但当前渲染和 `rebuildDoc()` 不保留完整语言信息
- `renderTable()`、`renderDoc()`、`mergeBlocks()` 等代码目前未进入主要交互路径，后续重构前应先确认是否保留
- Markdown 渲染结果会写入 `innerHTML`，目前没有接入 HTML 清理器；不要用它打开或粘贴不可信内容
- 页面内置示例中的快捷键说明可能随代码改动而过时，维护时应按上文“修改原则”同步更新

## 安全说明

`marked.js` 是解析器，不是 HTML 安全清理器。当前实现允许 Markdown 生成的 HTML 进入页面 DOM，因此 MarkEdit 只适合编辑可信内容。

如果未来需要处理外部文件、剪贴板内容或多人共享内容，应在渲染边界加入可靠的 HTML sanitizer，并为链接协议、图片来源和原始 HTML 制定明确策略。

## 后续演进建议

建议按以下顺序推进，以降低单文件原型快速膨胀后的维护成本：

1. 补齐纯函数级解析与重建测试
2. 修正内置帮助文本与实际快捷键的一致性
3. 实现明确的 Markdown 导入、导出与未保存提示
4. 引入可测试的状态层，并减少不必要的全量 DOM 重建
5. 在接受不可信输入前加入渲染清理与安全测试
6. 功能明显增多后，再评估是否继续保持单文件架构

## 许可证

仓库目前没有声明项目级许可证。内联的 `marked.js` 按其 MIT License 使用，版权与许可证注释保留在 `markdown-editor-v1.html` 中。
