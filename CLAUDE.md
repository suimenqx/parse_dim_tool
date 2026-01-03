# CLAUDE.md

本文件为 Claude Code (claude.ai/code) 在处理本仓库代码时提供指导。

## 项目概述

**离线表格分析器（Smart Parser）** 是一个单页、纯前端的表格解析/过滤/联表工具。零依赖、完全离线运行，整个应用封装在单个 `index.html` 文件中，包含所有 HTML、CSS 和 JavaScript 代码。

**核心价值**：
- 快速排查日志、对比数据快照、生成临时报表
- 完全离线运行，数据不离开本地浏览器，安全可控
- 无需安装、无需后端，双击即可使用

## 快速开始

### 运行应用
```bash
# 无需构建流程，直接在浏览器中打开
open index.html

# 或使用 HTTP 服务器测试
python -m http.server 8000
# 然后访问 http://localhost:8000
```

### 文件结构
- `index.html` - 单文件应用，包含所有 HTML、CSS 和 JavaScript 逻辑（约 2,500 行）
- `README.md` - 中文用户文档
- `CLAUDE.md` - 本文件，开发者指南

## 核心架构

所有代码模块都位于 `index.html` 文件中，主要模块如下：

### 1. **Store（状态管理）**
- **位置**：约 776-810 行
- **功能**：
  - 中央状态管理，持久化到浏览器 `localStorage`
  - 存储键：`v16_4_store`（版本化以支持未来迁移）
  - 管理多个标签页/文档，每个标签页有独立的原始数据、UI 规则和配置
  - 处理主题切换（亮色/暗色）和全局视图配置

### 2. **Parser（解析器）**
- **位置**：约 812-860 行
- **功能**：
  - 将文本输入解析为结构化表格数据
  - 支持多种格式：制表符分隔（TAB）、逗号分隔（CSV）、定宽格式（FIXED）、空格分隔（WS）
  - 基于 `validflag` 表头行自动检测格式
  - 表格通过 `table-data <表名>` 标记分隔

**数据格式要求**：
```text
table-data <表名>
validflag <列名1>    <列名2>    <列名3>
<数据1>    <数据2>    <数据3>
```

### 3. **Joiner（联表执行器）**
- **位置**：约 862-955 行
- **功能**：
  - 执行表或视图之间的 JOIN 操作（inner/left）
  - 支持递归视图引用，带循环依赖检测
  - 解析 select 令牌和别名（如 `left.col AS 别名`）
  - 提供 join 统计（匹配行数、左右独有行数）

### 4. **JoinEditor（联表设计器）**
- **位置**：约 957-1800+ 行
- **功能**：
  - 管理联表视图配置的模态 UI
  - 拖拽界面调整列顺序
  - 字段映射和别名配置
  - 视图存储在全局 `Store.state.globalViews` 中

### 5. **Exporter（导出器）**
- **位置**：约 600-774 行
- **功能**：
  - 生成 Excel (.xlsx) 文件，使用纯 JavaScript 手写的 ZIP 实现
  - 创建标准的 OOXML 结构（Content_Types、workbook、worksheets）
  - 导出 JSON 用于备份/恢复
  - 支持将选中单元格复制为 TSV/HTML 格式

**导出类型**：
- 导出原始：批量导出原始表为 Excel
- 导出预览 Excel：导出当前预览（含过滤、高亮、聚焦列、JOIN 结果）
- 导出页签：备份所有标签页的原始数据和 UI 状态
- 导出配置：仅同步配置（全局视图 + 各标签规则），不覆盖原始数据

### 6. **App（主控制器）**
- **位置**：约 1800+ 行
- **功能**：
  - 主应用程序控制器
  - `proc()` 方法应用过滤、高亮和列聚焦规则
  - 渲染支持内联编辑的预览表
  - 管理单元格选择弹窗用于列过滤

## 数据流程

```
1. 用户粘贴原始文本到"数据源"文本框
2. 点击"解析" → Parser 提取表格
3. 表格存储到当前标签页的 Store.raw 字段
4. App.proc() 应用 UI 规则：
   - 全局过滤（所有表）
   - 表级过滤（当前目标表）
   - 高亮规则
   - 列过滤（每列搜索）
   - 聚焦列（仅显示选定列）
5. 通过 App.renderPreview() 渲染结果
6. 导出按钮调用 Exporter.toExcel()
```

## 过滤规则语法

规则支持：
- **精确匹配**：`key=value`、`key!=value`
- **包含匹配**：`Msg:Error`（冒号是包含操作符）
- **数值比较**：`CPU>90`、`Score<=60`
- **正则表达式**：`/timeout|error/`
- **逻辑运算**：空格 = AND，竖线 `|` = OR

