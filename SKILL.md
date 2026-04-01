---
name: must-play-recommendations
description: 为用户推荐城市的必玩景点。当用户询问某个城市有什么好玩的、景点推荐、必去景点、旅游攻略时触发。核心能力是基于 FlyAI CLI 获取城市的5A级景区并提供详细推荐信息。
triggers:
  - 有什么好玩
  - 景点推荐
  - 必去景点
  - 必玩景点
  - 旅游推荐
  - 玩什么
  - 去哪玩
  - 5A景区
  - 名胜古迹
---

# 必玩景点推荐 Skill

## 概述

本技能为用户提供城市必玩景点的智能推荐服务。聚焦热门、打卡地、高品质景点，通过 FlyAI CLI 获取景点数据，整合生成结构化 JSON 格式输出。

---

## 推荐定位与策略

**核心定位**：聚焦热门、打卡地、高品质景点，为用户提供最具价值的旅行体验推荐。

**推荐优先级**：
1. **热门度** - 首先推荐人气最高、最热门的景点
2. **打卡价值** - 具有独特体验、标志性景观的景点
3. **景区等级** - 5A级优先，但不唯等级论
4. **口碑评分** - 用户评价高、体验好的景点

**禁止行为**：
- ❌ 无脑推荐免费景点
- ❌ 以"性价比"为由推荐低品质景点
- ❌ 推荐冷门、无特色景点

---

## 工作流程

### Step 1: 解析用户输入

1. 从用户请求中提取城市名称
2. 确认城市名称的有效性（支持中国主要城市）
3. 判断是否需要特定类别筛选（如历史古迹、自然风光等）

### Step 2: 获取5A级景区（核心推荐）

使用 FlyAI CLI 的 `search-poi` 命令获取城市5A级景区：

```bash
flyai search-poi --city-name "{城市名称}" --poi-level 5
```

**处理逻辑**：
- 如果返回结果为空，降级搜索4A级景区
- 最多取前5个高等级景区作为核心推荐
- 提取字段：name, address, category, poiLevel, freePoiStatus, ticketInfo, jumpUrl, mainPic

### Step 3: 获取补充推荐（可选）

使用多种方式获取补充景点：

**方式A - 关键词搜索热门**：
```bash
flyai keyword-search --query "{城市名称}有什么好玩的"
```

**方式B - 类别搜索**：
```bash
flyai search-poi --city-name "{城市名称}" --category "{类别}"
```

常用类别：
- `historic site` - 历史古迹
- `nature` - 自然风光
- `theme park` - 主题乐园
- `ancient town` - 古镇古村
- `museum` - 博物馆
- `landmark` - 地标建筑

**方式C - AI语义搜索**（复杂需求）：
```bash
flyai ai-search --query "推荐{城市名称}适合{场景}的景点"
```

### Step 4: 生成推荐理由

根据景点特征自动生成推荐理由：

| 等级+特色 | 推荐理由模板 |
|---------|-------------|
| 5A + 地标 | "国家5A级景区，{城市}标志性打卡地，来{城市}必去的经典景点" |
| 5A + 历史文化 | "国家5A级景区，深厚历史文化底蕴，体验{城市}独特的人文魅力" |
| 5A + 自然风光 | "国家5A级景区，绝美自然风光，摄影爱好者和旅行打卡的绝佳选择" |
| 5A + 主题乐园 | "国家5A级景区，热门游乐体验，适合亲子/年轻人打卡游玩" |
| 4A + 高热度 | "人气爆棚的打卡胜地，游客好评如潮，{城市}旅游热门推荐" |
| 特色体验 | "独特体验项目，{城市}旅行不可错过的精彩亮点" |

**类别映射规则**：
- `historic site` / `ancient town` / `temple` / `museum` → 历史文化
- `nature` / `lake` / `forest` / `canyon` / `beach` → 自然风光
- `theme park` / `water park` / `zoo` → 主题乐园
- `landmark` → 地标

### Step 5: 整合输出

将所有景点数据整合为标准 JSON 格式输出。

---

## 输出格式规范

### JSON 输出结构

```json
{
  "status": "success",
  "city": "{城市名称}",
  "summary": {
    "totalAttractions": 8,
    "5aCount": 3,
    "4aCount": 2
  },
  "recommendations": [
    {
      "rank": 1,
      "name": "故宫博物院",
      "level": "5A",
      "category": "博物馆",
      "address": "北京市东城区景山前街4号",
      "isFree": false,
      "ticketInfo": {
        "price": "60",
        "ticketName": "大门票 成人票",
        "priceDate": "2026-04-01"
      },
      "recommendReason": "国家5A级景区，北京标志性打卡地，来北京必去的经典景点",
      "jumpUrl": "https://a.feizhu.com/xxx",
      "mainPic": "https://img.alicdn.com/xxx.jpg",
      "tags": ["历史文化", "地标", "博物馆"]
    }
  ],
  "metadata": {
    "generatedAt": "2026-04-01T10:30:00Z",
    "dataSource": "FlyAI CLI",
    "version": "1.0.0"
  }
}
```

### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|-----|------|-----|------|
| `status` | string | Y | "success" 或 "error" |
| `city` | string | Y | 用户输入的城市名称 |
| `summary.totalAttractions` | number | Y | 总推荐景点数 |
| `summary.5aCount` | number | Y | 5A级景区数量 |
| `summary.4aCount` | number | N | 4A级景区数量 |
| `recommendations` | array | Y | 景点推荐列表 |
| `recommendations[].rank` | number | Y | 推荐排名 |
| `recommendations[].name` | string | Y | 景点名称 |
| `recommendations[].level` | string | Y | 景点等级（5A/4A/3A/未评级） |
| `recommendations[].category` | string | Y | 景点类别 |
| `recommendations[].address` | string | Y | 景点地址 |
| `recommendations[].isFree` | boolean | Y | 是否免费 |
| `recommendations[].ticketInfo` | object | N | 门票信息（收费景点） |
| `recommendations[].ticketInfo.price` | string | N | 门票价格，未知则返回"未知" |
| `recommendations[].recommendReason` | string | Y | 推荐理由 |
| `recommendations[].jumpUrl` | string | Y | 预订链接 |
| `recommendations[].mainPic` | string | Y | 主图链接 |
| `recommendations[].tags` | array | N | 标签列表 |

---

## 规则与约束

### 必须遵守

1. **优先高等级**: 始终优先推荐5A级景区，不足时降级4A
2. **聚焦热门**: 推荐人气高、打卡价值高的景点
3. **数量限制**: 核心推荐不超过5个，补充推荐不超过3个
4. **JSON输出**: 最终输出必须是完整有效的JSON
5. **价格显示**: 门票价格需注明币种（CNY），未知则返回"未知"

### 禁止操作

1. 禁止虚构景点信息
2. 禁止无脑推荐免费景点
3. 禁止以"性价比"为由推荐低品质景点
4. 禁止在没有数据时返回空数组（应返回错误提示）
5. 禁止输出非JSON格式的结果

---

## 扩展性设计

**扩展维度**：推荐依据和数据源的扩展能力

当前版本 v1.0.0 依赖 FlyAI CLI 的 `search-poi` 数据，推荐理由基于等级+类别规则（6项）。

**未来可扩展的推荐维度**：

| 数据源 | 扩展内容 | 增强推荐价值 |
|-------|---------|-------------|
| FlyAI keyword-search | 热门搜索结果 | 发现用户关注度最高的景点 |
| FlyAI ai-search | AI语义推荐 | 理解用户偏好场景（亲子、摄影、打卡） |
| 小红书文章 | 热门打卡笔记 | 真实用户体验、网红打卡点、拍照攻略 |
| 大众点评 | 用户评分口碑 | 真实评价、游玩体验反馈 |
| 携程/马蜂窝 | 游记攻略热度 | 行程推荐、必玩榜单排名 |

**架构设计原则**：
- 推荐引擎模块化，支持多数据源接入
- 推荐理由生成器可扩展规则库
- 输出JSON预留扩展字段（如 `socialHotRate`、`userReviewScore`）

---

## 错误处理

### 错误输出格式

```json
{
  "status": "error",
  "errorCode": "CITY_NOT_FOUND",
  "errorMessage": "未找到城市'{城市名称}'的景点数据",
  "suggestion": "请确认城市名称是否正确，或尝试使用附近城市名称"
}
```

### 错误类型

| 错误码 | 说明 | 处理建议 |
|-------|------|---------|
| `CITY_NOT_FOUND` | 城市无景点数据 | 提示用户检查城市名 |
| `FLYAI_ERROR` | FlyAI CLI调用失败 | 检查CLI安装状态 |
| `NO_ATTRACTIONS` | 无可用景点数据 | 尝试其他搜索方式 |
| `INVALID_INPUT` | 输入无效 | 提示用户提供城市名 |

---

## 使用示例

### 示例1：基础查询

**用户输入**：
```
推荐北京的必玩景点
```

**执行命令**：
```bash
flyai search-poi --city-name "北京" --poi-level 5
```

### 示例2：带类别筛选

**用户输入**：
```
西安有什么历史古迹推荐
```

**执行命令**：
```bash
flyai search-poi --city-name "西安" --category "historic site" --poi-level 5
flyai search-poi --city-name "西安" --category "historic site" --poi-level 4
```

### 示例3：热门探索

**用户输入**：
```
杭州有什么好玩的
```

**执行命令**：
```bash
flyai search-poi --city-name "杭州" --poi-level 5
flyai keyword-search --query "杭州有什么好玩的"
```

---

## Markdown 展示规则

当需要将结果以 Markdown 格式展示给用户时：

### 景点卡片格式

```markdown
## {rank}. {景点名称} ({等级})

![]({mainPic})

**类别**: {category}
**地址**: {address}
**门票**: {price} CNY（{ticketName}）
**推荐理由**: {recommendReason}

[点击预订]({jumpUrl})
```

### 汇总格式

```markdown
# {城市}必玩景点推荐

共推荐 {total} 个景点，其中5A级 {count} 个。

---

{景点卡片列表}

---

> 基于 fly.ai 实时结果 | 生成时间: {timestamp}
```

---

## 相关资源

- [FlyAI CLI 详细文档](./flyai-readme.md)