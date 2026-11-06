<div align="center">

[![协议](https://img.shields.io/badge/协议-AGPL--3.0-blue)](LICENSE)
[![语言](https://img.shields.io/badge/语言-简体中文-blueviolet)](https://zh.wikipedia.org/wiki/中文)
[![平台](https://img.shields.io/badge/平台-mpv%20%3E%3D%200.41-orange)](https://mpv.io)
[![作者](https://img.shields.io/badge/作者-awujiana-orange)](https://github.com/awujiana)

[![Release](https://img.shields.io/github/v/release/awujiana/uosc_subtitle_style)](https://github.com/awujiana/uosc_subtitle_style/releases)
[![Stars](https://img.shields.io/github/stars/awujiana/uosc_subtitle_style?style=flat-square)](https://github.com/awujiana/uosc_subtitle_style/stargazers)
[![Issues](https://img.shields.io/github/issues/awujiana/uosc_subtitle_style)](https://github.com/awujiana/uosc_subtitle_style/issues)

<h1>uosc_subtitle_style</h1>

<strong>mpv 内封字幕样式快速切换 —— 基于 uosc 菜单，主 / 次字幕（双语）分轨管理 · v1.0.11</strong>

</div>

---

> [!WARNING]
> 本项目的相当一部分代码由 AI 辅助生成，并经过持续的人工作业、联调和重构，但它仍然可能存在遗漏、边界情况处理不足或行为与预期不完全一致的问题。如果你在使用过程中发现 Bug、兼容性问题、异常行为或文档缺失，欢迎积极提交 Issue。最好附上复现步骤、日志、截图和系统版本信息，这会非常有帮助。

<a id="sec-toc"></a>
## 📚 目录

- [🚀 项目简介](#sec-1)
- [✨ 核心特色](#sec-2)
- [🆕 v1.0.6 – v1.0.11 更新要点](#sec-15)
- [🖼 展示图](#sec-3)
- [📦 安装 / 快速开始](#sec-4)
- [🎮 使用说明](#sec-5)
- [⚙ 配置项](#sec-6)
- [⚠ 注意事项](#sec-7)
- [🧪 测试](#sec-8)
- [📄 文档](#sec-9)
- [💬 反馈建议](#sec-10)
- [🙏 参考与致谢](#sec-11)
- [🔗 关联仓库](#sec-12)
- [📮 作者联系方式](#sec-13)
- [📄 开源协议](#sec-14)

---

<a id="sec-1"></a>
## 🚀 项目简介

**uosc_subtitle_style** 是一个 mpv Lua 脚本，让你在播放途中通过 uosc 菜单一键调整**内封（内嵌）字幕**的样式：位置、缩放、字体、字号、粗体、主色、描边粗细与描边颜色。不需要记命令行属性，不需要重载字幕，点几下菜单即可。

它要解决的真问题有三个：

1. **内封字幕改样式会卡住**。mpv 的 `sub-reload` 与解码器重建会让已消费的字幕包丢失——外挂字幕因为 `sub_preload` 全量重载而侥幸无恙，**内封字幕必然中招**（表现为「字幕定住一句不跳」）。本脚本的核心工程约束就是**绝不轻易重建解码器**：位置/缩放走属性即时生效；字体/字号/描边在已是 `force` 时只写属性 + 一次 OSD 微扰；只有进出 `force` 的瞬间才允许重建一次。
2. **双语字幕（双轨）没有分轨样式通道**。mpv 的字体系列属性是单份值，主/次解码器共享。本脚本采用「编辑目标」制：菜单顶部切换主字幕/次字幕，样式区只作用于目标轨，另一轨恢复原样（各自的记忆保留，切回即恢复）。位置与缩放则是两轨完全独立的属性，并带**自动避让**（两轨至少相差一档，不重叠）。
3. **预设改色容易误伤原版字幕**。载入时可自动套用预设样式（白字/标准描边），但仅当字幕本身是彩色的才动；**双语字幕（双轨）默认保持原样不改颜色**——两条都是可配置、可热切换的。

<a id="sec-2"></a>
## ✨ 核心特色

| 功能 | 说明 |
| --- | --- |
| 位置一键切换 | 原位 / 上移 / 中下 / 中部 / 中上 / 顶部 六档预设，另支持 ±1% 微调（v1.0.11 由 5% 改 1%）与整体缩放 |
| 主 / 次字幕分轨 | 菜单顶部切换编辑目标；位置/缩放/字体/字号/描边分轨生效，另一轨恢复原样 |
| 位置自动避让 | 两轨 `sub-pos` 至少相差 `pos_min_gap`（默认 25），目标轨设新位置时另一轨自动让位 |
| 字体选择器 | 枚举本机全部 sfnt/TTC 字体（含 CJK 标记），懒加载 + 签名缓存，支持重新扫描 |
| 样式覆盖 | 字体、字号（720p 等效像素）、粗体、主色、描边粗细、描边颜色 |
| 预设样式 | 载入时自动套用；仅彩色字幕生效（主色或描边任一为彩即算）、双语（双轨）默认跳过；可改可存可恢复出厂 |
| 彩色预设不串台 | 自动套用预设只对彩色字幕生效；切到黑白字幕自动恢复该字幕原样，不继承上个彩色视频的预设（内封/外挂均适用） |
| 中日双语布局 | 双轨按语言识别为一中一日时，日文轨自动置底 + 缩小（默认 ×0.8）、中文轨在其上；带 `\pos` 的注释幕样式/位置不变 |
| 样式检测报告 | 载入后自动解析外挂 ASS / 内封字幕配色，左上角报告「已套用 / 保持原样 / 双语跳过」及原因 |
| `\pos` 内封 ASS 也能改 | 字体 / 主色 / 描边色写进 `sub-ass-style-overrides` 改写 ASS 样式表本身；缩放与字号折成 `Fontsize` |
| 字号 ±1px 微调 | 连续点击精准加减字号，边界有提示；黑边贴合时的放大值自动存进全局预设 |
| 上下黑边贴合 | 检测视频**内嵌**黑条，主 / 次对白分别放进下 / 上黑边，按较薄一侧自适应字号，不遮挡画面 |
| 全内封轨检查 | 逐轨检测所有内封字幕（示例 MKV 有 42 条）；切到未检查过的轨自动重检；非标准配色一律套黑白预设 |
| 对外 API | `user-data` 状态广播 + 请求-响应 + 白名单动作转发，供其它脚本 / `input.conf` 集成 |
| 状态持久化 | 当前样式跨文件、跨重启记忆（`persist=yes`，数据文件在 mpv 配置目录） |
| 无 uosc 降级 | 未检测到 uosc 时菜单降级为位置循环切换 + OSD 文本列表 |

<a id="sec-15"></a>
## 🆕 v1.0.6 – v1.0.11 更新要点

| 版本 | 关键变更 |
| --- | --- |
| v1.0.6 | 修复颜色命令里的裸 `#RRGGBB` 被 mpv 当注释吞掉；`sub-ass-style-overrides` 让带 `\pos` 的顶部字幕 / signs 也能换字体转黑白；识别灰色字幕 `#8B8B8B`；主色候选加入黑色与灰阶；菜单点击后保持打开并刷新选中态 |
| v1.0.7 | 上下**内嵌黑边**贴合，主 / 次对白分别进下 / 上黑边；`sub-margin-y` 取整并在关闭 / 切文件时显式还原 |
| v1.0.8 | **所有内封字幕轨都要被检查**：`sid` / `secondary-sid` 切到未检查轨自动重检；内封除标准白字黑边外一律套用黑白预设，不再依赖窄颜色判据 |
| v1.0.9 | 修复「下移 / 整体缩放无效」：真机证实带 `\pos` 的内封 ASS 对 `sub-pos` / `sub-scale` / `sub-font-size` 全部豁免，缩放与字号改走 `Fontsize`；新增 ±1px 字号微调并存入全局预设；纯 Python 生成最小 Matroska 夹具做真机回归 |
| v1.0.10 | 上下黑边双对话字幕字号自适应判据（较薄黑边决定倍率、上下同倍率、不溢出）；新增对外 API v1.0.0（状态广播 / 请求-响应 / 白名单转发）；单测扩到 328 条 |
| v1.0.11 | 修复「内封字幕点下移没反应 / 对话轨道字幕没进黑边」：真机证实 `\pos` explicit ASS 唯一能改 Y 的通道是覆盖 `[Script Info]` 的 `PlayResY`，据此把微调与黑边贴合折成 `ratio` 写进 `PlayResY` 并用 `Fontsize` 反补字号；步长 5% → 1%；新增可关闭的左上角报告（`show_report` / `report-toggle`）；API 快照补 7 个字段、白名单补 2 个动作；单测扩到 373 条 |

> 若你的 mpv 仍显示旧的「非彩色字幕，保持原样」文案，说明内存里跑的是被自更新器降级过的 v1.0.5 —— **完全退出并重启播放器**。自更新器以文件头 `-- @version:` 判定本地版本，同版本不更新、本地更新时禁止降级。

<a id="sec-3"></a>
## 🖼 展示图

![uosc 字幕样式菜单](https://img.meituan.net/content/50ff6c2b3c5bbe2b66cc363ac948bc85356475.png)

![位置预设](https://img.meituan.net/content/1e35c10e718a0c79f640fa255e3b1115376117.png)

![字体选择器](https://img.meituan.net/content/736a4a418027744a4cf2f8e23b2f233f429096.png)

![颜色与描边](https://img.meituan.net/content/7856c0c121fb14cee2286b83d5d1772d417958.png)

![中日双语布局：中文在上、日文置底缩小](https://img.meituan.net/content/049ea738320986af80c11bcc7e461259322402.png)

---

<a id="sec-4"></a>
## 📦 安装 / 快速开始

<details open>
<summary><strong>方式一：直接拖文件（推荐）</strong></summary>

把本仓库 `scripts/` 文件夹里的 `uosc_subtitle_style.lua`，拖进 mpv 配置目录的 `scripts/` 文件夹：

```text
portable_config/                  ← mpv 配置目录（便携版在 mpv.exe 旁；
├── scripts/                         标准版在 ~/.config/mpv/）
│   └── uosc_subtitle_style.lua  ← 拖到这里
└── script-opts/
    └── uosc_subtitle_style.conf ← 可选：不放则全部用内置默认值
```

> [!IMPORTANT]
> 拖的是 `scripts/` **里面的那个 `.lua` 文件**，不要把整个 `scripts` 文件夹再套一层
> （`scripts/scripts/` 这种嵌套 mpv 不会加载）。

</details>

<details>
<summary><strong>方式二：从 Releases 下载</strong></summary>

前往 [Releases](https://github.com/awujiana/uosc_subtitle_style/releases) 下载最新版的两个**直接文件**（不打包 zip）：
`uosc_subtitle_style.lua` 放进 `scripts/`、`uosc_subtitle_style.conf` 放进 `script-opts/`。
`version.json` 是同一版本的机器可读清单，供脚本化安装与版本比对使用。

</details>

### 绑定按键（可选）

在 `input.conf` 里加一行，即可一键呼出字幕样式菜单：

```conf
ALT+Q script-message-to uosc_subtitle_style open-menu    # 字幕样式菜单
```

### 运行依赖

- **mpv ≥ 0.41**（用到 `secondary-sub-pos` 等分轨属性，低版本无此属性）。
- **uosc**（[tomasklaen/uosc](https://github.com/tomasklaen/uosc)）：菜单由它渲染。
  没装 uosc 也能用——自动降级为位置循环切换。

<a id="sec-5"></a>
## 🎮 使用说明

### uosc 菜单

呼出菜单后，顶部是**编辑目标**（主字幕 / 次字幕），下面是位置预设、缩放、字体、字号、
粗体、主色、描边、预设样式等区。所有修改即时生效并写入记忆；次字幕加载与否自动决定
「次字幕」项是否可点。

### 命令消息（script-message）

所有功能都可以用 `script-message-to uosc_subtitle_style <消息> [参数]` 驱动，
方便自绑按键或接入其它菜单：

| 消息 | 作用 |
| --- | --- |
| `open-menu` | 打开 uosc 菜单（无 uosc 时降级循环切换） |
| `cycle` | 循环切换目标轨的位置预设 |
| `target <main\|sec>` | 切换编辑目标轨道 |
| `pos <0-150>` | 设置目标轨 `sub-pos` |
| `scale <倍率>` | 设置目标轨 `sub-scale` |
| `nudge <±%>` | 以当前值为基准微调目标轨 `sub-pos` |
| `font <字体名\|reset>` | 字体覆盖（字体名含空格也不用引号，参数自动拼回） |
| `size <像素值\|reset>` | 字号覆盖（720p 等效像素） |
| `bold <on\|off\|reset>` | 粗体覆盖 |
| `color <#RRGGBB\|reset>` | 主颜色（`sub-color`） |
| `outline <粗细\|reset>` | 描边粗细（`sub-outline-size`） |
| `outline-color <#RRGGBB\|reset>` | 描边颜色（`sub-outline-color`） |
| `style-preset <big-bold\|original\|sync-other>` | 快捷预设 |
| `open-fonts` / `rescan-fonts` | 本机字体选择器 / 重新扫描 |
| `preset-apply` / `preset-save` / `preset-reset` | 套用 / 保存当前样式为预设 / 恢复出厂预设 |
| `preset-toggle-colored` | 切换「仅彩色字幕才套用预设」 |
| `preset-toggle-dual` | 切换「双语字幕（双轨）时保持原样」 |
| `preset-toggle-enabled` | 预设总开关 |
| `toggle-gray` / `toggle-all` | 灰色字幕也转黑白 / 全部字幕统一样式（共享样式表覆盖）开关 |
| `size-step <±px>` | 字号 ±1px 连续微调；黑边贴合时自动存进全局预设 |
| `ass-fontsize-toggle` | 内封定位 ASS 的 `Fontsize` 缩放 / 字号通道开关 |
| `letterbox-toggle` / `letterbox-refresh` | 上下黑边贴合开关 / 重新检测黑边 |
| `api <reply_to> <request_id> <action>` | 对外 API 请求-响应，见 [契约文档](docs/api-uosc-subtitle-style.md) |
| `inspect` | 重新运行样式检测并出报告 |
| `reset` | 恢复默认并清除记忆 |
| `fix-override` | 把目标轨 `sub-ass-override` 设为 `scale`（位置/样式的前置条件） |

> `api` 只做只读快照 + 白名单动作转发，**不会**新增字幕属性写入路径；旧入口参数与副作用原样保留。
> `user-data` 是 mpv node 属性，订阅必须用 `observe_property(..., 'native')` / `get_property_native()`。

### 数据文件

运行时在 mpv 配置目录（`~~/`）写三个 JSON，删除即重置：

| 文件 | 内容 |
| --- | --- |
| `uosc-subtitle-style-state.json` | 当前各轨样式记忆（位置/缩放/字体/字号/粗体/颜色/描边） |
| `uosc-subtitle-style-preset.json` | 预设样式（颜色/描边/字体/字号/粗体/开关） |
| `uosc-subtitle-style-fonts.json` | 本机字体枚举缓存（按目录签名失效） |

### 在 uosc 控制栏加按钮（uosc.conf）

uosc 的播放控制栏（`controls`）是在它**自己的**配置文件 `script-opts/uosc.conf` 里定义的
（不是本脚本的 `uosc_subtitle_style.conf`）。本脚本的所有功能都通过
`script-message-to uosc_subtitle_style <消息>` 暴露（见上方「命令消息」表），所以只要在
uosc 的 `controls` 里加一个 `command:` 按钮、把点击动作指向这条消息，就能在控制栏上多一个
一键触发本脚本的按钮。

#### 语法

`controls` 是一串**逗号分隔**的条目，每个条目可选带可见性条件前缀 `<条件>`，整体格式：

```text
<可见条件>command:<图标名>:<点击时执行的 mpv 命令>?<悬停提示>
```

- **可见条件（可选）**：`<has_sub>` 仅当有字幕时显示；常用还有 `<video>` / `<audio>` /
  `<idle>` / `<stream>`，取反加 `!`（如 `<!idle>` 表示非空闲时）。多个条件用 `,` 或
  ` *+ *` 组合成「或 / 与」组（例如 `<video,audio>`、`<audio+!video>`）。
- **kind**：`command:` 表示「点击执行一条 mpv 命令」；此外还有 `button:`（uosc 内置按钮名，
  如 `subtitles` / `fullscreen`）、`script-binding:`（脚本绑定）、`cycle:`（状态切换按钮）、
  `space` / `gap`（分隔）。本脚本用 `command:` 最合适。
- **图标名**：Material Symbols 图标名，例如 `tune`（调节）、`format_color_text`（文字颜色）、
  `text_fields`（字体）、`font_download`（字体下载）、`refresh`（重新扫描）。找不到的图标名
  会被渲染成文字，换一个即可。
- **命令**：填 `script-message-to uosc_subtitle_style <消息>`，`<消息>` 取上方表格任意一项。
- **悬停提示**：`?` 后面是鼠标悬停显示的文字（中文直接写，无需引号）。

#### 在哪改

打开 mpv 配置目录下的 `script-opts/uosc.conf`，找到里面那一长行 `controls=...`，
把新条目**用逗号**拼进现有列表即可（建议放在靠近 `subtitles` 的位置）。

#### 示例一：加一个「字幕样式」按钮（呼出菜单）

```conf
<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式
```
[![fbba665ca97f2816.png](https://img.meituan.net/content/4290cf0f128fad52ebe9db2cecdc99a48169.png)](https://img.meituan.net/content/4290cf0f128fad52ebe9db2cecdc99a48169.png)
- `<has_sub>`：只有加载了字幕（内封或外挂）才显示这个按钮；
- `tune`：用「调节」图标；
- 点击执行 `script-message-to uosc_subtitle_style open-menu`，即呼出本脚本菜单；
- `?字幕样式`：鼠标悬停提示。

拼到 `controls=` 里长这样（片段示意）：

```conf
controls=...,<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式,<has_sub>subtitles,...
```

#### 示例二：加多个按钮（菜单 / 字体选择器 / 重新扫描字体）

```conf
<has_sub>command:tune:script-message-to uosc_subtitle_style open-menu?字幕样式,
<has_sub>command:font_download:script-message-to uosc_subtitle_style open-fonts?字体选择器,
<has_sub>command:refresh:script-message-to uosc_subtitle_style rescan-fonts?重新扫描字体
```

（三个条目用逗号连写；`<has_sub>` 让它们在无字幕时自动隐藏。）

#### 示例三：状态切换按钮（cycle，进阶）

若希望按钮能反映「预设开关」的开 / 关并随状态换图标，需要本脚本先注册一个可被 uosc 观察的
属性。本脚本当前**未**注册此类属性，所以最稳妥是用上面的 `command:` 按钮触发
`preset-toggle-enabled`（点击即切换，但图标不随状态变化）。等脚本注册了属性后，可用
`cycle:` 语法（4 个参数：kind、观察属性、点击命令、状态组）：

```conf
<has_sub>cycle:prop_name:script-message-to uosc_subtitle_style preset-toggle-enabled:on=toggle_on/off=toggle_off?预设开关
```

> [!TIP]
> - 改完保存 `uosc.conf` 后**重启 mpv** 才生效（uosc 不热加载 `controls` 配置）。
> - 图标名清单见 [uosc 仓库](https://github.com/tomasklaen/uosc)（Material Symbols）；
>   图标名写错只会显示成文字，换一个正确的即可，不影响功能。
> - 不想要按钮了，把对应条目从 `controls=` 里删掉即可。

<a id="sec-6"></a>
## ⚙ 配置项

全部配置在 `script-opts/uosc_subtitle_style.conf`（该文件带中文注释，47 项与脚本默认值一一对应，
由 `tools/sub-style-conf-verify.py` 强制校验键集合一致）。不放此文件 = 全部使用默认值。常用几项：

| 配置 | 默认 | 说明 |
| --- | --- | --- |
| `persist` | `yes` | 跨文件/重启记忆当前样式 |
| `pos_default` / `sec_pos_default` | `100` / `0` | 主/次字幕默认位置（mpv 原生默认值） |
| `pos_min_gap` | `25` | 两轨位置避让的最小间距（% 屏幕高度） |
| `dual_layout` | `yes` | 中日双语(双轨)自动布局：按语言识别，日文轨置底+缩小、中文轨其上；注释幕(`\pos`)不变 |
| `dual_layout_scale` | `0.8` | 双语布局中日文轨的缩放倍率（视觉上日文小于中文） |
| `nudge` | `5` | 菜单微调步长 |
| `fonts` | Noto CJK 四款 | 字体候选列表（逗号分隔） |
| `sizes` / `colors` / `outlines` / `outline_colors` | 见 conf | 各属性候选（`标签=值` 或裸值） |
| `preset_enabled` | `yes` | 载入时自动套用预设总开关 |
| `preset_only_colored` | `yes` | 仅当字幕本身是彩色时才套用 |
| `preset_skip_dual` | `yes` | **双语字幕（双轨）默认不套用、不改颜色** |
| `preset_color` / `preset_outline` / `preset_outline_color` | `#FFFFFF` / `2` / `#000000` | 预设主色/描边粗细/描边色 |
| `preset_font` / `preset_size` / `preset_bold` | 空 / `0` / `keep` | 预设字体/字号/粗体（空或 `keep` = 不动） |
| `colored_threshold` | `40` | 判定「彩色字幕」的最小饱和度（max-min(RGB)，主色与描边色分别判定，任一达标即算） |
| `colored_outline_min` | `0.005` | 内封像素采样：白字+**细**彩描边时，非主色高饱和像素的占比下限（彩描边常 <2%，故单用此更低阈值判彩色） |
| `inspect_delay` / `report_ms` | `1.2` / `6000` | 载入后检测等待（秒）/ 报告显示时长（毫秒） |
| `inspect_retries` / `inspect_retry_delay` | `15` / `1.0` | 检测重试次数 / 间隔：字幕未及时加载或片头无对白时，等第一句对白出现再检 |
| `rebuild_delay` | `0.12` | override 切换的延迟设置（秒） |
| `fallback_cycle` | `yes` | 无 uosc 时降级为循环切换 |

v1.0.7 – v1.0.11 新增的键（完整清单与注释见 conf 文件）：

| 配置 | 默认 | 说明 |
| --- | --- | --- |
| `letterbox_enabled` / `letterbox_*` | 见 conf | 检测视频**内嵌**上下黑边并把主 / 次对白分别放进下 / 上黑边；含暗场守卫、扫描范围、留白比例、限缩上限 |
| `inspect_all_internal` / `preset_all_internal` | `yes` / `yes` | 逐轨检查所有内封字幕；内封除标准白字黑边外一律套黑白预设 |
| `ass_fontsize` / `ass_fontsize_ratio` | `yes` / 见 conf | 把缩放与字号折成 ASS `Fontsize` 写进 `sub-ass-style-overrides`，让带 `\pos` 的内封字幕真正可缩放 |
| `size_step` / `size_min` / `size_max` / `size_step_saves_preset` | `1` / 见 conf / 见 conf / `yes` | 字号 ±1px 微调的步长、边界，以及黑边贴合时是否把放大值存进全局预设 |
| `ass_pos_shift` / `ass_anchor_ratio` | `yes` / `0.94` | v1.0.11：用 `PlayResY` 覆盖让 `\pos` 内封对白可下移并落进黑边；锚点比例用于反解黑边目标 |
| `show_report` | `yes` | v1.0.11：左上角检测报告 OSD 总开关；设 `no` 完全不输出、不改 `osd-align-*`，但检测 / 黑白预设 / 黑边贴合照常执行 |

> [!NOTE]
> mpv 的 conf **不支持行尾注释**（`key=value  # 注释` 会把整段当值），所以仓库里的
> conf 注释全部独立成行；自己改的时候请保持这个写法。

<a id="sec-7"></a>
## ⚠ 注意事项

- **mpv ≥ 0.41**：依赖 `secondary-sub-pos` / `secondary-sub-scale` 等分轨属性。
- **ASS 字幕**功能完整；**图形字幕（PGS/VOBSUB）**不走 libass，位置/样式均不可调。
- **外挂 ASS 与内封字幕**都支持；样式检测对外挂解析 `[V4+ Styles]`，内封用截图差分。
- 字体/字号/描边的修改只对**目标轨**生效；**粗体是解码器级共享**（mpv 无分轨粗体通道，菜单已注明）。
- 带 `\pos` 的 signs/特效幕属于 libass explicit 事件，**位置**不受 `sub-pos` / `sub-scale` / `sub-margin-y` 影响；字体 / 主色 / 描边色 / 字号靠改写 ASS 样式表生效（v1.0.6 / v1.0.9）。**v1.0.11 起**改由覆盖 `[Script Info]` 的 `PlayResY` 做整体比例位移，1% 微调与黑边贴合同步作用于 `\pos` 字幕，`Fontsize` 同比反补保持物理字号；关掉 `ass_pos_shift` 才回到「保持原位」。⚠ 该通道解码器级共享、以画面顶部为原点，次轨 `\pos` 无法同时进上黑边；**禁止** `strip`（丢定位标签且让 style-overrides 整体失效）。
- 视频**内嵌**上下黑边 ≠ 播放器窗口 letterbox：mpv 的 `sub-use-margins` / `sub-ass-force-margins` 只管后者，本脚本的 `letterbox_*` 用逐行亮度采样处理前者。
- `sub-margin-y` 是 INT 属性且主 / 次轨共享，脚本会取整并在关闭贴合 / 切文件时显式还原。
- Windows 实测（mpv 0.41）；Linux/macOS 的字体枚举路径已内置，理论可用但未经真机验证。
- 首次打开字体选择器会有一次全盘字体冷枚举（约 1-3 秒），之后走签名缓存。

<a id="sec-8"></a>
## 🧪 测试

仓库自带完整的单测与真机端到端测试（改代码后请先跑门禁，判据是退出码与汇总行，不要只看 tail）。
所有命令都在**仓库根目录**执行；`luajit.exe` 可用任意 LuaJIT 5.1 可执行文件替代：

```bash
# 1. Lua 语法门禁（递归检查全部 .lua）
luajit tools/lua-syntax-check.lua

# 2. 单元测试（373 条断言，stub 掉 mpv API，任意目录可跑）
luajit tools/sub-style-test.lua

# 3. conf 静态门禁（按 mpv 解析语义复算：键集合 / 布尔 / 数值 / 字符串含 # ）
python tools/sub-style-conf-verify.py

# 4. 真机 E2E：分轨 / 状态 / 重置 / 字体选择器 / 字幕流连续性 / 预设六用例 /
#    截图视觉判据 / 灰字与 \pos 顶部字幕 / 黑边贴合 / 最小 MKV 夹具的 Fontsize
#    通道 / 对外 API 探针 / 5g \pos 对白进黑边 / 5h show_report 开关两轮对照
#    需要 mpv.exe 与装了 Pillow 的 Python（本机为 C:\Python314\python.exe）
MPV_EXE=/path/to/mpv.exe python tools/sub-style-e2e.py
```

E2E 会自动识别两种布局：

- **生产整合包**（仓库上一级目录有 `mpv.exe`）：直接复用现有 `portable_config` 与 uosc，行为与旧版一致。
- **独立仓库**（本仓库）：读 `MPV_EXE` → PATH，自动追加 `--config-dir=<仓库根>` 与 `--idle=no` 隔离用户全局配置；uosc 从 `UOSC_MAIN` 或 `MPV_EXE` 旁的 `portable_config/scripts/uosc/main.lua` 复制测试副本到 `scripts/uosc/`；该目录已被 `.gitignore` 排除，保留供后续 E2E 复用，不会进入提交或源码分发。找不到 uosc 时字体选择器三项记 SKIP 并打印原因，不影响其余断言。
- `uosc_staff_credits` 是整合包里的另一份脚本；本仓库没有它时，API 探针自动跳过那 7 条职员表断言（`SUBSTYLE_STAFF_API=0`），**不伪造 PASS**，期望表同步收缩。因此本仓库实测为 **226 条 PASS**（生产整合包 233 条），API 探针 14 / 14。

关键夹具 / 探针：`tools/sub-style-mkv-fixture.py` 用纯 Python 写最小 Matroska（MJPEG 单帧 + 内封 ASS `\pos` 事件），
不依赖 ffmpeg 也不依赖 4GB 示例视频；`tools/sub-style-probe-{pos,bars,fontsize,api,shift}.lua` 分别覆盖定位字幕、黑边、字号通道、API 与 v1.0.11 位移 / 报告开关。

<a id="sec-9"></a>
## 📄 文档

- [架构文档](docs/uosc-subtitle-style-architecture.md) —— 数据模型、mpv 能力矩阵、应用引擎、真机实证结论（§19 为 v1.0.10 API 与黑边判据，§20 为 v1.0.11 位移通道 / 黑边反解 / 报告开关）。
- [对外 API 契约](docs/api-uosc-subtitle-style.md) —— 状态字段表、请求-响应协议、动作白名单、错误码与集成示例。
- [贡献指南](CONTRIBUTING.md) —— 门禁清单与代码 / 文档约定。
- [更新日志](CHANGELOG.md) —— 各版本变更记录。
- [维护约定](AGENTS.md) —— 项目硬约束与改动前检查。
- [Release Notes](https://github.com/awujiana/uosc_subtitle_style/releases) —— 各版本发布说明（见 Releases 页面）。

<a id="sec-10"></a>
## 💬 反馈建议

提交 Issue 时建议附上：

- 系统 / 运行环境版本（mpv 版本、操作系统、uosc 版本）
- 复现步骤
- 相关日志（mpv 输出，可用 `--msg-level=all=info`）
- 截图或录屏
- 出问题的具体对象（字幕文件 / 轨道 / 配置项）

问题反馈：[Issues](https://github.com/awujiana/uosc_subtitle_style/issues) ｜ [Discussions](https://github.com/awujiana/uosc_subtitle_style/discussions)

<a id="sec-11"></a>
## 🙏 参考与致谢

- 运行时依赖 [uosc](https://github.com/tomasklaen/uosc)（LGPL-3.0，**不随本仓库分发**，菜单经 `script-message` 与其交互）。
- 视觉测试判据依赖 [Pillow](https://python-pillow.org)（仅测试需要，运行时不依赖）。
- 文档风格参考《我的 README 风格》输出规范。

## 🏷 关联标签

<p align="center">
  <a href="https://github.com/topics/mpv"><img alt="mpv" src="https://img.shields.io/badge/mpv-1f6feb?style=flat-square"></a>
  <a href="https://github.com/topics/uosc"><img alt="uosc" src="https://img.shields.io/badge/uosc-1f6feb?style=flat-square"></a>
  <a href="https://github.com/topics/lua"><img alt="lua" src="https://img.shields.io/badge/lua-1f6feb?style=flat-square"></a>
  <a href="https://github.com/topics/subtitle"><img alt="subtitle" src="https://img.shields.io/badge/subtitle-1f6feb?style=flat-square"></a>
  <a href="https://github.com/topics/mpv-script"><img alt="mpv-script" src="https://img.shields.io/badge/mpv--script-1f6feb?style=flat-square"></a>
</p>

> 以上为本仓库的 GitHub Topics（关联标签）；可在仓库首页 Topics 区或 `gh repo edit awujiana/uosc_subtitle_style --add-topic "..."` 维护。

<a id="sec-12"></a>
## 🔗 关联仓库

- [awujiana/mpv-bingshan](https://github.com/awujiana/mpv-bingshan) — 冰山 MPV 配置整合包，本脚本的生产运行环境（开箱即用配置树）
- [awujiana/mpv-script-updater](https://github.com/awujiana/mpv-script-updater) — 公共 `updater.lua` 自更新模块与脚本更新规范（本脚本的文件头 `@version` 由它读取）
- [awujiana/uosc-staff-credits](https://github.com/awujiana/uosc-staff-credits) — 姊妹实现，同样提供对外 API v1.0.0
- [awujiana/uosc_danmaku](https://github.com/awujiana/uosc_danmaku) — 同生态的 uosc 弹幕插件
- [tomasklaen/uosc](https://github.com/tomasklaen/uosc) — 本插件依赖的 uosc UI 框架（第三方）

> 完整的脚本 / 组件仓库索引（表格形式）见 [mpv-bingshan 脚本仓库索引](https://github.com/awujiana/mpv-bingshan/blob/main/docs/script-repositories.md)。

<a id="sec-13"></a>
## 📮 作者联系方式

- GitHub：[@awujiana](https://github.com/awujiana)
- 邮箱：[bingshanlengtie@qq.com](mailto:bingshanlengtie@qq.com)
- 问题反馈：[Issues](https://github.com/awujiana/uosc_subtitle_style/issues) ｜ [Discussions](https://github.com/awujiana/uosc_subtitle_style/discussions)

<a id="sec-14"></a>
## 📄 开源协议

本项目基于 [AGPL-3.0](LICENSE) 开源。任何人使用、修改或分发本项目的代码，
若以网络服务器形式向用户提供交互服务，必须同样以 AGPL-3.0 公开其完整对应源代码。
