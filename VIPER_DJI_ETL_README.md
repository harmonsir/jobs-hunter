# 唯品会(Viper)和大疆(DJI) ETL脚本说明

## 概述

本文档说明如何使用唯品会和大疆的ETL脚本进行职位数据提取。

## 文件列表

- `etl_viper.py` - 唯品会招聘ETL脚本
- `etl_dji.py` - 大疆招聘ETL脚本
- `test_viper_dji.py` - 测试脚本
- `batch_etl.py` - 批量处理脚本(已更新,包含唯品会和大疆)

## 唯品会(Viper) ETL脚本

### 数据结构

唯品会的HTML结构:
- 职位卡片: `<div class="container-aOp138AX_X">`
- 职位链接: `<a class="link-txmgVOCVz9" href="#/job/{uuid}">`
- 职位标题: `<span class="title-u2qk9xX9Ie">职位名称</span>`
- 发布时间: `<span class="published-at-PQ5IBWmbJV">发布于 YYYY-MM-DD</span>`
- 职位性质: `<div class="sd-Ellipsis-hiddenContent-1Skwh">全职</div>`
- 工作地点: `<div class="sd-Ellipsis-hiddenContent-1Skwh">广东·广州市</div>`

### 提取字段

| 字段 | 提取方式 | 说明 |
|------|---------|------|
| id | 从URL中提取 | UUID格式 |
| site_id | 固定值 | "viper" |
| title | HTML文本 | 职位标题 |
| url | 拼接完整URL | https://app-tc.mokahr.com/#/job/{id} |
| company_name | 固定值 | "唯品会" |
| city | 解析位置文本 | 从"广东·广州市"提取"广州市" |
| location_text | HTML文本 | 完整位置文本 |
| job_category | 空字符串 | 未提供 |
| job_level | 推断 | 根据标题关键词推断 |
| employment_type | 标准化 | 全职->full-time, 实习->intern |
| work_mode | 固定值 | "onsite" |
| salary_text | 空字符串 | 未提供 |
| posted_time | 解析 | "发布于 YYYY-MM-DD" -> ISO格式 |
| tags | 空数组 | 未提供 |
| summary | 空字符串 | 未提供 |
| discover_time | 当前时间 | ISO格式 |

### 使用方法

```python
from etl_viper import etl

# 处理单个HTML文件
etl(
    html_file="ds-viper/p-1.html",
    output_json="output/viper_jobs.json",
    site_id="viper"
)
```

## 大疆(DJI) ETL脚本

### 数据结构

大疆的HTML结构:
- 职位卡片: `<li class="social_position_card__epffd">`
- 职位链接: `<a href="/zh-CN/position/detail?positionId={id}">`
- 职位标题: `<span class="PositionCard_text__2BdZa">职位名称</span>`
- 关键信息: `<p class="PositionCard_keyword__FFaH5">深圳市 \| 技术类 \| 研发团队 \| 2025-12-17</p>`
- 职位描述: `<p class="PositionCard_phase__o1mPk">...</p>`
- 热招标签: `<span class="PositionCard_hot__tTBG8">热招</span>`

### 提取字段

| 字段 | 提取方式 | 说明 |
|------|---------|------|
| id | 从URL中提取 | positionId参数 |
| site_id | 固定值 | "dji" |
| title | HTML文本 | 职位标题 |
| url | 拼接完整URL | https://we.dji.com/zh-CN/position/detail?positionId={id} |
| company_name | 固定值 | "大疆" |
| city | 解析关键词行 | 从"深圳市 \| 技术类 \| 研发团队 \| 2025-12-17"提取 |
| location_text | 城市名称 | 与city相同 |
| job_category | 解析关键词行 | 从"深圳市 \| 技术类 \| 研发团队 \| 2025-12-17"提取 |
| job_level | 推断 | 根据标题关键词推断 |
| employment_type | 固定值 | "full-time" |
| work_mode | 固定值 | "onsite" |
| salary_text | 空字符串 | 未提供 |
| posted_time | 解析关键词行 | 从"深圳市 \| 技术类 \| 研发团队 \| 2025-12-17"提取 |
| tags | 数组 | 包含"热招"(如果有)和团队名称 |
| summary | 职位描述前200字符 | 截取职位描述 |
| discover_time | 当前时间 | ISO格式 |

### 使用方法

```python
from etl_dji import etl

# 处理单个HTML文件
etl(
    html_file="ds-dji/p-1.html",
    output_json="output/dji_jobs.json",
    site_id="dji"
)
```

## 批量处理

使用批量处理脚本可以一次性处理所有网站(包括唯品会和大疆):

```bash
# 处理所有网站
python batch_etl.py

# 只处理唯品会和大疆
python batch_etl.py --sites viper dji
```

