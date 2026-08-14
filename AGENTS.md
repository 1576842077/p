# PingUI 手机操作系统 - 需求拆解文档

## 产品概述

- **产品类型**: 移动操作系统模拟器（Web SPA）
- **场景类型**: <scene_type>prototype-app</scene_type>
- **目标用户**: 体验 Web 版手机操作系统的前端开发者 / 产品体验者 / 演示场景用户
- **核心价值**: 在浏览器中完整模拟一款名为 PingUI 的手机操作系统，包含开机、锁屏、桌面、多应用、控制中心等全链路交互，数据全部本地化存储
- **界面语言**: 中文
- **主题偏好**: 浅色（支持开发者选项中的强制深色模式切换）
- **导航模式**: 单页应用 + 应用栈管理（无传统导航，通过应用启动/关闭切换界面）
- **导航布局**: 无（单页应用，手机壳内全屏渲染系统界面）

---

## 页面结构总览

> **说明**：本项目为单页应用（SPA），所有界面通过状态切换渲染。以下按系统层级列出所有界面模块。

**主页面文件**: `PingUIPhone.tsx`（手机外壳 + 系统状态机容器）

| 界面模块 | 说明 | 类型 | 入口来源 |
|---------|------|------|---------|
| 开机动画 | 黑色背景 + PingUI Logo 渐变发光淡入动画 | 系统状态 | 首次加载 / 重启 |
| 锁屏界面 | 显示时间日期，点击/滑动解锁 | 系统状态 | 开机动画结束 |
| 桌面（主屏幕） | 状态栏 + 小组件 + 应用网格 + Dock 栏 | 系统状态 | 锁屏解锁 / 应用关闭返回 |
| 控制中心 | 顶部下滑呼出，毛玻璃效果，快捷开关 | 系统浮层 | 桌面/应用内顶部下滑 |
| 相机应用 | 取景界面 + 快门 + 前后摄切换 | 内置应用 | 桌面图标点击 |
| 相册应用 | 照片网格 + 大图浏览 + 删除 | 内置应用 | 桌面图标点击 / 控制中心截图后 |
| 浏览器应用 | 地址栏 + iframe 浏览 + 书签 + 历史 | 内置应用 | 桌面图标点击 |
| 设置应用 | 设置主列表 + 各子页面 + 关于本机 + 开发者选项 | 内置应用 | 桌面图标点击 |
| 功能介绍应用 | PingUI 系统功能特色图文介绍 | 内置应用 | 桌面图标点击 |
| 计算器应用 | 基础计算器 | 内置应用 | 控制中心快捷入口 |
| 计时器应用 | 倒计时 / 正计时 | 内置应用 | 控制中心快捷入口 |
| 手电筒效果 | 全屏白光覆盖层 | 系统功能 | 控制中心开关 |

> **说明**：设置应用为多级嵌套结构（设置主列表 → 子功能页 → 关于本机 → 开发者选项），均在设置应用容器内切换。

---

## 页面布局建议

- **布局模式**: 手机壳居中 + 内部系统全屏堆叠（应用栈 z-index 管理）
- **视觉重心**: 手机屏幕内的系统界面（桌面/应用），手机外壳为视觉容器
- **状态管理**: 采用系统状态机（boot → lockscreen → home → app），应用以栈式叠加打开，关闭时出栈
- **结果承载区**:
  - 相册/相机：拍摄结果以 dataURL 存入 localStorage，相册网格展示
  - 浏览器：iframe 作为内容承载区，书签/历史在侧边/底部面板
  - 设置：各子功能页承载对应配置表单与实时预览
  - 初始态：开机动画为初始态，桌面为解锁后默认态

---

## 数据来源声明

