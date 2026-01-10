# Smart Parser 测试文档

## 概述

本测试套件为 Smart Parser V17.0 提供单元测试和集成测试覆盖。测试框架采用纯 JavaScript 实现，无需任何外部依赖，可直接在浏览器中运行。

## 测试覆盖范围

### 1. Parser - 数据解析模块
- ✅ 解析 TAB 格式的表格
- ✅ 解析多个表格
- ✅ 处理空输入
- ✅ 跳过无效行

**测试数据示例：**
```
table-data Users
validflag	id	name	email
1	张三	zhangsan@example.com
2	李四	lisi@example.com
```

### 2. Store - 状态管理模块
- ✅ 保存和加载状态
- ✅ 清除状态
- ✅ 创建默认状态
- ✅ localStorage 持久化

**数据结构：**
```javascript
{
  docs: [{ id, title, raw, ui }],
  activeId: string,
  theme: 'light' | 'dark',
  globalViews: []
}
```

### 3. 过滤规则引擎
- ✅ 精确相等规则 (`status=active`)
- ✅ 数值比较规则 (`age>18`)
- ✅ 包含规则 (`msg:error`)
- ✅ OR 逻辑 (`status=active|status=pending`)
- ✅ 正则表达式 (`/timeout|error/`)

### 4. JOIN 操作模块
- ✅ INNER JOIN（仅匹配行）
- ✅ LEFT JOIN（保留左表）
- ✅ 处理空表
- ✅ 多列关联

### 5. Excel 导出工具
- ✅ XML 特殊字符转义
- ✅ 处理 null/undefined
- ✅ 数字类型转换

### 6. 主题切换
- ✅ 亮色主题切换
- ✅ 暗色主题切换
- ✅ 主题属性移除

## 如何运行测试

### 方法 1：浏览器直接打开（推荐）

1. 在浏览器中打开 `tests/test.html`
2. 测试将自动运行
3. 查看测试结果汇总

```bash
# macOS
open tests/test.html

# Linux
xdg-open tests/test.html

# Windows
start tests/test.html
```

### 方法 2：本地 HTTP 服务器

```bash
# Python 3
python -m http.server 8000

# Python 2
python -m SimpleHTTPServer 8000

# Node.js (需要安装 http-server)
npx http-server -p 8000

# 然后访问
# http://localhost:8000/tests/test.html
```

### 方法 3：在 Termux 中运行

```bash
# 进入项目目录
cd /storage/emulated/0/xin/code/github/parse_dim_tool

# 使用 Python 启动服务器
python -m http.server 8000

# 在浏览器中访问
# http://localhost:8000/tests/test.html
```

## 测试结果解读

测试结果页面包含以下信息：

### 汇总统计
- **总测试数**：执行的测试用例总数
- **通过**：成功通过的测试数量
- **失败**：失败的测试数量
- **通过率**：成功百分比

### 测试套件详情
每个测试套件显示：
- 套件名称
- 通过/失败数量
- 每个测试用例的状态

**状态指示：**
- ✓ 绿色：测试通过
- ✗ 红色：测试失败（显示错误详情）

## 测试框架 API

### 基础用法

```javascript
// 定义测试套件
TestFramework.describe('模块名称', () => {

    // 定义测试用例
    TestFramework.it('应该执行某操作', () => {
        // 测试代码
        assert.equal(actual, expected);
    });
});

// 运行所有测试
await runAllTests();
```

### 断言方法

| 方法 | 说明 | 示例 |
|------|------|------|
| `assert.equal(a, b)` | 相等断言 | `assert.equal(1 + 1, 2)` |
| `assert.deepEqual(a, b)` | 深度相等 | `assert.deepEqual({a:1}, {a:1})` |
| `assert.ok(value)` | Truthy 断言 | `assert.ok(true)` |
| `assert.throws(fn)` | 异常断言 | `assert.throws(() => error())` |
| `assert.fail(msg)` | 失败断言 | `assert.fail('自定义错误')` |

## 添加新测试

### 步骤

1. 在 `tests/test.html` 中找到 `TestFramework.describe` 区域
2. 添加新的测试套件或测试用例：

```javascript
TestFramework.describe('新模块名称', () => {
    TestFramework.it('应该执行某功能', () => {
        const result = yourFunction();
        assert.equal(result, expectedValue);
    });
});
```

3. 刷新浏览器查看结果

### 测试最佳实践

1. **单一职责**：每个测试只验证一个功能点
2. **清晰命名**：测试名称应该描述被测试的行为
3. **独立性**：测试之间不应有依赖关系
4. **可读性**：使用 arrange-act-assert 模式

```javascript
TestFramework.it('应该过滤不匹配的行', () => {
    // Arrange（准备数据）
    const data = [['id', 'status'], ['1', 'active']];
    const filter = parseFilter('status=active');

    // Act（执行操作）
    const result = filter(data[1], data[0]);

    // Assert（验证结果）
    assert.ok(result);
});
```

## 当前测试覆盖

| 模块 | 测试用例数 | 覆盖功能 |
|------|-----------|----------|
| Parser | 4 | 数据解析、多表处理 |
| Store | 3 | 状态持久化、默认值 |
| 过滤引擎 | 4 | 各种过滤语法 |
| JOIN | 3 | 内联、左联、空表 |
| Excel | 3 | 字符转义、类型转换 |
| 主题 | 3 | 亮色/暗色切换 |
| **总计** | **20** | - |

## 已知限制

1. **测试隔离**：测试共享 localStorage，可能相互影响
2. **异步测试**：当前不支持异步测试用例
3. **覆盖率**：没有代码覆盖率统计

## 未来改进

- [ ] 添加 CI/CD 集成
- [ ] 支持异步测试
- [ ] 添加代码覆盖率报告
- [ ] 集成 Jest 或 Vitest
- [ ] 添加端到端测试（Playwright）
- [ ] 性能基准测试

## 常见问题

### Q: 测试失败后如何调试？
A: 查看测试用例下方的红色错误信息框，其中包含具体的失败原因。

### Q: 如何单独运行某个测试套件？
A: 当前需要修改代码，可以注释掉不需要的套件，或创建单独的测试文件。

### Q: 测试是否会在 CI 环境中运行？
A: 目前需要手动配置，可以使用 Puppeteer 或 Playwright 在无头浏览器中运行。

## 贡献指南

欢迎贡献更多测试用例！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b test/feature-name`)
3. 添加测试并确保通过
4. 提交更改 (`git commit -m 'Add tests for feature'`)
5. 推送到分支 (`git push origin test/feature-name`)
6. 创建 Pull Request

## 许可证

MIT License - 与主项目保持一致
