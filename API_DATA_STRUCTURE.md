# API 数据结构文档

本文档详细说明 NowInOpenHarmony Server 所有 API 接口的请求参数和响应数据结构，帮助团队成员理解和使用现有 API，以及开发新的数据源时保持一致性。

---

## 目录

- [新闻 API (News)](#新闻-api-news)
  - [数据模型](#新闻数据模型)
  - [接口列表](#新闻接口列表)
- [轮播图 API (Banner)](#轮播图-api-banner)
  - [数据模型](#轮播图数据模型)
  - [接口列表](#轮播图接口列表)
- [通用规范](#通用规范)
- [开发新数据源指南](#开发新数据源指南)

---

## 新闻 API (News)

### 新闻数据模型

#### 1. NewsContentBlock - 新闻内容块

新闻正文内容采用 **结构化块** 的方式组织，支持多种内容类型。

```python
class ContentType(str, Enum):
    TEXT = "text"      # 文本段落
    IMAGE = "image"    # 图片
    VIDEO = "video"    # 视频
    CODE = "code"      # 代码块

class NewsContentBlock(BaseModel):
    type: ContentType  # 内容类型
    value: str         # 内容值
```

**示例**:
```json
{
  "type": "text",
  "value": "OpenHarmony 5.0 正式发布..."
}
```

```json
{
  "type": "image",
  "value": "https://www.openharmony.cn/images/banner.jpg"
}
```

```json
{
  "type": "code",
  "value": "import { hilog } from '@kit.PerformanceAnalysisKit';"
}
```

#### 2. NewsArticle - 新闻文章模型

**核心字段说明**:

| 字段名 | 类型 | 必填 | 说明 | 示例 |
|--------|------|------|------|------|
| `id` | string | 否 | 文章唯一标识 | "openharmony_20240115_001" |
| `title` | string | 是 | 文章标题 | "OpenHarmony 5.0 正式发布" |
| `date` | string | 是 | 发布日期 (YYYY-MM-DD 格式) | "2024-01-15" |
| `url` | string | 是 | 文章原文链接 | "https://www.openharmony.cn/news/123" |
| `content` | array | 是 | 文章内容块数组 | 见下方示例 |
| `category` | string | 否 | 文章分类 | "官方动态" / "技术博客" |
| `summary` | string | 否 | 文章摘要/简介 | "本次发布包含..." |
| `source` | string | 否 | 数据源标识 | "OpenHarmony" / "OpenHarmony技术博客" |
| `created_at` | datetime | 否 | 记录创建时间 | "2024-01-15T10:30:00" |
| `updated_at` | datetime | 否 | 记录更新时间 | "2024-01-15T14:20:00" |

**完整示例**:
```json
{
  "id": "openharmony_20240115_001",
  "title": "OpenHarmony 5.0 正式发布",
  "date": "2024-01-15",
  "url": "https://www.openharmony.cn/news/123",
  "content": [
    {
      "type": "text",
      "value": "OpenHarmony 5.0 版本正式发布，带来了全新的特性..."
    },
    {
      "type": "image",
      "value": "https://www.openharmony.cn/images/5.0-banner.jpg"
    },
    {
      "type": "text",
      "value": "主要更新包括..."
    }
  ],
  "category": "官方动态",
  "summary": "OpenHarmony 5.0 带来了性能优化、新API等重要更新",
  "source": "OpenHarmony",
  "created_at": "2024-01-15T10:30:00",
  "updated_at": "2024-01-15T10:30:00"
}
```

#### 3. NewsResponse - 新闻列表响应模型

用于分页返回新闻列表的标准响应格式。

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `articles` | array | 新闻文章数组 | 见 NewsArticle |
| `total` | int | 总记录数 | 156 |
| `page` | int | 当前页码（从1开始） | 1 |
| `page_size` | int | 每页数量 | 20 |
| `has_next` | bool | 是否有下一页 | true |
| `has_prev` | bool | 是否有上一页 | false |

**完整示例**:
```json
{
  "articles": [
    {
      "id": "openharmony_20240115_001",
      "title": "OpenHarmony 5.0 正式发布",
      "date": "2024-01-15",
      "url": "https://www.openharmony.cn/news/123",
      "content": [...],
      "category": "官方动态",
      "summary": "OpenHarmony 5.0 带来了性能优化...",
      "source": "OpenHarmony"
    }
  ],
  "total": 156,
  "page": 1,
  "page_size": 20,
  "has_next": true,
  "has_prev": false
}
```

#### 4. 其他新闻模型

**SearchRequest - 搜索请求模型**:
```python
{
  "keyword": str,        # 搜索关键词
  "category": str,       # 可选：分类过滤
  "start_date": str,     # 可选：开始日期 YYYY-MM-DD
  "end_date": str        # 可选：结束日期 YYYY-MM-DD
}
```

**TopicArticle - 论坛话题模型**:
```python
{
  "id": str,
  "title": str,
  "content": str,
  "author": str,
  "reply_count": int,
  "view_count": int,
  "tags": List[str],
  "created_at": datetime,
  "url": str
}
```

**ReleaseInfo - 版本发布信息模型**:
```python
{
  "id": str,
  "version": str,              # 版本号 "5.0.0"
  "title": str,
  "release_date": str,
  "description": str,
  "features": List[str],       # 新功能列表
  "bug_fixes": List[str],      # 修复列表
  "compatibility": str,        # 兼容性说明
  "download_url": str,
  "created_at": datetime
}
```

---

### 新闻接口列表

#### 1. GET `/api/news/` - 获取新闻列表（聚合所有来源）

**功能**: 获取所有来源的新闻，支持分页、分类过滤、搜索

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `page` | int | 否 | 1 | 页码（≥1） |
| `page_size` | int | 否 | 20 | 每页数量（1-100） |
| `category` | string | 否 | null | 分类过滤（如 "官方动态"） |
| `search` | string | 否 | null | 搜索关键词（标题/摘要） |
| `all` | bool | 否 | false | 是否返回全部数据不分页 |

**响应**: `NewsResponse` 对象

---

### 🏷️ Category（分类）功能详解

#### 📋 分类机制说明

`category` 参数是新闻接口的**核心过滤功能**，用于按新闻类型筛选数据。

**处理流程**：
1. **先过滤**：从缓存中筛选出指定 category 的所有文章
2. **后分页**：对筛选结果进行分页处理
3. **准确统计**：`total` 返回该 category 的文章总数，`has_next`/`has_prev` 基于过滤后的数据判断

**代码实现**（`core/cache.py`）：
```python
# 1. 先按 category 过滤
if category:
    filtered_news = [news for news in all_news if news.category == category]

# 2. 再对过滤结果分页
total = len(filtered_news)  # 过滤后的总数
start = (page - 1) * page_size
end = start + page_size
paginated_news = filtered_news[start:end]
```

#### 📌 当前可用的 Category 值

| Category 值 | 说明 | 数据源 | 爬虫文件 |
|------------|------|--------|---------|
| `"官方动态"` | OpenHarmony 官网新闻公告 | OpenHarmony 官网 | `openharmony_news_crawler.py` |
| `"技术博客"` | OpenHarmony 官方技术博客文章 | OpenHarmony 技术博客 | `openharmony_blog_crawler.py` |

> **⚠️ 重要**：`category` 是字符串类型，**区分大小写**，必须完全匹配上述值。

#### 🎯 使用示例

**获取所有官方动态**：
```bash
# 第1页，每页20条
GET /api/news/?category=官方动态&page=1&page_size=20

# 搜索官方动态中包含"5.0"的新闻
GET /api/news/?category=官方动态&search=5.0
```

**获取所有技术博客**：
```bash
# 第1页，每页20条
GET /api/news/?category=技术博客&page=1&page_size=20

# 获取所有技术博客（不推荐，数据量大）
GET /api/news/?category=技术博客&all=true
```

#### ✅ 优势对比

| 方式 | 接口路径 | 分页准确性 | 推荐度 |
|------|---------|-----------|--------|
| ✅ 使用 category 参数 | `/api/news/?category=官方动态` | ✅ 准确 | **强烈推荐** |
| ❌ 使用专用接口 | `/api/news/openharmony` | ❌ 不准确 | 不推荐 |

**为什么推荐使用 category？**
- ✅ **分页准确**：先过滤后分页，确保每页返回正确数量
- ✅ **统计准确**：`total` 反映该分类的真实数量
- ✅ **判断准确**：`has_next` 基于过滤后的数据计算
- ✅ **性能一致**：与主接口使用相同的缓存机制

---

### 📖 分页机制详解

#### 🔍 分页逻辑说明

1. **数据来源**: 数据从内存缓存中读取，已按日期倒序排列（最新的在前）
2. **分页算法**: 
   ```python
   start = (page - 1) * page_size
   end = start + page_size
   paginated_news = filtered_news[start:end]
   ```
3. **总页数计算**: `总页数 = ceil(total / page_size)`

#### ✅ 最后一页数据不足的处理

**当请求的页面超出数据范围或最后一页数据不足时**：

- ✅ **不会报错**：Python 切片操作会自动处理越界情况
- ✅ **返回实际数据**：返回该页能获取到的所有数据（可能少于 `page_size`）
- ✅ **空数据处理**：如果 `page` 远超最后一页，返回空数组 `articles: []`
- ✅ **`has_next` 准确**：通过 `end < total` 判断，确保准确反映是否有下一页

**示例场景**：

假设总共有 156 条新闻，每页 20 条：
- 第 1-7 页：每页返回 20 条数据
- **第 8 页**（最后一页）：只返回 16 条数据（156 - 7×20 = 16）
  - `has_next: false`
  - `has_prev: true`
  - `page: 8`
  - `page_size: 20`
  - `total: 156`
  - `articles: [16条数据]`
- **第 9+ 页**：返回空数组
  - `has_next: false`
  - `has_prev: true`
  - `page: 9`
  - `page_size: 20`
  - `total: 156`
  - `articles: []`

#### 🚀 推荐的分页请求方式

**方式1：基于 `has_next` 判断（推荐）**
```javascript
let page = 1;
const pageSize = 20;
let allArticles = [];

while (true) {
    const response = await fetch(`/api/news/?page=${page}&page_size=${pageSize}`);
    const data = await response.json();
    
    allArticles.push(...data.articles);
    
    // 关键：通过 has_next 判断是否继续
    if (!data.has_next) {
        break;
    }
    page++;
}

console.log(`共加载 ${allArticles.length} 条新闻`);
```

**方式2：基于总页数计算**
```javascript
const pageSize = 20;
let allArticles = [];

// 第一次请求获取总数
const firstResponse = await fetch(`/api/news/?page=1&page_size=${pageSize}`);
const firstData = await firstResponse.json();
const totalPages = Math.ceil(firstData.total / pageSize);

// 加载所有页
for (let page = 1; page <= totalPages; page++) {
    const response = await fetch(`/api/news/?page=${page}&page_size=${pageSize}`);
    const data = await response.json();
    allArticles.push(...data.articles);
}

console.log(`共加载 ${allArticles.length} 条新闻`);
```

**方式3：并发加载（最快，但请控制并发数）**
```javascript
const pageSize = 20;

// 第一次请求获取总数
const firstResponse = await fetch(`/api/news/?page=1&page_size=${pageSize}`);
const firstData = await firstResponse.json();
const totalPages = Math.ceil(firstData.total / pageSize);

// 并发加载所有页（建议限制并发数为 3-5）
const promises = [];
for (let page = 1; page <= totalPages; page++) {
    promises.push(
        fetch(`/api/news/?page=${page}&page_size=${pageSize}`)
            .then(res => res.json())
    );
}

const results = await Promise.all(promises);
const allArticles = results.flatMap(data => data.articles);

console.log(`共加载 ${allArticles.length} 条新闻`);
```

#### ⚠️ 关于 `all=true` 参数的性能说明

- **问题**: 直接请求 `/api/news/?all=true` 会一次性返回所有数据（可能数百条），导致：
  - 响应时间长（需等待完整数据序列化）
  - 内存占用大（前端一次性渲染大量数据）
  - 用户体验差（长时间白屏等待）

- **实现方式**: 
  ```python
  if all:
      # 内部设置 page_size=10000 获取所有数据
      result = cache.get_news(page=1, page_size=10000, category=category, search=search)
      result.page_size = result.total  # 返回时显示实际总数
  ```

- **建议**: 
  - ✅ 使用分页加载，配合虚拟滚动或无限滚动
  - ✅ 推荐 `page_size=20` 或 `page_size=50`
  - ❌ 避免使用 `all=true`，除非数据量很小（<100条）

---

**示例请求**:
```bash
# 获取第1页，每页20条
curl "http://localhost:8001/api/news/?page=1&page_size=20"

# 搜索包含"鸿蒙"的新闻
curl "http://localhost:8001/api/news/?search=鸿蒙"

# 获取"官方动态"分类，第2页
curl "http://localhost:8001/api/news/?category=官方动态&page=2&page_size=20"

# ⚠️ 不推荐：获取所有新闻（可能很慢）
curl "http://localhost:8001/api/news/?all=true"
```

**示例响应**:
```json
{
  "articles": [
    {
      "id": "openharmony_20240115_001",
      "title": "OpenHarmony 5.0 正式发布",
      "date": "2024-01-15",
      "url": "https://www.openharmony.cn/news/123",
      "content": [
        {"type": "text", "value": "OpenHarmony 5.0 版本正式发布..."}
      ],
      "category": "官方动态",
      "summary": "本次发布包含性能优化...",
      "source": "OpenHarmony",
      "created_at": "2024-01-15T10:30:00"
    }
  ],
  "total": 156,
  "page": 1,
  "page_size": 20,
  "has_next": true,
  "has_prev": false
}
```

---

#### 2. GET `/api/news/openharmony` - 获取官网新闻

**功能**: 仅返回 OpenHarmony 官网来源的新闻

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `page` | int | 否 | 1 | 页码 |
| `page_size` | int | 否 | 20 | 每页数量 |
| `search` | string | 否 | null | 搜索关键词 |

**响应**: `NewsResponse` 对象（只包含 `source="OpenHarmony"` 的文章）

**示例请求**:
```bash
curl "http://localhost:8001/api/news/openharmony?page=1&page_size=10"
```

---

#### 3. GET `/api/news/blog` - 获取技术博客

**功能**: 仅返回 OpenHarmony 技术博客来源的文章

**请求参数**: 同 `/api/news/openharmony`

**响应**: `NewsResponse` 对象（只包含 `source="OpenHarmony技术博客"` 的文章）

**示例请求**:
```bash
curl "http://localhost:8001/api/news/blog?search=ArkTS"
```

---

#### 4. GET `/api/news/{article_id}` - 获取单篇新闻详情

**功能**: 根据文章ID获取完整新闻内容

**路径参数**:

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| `article_id` | string | 是 | 文章唯一标识 |

**响应**: `NewsArticle` 对象

**示例请求**:
```bash
curl "http://localhost:8001/api/news/openharmony_20240115_001"
```

**示例响应**:
```json
{
  "id": "openharmony_20240115_001",
  "title": "OpenHarmony 5.0 正式发布",
  "date": "2024-01-15",
  "url": "https://www.openharmony.cn/news/123",
  "content": [
    {"type": "text", "value": "OpenHarmony 5.0 版本正式发布..."}
  ],
  "category": "官方动态",
  "source": "OpenHarmony"
}
```

**错误响应**:
```json
{
  "detail": "文章不存在"
}
```
HTTP 状态码: 404

---

#### 5. POST `/api/news/crawl` - 手动触发新闻爬取

**功能**: 手动触发指定来源的新闻爬取任务（后台执行）

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `source` | enum | 否 | ALL | 新闻来源（ALL/OPENHARMONY/BLOG） |
| `limit` | int | 否 | 10 | 返回数量限制（1-100） |

**NewsSource 枚举值**:
- `ALL`: 所有来源
- `OPENHARMONY`: 仅官网新闻
- `BLOG`: 仅技术博客

**响应**:
```json
{
  "message": "爬取任务已启动 - 来源: all",
  "source": "all",
  "timestamp": "2024-01-15T10:30:00",
  "note": "爬取任务在后台执行，请稍后查看缓存更新状态"
}
```

**示例请求**:
```bash
# 爬取所有来源
curl -X POST "http://localhost:8001/api/news/crawl?source=ALL"

# 仅爬取官网新闻
curl -X POST "http://localhost:8001/api/news/crawl?source=OPENHARMONY"
```

---

#### 6. GET `/api/news/status/info` - 获取服务状态

**功能**: 获取新闻服务的运行状态和缓存信息

**响应**:
```json
{
  "service_status": {
    "status": "ready",
    "last_update": "2024-01-15T10:30:00",
    "cache_count": 156,
    "error_message": null
  },
  "news_sources": [
    {
      "name": "OpenHarmony官网",
      "identifier": "OpenHarmony",
      "article_count": 89
    },
    {
      "name": "OpenHarmony技术博客",
      "identifier": "OpenHarmony技术博客",
      "article_count": 67
    }
  ],
  "timestamp": "2024-01-15T10:35:00",
  "endpoints": {
    "all_news": "/api/news/",
    "openharmony_news": "/api/news/openharmony",
    "openharmony_blog": "/api/news/blog",
    "news_detail": "/api/news/{article_id}",
    "manual_crawl": "/api/news/crawl",
    "service_status": "/api/news/status/info",
    "cache_refresh": "/api/news/cache/refresh"
  }
}
```

**服务状态说明**:
- `ready`: 服务就绪，缓存数据可用
- `preparing`: 服务准备中，首次加载或更新中
- `error`: 服务错误，需要检查日志

---

#### 7. POST `/api/news/cache/refresh` - 手动刷新缓存

**功能**: 强制刷新新闻缓存

**响应**:
```json
{
  "message": "缓存刷新成功",
  "timestamp": "2024-01-15T10:30:00"
}
```

---

## 轮播图 API (Banner)

### 轮播图数据模型

#### 1. BannerImage - 轮播图图片模型

```python
class BannerImage(BaseModel):
    url: str           # 图片URL路径（必填）
    alt: str           # 图片描述（可选）
```

**示例**:
```json
{
  "url": "https://www.openharmony.cn/mainImg/banner1.png",
  "alt": "OpenHarmony 5.0 发布"
}
```

#### 2. BannerResponse - 轮播图响应模型

**标准响应格式**:

| 字段名 | 类型 | 说明 | 示例 |
|--------|------|------|------|
| `success` | bool | 请求是否成功 | true |
| `images` | array | 图片URL字符串数组 | ["url1", "url2"] |
| `total` | int | 图片总数 | 5 |
| `message` | string | 响应消息 | "获取手机版Banner图片成功" |
| `timestamp` | string | 响应时间戳（ISO格式） | "2024-01-15T10:30:00" |

**完整示例**:
```json
{
  "success": true,
  "images": [
    "https://www.openharmony.cn/mainImg/banner1.png",
    "https://www.openharmony.cn/mainImg/banner2.png",
    "https://www.openharmony.cn/mainImg/banner3.png"
  ],
  "total": 3,
  "message": "获取手机版Banner图片成功（缓存），共 3 张",
  "timestamp": "2024-01-15T10:30:00.123456"
}
```

---

### 轮播图接口列表

#### 1. GET `/api/banner/mobile` - 获取移动端轮播图

**功能**: 获取 OpenHarmony 官网手机版 Banner 图片 URL 列表

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `force_crawl` | bool | 否 | false | 是否强制重新爬取（否则返回缓存） |

**响应**: `BannerResponse` 对象

**爬虫策略**:
1. 默认返回缓存数据（快速响应）
2. `force_crawl=true` 时强制重新爬取
3. 自动回退机制：增强版爬虫失败时自动切换到传统爬虫

**示例请求**:
```bash
# 获取缓存的轮播图
curl "http://localhost:8001/api/banner/mobile"

# 强制重新爬取
curl "http://localhost:8001/api/banner/mobile?force_crawl=true"
```

**示例响应（成功）**:
```json
{
  "success": true,
  "images": [
    "https://www.openharmony.cn/mainImg/banner1.png",
    "https://www.openharmony.cn/mainImg/banner2.png"
  ],
  "total": 2,
  "message": "获取手机版Banner图片成功（缓存），共 2 张",
  "timestamp": "2024-01-15T10:30:00"
}
```

**示例响应（准备中）**:
```json
{
  "success": false,
  "images": [],
  "total": 0,
  "message": "轮播图服务正在准备中，请稍后再试或使用force_crawl=true强制爬取",
  "timestamp": "2024-01-15T10:30:00"
}
```

---

#### 2. GET `/api/banner/mobile/enhanced` - 使用增强版爬虫获取轮播图

**功能**: 使用 Selenium 增强版爬虫获取轮播图（更稳定但速度较慢）

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `force_crawl` | bool | 否 | false | 是否强制重新爬取 |
| `download_images` | bool | 否 | false | 是否下载图片到本地 |

**响应**: `BannerResponse` 对象

**特点**:
- 使用 Selenium WebDriver 模拟真实浏览器
- 可处理 JavaScript 动态加载的内容
- 支持下载图片到服务器本地

**示例请求**:
```bash
# 使用增强版爬虫获取轮播图
curl "http://localhost:8001/api/banner/mobile/enhanced"

# 获取并下载图片到本地
curl "http://localhost:8001/api/banner/mobile/enhanced?download_images=true"
```

---

#### 3. GET `/api/banner/status` - 获取轮播图服务状态

**功能**: 获取轮播图服务的详细状态信息

**响应**:
```json
{
  "service": "banner",
  "cache_status": {
    "status": "ready",
    "last_update": "2024-01-15T10:30:00",
    "cache_count": 5,
    "update_count": 12
  },
  "scheduler_jobs": [
    {
      "id": "banner_crawl_job",
      "name": "轮播图定时爬取",
      "next_run": "2024-01-15T11:00:00"
    }
  ],
  "api_endpoints": {
    "mobile_banners": "/api/banner/mobile",
    "enhanced_banners": "/api/banner/mobile/enhanced",
    "status": "/api/banner/status",
    "manual_crawl": "/api/banner/crawl",
    "clear_cache": "/api/banner/cache/clear",
    "cache_info": "/api/banner/cache"
  },
  "status_explanation": {
    "preparing": "轮播图服务正在准备中，首次爬取尚未完成或当前正在更新",
    "ready": "轮播图服务就绪，可以正常获取轮播图数据",
    "error": "轮播图服务出现错误，需要检查日志或手动重新爬取"
  },
  "timestamp": "2024-01-15T10:35:00"
}
```

---

#### 4. POST `/api/banner/crawl` - 手动触发轮播图爬取

**功能**: 手动触发轮播图爬取任务并立即返回结果

**请求参数**:

| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `use_enhanced` | bool | 否 | true | 是否使用增强版爬虫 |
| `download_images` | bool | 否 | false | 是否下载图片到本地 |

**响应**:
```json
{
  "success": true,
  "message": "手动爬取完成，共获取 5 张图片",
  "images": [
    "https://www.openharmony.cn/mainImg/banner1.png",
    "https://www.openharmony.cn/mainImg/banner2.png"
  ],
  "total": 2,
  "used_enhanced": true,
  "downloaded": false,
  "timestamp": "2024-01-15T10:30:00"
}
```

**示例请求**:
```bash
# 使用增强版爬虫手动爬取
curl -X POST "http://localhost:8001/api/banner/crawl?use_enhanced=true"

# 使用传统爬虫
curl -X POST "http://localhost:8001/api/banner/crawl?use_enhanced=false"
```

---

#### 5. DELETE `/api/banner/cache/clear` - 清空轮播图缓存

**功能**: 清空轮播图缓存数据

**响应**:
```json
{
  "success": true,
  "message": "轮播图缓存已清空，原有 5 张图片",
  "cleared_count": 5,
  "timestamp": "2024-01-15T10:30:00"
}
```

**示例请求**:
```bash
curl -X DELETE "http://localhost:8001/api/banner/cache/clear"
```

---

#### 6. GET `/api/banner/cache` - 获取轮播图缓存详细信息

**功能**: 查看缓存中的所有轮播图详细信息

**响应**:
```json
{
  "cache_status": {
    "status": "ready",
    "last_update": "2024-01-15T10:30:00",
    "cache_count": 5
  },
  "cached_images": [
    {
      "url": "https://www.openharmony.cn/mainImg/banner1.png",
      "alt": "OpenHarmony 5.0",
      "filename": "banner1.png",
      "extracted_at": "2024-01-15T10:30:00",
      "source": "enhanced_crawler"
    }
  ],
  "summary": {
    "total_images": 5,
    "last_update": "2024-01-15T10:30:00",
    "update_count": 12
  },
  "timestamp": "2024-01-15T10:35:00"
}
```

---

## 通用规范

### 1. HTTP 状态码

| 状态码 | 说明 | 使用场景 |
|--------|------|----------|
| 200 | 成功 | 请求成功，数据正常返回 |
| 404 | 未找到 | 资源不存在（如文章ID不存在） |
| 500 | 服务器错误 | 服务器内部错误 |
| 503 | 服务不可用 | 服务暂时不可用（如缓存错误） |

### 2. 日期时间格式

**日期字段** (`date`):
- 格式: `YYYY-MM-DD`
- 示例: `"2024-01-15"`

**时间戳字段** (`created_at`, `updated_at`, `timestamp`):
- 格式: ISO 8601 格式
- 示例: `"2024-01-15T10:30:00"` 或 `"2024-01-15T10:30:00.123456"`

### 3. 分页规范

**请求参数**:
- `page`: 页码，从 **1** 开始（不是0）
- `page_size`: 每页数量，范围 1-100

**响应字段**:
- `total`: 总记录数
- `page`: 当前页码
- `page_size`: 每页数量
- `has_next`: 是否有下一页
- `has_prev`: 是否有上一页

**计算公式**:
```python
has_next = (page * page_size) < total
has_prev = page > 1
```

### 4. 搜索和过滤

**搜索逻辑**:
- 默认搜索 `title` 和 `summary` 字段
- 使用 SQL `LIKE` 模糊匹配
- 不区分大小写

**过滤逻辑**:
- `category`: 精确匹配
- `source`: 精确匹配
- 日期范围: 使用 `>=` 和 `<=` 比较

### 5. 缓存状态

所有缓存服务支持三种状态:

| 状态 | 值 | 说明 |
|------|------|------|
| 准备中 | `preparing` | 首次加载或正在更新 |
| 就绪 | `ready` | 缓存可用 |
| 错误 | `error` | 服务错误 |

### 6. 错误响应格式

**标准错误响应**:
```json
{
  "detail": "错误描述信息"
}
```

**示例**:
```json
{
  "detail": "文章不存在"
}
```

---

## 开发新数据源指南

### 1. 数据模型兼容性检查

在开发新数据源之前，先评估数据结构:

**✅ 可以直接使用 NewsArticle 模型的场景**:
- 新闻/文章类数据
- 包含标题、日期、URL、内容
- 内容可以拆分为文本/图片/代码块

**⚠️ 需要扩展模型的场景**:
- 需要额外字段（如作者、阅读量、标签）
- 参考 `templates/model_template.py` 的 Option 2

**❌ 需要完全自定义模型的场景**:
- 数据结构与新闻完全不同
- 参考 `templates/model_template.py` 的 Option 3

### 2. 保持 API 响应一致性

**分页接口必须包含**:
```python
{
  "total": int,
  "page": int,
  "page_size": int,
  "items": List[...],  # 或 "articles"
  "has_next": bool,     # 可选
  "has_prev": bool      # 可选
}
```

**状态接口必须包含**:
```python
{
  "service_status": {
    "status": "ready" | "preparing" | "error",
    "last_update": str,
    "cache_count": int
  }
}
```

### 3. 新增字段的最佳实践

**在 NewsArticle 模型基础上新增字段**:

```python
# models/yourdatasource.py
from models.news import NewsArticle

class YourDataSourceArticle(NewsArticle):
    author: Optional[str] = None
    view_count: Optional[int] = None
    tags: Optional[List[str]] = None
```

**API 响应保持兼容**:

```python
# api/yourdatasource.py
@router.get("/news", response_model=NewsListResponse)
async def get_news():
    # ...
    return NewsListResponse(
        total=total,
        page=page,
        page_size=page_size,
        items=articles  # 即使有额外字段，也兼容基础模型
    )
```

### 4. 数据源标识规范

**source 字段命名建议**:
- 使用清晰的中文或英文标识
- 避免特殊字符
- 保持唯一性

**示例**:
```python
source = "HuaweiDev"           # ✅ 简洁清晰
source = "华为开发者社区"        # ✅ 中文标识
source = "huawei-dev-community" # ✅ 短横线分隔
source = "huawei_dev"          # ✅ 下划线分隔

source = "huawei!dev"          # ❌ 避免特殊字符
source = "OpenHarmony"         # ❌ 已被使用
```

---

### 5. Category 分类规范（⚠️ 必须遵守）

> **🔴 重要提醒**：开发新数据源爬虫时，**必须**为每篇文章设置 `category` 字段，以支持主接口 `/api/news/` 的分类过滤功能。这是确保数据能够正确集成到系统的**强制要求**。

#### 📋 为什么必须支持 category？

主接口 `/api/news/` 使用 `category` 参数作为核心过滤机制：
- ✅ 用户可以通过 `/api/news/?category=XXX` 筛选特定类型的新闻
- ✅ 先过滤后分页，确保分页准确性
- ✅ 统一的数据管理和展示方式

**如果不设置 category**：
- ❌ 你的数据无法通过主接口的 category 参数筛选
- ❌ 前端无法正确分类展示你的数据
- ❌ 与现有数据源不兼容

#### 🎯 如何设置 category

**在爬虫的 `_format_article` 方法中设置**：

```python
# services/your_crawler.py

class YourCrawler:
    def _format_article(self, article):
        """将文章格式化为统一的新闻格式"""
        import hashlib
        
        article_id = hashlib.md5(article['url'].encode()).hexdigest()[:16]
        
        return {
            "id": article_id,
            "title": article['title'],
            "date": article.get('date', ''),
            "url": article['url'],
            "content": article.get('content', []),
            "category": "你的分类名称",  # ⚠️ 必须设置
            "summary": article.get('summary', ''),
            "source": "你的数据源标识",
            "created_at": datetime.now().isoformat(),
            "updated_at": datetime.now().isoformat()
        }
```

#### 📌 Category 命名规范

1. **使用有意义的中文名称**（推荐）或英文名称
2. **保持简洁**，2-6 个字符为宜
3. **避免重复**，查看现有 category 值
4. **区分大小写**，用户请求时必须完全匹配

**当前已使用的 category**：
| Category | 数据源 | 说明 |
|----------|--------|------|
| `"官方动态"` | OpenHarmony 官网 | 官方新闻公告 |
| `"技术博客"` | OpenHarmony 技术博客 | 技术深度文章 |

**命名建议**：
```python
category = "社区动态"    # ✅ 简洁清晰
category = "开发者分享"  # ✅ 描述性强
category = "版本发布"    # ✅ 明确分类
category = "技术教程"    # ✅ 易于理解

category = "官方动态"    # ❌ 已被使用，避免重复
category = "OpenHarmony官网的最新新闻动态"  # ❌ 太长
category = "dev-share"   # ⚠️ 可以但不推荐，优先使用中文
```

#### ✅ 完整示例

以下是一个完整的爬虫示例，展示如何正确设置 category：

```python
# services/huawei_dev_crawler.py

from typing import List, Dict
from datetime import datetime
import logging

logger = logging.getLogger(__name__)

class HuaweiDevCrawler:
    """华为开发者社区爬虫"""
    
    def __init__(self):
        self.base_url = "https://developer.huawei.com"
    
    def _format_article(self, article):
        """格式化文章数据"""
        import hashlib
        
        # 生成文章ID
        article_id = hashlib.md5(article['url'].encode()).hexdigest()[:16]
        
        # 返回统一格式 - 注意 category 和 source 的设置
        return {
            "id": article_id,
            "title": article['title'],
            "date": article.get('date', ''),
            "url": article['url'],
            "content": article.get('content', []),
            "category": "开发者分享",     # ⚠️ 必须设置 category
            "summary": article.get('summary', ''),
            "source": "华为开发者社区",    # 数据源标识
            "created_at": datetime.now().isoformat(),
            "updated_at": datetime.now().isoformat()
        }
    
    def crawl_articles(self, batch_callback=None, batch_size=20) -> List[Dict]:
        """爬取文章"""
        articles = []
        
        try:
            # 爬取逻辑...
            raw_articles = self._fetch_articles()
            
            # 格式化每篇文章（包含 category）
            for raw_article in raw_articles:
                formatted = self._format_article(raw_article)
                articles.append(formatted)
                
                # 分批回调
                if batch_callback and len(articles) % batch_size == 0:
                    batch_callback(articles[-batch_size:])
            
            logger.info(f"✅ 爬取完成，共 {len(articles)} 篇文章")
            return articles
            
        except Exception as e:
            logger.error(f"❌ 爬取失败: {e}")
            raise
```

#### 🔗 集成到主接口

设置 category 后，你的数据会自动支持主接口的分类过滤：

```bash
# 用户可以通过主接口筛选你的数据
GET /api/news/?category=开发者分享&page=1&page_size=20

# 响应示例
{
  "articles": [
    {
      "id": "abc123",
      "title": "HarmonyOS 开发实践",
      "category": "开发者分享",    # 你设置的 category
      "source": "华为开发者社区",   # 你设置的 source
      ...
    }
  ],
  "total": 45,
  "page": 1,
  "page_size": 20,
  "has_next": true,
  "has_prev": false
}
```

#### 📝 集成检查清单

在将新数据源合并到主线之前，请确认：

- [ ] ✅ 每篇文章都设置了 `category` 字段
- [ ] ✅ `category` 值简洁、有意义、不与现有值重复
- [ ] ✅ 测试 `/api/news/?category=你的分类名` 能正确返回数据
- [ ] ✅ 更新 `附录 B. 数据源对照表`，添加你的数据源信息
- [ ] ✅ 在 PR 描述中说明新增的 category 值及其含义

#### 🚫 常见错误

**错误1：不设置 category**
```python
# ❌ 错误示例
return {
    "title": "文章标题",
    "source": "我的数据源",
    # category 未设置或为 None
}
```

**错误2：使用与 source 相同的值**
```python
# ❌ 不推荐
return {
    "category": "华为开发者社区",  # 与 source 重复
    "source": "华为开发者社区"
}

# ✅ 正确做法
return {
    "category": "开发者分享",      # 描述内容类型
    "source": "华为开发者社区"     # 标识数据来源
}
```

**错误3：动态生成 category**
```python
# ❌ 错误示例：每篇文章的 category 都不同
return {
    "category": f"分类_{article_id}",  # 失去分类意义
}

# ✅ 正确做法：同一数据源的所有文章使用相同的 category
return {
    "category": "开发者分享",  # 所有文章统一分类
}
```

---

### 7. 内容结构化建议

**如果新数据源的文章内容包含多种类型**，建议使用 `NewsContentBlock` 结构:

```python
content = [
    {"type": "text", "value": "文章第一段..."},
    {"type": "image", "value": "https://example.com/image.jpg"},
    {"type": "code", "value": "const foo = 'bar';"},
    {"type": "text", "value": "文章第二段..."}
]
```

**如果内容比较简单**（纯文本），可以使用单个文本块:

```python
content = [
    {"type": "text", "value": "整篇文章的内容..."}
]
```

### 8. 测试新数据源

**开发完成后，测试以下场景**:

1. **基础功能**:
```bash
# 获取列表
curl "http://localhost:8001/api/yourdatasource/news?page=1&page_size=10"

# 搜索
curl "http://localhost:8001/api/yourdatasource/news?search=关键词"

# 获取详情
curl "http://localhost:8001/api/yourdatasource/news/123"
```

2. **边界情况**:
```bash
# 空结果
curl "http://localhost:8001/api/yourdatasource/news?search=不存在的关键词"

# 分页边界
curl "http://localhost:8001/api/yourdatasource/news?page=999999"

# 无效ID
curl "http://localhost:8001/api/yourdatasource/news/invalid_id"
```

3. **服务状态**:
```bash
# 检查服务状态
curl "http://localhost:8001/api/yourdatasource/status"

# 手动爬取
curl -X POST "http://localhost:8001/api/yourdatasource/crawl"
```

---

## 附录

### A. 完整字段参考表

#### NewsArticle 所有字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | string | 否 | 唯一标识 |
| title | string | 是 | 标题 |
| date | string | 是 | 发布日期 YYYY-MM-DD |
| url | string | 是 | 原文链接 |
| content | List[NewsContentBlock] | 是 | 内容块数组 |
| category | string | 否 | 分类 |
| summary | string | 否 | 摘要 |
| source | string | 否 | 数据源标识 |
| created_at | datetime | 否 | 创建时间 |
| updated_at | datetime | 否 | 更新时间 |

#### BannerResponse 所有字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| success | bool | 是 | 是否成功 |
| images | List[str] | 是 | 图片URL数组 |
| total | int | 是 | 图片总数 |
| message | string | 是 | 响应消息 |
| timestamp | string | 是 | 响应时间 ISO格式 |

### B. 数据源对照表

> **说明**：所有新数据源都必须设置 `category` 字段，以支持主接口 `/api/news/?category=XXX` 的分类过滤功能。

| 数据源名称 | source 标识 | category 值 | 主接口筛选 | 专用接口 |
|-----------|------------|------------|-----------|---------|
| OpenHarmony 官网 | OpenHarmony | 官方动态 | `/api/news/?category=官方动态` | `/api/news/openharmony` |
| OpenHarmony 技术博客 | OpenHarmony技术博客 | 技术博客 | `/api/news/?category=技术博客` | `/api/news/blog` |
| 官网手机版 Banner | - | - | - | `/api/banner/mobile` |

**添加新数据源时，请更新此表**，格式如下：
```markdown
| 你的数据源名称 | 你的source标识 | 你的category值 | `/api/news/?category=你的category值` | `/api/your_path` |
```

### C. 快速参考 - API Cheat Sheet

```bash
# 新闻 API - 主接口（推荐）
GET  /api/news/                              # 获取所有新闻（分页）
GET  /api/news/?category=官方动态             # 按分类筛选（推荐）✅
GET  /api/news/?category=技术博客             # 按分类筛选（推荐）✅
GET  /api/news/?search=关键词                 # 搜索新闻
GET  /api/news/?category=官方动态&search=5.0  # 组合筛选
GET  /api/news/?all=true                     # 获取所有新闻（不推荐）

# 新闻 API - 其他接口
GET  /api/news/openharmony         # 获取官网新闻（不推荐，使用category代替）
GET  /api/news/blog                # 获取技术博客（不推荐，使用category代替）
GET  /api/news/{id}                # 获取新闻详情
POST /api/news/crawl               # 手动爬取新闻
GET  /api/news/status/info         # 获取服务状态
POST /api/news/cache/refresh       # 刷新缓存

# 轮播图 API
GET    /api/banner/mobile          # 获取轮播图（缓存）
GET    /api/banner/mobile/enhanced # 增强版爬虫获取
GET    /api/banner/status          # 获取服务状态
POST   /api/banner/crawl           # 手动爬取
DELETE /api/banner/cache/clear     # 清空缓存
GET    /api/banner/cache           # 查看缓存详情
```

---

**文档版本**: v1.1
**最后更新**: 2025-01-20
**维护者**: XBXyftx

如有疑问，请参考:
- `COLLABORATION_GUIDE.md` - 团队协作指南
- `templates/README.md` - 开发教程
- `README.md` - 项目总体文档
