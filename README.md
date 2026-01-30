# Job Hunter ETL V3

## 概述

本目录包含 Job Hunter 招聘数据 ETL 脚本的 V3 版本，完全符合 Job Hunter V3 规范。

## 主要改进

### 1. 符合 V3 规范
- ✅ 添加 `uid` 字段，使用 `site_id + "-" + id` 格式
- ✅ 使用 `uuid5` 生成缺失 ID 的唯一标识
- ✅ 所有必需字段完整：`uid`, `id`, `site_id`, `title`, `url`, `posted_time`
- ✅ 使用 ISO 8601 格式的时间字符串（YYYY-MM-DDTHH:MM:SS）

### 2. 改进的标准化函数
- ✅ `infer_job_level()`: 根据标题和工作年限推断职级
- ✅ `infer_employment_type()`: 根据经验和标题推断雇佣类型
- ✅ `build_job_url()`: 正确处理完整 URL 和相对路径（http 开头不拼接）
- ✅ `build_uid()`: 生成标准化的 UID

### 3. 增强的数据提取
- ✅ 更稳定的 HTML 选择器（使用正则匹配而非随机 class）
- ✅ 更准确的字段提取逻辑
- ✅ 改进的错误处理和容错能力

### 4. 双重去重机制
- ✅ 先检查 URL 重复
- ✅ 再检查 UID 重复
- ✅ 确保数据唯一性

## 文件结构

```
etl-V3/
├── etl_alibaba.py          # 阿里巴巴 ETL 脚本
├── etl_aliyun.py            # 阿里云 ETL 脚本
├── etl_tencent.py           # 腾讯 ETL 脚本
├── etl_byd.py              # 比亚迪 ETL 脚本
├── etl_bytedance.py         # 字节跳动 ETL 脚本
├── etl_dji.py              # 大疆 ETL 脚本
├── etl_didi.py             # 滴滴 ETL 脚本
├── etl_game163.py           # 网易游戏 ETL 脚本
├── etl_hsbc.py             # HSBC ETL 脚本
├── etl_jd.py               # 京东 ETL 脚本
├── etl_meituan.py           # 美团 ETL 脚本
├── etl_minimax.py           # MiniMax ETL 脚本
├── etl_nio.py              # 蔚来 ETL 脚本
├── etl_viper.py            # 唯品会 ETL 脚本
├── etl_weride.py           # 文远知行 ETL 脚本
├── etl_xiaomi.py           # 小米 ETL 脚本
├── etl_zhipu.py           # 智谱AI ETL 脚本
├── etl_sc_expert.py       # Standard Chartered Bank ETL 脚本
└── README.md                # 本文档
```

## 使用方法

### 1. 解析单个文件

```python
from etl_viper import etl

jobs = etl(
    input_file="../ds-viper/p-1.html",
    output_json="../output/viper_jobs_v3.json",
    site_id="viper"
)
```

### 2. 批量解析并写入数据库

```python
from etl_viper import example_batch
example_batch()
```

[各站点字段映射.md](%E5%90%84%E7%AB%99%E7%82%B9%E5%AD%97%E6%AE%B5%E6%98%A0%E5%B0%84.md)

## 职级推断规则

| 职级 | 标题关键词 | 工作年限 |
|-----|-----------|---------|
| intern | 实习, intern | < 1年 |
| junior | 初级, 助理, junior | 1-3年 |
| mid | （默认） | 3-5年 |
| senior | 资深, 高级, senior, 主任 | 5-10年 |
| expert | 合伙人, partner | ≥ 10年 |
| lead | 负责人, 主管, lead | ≥ 5年 |
| manager | 经理, manager | - |
| director | 总监, director | - |
| chief | 首席, cto, chief | - |

## 雇佣类型映射

| V3 值 | 匹配关键词 |
|-------|-----------|
| full-time | 全职, 社招, 校招, 正式 |
| part-time | 兼职, part-time |
| intern | 实习, intern |
| contract | 外包, contract |

## URL 处理规则

- 如果原始 URL 以 `http` 或 `https` 开头，则直接使用，不进行拼接
- 如果是相对路径，则拼接到 BASE_URL

