# 字节跳动 ETL 脚本说明

## 概述

`etl_bytedance.py` 是用于处理字节跳动招聘数据的 ETL 脚本。与其他网站的 ETL 脚本不同,字节跳动的数据源是 JSON API 响应格式,而不是 HTML 页面。

## 数据来源

- **数据格式**: JSON API 响应
- **数据目录**: `ds-bytedance/`
- **文件命名**: `p-*.json` (如 `p-00bf5dd3-36c2-44f5-8f8f-205b7a8a1753.json`)

## 数据结构

### API 响应结构

```json
{
  "code": 0,
  "data": {
    "job_post_list": [
      {
        "id": "7526762608633563399",
        "title": "数据挖掘算法工程师（营销方向）-Data",
        "description": "...",
        "requirement": "...",
        "job_category": {
          "id": "6704215956018694411",
          "name": "算法",
          "en_name": "Algorithm",
          "i18n_name": "算法"
        },
        "city_info": {
          "code": "CT_11",
          "name": "北京",
          "en_name": "Beijing"
        },
        "recruit_type": {
          "id": "101",
          "name": "正式",
          "en_name": "Regular"
        },
        "publish_time": 1752461421047,
        "code": "A110507"
      }
    ],
    "count": 10000
  },
  "message": "ok"
}
```

## 字段映射

### 输入字段 → 输出字段

| 输入字段 | 输出字段 | 说明 |
|---------|---------|------|
| `id` | `id` | 职位唯一标识符 |
| - | `site_id` | 固定为 "bytedance" |
| `title` | `title` | 职位标题 |
| `code` | `url` | 构建为 `https://jobs.bytedance.com/experienced/position/{code}` |
| - | `company_name` | 固定为 "字节跳动" |
| `city_info.name` | `city` | 工作城市 |
| `city_info.name` | `location_text` | 完整位置文本 |
| `job_category.i18n_name` | `job_category` | 职位类别 |
| - | `job_level` | 根据标题推断 |
| `recruit_type.name` | `employment_type` | 映射为标准类型 |
| - | `work_mode` | 固定为 "onsite" |
| - | `salary_text` | 暂无数据 |
| `publish_time` | `posted_time` | 转换为 ISO 8601 格式 |
| - | `tags` | 暂无数据 |
| `description` + `requirement` | `summary` | 组合后截取前 300 字符 |
| - | `discover_time` | 当前时间 (ISO 8601) |

### 雇佣类型映射

| 招聘类型 | 标准类型 |
|---------|---------|
| 实习 / intern | `intern` |
| 兼职 / part | `part-time` |
| 外包 / contract | `contract` |
| 其他 | `full-time` |

### 职位级别推断

根据职位标题中的关键词推断:

| 关键词 | 职位级别 |
|-------|---------|
| 负责人、技术负责人、主管、经理、manager、lead、head | `Manager` |
| 总监、director | `Director` |
| 资深、高级、senior、专家、principal、architect | `Senior` |
| 初级、助理、assistant、junior | `Junior` |
| 实习 / intern | `intern` |
| 其他 | 空字符串 |

## 使用方法

### 单独运行

```bash
python etl_bytedance.py
```

或使用测试脚本:

```bash
python test_bytedance_etl.py
```

### 批量处理

在 `batch_etl.py` 中已经集成了字节跳动的配置:

```bash
python batch_etl.py
```

或只处理字节跳动:

```python
from batch_etl import batch_process

batch_process(
    sites=["bytedance"],
    output_dir="output",
    merge_all=False
)
```

### 验证解析结果

```bash
python verify_bytedance.py
```

## 输出示例

