# MiniMax ETL 快速开始

## 📦 安装依赖

```bash
pip install beautifulsoup4 lxml
```

## 🚀 快速运行

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
# 处理所有网站（包括MiniMax）
python batch_etl.py

# 只处理MiniMax
python batch_etl.py --sites minimax
```

## 📊 查看输出

```bash
# 查看MiniMax职位数据
cat output/ds-minimax/jobs.json

# 查看合并后的所有职位
cat output/all_jobs.json
```

## ✅ 验证结果

```bash
# 统计MiniMax职位数量
python -c "import json; data=json.load(open('output/ds-minimax/jobs.json')); print(f'Total jobs: {len(data)}')"

# 查看第一个职位
python -c "import json; data=json.load(open('output/ds-minimax/jobs.json')); print(json.dumps(data[0], indent=2, ensure_ascii=False))"
```

## 📁 文件结构

```
job-hunter-v2/
├── etl_minimax.py              # MiniMax ETL脚本
├── test_minimax_etl.py         # 测试脚本
├── batch_etl.py                # 批量处理脚本（已集成MiniMax）
├── ds-minimax/                 # MiniMax数据源目录
│   ├── p-1.html
│   ├── p-2.html
│   └── p-3.html
└── output/
    └── ds-minimax/
        └── jobs.json           # 输出的MiniMax职位数据
```

## 🎯 核心功能

- ✅ 解析MiniMax招聘页面HTML
- ✅ 提取职位标题、URL、城市、类别等信息
- ✅ 标准化雇佣类型（全职/实习）
- ✅ 生成符合schema.json的JSON格式
- ✅ 支持批量处理多个页面
- ✅ 集成到batch_etl.py统一管理

## 📝 提取字段

| 字段 | 状态 | 说明 |
|------|------|------|
| id | ✅ | 从URL提取 |
| site_id | ✅ | 固定值"minimax" |
| title | ✅ | 职位标题 |
| url | ✅ | 完整详情页URL |
| company_name | ✅ | 固定值"MiniMax" |
| city | ✅ | 工作城市 |
| job_category | ✅ | 职位类别 |
| employment_type | ✅ | 雇佣类型（已标准化） |
| summary | ✅ | 职位描述（前200字） |
| posted_time | ✅ | 当前时间ISO格式 |
| discover_time | ✅ | 当前时间ISO格式 |

## 🔧 自定义配置

### 修改输入输出路径

编辑 `etl_minimax.py` 的 `__main__` 部分：

```python
if __name__ == "__main__":
    etl(
        html_file="ds-minimax/p-1.html",  # 修改输入路径
        output_json="output/minimax_jobs.json",  # 修改输出路径
        site_id="minimax"
    )
```

### 处理多个页面

编辑 `test_minimax_etl.py`：

```python
from etl_minimax import parse_job_brief, save_to_json
from pathlib import Path

if __name__ == "__main__":
    all_jobs = []

    # 处理所有页面
    for html_file in sorted(Path("ds-minimax").glob("p-*.html")):
        with open(html_file, "r", encoding="utf-8") as f:
            content = f.read()

        jobs = parse_job_brief(content, "minimax")
        all_jobs.extend(jobs)

    # 保存合并结果
    save_to_json(all_jobs, "output/minimax_jobs.json")
    print(f"共解析 {len(all_jobs)} 个职位")
```

## ❓ 常见问题

### Q1: 如何添加新的HTML页面？

将HTML文件放到 `ds-minimax/` 目录下，命名格式为 `p-*.html`，然后运行 `batch_etl.py`。

### Q2: 如何只处理MiniMax？

```bash
python batch_etl.py --sites minimax
```

### Q3: 输出的JSON格式不对？

检查是否安装了正确的依赖：

```bash
pip install beautifulsoup4 lxml
```

### Q4: 时间字段是当前时间，如何获取实际发布时间？

当前版本使用当前时间作为占位符。如果页面包含实际发布时间，可以修改 `etl_minimax.py` 中的 `parse_job_brief` 函数来提取。

## 📚 更多文档

详细使用说明请参考 [MINIMAX_ETL_README.md](./MINIMAX_ETL_README.md)

---

**版本**: v1.0.0
**更新时间**: 2025-01-15
