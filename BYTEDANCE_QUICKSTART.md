# 字节跳动 ETL 快速开始

## 一分钟上手

### 1. 运行字节跳动 ETL

```bash
python etl_bytedance.py
```

输出将保存到 `output/bytedance_jobs.json`

### 2. 验证结果

```bash
python verify_bytedance.py
```

查看解析统计和职位详情。

### 3. 集成到批量处理

```bash
python batch_etl.py
```

或只处理字节跳动:

```python
from batch_etl import batch_process

batch_process(sites=["bytedance"])
```

## 文件说明

| 文件 | 说明 |
|-----|------|
| `etl_bytedance.py` | 主 ETL 脚本 |
| `test_bytedance_etl.py` | 测试脚本 |
| `verify_bytedance.py` | 验证脚本 |
| `ds-bytedance/*.json` | 数据源文件 |
| `output/bytedance_jobs.json` | 输出文件 |

## 数据流程

```
ds-bytedance/p-*.json (API 响应)
    ↓
etl_bytedance.py (解析 JSON)
    ↓
output/bytedance_jobs.json (标准格式)
    ↓
batch_etl.py (可选: 合并所有网站)
    ↓
output/all_jobs.json (合并结果)
```

## 关键特性

✅ 支持 JSON API 响应格式  
✅ 自动时间戳转换 (毫秒 → ISO 8601)  
✅ 智能职位级别推断  
✅ 雇佣类型标准化  
✅ 与其他网站无缝集成  

## 常见问题

**Q: 为什么字节跳动使用 JSON 而不是 HTML?**  
A: 字节跳动的数据源来自 API 响应,不是爬取的 HTML 页面。

**Q: 如何添加新的 JSON 文件?**  
A: 将 JSON 文件放入 `ds-bytedance/` 目录,文件名任意(如 `p-2.json`)。

**Q: 输出格式与其他网站一致吗?**  
A: 是的,完全符合 `schema.json` 定义的 `job_brief` 结构。

## 获取帮助

详细文档: `BYTEDANCE_ETL_README.md`  
项目文档: `PROMPT.md`  