| 数据/操作 | 来源类型 | 实现要求 | mock 兜底 |
|---|---|---|---|
| 系统配置（WiFi/蓝牙/亮度/音量/壁纸/电池/主题等） | local-persist | localStorage key=`__pingui_system_config`，JSON 存储系统全局状态 | 初始默认值（WiFi 开、亮度 80%、默认壁纸等） |
| 相册照片 | local-persist | localStorage key=`__pingui_photos`，数组存储 {id, dataURL, timestamp, size} | 空数组（空相册友好提示） |
| 浏览历史记录 | local-persist | localStorage key=`__pingui_browser_history`，数组存储 {url, title, timestamp} | 初始 3 条常用网站（百度/知乎/B站）作为书签示例 |
| 书签/常用网站 | local-persist | localStorage key=`__pingui_browser_bookmarks`，数组存储 {url, title, icon} | 初始 6 条默认书签 |
| 开发者选项开关状态 | local-persist | localStorage key=`__pingui_dev_options_enabled`，布尔值 | false（默认隐藏） |
| 开发者选项参数（版本号/设备名/主题/动画速度等） | local-persist | localStorage key=`__pingui_dev_config`，JSON 存储 | 默认系统参数 |
| 设备信息（型号/序列号/存储/内存等） | demo-mock | src/data/deviceInfo.ts 静态常量，展示用 | ✅ 本身就是 mock 数据 |
| 功能介绍内容 | demo-mock | src/data/featureIntro.ts 静态图文内容 | ✅ 本身就是 mock 数据 |
| 截图保存到相册 | local-persist | html2canvas 截图 → dataURL → 追加到 `__pingui_photos` | 失败提示 toast |
| 重置系统数据 | local-persist | 清空所有 `__pingui_*` 前缀的 localStorage 键 | 无 |

---

## 功能列表

### 全局系统功能

- **系统状态机**:
  - **页面目标**: 管理开机 → 锁屏 → 桌面 → 应用的完整生命周期
  - **功能点**:
    - **开机动画播放**: 黑色背景 + PingUI Logo 渐变发光淡入 + 缩放动画，2-3 秒后自动跳转锁屏
    - **锁屏解锁**: 显示实时时间日期，支持点击解锁按钮或向上滑动解锁，解锁后过渡动画进入桌面
    - **应用栈管理**: 应用打开时入栈（从图标位置放大过渡），关闭时出栈（缩小回图标位置），支持应用间切换
    - **状态栏实时更新**: 时间每秒刷新，电量图标缓慢消耗（模拟放电），信号/WiFi/飞行模式图标随系统配置变化
    - **控制中心呼出**: 从屏幕顶部下滑跟手拖动，松手后根据位置自动展开或收起，毛玻璃背景效果

### 桌面（主屏幕）

- **页面目标**: 作为系统主界面，展示应用入口、小组件、Dock 栏
- **功能点**:
  - **状态栏**: 显示时间、信号强度、WiFi 状态、电池电量百分比 + 图标，实时更新
  - **小组件区**: 包含时钟小组件（模拟时钟/数字时钟）、天气小组件（静态模拟数据）、快捷功能小组件
  - **应用图标网格**: 4 列网格布局，图标为 SVG/CSS 绘制，点击有缩放反馈动画 + 应用打开过渡
  - **Dock 栏**: 底部固定 4 个常用应用（相机、相册、浏览器、设置），毛玻璃背景
  - **壁纸适配**: 根据系统配置的壁纸设置桌面背景图/渐变

### 相机应用

- **页面目标**: 模拟相机拍照功能，照片保存到本地相册
- **功能点**:
  - **取景界面**: 模拟相机取景框，顶部工具栏（切换前后摄、设置图标），底部快门区
  - **快门拍摄**: 点击快门按钮触发白屏闪烁动画 + 快门音效（可选 Web Audio），模拟拍摄
  - **前后摄切换**: 点击切换按钮，取景画面做翻转动画，模拟前后摄像头切换
  - **照片保存**: 拍摄后将画布内容（或模拟画面）转为 dataURL 存入 localStorage 相册
  - **拍照反馈**: 快门动画 + 震动反馈（navigator.vibrate 如支持）+ 缩略图预览

### 相册应用

- **页面目标**: 浏览、查看、删除拍摄的照片
- **功能点**:
  - **照片网格**: 3 列网格展示所有照片缩略图，按时间倒序排列
  - **大图查看**: 点击照片进入全屏大图模式，支持双指缩放（可选）
  - **滑动切换**: 大图模式支持触摸滑动和鼠标拖拽左右切换上一张/下一张
  - **删除照片**: 大图模式下删除按钮，确认后从 localStorage 移除
  - **空状态**: 无照片时显示友好空状态提示 + 引导去相机拍照

### 浏览器应用

