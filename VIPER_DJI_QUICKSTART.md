# 唯品会和大疆 ETL 快速开始指南

## 快速开始

### 1. 测试单个网站

```bash
# 测试唯品会
python -c "from etl_viper import etl; etl('ds-viper/p-1.html', 'output/viper_jobs.json', 'viper')"

# 测试大疆
python -c "from etl_dji import etl; etl('ds-dji/p-1.html', 'output/dji_jobs.json', 'dji')"
```

### 2. 测试两个网站

```bash
python test_viper_dji.py
```

### 3. 批量处理所有网站(包括唯品会和大疆)

```bash
python batch_etl.py
```

### 4. 批量处理指定网站

```python
from batch_etl import batch_process

# 只处理唯品会和大疆
batch_process(
    sites=["viper", "dji"],
    output_dir="output",
    merge_all=True
)
```

## 验证输出

### 检查JSON格式

```bash
# 验证唯品会输出
python -m json.tool output/viper_jobs.json > /dev/null && echo "格式正确"

# 验证大疆输出
python -m json.tool output/dji_jobs.json > /dev/null && echo "格式正确"
```

### 统计职位数量

```bash
# 统计唯品会职位数
python -c "import json; data=json.load(open('output/viper_jobs.json')); print(f'唯品会: {len(data)} 个职位')"

# 统计大疆职位数
python -c "import json; data=json.load(open('output/dji_jobs.json')); print(f'大疆: {len(data)} 个职位')"
```

### 查看示例职位

```bash
# 查看唯品会第一个职位
python -c "import json; data=json.load(open('output/viper_jobs.json')); print(json.dumps(data[0], indent=2, ensure_ascii=False))"

# 查看大疆第一个职位
python -c "import json; data=json.load(open('output/dji_jobs.json')); print(json.dumps(data[0], indent=2, ensure_ascii=False))"
```

## 文件结构

```
job-hunter-v2/
├── etl_viper.py              # 唯品会ETL脚本
├── etl_dji.py                # 大疆ETL脚本
├── test_viper_dji.py         # 测试脚本
├── batch_etl.py              # 批量处理脚本(已更新)
├── VIPER_DJI_ETL_README.md   # 详细说明文档
├── ds-viper/                 # 唯品会HTML数据目录
│   ├── p-1.html
│   ├── p-2.html
│   ├── p-3.html
│   ├── p-4.html
│   └── p-5.html
├── ds-dji/                   # 大疆HTML数据目录
│   └── p-1.html
└── output/                   # 输出目录
    ├── viper_jobs.json        # 唯品会职位数据
    ├── dji_jobs.json         # 大疆职位数据
    └── all_jobs.json         # 所有网站合并数据
```

## 关键特性

### 唯品会(Viper)

- ✅ 提取职位ID(UUID格式)
- ✅ 提取职位标题
- ✅ 提取发布时间(ISO格式)
- ✅ 提取工作地点
- ✅ 解析雇佣类型(全职/实习/兼职)
- ✅ 推断职位级别
- ✅ 标准化输出格式

### 大疆(DJI)

- ✅ 提取职位ID(positionId)
- ✅ 提取职位标题
- ✅ 提取职位类别
- ✅ 提取团队信息
- ✅ 提取发布时间(ISO格式)
- ✅ 提取职位描述摘要
- ✅ 提取热招标签
- ✅ 推断职位级别
- ✅ 标准化输出格式

## 常见问题

### Q1: 如何处理多个HTML文件?

唯品会和大疆都支持批量处理多个HTML文件,文件命名格式为`p-1.html`, `p-2.html`, ...

```python
from batch_etl import process_site

# 处理唯品会所有页面
jobs = process_site("viper")

# 处理大疆所有页面
jobs = process_site("dji")
```

### Q2: 如何合并两个网站的职位数据?

```python
from batch_etl import batch_process

# 批量处理并合并
batch_process(
    sites=["viper", "dji"],
    output_dir="output",
    merge_all=True
)
```

### Q3: 如何自定义输出路径?

```python
from etl_viper import etl
from etl_dji import etl

# 自定义输出路径
etl("ds-viper/p-1.html", "custom/viper_output.json", "viper")
etl("ds-dji/p-1.html", "custom/dji_output.json", "dji")
```

### Q4: 如何处理解析失败的职位?

脚本会自动跳过解析失败的职位,并打印错误信息,不会影响其他职位的处理。

### Q5: 时间格式是什么?

所有时间字段使用ISO 8601格式: `YYYY-MM-DDTHH:MM:SS`

例如: `2026-01-14T00:00:00`

## 下一步

1. 查看详细文档: `VIPER_DJI_ETL_README.md`
2. 运行测试脚本: `python test_viper_dji.py`
3. 批量处理所有网站: `python batch_etl.py`
4. 验证输出数据: 检查JSON格式和字段完整性

## 技术支持

如遇问题,请检查:
1. Python版本 >= 3.10
2. 依赖库已安装: `pip install beautifulsoup4 lxml`
3. HTML文件编码为UTF-8
4. 文件路径正确
