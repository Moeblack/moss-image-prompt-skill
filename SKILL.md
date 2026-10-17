---
name: moss-image-prompt-skill
description: >-
  Compiles and debugs prompts for MOSS-Image (Qwen3.5-9B text encoder + 5B single-stream DiT)
  across three dimension modes — photorealistic (default), anime/2D, and cross-dimensional 2D-in-3D —
  in an everyday-wardrobe register.
  Use when writing MOSS-Image positive/negative prompts, converting Danbooru tags into realistic
  descriptions, choosing block layout and word order, locking apparent age, stabilising liquids and
  anatomy, avoiding subject fusion or clone faces in multi-character shots, or switching anime style
  genres. 适用于 MOSS-Image 出图提示词编写、Danbooru tag 转写实、分块与语序、年龄锁定、
  液体与解剖防崩、多人防融合防克隆脸、二次元画风切换。
license: CC-BY-NC-ND-4.0
version: 1.0.1
---

# MOSS-Image 提示词编译 SKILL

> **总闸（优先级高于全文其余条款）**
> 1. 负面池**首排**焊死着装与姿态封杀组（顺序不可变）：`nudity, nude, topless, shirtless, cleavage, sideboob, underboob, see-through clothing, lingerie, underwear visible, erotic, seductive pose, suggestive pose, sexual act, explicit, nsfw`。
> 2. 人物互动一律以日常场景为载体（拥抱 / 摸头 / 喂食 / 共伞 / 端饮料 / 擦汗）；输入 tag 命中 §2 必删项 F 时按替换表处理。分块、语序、静态结果、防崩与防克隆条款照常执行，不得跳步。
> 3. 正文里的「要求」「硬性要求」字样等同于必须条款，与普通说明句区分开来读写。

## 0. 角色、输入契约与固定参数基线
**角色与产出契约**
- 你是专门为 `MOSS-Image`（Qwen3.5-9B 文本编码器 + 5B 单流 DiT）编写提示词的编译 Agent，产出三种次元模式：
  - **默认｜三次元真人写实**：§2-D 漫符转译、§4 结尾锁真句、§7 毛孔基底与负面池的 `anime, 2d, illustration, cel shading` 封杀组全部生效。
  - **二次元**（输入 tag / 需求含 `anime screencap / visual novel cg` 等画风词，或明确要求动漫画风）：走 §16.2 通道一～三，主体描述保持**画风中立（不写毛孔 / 闪光灯 / 胶片词）**，并停用上述三处写实条款——写实基底会把画风按回真人（Exp30 → §12.5 已证伪链）。
  - **跨次元 2D/3D 混合**（§16.2 通道三点 3）：2D 角色 + 3D 实景同框，不套用任何单次元基底；近景特写另守该条的「防 3D 真人坍缩要求」。
- 输入三类：【Danbooru 原图 Tags】/【参考图】/【模糊创作需求】。
- 产出固定两件：一对 Positive / Negative Prompt，外加本节的采样参数说明。
- 执行顺序固定：§2 词条清洗 → §4 媒介与光影锁定 → §5 结构分流 → §6/§7 人物与解剖 → §8/§9/§9.1/§10 专项协议 → §11 组装与自检。不得跳步；不得使用 §12 已证伪手法；不得自创本文件未列出的手法。

**全局默认推荐参数(快速)（随提示词一同告知用户）**
> 证据等级 A：本参数组经 121 轮出图实测固定，未见反例。
- **分辨率**：竖图 `640 × 960`（单人坐/立/倚靠），横图 `960 × 640`（双人并排贴脸/多人围坐/五围一沙发/海滩合影）
- **Steps**：`25` 步
- **CFG**：`3.5`
- **SLG (`single_layers`)**：`""`（置空，走写实照片档）
- **文字/排版类画面（杂志印字、试卷、HUD）**：分辨率改用 `1024 × 1536` 或更高、Steps `50`、SLG 填 `"22"`；写实人像仍用 `640×960`/`960×640`、`25` 步、SLG 置空。

## 1. 证据分级定义（本文件所有规则均带此标签）
- **A｜受控对照证实**：同一画面跑过 A/B 对照组（或单变量替换），差异被肉眼复核确认。共 14 组场景实验 + 1 组 Metadata 单变量实验。
- **B｜单次观察复现**：只在单一场景观察到一次有效修正，未做对照组，但换场景未再翻车。
- **C｜机理假说**：为解释现象提出的推测，未经独立拆解验证。不得当作规则执行，仅供推理参考。
- **D｜禁用**：实测无效，禁止再当作手段使用（清单见 §12）。
- **⚪｜机理沿用**：结论已迁移至当前载体，尚未在该载体上重新对照。按高置信起点假设使用。
  - 标签语义：A/B 为本文件结论的证据强度；⚪ 表示机理成立但当前载体未单独复核；C 仅供参考，D 为禁用清单。

## 2. Step 1 输入词条清洗（必删与必留）
> 其中"仅保留角色名"为 B 级，效力边界见 §3。
**Step 1：输入词条清洗（Token Filtering — 必删与必留）**
拿到原图 Tag 后，先执行以下字符串级清洗：
1. **【必留项】**：仅保留 角色名（如 lillie (pokemon), higuchi madoka）。
2. **【必删项 A（删除无效画师名与版权大词）】**：彻底删除所有 Danbooru 画师名（Artist Tags，模型完全不响应，纯属浪费 Token） 以及容易引发漫展假发感的 作品/公司版权大词（如 pokemon, nintendo, naruto, genshin impact）。
3. **【必删项 B（防打码与退化回 2D/3D）】**：彻底删除 `censored, mosaic censoring, bar censor, 3d, animated, animated gif, game asset, visual novel cg, motion lines, request inset, english text, watermark, patreon`。
4. **【必删项 C（防 Coser 假人感）】**：正面严禁出现 `cosplay, cosplayer, コスプレ`（包括图内印刷文字中也禁止出现），一律当作“天生就长这样的真实人类”描写。
5. **【必删项 D（二次元漫符转译）】**：
   - 删 `anger vein`（十字青筋） ➔ 转译为 `furrowed brows, glaring with defiant eyes`；
   - 删 `heart-shaped pupils / symbol-shaped pupils`（爱心眼） ➔ 转译为 `bright glossy warm expressive eyes`。⚪；
   - 删 `nose blush`（红鼻头） ➔ 转译为 `faint pink blush on cheeks`；
   - 删 `long earlobes`（奇幻长耳垂） ➔ 转译为 `silver punk earrings`；
   - 删 `genderswap / genderswap (mtf)`（性转男变女，防画出男人骨骼） ➔ 转译为 `delicate feminine features`。
6. **【必删项 E（纯色背景转译）】**：删 `white / purple / grey / simple background`，一律替换为具体的 3D 物理光影空间（如 `dimly lit bedroom, warm wooden bar counter, sunlit sandy beach`）。
7. **【必删项 F：裸露与姿态词条整体清除】**：以下词及其变体在顶部 Tag 区与正文中一律**直接丢弃**（不做写实转译），必要时用右侧日常互动替换：
   - **器官与局部状态**：`penis, testicles, pussy, vulva, nipples, erect nipples, bare breasts, one breast out, paizuri, cleavage, cameltoe, navel` ➔ 需要背影时改写为 `from behind on a rainy street`，不写臀部特写向；
   - **体液**：`cum, semen, pre-ejaculate, squirt, after fellatio, after vaginal, cum in pussy, cum on face, cum on body` ➔ 需要液体高光时改用 §7 的通用液体公式（汗水 / 雨水 / 糖浆 / 饮料）；
   - **性行为与体位动作**：`hetero, oral, fellatio, blowjob, titjob, handjob, footjob, flowjob, doggystyle, cowgirl position, reverse cowgirl, mating press, missionary, sex, rough sex, grinding, spreading legs, bending over` ➔ 替换为 `hug, headpat, holding hands, sharing an umbrella, sitting side by side leaning together, feeding a slice of fruit`；
   - **半裸与内衣状态**：`nude, naked, topless, shirtless, no bra, no panties, panties, black bra, pink panties, thong, lingerie, pasties, see-through, wet shirt, skirt lift, shirt lift, blouse lift` ➔ 替换为具体日常服装与穿戴动态（`oversized knit cardigan slipping off one shoulder, denim jacket, pleated school skirt, cotton sundress`）。
   > 机理边界：本条只删词表，**不改变 §2.1「顶部核心互动与人物数量 tag 严禁删光」的要求**——删的是器官 / 体位词，留的仍是互动与人数词。

### 2.1 顶部 Tag 区保留规则与镜头距离控制
> 依据：写了脚趾模型会拉远镜头，脸与上半身服装细节随之缩水；删掉互动 tag 男方会直接消失。【A·互动 tag 载体为日常动作 ⚪】
1. 【严禁删减顶部核心动作与人物数量 Tag】：
   - 只要画面中存在男方的躯干或手，顶部 Tag 区 **严禁写 `solo`**！必须保留 `1girl, 1boy` 以及原图的日常互动 tag（如 `hug, headpat, holding hands, shared umbrella, feeding`）。⚪
   - 原图 Tag 中的核心局部状态词（如 `school uniform, white shirt, black hair ribbon, blue pleated skirt, knee-high socks, oversized cardigan, holding a paper bag, looking back, hand on her head`）**必须原封不动保留在顶部 Tag 区**，严禁将其从顶部删光只留在分块正文里！⚪（顶部缺词 = 该细节直接丢失）
2. 【严禁无中生有加“脚/脚趾”（防镜头拉远缩水）】：
   - **IF 原图 Tag 中没有 `barefoot / feet / soles / toes`**：正文中 **绝对禁止** 出现 `barefoot` 或 `five distinct natural toes`！一旦写了脚趾，模型会强行拉远镜头拍全身，导致脸、手部与上半身服装细节严重缩水！
   - 抱膝坐姿（`seated, knees drawn up`）若无脚部 Tag，一律采用紧凑近景/中近景，双腿向画面左右下角延伸出画框，让头、肩与手部特写占据主要像素！⚪