- **页面目标**: 实现真实网页浏览功能（iframe 加载）
- **功能点**:
  - **地址栏导航**: 顶部地址栏可输入 URL，回车后加载网页，支持自动补全 http/https
  - **导航控制**: 前进、后退、刷新、主页按钮，前进/后退基于历史栈
  - **iframe 加载**: 使用 iframe 加载真实网页，处理 X-Frame-Options 限制（无法嵌入时提示并提供新窗口打开）
  - **书签/快捷入口**: 首页展示常用网站快捷入口，点击直接跳转
  - **历史记录**: 自动记录浏览历史，可查看和清除
  - **搜索代理**: 对于非 URL 输入，使用搜索引擎（如 Bing）搜索

### 设置应用

- **页面目标**: 提供系统各项配置功能
- **功能点**:
  - **设置主列表**: 分组展示所有设置项（网络、连接、显示、声音、壁纸、电池、存储、关于本机）
  - **网络与互联网**: WiFi 开关、移动数据开关、飞行模式开关，切换后实时影响状态栏图标
  - **蓝牙**: 蓝牙开关，切换后状态保存
  - **显示与亮度**: 亮度调节滑块，实时改变屏幕遮罩层透明度模拟亮度变化
  - **声音与振动**: 音量调节滑块、静音模式开关，控制系统音量状态
  - **壁纸设置**: 提供多张预设壁纸选择，点击应用后实时更新桌面背景
  - **电池**: 显示当前电量、省电模式开关，省电模式开启后电量消耗减缓
  - **存储**: 显示 localStorage 已用空间 / 总容量估算，进度条可视化
  - **关于本机**: 显示系统名称、版本号、设备名称、型号、序列号、存储、内存等信息；**点击"系统名称(PingUI)"行进入开发者选项**
  - **开发者选项**: 修改系统版本号、设备名称、系统配色主题、动画速度调节、显示触摸位置开关、GPU 渲染模拟开关、强制深色模式开关、重置系统数据按钮、隐藏开发者选项开关

### 功能介绍应用

- **页面目标**: 展示 PingUI 系统的功能特色
- **功能点**:
  - **图文排版**: 精美的卡片式布局，分模块介绍系统功能
  - **滚动浏览**: 纵向滚动浏览全部介绍内容
  - **动效点缀**: 卡片渐入、图标微动效，提升阅读体验

### 控制中心

- **页面目标**: 提供快捷功能开关和工具入口
- **功能点**:
  - **跟手下滑**: 从屏幕顶部状态栏区域开始下滑，控制中心面板跟随手指位置移动
  - **毛玻璃效果**: backdrop-filter 实现半透明毛玻璃背景
  - **快捷开关**: 飞行模式、WiFi、蓝牙、手电筒、静音模式，点击切换状态并有动画反馈
  - **亮度调节**: 亮度滑块，实时调整屏幕亮度
  - **音量调节**: 音量滑块，实时调整系统音量状态
  - **截图功能**: 点击截图按钮，使用 html2canvas 截取当前手机屏幕画面并保存到相册
  - **快捷入口**: 计算器、计时器快捷图标，点击打开对应应用
  - **手电筒效果**: 开启手电筒时，全屏覆盖半透明白色层模拟手电照明效果

### 计算器 & 计时器应用

- **页面目标**: 控制中心快捷入口对应的两个实用工具
- **功能点**:
  - **计算器**: 基础四则运算，数字键盘 + 运算结果显示
  - **计时器**: 正计时功能，开始/暂停/重置，秒级精度

---

## 数据共享配置

| 存储键名 | 数据说明 | 使用模块 |
|---------|---------|---------|
| `__pingui_system_config` | 系统全局配置，类型 `ISystemConfig` | 状态栏、控制中心、设置、桌面 |
| `__pingui_photos` | 相册照片列表，类型 `IPhoto[]` | 相机、相册、控制中心（截图） |
| `__pingui_browser_history` | 浏览历史，类型 `IBrowserHistoryItem[]` | 浏览器 |
| `__pingui_browser_bookmarks` | 书签列表，类型 `IBookmark[]` | 浏览器 |
| `__pingui_dev_config` | 开发者选项配置，类型 `IDevConfig` | 设置 - 关于本机 - 开发者选项 |
| `__pingui_dev_options_enabled` | 开发者选项是否显示，类型 `boolean` | 设置主列表 |
| `__pingui_current_app` | 当前打开的应用标识，类型 `string \| null` | 系统状态机 |

