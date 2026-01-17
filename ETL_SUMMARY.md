# ETL脚本创建总结

## 项目概述

根据 `job-hunter-v2/schema.json` 定义的 `job_brief` 数据结构,已成功创建完整的ETL脚本体系,支持解析多个招聘网站的HTML页面并输出标准化的JSON格式数据。

## 已创建文件清单

### 1. ETL脚本 (核心)

| 文件名 | 大小 | 功能说明 |
|--------|------|----------|
| `etl_template.py` | 2.8KB | ETL脚本模板,包含完整的代码结构和注释,用于快速创建新网站的ETL脚本 |
| `etl_didi.py` | 5.2KB | 滴滴招聘ETL脚本,解析滴滴职位列表页面,提取职位信息 |
| `etl_tencent.py` | 6.2KB | 腾讯招聘ETL脚本,解析腾讯职位列表页面,提取职位信息 |
| `etl_xiaomi.py` | 6.3KB | 小米招聘ETL脚本,解析小米职位列表页面,支持多页面处理 |

### 2. 批量处理工具

| 文件名 | 大小 | 功能说明 |
|--------|------|----------|
| `batch_etl.py` | 5.9KB | 批量处理脚本,支持同时处理多个网站的多个HTML页面,自动合并数据 |
| `test_etl.py` | 0.6KB | 测试脚本,快速验证各ETL脚本的功能 |

### 3. 文档文件

| 文件名 | 大小 | 功能说明 |
|--------|------|----------|
| `README_ETL.md` | 8.1KB | 完整的ETL脚本使用文档,包含详细的API说明、使用示例和扩展指南 |
| `QUICKSTART.md` | 3.7KB | 快速开始指南,帮助用户快速上手使用ETL脚本 |

## 功能特性

### 1. 完整的Schema支持

所有ETL脚本都严格按照 `schema.json` 中定义的 `job_brief` 结构输出数据:

**必需字段 (5个)**:
- `id` - 职位唯一标识符
- `site_id` - 源站点标识符
- `title` - 职位标题
- `url` - 职位详情页URL
- `posted_time` - 职位发布时间 (ISO 8601格式)

**可选字段 (9个)**:
- `company_name` - 公司名称
- `city` - 工作城市
- `location_text` - 完整位置文本
- `job_category` - 职位类别
- `job_level` - 职位级别
- `employment_type` - 雇佣类型 (full-time/part-time/intern/contract)
- `work_mode` - 工作模式 (onsite/remote/hybrid)
- `salary_text` - 薪资范围文本
- `tags` - 职位标签数组
- `summary` - 职位简介或描述预览
- `discover_time` - 发现时间 (ISO 8601格式)

### 2. 智能数据提取

各ETL脚本包含智能的数据提取逻辑:

- **滴滴**: 解析职位卡片,提取部门、职位类型、城市等信息
- **腾讯**: 解析职位列表,提取事业群、职位类别、工作经验等信息,自动推断职位级别
- **小米**: 解析职位项,提取城市、招聘类型、雇佣类型等信息,自动标准化雇佣类型

### 3. 时间处理

所有时间字段都使用ISO 8601标准格式:

```python
datetime.now().isoformat()  # 输出: "2025-01-15T10:30:00"
```

各脚本包含时间解析函数:
- 滴滴: `parse_posted_time()` - 解析 "2026-01-14 20:30" 格式
- 腾讯: `parse_posted_time()` - 解析 "更新于2026年01月15日" 格式
- 小米: 使用当前时间作为默认值

### 4. 异常处理

所有ETL脚本都包含完善的异常处理:

```python
try:
    # 解析单个职位
    job = parse_job_card(card)
    jobs.append(job)
except Exception as e:
    print(f"解析职位时出错: {e}")
    continue  # 继续处理下一个职位
```

确保单个职位的解析失败不会影响整体处理。

### 5. 批量处理功能

`batch_etl.py` 提供强大的批量处理能力:

