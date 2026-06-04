# 行走的宝藏店 - 架构设计文档

## 项目定位
一款**像素风探店记录**微信小程序。用户可以：
- 发现/添加宝藏店铺（含地图定位）
- 记录每家店的菜品（拍照 + 价格）
- 按标签、评分分类
- 心愿单 & 去过 管理
- 消费统计 & 探店笔记
- 精美分享卡片 & 图片墙

---

## 技术栈

| 层面 | 选型 | 说明 |
|-----|------|------|
| 框架 | 原生微信小程序 + TypeScript + Sass | 沿用现有 |
| 组件框架 | glass-easel（已启用） | 沿用现有 |
| 渲染引擎 | Skyline（已启用） | 性能好 |
| 后端/数据 | **微信云开发** | 云数据库 + 云存储（图片）+ 云函数 |
| 主题系统 | CSS Custom Properties | 像素风 / 简约风 / 暗黑风 |

---

## 数据模型

### 1. stores（店铺集合）
```
{
  _id: string,
  name: string,              // 店名
  address: string,           // 地址文字
  location: {                // 地图坐标
    lat: number,
    lng: number
  },
  tags: string[],            // 标签：川菜/日料/火锅/烧烤...
  cuisine: string,           // 菜系
  avgPrice: number,          // 人均（自动计算）
  rating: {                  // 综合评分
    taste: number,           // 口味 1-5
    environment: number,     // 环境 1-5
    service: number,         // 服务 1-5
    value: number            // 性价比 1-5
  },
  coverImage: string,        // 店铺封面（云存储 fileID）
  status: 'visited' | 'wishlist',  // 去过 / 想去
  visitCount: number,        // 到访次数
  notes: string,             // 店铺笔记
  createTime: Date,
  updateTime: Date
}
```

### 2. dishes（菜品集合）
```
{
  _id: string,
  storeId: string,           // 所属店铺
  name: string,              // 菜品名
  price: number,             // 价格
  photos: string[],          // 菜品照片（云存储 fileID 数组）
  category: string,          // 分类：招牌/凉菜/热菜/主食/甜品/饮品
  rating: number,            // 评分 1-5
  tags: string[],            // 标签：推荐/必点/一般/踩雷
  notes: string,             // 菜品点评
  createTime: Date
}
```

### 3. visits（探店记录）
```
{
  _id: string,
  storeId: string,
  storeName: string,         // 冗余方便展示
  date: Date,                // 到店日期
  dishes: [                  // 本次点的菜品
    {
      dishId: string,
      name: string,
      price: number
    }
  ],
  totalCost: number,         // 总花费
  personCount: number,       // 人数
  rating: {                  // 本次体验评分
    taste: number,
    environment: number,
    service: number,
    value: number
  },
  notes: string,             // 探店笔记
  photos: string[],          // 本次探店环境照等
  createTime: Date
}
```

### 4. wishlist（心愿单 - 独立于 stores 的轻量记录）
```
{
  _id: string,
  name: string,
  address: string,
  reason: string,            // 为什么想去
  source: string,            // 来源：朋友推荐/小红书/抖音/路过
  createTime: Date
}
```

---

## 页面路由结构

```
pages/
├── index/                  # 首页 - 瀑布流展示所有宝藏店
├── store-detail/           # 店铺详情 - 菜品列表 + 探店历史
├── store-edit/             # 添加/编辑店铺
├── dish-edit/              # 添加/编辑菜品
├── visit-record/           # 记录一次探店
├── photo-wall/             # 图片墙
├── wishlist/               # 心愿单
├── statistics/             # 消费统计
├── search/                 # 搜索
├── share-card/             # 分享卡片生成
└── settings/               # 设置（主题切换等）
```

---

## 像素风主题系统

### 设计理念
- 8-bit 像素风 UI，带有复古游戏机/街机感
- 使用 CSS `box-shadow` 模拟像素边框
- 像素风字体（内置或导入）
- 颜色使用高饱和对比色

### 主题 CSS 变量
```scss
// 像素风（默认）
:root {
  --font-pixel: 'Press Start 2P', monospace;
  --font-body: 'VT323', monospace;
  --color-bg: #1a1a2e;
  --color-surface: #16213e;
  --color-primary: #e94560;
  --color-secondary: #0f3460;
  --color-accent: #f5c518;
  --color-text: #eee;
  --color-text-dim: #888;
  --border-pixel: 4px solid #fff;
  --shadow-pixel: 4px 4px 0 #000;
  --radius: 0px;            // 像素风不要圆角
}

// 简约风
[data-theme="minimal"] {
  --font-pixel: system-ui;
  --font-body: system-ui;
  --color-bg: #fafafa;
  --color-surface: #fff;
  --color-primary: #333;
  --color-text: #222;
  --border-pixel: 1px solid #ddd;
  --shadow-pixel: 0 2px 8px rgba(0,0,0,0.1);
  --radius: 12rpx;
}

// 暗黑风
[data-theme="dark"] { ... }
```

