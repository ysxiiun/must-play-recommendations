# Must-Play Recommendations

> 城市必玩景点推荐 Agent Skill

## 简介

本技能为用户提供城市必玩景点的智能推荐服务。基于 FlyAI CLI 获取景点数据，聚焦热门、打卡地、高品质景点，以结构化 JSON 格式输出推荐结果。

## 核心能力

- **5A级景区优先**：自动获取城市最高等级景区
- **智能降级补充**：5A不足时自动搜索4A级景区补充
- **热门探索**：通过关键词搜索发现用户关注度最高的景点
- **推荐理由生成**：基于等级+类别自动生成推荐文案
- **结构化输出**：标准 JSON 格式，便于下游处理

## 触发词

| 触发词 | 示例 |
|-------|------|
| `有什么好玩` | "杭州有什么好玩的" |
| `景点推荐` | "景点推荐 北京" |
| `必去景点` | "西安必去景点" |
| `必玩景点` | "三亚必玩景点" |
| `旅游推荐` | "成都旅游推荐" |
| `5A景区` | "上海5A景区" |

## 输入输出

### 输入

```
城市名称（必填）
```

示例：`北京`、`上海`、`杭州`、`西安`、`成都`、`三亚`

### 输出

JSON 格式景点推荐结果：

```json
{
  "status": "success",
  "city": "杭州",
  "summary": {
    "totalAttractions": 5,
    "5aCount": 4
  },
  "recommendations": [
    {
      "rank": 1,
      "name": "西湖风景名胜区",
      "level": "5A",
      "category": "山湖田园",
      "address": "浙江省杭州市西湖区龙井路1号",
      "isFree": true,
      "ticketInfo": null,
      "recommendReason": "国家5A级景区，杭州标志性打卡地，来杭州必去的经典景点",
      "jumpUrl": "https://a.feizhu.com/xxx",
      "mainPic": "https://img.alicdn.com/xxx.jpg",
      "tags": ["自然风光", "地标"]
    }
  ],
  "metadata": {
    "generatedAt": "2026-04-01T10:30:00Z",
    "dataSource": "FlyAI CLI",
    "version": "1.0.0"
  }
}
```

### 输出字段说明

| 字段 | 说明 |
|-----|------|
| `name` | 景点名称 |
| `level` | 景点等级（5A/4A/3A/未评级） |
| `category` | 景点类别 |
| `address` | 景点地址 |
| `isFree` | 是否免费 |
| `ticketInfo.price` | 门票价格（未知返回"未知"） |
| `recommendReason` | 推荐理由 |
| `jumpUrl` | 预订链接 |
| `mainPic` | 主图链接 |
| `tags` | 标签列表 |

## 推荐定位

**聚焦热门、打卡地、高品质景点**

推荐优先级：
1. 热门度 - 人气最高、最热门的景点
2. 打卡价值 - 独特体验、标志性景观
3. 景区等级 - 5A级优先，但不唯等级论
4. 口碑评分 - 用户评价高、体验好

## 扩展性

当前版本依赖 FlyAI CLI 数据源，未来可扩展：

| 数据源 | 增强价值 |
|-------|---------|
| FlyAI keyword-search | 发现热门搜索景点 |
| FlyAI ai-search | 理解用户偏好场景 |
| 小红书文章 | 网红打卡点、拍照攻略 |
| 大众点评 | 真实评价、体验反馈 |
| 携程/马蜂窝 | 必玩榜单排名 |

## 文件结构

```
must-play-recommendations/
├── SKILL.md              # 核心配置文件
├── package.json          # Skill 元信息
├── README.md             # 使用说明
└── flyai-readme.md       # FlyAI CLI 参考文档
```

## 依赖

- FlyAI CLI (`@fly-ai/flyai-cli`)
- Claude Code >= 1.0.0

## 许可证

MIT