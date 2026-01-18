# 智谱AI (Zhipu AI) ETL脚本使用说明

## 概述

`etl_zhipu.py` 是用于解析智谱AI招聘数据的ETL脚本。该脚本从智谱AI的API响应JSON文件中提取职位信息，并将其转换为符合 `schema.json` 定义的标准化格式。

## 数据格式

智谱AI的数据是JSON格式，结构如下：

```json
{
  "code": 0,
  "data": {
    "job_post_list": [
      {
        "id": "职位ID",
        "title": "职位标题",
        "description": "职位描述",
        "requirement": "职位要求",
        "job_category": {
          "id": "类别ID",
          "name": "类别名称",
          "i18n_name": "国际化类别名称"
        },
        "recruit_type": {
          "id": "类型ID",
          "name": "类型名称（如：全职、实习）"
        },
        "publish_time": 1747725210956,
        "city_list": [
          {
            "code": "城市代码",
            "name": "城市名称"
          }
        ]
      }
    ],
    "count": 181
  },
  "message": "ok"
}
```

## 字段映射

| 智谱AI字段 | 标准字段 | 说明 |
|-----------|---------|------|
| `id` | `id` | 职位唯一标识 |
| `title` | `title` | 职位标题 |
| `city_list[0].name` | `city` | 工作城市 |
| `job_category.i18n_name` | `job_category` | 职位类别 |
| `recruit_type.name` | `employment_type` | 雇佣类型（映射为标准值） |
| `publish_time` | `posted_time` | 发布时间（毫秒时间戳 → ISO 8601） |
| `description` + `requirement` | `summary` | 职位摘要 |
| `id` | `url` | 构建职位详情URL |

## 雇佣类型映射

| 智谱AI类型 | 标准类型 |
|-----------|---------|
| 全职 | full-time |
| 实习 | intern |
| 兼职 | part-time |
| 外包 | contract |

## 职位级别推断

脚本根据职位标题自动推断职位级别：

| 标题关键词 | 职位级别 |
|-----------|---------|
| 首席、CTO、CFO、CEO | chief |
| 总监 | director |
| 负责人、技术负责人、主管、经理 | manager |
| 资深、高级、专家、架构师 | senior |
| 初级、助理 | junior |
| 实习 | intern |

## 使用方法

### 方法1: 单个文件测试

```bash
cd job-hunter-v2
python test_etl_zhipu.py
```

或直接运行：

```bash
python etl_zhipu.py
```

输出：
- `output/zhipu_jobs.json` - 解析后的职位数据

### 方法2: 批量处理所有文件

```bash
cd job-hunter-v2
python -c "from etl_zhipu import example_batch; example_batch()"
```

### 方法3: 通过 batch_etl.py 批量处理

```bash
cd job-hunter-v2
python batch_etl.py
```

这将处理所有配置的网站，包括智谱AI。

### 方法4: 只处理智谱AI

```bash
cd job-hunter-v2
python -c "from batch_etl import batch_process; batch_process(sites=['zhipu'])"
```

### 方法5: 批量处理并写入数据库

```bash
cd job-hunter-v2
python -c "from batch_etl import batch_process; batch_process(sites=['zhipu'], write_db=True)"
```

或：

```bash
cd job-hunter-v2
python -c "from etl_zhipu import example_batch; example_batch()"
```

## 输出格式

每个职位包含以下字段（符合 `schema.json` 定义）：

```json
{
  "id": "职位唯一ID",
  "site_id": "zhipu",
  "title": "职位标题",
  "url": "https://zhipuai.com/jobs/{id}",
  "company_name": "智谱AI",
  "city": "工作城市",
  "location_text": "工作城市",
  "job_category": "职位类别",
  "job_level": "职位级别（推断）",
  "employment_type": "雇佣类型",
  "work_mode": "onsite",
  "salary_text": "",
  "posted_time": "2024-01-15T10:30:00+00:00",
  "tags": [],
  "summary": "职位摘要...",
  "discover_time": "2024-01-15T10:30:00+00:00"
}
```

## 配置说明

智谱AI已在 `batch_etl.py` 中配置：

```python
"zhipu": {
    "parser": parse_zhipu,
    "base_dir": "ds-zhipu",
    "output_file": "jobs.json",
    "site_id": "zhipu",
    "file_type": "json",
    "file_pattern": "*.json",
}
```

## 数据源目录

```
job-hunter-v2/
└── ds-zhipu/
    ├── p-07fc77c5-7bd4-419d-8273-9c1b59c02c91.json
    ├── p-2c84a322-fb58-4a20-89b5-fea71df24c66.json
    └── p-4ff03bf0-fd10-4196-9a23-c3d5ea5d2bed.json
```

## 测试验证

运行测试脚本后，检查输出：

1. 查看控制台输出，确认解析的职位数量
2. 打开 `output/zhipu_jobs.json` 查看解析结果
3. 验证关键字段是否正确提取

## 注意事项

1. **时间戳处理**: 智谱AI使用毫秒时间戳，脚本会自动转换为ISO 8601格式
2. **城市列表**: 智谱AI使用 `city_list` 数组，脚本取第一个城市
3. **职位级别**: 通过标题关键词推断，可能不够准确
4. **薪资信息**: 智谱AI的JSON数据中不包含薪资信息，`salary_text` 为空
5. **URL构建**: 使用职位ID构建详情页URL，格式为 `https://zhipuai.com/jobs/{id}`

## 常见问题

### Q: 如何处理新添加的JSON文件？

A: 直接将新的JSON文件放入 `ds-zhipu/` 目录，然后运行 `batch_etl.py` 或 `example_batch()`。

### Q: 职位级别不准确怎么办？

A: 可以修改 `infer_job_level()` 函数，添加更多关键词或调整判断逻辑。

### Q: 如何添加薪资信息？

A: 需要先确认智谱AI的API响应中是否包含薪资字段，如果有，可以在 `parse_job_brief()` 函数中添加提取逻辑。

### Q: 如何处理数据重复？

A: 脚本在写入数据库时会根据 `url` 和 `id` 字段进行去重。

## 版本历史

- **v1.0.0** (2024-01-18): 初始版本
  - 支持解析智谱AI JSON格式数据
  - 自动推断职位级别
  - 支持批量处理和数据库写入

## 相关文件

- `etl_zhipu.py` - 智谱AI ETL脚本
- `batch_etl.py` - 批量处理脚本（已集成智谱AI）
- `test_etl_zhipu.py` - 测试脚本
- `schema.json` - 数据结构定义
- `docs/QUICKSTART-v2.md` - 快速开始指南
