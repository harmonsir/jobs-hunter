# MiniMax ETL 集成完成总结

## ✅ 已完成的工作

### 1. 创建核心文件

#### `etl_minimax.py` - MiniMax ETL 主脚本
- 解析 MiniMax 招聘页面 HTML
- 提取职位信息（标题、URL、城市、类别、雇佣类型、描述等）
- 标准化雇佣类型（全职/实习）
- 生成符合 schema.json 的 JSON 格式
- 完善的错误处理机制

**核心功能：**
- `parse_job_id()` - 从 URL 提取职位 ID
- `parse_employment_type()` - 标准化雇佣类型
- `parse_job_brief()` - 解析 HTML 并提取职位信息
- `save_to_json()` - 保存为 JSON 文件
- `etl()` - 主流程函数

### 2. 集成到批量处理

#### 更新 `batch_etl.py`
- 导入 `etl_minimax.parse_job_brief`
- 在 `SITE_CONFIGS` 中添加 MiniMax 配置
- 支持批量处理 MiniMax 的多个 HTML 页面

### 3. 测试脚本

#### `test_minimax_etl.py`
- 快速测试 MiniMax ETL 功能
- 默认处理 `ds-minimax/p-1.html`
- 输出到 `output/minimax_jobs.json`

### 4. 文档

#### `MINIMAX_ETL_README.md`
- 详细的使用说明
- HTML 结构解析说明
- 字段提取方式说明
- 输出示例
- 扩展建议

#### `MINIMAX_QUICKSTART.md`
- 快速开始指南
- 安装依赖
- 运行方式
- 验证结果
- 常见问题解答

## 📊 数据提取能力

### 必需字段（100%实现）
| 字段 | 状态 | 提取方式 |
|------|------|----------|
| id | ✅ | 从 URL 正则提取 |
| site_id | ✅ | 固定值 "minimax" |
| title | ✅ | 从职位标题元素提取 |
| url | ✅ | 拼接完整 URL |
| posted_time | ✅ | 当前时间 ISO 格式 |

### 可选字段（部分实现）
| 字段 | 状态 | 提取方式 |
|------|------|----------|
| company_name | ✅ | 固定值 "MiniMax" |
| city | ✅ | 从副标题提取 |
| location_text | ✅ | 同 city |
| job_category | ✅ | 从职位类别元素提取 |
| job_level | ⚠️ | 未实现（可根据标题推断） |
| employment_type | ✅ | 标准化（full-time/intern） |
| work_mode | ✅ | 固定值 "onsite" |
| salary_text | ⚠️ | 未实现（页面可能不显示） |
| tags | ⚠️ | 固定空数组 |
| summary | ✅ | 职位描述前 200 字 |
| discover_time | ✅ | 当前时间 ISO 格式 |

## 🎯 核心特性

1. **符合项目规范**
   - 遵循 Google Python Style Guide
   - 使用双引号
   - 完整的类型注解
   - 清晰的文档字符串

2. **健壮的错误处理**
   - 单个职位解析失败不影响整体
   - 异常捕获并打印错误信息
   - 缺失字段设置默认值

3. **易于扩展**
   - 模块化设计
   - 单一职责函数
   - 清晰的代码结构

4. **完整集成**
   - 集成到 `batch_etl.py`
   - 支持批量处理
   - 统一的输出格式

## 📁 文件清单

```
job-hunter-v2/
├── etl_minimax.py              # ✅ MiniMax ETL 主脚本
├── test_minimax_etl.py         # ✅ 测试脚本
├── batch_etl.py                # ✅ 已更新（集成MiniMax）
├── MINIMAX_ETL_README.md       # ✅ 详细使用说明
├── MINIMAX_QUICKSTART.md       # ✅ 快速开始指南
├── ds-minimax/                 # 数据源目录
│   ├── p-1.html
│   ├── p-2.html
│   └── p-3.html
└── output/
    └── ds-minimax/
        └── jobs.json           # 输出文件（运行后生成）
```

## 🚀 使用方式

### 方式1：测试单个页面
```bash
python test_minimax_etl.py
```

### 方式2：运行ETL脚本
```bash
python etl_minimax.py
```

### 方式3：批量处理（推荐）
```bash
# 处理所有网站
python batch_etl.py

# 只处理MiniMax
python batch_etl.py --sites minimax
```

## 📈 预期输出

处理 `ds-minimax/p-1.html` 预计提取约 10 个职位。

输出示例：
```json
[
  {
    "id": "7594839691242359067",
    "site_id": "minimax",
    "title": "HR 招聘日常实习生（职能方向）",
    "url": "https://hr.minimax.ai/index/position/7594839691242359067/detail",
    "company_name": "MiniMax",
    "city": "北京",
    "location_text": "北京",
    "job_category": "互联网 / 电子 / 网游",
    "job_level": "",
    "employment_type": "intern",
    "work_mode": "onsite",
    "salary_text": "",
    "posted_time": "2025-01-15T10:30:00.123456",
    "tags": [],
    "summary": "你是否已经厌倦了在大厂按步就班做推进流程和搜索人才库的HR...",
    "discover_time": "2025-01-15T10:30:00.123456"
  }
]
```

## 🔧 技术细节

### HTML 解析策略
- 使用 BeautifulSoup + lxml 解析器
- 查找 `div.positionItem` 作为职位卡片
- 获取父级 `a` 标签提取 URL
- 使用 CSS 选择器提取各字段

### 雇佣类型标准化
```python
def parse_employment_type(emp_type: str) -> str:
    if "全职" in emp_type or "full" in emp_type:
        return "full-time"
    elif "实习" in emp_type or "intern" in emp_type:
        return "intern"
    # ...
```

### ID 提取逻辑
```python
def parse_job_id(href: str) -> str:
    match = re.search(r"/position/(\d+)/detail", href)
    return match.group(1) if match else ""
```

## ⚠️ 注意事项

1. **时间字段**：当前使用当前时间作为 `posted_time`，如果页面包含实际发布时间，需要优化提取逻辑
2. **职位级别**：未实现 `job_level` 字段，可根据职位标题关键词推断
3. **薪资信息**：页面可能不显示薪资信息
4. **标签**：当前未提取职位标签

## 🎓 扩展建议

### 1. 职位级别推断
可根据职位标题关键词（实习、资深、高级、专家等）推断职位级别。

### 2. 实际发布时间提取
如果页面包含实际发布时间，可以提取并转换为 ISO 格式。

### 3. 标签提取
可以提取页面中的职位标签（如高优、紧急）到 `tags` 数组。

### 4. 薪资信息提取
如果页面显示薪资信息，可以添加提取逻辑。

## ✅ 验证清单

- [x] 创建 `etl_minimax.py` ETL 脚本
- [x] 更新 `batch_etl.py` 集成 MiniMax
- [x] 创建 `test_minimax_etl.py` 测试脚本
- [x] 创建 `MINIMAX_ETL_README.md` 详细文档
- [x] 创建 `MINIMAX_QUICKSTART.md` 快速开始指南
- [x] 遵循 Google Python Style Guide
- [x] 完整的类型注解
- [x] 符合 schema.json 规范
- [x] 完善的错误处理
- [x] 清晰的文档字符串

## 📞 支持

如有问题，请参考：
- 详细文档：`MINIMAX_ETL_README.md`
- 快速开始：`MINIMAX_QUICKSTART.md`
- 项目规范：`PROMPT.md`

---

**完成时间**: 2025-01-15
**版本**: v1.0.0
**状态**: ✅ 已完成