### 批量处理配置

唯品会和大疆已添加到`batch_etl.py`的配置中:

```python
SITE_CONFIGS = {
    # ... 其他网站配置 ...
    "viper": {
        "parser": parse_viper,
        "base_dir": "ds-viper",
        "output_file": "jobs.json",
        "site_id": "viper",
        "file_type": "html",
        "file_pattern": "p-*.html",
    },
    "dji": {
        "parser": parse_dji,
        "base_dir": "ds-dji",
        "output_file": "jobs.json",
        "site_id": "dji",
        "file_type": "html",
        "file_pattern": "p-*.html",
    },
}
```

## 测试

运行测试脚本来验证ETL脚本:

```bash
python test_viper_dji.py
```

测试脚本会:
1. 处理`ds-viper/p-1.html`,输出到`output/viper_jobs.json`
2. 处理`ds-dji/p-1.html`,输出到`output/dji_jobs.json`
3. 显示解析到的职位数量

## 输出示例

### 唯品会输出示例

```json
[
  {
    "id": "a1b257a8-15bd-49db-b619-41331f44d5a1",
    "site_id": "viper",
    "title": "主任算法工程师（搜索）",
    "url": "https://app-tc.mokahr.com/#/job/a1b257a8-15bd-49db-b619-41331f44d5a1",
    "company_name": "唯品会",
    "city": "广州市",
    "location_text": "广东·广州市",
    "job_category": "",
    "job_level": "senior",
    "employment_type": "full-time",
    "work_mode": "onsite",
    "salary_text": "",
    "posted_time": "2026-01-14T00:00:00",
    "tags": [],
    "summary": "",
    "discover_time": "2025-01-15T10:30:00"
  }
]
```

### 大疆输出示例

```json
[
  {
    "id": "1770787233154461696",
    "site_id": "dji",
    "title": "中级硬件工程师",
    "url": "https://we.dji.com/zh-CN/position/detail?positionId=1770787233154461696",
    "company_name": "大疆",
    "city": "深圳市",
    "location_text": "深圳市",
    "job_category": "技术类",
    "job_level": "mid-level",
    "employment_type": "full-time",
    "work_mode": "onsite",
    "salary_text": "",
    "posted_time": "2025-12-17T00:00:00",
    "tags": ["热招", "研发团队"],
    "summary": "1. 负责硬件需求分析、方案设计、器件选型、原理图设计，电路分析仿真；2. 负责PCB layout指导；3. 负责板级测试、整机测试，参与产品可靠性测试、转产以及生产的支持工作；4. 完成整机EMC测试与电路优化，协助产品认证相关工作；5. 负责产品硬件技术风险管理，解决硬件板级、模块或整机系统级别的疑难问题。",
    "discover_time": "2025-01-15T10:30:00"
  }
]
```

## 特殊处理说明

### 唯品会

1. **职位级别推断**: 根据职位标题中的关键词推断级别
   - "实习"/"intern" -> "intern"
   - "总监"/"专家"/"架构"/"leader" -> "expert"
   - "资深"/"高级"/"senior"/"主管"/"经理" -> "senior"
   - "初级"/"junior"/"助理" -> "junior"

2. **位置解析**: 从"广东·广州市"中提取城市名称"广州市"

3. **雇佣类型标准化**: 将"全职"、"实习"、"兼职"转换为标准值

### 大疆

1. **关键词行解析**: 从"深圳市 \| 技术类 \| 研发团队 \| 2025-12-17"中提取多个字段

2. **职位级别推断**: 根据职位标题中的关键词推断级别,增加了"中级"级别

3. **热招标签**: 如果职位有"热招"标签,添加到tags数组中

4. **职位摘要**: 截取职位描述的前200字符作为摘要

## 注意事项

1. **文件编码**: 所有HTML文件使用UTF-8编码

2. **异常处理**: 单个职位解析失败不会影响其他职位的处理

3. **时间格式**: 所有时间字段使用ISO 8601格式(YYYY-MM-DDTHH:MM:SS)

4. **必需字段**: 确保所有必需字段(id, site_id, title, url, posted_time)都存在

5. **可选字段**: 对于未提供的可选字段,使用空字符串或空数组

## 故障排查

### 常见问题

1. **解析不到职位**
   - 检查HTML文件是否存在
   - 检查HTML结构是否与预期一致
   - 查看错误输出信息

2. **职位数量不对**
   - 检查HTML文件是否完整
   - 检查是否有分页

3. **字段为空**
   - 检查HTML元素是否存在
   - 检查CSS类名是否正确

## 更新日志

### 2025-01-15
- 初始版本
- 支持唯品会(Viper)和大疆(DJI)的职位数据提取
- 集成到批量处理脚本中
