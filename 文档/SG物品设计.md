## 背包系统设计
系统中引入pvp开场技能，技能可通过观看广告获得。之前设计为，每次游戏之后刷新技能栏。现在需求，可以将观看广告获得的技能保存。所以设计背包系统。
### 物品数据结构
``` erlang
#goods{
    id        = Id,
    player_id = PlayerId,
    goods_id  = GoodsId,
    count     = Count,
    repo      = ?GOODS_REPO_BAG,
    type      = Type,
    subtype   = SubType
  }.
```
- 类型：物品一级类型
  - 消耗品、装备、材料
- 子类型：物品二级类型
  - 消耗品：技能buff、体力瓶
- repo：物品存储类型
  - 玩家背包
  - 仓库
- id:数据库存储id
- player_id: 玩家id
- goods_id: 物品id，例如buffid
- count:物品数量
### 技能Buff背包存储设计：
- 类型：消耗品->1
- 子类型：技能buff->1
- repo：玩家背包
- goods_id: buffid 1,2,3,4