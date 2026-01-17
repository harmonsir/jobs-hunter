# 蔚来(NIO) ETL 脚本更新说明

## 更新时间
2025-01-15

## 更新内容

### 1. 新增文件

#### etl_nio.py - 蔚来招聘ETL脚本
**文件路径**: `job-hunter-v2/etl_nio.py`

**主要功能**:
- 解析蔚来招聘网站的HTML页面
- 提取职位信息并转换为符合schema.json的JSON格式
- 支持多种时间格式的解析
- 自动推断职位级别

**核心函数**:

##### parse_posted_time(time_str: str) -> str
解析发布时间字符串为ISO格式,支持:
- 绝对时间: `2026-01-15`
- 相对时间: `3天前`, `5小时前`, `30分钟前`

##### parse_tags(tags_text: str) -> dict[str, Any]
解析标签文本,提取城市、薪资、经验等信息:
- 输入格式: `上海 · 30-50K · 3-5年 · 本科`
- 输出包含: city, salary_text, experience, education, tags

##### infer_job_level(experience: str) -> str
根据工作经验推断职位级别:
- 实习 → intern
- 应届/0-1年 → junior
- 3-5年 → middle
- 5-10年 → senior
- 10年以上 → expert

##### parse_job_brief(html_content: str, site_id: str) -> list[dict[str, Any]]
主解析函数,提取所有职位信息,返回符合job_brief结构的列表

**提取的字段**:
- ✅ id: 职位ID (从URL提取)
- ✅ site_id: 站点标识符
- ✅ title: 职位标题
- ✅ url: 职位详情页URL
- ✅ company_name: 公司名称
- ✅ city: 工作城市
- ✅ location_text: 位置文本
- ✅ job_category: 职位类别
- ✅ job_level: 职位级别 (自动推断)
- ✅ employment_type: 雇佣类型 (默认full-time)
- ✅ work_mode: 工作模式 (默认onsite)
- ✅ salary_text: 薪资范围
- ✅ posted_time: 发布时间 (ISO格式)
- ✅ tags: 职位标签数组
- ✅ summary: 职位摘要
- ✅ discover_time: 发现时间

### 2. 更新文件

#### batch_etl.py - 批量处理脚本
**更新内容**:
- 添加了蔚来网站的导入: `from etl_nio import parse_job_brief as parse_nio`
- 在SITE_CONFIGS中添加了蔚来配置:
  ```python
  "nio": {
      "parser": parse_nio,
      "base_dir": "ds-nio",
      "output_file": "jobs.json",
      "site_id": "nio",
  },
  ```

#### README_ETL.md - 文档更新
**更新内容**:
- 文件结构中添加了 `etl_nio.py` 和 `ds-nio/` 目录
- 使用方法中添加了蔚来招聘的ETL说明
- 输出格式示例中更新了网站数量 (4个网站)
- 新增"各网站特性说明"章节,详细说明了蔚来的特性:
  - 职位ID提取方式
  - 时间格式支持
  - 标签格式
  - 职位级别推断规则
- 常见问题中添加了蔚来相对时间处理的说明
- 扩展建议中添加了智能推断的说明

## 使用方法

### 单独运行蔚来ETL
```bash
cd job-hunter-v2
python etl_nio.py
```

### 批量处理包含蔚来
```bash
cd job-hunter-v2
python batch_etl.py
```

### 只处理蔚来
```python
from batch_etl import batch_process

batch_process(
    sites=["nio"],
    output_dir="output",
    merge_all=True
)
```

## 数据源准备

1. 在 `job-hunter-v2/` 目录下创建 `ds-nio/` 目录
2. 将蔚来的HTML招聘页面保存到该目录
3. 命名格式: `p-1.html`, `p-2.html`, ...
4. 运行ETL脚本

## 输出示例

### 单个职位JSON
```json
{
  "id": "12345",
  "site_id": "nio",
  "title": "高级算法工程师",
  "url": "https://nio.jobs/jobs/12345",
  "company_name": "蔚来",
  "city": "上海",
  "location_text": "上海",
  "job_category": "技术研发",
  "job_level": "senior",
  "employment_type": "full-time",
  "work_mode": "onsite",
  "salary_text": "30-50K",
  "posted_time": "2025-01-12T10:30:00",
  "tags": ["上海", "30-50K", "3-5年", "本科"],
  "summary": "负责自动驾驶算法研发...",
  "discover_time": "2025-01-15T14:30:00"
}
```

## 技术亮点

1. **智能时间解析**: 支持绝对时间和多种相对时间格式
2. **职位级别推断**: 根据工作经验自动推断职位级别
3. **标签解析**: 从 `城市 · 薪资 · 经验 · 学历` 格式中提取多维信息
4. **异常处理**: 单个职位解析失败不影响整体处理
5. **完整字段覆盖**: 满足schema.json定义的所有字段

## 测试建议

1. 测试不同时间格式的解析
2. 验证职位级别推断的准确性
3. 检查标签提取的完整性
4. 确认URL构建的正确性
5. 验证输出JSON符合schema

## 后续优化建议

1. 添加更多职位级别的推断规则
2. 支持更多的雇佣类型识别
3. 添加工作模式的识别 (remote/hybrid)
4. 优化薪资范围的标准化
5. 添加职位描述的详细提取

## 兼容性说明

- Python版本: 3.10+
- 依赖库: beautifulsoup4, lxml
- 编码: UTF-8
- 跨平台: Windows/Linux/macOS

## 联系与反馈

如有问题或建议,请通过项目Issue反馈。
