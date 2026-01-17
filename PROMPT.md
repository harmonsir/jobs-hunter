# Job Hunter V2 - ETL系统构建任务

## 📋 项目概述

构建一个职位数据ETL系统,从多个招聘网站的HTML页面中提取职位信息,并按照统一的JSON Schema输出结构化数据。

### 核心目标
1. **一个网站一个ETL脚本** - 每个招聘网站对应独立的ETL脚本
2. **统一数据结构** - 所有脚本输出符合 `schema.json` 定义的 `job_brief` 结构
3. **可扩展架构** - 支持快速添加新网站的ETL脚本
4. **生产级代码** - 遵循Google Python Style Guide,可维护、可扩展

---

## 📁 项目结构

```
job-hunter-v2/
├── schema.json              # 数据结构定义
├── example.py                   # 参考示例代码
├── etl_template.py         # ETL脚本模板
├── etl_didi.py             # 滴滴招聘ETL脚本
├── etl_tencent.py          # 腾讯招聘ETL脚本
├── etl_xiaomi.py           # 小米招聘ETL脚本
├── etl_nio.py              # 蔚来招聘ETL脚本
├── batch_etl.py            # 批量处理脚本
├── test_etl.py             # 测试脚本
├── ds-didi/                # 滴滴HTML数据目录
│   ├── p-1.html
│   └── ...
├── ds-tencent/             # 腾讯HTML数据目录
│   ├── p-1.html
│   └── ...
├── ds-xiaomi/              # 小米HTML数据目录
│   ├── p-1.html
│   └── ...
├── ds-nio/                 # 蔚来HTML数据目录
│   ├── p-1.html
│   └── ...
└── output/                 # 输出目录
    ├── didi_jobs.json
    ├── tencent_jobs.json
    ├── xiaomi_jobs.json
    ├── nio_jobs.json
    └── all_jobs.json
```

---

## 📊 数据结构规范

### Schema位置
`schema.json` - 包含完整的 `job_brief` 数据结构定义

### 必需字段 (5个)
```json
{
  "id": "string",              // 职位唯一标识符
  "site_id": "string",         // 站点标识符 (如: "didi", "tencent")
  "title": "string",           // 职位标题
  "url": "string",             // 职位详情页URL
  "posted_time": "string"      // 发布时间 (ISO 8601格式: YYYY-MM-DDTHH:MM:SS)
}
```

### 可选字段 (9个)
```json
{
  "company_name": "string",    // 公司名称
  "city": "string",            // 工作城市
  "location_text": "string",   // 完整位置文本 (城市、地区、国家)
  "job_category": "string",    // 职位类别 (如: Engineering, Design, Product)
  "job_level": "string",       // 职位级别 (如: Junior, Senior, Lead, Manager)
  "employment_type": "string", // 雇佣类型 (full-time, part-time, intern, contract)
  "work_mode": "string",       // 工作模式 (onsite, remote, hybrid)
  "salary_text": "string",     // 薪资范围文本
  "tags": ["string"],          // 职位标签数组
  "summary": "string",         // 职位摘要或描述预览
  "discover_time": "string"    // 发现时间 (ISO 8601格式)
}
```

---

## 🛠️ 技术要求

### 代码规范
- **Python版本**: ≥3.10 (优先3.12+)
- **风格指南**: Google Python Style Guide
- **类型注解**: 全量类型注解,复杂类型用双引号包裹
- **字符串**: 统一使用双引号
- **导入**: `from pathlib import Path` (不要 `import pathlib`)
- **文档字符串**: Google或NumPy风格,简洁完整

### 依赖库
```python
from bs4 import BeautifulSoup    # HTML解析
from datetime import datetime     # 时间处理， 或者 `import datetime as dt`
from pathlib import Path          # 路径操作
from typing import Any            # 类型注解
import json                       # JSON处理
from cells.date.core import DateTimeParser  # 处理时间解析，不支持 「时间戳」，「相对时间」
```

### 异常处理
- 显式捕获异常,分类清晰
- 单个职位解析失败不影响整体
- 提供有意义的错误信息

---

## 📝 参考文件说明

### 1. example.py - 参考示例
```python
from bs4 import BeautifulSoup

with open("p-8.html", "r", encoding="utf-8") as f:
    c = f.read()
    soup = BeautifulSoup(c, "lxml")
    tags = soup.find_all(href=True)
    print([tag['href'] for tag in tags])
```

**关键要点**:
- 使用 `BeautifulSoup` 解析HTML
- 使用 `lxml` 解析器
- 使用 `find_all()` 查找元素
- 通过标签属性提取数据

