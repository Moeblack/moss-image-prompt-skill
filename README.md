# moss-image-prompt-skill

给 `MOSS-Image`（OpenMOSS，Qwen3.5-9B 文本编码器 + 5B 单流 DiT）编写图像提示词的规则集（写实向，日常着装与日常互动场景）。

[![协议](https://img.shields.io/badge/%E5%8D%8F%E8%AE%AE-CC%20BY--NC--ND%204.0-orange.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans) [![须署名](https://img.shields.io/badge/%E9%A1%BB%E7%BD%B2%E5%90%8D-Attribution-blue.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans) [![非商用](https://img.shields.io/badge/%E9%9D%9E%E5%95%86%E7%94%A8-NonCommercial-yellow.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans) [![禁止修改](https://img.shields.io/badge/%E7%A6%81%E6%AD%A2%E4%BF%AE%E6%94%B9-NoDerivatives-red.svg)](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans)

## 📜 授权条款（用之前请先读这一节）

本仓库全部文件（`SKILL.md`、`README.md`、`CHANGELOG.md`）依 **CC BY-NC-ND 4.0**
——**知识共享 署名-非商业性使用-禁止演绎 4.0 国际版**——提供。
中文官方摘要：<https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans>；法律全文：[`LICENSE`](LICENSE)。

| ✅ 你可以 | ❌ 你不可以 |
| :--- | :--- |
| 复制、原样转发、下载自用（含把规则接进自己的出图工作流跑图） | 用于商业目的：售卖、塞进付费产品、商用外包交付、广告物料产出 |
| 引用其中一两条规则，并署名出处 | 改写 / 翻译 / 精简 / 换题材后当作新版本发布（ND = 禁止演绎） |
| 提 Issue 反馈问题、请求改写许可 | 去掉署名与协议声明，或声称这是官方出品 |

**一句话**：拿去用没问题；别改着发，别拿去卖。
想改着用（本项目专用、换语言、精简条款）？开个 Issue 说一声，书面许可基本都批。

## 内容

| 文件 | 说明 |
| :--- | :--- |
| `SKILL.md` | 规则本体：§0 参数基线与三种次元模式 → §2 词条清洗 → §4 媒介与光影 → §5 结构分流 → §6/§7 人物与解剖 → §8–§10 专项协议 → §11 组装与自检 → §12 已证伪手法 → §16 二次元画风 → §17 闭合标签格式 |
| `LICENSE` | CC BY-NC-ND 4.0 法律全文 |
| `CHANGELOG.md` | 版本记录 |

## 用法

放进 Agent Skills 目录（目录名需等于 frontmatter 里的 `name`）：

```
<skills-dir>/moss-image-prompt-skill/SKILL.md
```

或直接把它贴进系统提示词末尾。产出固定两件：一对 Positive / Negative Prompt，外加 §0 的采样参数说明。

## 三种次元模式

- **默认｜三次元真人写实**：漫符转译、结尾锁真句、写实毛孔基底与负面池的 2D 封杀组全部生效。
- **二次元**：主体描述保持画风中立（不写毛孔 / 闪光灯 / 胶片词），并停用上述三处写实条款。
- **跨次元 2D/3D 混合**：不套用任何单次元基底。

## 证据分级

`SKILL.md` §1 定义四档：A（本地左右对照实测）/ B（单变量观察）/ C（官方文档或社区经验，未复现）/ ⚪（结论迁移到新载体，尚未在该载体上重新对照）。标 ⚪ 的条目按高置信起点假设使用。

## 出处与授权

规则来自 OpenMOSS 官方 User Guide 与技术报告，以及本地 A/B 出图实测。

Copyright © 2026 Moeblack. 授权条款见文首「📜 授权条款」一节；法律全文见 [`LICENSE`](LICENSE)，中文官方摘要见 <https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh-hans>。