```python
def build_job_url(raw_url: str, base_url: str = "") -> str:
    if not raw_url:
        return ""
    if raw_url.startswith("http"):
        return raw_url
    return f"{base_url}{raw_url}"
```

## UID 生成规则

UID 格式：`{site_id}-{id}`

```python
def build_uid(site_id: str, raw_id: str | None, url: str) -> Tuple[str, str]:
    normalized_id = str(raw_id).strip() if raw_id else uuid.uuid5(uuid.NAMESPACE_URL, url).hex
    uid = f"{site_id}{UID_SEPARATOR}{normalized_id}"
    return normalized_id, uid
```

## DOM 选择策略（V3 新增）

遵循 V3 规范的 HTML 解析策略：

### 优先使用属性选择器
- ✅ `a[href*="job-detail.html?id="]` - 使用 href 属性匹配
- ✅ `input[id="requemtId"]` - 使用 id 属性匹配
- ✅ `div[class*="job"]` - 使用 class 模糊匹配

### 避免随机 class 名称
- ❌ `div.container-aOp138` - 包含随机字符
- ❌ `div._1x2y3z` - 无意义的 hash 值

### 使用稳定的语义化选择器
- ✅ `div.positionItem` - 有意义的单词
- ✅ `span.post` - 职位相关的 class
- ✅ `span.sel` - 选择器相关的 class

### 使用标签组合与属性匹配
- ✅ `li[data-type="job"]` - 使用 data 属性
- ✅ `div[class*="job"]` - 使用 class 模糊匹配

### 使用正则表达式匹配
- ✅ `re.compile(r"container")` - 匹配包含 "container" 的 class
- ✅ `re.compile(r"title")` - 匹配包含 "title" 的 class
- ✅ `re.compile(r"link")` - 匹配包含 "link" 的 class



## 注意事项

1. **数据库依赖**: `example_batch()` 需要 `jobs.engine` 模块，如果不可用会自动跳过
2. **时区处理**: 所有时间都使用本地时间，格式为 ISO 8601（YYYY-MM-DDTHH:MM:SS）
3. **ID 生成**: 如果原始数据缺失 ID，会使用 `uuid5(NAMESPACE_URL, url)` 生成
4. **去重逻辑**: 在插入数据库前会先检查 URL 和 UID 的重复性
5. **HTML 解析**: 对于使用随机 class 的站点（如唯品会、文远知行），使用正则表达式模糊匹配

## 从 V2 迁移到 V3 的主要变更

1. **添加 UID 字段**: 使用 `build_uid()` 函数生成
2. **改进职级推断**: 使用新的 `infer_job_level()` 函数
3. **添加雇佣类型**: 使用 `infer_employment_type()` 函数
4. **时间格式化**: 使用 ISO 8601 格式（YYYY-MM-DDTHH:MM:SS）
5. **双重去重**: 先检查 URL，再检查 UID
6. **URL 处理**: http 开头的完整 URL 不再拼接
7. **DOM 选择**: 使用更稳定的 HTML 选择器（正则表达式模糊匹配）
8. **ID 生成**: 使用 `uuid5` 替代 `hashlib.md5`
9. **JSON 数据支持**: 支持小米、智谱AI等使用 JSON API 的站点

## 支持的站点

- ✅ 阿里巴巴（JSON）
- ✅ 阿里云（JSON）
- ✅ 腾讯（HTML）
- ✅ 比亚迪（HTML）
- ✅ 字节跳动（JSON）
- ✅ 大疆（HTML）
- ✅ 滴滴（HTML）
- ✅ 网易游戏（HTML）
- ✅ HSBC（JSON）
- ✅ 京东（HTML）
- ✅ 美团（JSON）
- ✅ MiniMax（HTML）
- ✅ 蔚来（HTML）
- ✅ 唯品会（HTML）
- ✅ 文远知行（HTML）
- ✅ 小米（JSON）
- ✅ 智谱AI（JSON）
- ✅ Standard Chartered Bank（HTML）

## 问题反馈

如有问题，请检查：
1. 数据文件路径是否正确
2. 数据库连接是否可用（如需写入数据库）
3. JSON/HTML 解析库是否已安装