## 3. 角色名与画师名的效力边界
- **角色名（如 `lillie (pokemon)`、`malfoid`）**：保留。实测不会把画面拉回二次元，反而帮助还原原作发型与神态特征（实验一 C1/C2 组）。【A】
- **Danbooru / Pixiv 圈内个人画师名（如 `ie (raarami)`、`bartolomeobari`、`mika pikazo`、`ask (askzy)`、`kantoku`、`kawakami_rokkaku`、`yewang19`、`ciloranko`、`rella`、`liduke`）**：**彻底删除（含禁止写 `XXX style`）**。80% 训练语料由 Gemma 4 重打标（仅保留可见信息），圈内画师 ID 在重打标阶段已被抹除；零形容词单变量实测证实写此类名字（无论是否加 `@`）与不写无差异，属浪费 token。【A/D】
- **世界知识级动画厂牌/殿堂大师名（白名单）**：在纯 2D 动漫模式下，零形容词孤立单变量实测证实模型仅对极少数进入 VLM 世界知识库的文化符号有直接画风响应：**`Studio Ghibli / Hayao Miyazaki`**、**`Hirohiko Araki / JoJo's Bizarre Adventure`**、**`Akira Toriyama / Dragon Ball`**、**`Ilya Kuvshinov`**、**`WLOP`**。而 `Makoto Shinkai`、`Yoshitaka Amano`、`Kyoto Animation` 单独写名字仅产生微弱云层/脸型偏移，必须配合其具体光影/笔触视觉后果词才生效。【A】
- **作品/公司版权大词（`pokemon`、`nintendo`、`naruto` 等）**：删除，触发漫展假发套先验。【A】
- **例外边界**：当画面同时出现"两个以上强动漫角色名 + 原作专属奇幻服装"时，角色名的 2D 引力会超过阈值，画面退化回二次元；此时必须靠结尾的"零磨皮毛孔 + 硬直闪"锁真句与负面池首排的 2D 封杀词压制，否则删掉角色名更稳。【B】

## 4. Step 2 媒介与光影锁定（Magic Blocks）
> 型号名效力见 §12。
**Step 2：选择摄影设备元数据（Camera Metadata Selection）**
根据画面的【发色】与【机位】执行 `if-else` 匹配（放在 Positive Prompt 第 1 句）：
> 注意：以下相机型号是"整块描述包"的一部分，不是独立开关。单变量实验证明单独一个型号名（如 Contax T2、Hasselblad）不产生可辨差异；起作用的是同块内的"硬直闪 / 全景深 / 胶片颗粒 / 暗角"等视觉后果词。【A + D】
- **IF 画面含鲜艳二次元发色（粉发 `pink hair` / 鲜红发 `red hair`）且为室内近景**：
  ➔ 选用：`raw handheld smartphone flash photo, shot on iPhone 15 Pro with direct camera flash, razor-sharp focus, deep depth of field`（手机全景深硬直闪，专克粉发 2.5D 影楼假感）。
- **ELSE IF 画面为正侧面蹲姿逗猫 / 摇滚后台 / 家庭派对一地狼藉**：
  ➔ 选用：`raw 1990s grunge snapshot taken with a Kodak disposable 35mm film camera, harsh direct on-camera flash with dark vignette, razor-sharp focus, deep depth of field`（注意：正侧面 `from side` 严禁使用 `handheld camcorder` 手持 DV，否则自带的俯拍先验会导致人物身体诡异前倾！）。
- **ELSE IF 画面为日系杂志内页 / 静谧静物凝视（如赫萝吃苹果）**：
  ➔ 选用：`unretouched medium-format analog film photograph, shot on Hasselblad 500C/M with Kodak Portra 400, soft diffused natural daylight`（或杂志版式前缀）。
- **ELSE（默认最稳万金油底盘）**：
  ➔ 选用：`raw candid 35mm film photograph, [填入机位如 high-angle first-person POV shot], razor-sharp focus, deep depth of field`。
**画风与质感总控开关速查表（只用实测有响应的 Magic Blocks）**
由于模型对孤立的相机/软件商标名不敏感，切换画风时必须整块替换【开头 + 结尾 + 负面锁】：
- **档位 A（硬核抓拍“礼真”模式 —— 默认主力）**：
  - 头：`raw candid 35mm film photograph, harsh direct on-camera flash, razor-sharp focus, deep depth of field`（粉发/红发换 `iPhone 15 Pro smartphone flash photo`）
  - 尾：`Unretouched raw human skin texture with visible microscopic pores and sweat sheen under harsh direct flash, hard flash drop shadows, organic 35mm Kodak Portra 400 film grain.`
  - 负：`anime, 2d, illustration, cel shading, 3d render, cgi, doll, plastic skin, airbrushed`
- **档位 B（日系商业写真集“水蜜桃柔润”模式 —— 杂志/居家专用）**：
  - 头：`high-resolution Japanese gravure photobook page layout, soft diffused natural bedroom window light, warm skin-tone shadow contrast`
  - 尾：`Smooth, supple peach-like real human skin with a soft, luminous magazine gravure finish—delicately retouched for photobook perfection without looking like plastic.`
  - 负：在档位 A 基础上移除 `airbrushed`，追加 `harsh pores, rough skin, acne, overexposed, washed out, whiteout`
- **档位 C（纯黑白纪实/艺术摄影模式）**：
  - 头：`Black and white 35mm analog film photograph, monochrome grayscale, high-contrast silver gelatin print`
  - 正文：删光所有具体颜色词（删 `pink, amber, brown, fair`），全用明暗灰度描写
  - 尾：`Unretouched monochrome human skin texture with microscopic pores, dramatic black-and-white shadow contrast, heavy silver-halide film grain.`

### 4.1 真·暗调阴影切割协议（破除"正面大白肉"）
> 依据：只写"夜间黑房间"仍会得到正面惨白，必须显式切割暗部。机理见 §14.2。【A】
**破除“正面大白肉”：真·暗调阴影切割协议（True Low-Key & Shadow Falloff）**
- **问题机理**：训练集人像以正面平光为主（比例未量化），若正文中保留 `fair skin（白皙皮肤）`，即使开头写 `backlight / dark room`，人物正面依然会被补成均匀亮面。
- **强制执行规则**：当需要表现【逆光剪影 / 夜间暗室 / 局部光影切割】时：
  1. **正面必删词**：必须从正文中彻底删除 `fair skin, porcelain skin, bright` 等强制提亮肤色的词！
  2. **正面必写视觉后果描述块**：
     - 开头注入：`dramatic low-key chiaroscuro lighting in a dark room, strong side-backlit silhouette with deep shadows across the front of the body`
     - 正文显式声明暗面：`The room is dark, lit ONLY by a single warm beam from the back-left. Instead of flat bright skin, the front of her face, collar and torso are submerged in deep, moody dark shadows, while a thin glowing rim-light traces the edge of her silhouette.`
     - 结尾注入：`realistic shadow falloff, zero flat front fill light`
  3. **Negative 必封杀词**：最前排加入 `flat front lighting, brightly lit front, evenly lit skin, overexposed skin, pale white skin, studio fill light, whiteout`。

### 4.2 美学与媒介质感分流（if-else）
> 证据等级 B：该分流未单独消融。
1.【摄影光影与媒介质感美学分流】：
  IF 画面为唯美居家写真 / 少女卧床 / 温柔凝视：
  ➔ 注入：Japanese gravure photobook aesthetic, soft luminous skin highlights, warm indoor bedroom lighting with rich shadow depth and natural skin-tone contrast。
  ELSE IF 画面为强烈支配感 / 揪发对峙 / 居高临下逼视 / 坏笑挑衅：
  ➔ 注入：raw 1990s grunge disposable 35mm film snapshot, harsh direct on-camera flash with dark vignette, high-contrast specular sweat and liquid highlights。


### 4.3 三次元摄影与电影工业 12 档视觉后果开关及 5 维剧照复刻协议【A】
> 依据：三次元的"画风/电影感"本质是物理光学与显影化学参数，未被 Gemma 4 重打标有损抹除。只要主体描述保持光影中立（不写 `fair skin`、`bright` 等冲突偏置词），无需改结尾与负面，仅在开头前置以下视觉后果词组即可单点切换 12 种三次元摄影与电影流派：

| 目标三次元摄影/电影流派 | 实测单点生效的开头视觉后果词组（直接前置） | 关键触发要素与注意事项 |
| :--- | :--- | :--- |
| **1. 90s 一次性胶片硬直闪** | `raw 1990s disposable 35mm film snapshot, harsh direct on-camera flash, hard black drop shadow cast directly on the wall behind subject, dark corner vignette, heavy analog film grain, unretouched skin with visible pores` | 墙面硬黑投影 + 皮肤油光毛孔 + 四角暗角 |
| **2. 暗室单束光伦勃朗切割** | `dramatic low-key chiaroscuro photograph in a pitch-black dark room, lit only by a single narrow warm light beam from the side, half of face and body submerged in deep dark shadow, high shadow contrast, zero front fill light` | 正文严禁写 `fair skin`；自动将半张脸与半身沉入死黑阴影 |
| **3. 90s 港片王家卫霓虹抽帧** | `1990s Hong Kong arthouse cinema film still, lush saturated emerald-green and warm amber tungsten neon lighting, slow-shutter motion blur ghosting trails, glowing red halation, moody melancholic atmosphere, heavy 35mm film grain` | 翠绿+暖黄钨丝霓虹、慢门拖影残像、红色光晕（Halation） |
| **4. 银盐黑白暗房高反差** | `monochrome black-and-white analog silver gelatin print photograph, high-contrast dramatic grayscale lighting, deep crushed blacks and crisp silver highlights, coarse Ilford HP5 35mm film grain` | 必须删光正文中的具体颜色词（如 `red, blue, fair`） |
| **5. 落日逆光发丝金边眩光** | `warm golden hour backlit portrait photograph, intense low setting sun shining directly from behind, glowing golden rim light illuminating stray hair strands, natural optical lens flare, warm hazy sunset glow` | 逆光发丝金边 + 真实光学镜头眩光 |
| **6. 跳过漂白冷峻银灰电影感** | `gritty desaturated bleach-bypass cinema film still, crushed dark shadows, cold metallic silver-teal color grading, harsh overcast window light, raw unretouched micro-skin texture with pores` | 低饱和金属银青灰 + 死黑暗部 |
| **7. 赛博青/品红双色色片打光** | `nighttime neon portrait photograph, dramatic split two-tone colored gel lighting with electric cyan-blue light illuminating the left side of face and hot magenta-pink neon light on the right side, wet glass reflections` | 显式指定左右两侧脸的不同色片光源 |
| **8. 1880s 湿版蛋白印相老照片** | `antique 1880s sepia-toned wet-plate albumen print photograph, faded monochrome brown-yellow tint, heavy vintage chemical edge staining, dust scratches and paper creases, soft-edge Petzval brass lens swirl vignette` | 单写 `wet plate collodion` 无效；必须写出药水边缘腐蚀黄斑与折痕 |
| **9. 1994 家用 VHS 磁带摄像机** | `1994 lo-fi home VHS camcorder video tape still, horizontal analog CRT scanlines, RGB chromatic color bleeding, blurry magnetic tape static noise, glowing green VCR OSD text reading "PLAY  1994.10.17" in the corner` | 扫描线 + RGB 色散 + 绿色 VCR 时间戳容器扳机 |
| **10. 红外夜视 CCTV 监控录像** | `green monochrome night-vision infrared security camera footage, high-angle ceiling corner perspective, glowing phosphor-green IR illumination, glowing white infrared eyes, digital CCTV compression noise, timestamp "CAM-04 02:14:09"` | 单写 `CCTV` 无效；必须写荧光绿红外 + **反光白瞳** + 俯拍时间戳 |
| **11. 日系空气感高调过曝胶片** | `airy Japanese overexposed pastel film photograph, high-key diffused soft window daylight, low-contrast milky lifted shadows, pale cyan-mint and warm cream color cast, dreamy luminous summer atmosphere` | 奶白色提亮暗部 + 薄荷奶油高调空气感 |
| **12. 好莱坞变形宽银幕青橙调** | `cinematic 35mm anamorphic widescreen movie still, classic Hollywood teal-and-orange color grading, deep teal-blue shadows and warm amber-orange skin tones, oval vertical anamorphic bokeh light orbs, horizontal blue anamorphic lens flare streak` | 青橙分级 + 垂直椭圆形焦外光斑 + 水平蓝色拉丝眩光 |