```json
[
  {
    "id": "7526762608633563399",
    "site_id": "bytedance",
    "title": "数据挖掘算法工程师（营销方向）-Data",
    "url": "https://jobs.bytedance.com/experienced/position/A110507",
    "company_name": "字节跳动",
    "city": "北京",
    "location_text": "北京",
    "job_category": "算法",
    "job_level": "",
    "employment_type": "full-time",
    "work_mode": "onsite",
    "salary_text": "",
    "posted_time": "2025-07-14T08:17:01.047000+00:00",
    "tags": [],
    "summary": "1、负责多种数据源（SDK、运营商、支付等）的数据分析、特征构建、建模出包等流程；\n2、负责单个数据源建模、多个数据源联合建模。\n\n要求:\n1、熟悉多种数据源的特性、与业务匹配度等，并拥有实践项目经验；\n2、掌握沙箱环境建模技术与流程，能搭建沙箱建模环境的优先；\n3、掌握数据挖掘模型算法、大数据分析技术与工具；\n4、动手能力强，熟悉SQL，Python，能熟练处理海量数据；\n5、良好的逻辑思维能力，优秀的数据敏感度，定位数据问题和分析数据能力优；\n6、良好的团队合作精神，较强的沟通能力。",
    "discover_time": "2025-01-16T12:00:00.000000+00:00"
  }
]
```

## 特殊处理

### 1. 时间戳转换

字节跳动的 `publish_time` 是毫秒级时间戳,需要转换为 ISO 8601 格式:

```python
from datetime import datetime, timezone

timestamp_ms = 1752461421047
dt = datetime.fromtimestamp(timestamp_ms / 1000, tz=timezone.utc)
iso_time = dt.isoformat()  # "2025-07-14T08:17:01.047000+00:00"
```

### 2. JSON 文件处理

与其他网站的 HTML 解析不同,字节跳动直接解析 JSON:

```python
def parse_job_brief(json_content: str, site_id: str) -> list[dict[str, Any]]:
    data = json.loads(json_content)
    job_post_list = data.get("data", {}).get("job_post_list", [])
    # ...
```

### 3. 错误处理

- JSON 解析失败时返回空列表
- API 响应错误时返回空列表
- 单个职位解析失败时跳过,继续处理其他职位

## 与其他网站的差异

| 特性 | 其他网站 | 字节跳动 |
|-----|---------|---------|
| 数据格式 | HTML | JSON API 响应 |
| 解析库 | BeautifulSoup | json 模块 |
| 时间格式 | 相对/绝对时间 | 毫秒时间戳 |
| URL 构建 | 从 HTML 提取 | 根据 code 构建 |
| 文件扩展名 | `.html` | `.json` |

## 集成到批量处理

`batch_etl.py` 已更新,支持字节跳动的批量处理:

```python
SITE_CONFIGS = {
    # ... 其他网站 ...
    "bytedance": {
        "parser": parse_bytedance,
        "base_dir": "ds-bytedance",
        "output_file": "jobs.json",
        "site_id": "bytedance",
        "file_type": "json",  # 关键:指定文件类型
        "file_pattern": "*.json",
    },
}
```

## 注意事项

1. **文件类型**: 字节跳动的数据源是 JSON,不是 HTML
2. **时间戳**: 需要将毫秒时间戳转换为 ISO 8601 格式
3. **URL 构建**: 需要根据 `code` 字段构建完整的 URL
4. **职位级别**: 目前根据标题关键词推断,可能不够准确
5. **标签**: 当前数据源不包含标签信息,留空

## 未来改进

- [ ] 从职位描述中提取更多标签
- [ ] 更精确的职位级别推断逻辑
- [ ] 添加薪资信息的提取(如果 API 返回)
- [ ] 支持多文件批量处理
- [ ] 添加数据验证和质量检查

## 相关文件

- `etl_bytedance.py` - 主 ETL 脚本
- `test_bytedance_etl.py` - 测试脚本
- `verify_bytedance.py` - 验证脚本
- `batch_etl.py` - 批量处理脚本(已更新)
- `ds-bytedance/` - 数据目录
- `output/bytedance_jobs.json` - 输出文件