### 2. etl_template.py - ETL模板
包含完整的ETL脚本结构:
- `parse_job_brief()` - 解析HTML,提取职位信息
- `save_to_json()` - 保存为JSON文件
- `etl()` - 主流程函数

---

## 🎯 ETL脚本编写规范

### 核心函数结构

#### 1. parse_job_brief() - 解析函数
```python
def parse_job_brief(html_content: str, site_id: str) -> list[dict[str, Any]]:
    """
    解析HTML内容,提取职位信息
    
    Args:
        html_content: HTML字符串
        site_id: 站点标识符
    
    Returns:
        职位列表,每个职位符合job_brief结构
    """
    soup = BeautifulSoup(html_content, "lxml")
    jobs = []
    
    # 根据具体网站的HTML结构实现解析逻辑
    # 1. 找到所有职位卡片/行
    # 2. 遍历每个职位元素
    # 3. 提取所有可用字段
    # 4. 构建job_brief字典
    # 5. 添加到jobs列表
    
    return jobs
```

#### 2. save_to_json() - 保存函数
```python
def save_to_json(jobs: list[dict[str, Any]], output_path: str) -> None:
    """
    将职位数据保存为JSON文件
    
    Args:
        jobs: 职位列表
        output_path: 输出文件路径
    """
    output_file = Path(output_path)
    output_file.parent.mkdir(parents=True, exist_ok=True)
    
    with open(output_file, "w", encoding="utf-8") as f:
        json.dump(jobs, f, ensure_ascii=False, indent=2)
```

#### 3. etl() - 主流程函数
```python
def etl(html_file: str, output_json: str, site_id: str) -> None:
    """
    ETL主流程:读取HTML -> 解析 -> 导出JSON
    
    Args:
        html_file: 输入HTML文件路径
        output_json: 输出JSON文件路径
        site_id: 站点标识符
    """
    with open(html_file, "r", encoding="utf-8") as f:
        html_content = f.read()
    
    jobs = parse_job_brief(html_content, site_id)
    save_to_json(jobs, output_json)
    
    print(f"已解析 {len(jobs)} 个职位,保存至 {output_json}")
```

### 数据提取策略

#### 常见HTML结构类型

**类型1: 卡片式布局 (小米、滴滴)**
```html
<div class="job-card">
  <a href="/job/12345">职位标题</a>
  <span class="city">北京</span>
  <span class="category">研发</span>
  <span class="time">2024-01-15</span>
</div>
```

**类型2: 表格布局 (蔚来)**
```html
<table>
  <tbody>
    <tr>
      <td href="http://nio.jobs.feishu.cn/...">职位标题</td>
      <td>职能分类</td>
      <td>工作地点</td>
      <td>发布时间</td>
    </tr>
  </tbody>
</table>
```

**类型3: 列表布局 (腾讯)**
```html
<ul class="job-list">
  <li>
    <a href="/position/12345">职位标题</a>
    <div class="meta">
      <span>北京</span>
      <span>技术</span>
      <span>2024-01-15</span>
    </div>
  </li>
</ul>
```

#### 字段提取技巧

**ID提取**:
- 从URL中提取: `url.split("/")[-1]`
- 从data属性提取: `card.get("data-id")`

**时间处理**:
```python
# 相对时间转换
def parse_relative_time(text: str) -> str:
    """转换相对时间为ISO格式"""
    if "天前" in text:
        days = int(text.replace("天前", ""))
        return (datetime.now() - timedelta(days=days)).isoformat()
    # 其他时间格式...

# 绝对时间转换
def parse_absolute_time(text: str) -> str:
    """转换绝对时间为ISO格式"""
    return datetime.strptime(text, "%Y-%m-%d").isoformat()

# （可选）增强版时间解析器
def parse_time_as_date(text: str) -> str:
    date_parser = DateTimeParser(time_str, dt_format=None)
    date_obj: "dt.datetime" = date_parser.format("%Y-%m-%d").isoformat()
```

**职位级别推断**:
```python
def infer_job_level(title: str) -> str:
    """根据职位标题推断级别"""
    if any(kw in title for kw in ["实习", "intern"]):
        return "intern"
    elif any(kw in title for kw in ["专家", "架构", "lead"]):
        return "expert"
    elif any(kw in title for kw in ["资深", "高级", "senior"]):
        return "senior"
    # 其他级别...
    return ""
```

---

## 🚀 执行步骤

### 步骤1: 分析HTML结构
```bash
# 使用调试脚本查看HTML结构
python debug_nio.py
```