---

## 像素风 UI 实现技巧

1. **像素边框**：用多层 `box-shadow` 代替 `border`
2. **字体**：小程序内嵌像素字体或使用兼容方案
3. **图标**：纯 CSS 像素图标或 sprite 雪碧图
4. **动画**：逐帧动画（steps()），类似 RPG 文字弹出
5. **按钮**：按下时 `translate(2px, 2px)` + box-shadow 缩小，模拟"按下"效果
6. **分割线**：用重复的 box-shadow 点阵模拟像素虚线

---

## 云开发配置

### 需开通
- 云数据库（stores, dishes, visits, wishlist）
- 云存储（店铺封面、菜品照片、环境照）
- 云函数（可选：复杂查询、数据统计）

### 数据库安全规则
- 个人小程序，仅创建者可读写

---

## 首页交互设计

- 顶部：像素风标题 "行走的宝藏店" + 搜索图标
- 筛选栏：标签 chips（像素风按钮）
- 主内容：双列瀑布流，每个卡片包含：
  - 封面图（像素风边框）
  - 店名（像素字体）
  - 标签
  - 评分星星（像素星星 ✦）
  - 人均价格
- 底部：+ 添加按钮（像素风大按钮）
- 底部导航：首页 | 心愿单 | 图片墙 | 统计 | 我的

---

## 文件结构

```
miniprogram/
├── app.json
├── app.ts
├── app.scss                      # 全局主题变量
├── theme/
│   ├── pixel.scss                # 像素风主题
│   ├── minimal.scss              # 简约风主题
│   └── dark.scss                 # 暗黑风主题
├── components/
│   ├── navigation-bar/           # 自定义导航栏（已有）
│   ├── store-card/               # 店铺卡片
│   ├── dish-card/                # 菜品卡片
│   ├── star-rating/              # 评分星星
│   ├── tag-chip/                 # 标签 chip
│   ├── pixel-button/             # 像素风按钮
│   ├── pixel-input/              # 像素风输入框
│   ├── empty-state/              # 空状态
│   └── image-picker/             # 图片选择器
├── pages/
│   ├── index/                    # 首页
│   ├── store-detail/             # 店铺详情
│   ├── store-edit/               # 编辑店铺
│   ├── dish-edit/                # 编辑菜品
│   ├── visit-record/             # 探店记录
│   ├── photo-wall/               # 图片墙
│   ├── wishlist/                 # 心愿单
│   ├── statistics/               # 统计
│   ├── search/                   # 搜索
│   └── settings/                 # 设置
├── utils/
│   ├── cloud.ts                  # 云开发初始化 & 工具函数
│   ├── theme.ts                  # 主题切换工具
│   ├── pixel-style.ts            # 像素风工具函数
│   └── util.ts                   # 通用工具（已有）
├── models/
│   └── index.ts                  # TypeScript 类型定义
└── images/                       # 图标、空状态图等
```

---

## 当前进度 (2026-06-04)

✅ **核心代码已全部完成** — 共 75+ 个文件，12 个页面，7 个组件，3 套主题。

### 已完成
- [x] 三个主题系统：像素风（默认）/ 简约风 / 暗黑风
- [x] 数据模型定义（TypeScript）
- [x] 本地存储 CRUD 工具（含云开发预留接口）
- [x] 7 个可复用组件：store-card, dish-card, pixel-button, star-rating, tag-chip, empty-state, image-picker
- [x] 12 个业务页面全部完成
- [x] 5 tab 底部导航栏
- [x] 主题切换功能
- [x] 数据导出/导入/清除
- [x] 像素风 UI 交互（button 按下位移、box-shadow 边框、steps() 动画）

### 待做
- [ ] 云开发环境配置（打开 `utils/cloud.ts`，替换 `CLOUD_ENV` 为实际环境 ID）
- [ ] tabBar 图标 PNG（备选：不使用 iconPath，纯文字 tabBar 当前已能跑）
- [ ] Canvas 绘制分享卡片并保存为图片
- [ ] 像素字体文件导入（如需外部字体）
- [ ] 微信开发者工具中真机预览调试

### 快速开始
1. 微信开发者工具打开此项目
2. 如需云开发：替换 `utils/cloud.ts` 中的 `CLOUD_ENV`，取消 `initCloud()` 调用
3. 编译运行，默认使用本地存储模式

---

## 文件总数概览

```
miniprogram/ 目录总计:
  12 页面 × 4 文件 = 48 页面文件
  7 组件 × 4 文件 = 28 组件文件 (含已有 navigation-bar)
  3 主题文件
  1 数据模型文件
  4 工具函数文件
  3 入口文件 (app.json/ts/scss)
  1 已有配置文件 (sitemap.json)
  ────────────────────────────
  共约 88 个文件
```