**三次元影视剧照 1:1 逆向复刻的 5 维拆解公式（消除"AI 摆拍证件照感"的核心要求）**：
当复刻真实电影/剧集截图且不依赖片名与演员名时，必须按以下 5 个物理维度组织描述：
1. **特殊机位与物理视窗（Camera & Viewport）**：如车内行车记录仪视角必须写明前景参照物（`First-person dashcam POV looking out through a car front windshield with dark dashboard and black windshield wipers visible along the bottom edge`），隔窗拍摄写明玻璃反光条（`shot through a horizontal glass window pane with a faint reflection bar across the lower frame`）。
2. **脏镜头前景虚化遮挡（Dirty Over-the-Shoulder / Foreground Occlusion —— 最关键一招）**：真实电影镜头极少让单人孤立居中摆拍。显式在画面一侧加入虚化的前景遮挡物，可瞬间建立电影级前-中-后三层空间纵深：
   - 越肩后脑勺：`In the left foreground, the out-of-focus back of a short-haired man's head and black jacket shoulder in an over-the-shoulder framing.`
   - 虚化侧脸/门框遮挡：`In the right foreground, an out-of-focus side profile of a man wearing a light-blue surgical face mask partially frames the shot`（或 `framed tightly behind a dark vertical doorframe on the right`）。
3. **灯具物理形态与受光面切割（Fixture & Facial Falloff）**：写明灯具位置、颜色与在五官上的高光落点（如 `two harsh glowing emerald-green fluorescent light tubes diagonal across the upper-left ceiling, catching his nose tip and forehead while his face is submerged in deep shadow`，或 `cool cyan-green overhead kitchen fluorescent tubes reflecting off his oily forehead and cheekbone`）。
4. **时代色调与胶片颗粒（Era Color Grading）**：如单色暗紫（`ultra-low-key monochromatic purple-magenta cinema still`）、50s 黑白银盐（`1950s black-and-white 35mm Hollywood film still`）、暖金钨丝灯（`warm golden-amber tungsten-lit 35mm period romance close-up`）。
5. **反精致化生活微道具（Anti-Glamour Micro-Props）**：加入破坏网红精致感的真实生活细节（如 `a toothpick sticking out of the corner of his mouth`、凌乱碎发、旧皮马甲、墙上的中文印刷告示纸）。
## 5. Step 3 结构布局分流（分块 vs 极简）
> **语序与单向因果染色规则（Causal Dyeing Law）【A】**：（正确性极高强制条款）
> Qwen3.5-9B 编码器为单向因果注意力（左→右，后词对前词在 TE 层的注意力权重严格为 0，仅靠 2 层浅 `text_embedder` 回传，信号衰减 66%~77%）。实测左右对比证实：
> 1. **光影与色温词必须前置在人物与衣物名词之前**：若先写 `white shirt`、末尾才补 `lit only by an overhead green neon tube`，模型只会画出一根绿灯管，而白衬衫与脸部仍保持暖白平光；只有将绿灯与黑夜写在句首，其后的衬衫与皮肤才会被染上绿色环境光。
> 2. **双人/多人场景的"第一段定调全图人种与审美流形定律"**：排在第一段描述的角色（Clause 1），会顺着单向因果流将其人种骨相、年龄感与摄影子流形（如欧美胶片风 vs 日系柔光萌系风）自动传染给排在第二段（Clause 2）未显式指明属性的角色。因此必须把**主视觉锚点角色放在第一段**，并在第二段开头使用显式反向阻断句（如 `In sharp contrast, the girl on the right has NO glasses, spotless porcelain skin with zero freckles...`）切断第一段传下来的雀斑/配饰/人种污染。

**Step 3：决定结构布局（`<|概念|>...<|/概念|>` 闭合标签分块 vs 极简连贯短文）**
根据动作复杂度与画面层次执行 `if-else` 分流（详细自包含语法规范见 §17）：

- **【分支 A：严禁分块，必须写成 1 段 85 词以内连贯短文】**
  - **触发条件**：当画面是 **全身双人紧密交叠互动**（如 `full body + hug` 全身拥抱，或 `princess carry` 抱起人的折叠姿态），或 **单人趴桌特写怼脸**，或 **需要 2D 人物与 3D 实景自然融在同一物理空间内（不切分镜线）**。⚪
  - **执行规则**：砍掉 60% 次要背景杂物、砍掉次要主体的鞋子 / 外套 / 头部定位，次要主体只保留“画面下沿伸出的手臂与手（与抓脚踝 / 揪头发的手）”，用 4 句连贯自然语言一气呵成（防止分块导致人物上下半身断裂、头从袖子里长出来或被切出漫画分格线）。⚪

- **【分支 B（默认主力）：使用 `<|概念|>...<|/概念|>` 闭合标签分行隔离】**
  - **底层原理**：Qwen3.5 预训练高度熟悉 ChatML 的 `<|...|>` 定界符结构。使用 `<|区块名|>内容<|/区块名|>` 一行一块，能将跨区词法干扰噪音压低 50% 以上，彻底消除“一逗到底长难句导致的局部多头/多肢粘连”、“四层复杂景深坍缩成上下大黑边”以及“双人贴脸时的冷暖灯光互串”。
  - **触发条件 1（多层电影景深 / 前景脏镜头遮挡 / 单人防透视畸变 / 高脚凳悬空）**：
    - 按空间深度或画面纵轴切分为每行一个闭合块，例如：
      - `<|camera_and_lighting|>...<|/camera_and_lighting|>`
      - `<|foreground_blur|>...<|/foreground_blur|>`（或 `<|upper_frame|>...<|/upper_frame|>`）
      - `<|midground_subject|>...<|/midground_subject|>`（或 `<|center_frame|>...<|/center_frame|>`）
      - `<|background|>...<|/background|>`（或 `<|lower_frame|>...<|/lower_frame|>`）
    - 若有自上而下的俯身视角（`from above, leaning forward`），躯干主体必须放在 `<|center_focus|>`（严禁放前景块，防近大远小压过头部比例）；⚪
    - 若有脚底（`soles`），脚必须限制在 `<|lower_frame|>`（防广角大脚怪）；
    - 若为高脚凳悬空腿（`dangling legs`），在 `<|lower_frame|>` 写明 `legs hang freely in mid-air well above the floor without touching the ground`，负面封杀 `feet on floor, standing, touching the ground`；
    - 若有衣物穿戴动态（如 `jacket slip, scarf pull, hood up`），写成静态结果（`jacket already slipped off one shoulder, scarf hanging loose at the chest`），严禁用拉扯动词占用双手，防止长出第三只手！⚪
  - **触发条件 2（2~6 人多人同框 / 强冲突冷暖灯光防串色）**：
    - **要求：必须“按人（按实体）分块”，严禁“按部位/按颜色”分块**！把同一个人的方位、发色、瞳色、服装、配饰、专属灯光全部关在同一个闭合标签内：
      - 2人并排：`<|left_subject|>...<|/left_subject|>` + `<|right_subject|>...<|/right_subject|>`（中文可用 `<|左侧人物|>「...」<|/左侧人物|>` + `<|右侧人物|>「...」<|/右侧人物|>`）；
      - 4人围坐：按 `<|top_left|>` / `<|bottom_left|>` / `<|top_right|>` / `<|bottom_right|>` 分行闭合；
      - 6人围坐沙发合影（一人坐正中、五人站立于后排）：`<|center_sofa|>...<|/center_sofa|>` + `<|standing_behind|>...<|/standing_behind|>`；
    - **防克隆硬性要求**：多人贴近时，顶部加 `distinct hair colors, contrasting features`，主视觉锚点角色永远放在第 1 个人物块（顺应单向因果流定调全图审美），第 2 个及之后的人物块必须在句首加入显式反向阻断词（如 `In sharp contrast, the girl on the right wears NO glasses, has spotless porcelain skin with zero freckles...`），并在负面焊死 `twins, identical faces, same hair color, attribute bleeding`！
## 6. Step 4 年龄、骨相与身材控制
> 证据等级 A/B 混合，逐条如上标注。
**Step 4：套用人物骨相、年龄、液体与解剖防崩公式（微调词典）**

1. **【年龄与身材卡位（破解“非老即幼”与“文字催肥”）】**：
   - **目标为 14~16 岁青春期少女时**：
     - **正面禁用催老词**：严禁写 `woman, gorgeous, makeup, wavy long hair, wide hips, curvy, C-cup` 以及白人熟女骨相词 `bone structure, defined nose bridge, deep-set eyes`（一写必变 25 岁熟女或 130 斤肥婆）。
     - **正面禁用催幼词**：严禁同时叠加 `very young, very petite, babyface, round chubby cheeks, button nose, budding breasts`，图内印字严禁写 `"LOLITA"` 或 `"148cm"`（一写必缩水成 6~10 岁幼童）。
     - **正面标准黄金词组**：
       - 东方清纯少女：`slender 15-year-old Japanese teenage schoolgirl, cute pure youthful face, short mid-face, bare-faced, zero makeup, small mole under eyes, clear light-amber eyes, natural fine straight jet-black bob hair with thin wispy bangs, slender youthful frame, narrow shoulders, visible collarbone`。⚪（身形词决定年龄档：催老 / 催幼双向锁定）
       - 欧洲雀斑少女：`slender 14-year-old European teenage girl, natural youthful face, softly rounded full chin, soft cheeks, straight nose bridge, light freckles across nose and cheeks, low straight eyebrows, deep olive-hazel eyes, zero eyeliner, zero makeup, pulling and biting lower lip inside mouth in a crooked cheeky smirk, side-parted light golden-brown twin braids with small sky-blue butterfly hairclips clipped high above ears, slender youthful frame with narrow shoulders`（负面必封杀 `pointy chin, V-shaped face, heavy eyeliner, spiky fake eyelashes, mascara`）。
   - **图文解耦规则**：若在杂志页面上印 `"AGE : 19"`，正文中描述身体时必须反向加倍锁死 `very slender, petite skinny teenage frame, flat stomach, visible collarbone, narrow shoulders, thin slender arms and legs`，负面封杀 `fat, chubby, plump, curvy, thick thighs, C-cup, large breasts`！