```ts
interface ISystemConfig {
  /** WiFi 开关 */
  wifi: boolean;
  /** 移动数据开关 */
  mobileData: boolean;
  /** 飞行模式开关 */
  airplaneMode: boolean;
  /** 蓝牙开关 */
  bluetooth: boolean;
  /** 屏幕亮度 0-100 */
  brightness: number;
  /** 系统音量 0-100 */
  volume: number;
  /** 静音模式 */
  silentMode: boolean;
  /** 壁纸标识 */
  wallpaper: string;
  /** 当前电量 0-100 */
  batteryLevel: number;
  /** 省电模式 */
  powerSaving: boolean;
  /** 手电筒状态 */
  flashlight: boolean;
  /** 系统主题色 */
  themeColor: string;
  /** 动画速度倍率 0.5 / 1 / 1.5 / 2 */
  animationSpeed: number;
  /** 显示触摸位置 */
  showTouchPosition: boolean;
  /** 强制深色模式 */
  forceDarkMode: boolean;
  /** 设备名称 */
  deviceName: string;
  /** 系统版本号（可被开发者选项修改） */
  systemVersion: string;
}

interface IPhoto {
  id: string;
  /** 图片 dataURL */
  dataURL: string;
  /** 拍摄时间戳 */
  timestamp: number;
  /** 文件大小（字节） */
  size: number;
  /** 来源：camera / screenshot */
  source: 'camera' | 'screenshot';
}

interface IBrowserHistoryItem {
  id: string;
  url: string;
  title: string;
  timestamp: number;
}

interface IBookmark {
  id: string;
  url: string;
  title: string;
  /** 图标 URL 或 emoji */
  icon: string;
}

interface IDevConfig {
  customVersion: string;
  customDeviceName: string;
  customThemeColor: string;
  animationSpeed: number;
  showTouches: boolean;
  gpuRenderingMock: boolean;
  forceDark: boolean;
}
```

---

## 技术实现要点（非功能需求）

> 供 Code Agent 参考的关键实现约束

1. **性能优化**: 使用 CSS transform / opacity 做动画，避免 layout 重排；应用关闭后可卸载组件释放内存
2. **动画流畅**: 所有过渡动画使用 CSS transition / animation，配合 `animationSpeed` 系统配置统一调速
3. **响应式**: 手机外壳宽度自适应（最大 390px，最小 320px），在不同屏幕尺寸下居中显示并保持比例
4. **图标方案**: 统一使用 SVG 组件或 CSS 绘制，不依赖外部图标库，确保离线可用
5. **localStorage 容错**: 所有读写操作包裹 try-catch，容量不足时有提示
6. **截图实现**: 使用 html2canvas 库截取手机屏幕容器（非整个页面），生成图片存入相册

-------

<scene_type>prototype-app</scene_type>

# UI 设计指南

## 1. 设计推导依据

- **参考意图**: Free Direction —— 无参考材料，从 PingUI 手机系统模拟的产品语义与现代移动端审美出发自主设计。
- **核心情绪 / 应用类型**: 轻量精致的掌上系统模拟，强调顺滑、跟手、拟真又有设计感的交互体验。
- **独特记忆点**: 以"Ping"的脉冲光点为视觉母题——开机动画中一圈圈扩散的淡蓝光晕、图标激活时的微弹脉冲、控制中心毛玻璃上的青色呼吸点，贯穿开机、解锁、应用打开全程。

## 2. Art Direction

- **方向名**: 脉冲玻璃 · Ping Glass
- **Design Style**: Frosted Glass 毛玻璃 + Soft Minimal 柔感极简 —— 手机系统需要在壁纸之上承载多层界面（状态栏、小组件、Dock、控制中心、应用页），毛玻璃提供层次感与通透感；柔感极简保证图标与文字在任何壁纸上都清晰可读。
- **DNA 参数**: 圆角 soft（`rounded-xl` ~ `rounded-2xl`）/ 阴影 layered（多层模糊叠加，毛玻璃配 `shadow-lg`）/ 间距 standard（`gap-4` / `p-4`）/ 字体方向圆润几何无衬线 / 装饰手法：脉冲光点、细描边图标、半透明分隔线。
- **应用类型**: Tool（系统模拟工具）—— 单屏手机外壳居中，内部为完整系统画布。

## 3. Color System