**关键点**:
- 找到职位列表容器
- 确定每个职位的HTML结构
- 识别关键字段的位置
- 处理多页面结构

### 步骤2: 创建ETL脚本
```bash
# 复制模板
cp etl_template.py etl_<site>.py

# 修改脚本
# 1. 更新文件头部注释
# 2. 实现parse_job_brief()函数
# 3. 添加辅助函数(时间解析、级别推断等)
# 4. 更新main函数调用
```

### 步骤3: 测试ETL脚本
```bash
# 运行单个网站
python etl_<site>.py

# 检查输出
cat output/<site>_jobs.json | head -20
```

### 步骤4: 集成到批量处理
```python
# 在batch_etl.py中添加配置
SITE_CONFIG = {
    "<site>": {
        "name": "公司名称",
        "html_dir": "ds-<site>",
        "output_file": "output/<site>_jobs.json",
        "site_id": "<site>",
        "etl_module": "etl_<site>",
        "etl_function": "etl",
        "max_pages": 5,
    },
    # ...
}
```

### 步骤5: 批量运行
```bash
# 运行所有网站
python batch_etl.py

# 运行指定网站
python batch_etl.py --sites didi tencent
```

---

## ✅ 验证标准

### 代码质量
- [ ] 遵循Google Python Style Guide
- [ ] 完整的类型注解
- [ ] 清晰的文档字符串
- [ ] 完善的异常处理
- [ ] 可读的变量命名

### 数据质量
- [ ] 所有必需字段都存在
- [ ] 时间格式符合ISO 8601标准
- [ ] 枚举值在允许范围内
- [ ] 无重复的职位ID
- [ ] URL格式正确

### 功能完整性
- [ ] 能够解析HTML文件
- [ ] 能够提取所有可用字段
- [ ] 能够输出JSON文件
- [ ] 能够处理多页面
- [ ] 能够集成到批量处理

### 输出格式验证
```bash
# 验证JSON格式
python -m json.tool output/<site>_jobs.json > /dev/null

# 统计职位数量
python -c "import json; data=json.load(open('output/<site>_jobs.json')); print(f'Total jobs: {len(data)}')"

# 检查必需字段
python -c "
import json
data = json.load(open('output/<site>_jobs.json'))
required = ['id', 'site_id', 'title', 'url', 'posted_time']
for job in data:
    missing = [f for f in required if f not in job]
    if missing:
        print(f'Job {job.get(\"id\")} missing: {missing}')
"
```

---

## 📚 示例网站

### 已完成的ETL脚本

#### 1. 滴滴 (etl_didi.py)
- **HTML结构**: 卡片式布局
- **关键元素**: `div.job-card`
- **提取字段**: 部门、职位类型、城市、发布时间
- **特殊处理**: 职位级别推断、时间解析

#### 2. 腾讯 (etl_tencent.py)
- **HTML结构**: 列表式布局
- **关键元素**: `ul.job-list > li`
- **提取字段**: 事业群、职位类别、工作经验
- **特殊处理**: 职位级别推断、多事业群处理

#### 3. 小米 (etl_xiaomi.py)
- **HTML结构**: 卡片式布局
- **关键元素**: `div.job-item`
- **提取字段**: 城市、招聘类型、雇佣类型
- **特殊处理**: 多页面处理、类型标准化

#### 4. 蔚来 (etl_nio.py)
- **HTML结构**: 表格式布局
- **关键元素**: `table tbody tr`
- **提取字段**: 职能分类、职能子分类、工作地点
- **特殊处理**: 表格解析、ID提取

---

## 🔧 扩展新网站

### 快速添加新网站

#### 1. 准备HTML数据
```bash
# 创建数据目录
mkdir ds-<site>

# 下载HTML文件
# 将招聘网站的HTML页面保存到 ds-<site>/p-1.html, p-2.html, ...
```

#### 2. 分析HTML结构
```python
# 创建临时分析脚本
from bs4 import BeautifulSoup

with open("ds-<site>/p-1.html", "r", encoding="utf-8") as f:
    soup = BeautifulSoup(f.read(), "lxml")

# 查找职位元素
job_elements = soup.find_all("div", class_="job-card")
# 或其他选择器...

# 分析第一个职位的结构
first_job = job_elements[0]
print(first_job.prettify())
```

#### 3. 创建ETL脚本
```bash
# 复制模板
cp etl_template.py etl_<site>.py

# 编辑脚本
# 1. 更新文件头部
# 2. 实现parse_job_brief()
# 3. 添加辅助函数
# 4. 更新main函数
```