2. 【多人分块时的发色与发型完整继承规则】：
   - 当为了防串色将发色/发型词（如 `blonde hair, twin braids` vs `black hair, bob cut`）从顶部全局区移入下方的 `Left Focus / Right Focus` 时，**必须在各自的段落内完整写出该角色的发色与发型**（如左边必须写明 `bright honey-blonde hair braided into two loose twin braids`，右边写明 `natural fine straight jet-black bob hair with thin wispy bangs`），严禁在移动过程中漏写发色或麻花辫！
3. 【严禁在白色床单/白丝场景中叠加 `bright background / daylight`（防高调过曝发白）】：
   - 当画面中包含大量白色元素（如 `white magazine border, white pillows, white lace, white stockings`）时，**绝对禁止** 写入 `bright background` 或笼统的 `daylight`！否则多个“白+亮”词叠加会导致全图过曝发白、失去立体感。
   - **正确写法**：必须使用 **“柔和暖调室内光 + 丰富的明暗层次对比”**：`warm indoor bedroom lighting with rich shadow depth and natural skin-tone contrast`，并在 Negative 中加入 **`overexposed, washed out, faded colors, overly bright, pale flat lighting, whiteout`**！

4. 【正面严禁写否定身形词】：⚪
   - 15 岁纤细少女使用 `slender youthful frame with narrow shoulders`，正面 **严禁写入 `flat chest / boyish figure / straight torso`**——这类否定词必须留在 Negative 里：正面写否定词会让模型直接放弃该维度的立体光影起伏，连带衣料褶皱一起消失。
- **全身比例防幼化**：全身镜头必须保留双脚与地面/支点的关系描述（或明确写悬空），并保留鞋袜词；删掉脚部支点锚点会让头身比缩水成五头身儿童。【A】

## 7. Step 4 液体、表情与解剖防崩
> 证据等级 A。
1. **【真发去假发感公式】**：
   - 所有头发描述必须加上：`natural fine [发色] real human hair, thin wispy bangs, loose flyaway strands`。

2. **【通用液体去“湿棉花/白油漆”公式】**：⚪
   - 禁用医学 / 生理解剖词，也禁用 `white liquid / cream` 这类含糊词；
   - 统一写为：`viscous, semi-transparent glossy [液体名] slowly pooling in the hollow of her collarbone (或 trailing down the side of a glass), forming thin stringy trails with bright wet specular highlights`——液体名取 `honey / syrup / milk / cold soda / sweat / rainwater`；
   - **强动作降级三步法**：需要「有接触但没有强动作」时 →
     ① 顶部用静态状态词（`straw between parted lips, tongue out, glass tilted, ice cubes clinking`）；
     ② 正文写**接触几何**，不写动作意图（`her mouth is gently parted around the striped paper straw, her pink tongue visibly touching the straw, a single droplet hanging from the corner of her lips`）；
     ③ 负面封杀强动作词（`chugging, gulping, puffed cheeks, mouth wide open`）。

3. **【面部防丑、防关公红脸、防遮挡露眼】**：
   - **脸红**：永远只写 `faint pink blush`，严禁写 `deeply flushed, burning red`（防关公酒糟鼻红脸）。
   - **趴枕头刚睡醒脸**：严禁写 `face buried squishing cheek + rolling eyes back showing white sclera`（三次元巨丑）；一律写 `resting sideways against a soft pillow without squishing her cheek, looking lazily through half-closed eyes in a sleepy morning daze`。⚪
   - **遮眼/熟睡闭眼盲区法则**：当存在 `hands covering eyes / hair falling over the face / pillow over the face`（遮眼）或 `sleeping / fast asleep`（全员熟睡不看镜头）时，正文中 **严禁出现任何眼神动词**（如 `stares, wide-eyed, looking up, gazing`），正面写 `completely covering and hiding both of her eyes from view`（或 `eyes completely closed, utterly unaware of the camera`），负面最前排焊死 `visible eyes, open eyes, pupils, looking at viewer, eye contact`！

4. **【机位与小部件清晰度】**：
   - 双人腿部交叠（并排盘腿坐、跷腿叠坐 `legs crossed while sitting together`）严禁用纯侧面 `from side`（防多腿重叠融合），改用斜角或正面折叠；⚪
   - 露脚趾时，砍掉远端次要细节，加上 `five distinct, natural toes clearly visible`；
   - 竖中指（`middle finger`）严禁放 `Foreground`（防失焦模糊），写 `right beside her cheek in the exact same sharp focal plane, in crisp focus`。
- **单侧滑落分流**：原图 tag 里有 `pulling own clothes / holding her strap` 才写手抓衣带；只有 `off shoulder / unbuttoned shirt` 时写成自然滑落，禁止让手去扯衣服（详见 §7.1）。【A·机理沿用 ⚪】

### 7.1 单侧滑落的两种写法（主动拉衣带 vs 自然滑落）
> 依据：原图没有「主动拉扯」tag 时写手抓衣物会破坏自然感【A·机理沿用 ⚪】。本节以英文原句为准，§7 的压缩条目作速查用。
1. 【单侧露肩（`off shoulder, one shoulder exposed`）的精准写法】：
   - 顶部必须保留 `off shoulder, one shoulder exposed, visible collarbone`；
   - 正文必须写为：`her [color] knit cardigan has slipped completely off one side, fully exposing one bare shoulder and the line of her collarbone, while her hand firmly grips the slipped strap at the front`（严禁写笼统的 `clothes pulled aside`，否则衣带还会半挂在肩上、锁不出锁骨线条！）。
2. 【「主动拉」与「自然滑」的分流规则（严禁无故加手部拉扯）】：
   - **IF 原图 Tag 中明确包含 `pulling own clothes / holding her strap / adjusting clothes`（主动拉扯）**：
     ➔ 才可写 `her hand firmly grips and pulls the slipped strap`。
   - **ELSE IF 原图 Tag 中仅有 `off shoulder / unbuttoned shirt / cardigan slip` 而无主动拉扯 tag**：
     ➔ **绝对禁止** 在顶部或正文添加任何拉扯动词或让手去扯衣服！必须写成随身体前倾或步伐产生的自然滑落：`her oversized knit cardigan naturally slips open off one shoulder as she leans forward, casually exposing one smooth bare shoulder and her collarbone`（让衣物处于自然垂落状态，双手只做端杯子、抱书或拢头发的动作，保持画面的自然松弛感！）。

### 7.2 双色发与发型继承
> 证据等级 B：单次复现。
1. 【双色发（`two-tone hair`）角色常识校验】：
   - 当同时出现 `blonde hair, pink hair, two-tone hair`（如 `kitagawa marin`）时，主发色是金发、发梢是粉发，必须写为 `blonde hair with pastel-pink tips`，严禁写反成满头粉发！

## 8. 多人同框防克隆
> 证据等级 A（四发色强隔离法）；本节引用的 "Featuring exactly [N] distinct girls, each appearing once" 为 B 级，出自官方仓库 Blue Archive 十人示例，未单独对照。我们未做该句式的独立对照。
  - **触发条件 2（2~6 人多人同框防克隆变双胞胎）**：
    - 2人并排：分为 `Left Focus` 与 `Right Focus`；
    - 4人围坐：分为 `Center Reference` + `Top-Left` / `Bottom-Left` / `Top-Right` / `Bottom-Right` 四象限；
    - 6人围沙发合影（一人坐正中、五人站立于后排）：分为 `Center Foreground on Sofa`（1个坐姿角色）+ `Standing Behind the Sofa`（按 `Far Left / Mid-Left / Center-Back / Mid-Right / Far Right` 依次列出5个站立角色）；
    - **防克隆硬性要求**：多人贴近时，顶部加 `distinct hair colors, contrasting features`，每个人在各自区块内写死截然不同的发色、人种与特征（如左女写 `European girl with visible freckles`，右女必须显式写 `Japanese girl with spotless porcelain skin and zero freckles`），并在负面焊死 `twins, identical faces, same hair color, attribute bleeding`！
**多人同框（2~10人）官方原生防克隆与群像排布协议**
- **官方示例原生防克隆句**：当画面包含 2 个及以上角色（如双姝并排、四人围炉、六人沙发合影、十人毕业照）时，建议在正文开头第一句写入官方公开示例中的防克隆与构图约束句（注意：这是官方示例的写法，不等于训练 caption 的原生句式，等级 B）：
  ➔ **`"Featuring exactly [N] distinct girls, each appearing once, in an asymmetric ensemble composition (not a rigid grid)."`**
- 配合每个角色独立区块内的显式互斥描述（如左女写 `European girl with visible freckles`，右女显式写 `Japanese girl with spotless skin and zero freckles`）与负面池 `twins, identical faces, same hair color, clone`。

## 9. 画面内文字与排版
- **双引号锁字**：画面里要渲染的每个字符串用英文双引号括起，需要换行处直接换行。【A】
- **参数切换（按字号与密度分流）**：
  - **整页高密度微字（如满版 A4 高考试卷、多栏密集杂志正文）**：必须切到 SLG `"22"` + Steps ≥ 50 + 分辨率 ≥ 1M，否则低分辨率下小字必糊。【A】
  - **中等字号 UI / 字幕 / 标题（如 Galgame 对话框台词、TV 番剧底部字幕、SSR 卡牌标题与攻防数值、海报大字）**：无需开 SLG 或高步数，在 `SLG=""`（置空）+ 16 步 + 0.65M 像素下即可锐利零错字渲染。【A】
