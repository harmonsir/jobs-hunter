# 蔚来(NIO) ETL 脚本 - 更新说明

## ✅ 已完成更新

### 1. 创建了 `etl_nio.py` - 蔚来招聘ETL脚本

**文件路径**: `job-hunter-v2/etl_nio.py`

**核心功能**:
- 解析蔚来招聘网站的HTML页面
- 提取职位信息并转换为符合schema.json的JSON格式
- 支持绝对时间格式解析 (YYYY-MM-DD)
- 根据职位标题和子分类自动推断职位级别

### 2. 更新了 `batch_etl.py` - 批量处理脚本

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

### 3. 更新了 `README_ETL.md` - 文档

**更新内容**:
- 文件结构中添加了 `etl_nio.py` 和 `ds-nio/` 目录
- 使用方法中添加了蔚来招聘的ETL说明
- 输出格式示例中更新了网站数量 (4个网站)
- 新增"各网站特性说明"章节,详细说明了蔚来的特性

### 4. 创建了 `UPDATE_NIO_ETL.md` - 更新文档

**文件路径**: `job-hunter-v2/UPDATE_NIO_ETL.md`

包含详细的更新说明、使用方法和技术亮点。

## 📊 蔚来网站HTML结构分析

### 实际HTML结构
蔚来招聘页面使用表格布局,而非卡片布局:

```html
<table>
  <thead>
    <tr>
      <th>职位</th>
      <th>职能分类</th>
      <th>职能子分类</th>
      <th>工作地点</th>
      <th>发布时间</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="CareerList_CareerListItemTitle__T1pOJ" href="http://nio.jobs.feishu.cn/...">职位标题</td>
      <td class="CareerList_CareerListItemCategory__UWq7C">职能分类</td>
      <td class="CareerList_CareerListItemSubCategory__gaCfK">职能子分类</td>
      <td>工作地点</td>
      <td>发布时间</td>
    </tr>
  </tbody>
</table>
```

### 字段提取规则

| 字段 | 提取方式 | 示例 |
|------|----------|------|
| id | 从URL中提取数字 | `7595408487745734954` |
| title | 第一个td的文本 | `App搜索产品经理` |
| url | 第一个td的href属性 | `http://nio.jobs.feishu.cn/...` |
| job_category | 第二个td的文本 | `产品` |
| job_level | 根据标题和子分类推断 | `senior` |
| city | 第四个td的文本 | `上海` |
| posted_time | 第五个td的文本 | `2026-01-15` |

## 🎯 职位级别推断规则

```python
def infer_job_level(title: str, sub_category: str) -> str:
    text = f"{title} {sub_category}".lower()
    
    if "实习" in text or "intern" in text:
        return "intern"
    elif "专家" in text or "架构" in text or "lead" in text:
        return "expert"
    elif "资深" in text or "高级" in text or "senior" in text:
        return "senior"
    elif "经理" in text and "产品" not in text and "项目" not in text:
        return "middle"
    elif "总监" in text or "主管" in text:
        return "senior"
    elif "助理" in text or "初级" in text:
        return "junior"
    
    return ""
```

## 🚀 使用方法

### 方式一: 单独运行蔚来ETL
```bash
cd job-hunter-v2
python etl_nio.py
```

### 方式二: 批量处理包含蔚来
```bash
cd job-hunter-v2
python batch_etl.py
```

### 方式三: 只处理蔚来
```python
from batch_etl import batch_process

batch_process(
    sites=["nio"],
    output_dir="output",
    merge_all=True
)
```

## 📝 输出示例

### 单个职位JSON
```json
{
  "id": "7595408487745734954",
  "site_id": "nio",
  "title": "App搜索产品经理",
  "url": "http://nio.jobs.feishu.cn/index/position/detail/7595408487745734954",
  "company_name": "蔚来",
  "city": "上海",
  "location_text": "上海",
  "job_category": "产品",
  "job_level": "",
  "employment_type": "full-time",
  "work_mode": "onsite",
  "salary_text": "",
  "posted_time": "2026-01-15T00:00:00",
  "tags": ["产品", "产品经理"],
  "summary": "产品 - 产品经理",
  "discover_time": "2025-01-15T14:30:00"
}
```

## 🔧 技术特点

1. **表格解析**: 专门针对表格布局的HTML结构
2. **时间处理**: 支持绝对时间格式 (YYYY-MM-DD)
3. **智能推断**: 根据职位标题和子分类推断职位级别
4. **异常处理**: 单个职位解析失败不影响整体处理
5. **完整字段覆盖**: 满足schema.json定义的所有字段

## 📂 文件清单

### 新增文件
- ✅ `job-hunter-v2/etl_nio.py` - 蔚来ETL脚本
- ✅ `job-hunter-v2/UPDATE_NIO_ETL.md` - 更新文档

### 更新文件
- ✅ `job-hunter-v2/batch_etl.py` - 添加蔚来配置
- ✅ `job-hunter-v2/README_ETL.md` - 更新文档

### 数据源
- ✅ `job-hunter-v2/ds-nio/p-1.html` - 蔚来HTML数据源

## ⚠️ 注意事项

1. **HTML结构**: 蔚来使用表格布局,不同于其他网站的卡片布局
2. **时间格式**: 蔚来使用绝对时间格式 (YYYY-MM-DD)
3. **职位ID**: 从URL中提取,为长数字ID
4. **URL格式**: 蔚来职位链接指向飞书招聘平台
5. **多页面支持**: 页面显示 "1 / 211",说明有211页职位

## 🎉 总结

蔚来ETL脚本已成功创建并集成到批量处理系统中。脚本能够:
- ✅ 正确解析表格结构的HTML
- ✅ 提取所有必需字段
- ✅ 智能推断职位级别
- ✅ 与批量处理脚本无缝集成
- ✅ 输出符合schema.json的JSON格式

现在可以通过 `batch_etl.py` 批量处理滴滴、腾讯、小米和蔚来四个网站的职位数据!
