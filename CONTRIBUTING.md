<div align="center">

[![协议](https://img.shields.io/badge/协议-AGPL--3.0-blue)](LICENSE)
[![语言](https://img.shields.io/badge/语言-简体中文-blueviolet)](https://zh.wikipedia.org/wiki/中文)
[![PRs](https://img.shields.io/badge/PRs-welcome-brightgreen)](CONTRIBUTING.md)
[![AI 辅助生成](https://img.shields.io/badge/AI辅助生成-是-orange)](https://github.com/awujiana)

<h1>贡献指南</h1>

<strong>欢迎参与 uosc_subtitle_style 的共建</strong>

</div>

---

> [!WARNING]
> 本项目的相当一部分代码由 AI 辅助生成，并经过持续的人工作业、联调和重构，但它仍然可能存在遗漏、边界情况处理不足或行为与预期不完全一致的问题。如果你在使用过程中发现 Bug、兼容性问题、异常行为或文档缺失，欢迎积极提交 Issue。最好附上复现步骤、日志、截图和系统版本信息，这会非常有帮助。

感谢你关注 **uosc_subtitle_style** 并愿意贡献。本指南用于确保协作顺畅、行为一致、许可证统一。

> 维护者：awujiana ｜ 开源协议：AGPL-3.0

<a id="sec-toc"></a>
## 📚 目录

- [🤝 我能如何参与](#sec-1)
- [🐞 提交 Issue 前](#sec-2)
- [🔀 提交 Pull Request 前](#sec-3)
- [📋 代码与文档约定](#sec-4)
- [🤗 行为准则](#sec-5)
- [📄 许可证](#sec-6)

<a id="sec-1"></a>
## 🤝 我能如何参与

- **提交 Issue**：反馈 bug、兼容性问题、新功能建议。
- **改进脚本**：修复缺陷、增强功能、补充字体/配置候选。
- **完善文档**：修正 README / 架构文档 / conf 注释。
- **补充测试**：本项目极其欢迎带测试的贡献——单测与 E2E 都容易扩展。

<a id="sec-2"></a>
## 🐞 提交 Issue 前

1. 先搜索确认问题未被重复提出。
2. 说明环境：mpv 版本、操作系统、uosc 版本、字幕类型（外挂 ASS / 内封 / 图形字幕）。
3. 描述「期望行为 vs 实际行为」，能附最小复现步骤更好。
4. 若涉及「字幕卡住不跳」，请确认是否发生在内封字幕上（已知约束见 README「注意事项」节）。

<a id="sec-3"></a>
## 🔀 提交 Pull Request 前

1. 从 `main` 切出特性分支，命名如 `fix/pos-gap` 或 `feat/font-picker`。
2. **改完代码必须跑全部门禁**（判据 = 退出码 + 汇总行，不是 tail）：

   ```bash
   # 语法（glob 相对路径，须在 portable_config/ 或本仓库根下跑）
   ../luajit.exe tools/lua-syntax-check.lua
   # 单元测试（191 断言）
   ../luajit.exe tools/sub-style-test.lua
   # conf 静态门禁（按 mpv 解析语义复算：键集合/布尔/数值/字符串含 #）
   python tools/sub-style-conf-verify.py
   # 真机 E2E（83 断言；需要 mpv.exe 与装了 Pillow 的 Python）
   MPV_EXE=/path/to/mpv.exe python tools/sub-style-e2e.py
   ```

3. 新功能请同步补充：conf 项（带中文注释，**注释独立成行**——mpv 不支持行尾注释）、
   消息表（README）、测试用例。
4. 提交信息清晰说明「做了什么 / 为什么」。
5. 不要提交运行时数据文件（`*.json`）与临时复制的 `scripts/uosc/`（已被 `.gitignore` 排除）。

<a id="sec-4"></a>
## 📋 代码与文档约定

- 正文（README / 注释 / 文档）以简体中文为主，专有名词保留英文原文。
- README 遵循《我的 README 输出规范》：顶部徽章 + AI 辅助生成 `> [!WARNING]` 声明、目录、底部三节齐全。
- conf 注释一律独立成行；键名与脚本 `o` 表严格对齐（`sub-style-conf-verify.py` 会查）。
- Lua 局部量注意作用域从声明之后才开始（`local` 写在引用它的闭包之后会静默变全局）。
- 命令行、配置片段使用围栏代码块并标注语言；参数类信息优先用表格。
- 标题层级连续，不跳级。

<a id="sec-5"></a>
## 🤗 行为准则

请保持友善、专业、就事论事。尊重不同经验水平的贡献者，对设计分歧以讨论达成共识，
不人身攻击、不贬低他人工作。

<a id="sec-6"></a>
## 📄 许可证

向本仓库提交贡献，即表示你同意以 **AGPL-3.0** 发布你的贡献，并确认你拥有相应的授权。
详见仓库内 [LICENSE](LICENSE)。

[![AGPL-3.0](https://img.shields.io/badge/License-AGPL--3.0-blue.svg)](LICENSE)