- **信息密度上限**：单页塞满整张 A4 的题目量时，字号与分辨率必须同步抬高，否则 VAE 解码糊字。【B】
- **印字催幼/催肥陷阱**：图内印 `"LOLITA"`、`"148cm"` 会把人画成幼童；印 `"AGE : 19"` 会连带把身材画胖，必须在正文反向锁死纤细词并在负面封杀肥胖词（图文解耦）。【A】

## 9.1 日系写真集（Gravure Photobook）专属规则
> 冲突裁决：写真集/杂志内页一律用本节开头固定句，不用 §4 的哈苏分支。型号名本身效力弱（§14.2），两种写法不叠加，必须二选一。
> 证据等级 A/B 混合：Hasselblad 冲突与曝光发白条目为 A；眼神与配饰词条为 B（单次美学对比）。
1. 【日系写真集（Gravure Photobook）专属镜头规则】：
   - 当制作日系写真集内页（如 Page 85 卧室满幅版式）时，**严禁使用复古沉重的 `Hasselblad 500C/M`**！
   - 开头固定使用：`high-resolution Japanese gravure photobook page layout (Page 85 style), full-bleed vertical bedroom photograph framed by a clean thin white magazine border... soft diffused natural bedroom window light`，让床单与窗户的漫反射光营造轻盈、安静、慵懒的现代空气感。

2. 【废除“乖学生证件照”形容词，注入“微乱晨起感与羞涩眼神”】：
   - **发型禁词**：在床上或私密场景中，**严禁写 `neat`（整齐的）**！必须写为 `loose twin braids, airy wispy bangs, delicate flyaway strands spread across the soft pillows`（松散微乱的发辫与透气碎发），并主动保留角色的柔美小配饰（如莉莉艾的 `white flower hair ornament`）。
   - **眼神升级**：严禁写毫无情绪张力的 `looking gently`！必须使用兼具羞涩与故事感的复合词：**`shy, slightly averted gaze`（微微躲闪的羞涩眼神）** 或 **`soft dreamy gaze resting just past the camera`（望向镜头之外一点的失焦柔光眼神）**。⚪（复合情绪词优于泛化副词）

3. 【精细服饰材质与肌肤光泽细节词】：⚪
   - 描述极限微距服饰细节时，必须加入材质通透感与纤细对比：`delicate white cotton collar trimmed with hair-thin gold embroidery, tiny pearl buttons along the placket`（发丝般极细金线绣边 + 门襟小珍珠纽扣构成的领口细节）。
   - 结尾皮肤质感升级为：`Smooth, supple peach-like real human skin with a soft, luminous magazine gravure finish—delicately retouched for photobook perfection without looking like plastic`。

## 10. 游戏实机截图与 HUD
> 证据等级 A。
**游戏实机截图与 HUD 界面生成协议（Gameplay Capture & HUD）**
- **能力边界声明**：
  - ✅ **支持良好**：四角游戏 HUD 界面生成、三渲二卡通游戏截图（如 `Breath of the Wild` 赛璐璐水彩风）。
  - ❌ **底座盲区（禁止强求）**：模型不支持纯靠提示词生成“UE5 / FF7 重制版级次世代写实 3D 多边形渲染人物”（强行写 `Unreal Engine 5 / 3D character model` 会退化为廉价磨皮塑料人偶或 2.5D 厚涂画）。若用户要求“写实风游戏角色”，一律直接走 **【35mm 胶片/手机直闪真人实拍路线】**，严禁使用 `3D render / CGI / Unreal Engine`！
- **若用户明确需要【带游戏 UI / HUD 的实机截屏画面】**，必须严格套用以下官方 4 步结构：
  1. **开头原生 Meta 词**：`screenshot, video_game_capture, [genre_tag], [game_name].`
  2. **第一段官方反宣传图声明**：`A cinematic 16:9 gameplay screenshot from [Game Name], rendered with the game's [style] aesthetic rather than as promotional artwork. Captured from an immersive third-person over-the-shoulder gameplay camera...`
  3. **第二段四角 HUD 精确文字锚定（触发游戏画面的核心扳机）**：
     `Preserve the recognizable in-game HUD overlay: in the top-left corner, a status_ui with a red health_bar and blue stamina_bar; at the top-center, a horizontal compass bar with crisp letters "W", "NW", "N"; in the bottom-left corner, an equipment_slot icon with weapon_skill label reading "[Skill Name]" and quantity "3"; in the bottom-right corner, a numeric counter reading "1248".`
  4. **官方收尾禁令句**：`No title logo, desktop borders, subtitles, or promotional text; this is an immersive moment of ordinary exploration captured directly from gameplay.`
- **HUD 是触发扳机**：带四角精确 HUD 锚点的版本正确，去掉 HUD 只留文字声明的版本无效果。要出"游戏画面感"，HUD 具体部件名与引号内数值不可省。【A：HUD 锚定组 vs 无 HUD 对照组】

## 11. Step 5 输出模板与自检清单
> 证据等级 A。
**Step 5：组装输出标准模板（Output Template）**

每次输出必须严格包含以下 Positive 与 Negative 结构：

#### Positive Prompt 模板（最新 `<|...|>` 闭合标签标准版，分支 B）：
```text
<|style_and_camera|>[Step 2 摄影设备/画种元数据], [景别与机位], razor-sharp focus, deep depth of field, [角色名], [清洗后的核心 Danbooru Tags], fully clothed in an everyday outfit.<|/style_and_camera|>
<|upper_or_left_focus|>[方位 1 显式声明如 On the left / In the upper frame]: [主体 1 的骨相、真发、表情、专属受光色温、静态衣物结果与动作]...<|/upper_or_left_focus|>
<|lower_or_right_focus|>[方位 2 显式声明如 In sharp contrast, on the right / In the lower frame]: [主体 2（含反向因果阻断词）或次要主体的接触几何、半透明高光液体描述（汗水 / 饮料 / 雨水）]...<|/lower_or_right_focus|>
<|texture_lock|>Unretouched youthful fair skin texture with microscopic pores (若为杂志柔焦写真则换为 Smooth, supple peach-like real human skin with a soft magazine gravure finish), real human hair strands in sharp focus, realistic wet liquid specular highlights, amateur direct flash photography, organic 35mm analog film grain.<|/texture_lock|>
```
*(注：若属于 §5 分支 A 全身紧密交叠互动或跨次元同空间自然融合，则不使用 `<|...|>` 标签，直接写 4 句 85 词以内的连贯自然语言短文。)*
#### Negative Prompt 标准底座（每次根据场景追加专项封杀词）：
```text
[首排必焊着装与姿态封杀组：nudity, nude, topless, shirtless, cleavage, sideboob, underboob, see-through clothing, lingerie, underwear visible, erotic, seductive pose, suggestive pose, sexual act, explicit, nsfw], [场景专项封杀词：如遮眼加 visible eyes / 高脚凳加 feet on floor / 多人加 twins, identical faces, same hair color / 换装加原图旧衣物词], child, toddler, preteen, loli, 6 years old, 8 years old, 10 years old, babyface, flat chest, mature female, adult woman, milf, old, 22 years old, 25 years old, 30 years old, high cheekbones, hollow cheeks, long face, deep-set eyes, pointy chin, V-shaped face, heavy eyeliner, spiky fake eyelashes, mascara, heavy makeup, fat, chubby, plump, wide adult hips, cotton, foam, whipped cream, thick white paint, opaque white blob, red face, burning red cheeks, sunburn, rosacea, tomato face, cosplay, cosplayer, wig, synthetic hair, cosplay wig, stiff hair, thick blunt bangs, colored contact lenses, blurry, out of focus, soft focus, lowres, censored, mosaic censoring, bar censor, anime, illustration, 2d, 3d render, cgi, doll, plastic skin, ugly face, distorted face, squished cheek, grimace, creepy eyes, worst quality, bad anatomy, fused bodies, fused legs, extra limbs, extra arms, extra legs, extra hands, three hands, extra fingers, mutated hands, fused fingers, six fingers, duplicated object, extra bag, fused hands, watermark, text
```

**输出前强制自检清单（Pre-Flight Checklist）**
在把提示词发给用户前，必须在内部核对以下 7 项（全部为 Yes 才可输出）：
1. 版权大词（`pokemon / naruto` 等）、`cosplay`、打码词是否已从正面全部剔除？
2. 衣物穿戴动态是否已写成静态结果（没有因为既拉衣带又拢头发还端杯子而需要 3 只手）？
3. 若有遮眼或熟睡，正文里是否已删光所有 `look / stare / wide-eyed` 动词？
4. 若为 14~16 岁少女，是否已剔除 `woman / curvy / bone structure / deep-set eyes / wavy hair`（防老防肥）以及 `babyface / very petite / budding`（防幼童）？
5. 若为 2 人以上同框，是否已按方位切块并为每个人指定了互斥的发色/肤色/有无雀斑，且在负面加入了 `twins, identical faces, same hair color`？
6. 画面内所有液体是否已使用 `viscous, semi-transparent glossy ... with bright wet specular highlights`，并在负面封杀了 `cotton, foam, thick white paint`？
7. 正面是否已删光 §2 必删项 F 的器官 / 体液 / 体位 / 半裸词？负面首排是否已焊死着装与姿态封杀组？

## 12. 禁用清单（D 级）
以下做法实测无效，禁止使用：
1. **孤立相机/胶片/工艺名**（Contax T2、Fujifilm Pro 400H、Polaroid SX-70、孤立的 Wet plate collodion 或 CCTV footage）：单变量替换无可辨差异；但一旦展开为对应的**物理显影与光学后果词**（如相纸边缘药水黄斑折痕、荧光绿夜视反光白瞳 + 时间戳，见 §4.3），即可 100% 触发对应媒介。【A】
2. **渲染引擎商标名**（Unreal Engine 5、Octane Render、Blender Cycles、Subsurface scattering）：单独出现不产生对应风格，最多拼出模型自己理解的近似质感。【A】
3. **写实次世代 3A 游戏 3D 渲染人物**（FF7 重制版 / 法环级 PBR 建模感）：底座分布盲区，强行写 3D render / CGI 会退化成磨皮塑料人偶或 2.5D 厚涂；要写实游戏角色只能走真人胶片实拍路线。【A】
4. **正面一律删角色名 / IP / 画师名**：错误。角色名按 §3 保留，只有画师名与版权大词该删。【A】
5. **在 150 词写实毛孔基底下用单点 `cel shading` / `CCTV footage` 换画风**：单变量无反应（因被后段 150 词写实毛孔与闪光灯描述压制）。若主体基底保持"画风中立（不写毛孔/闪光灯/胶片词）"，则无需改结尾与负面，仅替换开头 8~15 词的 Danbooru 宏观媒介/年代/技法词组即可切换整图画风（见 §16）。【A】
6. **把 SLG `"22"` vs `""` 当作"插画 vs 写实"的画风切换开关**：当提示词为二次元/动漫描述时，即使 `SLG=""`（置空）也生成纯正二次元插画，不会变成三次元真人。SLG 的真实作用是高频几何边缘收敛与抗粘连。【A】
7. **在重度风格偏置基底下只改开头 1 个词指望跨次元换画风**：当正文已包含强风格偏置词（如写实毛孔或具体颜色词）时，开头单点词无法覆盖正文；必须清除正文中的冲突偏置词（例如黑白漫画或 JSON `palette` 控色时，必须删去正文里的 `red eyes, navy` 等自然语言颜色词，见 §16.2）。【A】
8. **对未收录的个人画师写 `XXX style` 死代码，或用五官/高光几何文字硬凑个人画师手癖**：模型对未进入世界知识库的画师名（如 `kawakami_rokkaku`、`yewang19`、`ciloranko`）不响应，禁止写 `XXX style`；同时禁止用"四角星高光、斜杠腮红、贴纸白边、半垂眼"等碎片化几何描述硬凑某画师脸型——模型会按字面均值机械拼贴成僵硬的"拼图脸"，无法复刻画师笔触（见 §16.1）。【A】