**色彩关系**: 青蓝脉冲主色 + 深墨系统底 + 半透明白玻璃卡片 + 低饱和青灰辅色；深色模式下反转通透感。
**配色设计理由**: primary 青蓝承担"Ping"脉冲、激活按钮、进度高亮与品牌识别；bg 在桌面为壁纸承载层，在应用内为纯白/纯黑内容底；accent 使用极浅灰蓝，承接 hover、选中态与毛玻璃高光，避免与主色抢戏。
**主色推导**: 从产品名 "Ping" 的网络脉冲语义出发，选取接近 iOS/Android 系统蓝但偏青的色调（hue 200），既符合手机系统的熟悉感，又通过"脉冲扩散"的动效语言建立独特识别。
**使用比例**: 65% 中性（壁纸/内容底 + 文字）/ 25% 辅助（毛玻璃、accent、border）/ 10% primary（脉冲光点、主按钮、激活态、关键高亮）。

| 角色 | CSS 变量 | Tailwind Class | HSL 值 | 设计说明 |
|---|---|---|---|---|
| bg | `--background` | `bg-background` | hsl(210 20% 98%) | 系统级背景，桌面为壁纸层，应用内为纸白内容底 |
| card | `--card` | `bg-card` | hsl(0 0% 100% / 0.85) | 毛玻璃卡片、设置列表、控制中心面板，带 backdrop-blur |
| text | `--foreground` | `text-foreground` | hsl(222 18% 12%) | 标题与正文，深色模式下为 hsl(210 20% 96%) |
| textMuted | `--muted-foreground` | `text-muted-foreground` | hsl(220 10% 45%) | 次要文字、说明、时间戳 |
| primary | `--primary` | `bg-primary` / `text-primary` | hsl(200 85% 55%) | Ping 脉冲主色，用于 CTA、激活开关、进度、品牌光点 |
| primaryForeground | `--primary-foreground` | `text-primary-foreground` | hsl(0 0% 100%) | primary 上的文字与图标 |
| accent | `--accent` | `bg-accent` | hsl(210 25% 94%) | hover/focus 浅底、选中项背景、Skeleton |
| accentForeground | `--accent-foreground` | `text-accent-foreground` | hsl(222 18% 22%) | accent 上的文字与图标 |
| border | `--border` | `border-border` | hsl(220 12% 88%) | 卡片边界、分隔线、输入框边框，偏通透 |

**语义色提示**: 
- 成功：hsl(150 55% 45%) —— bg: hsl(150 60% 95%) / border: hsl(150 45% 80%) / text: hsl(150 60% 30%)；饱和度与 primary 对齐（±10%）。
- 警告：hsl(38 90% 55%) —— bg: hsl(40 90% 95%) / border: hsl(38 80% 78%) / text: hsl(30 85% 35%)。
- 错误：hsl(0 75% 55%) —— bg: hsl(0 70% 96%) / border: hsl(0 65% 82%) / text: hsl(0 70% 38%)。
- 所有语义色饱和度控制在 55%-90% 区间，与 primary 的 85% 保持同一量级，避免突兀。

## 4. 字体与节奏

- **font-display**: Space Grotesk —— 几何感与科技感兼备，契合系统品牌名 PingUI 的脉冲科技语义，用于开机动画 logo、锁屏大时钟、应用名称。
- **font-body**: Noto Sans SC —— 中文系统界面的基础字体，清晰中性，小字号下仍保持可读性，用于设置项、应用内正文、控制中心标签。
- **字号**: 锁屏时钟 text-6xl ~ text-7xl（细字重）；应用标题 text-2xl；设置列表 text-base；状态栏 text-xs。
- **圆角**: 大（`rounded-xl` ~ `rounded-2xl`）—— 手机外壳、应用图标、控制中心卡片均使用大圆角，呼应现代手机的圆润美学。

## 5. 全局布局契约

