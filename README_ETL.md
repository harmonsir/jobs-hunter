# Job Hunter ETL 脚本说明

## 概述

本项目包含针对不同招聘网站的ETL脚本,用于解析HTML页面并提取职位信息,输出符合 `schema.json` 定义的JSON格式。

## 文件结构

```
job-hunter-v2/
├── schema.json              # 数据结构定义
├── etl_template.py         # ETL脚本模板
├── etl_didi.py             # 滴滴招聘ETL脚本
├── etl_tencent.py          # 腾讯招聘ETL脚本
├── etl_xiaomi.py           # 小米招聘ETL脚本
├── etl_nio.py              # 蔚来(NIO)招聘ETL脚本
├── test_etl.py             # 单个网站测试脚本
├── batch_etl.py            # 批量处理脚本
├── README_ETL.md           # 本文档
├── ds-didi/                # 滴滴数据源目录
│   ├── p-1.html
│   └── jobs.json           # 输出JSON文件
├── ds-tencent/             # 腾讯数据源目录
│   ├── p-1.html
│   └── jobs.json           # 输出JSON文件
├── ds-xiaomi/             # 小米数据源目录
│   ├── p-1.html
│   ├── p-2.html
│   └── jobs.json           # 输出JSON文件
├── ds-nio/                # 蔚来数据源目录
│   ├── p-1.html
│   └── jobs.json           # 输出JSON文件
└── output/                # 批量处理输出目录
    ├── ds-didi/
    │   └── jobs.json
    ├── ds-tencent/
    │   └── jobs.json
    ├── ds-xiaomi/
    │   └── jobs.json
    ├── ds-nio/
    │   └── jobs.json
    └── all_jobs.json      # 合并后的所有职位
```

## Schema字段说明

基于 `schema.json` 中的 `job_brief` 定义:

### 必需字段
- `id`: 职位唯一标识符
- `site_id`: 源站点标识符
- `title`: 职位标题
- `url`: 职位详情页URL
- `posted_time`: 职位发布时间 (ISO 8601格式)

### 可选字段
- `company_name`: 公司名称
- `city`: 工作城市
- `location_text`: 完整位置文本
- `job_category`: 职位类别 (如 Engineering, Design, Product)
- `job_level`: 职位级别 (如 Junior, Senior, Lead, Manager)
- `employment_type`: 雇佣类型 (full-time, part-time, intern, contract)
- `work_mode`: 工作模式 (onsite, remote, hybrid)
- `salary_text`: 薪资范围文本
- `tags`: 职位标签数组
- `summary`: 职位简介或描述预览
- `discover_time`: 发现时间 (ISO 8601格式)

## 使用方法

### 方式一: 单个网站ETL

#### 1. 滴滴招聘

```bash
cd job-hunter-v2
python etl_didi.py
```

或直接运行:

```python
from etl_didi import etl

etl(
    html_file="ds-didi/p-1.html",
    output_json="ds-didi/jobs.json",
    site_id="didi"
)
```

#### 2. 腾讯招聘

```bash
cd job-hunter-v2
python etl_tencent.py
```

或直接运行:

```python
from etl_tencent import etl

etl(
    html_file="ds-tencent/p-1.html",
    output_json="ds-tencent/jobs.json",
    site_id="tencent"
)
```

#### 3. 小米招聘

```bash
cd job-hunter-v2
python etl_xiaomi.py
```

或直接运行:

```python
from etl_xiaomi import etl

etl(
    html_file="ds-xiaomi/p-1.html",
    output_json="ds-xiaomi/jobs.json",
    site_id="xiaomi"
)
```

#### 4. 蔚来(NIO)招聘

```bash
cd job-hunter-v2
python etl_nio.py
```

或直接运行:

```python
from etl_nio import etl

etl(
    html_file="ds-nio/p-1.html",
    output_json="ds-nio/jobs.json",
    site_id="nio"
)
```

### 方式二: 批量处理 (推荐)

批量处理所有配置的网站,自动合并所有职位数据:

```bash
cd job-hunter-v2
python batch_etl.py
```

批量处理特定网站:

```python
from batch_etl import batch_process

# 只处理滴滴和腾讯
batch_process(
    sites=["didi", "tencent"],
    output_dir="output",
    merge_all=True
)

# 只处理蔚来
batch_process(
    sites=["nio"],
    output_dir="output",
    merge_all=True
)
```

### 方式三: 测试运行

测试所有已配置的网站:

```bash
cd job-hunter-v2
python test_etl.py
```

## 输出格式

### 单个网站输出

```json
[
  {
    "id": "60495",
    "site_id": "didi",
    "title": "资深Java工程师/-墨西哥银行-银行核心",
    "url": "https://talent.didiglobal.com/social/p/60495",
    "company_name": "滴滴",
    "city": "上海市",
    "location_text": "上海市",
    "job_category": "技术",
    "job_level": "",
    "employment_type": "full-time",
    "work_mode": "onsite",
    "salary_text": "",
    "posted_time": "2026-01-14T20:30:00",
    "tags": [],
    "summary": "Fintech Technology",
    "discover_time": "2025-01-15T10:30:00"
  }
]
```

### 合并输出 (all_jobs.json)

```json
{
  "total_jobs": 65,
  "total_sites": 4,
  "sites": {
    "didi": 15,
    "tencent": 20,
    "xiaomi": 15,
    "nio": 15
  },
  "generated_at": "2025-01-15T10:30:00",
  "jobs": [...]
}
```

## 各网站特性说明

