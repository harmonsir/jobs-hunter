# 快速开始指南

## 已创建的ETL脚本

### 1. 核心文件

| 文件名 | 说明 |
|--------|------|
| `etl_template.py` | ETL脚本模板,用于创建新网站的ETL脚本 |
| `etl_didi.py` | 滴滴招聘ETL脚本 |
| `etl_tencent.py` | 腾讯招聘ETL脚本 |
| `etl_xiaomi.py` | 小米招聘ETL脚本 |
| `batch_etl.py` | 批量处理脚本,支持多网站、多页面 |
| `test_etl.py` | 测试脚本,快速验证ETL功能 |

### 2. 文档文件

| 文件名 | 说明 |
|--------|------|
| `README_ETL.md` | 完整的ETL脚本使用文档 |
| `QUICKSTART.md` | 本文件,快速开始指南 |

## 快速开始

### 方式一: 批量处理所有网站 (推荐)

```bash
cd job-hunter-v2
python batch_etl.py
```

输出:
- `output/ds-didi/jobs.json` - 滴滴职位数据
- `output/ds-tencent/jobs.json` - 腾讯职位数据
- `output/ds-xiaomi/jobs.json` - 小米职位数据
- `output/all_jobs.json` - 所有职位合并数据

### 方式二: 单个网站处理

```bash
# 滴滴
python etl_didi.py

# 腾讯
python etl_tencent.py

# 小米
python etl_xiaomi.py
```

### 方式三: 测试运行

```bash
python test_etl.py
```

## 输出JSON格式

每个职位包含以下字段 (符合 `schema.json` 定义):

```json
{
  "id": "职位唯一ID",
  "site_id": "网站标识符",
  "title": "职位标题",
  "url": "职位详情URL",
  "company_name": "公司名称",
  "city": "工作城市",
  "location_text": "完整位置",
  "job_category": "职位类别",
  "job_level": "职位级别",
  "employment_type": "雇佣类型",
  "work_mode": "工作模式",
  "salary_text": "薪资描述",
  "posted_time": "发布时间(ISO格式)",
  "tags": ["标签数组"],
  "summary": "职位简介",
  "discover_time": "发现时间(ISO格式)"
}
```

## 添加新网站

### 步骤1: 创建ETL脚本

复制 `etl_template.py` 并重命名:

```bash
cp etl_template.py etl_yoursite.py
```

### 步骤2: 实现解析逻辑

编辑 `etl_yoursite.py`,实现 `parse_job_brief()` 函数:

```python
def parse_job_brief(html_content: str, site_id: str) -> list[dict[str, Any]]:
    soup = BeautifulSoup(html_content, "lxml")
    jobs = []
    
    # 解析HTML,提取职位信息
    # ...
    
    return jobs
```

### 步骤3: 添加到批量处理

编辑 `batch_etl.py`,在 `SITE_CONFIGS` 中添加:

```python
SITE_CONFIGS = {
    # ... 其他配置
    "yoursite": {
        "parser": parse_yoursite,  # 从etl_yoursite导入
        "base_dir": "ds-yoursite",
        "output_file": "jobs.json",
        "site_id": "yoursite",
    },
}
```

### 步骤4: 测试

```bash
python batch_etl.py
```

## 数据源目录结构

```
job-hunter-v2/
├── ds-didi/
│   └── p-1.html          # 滴滴HTML文件
├── ds-tencent/
│   └── p-1.html          # 腾讯HTML文件
├── ds-xiaomi/
│   ├── p-1.html          # 小米HTML文件(第1页)
│   ├── p-2.html          # 小米HTML文件(第2页)
│   └── ...
└── ds-yoursite/          # 你的网站数据源
    └── p-1.html
```

## 常用命令

```bash
# 处理所有网站
python batch_etl.py

# 只处理滴滴和腾讯
python -c "from batch_etl import batch_process; batch_process(sites=['didi', 'tencent'])"

# 测试单个网站
python etl_didi.py
```

## 下一步

- 查看 `README_ETL.md` 了解详细文档
- 查看 `schema.json` 了解数据结构定义
- 根据实际需求修改ETL脚本

## 支持

如有问题,请查看:
1. `README_ETL.md` - 完整文档
2. `schema.json` - 数据结构定义
3. 各ETL脚本中的注释和示例代码