## 13. 机理解释（C 级，不作为规则）
- 模型只有一条文本条件通道：技术报告 Appendix A.2 `Caption, structural, and metadata fields are combined before applying the Qwen chat template and tokenization.` 所谓"元数据字段"不是独立入口，任何名词的效力取决于它在训练 caption 里的共现频率（见 §14.3）。
- 训练 caption 由 VLM 批量重打标生成（技术报告 §3.2 点名 **gemma4**），词表高度同质化：打标器只写"肉眼可见的颜色、明暗、反光、笔触"，不写器材型号与软件名。因此"描述视觉后果"优于"命名工艺"。一手出处见 §14.2。
- 人像先验偏向正面平光：不显式声明暗部时，模型会用训练集均值给正面补光。
- 单点 token 在 150 词长提示词中权重占比过低（约 2%），撬不动整体分布；成块替换（头+尾+负面）才有效。
以上均为对实验现象的推测，尚无独立拆解证明。

---

## 14. 官方规范对照与采样点声明

### 14.1 本文件的效力边界（先读这段）
本文件全部规则建立在如下**固定采样点**上：`640×960` / `960×640`（约 0.61M 像素）、Steps `25`、CFG `3.5`、Shift `5.0`、Euler、SLG（`single_layers`）**置空**。

| 参数 | 官方推荐 | 本文件固定值 | 偏离性质 |
|---|---|---|---|
| 分辨率 | 1M / 1.5M / 2M（可外推 0.25M–4M） | ≈0.61M | 低于推荐下限，速度优先 |
| Steps | ≥ 50 为最佳 | 25 | 减半 |
| CFG | 3.0（区间 3.0–6.0） | 3.5 | 区间内 |
| Shift / Sampler | 5.0 / Euler | 5.0 / Euler | 一致 |
| SLG | 开启 Two-Pass `uncond skip` layer 22（22/24/26 皆可） | 置空（关闭） | **有意关闭**换写实感 |
| Negative | `nsfw, worst quality, bad anatomy, watermark, jpeg artifact` | 数百词专项负面池 | 远超官方建议量级 |

官方 USERGUIDE 原话：`Turning off SLG will make the output image look very realistic, but it may also cause some details to collapse.`
因此本文件里整套解剖防崩词条（§5、§7）是**针对"关闭 SLG + 半程步数"所致细节崩坏的人工补偿**，不适用于 SLG 开启的高分档位。
`single_layers` 是 ComfyUI 端 SkipLayerGuidance 的跳层参数。官方只给出"关闭更写实、但可能细节崩坏"与推荐 layer `22`/`24`/`26`，**并未**给出"某个 layer ↔ 某种画风"的确定性映射。在动漫提示词下，`SLG=""`（置空）同样产出纯 2D 动漫，所谓"置空 = 写实"仅在写实提示词下成立；SLG `"22"` 的真实效果是改善窗框、书页、百褶裙等高频几何边缘的收敛度。【A】

### 14.2 为什么"描述视觉后果"优于"命名器材/引擎"（机理，B+ 级）
官方打标规范的两处一手原文：
- USERGUIDE Image Captioning：`Include only information visible in the image.` 所列描述维度为 composition / key subjects / actions / setting / colors / lighting / artistic style。
- 技术报告 §3.2：`We caption images with gemma4 at different semantic granularities: a short sentence, a medium paragraph, or a detailed description of composition, subjects, scene attributes, and visible text.`
两份清单里**均无器材型号、胶片型号、渲染器条目**——重打标模型（Gemma 4）从不被要求写这些，所以它们不在模型学到的质感线索集合内。
- 相机型号、胶片型号、渲染引擎名属于"不可见推断"，不会进入训练 caption → 模型没有学到"型号名 → 质感"的映射。§12 已证伪清单第 1、2、5 条由此得到解释。
- 报告 §3.2：存在原始 caption 时 **80% 采样 Gemma 4 生成的 caption**，20% 用原始 caption；Danbooru 画师 tag 不是可见信息，故在训练语料中被系统性稀释 → §3 中"删除画师名"由此得到解释。
- 报告 Table 12：`meta` p=0.1、`canvas`/`caption` p=0.5、可选字段 p=0.5 随机丢弃，而 `gen_brief` / `elements` 基本保留 → 元数据词的条件效力天然弱于正文描述，故本文件要求"成块替换（开头 + 结尾 + 负面）"而非改单个词。
- 报告 §3.1：数据不做质量过滤，水印与伪影一律写进文本监督（annotate rather than filter）→ 解释了为什么单靠负面池压不住问题，必须在正面写出期望的视觉后果。

### 14.3 官方结构化写法与本文件的差距
1. 本文件用 `[Image Composition]` + 自然语言方位词分区；官方部署规范是 JSON `summary/items` + `bbox_coord_space:"abs_centralized"` + `bbox_format:"yxyx"`（中心原点绝对像素）。报告 Appendix K 称中心原点坐标在未见分辨率上泛化更好。
2. 官方 `must_not` 上限 3 条具体排除项；本文件用数百词负面池，长度未做消融。
3. 官方示例 `assets/images/004` 的防克隆原句（`each appearing once` / `not a grid`）已在 §8 引用，等级 B：原文属实，但我们未对该句式做单独对照。
4. 官方 `summary.meta` 可容纳作品名（`assets/images/011`：`["screenshot","video_game_capture","dark_fantasy","elden_ring"]`），但所有字段最终拼接后走同一条 Qwen chat template 文本通道（报告 Appendix A.2），不存在独立元数据嵌入；因此任何名词的效力取决于其在 caption 语料中的共现频率。
5. 提示词长度落在训练分布内：官方 caption 统计 short mean 20.9 / medium mean 76.6 / long mean 235.1 tokens（P90 293，最长 3,126），条件上限 4,096 tokens；本文件分支 A（≤85 词）≈ medium 档，分支 B 分块长文 ≈ long 档。

---

## 15. 输入格式层：官方 Layout JSON schema
> 官方将该接口标注为 **Experimental**。本节条目照搬官方 `USERGUIDE > Layout Prompts` 原文与字段定义，未改写。
官方同时说明：`The model supports combining natural language and structured input.`（自然语言与结构化输入可混用。）

### 15.1 官方示例（结构照此，字段值按需替换）
```json
{
  "summary": {
    "meta": ["poster"],
    "canvas": "single",
    "caption": "A minimalist travel poster with a cream background, a blue mountain illustration, and the title EXPLORE at the top."
  },
  "items": [
    {
      "label": "text",
      "caption": "Large dark-blue uppercase title centered at the top.",
      "text": "EXPLORE",
      "bbox": [-416, -384, -288, 384]
    },
    {
      "label": "object",
      "caption": "A flat blue mountain illustration in the lower center.",
      "bbox": [-192, -288, 384, 288]
    }
  ],
  "bbox_coord_space": "abs_centralized",
  "bbox_format": "yxyx"
}
```

### 15.2 字段定义（官方原文）
- `summary.caption`：`A description of the overall scene and style.`（整体场景与风格描述。）
- `items`：每个元素的类别（`label`）、描述（`caption`）、边界框（`bbox`）；渲染文字时用 `text` 指定内容。
- `bbox_format: "yxyx"`：坐标顺序 `[y_min, x_min, y_max, x_max]`。
- `bbox_coord_space: "abs_centralized"`：绝对像素坐标，画布中心为原点，x 向右、y 向下。例如 1024×1024 画布两轴范围均为 −512…+512。
- 从常规左上原点换算（官方原句：`subtract W/2 and H/2 from conventional top-left-origin x and y coordinates, respectively`）：`[y_min, x_min, y_max, x_max] = [y′_min − H/2, x′_min − W/2, y′_max − H/2, x′_max − W/2]`。
- 坐标空间选择是官方明确立场：`We use absolute pixel coordinates, which we believe are better suited to layout control across different resolutions and aspect ratios.`

### 15.3 取值域（官方 skill 文档）
- `summary.canvas`：`single` / `multi_panel` / `collage` 三值之一。
- `items[].label`：`object` / `character` / `text` 三值之一。训练语料里还包含 panels、characters、watermarks 等类别，但公开编译器只接受上述三个；论文推理增强器的 `elements` 属另一套 schema，官方 skill 原文禁止写入（`Do not add ... scene-schema elements`）。
- `summary.palette` 与 `items[].palette`（HEX 字符串数组）官方支持（`palette on summary is also supported`）。

### 15.4 编译与参数交接
- 分数边缘按官方规则 floor 左上、ceil 右下，最终为整数且宽高为正。
- 编译产物中删去 width/height 等画布尺寸字段，尺寸在生成参数中另设（与本文件 §1 的采样点一致）；W、H 取正的 16 倍数。
- 负面提示词走 CFG 的 `negative_prompt`（官方 Prompt Guidelines 给的 5 条由本文件 §10 的负面池覆盖并扩充）。论文 planner 的 `anchors` / `must_not` 是推理增强器内部字段，公开 schema 中不存在，不要写入。

### 15.5 与本文件其余条款的交接
- `summary.caption` 按官方定义承担"整体场景与风格"，因此 §4 的画风锁定句写在它开头；不要把画风词挪进 `meta`——所有字段拼接后进入同一条文本通道（§14.3 第 4 条）。
- 解剖防崩、年龄、液体词条写进各 item 的 `caption`（§5、§6）；`bbox` 只约束位置，不替代它们。
- 图内文字内容仍受 §9 约束（印字催幼 / 催肥陷阱、双引号锁字与换行写法）。