### 滴滴招聘
- **职位ID**: 从URL中提取,如 `/social/p/60495` -> `60495`
- **时间格式**: `YYYY-MM-DD HH:MM`
- **详情格式**: `部门 / 职位类别 / 城市`
- **特点**: 包含部门信息,职位分类清晰

### 腾讯招聘
- **职位ID**: 从URL中提取
- **时间格式**: 相对时间 (如 "3天前")
- **特点**: 包含事业群信息,职位级别通过经验自动推断

### 小米招聘
- **职位ID**: 从URL中提取
- **时间格式**: 相对时间
- **特点**: 多页面支持,雇佣类型和招聘类型明确标注

### 蔚来(NIO)招聘
- **职位ID**: 从URL中提取,如 `/jobs/12345` -> `12345`
- **时间格式**: 支持多种格式
  - 绝对时间: `2026-01-15`
  - 相对时间: `3天前`, `5小时前`, `30分钟前`
- **标签格式**: `城市 · 薪资 · 经验 · 学历`
  - 示例: `上海 · 30-50K · 3-5年 · 本科`
- **职位级别**: 根据工作经验自动推断
  - 实习 → intern
  - 应届/0-1年 → junior
  - 3-5年 → middle
  - 5-10年 → senior
  - 10年以上 → expert
- **特点**: 标签信息丰富,支持多维度筛选

## 为新网站创建ETL脚本

### 步骤

1. **复制模板**: 复制 `etl_template.py` 并重命名,如 `etl_dji.py`

2. **分析HTML结构**: 使用浏览器开发者工具或BeautifulSoup分析目标网站的HTML结构

3. **实现解析逻辑**: 修改 `parse_job_brief()` 函数,提取所需字段

4. **添加到批量处理**: 在 `batch_etl.py` 中添加网站配置

5. **测试验证**: 运行脚本并检查输出的JSON是否符合schema

### 示例代码结构

```python
def parse_job_brief(html_content: str, site_id: str) -> list[dict[str, Any]]:
    soup = BeautifulSoup(html_content, "lxml")
    jobs = []
    
    # 1. 查找职位卡片容器
    job_cards = soup.find_all("div", class_="job-card")
    
    for card in job_cards:
        # 2. 提取必需字段
        job_id = extract_id(card)
        title = extract_title(card)
        url = extract_url(card)
        posted_time = extract_posted_time(card)
        
        # 3. 提取可选字段
        company_name = extract_company(card)
        city = extract_city(card)
        # ... 其他字段
        
        # 4. 构建职位对象
        job = {
            "id": job_id,
            "site_id": site_id,
            "title": title,
            "url": url,
            "posted_time": posted_time,
            # ... 其他字段
            "discover_time": datetime.now().isoformat(),
        }
        
        jobs.append(job)
    
    return jobs
```

### 添加到批量处理

在 `batch_etl.py` 的 `SITE_CONFIGS` 中添加:

```python
SITE_CONFIGS = {
    # ... 其他网站配置
    "dji": {
        "parser": parse_dji,
        "base_dir": "ds-dji",
        "output_file": "jobs.json",
        "site_id": "dji",
    },
}
```

## 依赖项

```bash
pip install beautifulsoup4 lxml
```

## 注意事项

1. **编码问题**: 确保HTML文件使用UTF-8编码
2. **时间格式**: 所有时间字段使用ISO 8601格式
3. **必填字段**: 确保至少包含id, site_id, title, url, posted_time
4. **异常处理**: 建议在解析单个职位时添加try-except,避免一个解析失败影响整体
5. **字段映射**: 根据网站实际情况调整字段映射逻辑
6. **多页面支持**: 小米等网站有多个页面,批量处理脚本会自动处理所有p-*.html文件
7. **相对时间**: 蔚来等网站使用相对时间,脚本会自动转换为绝对时间

## 扩展建议

1. **增量更新**: 根据posted_time过滤已处理的职位
2. **数据验证**: 添加schema验证确保数据质量
3. **日志记录**: 添加详细的日志记录便于调试
4. **并发处理**: 对于大量数据,考虑使用多线程/多进程加速处理
5. **去重处理**: 根据id和site_id对职位进行去重
6. **数据统计**: 添加更详细的数据统计和分析功能
7. **智能推断**: 参考蔚来脚本,根据经验自动推断职位级别

## 常见问题

### Q: 如何处理多个HTML文件?

A: 使用 `batch_etl.py`,它会自动处理每个网站目录下的所有 `p-*.html` 文件。

### Q: 如何只处理特定页面?

A: 修改 `batch_etl.py` 中的 `process_site()` 函数,传入指定的HTML文件列表。

### Q: 如何合并不同网站的职位?

A: 使用 `batch_etl.py` 并设置 `merge_all=True`,会自动生成 `all_jobs.json`。

### Q: 如何验证输出的JSON是否符合schema?

A: 可以使用jsonschema库进行验证:

```python
import json
from jsonschema import validate

with open("schema.json", "r", encoding="utf-8") as f:
    schema = json.load(f)

with open("output.json", "r", encoding="utf-8") as f:
    jobs = json.load(f)

for job in jobs:
    validate(instance=job, schema=schema["definitions"]["job_brief"])
```

### Q: 蔚来网站的相对时间如何处理?

A: `etl_nio.py` 中的 `parse_posted_time()` 函数会自动将相对时间转换为绝对时间:
- `3天前` → 3天前的ISO时间
- `5小时前` → 5小时前的ISO时间
- `30分钟前` → 30分钟前的ISO时间

### Q: 如何为蔚来网站添加更多页面?

A: 将HTML文件保存到 `ds-nio/` 目录,命名为 `p-1.html`, `p-2.html` 等,批量处理脚本会自动处理所有页面。