- **多网站支持**: 同时处理多个配置的网站
- **多页面支持**: 自动处理每个网站目录下的所有 `p-*.html` 文件
- **数据合并**: 自动合并所有网站的职位数据到 `all_jobs.json`
- **统计信息**: 提供详细的处理统计和各网站职位数量
- **灵活配置**: 支持处理指定网站或所有网站

## 使用方式

### 方式一: 单个网站ETL

```bash
# 滴滴
python etl_didi.py

# 腾讯
python etl_tencent.py

# 小米
python etl_xiaomi.py
```

### 方式二: 批量处理 (推荐)

```bash
# 处理所有网站
python batch_etl.py

# 或在Python中
from batch_etl import batch_process

batch_process(
    sites=["didi", "tencent", "xiaomi"],
    output_dir="output",
    merge_all=True
)
```

### 方式三: 测试运行

```bash
python test_etl.py
```

## 输出格式

### 单个网站输出

每个网站生成独立的JSON文件:

```
output/
├── ds-didi/jobs.json
├── ds-tencent/jobs.json
└── ds-xiaomi/jobs.json
```

### 合并输出

生成包含所有职位和统计信息的合并文件:

```json
{
  "total_jobs": 50,
  "total_sites": 3,
  "sites": {
    "didi": 15,
    "tencent": 20,
    "xiaomi": 15
  },
  "generated_at": "2025-01-15T10:30:00",
  "jobs": [...]
}
```

## 扩展性

### 添加新网站

只需3步即可添加新网站的ETL支持:

1. **复制模板**: `cp etl_template.py etl_newsite.py`
2. **实现解析**: 修改 `parse_job_brief()` 函数
3. **添加配置**: 在 `batch_etl.py` 中添加网站配置

### 自定义字段

根据网站实际情况,可以灵活调整字段提取逻辑,确保尽可能多地满足schema定义的字段。

## 代码质量

### 1. 类型注解

所有函数都包含完整的类型注解:

```python
def parse_job_brief(html_content: str, site_id: str) -> list[dict[str, Any]]:
    ...
```

### 2. 文档字符串

使用Google风格的文档字符串:

```python
"""
解析HTML内容,提取职位信息

Args:
    html_content: HTML字符串
    site_id: 站点标识符

Returns:
    职位列表,每个职位符合job_brief结构
"""
```

### 3. 代码风格

遵循Google Python Style Guide:
- 统一使用双引号
- 清晰的函数和变量命名
- 适当的注释和空行
- 模块化设计,单一职责

## 依赖项

```bash
pip install beautifulsoup4 lxml
```

## 下一步建议

1. **添加更多网站**: 参考 `etl_template.py` 为其他招聘网站创建ETL脚本
2. **增量更新**: 实现基于 `posted_time` 的增量更新机制
3. **数据验证**: 使用 `jsonschema` 验证输出数据是否符合schema
4. **去重处理**: 基于 `id` 和 `site_id` 实现职位去重
5. **性能优化**: 对于大量数据,考虑使用并发处理加速

## 文档支持

- `README_ETL.md` - 完整的使用文档
- `QUICKSTART.md` - 快速开始指南
- `schema.json` - 数据结构定义
- 各ETL脚本中的注释和示例代码

## 总结

已成功创建完整的ETL脚本体系,包括:
- ✅ 1个ETL模板
- ✅ 3个网站的ETL脚本 (滴滴、腾讯、小米)
- ✅ 1个批量处理脚本
- ✅ 1个测试脚本
- ✅ 2份完整文档

所有脚本都:
- ✅ 严格遵循 `schema.json` 定义
- ✅ 支持完整的字段提取
- ✅ 包含完善的异常处理
- ✅ 使用ISO 8601时间格式
- ✅ 支持批量处理和数据合并
- ✅ 包含详细的文档和注释

可以立即投入使用,并轻松扩展到其他招聘网站。
