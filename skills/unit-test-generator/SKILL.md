---
name: unit-test-generator
description: 当用户选中一段代码，需要编写单元测试、补全单测或覆盖异常情况时触发。
---

# 单元测试专家技能 (Kevin's Unit Test Generator)

## 1. 触发场景与前置环境
- [cite_start]识别当前打开文件的后缀（如 `.py`, `.js`, `.ts`），自动采用项目现有的测试框架（如 Pytest, Jest） 。

## 2. 核心执行规则 (Rules)
- [cite_start]**隔离性**：必须 Mock 掉所有外部 API 请求、数据库连接及本地文件 I/O，确保测试可以在本地独立运行 。
- **覆盖率约束**：
  - [cite_start]必须包含 **正常路径（Happy Path）** 的输入与输出断言 。
  - [cite_start]必须显式编写 **边界条件**（如空值 `None`、空字符串 `""`、空列表 `[]`）的测试用例 。
  - [cite_start]必须编写 **异常捕获** 测试（验证代码在遇到错误输入时是否正确抛出预期的 Exception） 。

## 3. 黄金输出示例 (Few-Shot Example)

这里为 AI 提供一个标杆案例。当用户提出类似需求时，AI 必须模仿以下格式进行输出：

* **用户给你的输入（示例）：** > “帮我给这个 Python 减法函数写下单测”
    > ```python
    > def safe_divide(a, b):
    >     return a / b
    > ```

* **你（AI 助手）符合规范的输出（示例）：**
    > ```python
    > import pytest
    > from your_module import safe_divide
    > 
    > def test_safe_divide_normal_path():
    >     # 正常路径测试
    >     assert safe_divide(10, 2) == 5.0
    > 
    > def test_safe_divide_by_zero_exception():
    >     # 异常边界测试
    >     with pytest.raises(ZeroDivisionError):
    >         safe_divide(10, 0)
    > ```