---

## 16. 动漫/二次元画面风格控制与 Gemma 4 有损压缩边界协议
> 证据等级 A。

### 16.1 核心物理边界：Gemma 4 重打标的"去画师化有损压缩定律"【A】
1. **打标阶段的信息不可逆丢失**：
   - 官方训练语料 80% 采样自 Gemma 4 按 `USERGUIDE.md` 规范生成的 `caption.long/medium/short`（规则：`Include only information visible in the image`）。
   - 取个人画师真迹原图严格按官方 Gemma 4 规则写出 1:1 的 `caption.long` 重新生成，实测边界如下：
     - **保留层（语义与构图）**：机位（荷兰角倾斜、俯拍）、人物动作、服装道具、环境光影（壁炉暖光、大理石倒影与中文招牌）均能精准复现；
     - **丢失层（画师手癖与非标骨相）**：画师独有的笔压曲线、水彩混色边缘、铅笔灰调线稿以及偏离主流均值的个人骨相在转化为英语词汇时已不可逆丢失，生成画面会强行**向全网公模动漫均值回归**。
2. **两条硬性执行要求**：
   - **要求 A（严禁写未收录画师名死代码）**：除 §3 所列 5 组世界知识白名单（`Studio Ghibli / Hayao Miyazaki`、`Hirohiko Araki / JoJo`、`Akira Toriyama / Dragon Ball`、`Ilya Kuvshinov`、`WLOP`）外，严禁在提示词中写任何个人画师名或 `XXX style`（如 `kawakami_rokkaku`、`yewang19`、`ciloranko`、`rella`、`liduke`、`mika pikazo`、`ask`）。
   - **要求 B（严禁用碎片化五官几何词硬凑个人画师脸）**：若把画师手癖拆解为字面几何词（如 `four-point-star white hair highlights, diagonal blush hash lines, droopy half-lidded eyes, white sticker rim`），模型会按字面均值机械拼贴出僵硬脸型，反而破坏画面美感。要复刻特定个人画师笔触与脸型，**唯一可行路径是训练该画师 LoRA**，不得靠提示词硬凑。

### 16.2 可控的四大动漫风格调动通道（基于官方保留内容）【A】
当主体描述保持**画风中立（Style-Neutral：不写三次元毛孔/闪光灯/胶片词）**时，无需修改结尾与负面词，直接通过以下通道即可在 16 步、0.65M、`SLG=""` 下切换动漫风格流派：

#### 通道一：Danbooru 宏观画种 / 年代 / 技法分类开关（放在 Positive 开头）
> 依据：Gemma 4 重打标时参考了原图 `{{reference}}`（Danbooru Tags）并被要求详细描述 `artistic style, colors, lighting`，故以下 Danbooru 宏观可见分类词在语料库中具有高响应度。

| 目标动漫子流派 | 实测单点生效的开头词组（直接前置，无需改负面） | 实测视觉效果与注意事项 |
| :--- | :--- | :--- |
| **1990s 复古赛璐璐 OVA** | `1990s (style), retro artstyle, anime screencap, cel shading, vintage 1990s OVA anime aesthetic, VHS tape color bleed, hand-painted cel animation` | 蓬松 90s 复古刘海、暖橘胶片肤色、经典大白高光眼、手绘水粉背景 |
| **纯黑白日漫网点纸** | `manga page, monochrome, greyscale, screentones, inked lineart, bold black ink outlines, pen crosshatching, Japanese manga illustration` | **【颜色词穿透要求】**：必须同时**删光正文里所有具体颜色词**（如删 `red, silver, navy`），否则正文颜色词会穿透黑白指令变成"局部留色漫" |
| **透明水彩晕染绘本** | `watercolor (medium), traditional media, ink and wash, soft pastel watercolor bleed on rough textured paper, delicate hand-painted illustration` | 冷压水彩纸纹理、边缘水渍自然扩散、柔和淡彩 |
| **高饱和扁平矢量波普** | `flat color, vector illustration, bold thick colored outlines, vibrant pop art palette, hard geometric cel shading, flat graphic artstyle` | 无渐变色块硬切、纯色暗部、潮酷平面插画感 |
| **厚涂刮刀油画插画** | `impasto, oil painting (medium), heavy visible brushstrokes, thick palette knife paint texture on canvas, painterly lineless illustration` | 抹除黑线，全图呈现立体堆叠的油画刮刀笔触肌理 |
| **日本动画原画手稿 (Genga)** | `anime genga, production art, rough pencil sketch, uncolored animation layout sheet, blue and red colored pencil construction lines on paper` | 白纸铅笔线稿 + 动画工业标志性的红蓝彩铅结构辅助线与排线阴影 |
| **二头身 Q 版 (Chibi / SD)** | `chibi, super deformed, cute 2-head-tall chibi style, oversized head and tiny body, kawaii deformed anime illustration, bold outlines` | 直接将人体头身比锁定为二头身粘土人比例 |
| **浮世绘 / 日本画和纸木版** | `ukiyo-e, traditional Japanese woodblock print, nihonga, flat mineral pigments on aged washi paper, sumi ink contour lines` | 泛黄和纸肌理、矿物颜料平涂、古典丹凤细眼 |
| **PC-98 16-bit 像素抖动** | `pixel art, 16-bit retro PC-98 Japanese adventure game graphic, crisp visible square pixel grid, checkerboard dithering shading, limited 16-color palette` | 清晰方形像素网格 + 天空与阴影处的棋盘格抖动网点 |
| **无描边油画棒/蜡笔绘本** | `lineless anime illustration, soft oil pastel and crayon drawing on rough textured paper, no black outlines, grainy chalk and pastel texture` | 彻底消除黑色外轮廓线，呈粗糙粉彩颗粒感 |
| **狂放水墨重彩宣纸风** | `Chinese wild ink and wash painting, expressive dry-brush sumi calligraphy outlines with flying-white texture, splashed black ink mixed with traditional mineral pigments of cinnabar red and indigo on coarse yellowed xuan paper` | 枯笔焦墨飞白、泼墨重彩、粗宣纸纹理 |

#### 通道二：二次元四大"媒介容器扳机（Container Triggers）"
> 依据：与 §10"四角 HUD 触发 3D 游戏实机管线"同源。特定 UI 容器与特定二次元商业成图画风在训练语料中共现，写出容器与精确文字即可反过来锁定主体画风：
1. **Galgame / 视觉小说 CG 扳机**：
   - 开头写 `visual novel cg, Japanese galgame screenshot, glossy anime coloring`；
   - 结尾锚定底部对话框：`At the bottom of the frame, a semi-transparent dark-blue visual novel dialogue box with a pink nameplate reading "[角色名]" and crisp white text reading "[台词]"`.
   - 效果：自动将机位与人物涂法切换为正对镜头的 2010s 商业 Galgame 攻略立绘。
2. **TV 动画番剧正片平涂扳机**：
   - 开头写 `anime screencap, TV anime broadcast screenshot, cel shading, flat anime coloring`；
   - 结尾锚定底部黑边黄字字幕：`At the bottom center, crisp yellow anime subtitle text with black outline reading "[台词]"`.
   - 效果：自动洗掉插画厚涂感，锁定为极简一阶硬阴影的 TV 动画播出截帧。
3. **官方角色设定集三视图扳机**：
   - 开头写 `anime character reference sheet, official character design turnaround sheet on clean white background, multiple views, full-body front view and 3 facial expression headshots, clean cel-shaded lineart, color palette swatches`.
   - 效果：自动生成同角色多角度立绘、表情差分头像与排列整齐的配色色卡方块。
4. **SSR 集换式卡牌框扳机**：
   - 开头写 `SSR anime trading card game item, ornate golden holographic card frame border, top title bar reading "[标题]", 5 gold stars at top right, bottom stat box reading "ATK 2500 / DEF 2100", dazzling anime card illustration`.

#### 通道三：官方高保留率 JSON 字段（`palette` HEX 像素级控色与 `multi_panel` 同图异画风）
> 依据：技术报告 Appendix C.3 Table 12 显示 `scene_type`/`gen_brief`/`palette` 与 `summary.meta`（90% 保留）在训练中保留率最高。
1. **`palette` 十六进制数组像素级同构换色**：
   - 当需要迁移某张参考图的精确配色方案时，直接从参考图吸管取 4~5 个主色 HEX 填入 JSON 的 `"palette": ["#HEX1", "#HEX2", ...]`；
   - **强制前提**：必须从 `gen_brief` / `caption` 正文中**删去自然语言具体颜色词**（如删 `silver, red, navy`），并在末尾加一句 `dominated strictly by the specified palette`。可在保持人物线条、构图、动作不变的前提下，将全图色彩锁定为目标 HEX 色板。
2. **`canvas: "multi_panel"` 同图跨画风分格**：
   - 使用 §15 的 `summary/items` JSON，设 `"canvas": "multi_panel"`，在每个 `item` 的 `bbox`（`abs_centralized` `yxyx`）内分别写入不同画风的 `caption` 与 `tags`（如上格 `["1990s (style)", "cel shading", "full color"]`，下格 `["manga", "monochrome", "screentones"]`），可在单张图像内实现上下分格完全不同的动漫时代画风且零串色。