#### 4. 测试
```bash
# 运行测试
python etl_<site>.py

# 验证输出
cat output/<site>_jobs.json | jq '.[0]'
```

#### 5. 集成
```python
# 在batch_etl.py中添加配置
SITE_CONFIG = {
    "<site>": {
        "name": "公司名称",
        "html_dir": "ds-<site>",
        "output_file": "output/<site>_jobs.json",
        "site_id": "<site>",
        "etl_module": "etl_<site>",
        "etl_function": "etl",
        "max_pages": 5,
    },
}
```

---

## 📖 文档支持

### 主要文档
- `README_ETL.md` - 完整使用文档
- `QUICKSTART.md` - 快速开始指南
- `ETL_SUMMARY.md` - 项目总结
- `PROMPT.md` - 本文档(任务执行指南)

### 脚本文档
- 每个ETL脚本都包含详细的文档字符串
- 函数级别的注释说明
- 使用示例

---

## 🎯 成功标准

### 功能目标
- ✅ 至少完成3个网站的ETL脚本
- ✅ 所有脚本输出符合schema.json
- ✅ 支持批量处理
- ✅ 代码可维护、可扩展

### 质量目标
- ✅ 代码覆盖率 > 80%
- ✅ 所有必需字段100%填充
- ✅ 时间格式100%正确
- ✅ 无重复职位ID

### 性能目标
- ✅ 单个页面解析时间 < 1秒
- ✅ 批量处理10个页面 < 30秒
- ✅ 内存占用 < 500MB

---

## 📞 常见问题

### Q1: 如何处理缺失字段?
A: 根据schema.json,某些字段是可选的。如果无法提取,可以:
- 设置为空字符串 `""`
- 设置为空数组 `[]`
- 根据上下文推断合理的默认值

### Q2: 如何处理时间格式?
A: 使用ISO 8601格式: `YYYY-MM-DDTHH:MM:SS`
- 相对时间: 计算绝对时间
- 绝对时间: 转换为ISO格式
- 未知时间: 使用当前时间或留空

### Q3: 如何处理多页面?
A: 
- 文件命名: `p-1.html`, `p-2.html`, ...
- 批量处理: 在ETL脚本中循环处理多个文件
- 或使用batch_etl.py统一处理

### Q4: 如何处理异常?
A: 使用try-except捕获异常,记录错误信息,继续处理其他职位:
```python
try:
    job = extract_job_info(element)
    jobs.append(job)
except Exception as e:
    print(f"解析职位失败: {e}")
    continue
```

---

## 🎓 最佳实践

### 1. 代码组织
- 函数单一职责
- 避免过度嵌套
- 使用类型注解
- 添加文档字符串

### 2. 数据处理
- 保留中间变量
- 避免过度合并
- 提供默认值
- 验证数据格式

### 3. 错误处理
- 分类捕获异常
- 提供有意义的错误信息
- 记录失败原因
- 不影响整体流程

### 4. 性能优化
- 使用高效的CSS选择器
- 避免重复解析
- 批量处理数据
- 控制内存使用

---

## 📋 任务清单

### 阶段1: 基础设施
- [x] 创建ETL模板
- [x] 定义数据结构
- [x] 建立项目结构
- [x] 编写批量处理脚本

### 阶段2: 网站ETL
- [x] 滴滴ETL脚本
- [x] 腾讯ETL脚本
- [x] 小米ETL脚本
- [x] 蔚来ETL脚本
- [ ] 大疆ETL脚本
- [ ] ViperETL脚本

### 阶段3: 测试验证
- [x] 单元测试
- [x] 集成测试
- [ ] 性能测试
- [ ] 数据质量验证

### 阶段4: 文档完善
- [x] 使用文档
- [x] 快速开始指南
- [x] 任务执行指南
- [ ] API文档

---

## 🎉 总结

本PROMPT.md提供了完整的ETL系统构建指南,包括:

1. **项目背景和目标** - 明确任务范围
2. **数据结构规范** - 统一输出格式
3. **技术要求** - 代码质量标准
4. **参考文件说明** - 学习资源
5. **ETL脚本编写规范** - 实施细节
6. **执行步骤** - 具体操作流程
7. **验证标准** - 质量检查
8. **扩展指南** - 添加新网站
9. **最佳实践** - 经验总结

遵循本指南,可以快速构建一个生产级的职位数据ETL系统!

---

**版本**: 1.0.0  
**更新时间**: 2025-01-15  
**维护者**: Job Hunter Team