**作用范围**：
- **全局过滤**：作用于所有表（包括 JOIN 视图）
- **行过滤**：作用于当前目标表，隐藏不符合条件的行
- **高亮规则**：作用于当前目标表，标记行为黄色
- **仅显示高亮行**：勾选后只显示高亮行

## localStorage 数据结构

```javascript
{
  docs: [
    {
      id: string,           // 标签页唯一 ID
      title: string,        // 标签页标题
      raw: string,          // 原始输入文本
      ui: {
        displayTables: string[] | null,  // null = 显示所有表
        enabledViews: string[],          // 已启用的 JOIN 视图列表
        targetTable: string,             // 当前配置的目标表
        rules: {                         // 各表的规则
          [tableName]: {
            filter: string,              // 过滤条件
            hl: string,                  // 高亮规则
            focus: string[]              // 聚焦列
          }
        },
        columnFilters: {                  // 列过滤
          [tableName]: {
            [columnName]: string
          }
        },
        collapsedTables: {               // 折叠状态
          [tableName]: boolean
        },
        // ... 其他 UI 状态
      }
    }
  ],
  activeId: string,           // 当前激活的标签页 ID
  theme: 'light' | 'dark',    // 主题
  globalViews: [              // 全局 JOIN 视图配置
    {
      view: string,           // 视图名称
      left: string,           // 左表/视图名
      right: string,          // 右表/视图名
      type: 'inner' | 'left', // JOIN 类型
      on: string,             // ON 条件，如 "id=userId,name=userName"
      select: string          // 逗号分隔的输出列
    }
  ]
}
```

## 关键实现细节

- **零外部依赖**：所有代码包括 ZIP/Excel 生成都使用手写的原生 JavaScript
- **中文界面**：所有标签和消息都是中文
- **版本化 localStorage 键**：`v16_4_store` 允许未来迁移
- **表名前缀**：JOIN 视图使用 `JOIN:` 前缀区分原始表
- **智能空格解析**：解析器先尝试空格分割，列数不匹配时回退到检测模式
- **Excel 导出格式**：使用 .xlsx 格式（OOXML 标准），符合业界规范
- **时间戳精度**：导出的 Excel 文件名包含精确到秒的时间戳，避免重名冲突

## 修改代码指南

修改 `index.html` 时请注意：

1. **保持单文件结构**：所有代码必须在单个 HTML 文件中
2. **localStorage 向后兼容**：维护向后兼容性或更新存储键版本
3. **保留中文 UI**：维持所有中文界面文本
4. **测试双主题**：同时测试亮色和暗色主题
5. **验证离线功能**：确保无外部 CDN 依赖，完全离线可用
6. **Excel 导出规范**：
   - 使用 .xlsx 格式，避免使用旧的 .xls 格式
   - sheet 页名称避免使用冒号等特殊字符（Excel 不支持）
   - 使用标准的 OOXML 结构

## 性能建议

- **建议单表不超过 10,000 行**
- **总数据量不超过 50MB**（包括所有表）
- 大数据集时先使用行过滤减少数据量
- 避免同时启用多个 JOIN 视图

## 浏览器兼容性

| 浏览器 | 最低版本 | 说明 |
|--------|---------|------|
| Chrome | 90+ | 推荐 |
| Edge | 90+ | 完全支持 |
| Firefox | 88+ | 完全支持 |
| Safari | 14+ | 完全支持 |
| Opera | 76+ | 完全支持 |

**注意**：不支持 IE 浏览器

## 常见开发任务

### 添加新的过滤语法
1. 在 Parser 模块中添加解析逻辑
2. 在 App.proc() 中实现过滤逻辑
3. 更新 README.md 中的语法文档

### 修改 Excel 导出格式
1. 修改 Exporter.toExcel() 方法
2. 确保生成标准的 OOXML 结构
3. 测试在 Excel/WPS 中的兼容性

### 更新 localStorage 版本
1. 更新存储键（如 `v17_0_store`）
2. 添加迁移逻辑从旧版本读取数据
3. 更新本文件中的文档

## 调试技巧

### 查看存储数据
在浏览器控制台执行：
```javascript
console.log(JSON.parse(localStorage.getItem('v16_4_store')))
```

### 清空所有数据
```javascript
localStorage.removeItem('v16_4_store')
location.reload()
```

### 查看解析结果
在 Parser 模块添加 `console.log()` 查看解析的表结构

## 版本历史

- **v17.0**：当前版本
  - 完整的 JOIN 视图支持
  - 列过滤和聚焦功能
  - 多标签页管理
  - 亮/暗主题切换
  - Excel/TSV/HTML 多格式导出

## 许可证

MIT License
