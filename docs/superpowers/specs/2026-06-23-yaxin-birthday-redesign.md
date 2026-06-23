# 雅心生日网站重设计 · Design Spec

**日期：** 2026-06-23  
**目标上线：** 2026-06-26（生日当天）  
**Phase：** 1（生日礼物版）

---

## 概述

将现有的 Tab 式单页 HTML 网站重构为**单页滚动、居中卡片**风格的生日礼物网站。整体品牌保留「泡面专利事务所 · Poodle Noodles Intellectual Property」，泡面（雅心的猫）作为全站主持人贯穿始终。

部署方式：GitHub Pages（单文件 HTML + 静态资源在 repo）。  
动态内容（朋友祝福、故事、合照）：Firebase Realtime Database + Storage（新建项目）。

---

## 技术栈

- **HTML/CSS/JS**：单文件 `index.html`，Tailwind CDN，Lucide Icons
- **字体**：Ma Shan Zheng（手写标题）+ Noto Sans SC
- **动态后端**：Firebase Realtime DB（文字）+ Firebase Storage（语音、图片）
- **生日贺卡**：HTML Canvas 合成
- **语音录制**：MediaRecorder API → Firebase Storage
- **摄像头**：getUserMedia API + Canvas 截图
- **八戒音乐**：YouTube embed in modal（`https://www.youtube.com/embed/SzbW0r0z_iU`）
- **静态资源**：菜肴图片、泡面原声 `.mp3`、辣椒红酒背景图 → GitHub repo

---

## 页面结构

### 固定顶部导航
- 左侧：猫图标 + 「泡面专利事务所」/ `POODLE NOODLES INTELLECTUAL PROPERTY`
- 右侧：7 个锚点跳转 pill（Hero · 祝福墙 · 她的故事 · 招牌菜 · 贺卡 · 合照 · 彩蛋）
- 背景：`bg-wine-950/80` + `backdrop-blur`，sticky

---

### 板块 1：Hero

**内容：**
- 辣椒红酒图（`red-chili.png`）铺满，渐变遮罩
- 泡面 SVG 猫头像（圆形，金色边框），点击播放真实喵叫声（`meow.mp3` 静态文件）
- 泡面对话气泡：「喵呜～ 欢迎来到本喵的地盘。今天是雅心的日子，所有朋友集合！」
- 大标题：「生日快乐，雅心」（Ma Shan Zheng 字体，金色）
- 日期判断：6/26 之前显示倒计时；6/26 当天显示「今天就是你的日子！」
- 向下滚动引导箭头

**交互：**
- 点击猫头像 → 播放 `meow.mp3`（如文件存在）或合成音效（fallback）

---

### 板块 2：群星祝福墙

**内容：**
- 标题：「群星祝福墙」
- 表单：名字输入框 + 文字区域 + 录音按钮（最长 30s）+ 上传图片按钮
- 提交按钮：「挂上星光墙 ✨」
- 提交成功后泡面反应：「本喵收到了！已经为你挂上星光墙喵～」
- 下方展示所有祝福卡片，实时同步

**数据结构（Firebase）：**
```
/blessings/{id}
  author: string
  text: string
  audioUrl?: string   // Firebase Storage URL
  imageUrl?: string   // Firebase Storage URL
  timestamp: number
```

**祝福卡片显示：**
- 作者名 + 时间
- 文字内容
- 若有语音：播放按钮
- 若有图片：缩略图可点开大图

---

### 板块 3：雅心为我做过的事

**内容：**
- 标题：「雅心为我做过的事」
- 副标题：「她随手做的那些事，我们一直记得」
- 表单：名字 + 故事文本框（placeholder 示例：「拔了四颗智齿，是她来接我、买了药、煮了粥……」）
- 提交成功后泡面反应：「这件事本喵帮你记住了，雅心一定不知道有人还记得……」
- 下方展示故事卡片，每张卡片有作者名 + 故事全文

**数据结构（Firebase）：**
```
/stories/{id}
  author: string
  story: string
  timestamp: number
```

---

### 板块 4：雅心的招牌菜

**内容：**
- 标题：「雅心的招牌菜」
- 拍立得风格图片卡片网格（3 列）
- 每张卡片：菜肴照片 + 可选菜名 + 可选一句话
- 点击图片 → 全屏 lightbox

**实现：**
- 图片文件存于 `assets/food/` 目录（repo 内）
- 菜单数据硬编码在 JS 数组中（`{ src, name, caption }`）
- 需要 host（用户）在上线前把照片放入 repo

---

### 板块 5：生日贺卡生成

