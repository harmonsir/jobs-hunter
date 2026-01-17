# MiniMax ETL 使用说明

## 概述

MiniMax ETL 脚本用于解析 MiniMax 招聘网站的 HTML 页面，提取职位信息并导出为符合 schema.json 标准的 JSON 格式。

## 文件说明

- `etl_minimax.py` - MiniMax ETL 主脚本
- `ds-minimax/` - MiniMax HTML 数据源目录
  - `p-1.html`, `p-2.html`, `p-3.html` - 分页的职位列表页面

## 快速开始

### 1. 运行单个页面测试

```bash
python test_minimax_etl.py
```

或直接运行 ETL 脚本：

```bash
python etl_minimax.py
```

### 2. 集成到批量处理

```bash
# 处理所有网站（包括 MiniMax）
python batch_etl.py

# 只处理 MiniMax
python batch_etl.py --sites minimax
```

## 数据提取字段

### 必需字段（5个）

| 字段名 | 提取方式 | 示例 |
|--------|----------|------|
| `id` | 从 URL 提取 | `7594839691242359067` |
| `site_id` | 固定值 | `minimax` |
| `title` | 职位标题文本 | `HR 招聘日常实习生（职能方向）` |
| `url` | 拼接完整 URL | `https://hr.minimax.ai/index/position/7594839691242359067/detail` |
| `posted_time` | 当前时间 ISO 格式 | `2025-01-15T10:30:00` |

### 可选字段（已实现）

| 字段名 | 提取方式 | 示例 |
|--------|----------|------|
| `company_name` | 固定值 | `MiniMax` |
| `city` | 副标题第一个 span | `北京` / `上海` / `北京、上海` |
| `location_text` | 同 city | `北京` / `上海` |
| `job_category` | 职位类别 | `互联网 / 电子 / 网游` / `研发 - 基础架构` |
| `job_level` | 未实现 | - |
| `employment_type` | 雇佣类型标准化 | `full-time` / `intern` |
| `work_mode` | 固定值 | `onsite` |
| `salary_text` | 未实现 | - |
| `tags` | 固定空数组 | `[]` |
| `summary` | 职位描述前 200 字 | `你是否已经厌倦了在大厂按步就班...` |
| `discover_time` | 当前时间 ISO 格式 | `2025-01-15T10:30:00` |

## HTML 结构解析

### 职位卡片结构

```html
<a href="/index/position/7594839691242359067/detail">
  <div class="positionItem__fca8c0 positionItem">
    <!-- 职位标题 -->
    <div class="title__fca8c0 positionItem-title sofiaBold">
      <span class="positionItem-title-text">HR 招聘日常实习生（职能方向）</span>
    </div>

    <!-- 副标题：城市、招聘类型、雇佣类型、职位类别 -->
    <div class="subTitle__fca8c0 positionItem-subTitle">
      <span>北京</span>
      <div class="lineDevider__77ceb8"></div>
      <span class="infoText__fca8c0">社招</span>
      <div class="lineDevider__77ceb8"></div>
      <span class="infoText__fca8c0">全职</span>
      <div class="lineDevider__77ceb8"></div>
      <span class="infoText-category__fca8c0">互联网 / 电子 / 网游</span>
    </div>

    <!-- 职位描述 -->
    <div class="jobDesc__fca8c0 positionItem-jobDesc">
      你是否已经厌倦了在大厂按步就班...
    </div>
  </div>
</a>
```

### 解析逻辑

1. **查找职位卡片**：`soup.find_all("div", class_="positionItem")`
2. **获取父级链接**：`item.find_parent("a")`
3. **提取 URL 和 ID**：正则匹配 `/position/(\d+)/detail`
4. **提取标题**：`item.find("span", class_="positionItem-title-text")`
5. **提取城市**：副标题第一个 span
6. **提取雇佣类型**：副标题第三个 `infoText` span，标准化为 `full-time` / `intern`
7. **提取职位类别**：`item.find("span", class_="infoText-category__fca8c0")`
8. **提取描述**：`item.find("div", class_="positionItem-jobDesc")`

## 雇佣类型标准化

| 原始文本 | 标准化值 |
|----------|----------|
| 全职 | `full-time` |
| 兼职 | `part-time` |
| 实习 | `intern` |
| 合同 | `contract` |

## 输出示例

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

## 错误处理

- 单个职位解析失败不影响整体流程
- 异常会被捕获并打印错误信息
- 缺失字段会设置为默认值（空字符串或空数组）

## 注意事项

1. **时间字段**：当前使用当前时间作为 `posted_time`，如果页面包含实际发布时间，可以优化提取逻辑
2. **职位级别**：当前未实现 `job_level` 字段提取，可根据职位标题推断
3. **薪资信息**：当前未实现 `salary_text` 字段提取，页面中可能不显示薪资
4. **多城市**：如果职位支持多个城市（如 `北京、上海`），`city` 字段会保留完整文本

## 扩展建议

### 1. 职位级别推断

可根据职位标题关键词推断级别：

```python
def infer_job_level(title: str) -> str:
    """根据职位标题推断级别"""
    if any(kw in title for kw in ["实习", "intern"]):
        return "intern"
    elif any(kw in title for kw in ["资深", "高级", "senior"]):
        return "senior"
    elif any(kw in title for kw in ["专家", "架构", "lead"]):
        return "lead"
    return ""
```

### 2. 时间提取优化

如果页面包含实际发布时间，可以提取并转换为 ISO 格式。

### 3. 标签提取

可以提取职位标签（如 `高优`、`紧急`）到 `tags` 数组。

## 版本历史

- **v1.0.0** (2025-01-15) - 初始版本，支持基础字段提取
