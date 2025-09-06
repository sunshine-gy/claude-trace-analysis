# Claude Code 日志可视化页面提示词

## 核心需求描述

```
请创建一个 Claude Code API 日志可视化分析页面，具体要求：

1. 单文件 HTML 应用，用于可视化分析 Claude API 调用日志
2. 支持读取 JSONL 格式的日志文件（每行一个 JSON 对象）
3. 解析 Claude API 的请求响应数据，包括 SSE 流式响应
4. 提供直观的时间线视图展示所有 API 调用
5. 支持搜索、筛选和排序功能
6. 使用深色主题，现代化的卡片式设计
7. 支持详细内容的模态框展示
8. 智能处理工具调用和任务清单数据
```

## 数据格式要求

```
输入数据格式为 JSONL，每行包含：
{
  "request": {
    "timestamp": 1756897384.748,
    "method": "POST",
    "url": "https://idealab.alibaba-inc.com/api/code/v1/messages?beta=true",
    "headers": { /* 请求头对象 */ },
    "body": {
      "model": "claude-3-5-haiku-20241022",
      "max_tokens": 512,
      "messages": [{"role": "user", "content": "用户输入"}],
      "system": [{"type": "text", "text": "系统提示"}],
      "tools": [/* 工具定义数组 */],
      "temperature": 0,
      "metadata": { /* 元数据 */ },
      "stream": true
    }
  },
  "response": {
    "timestamp": 1756897386.245,
    "status_code": 200,
    "headers": { /* 响应头对象 */ },
    "body_raw": "event: message_start\ndata: {...}\n\nevent: content_block_delta\ndata: {...}\n\ndata: [DONE]\n\n"
  },
  "logged_at": "2025-09-03T11:03:06.348Z"
}
```

## UI 设计要求

```
颜色主题（深色）：
- 主背景：#0b0d12
- 卡片背景：#121622
- 边框颜色：#2a3348
- 文本颜色：#e6e9f2
- 次要文本：#a7b0c3
- 强调色：#7aa2ff
- 成功色：#2ecc71
- 警告色：#f1c40f
- 错误色：#e74c3c

布局结构：
- 顶部工具栏：文件选择按钮 + 文件名显示 + 统计卡片
- 筛选栏：搜索框 + 方法筛选 + 状态筛选
- 主时间线：标题栏（带排序按钮）+ 日志条目列表
- 模态框：用于显示详细内容

设计风格：
- 现代化卡片设计，带阴影效果
- 圆角边框（12px-16px）
- 渐变背景
- 响应式布局
```

## 核心功能要求

```
1. 文件处理功能：
   - 支持拖拽或点击选择 .jsonl/.json 文件
   - 解析 JSONL 格式（每行一个JSON对象）
   - 容错处理：跳过解析失败的行，处理超长行
   - 计算请求响应时间差

2. 数据统计功能：
   - 总请求数、成功数、错误数统计
   - HTTP 方法类型统计
   - 实时更新统计信息

3. 时间线视图功能：
   - 按时间顺序显示所有 API 调用
   - 每个条目显示：序号、HTTP方法、状态码、响应时间、URL
   - 可折叠的请求响应详情
   - 支持点击查看完整内容

4. 筛选和排序功能：
   - 按 URL 或内容搜索
   - 按 HTTP 方法筛选（GET/POST/PUT/DELETE）
   - 按状态码筛选（2xx/4xx/5xx）
   - 按时间或状态码排序

5. 内容解析功能：
   - SSE 流式响应解析（parseAnthropicSSE）
   - 智能内容格式化（Markdown、JSON、工具调用）
   - 任务清单（TodoWrite）特殊处理
   - 安全的 HTML 转义处理
```

## 特殊功能要求

```
1. SSE 流式响应解析：
   - 解析 "event: xxx\ndata: {...}" 格式
   - 重构 message_start、content_block_delta、message_stop 事件
   - 合并文本内容和工具调用结果

2. 内容智能展示：
   - 可折叠的层级化内容展示（System、Messages、Tools、Metadata）
   - 短内容直接显示，长内容显示预览+查看全部按钮
   - Markdown 内容使用 marked 库渲染，支持延迟渲染
   - JSON 内容格式化显示，支持层级化树形结构
   - 工具调用高亮显示，支持详细参数展示
   - 请求/响应内容的概览式展示（参数徽章形式）

3. 任务清单处理：
   - 从 messages 中提取最新的 TodoWrite 工具调用
   - 按状态（pending/in_progress/completed）显示不同样式
   - 完成的任务显示删除线

4. 模态框功能：
   - 点击详情区域弹出模态框
   - 支持 ESC 键和外部点击关闭
   - 大型 JSON 数据的树形展示
   - 支持复制和导出功能

5. 原始JSON查看功能：
   - 每个日志条目都支持查看原始JSON数据
   - 在新窗口中打开，支持层级化JSON树形展示
   - 自动检测JSON内容并美化显示
   - 支持展开/收起JSON节点进行详细查看（使用HTML5 details元素实现）
   - 与claude-log-viewer.html保持完全一致的交互体验
```