**内容：**
- 标题：「生日贺卡」
- 预览区：Canvas 渲染，以辣椒红酒图为底图，叠加：
  - 「生日快乐，雅心」大字（金色，Ma Shan Zheng）
  - 自动拉取祝福墙最多 8 条祝福（名字 + 截断至 20 字）
  - 泡面爪印印章
- 「一键生成贺卡」按钮：将 Canvas 导出为 PNG，长按/右键保存
- 泡面反应：「本喵盖章认证！这是全宇宙最用心的贺卡。」

**Canvas 合成逻辑：**
1. 绘制辣椒红酒图（`red-chili.png`）铺满，加深色遮罩
2. 绘制标题文字
3. 读取 Firebase blessings，绘制祝福人名 + 截断文字
4. 绘制底部泡面印章 SVG
5. `canvas.toDataURL('image/png')` → 触发下载

---

### 板块 6：大合照 & 朋友相册

**内容：**
- 标题：「大合照 & 朋友相册」
- 自拍区：点击「拍合照」→ 调用摄像头（getUserMedia）→ 倒计时 3-2-1 → Canvas 截图 → 上传 Firebase Storage → 实时出现在相册墙
- 「上传我的照片」按钮：file input → 上传 Firebase Storage
- 相册墙：瀑布流或网格，实时同步，点开大图

**数据结构（Firebase）：**
```
/photos/{id}
  url: string         // Firebase Storage URL
  uploader?: string
  timestamp: number
```

---

### 板块 7：彩蛋区

#### 7a. 🎰 喵星专利局（老虎机）
- 三个转轮（图标池：🌶️ 🍷 🐱 🎂 ⚖️ 📜）
- 「拉！」按钮触发动画（每轮随机停止）
- 胜利条件：三个相同图标 → Canvas 生成烫金专利证书弹窗（发明人名字可自定义输入）
- 🐱🐱🐱 三连有专属文案：「恭喜！获得泡面亲自颁发的喵星最高荣誉！」
- 其他三连文案根据图标类型随机从预设池选取
- 泡面盖章音效（合成短促击打声）

#### 7b. 🍷 深夜小酒馆（精美版）
- 中央大酒杯 SVG 动画
- 点击配料按钮（辣椒、红酒、厨艺、好运、泡面踩奶）→ 酒杯液面渐升动画
- 选满 3 种配料后「碰撞！」按钮亮起 → 生成诗意一句话祝福（预设文案，按配料组合）
- 优雅的颜色渐变和音效（玻璃碰撞声）

#### 7c. 🐷 八戒彩蛋
- 按钮文案（固定）：「听说这首歌能把人的老公都洗脑，我们不信邪，点点看 🐷」
- 点击 → modal 弹出，包含：
  - 一句说明：「你讲过的故事——爱人循环播放《八戒》，把老公都洗脑会唱了。我们记住了。」
  - YouTube iframe embed（`https://www.youtube.com/embed/SzbW0r0z_iU`，自动播放）
- **核心原则**：彩蛋的价值是「被认真听见、被记住」，歌是载体，故事才是主角

---

## 泡面的角色

泡面不是一个 Tab，是全站主持人：

| 位置 | 表现 |
|------|------|
| 顶部导航 Logo | 猫图标常驻 |
| Hero | SVG 猫头像，点击喵叫，说开场白 |
| 祝福墙提交成功 | Toast：「本喵收到了！已经为你挂上星光墙喵～」 |
| 故事墙提交成功 | Toast：「这件事本喵帮你记住了……」 |
| 贺卡生成 | Toast：「本喵盖章认证！」 |
| 合照拍完 | Toast：「喵！本喵已经存档，永久保存。」 |
| 老虎机 | 三连🐱 触发专利证书颁发 |

**喵叫声**：`assets/meow.mp3`（真实录音），点击任何地方的泡面头像都播放。合成音效作为 fallback。

---

## Firebase 配置

用户需要在上线前：
1. 在 Firebase console 新建项目
2. 开启 Realtime Database（测试模式）
3. 开启 Storage（测试模式）
4. 将 `firebaseConfig` 对象填入 `index.html` 顶部的配置区域

数据库规则（开发阶段，生日后可锁）：
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

---

## 静态资源清单（用户需提供）

| 文件 | 说明 |
|------|------|
| `assets/red-chili.png` | 辣椒红酒图（已有，背景 + 贺卡底图） |
| `assets/meow.mp3` | 泡面真实喵叫声录音 |
| `assets/food/*.jpg` | 雅心招牌菜照片（任意数量） |

---

## 不在 Phase 1 范围内

- 红酒品鉴日记
- Recipe 大集合（可持续上传）
- 任何登录/权限系统