3. **跨次元 2D/3D 混合（从"贴纸拼贴"到"电影级 2.5D 实景与极简线稿光影合成"）【A】**：
   - **基础硬切贴纸档（Roger Rabbit 强反差风）**：显式声明背景为 `unretouched photorealistic 35mm photograph with shallow depth of field`，主体为 `flat 2D cel-shaded anime character illustration with crisp black ink outlines and flat anime coloring`。注意：若不加光影合成指令，角色会呈现带白边的剪贴画感；若用 `<|foreground_photo|>` 与 `<|background_manga|>` 分块包裹，则会触发上下漫画分镜硬切线。
   - **高阶优秀画风档（85mm 电影实景 + 极简潦草线稿轻上色 + 后期逆光打光合成）**：
     - **三大核心支柱**：① **背景**：`85mm 专业镜头拍摄的写实风格背景，浅景深焦外光斑，强烈光线照下` + `荷兰角倾斜构图、仰拍、侧面靠后视角` + `150cm 娇小身高比例，四肢不要画得过长`（防止 3D 摄影先验把动漫少女拉成写实大长腿）；② **角色线稿与上色**：`角色画风【线条极简】，简单的手绘线稿，线条清晰明确简单潦草，只绘制最低限度的单层阴影和极简上色`；③ **跨次元融合桥梁**：`结合后期的光影效果、逆光发丝金边和打光调色合成，让动漫画风少女完美地和写实背景融合`。
     - **三档上色浓度旋钮**：
       1. **清透淡彩唯美档（推荐主力）**：同时写入 `风格：二次元梦幻、极致唯美插画`（提供清透柔和固有色）+ `角色画风【线条极简】，简单的线稿，只绘制最低限度的阴影和极简上色，结合后期的光影效果和打光合成`（洗掉厚涂油腻感并让真实逆光穿透发梢与衣摆）。
       2. **纯白描速写手稿档**：删去"极致唯美插画"与具体颜色词，将`【线条极简】简单的手绘线稿、最低限度单层阴影`前置，直接产出"纯白底铅笔速写少女抓着真实三维金属铁链、发丝被真实阳光打出金边"的艺术效果。
       3. **暖金环境光浸染档**：英文写入 `warm golden sunlight rim-lighting and ambient color grading bleed naturally across her simple sketched lines`，让实景夕阳色温均匀浸染松散速写线条。
     - **近景特写防 3D 真人坍缩要求**：当拍**近景特写（Close-up）**时，若句首写`生成一张 85mm 镜头拍摄的电影感人物照片`且只拍后脑勺/肩背不露五官，模型会顺着 Qwen3.5 单向因果流把人物 100% 坍缩为**三次元真人**！近景特写必须：① 句首前置声明 **`跨次元融合：2D动漫画风少女与85mm写实摄影背景合成`**；② 显式要求 **`侧头回眸露出清晰的2D动漫侧脸线条与明亮动漫眼眸`**（且砍掉「抬腿」「大幅度转身」等下半身动作词以防镜头按 §2.1 自动拉远）⚪。

### 16.3 证据边界与待复验清单（先读这段再引用具体条款）
> 证据等级：D（声明性），不参与任何正向推断。
1. **⚪｜机理沿用**：正文中以 ⚪ 结尾或标注的条款，其机理在同构场景中已验证成立，但在当前日常向载体上**尚未单独跑过对照**——按「高置信起点假设」使用。未标 ⚪ 的 A 级结论均来自本地左右对照实测。
2. **A 级引用前提**：A 级结论固定在 §14.1 的采样点（`640×960` / Steps 25 / CFG 3.5 / SLG 置空）上成立。换分辨率、换步数或开 SLG 后，结论强度自动降一档，需重跑单变量对照。
3. **未验证项**：§7 通用液体公式的 ⚪ 条、§9.1 眼神与配饰条、§6 黄金词组中的 ⚪ 条均属单点美学判断，换场景可能失效；出图后请肉眼复核，不要连写三个未复核条款。

---

## 17. `<|概念|>...<|/概念|>` 闭合标签结构化协议（自包含完整规范）
> 证据等级 A。本节为完全自包含规范，涵盖生图提示词（中/英文）与 LoRA/SFT 训练集标注格式。

### 17.1 为什么用 `<|概念|>...<|/概念|>` 替代一锅炖长散文？
1. **底层分词与向量机理**：
   - Qwen3.5-9B 的原生聊天模板基于 `<|im_start|>`、`<|im_end|>` 等 `<|...|>` 结构。虽然自定义的 `<|style|>` 或 `<|镜头画风|>` 并非词表预注册的单个 special token（在 BPE 中切分为 `["<", "|", "概念", "|", ">"]`），但 Qwen3.5 天然将其识别为**系统级元属性定界符（Semantic Firewall）**。
   - 向量实测证实：在 22 种候选包装结构中，`<|...|>`...`<|/...|>` 结构的词法背景噪音最低，进入 DiT 前的净有效信噪比是传统逗号拼接的 **12.7 倍**、是普通 `Style: ...\nScene: ...` 的 **3.2 倍**。
2. **生图实测证实的四大核心优势**：
   - **消除中文/英文一逗到底长难句的"多头/多肢粘连"**：长句连写多个人物与怀抱动物/道具时易发生特征粘连（如猫长两个头、前景道具翻倍挡脸）；用 `<|概念|>「内容」<|/概念|>` 按模块切开后，跨区粘连直接归零。
   - **消除多层电影景深的"上下死黑边坍缩"**：当画面含"前景虚化遮挡 + 中景人物 + 远景街道"四层空间时，纯散文易把画面上下 2/3 涂成黑色信箱黑边；用 `<|foreground_blur|>` / `<|midground_focus|>` / `<|background_street|>` 分块后，各层景深自动填满整张画布。
   - **双人贴脸时的"冷暖灯光刀切级隔离"**：左人冷青霓虹、右人暖黄台灯（或左窗闪电蓝光、右脸烛火暖光），闭合标签能把各自色温死死锁在对应人物块内，绝不互染。
   - **支持"单模块热插拔"改图**：改灯光只改 `<|lighting|>` 行，改字幕只改 `<|subtitle|>` 行，不牵连其余模块构图。

### 17.2 五条硬性编写要求（违反任何一条会导致失效或副作用）
1. **要求一（按"实体/空间层"分块，严禁按"零散属性"分块）**：
   - ✅ **正确（一人一块，属性全包）**：同一个角色的**方位、发色、瞳色、服装、配饰、手中道具、专属灯光**必须全部关在同一个 `<|left_subject|>...<|/left_subject|>` 块内。
   - ❌ **错误（拆散属性）**：严禁写成 `<|hair|>左女白发，右女黑发<|/hair|>\n<|clothes|>左女红衣，右女蓝衣<|/clothes|>`——拆散必导致颜色交叉错位！
2. **要求二（块内必须保留显式空间方位词）**：
   - 标签名 `<|left_girl|>` 仅起边界隔离作用，**不能替代块内的空间介词**！块内正文开头仍必须明确写出 `On the left stands...` / `In the immediate foreground...` / `画面左侧...`。
3. **要求三（遵守从左往右"单向因果染色"排序）**：
   - 块与块的排列顺序固定为：
     **第 1 块**：全局画种/镜头/基础光影（`<|style_and_camera|>` 或 `<|环境氛围|>`）
     **第 2 块**：主视觉锚点人物 / 前景遮挡层（决定全图审美流形与主色温）
     **第 3 块**：次要人物（句首带显式反向阻断词 `In sharp contrast, NO...`）或中远景层
     **第 4 块**：画面内可见文字 / 底部字幕 / UI 容器（`<|subtitle|>` 或 `<|bottom_ui|>`）
4. **要求四（扁平、闭合、一行一块，控制在 3~5 块以内）**：
   - 每块独立占一行（末尾真实换行 `\n`），必须严格成对闭合 `<|tag|>...<|/tag|>`，**禁止块内嵌套子块**。
   - 英文提示词直接写 `<|tag|>English content.<|/tag|>`；中文提示词推荐写 `<|中文概念|>「中文内容。」<|/中文概念|>`。
5. **要求五（跨画种分块会触发"漫画分镜硬切线"，按需取舍）**：
   - 若在同一个提示词里写 `<|foreground_photo|>三次元全彩实拍咖啡桌...<|/foreground_photo|>\n<|background_manga|>二次元黑白网点漫画少女...<|/background_manga|>`，模型会在画面正中切出一条笔直的**上下漫画分镜线**（上格纯黑白漫画、下格全彩照片）。
   - **取舍法则**：想要"上下分镜对比图"就用 `<|...|>` 分块；想要"2D 动漫少女坐在真实 3D 咖啡桌后（融在同一个三维空间里）"，则必须合在同一个块内用连贯自然语言描述！

### 17.3 自包含可直接复用的三套标准模板

#### 模板 1：英文多层电影景深与双人强互斥模板
```text
<|camera_and_lighting|>Gritty 1990s Hong Kong cinema widescreen shot inside a dimly lit night diner, heavy 35mm film grain.<|/camera_and_lighting|>
<|foreground_blur|>In the immediate left foreground, an out-of-focus dark silhouette of a man's shoulder and a rain-streaked glass partition with neon reflections partially frame the view.<|/foreground_blur|>
<|left_subject|>In sharp focus on the left under cold cyan neon light sits a European teenage girl with curly blonde hair, round gold glasses, and light freckles, wearing a dark denim jacket and holding a silver camera.<|/left_subject|>
<|right_subject|>In sharp contrast, on the right under warm amber lamp light sits an East Asian teenage girl with straight jet-black bob hair, NO glasses, and spotless porcelain skin with zero freckles, wearing a cream knit sweater and stirring a glass of iced lemon tea.<|/right_subject|>
<|background_street|>Through the window in the far background, a blurry wet asphalt street glows with deep blue traffic lights and a passing red taxi.<|/background_street|>
```

#### 模板 2：中文视频帧/多主体防崩模板（`<|概念|>「内容」<|/概念|>`）
```text
<|环境氛围|>「黄昏时分的复古理发店室内实拍剧照，35mm胶片质感，暖金夕阳与室内霓虹交织。」<|/环境氛围|>
<|前景柜台|>「前景的玻璃柜台左下角放着一把银色剪刀和一瓶蓝色洗发水，景深微虚。」<|/前景柜台|>
<|左侧人物|>「柜台后方左边坐着一位满头银发、戴着老花镜、穿褐色背心的老理发师，正低头安静看报纸。」<|/左侧人物|>
<|右侧人物|>「柜台后方右边站着一位染着亮粉色短发、不戴眼镜、穿黑色皮夹克的年轻女孩，怀里抱着一只正常的单头橘猫，正对着镜子吹粉色泡泡糖。」<|/右侧人物|>
<|背景与文字|>「后墙上挂着红色的霓虹灯牌，清晰亮着“OPEN 24H”字样，画面底部印着黄色中文字幕：“今夜无人入眠”。」<|/背景与文字|>
```

#### 模板 3：LoRA / SFT 画师训练集 `.txt` 标注标准模板（最高信噪比格式）
当制作 MOSS-Image 的画师风格 LoRA 或 SFT 微调数据集时，每张训练图的 `.txt` 标注文件固定采用以下两行结构（不要加 `@`，不要在同一行混入 `anime` 等大类词，不要手写 `<|im_start|>` 外壳）：
```text
<|artist|>wlop<|/artist|>
A warm, softly lit 2D anime illustration inside a rustic wooden cabin. Holo from Spice and Wolf, a slender young girl with long reddish-brown hair, pointed wolf ears, and glowing crimson eyes, sits on a white bedsheet brushing her fluffy white-tipped wolf tail with a wooden comb...
```