## 技术实现要求

```
技术栈：
- 纯 HTML5 + CSS3 + JavaScript（ES6+）
- 外部依赖：marked.js（Markdown解析）、DOMPurify.js（HTML净化）
- 不使用任何前端框架，纯原生实现

关键函数：
1. handleFileSelect() - 文件选择处理
2. parseLogs() - JSONL 解析和数据处理
3. parseAnthropicSSE() - SSE 流式响应解析
4. createLogEntry() - 日志条目HTML生成
5. formatContent() - 内容格式化（区分请求/响应）
6. applyFilters() - 筛选逻辑实现
7. showModal() / hideModal() - 模态框控制（含背景滚动锁定）
8. renderLogs() - 渲染日志列表（含动态事件绑定）
9. renderLazyMarkdown() - 延迟Markdown渲染
10. createCollapsibleSection() - 创建可折叠内容区域
11. toggleSection() - 切换折叠状态
12. formatSystemContent() - 格式化System内容
13. formatMessagesContent() - 格式化Messages内容
14. formatToolsContent() - 格式化Tools内容
15. openMarkdownFull() - 在新窗口打开完整内容
16. summarizeTopLevelRequest() - 请求内容概览
17. summarizeTopLevelResponse() - 响应内容概览

样式实现：
- CSS 变量定义颜色主题
- Flexbox 布局
- CSS Grid 用于复杂布局
- 响应式设计
- 动画过渡效果
- 可折叠区域样式（collapsible-section）
- 参数徽章样式（param-pill）
- 角色徽章样式（role-badge）
- 消息项样式（message-item）
- 层级化JSON树样式（json-tree, json-node）
- 交互式内容预览样式（open-modal, content-preview:hover）
- 错误消息特殊样式（error-message）
```

## 完整的开发指令

```
请创建一个完整的单文件 HTML 应用，包含：

1. HTML 结构：
   - 标准 HTML5 文档结构
   - 引入 marked 和 DOMPurify CDN
   - 完整的 UI 组件（工具栏、筛选栏、时间线、模态框）

2. CSS 样式：
   - 完整的深色主题样式定义
   - 响应式布局实现
   - 所有组件的视觉样式
   - 动画和过渡效果

3. JavaScript 功能：
   - 完整的文件处理逻辑
   - JSONL 解析和数据转换
   - SSE 流式响应解析算法
   - 内容格式化和渲染
   - 筛选、排序、搜索功能
   - 模态框交互逻辑
   - 错误处理和容错机制

4. 特殊功能：
   - 任务清单智能识别和显示
   - 工具调用高亮处理
   - Markdown 内容渲染
   - 大型数据的性能优化
   - 可折叠的层级化内容展示
   - 原始JSON数据的树形可视化
   - 智能内容预览和完整查看切换

请确保代码具有良好的可读性、健壮的错误处理，并且能够完美处理提供的 log.jsonl 数据格式。页面应该完全自包含，不依赖额外的外部文件。

## 优化版本说明

基于claude-log-viewer.html的优化功能，新版本包含以下增强特性：

### 关键技术修复：
- **JSON树形展示修复**：确保原始JSON数据直接传递给openMarkdownFull函数，而不是包装在markdown代码块中
- **数据格式一致性**：所有"查看全部"和"原始JSON"按钮都使用相同的数据传递格式
- **事件绑定优化**：使用动态事件绑定替代内联onclick，确保所有交互功能正常工作

1. **层级化内容展示**：
   - System、Messages、Tools、Metadata 等内容支持可折叠展示
   - 每个部分可独立展开/收起，提高内容组织性

2. **智能内容预览**：
   - 请求/响应内容在列表中以概览形式显示（参数徽章）
   - 点击查看详情时展示完整的格式化内容
   - 支持Markdown延迟渲染优化性能

3. **原始JSON查看**：
   - 每个日志条目提供"原始JSON"按钮
   - 在新窗口中打开层级化的JSON树形展示
   - 支持展开/收起JSON节点查看详细内容

4. **增强的工具调用展示**：
   - 工具调用参数以结构化形式展示
   - 支持工具结果的差异化显示
   - 工具描述和输入模式的详细展示

5. **交互体验优化**：
   - 点击内容区域直接查看详情
   - 动态事件绑定确保所有元素都能正确响应点击
   - 模态框背景滚动锁定优化用户体验
   - 错误响应的特殊样式处理
```