- **Reference Layout Use**: 按需求结构推导。
- **Page / Section Order**: Boot → LockScreen → Home (StatusBar + Widgets + AppGrid + Dock) → Apps (Camera / Photos / Browser / Settings / Intro) → ControlCenter（下拉覆盖层）。
- **Standard Content Zone**: 手机画布固定比例 9:19.5，桌面端 `max-w-[390px]` + `h-[844px]` 居中显示外框；移动端自适应全屏（隐藏外框）。
- **Shell / Frame Alignment**: 手机外框为独立 chrome，内部系统界面全屏铺满，安全区内独立布局（状态栏高 44px，底部指示条高 34px）。
- **Padding & Rhythm**: 桌面内边距 `px-5 py-4`，应用内 `px-4 py-3`，小组件与图标网格间距 `gap-4`，保持 4px 倍数节奏。
- **Full-bleed Zones**: 壁纸、相机取景、开机动画、手电筒白屏全幅铺满手机画布，不受 padding 约束。
- **Local Narrowing**: 设置详情页、功能介绍正文在应用内进一步收窄至内容区，两侧留呼吸空间。
- **Overflow Strategy**: 设置列表、相册网格、书签列表使用纵向滚动；浏览器 iframe 独立滚动。
- **Flexibility Boundary**: 允许移动端隐藏手机外框、调整状态栏高度；不允许更改圆角体系、主色脉冲视觉与毛玻璃语言。

## 6. 视觉与动效

- **装饰**: 脉冲光点 + 细描边线性图标
- **阴影/边界**: 中 —— 手机外框有深度阴影（`shadow-2xl`），毛玻璃卡片有柔和投影（`shadow-lg`），应用图标有微浮起阴影（`shadow-md`）。
- **动效**: 精致跟手 —— 应用打开：图标位置缩放扩散 + 透明度淡入（300ms cubic-bezier(0.22, 1, 0.36, 1)）；控制中心：顶部下滑跟手，释放后弹性到位；图标点击：scale 0.92 按压反馈；Ping 脉冲：光点由中心向外扩散 2-3 圈后消散（开机、解锁、截屏成功时触发）。

## 7. 组件原则

- 所有系统级控件（开关、滑块、按钮、列表项）遵循 iOS 风格比例，但用 PingUI 青蓝主色替代系统蓝。
- 开关：关闭态灰底白圆，开启态 primary 填充 + 微弹滑动动画。
- 滑块：轨道为半透明白/灰，填充部分为 primary，滑块圆点带阴影。
- 毛玻璃面板：统一 `backdrop-blur-xl` + `bg-white/70`（深色模式 `bg-black/60`）+ 1px 内描边高光。
- 空状态与加载态延续脉冲光点视觉，用青蓝光点呼吸动画替代默认 spinner。

## 8. Image Direction

- **Image Role**: 桌面壁纸（多张可选，设置中切换）+ 功能介绍页插图
- **Image Art Direction**: 壁纸走"极简自然 + 微光"方向——晨曦山脉、雾湖倒影、夜城市天际线、极光星空；构图上下留呼吸空间，中部偏亮区用于衬托图标；色温与 primary 青蓝协调（冷调为主，1-2 张暖橙日落作为点缀）。功能介绍插图为扁平几何风，用青蓝 + 浅灰 + 白的配色，表现系统界面抽象示意。
- **Image Prompt Keywords**: minimal nature wallpaper, soft cyan blue gradient, misty mountain silhouette, dawn light, clean composition, 4k phone wallpaper, subtle glow, atmospheric perspective, urban night skyline, aurora borealis
- **Image Avoidance**: 避免高饱和艳色壁纸、复杂图案导致图标不可读、人物肖像壁纸、卡通插画壁纸、带有品牌 logo 的素材图。

## 9. Anti-patterns

- **Split personality**: 设置页一套风格、控制中心另一套风格、应用内再变一套；全站统一青蓝脉冲 + 毛玻璃 + 大圆角语言。
- **Glass everywhere**: 毛玻璃滥用导致层次混乱；仅控制中心、Dock、小组件、设置弹层使用毛玻璃，应用内容页保持纯色底保证可读性。
- **Default iOS drift**: 直接照搬 iOS 红绿黄配色与图标风格；用 PingUI 青蓝脉冲母题建立独立识别。
- **Animation bloat**: 每个交互都加过度动效导致卡顿；仅关键交互（开机、解锁、应用打开/关闭、控制中心下拉）使用精致动效，常规操作保持即时响应。
- **Mono-hue tyranny**: primary 同时用于按钮、开关、图标、边框、链接；仅 CTA 与激活态用 primary，其余交给 accent 与中性色。
- **Wallpaper clash**: 壁纸过于花哨导致桌面图标与文字不可读；所有备选壁纸需经过文字对比度测试，保证状态栏与图标清晰。