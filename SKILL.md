---
name: moss-image-prompt-skill
description: >-
  Compiles and debugs prompts for MOSS-Image (Qwen3.5-9B text encoder + 5B single-stream DiT)
  across three dimension modes — photorealistic (default), anime/2D, and cross-dimensional 2D-in-3D.
  Covers Danbooru tag cleaning, medium and lighting, block layout vs single paragraph, age and bone
  structure, expression and limb stability, multi-character separation, in-image text, gravure page
  layouts, game HUD screenshots, positive/negative templates and self-check, official Layout JSON,
  anime style channels, `<|概念|>` closed-tag blocking, focal hierarchy, multi-panel comic pages, and
  speech-bubble speaker attribution.
  适用于 MOSS-Image 出图提示词编写、Danbooru tag 转写实、分块与语序、年龄骨相锁定、多人防融合、
  二次元画风切换、JSON 版式、多分镜漫画与气泡归属。
license: CC-BY-NC-ND-4.0
version: 2.0.0
---

# MOSS-Image 提示词编译 SKILL

## 0. 任务定义与默认参数

**你要做什么**

- 你的任务：把用户给的 Danbooru tags / 参考图 / 模糊创作需求，经过动用全部算力与最大思考能力、逼近自身能力上限的精心设计，生成一对 `MOSS-Image` 提示词（Positive + Negative），并附采样参数。

**MOSS-Image 是什么**：它使用 Qwen3.5-9B 这个单向注意力模型作为文本编码器，把你的提示词进行编码之后，交给一个 5B 参数的 DiT 模型进行采样，最后生成符合你提示词要求的图片。

**booru tag 部分**：提示词的第一部分，内容是清洗后的 Danbooru tag。如果存在，必须放在提示词的顶部；一般情况下可以不用写。

**正文**：正面提示词里描述画面内容的那一段，不含开头的 booru tag 部分和开头那个画风词。

**大部分情况下，用户的需求是以下三种形式之一**：

- **一、默认：三次元真人写实**——启用 §1 第 5 条的漫符改写、§3 结尾锁真句与负面封杀组。
- **二、二次元**——插画风格或漫画创作：按 §15.2 的通道一～三处理，正文保持画风中立（定义见 §16.2）。
- **三、跨次元**——二次元的角色处在真实世界中，或者真实的角色处在一个二次元的世界里：按 §15.2 通道三·第 3 条处理。

用户的输入可能是以下三类的其中之一、其中之二，或者自由组合：

- 【Danbooru 原图 Tags】
- 【参考图】
- 【模糊创作需求】

**每一次提示词创作要遵循以下固定的步骤，不得跳步**：

1. §1 词条清洗
2. §3 确定图片媒介与场景光影
3. §4 设计图片构图
4. §5/§6 根据用户的需求确定人物的年龄与身材，并确保表情与肢体结构正确
5. §7/§8/§8.1/§9 特殊画面按对应的专项规则处理
6. §10 按输出模板生成最终提示词，并逐条自查

不得使用 §11 已证伪手法，不得自创本文件未列出的手法。

**本文件之后的每条规则都会标一个证据等级，共四级**：

- **A｜受控对照证实**：判定标准为存在 A/B 对照实验结果，且差异经人工复核确认；单变量替换的实验亦计入此级。
- **B｜单次观察复现**：只在单一场景观察到一次有效修正，未做对照组，在其他场景同样成立。
- **C｜机理假说**：为解释现象提出的推测，未经独立拆解验证。不得当作规则执行，仅供推理参考。
- **D｜禁用**：实测无效，禁止再当作手段使用（清单见 §11）。

**默认推荐参数**：以下是快速出图的默认参数。每次生成提示词时，若此前未向用户说明这组参数，则一并告知。

> 证据等级 A

- **分辨率**：面积固定 0.6 百万像素，比例按用户需求自拟。
- **Steps**：`16` 步
- **CFG**：`3.0`
- **SLG**：填 `single_layers` 这一栏，不要填 `double_layers`，默认 `22`。画面里没有文字时留空或跳过这一项；含义见 §13.1。

**如果想要文字精细，则选择如下参数：**

- **分辨率**：约 1.6 百万像素或更高，例如 `1024 × 1536`，比例仍按用户需求自拟
- **Steps**：`50`
- **SLG**：必须开启，默认 `22`，同样填 `single_layers`。

## 1. Step 1：输入词条清洗

如果用户的输入是一串 Danbooru Tag，则先执行以下清洗：

1. 保留角色名，例如 `lillie (pokemon)`、`higuchi madoka`。

   > 证据等级 B，效力边界见 §2。

2. 删除所有 Danbooru 画师名：模型对这类 Tag 完全不响应，纯属浪费 Token。同时删除容易引发漫展假发感的作品与公司版权大词，例如 `pokemon`、`nintendo`、`naruto`、`genshin impact`。

   > 证据等级 A

3. 删除打码词与会导致画面退回二维或三维渲染的词条：`censored, mosaic censoring, bar censor, 3d, animated, animated gif, game asset, visual novel cg, motion lines, request inset, english text, watermark, patreon`。

   > 证据等级 B

4. 正面不能出现 `cosplay, cosplayer, コスプレ`，图内印刷文字里也不许出现；应当把 Coser 当作「天生就长这样的真实人类」来描写。

   > 证据等级 C

5. 画面要求写实时，遇到下列二次元符号按对应写法改写，例如：

   - `anger vein`（十字青筋）：改用 `furrowed brows, glaring with defiant eyes`
   - `heart-shaped pupils / symbol-shaped pupils`（爱心眼）：改用 `glossy, watery dazed eyes`
   - `nose blush`（红鼻头）：改用 `faint pink blush on cheeks`
   - `long earlobes`（奇幻长耳垂）：改用 `silver punk earrings`
   - `genderswap / genderswap (mtf)`（性转，男变女会把骨骼画成男性）：改用 `delicate feminine features`
   - 以及其他同类的二次元符号，应当按同样的思路改写。

6. 除非用户明确要求，否则删除纯色背景词 `white / purple / grey / simple background`，应当替换为具体的真实场景及其光照，例如 `dimly lit bedroom, warm wooden bar counter, sunlit sandy beach`。

   > 证据等级 C

### 1.1 booru tag 部分

> 证据等级 A

**以下是根据用户输入的 booru tag 生成提示词时的经验法则，后续可能还会补充：**

1. 不要删除核心动作与人数 tag：
   - 画面中存在男方的躯干或手时，booru tag 部分不能写 `solo`，要保留 `1girl, 1boy, hetero`。
   - 用户输入的 booru tag 里的核心局部状态词（如 `school uniform, white shirt, blue skirt, rolled-up sleeves, hands on hips, wet hair`）要原样留在 booru tag 部分，不能只写在后面的分块正文里。

## 2. 角色名保留、画师名删除，以及少数例外
- **角色名**（如 `lillie (pokemon)`、`malfoid`）：保留。实测不会把画面拉回二次元，反而有助于还原原作的发型和神态特征。

  > 证据等级 A

- **Danbooru / Pixiv 圈内个人画师名**（如 `ie (raarami)`、`bartolomeobari`、`mika pikazo`、`ask (askzy)`、`kantoku`、`kawakami_rokkaku`、`yewang19`、`ciloranko`、`rella`、`liduke`）：彻底删除，也不要写成 `XXX style`，这条已经列入 §12 的禁用清单。训练语料里约 80% 由 Gemma 4 重新打标，只写肉眼可见的内容，圈内画师 ID 在那一步就被抹掉了；实测时单独写这类名字（加不加 `@` 都一样）与不写没有差异，等于浪费 token。

  > 证据等级 A

- **世界知名的大厂牌与大师名**：纯 2D 动漫模式下，实测单独写名字、不加任何形容词，只有极少数模型在预训练里就认识的名字会直接带来画风变化——`Studio Ghibli / Hayao Miyazaki`、`Hirohiko Araki / JoJo's Bizarre Adventure`、`Akira Toriyama / Dragon Ball`、`Ilya Kuvshinov`、`WLOP`。而 `Makoto Shinkai`、`Yoshitaka Amano`、`Kyoto Animation` 单独写几乎没有变化，必须同时写出它们具体的光影和笔触特征才生效。

  > 证据等级 A

- **作品／公司版权大词**（`pokemon`、`nintendo`、`naruto` 等）：删除，容易把画面画成真人 cos 的假发质感。

  > 证据等级 A

- **例外**：当正面提示词里出现多个动漫角色名，而服装又描述为原作专属的奇幻装束时，提示词的效力更偏向二次元而非三次元；这时可以选择直接删掉角色名。

  > 证据等级 B

## 3. 确定图片媒介与场景光影
根据画面的发色与机位，选用对应的摄影描述放在正面提示词第 1 句。注意：以下句子里的相机型号要和整句一起写，不能单独拆出来用；单变量实验证明，只写一个相机型号名（如 Contax T2、Hasselblad）不会让画面产生可辨认的差异，单独写相机型号名已列入 §11 禁用清单，真正起作用的是同句里写明的硬直闪、全景深、胶片颗粒、暗角等具体画面表现。

> 证据等级 A

- **如果画面含鲜艳二次元发色（粉发 `pink hair` / 鲜红发 `red hair`）且为室内近景**：选用 `raw handheld smartphone flash photo, shot on iPhone 15 Pro with direct camera flash, razor-sharp focus, deep depth of field`，用手机全景深硬直闪消除鲜艳发色容易出现的 2.5D 影楼假感。
- **否则如果画面为摇滚后台 / 家庭派对一地狼藉**：选用 `raw 1990s grunge snapshot taken with a Kodak disposable 35mm film camera, harsh direct on-camera flash with dark vignette, razor-sharp focus, deep depth of field`。注意：正侧面 `from side` 不要使用 `handheld camcorder`（手持 DV），否则这个词默认带出的俯拍角度会让人物身体诡异前倾。
- **否则如果画面为日系杂志内页 / 静谧凝视**：选用 `unretouched medium-format analog film photograph, shot on Hasselblad 500C/M with Kodak Portra 400, soft diffused natural daylight`，或改用杂志版式前缀。
- **其余情况**：选用 `raw candid 35mm film photograph, [填入机位如 high-angle first-person POV shot], razor-sharp focus, deep depth of field`。

**切换画风与质感时，开头、结尾与负面提示词要成套替换**
由于模型对孤立的相机或软件商标名不敏感，切换画风时要把正面提示词的开头、结尾和负面提示词成套替换：
- **档位 A（私房模式，默认主力）**：
  - 开头：`raw candid 35mm film photograph, harsh direct on-camera flash, razor-sharp focus, deep depth of field`，粉发或红发时换成 `iPhone 15 Pro smartphone flash photo`
  - 结尾：`Unretouched raw human skin texture with visible microscopic pores and sweat sheen under harsh direct flash, hard flash drop shadows, organic 35mm Kodak Portra 400 film grain.`
  - 负面提示词：`anime, 2d, illustration, cel shading, 3d render, cgi, doll, plastic skin, airbrushed`
- **档位 B（日系商业写真集柔润肤质，用于杂志与私房）**：
  - 开头：`high-resolution Japanese gravure photobook page layout, soft diffused natural bedroom window light, warm skin-tone shadow contrast`
  - 结尾：`Smooth, supple peach-like real human skin with a soft, luminous magazine gravure finish—delicately retouched for photobook perfection without looking like plastic.`
  - 负面提示词：在档位 A 基础上移除 `airbrushed`，追加 `harsh pores, rough skin, acne, overexposed, washed out, whiteout`
- **档位 C（纯黑白纪实与艺术摄影）**：
  - 开头：`Black and white 35mm analog film photograph, monochrome grayscale, high-contrast silver gelatin print`
  - 正文：删掉所有具体颜色词（如 `pink, amber, brown, fair`），全部改用明暗灰度描写
  - 结尾：`Unretouched monochrome human skin texture with microscopic pores, dramatic black-and-white shadow contrast, heavy silver-halide film grain.`

**按画面氛围替代画师名**
- 画面为唯美私房写真 / 少女卧床 / 纯欲凝视：写入 `Japanese gravure photobook aesthetic, soft luminous skin highlights, warm indoor bedroom lighting with rich shadow depth and natural skin-tone contrast`，用日系写真集画面与柔亮肤质高光替代原来的 `kakure eria`。
- 画面为强烈支配感 / 坏笑挑逗：写入 `raw 1990s grunge disposable 35mm film snapshot, harsh direct on-camera flash with dark vignette, high-contrast specular sweat and liquid highlights`，用 90 年代一次性胶片机的硬直闪与暗角高反差替代原来的 `bartolomeobari`。

  > 证据等级 B

### 3.1 表现暗室与逆光时要直接写出人物正面的暗部，防止正面发白
只写“夜间黑房间”仍会让人物正面惨白，要在提示词里直接写出暗部位置；详细机理见 §13.2。

> 证据等级 A

- **原因**：训练集人像以正面平光为主，具体比例未量化；如果正文中保留 `fair skin（白皙皮肤）`，即使开头写 `backlight / dark room`，人物正面依然会被补成均匀亮面。
- **具体做法**：当需要表现逆光剪影、夜间暗室或局部明暗对比时：
  1. **正面提示词要删掉的词**：从正文中彻底删掉 `fair skin, porcelain skin, bright` 等会提亮肤色的词。
  2. **正面提示词要写入的阴影描述**：
     - 开头写入：`dramatic low-key chiaroscuro lighting in a dark room, strong side-backlit silhouette with deep shadows across the front of the body`
     - 正文写明暗面位置：`The room is dark, lit ONLY by a single warm beam from the back-left. Instead of flat bright skin, the front of her face, chest, and stomach are submerged in deep, moody dark shadows, while a thin glowing rim-light traces the edge of her silhouette.`
     - 结尾写入：`realistic shadow falloff, zero flat front fill light`
  3. **负面提示词要加入的词**：在最前面加入 `flat front lighting, brightly lit front, evenly lit skin, overexposed skin, pale white skin, studio fill light, whiteout`。

### 3.2 用开头词组切换 12 种三次元摄影与电影风格，以及还原影视剧照的 5 类要素
三次元的摄影感与电影感取决于物理光学与显影冲洗参数，这些内容在 Gemma 4 重新打标时没有被抹掉。只要主体描述保持光影中立，不写 `fair skin`、`bright` 等会干扰光影的词，就不用改正面提示词结尾与负面提示词，只在开头放上对应的光学与显影词组，就能切换 12 种三次元摄影与电影风格：

> 证据等级 A

| 三次元摄影与电影风格 | 放在正面提示词开头的词组 | 关键画面要素与注意事项 |
| :--- | :--- | :--- |
| **1. 90s 一次性胶片硬直闪** | `raw 1990s disposable 35mm film snapshot, harsh direct on-camera flash, hard black drop shadow cast directly on the wall behind subject, dark corner vignette, heavy analog film grain, unretouched skin with visible pores` | 墙面硬黑投影 + 皮肤油光毛孔 + 四角暗角 |
| **2. 暗室单束光伦勃朗切割** | `dramatic low-key chiaroscuro photograph in a pitch-black dark room, lit only by a single narrow warm light beam from the side, half of face and body submerged in deep dark shadow, high shadow contrast, zero front fill light` | 正文不要写 `fair skin`；把半张脸与半身压入深黑阴影 |
| **3. 90s 港片王家卫霓虹抽帧** | `1990s Hong Kong arthouse cinema film still, lush saturated emerald-green and warm amber tungsten neon lighting, slow-shutter motion blur ghosting trails, glowing red halation, moody melancholic atmosphere, heavy 35mm film grain` | 翠绿+暖黄钨丝霓虹、慢门拖影残像、红色光晕（Halation） |
| **4. 银盐黑白暗房高反差** | `monochrome black-and-white analog silver gelatin print photograph, high-contrast dramatic grayscale lighting, deep crushed blacks and crisp silver highlights, coarse Ilford HP5 35mm film grain` | 要删掉正文中的具体颜色词（如 `red, blue, fair`） |
| **5. 落日逆光发丝金边眩光** | `warm golden hour backlit portrait photograph, intense low setting sun shining directly from behind, glowing golden rim light illuminating stray hair strands, natural optical lens flare, warm hazy sunset glow` | 逆光发丝金边 + 真实光学镜头眩光 |
| **6. 跳过漂白冷峻银灰电影感** | `gritty desaturated bleach-bypass cinema film still, crushed dark shadows, cold metallic silver-teal color grading, harsh overcast window light, raw unretouched micro-skin texture with pores` | 低饱和金属银青灰 + 深黑暗部 |
| **7. 赛博青/品红双色色片打光** | `nighttime neon portrait photograph, dramatic split two-tone colored gel lighting with electric cyan-blue light illuminating the left side of face and hot magenta-pink neon light on the right side, wet glass reflections` | 明确写出左右两侧脸的不同色片光源 |
| **8. 1880s 湿版蛋白印相老照片** | `antique 1880s sepia-toned wet-plate albumen print photograph, faded monochrome brown-yellow tint, heavy vintage chemical edge staining, dust scratches and paper creases, soft-edge Petzval brass lens swirl vignette` | 只写 `wet plate collodion` 无效；要写出药水边缘腐蚀黄斑与折痕 |
| **9. 1994 家用 VHS 磁带摄像机** | `1994 lo-fi home VHS camcorder video tape still, horizontal analog CRT scanlines, RGB chromatic color bleeding, blurry magnetic tape static noise, glowing green VCR OSD text reading "PLAY  1994.10.17" in the corner` | 扫描线 + RGB 色散 + 带出录像带画面的绿色 VCR 时间戳 |
| **10. 红外夜视 CCTV 监控录像** | `green monochrome night-vision infrared security camera footage, high-angle ceiling corner perspective, glowing phosphor-green IR illumination, glowing white infrared eyes, digital CCTV compression noise, timestamp "CAM-04 02:14:09"` | 只写 `CCTV` 无效；要写荧光绿红外 + **反光白瞳** + 俯拍时间戳 |
| **11. 日系空气感高调过曝胶片** | `airy Japanese overexposed pastel film photograph, high-key diffused soft window daylight, low-contrast milky lifted shadows, pale cyan-mint and warm cream color cast, dreamy luminous summer atmosphere` | 奶白色提亮暗部 + 薄荷奶油高调空气感 |
| **12. 好莱坞变形宽银幕青橙调** | `cinematic 35mm anamorphic widescreen movie still, classic Hollywood teal-and-orange color grading, deep teal-blue shadows and warm amber-orange skin tones, oval vertical anamorphic bokeh light orbs, horizontal blue anamorphic lens flare streak` | 青橙分级 + 垂直椭圆形焦外光斑 + 水平蓝色拉丝眩光 |

**还原三次元影视剧照、避免 AI 摆拍证件照感的 5 项描写内容**：
当还原真实电影或剧集截图且不依赖片名与演员名时，应当按以下 5 个方面组织描述：
1. **写出特殊机位与取景遮挡物**：例如车内行车记录仪视角要写明前景的仪表盘与雨刮器 `First-person dashcam POV looking out through a car front windshield with dark dashboard and black windshield wipers visible along the bottom edge`，隔窗拍摄要写明玻璃反光条 `shot through a horizontal glass window pane with a faint reflection bar across the lower frame`。
2. **在画面一侧加入虚化的前景遮挡物**：真实电影镜头很少让单人孤立居中摆拍，在画面一侧写明虚化的前景遮挡物，可以拉开前、中、后三层空间纵深：
   - 越肩后脑勺：`In the left foreground, the out-of-focus back of a short-haired man's head and black jacket shoulder in an over-the-shoulder framing.`
   - 虚化侧脸或门框遮挡：`In the right foreground, an out-of-focus side profile of a man wearing a light-blue surgical face mask partially frames the shot`，或 `framed tightly behind a dark vertical doorframe on the right`。
3. **写明灯具形态与面部受光位置**：写出灯具位置、颜色以及落在五官上的高光，例如 `two harsh glowing emerald-green fluorescent light tubes diagonal across the upper-left ceiling, catching his nose tip and forehead while his face is submerged in deep shadow`，或 `cool cyan-green overhead kitchen fluorescent tubes reflecting off his oily forehead and cheekbone`。
4. **写明年代色调与胶片颗粒**：例如单色暗紫 `ultra-low-key monochromatic purple-magenta cinema still`、50s 黑白银盐 `1950s black-and-white 35mm Hollywood film still`、暖金钨丝灯 `warm golden-amber tungsten-lit 35mm period romance close-up`。
5. **加入打破精致感的日常小道具**：加入消除网红精致感的真实生活细节，例如嘴角叼牙签 `a toothpick sticking out of the corner of his mouth`、凌乱碎发、旧皮马甲、墙上的中文印刷告示纸。

## 4. 按画面复杂度决定分块还是生成单段短文
> **前面的词会影响后面的词，后面的词确难以影响前面的词语，语序不能颠倒**：
> Qwen3.5-9B 编码器采用从左到右的单向因果注意力，后面的词对前面的词在 TE 层的注意力权重严格为 0，仅靠 2 层浅 `text_embedder` 回传，信号衰减 66%~77%。实测前后语序对比证实：
> 1. **光影与色温词要放在人物与衣物名词之前**：如果先写 `white shirt`，末尾才补 `lit only by an overhead green neon tube`，模型只会画出一根绿灯管，而白衬衫与脸部仍保持暖白平光；只有把绿灯与黑夜写在句首，后面的衬衫与皮肤才会被染上绿色环境光。
> 2. **双人或多人场景里，第一段描述的角色会影响全图的人种与画风**：排在第一段描述的角色，会顺着从左到右的注意力方向，把她的人种骨相、年龄感以及欧美胶片风、日系柔光萌系风这类摄影风格自动带给排在第二段且未明确写明这些属性的角色。因此要把**主要角色放在第一段**，并在第二段开头直接写出相反的特征，例如 `In sharp contrast, the girl on the right has NO glasses, spotless porcelain skin with zero freckles...`，切断第一段传下来的雀斑、配饰或人种特征。

> 证据等级 A

**选择使用 `<|概念|>...<|/概念|>` 闭合标签分块还是生成单段连贯短文**
根据动作复杂度与画面层次，从以下两种写法中二选一，闭合标签的详细语法见 §16：

- **如果画面属于以下情况，不要分块，要生成 1 段 85 词以内的连贯短文**：
  - **哪些画面要写成单段短文**：画面是**全身双人紧密交叠**（如 `full body, hug, carrying` 双人背抱，或 `grappling` 近身缠斗），或**单人胸上特写怼脸**，或**需要 2D 人物与 3D 实景自然融在同一物理空间内且不切分镜线**。
  - **怎么精简内容与组织句子**：砍掉 60% 次要背景杂物，砍掉男方裤子拉链、鞋子与头部位置描写，用 4 句连贯的自然语言写完，防止分块导致人物上下半身断裂或被切出漫画分格线。

- **其余情况（默认主力写法）：使用 `<|概念|>...<|/概念|>` 闭合标签按行分块，替代旧版 `[Image Composition]`**：
  - **为什么用闭合标签分块**：Qwen3.5 在预训练阶段熟悉 ChatML 的 `<|...|>` 定界符结构。使用 `<|区块名|>内容<|/区块名|>` 一行写一块，能把跨区块的词语互相干扰压低 50% 以上，避免一逗到底的长难句导致局部多头或多肢粘连、四层景深坍缩成上下大黑边，以及双人贴脸时冷暖灯光互串。
  - **画面有多层电影景深、前景遮挡、单人易出现透视畸变或高脚凳悬空时，按空间前后或画面上下分块**：
    - 按空间深度或画面纵轴切分为每行一个闭合块，例如：
      - `<|camera_and_lighting|>...<|/camera_and_lighting|>`
      - `<|foreground_blur|>...<|/foreground_blur|>`，或 `<|upper_frame|>...<|/upper_frame|>`
      - `<|midground_subject|>...<|/midground_subject|>`，或 `<|center_frame|>...<|/center_frame|>`
      - `<|background|>...<|/background|>`，或 `<|lower_frame|>...<|/lower_frame|>`
    - 如果画面有翘臀（`top-down bottom-up`），臀部要放在 `<|center_focus|>`，不要放在前景块，防止把屁股画得比头还大；
    - 如果画面有脚底（`soles`），脚要放在 `<|lower_frame|>`，防止广角透视把脚画得过大；
    - 如果画面是高脚凳悬空腿（`dangling legs`），要在 `<|lower_frame|>` 写明 `legs hang freely in mid-air well above the floor without touching the ground`，并在负面提示词中加入 `feet on floor, standing, touching the ground`；
    - 如果画面有半脱衣服（如 `shirt lift, skirt pull`），要直接写成已经撩起或拨开的静态结果，例如 `shirt already pushed up, skirt hem already lifted`，不要用拉扯动词占用双手，防止多画出第三只手。
  - **画面有 2~6 人同框或冷暖强对比灯光时，按人物分块防止特征与灯光互串**：
    - **要按人物分块，不要按身体部位或颜色分块**：把同一个人的位置、发色、瞳色、服装、配饰和各自受光全部写在同一个闭合标签里：
      - 2 人并排：`<|left_subject|>...<|/left_subject|>` 加 `<|right_subject|>...<|/right_subject|>`，中文提示词可用 `<|左侧人物|>「...」<|/左侧人物|>` 加 `<|右侧人物|>「...」<|/右侧人物|>`；
      - 4 人围坐：按 `<|top_left|>`、`<|bottom_left|>`、`<|top_right|>`、`<|bottom_right|>` 分行闭合；
      - 6 人沙发梗：`<|center_sofa|>...<|/center_sofa|>` 加 `<|standing_behind|>...<|/standing_behind|>`；
    - **多人贴近时防止画成同一个人**：在顶部的 booru tag 部分或开头加上 `distinct hair colors, contrasting features`；把主要角色放在第 1 个人物块，由她给全图定调；第 2 个及之后的人物块要在句首直接写出与前一人相反的特征，例如 `In sharp contrast, the girl on the right wears NO glasses, has spotless porcelain skin with zero freckles...`，并在负面提示词中加入 `twins, identical faces, same hair color, attribute bleeding`。

## 5. 控制人物的年龄、骨相与身材
根据目标角色的年龄调整五官骨相与体型描写，避免把青春期少女画成成熟女性、幼童或偏胖体型：

> 证据等级 A

1. **控制青春期少女的年龄感与纤细身材，避免画老、画幼或被画面文字带胖**：
   - **目标为 14~16 岁青春期少女时**：
     - **正面提示词不要写显老与丰满词**：不要写 `woman, gorgeous, makeup, wavy long hair, wide hips, curvy, C-cup` 以及白人成年女性骨相词 `bone structure, defined nose bridge, deep-set eyes`，否则容易把人物画成 25 岁成年女性或 130 斤的偏胖体型。
     - **正面提示词不要叠加显幼词**：不要同时叠加 `very young, very petite, babyface, round chubby cheeks, button nose, budding breasts`，图内印字不要写 `"LOLITA"` 或 `"148cm"`，否则会把人物画成 6~10 岁幼童。
     - **正面提示词常用写法**：
       - 东方清纯少女：`slender 15-year-old Japanese teenage schoolgirl, cute pure youthful face, short mid-face, bare-faced, zero makeup, small mole under eyes, clear light-amber eyes, natural fine straight jet-black bob hair with thin wispy bangs, small perky A-cup breasts`。
       - 欧洲雀斑少女：`slender 14-year-old European teenage girl, natural youthful face, softly rounded full chin, soft cheeks, straight nose bridge, light freckles across nose and cheeks, low straight eyebrows, deep olive-hazel eyes, zero eyeliner, zero makeup, pulling and biting lower lip inside mouth in a crooked cheeky smirk, side-parted light golden-brown twin braids with small sky-blue butterfly hairclips clipped high above ears, small perky A-cup breasts`；负面提示词要加入 `pointy chin, V-shaped face, heavy eyeliner, spiky fake eyelashes, mascara`。
   - **防止画面印字把身材带胖**：如果在杂志页面上印 `"AGE : 19"`，正文中描述身体时要明确写上 `very slender, petite skinny teenage frame, flat stomach, visible collarbone, small perky A-cup breasts, thin slender thighs`，并在负面提示词中加入 `fat, chubby, plump, curvy, thick thighs, C-cup, large breasts`。

2. **正面提示词不要写 `flat chest`**：
   - 描写 15 岁纤细少女时使用 `small perky A-cup breasts`，正面提示词不要写入 `flat chest`；`flat chest` 应当保留在负面提示词中，防止胸部失去立体光影起伏。

  > 证据等级 B

3. **全身镜头要写明双脚与地面的关系，防止头身比画成儿童**：
   - 全身镜头要保留双脚与地面或支点的关系描述，或者明确写出双腿悬空，并保留鞋袜词；删掉脚部与支点的描述会让头身比缩水成五头身儿童。

  > 证据等级 A

## 6. 控制头发质感，确保面部表情与肢体结构准确
描写头发、面部神态与手脚局部时，按以下写法避免假发感、白油漆感及肢体畸变：

> 证据等级 A

1. **写出真实发丝细节，消除假发感**：
   - 所有头发描述都要加上：`natural fine [发色] real human hair, thin wispy bangs, loose flyaway strands`。
2. **控制脸红程度、卧床神态与遮眼闭眼状态**：
   - **脸红**：只写 `faint pink blush`，不要写 `deeply flushed, burning red`，防止把脸画成通红的关公脸或酒糟鼻。
   - **趴枕头脸**：不要写 `face buried squishing cheek + rolling eyes back showing white sclera`，这种写法在三次元画面里会让脸部严重变形；应当写 `resting sideways against a soft pillow without squishing her cheek, looking lazily through half-closed eyes `。
   - **遮住眼睛或熟睡闭眼时不写眼神动词**：当画面存在 `hands covering eyes`（双手遮眼）或 `sleeping / fast asleep`（全员熟睡不看镜头）时，正文中不要出现任何眼神动词，例如 `stares, wide-eyed, looking up, gazing`；正面提示词写 `completely covering and hiding both of her eyes from view` 或 `eyes completely closed, utterly unaware of the camera`，并在负面提示词最前排加入 `visible eyes, open eyes, pupils, looking at viewer, eye contact`。
3. **调整机位与焦平面，保证腿部、脚趾和手势清晰**：
   - 两人身体重叠时，不要使用纯侧面 `from side`，改用斜角或正面折叠，防止多条腿重叠融合；
   - 露出脚趾时，砍掉远端次要细节，加上 `five distinct, natural toes clearly visible`；
   - 竖中指（`middle finger`）不要放在 `Foreground` 区块，防止失焦模糊，应当写 `right beside her cheek in the exact same sharp focal plane, in crisp focus`。

### 6.1 写双色发角色时要分清主发色与发梢颜色
当双色发（`two-tone hair`）角色同时出现 `blonde hair, pink hair, two-tone hair`（如 `kitagawa marin`）时，主发色是金发、发梢是粉发，应当写为 `blonde hair with pastel-pink tips`，不要写反成满头粉发。

> 证据等级 B

## 7. 多人同框时按位置分区并写明各自差异，防止人物画成双胞胎
**2～6 人同框时按位置分区并写明差异特征**

> 证据等级 A

- **按人数划分画面区域**：
  - 2 人并排：分为 `Left Focus` 与 `Right Focus`；
  - 4 人围坐：分为 `Center Reference` 加 `Top-Left` / `Bottom-Left` / `Top-Right` / `Bottom-Right` 四象限；
  - 6 人沙发梗：分为 `Center Foreground on Sofa`（1 个娇小角色）与 `Standing Behind the Sofa`，后者按 `Far Left / Mid-Left / Center-Back / Mid-Right / Far Right` 依次列出 5 个高大丰满角色。
- **写明每个角色的差异特征**：多人贴近时，booru tag 部分或开头加上 `distinct hair colors, contrasting features`，每个人在各自区块内明确写出截然不同的发色、人种与特征，例如左侧女性写 `European girl with visible freckles`，右侧女性就要明确写 `Japanese girl with spotless porcelain skin and zero freckles`，并在负面提示词中加入 `twins, identical faces, same hair color, attribute bleeding`。

**2～10 人同框时的开头群像句式**
- **开头写入官方示例群像句**：当画面包含 2 个及以上角色（如双姝夹击、四人温泉、六人沙发梗、十人毕业照）时，建议在正文开头第一句写入官方公开示例中的人数与构图句：`"Featuring exactly [N] distinct girls, each appearing once, in an asymmetric ensemble composition (not a rigid grid)."`。注意这是官方公开示例的写法，不等于训练语料本身的标注句式。

  > 证据等级 B

- **配合分块差异描述与负面提示词**：同时在每个角色的独立区块内明确写出相反的特征，例如左侧女性写 `European girl with visible freckles`，右侧女性明确写 `Japanese girl with spotless skin and zero freckles`，并在负面提示词中加入 `twins, identical faces, same hair color, clone`。

- **多人分块时要在各自区块内写全发色与发型**：当为了防止串色，把发色与发型词（如 `blonde hair, twin braids` 与 `black hair, bob cut`）从顶部的 booru tag 部分移入下方的 `Left Focus / Right Focus`（新的 `<|left_subject|>`、`<|right_subject|>` 闭合标签同理）时，**要在各自的段落内完整写出该角色的发色与发型**，例如左边写明 `bright honey-blonde hair braided into two loose twin braids`，右边写明 `natural fine straight jet-black bob hair with thin wispy bangs`，不要在移动时漏写发色或麻花辫。

  > 证据等级 B

## 8. 按画面内文字密度设置写法与采样参数
- **用英文双引号括起要显示的文字**：画面里要生成的每个字符串都用英文双引号括起，需要换行处直接换行。

  > 证据等级 A

- **按字号与文字密度切换采样参数**：
  - **如果画面是整页高密度小字**（如满版 A4 高考试卷、多栏密集杂志正文）：要切换到 SLG `"22"` + Steps ≥ 50 + 分辨率 ≥ 1M，否则低分辨率下小字会模糊。
  - **如果画面是中等字号 UI、字幕或标题**（如 Galgame 对话框台词、TV 番剧底部字幕、SSR 卡牌标题与攻防数值、海报大字）：无需开启 SLG 或高步数，在 `SLG=""`（置空）+ 16 步 + 0.6 百万像素下即可清晰无错字地生成。

  > 证据等级 A

- **单页文字过多时同步调高字号与分辨率**：单页塞满整张 A4 的题目量时，字号与分辨率要同步调高，否则经 VAE 解码后文字会模糊。

  > 证据等级 B

- **防止图内印字改变人物年龄与体型**：图内印 `"LOLITA"`、`"148cm"` 会把人画成幼童；印 `"AGE : 19"` 会连带把身材画胖，要在正面提示词正文中明确写上纤细体型词，并在负面提示词中排除肥胖词。

  > 证据等级 A

### 8.1 生成日系写真集内页时的镜头、神态与肤质写法
生成日系写真集（Gravure Photobook）或杂志内页时，应当使用本节的开头固定句，不要使用 §3 的哈苏分支；相机型号名本身作用很弱（见 §13.2），两种写法不要叠加，应当二选一。

1. **开头使用日系写真集版式与自然窗光，不用哈苏相机**：
   - 制作日系写真集内页时，不要使用带有复古沉重感的 `Hasselblad 500C/M`。
   - 开头固定使用：`high-resolution Japanese gravure photobook page layout, full-bleed vertical bedroom photograph framed by a clean thin white magazine border... soft diffused natural bedroom window light`，用床单与窗户的漫反射光营造轻盈、私密、充满少女体香的现代空气感。

  > 证据等级 A

2. **写出微乱发丝与带羞涩诱惑感的眼神，不要写呆板整齐的形容词**：
   - **发型与配饰**：在床上或私密场景中，不要写 `neat`（整齐的），应当写为 `loose twin braids, airy wispy bangs, delicate flyaway strands spread across the soft pillows`（松散微乱的发辫与透气碎发），并主动保留角色的柔美小配饰，如莉莉艾的 `white flower hair ornament`。
   - **眼神描写**：不要写缺乏性张力的 `looking gently`，应当使用兼具羞涩与勾引感的复合词：`shy seductive gaze`（害羞又勾人的纯欲眼神）或 `watery, flustered gaze full of reluctant submission`（湿润动摇、欲迎还拒的顺从眼神）。

  > 证据等级 B

3. **补充微距内衣的通透材质与结尾的柔润皮肤质感**：
   - 描述极限微距内衣时，要写出材质通透感与纤细对比：`tiny sheer white lace flower pasties connected by hair-thin gold strings`（由发丝般极细的金线连接的半透明白蕾丝小花乳贴）。
   - 正面提示词结尾的皮肤质感改用：`Smooth, supple peach-like real human skin with a soft, luminous magazine gravure finish—delicately retouched for photobook perfection without looking like plastic`。

  > 证据等级 B

4. **白色床单或白丝场景不要叠加 `bright background` 或 `daylight`，防止画面过曝发白**：
   - 当画面中包含大量白色元素（如 `white magazine border, white pillows, white lace, white stockings`）时，不要写入 `bright background` 或笼统的 `daylight`，否则多个白色与高亮词叠加会让全图过曝发白、失去立体感。
   - **具体写法**：要使用柔和暖调室内光与丰富的明暗层次对比：`warm indoor bedroom lighting with rich shadow depth and natural skin-tone contrast`，并在负面提示词中加入 `overexposed, washed out, faded colors, overly bright, pale flat lighting, whiteout`。

  > 证据等级 A

## 9. 生成游戏实机截图时要在四角写出具体界面元素
如果画面需要表现游戏实机截图与 HUD（抬头显示界面），按以下范围与结构生成提示词：

> 证据等级 A

- **模型支持与不支持的游戏画面类型**：
  - **支持生成的画面**：四角游戏 HUD 界面生成、三渲二卡通游戏截图（如 `Breath of the Wild` 赛璐璐水彩风）。
  - **不支持生成的画面**：模型不支持只靠提示词生成 UE5 或 FF7 重制版那样的次世代写实 3D 多边形渲染人物，强行写 `Unreal Engine 5 / 3D character model` 会退化为廉价磨皮塑料人偶或 2.5D 厚涂画。如果用户要求写实风游戏角色，应当直接改用 35mm 胶片或手机直闪真人实拍写法，不要使用 `3D render / CGI / Unreal Engine`。
- **如果用户明确需要带游戏 UI 或 HUD 的实机截屏画面**，应当按以下 4 步结构生成正面提示词：
  1. **开头填入截图与游戏名称标签**：`screenshot, video_game_capture, [genre_tag], [game_name].`
  2. **第一段写明实机游玩的机位，排除宣传图画风**：`A cinematic 16:9 gameplay screenshot from [Game Name], rendered with the game's [style] aesthetic rather than as promotional artwork. Captured from an immersive third-person over-the-shoulder gameplay camera...`
  3. **第二段写出画面四角的 HUD 部件与具体文字**：
     `Preserve the recognizable in-game HUD overlay: in the top-left corner, a status_ui with a red health_bar and blue stamina_bar; at the top-center, a horizontal compass bar with crisp letters "W", "NW", "N"; in the bottom-left corner, an equipment_slot icon with weapon_skill label reading "[Skill Name]" and quantity "3"; in the bottom-right corner, a numeric counter reading "1248".`
  4. **结尾写明不出现标题标志与宣传字样**：`No title logo, desktop borders, subtitles, or promotional text; this is an immersive moment of ordinary exploration captured directly from gameplay.`
- **不要省略四角 HUD 的具体部件名与引号内数值**：写出四角 HUD 具体部件与数值时才能生成游戏实机画面感，去掉 HUD 只保留文字声明则没有效果；要表现游戏画面感，HUD 的具体部件名与引号内的数值都不能省略。

  > 证据等级 A

## 10. 按模板生成正面与负面提示词并逐项自检
每次生成提示词时，都要包含以下正面提示词与负面提示词结构：

> 证据等级 A

#### 按闭合标签分块生成正面提示词的标准模板
默认采用 §4 分支 B 的 `<|...|>` 闭合标签分块写法生成正面提示词：
```text
<|style_and_camera|>[Step 2 摄影设备/画种元数据], [景别与机位], razor-sharp focus, deep depth of field, [角色名], [清洗后的核心 Danbooru Tags], uncensored.<|/style_and_camera|>
<|upper_or_left_focus|>[方位 1 显式声明如 On the left / In the upper frame]: [主体 1 的骨相、真发、表情、专属受光色温、静态衣物结果与动作]...<|/upper_or_left_focus|>
<|lower_or_right_focus|>[方位 2 显式声明如 In sharp contrast, on the right / In the lower frame]: [主体 2（含反向因果阻断词）或两主体接触部位的具体几何，如握手、背抱的接触点]...<|/lower_or_right_focus|>
<|texture_lock|>Unretouched youthful fair skin texture with microscopic pores (若为杂志柔焦写真则换为 Smooth, supple peach-like real human skin with a soft magazine gravure finish), real human hair strands in sharp focus, realistic wet liquid specular highlights, amateur direct flash photography, organic 35mm analog film grain.<|/texture_lock|>
```
如果画面属于 §4 分支 A 的全身紧密交叠或跨次元同空间自然融合，就不要使用 `<|...|>` 标签，直接生成 4 句、85 词以内的连贯自然语言短文；旧版 `[Image Composition]` 写法仍兼容保留，可作为备选。

#### 按固定模板生成负面提示词并追加场景排除词
每次生成负面提示词时都要包含以下基础词条，并根据具体场景在最前面追加对应的排除词：
```text
[场景专项封杀词：如遮眼加 visible eyes / 高脚凳加 feet on floor / 多人加 twins, identical faces, same hair color / 换装扒衣加原图旧衣物词], child, toddler, preteen, loli, 6 years old, 8 years old, 10 years old, babyface, flat chest, mature female, adult woman, milf, old, 22 years old, 25 years old, 30 years old, high cheekbones, hollow cheeks, long face, deep-set eyes, pointy chin, V-shaped face, heavy eyeliner, spiky fake eyelashes, mascara, heavy makeup, fat, chubby, plump, wide adult hips, cotton, foam, whipped cream, thick white paint, opaque white blob, red face, burning red cheeks, sunburn, rosacea, tomato face, cosplay, cosplayer, wig, synthetic hair, cosplay wig, stiff hair, thick blunt bangs, colored contact lenses, blurry, out of focus, soft focus, lowres, censored, mosaic censoring, bar censor, anime, illustration, 2d, 3d render, cgi, doll, plastic skin, ugly face, distorted face, squished cheek, grimace, creepy eyes, worst quality, bad anatomy, fused bodies, fused legs, extra limbs, extra arms, extra legs, extra hands, three hands, extra fingers, mutated hands, fused fingers, six fingers, extra penis, detached penis, watermark, text
```

#### 输出前逐项自检

把提示词发给用户之前，内部核对以下五项，五项都成立才输出：

1. 正面提示词里是否已经剔除版权大词（`pokemon / naruto` 等）、`cosplay` 和打码词？
2. 衣服拉扯是否写成静态结果——不能出现因为又要掀衣服而需要三只手的情况？
3. 画面如果有遮眼或熟睡，正文里是否删光了 `look / stare / wide-eyed` 这类动词？
4. 对象是 14~16 岁少女时，是否剔除了 `woman / curvy / bone structure / deep-set eyes / wavy hair`（避免画老画胖）和 `babyface / very petite / budding`（避免画成幼童）？
5. 两人以上同框时，是否按方位切块、给每个人指定了互斥的发色、肤色与有无雀斑，并在负面提示词加入 `twins, identical faces, same hair color`？

## 11. 实测无效的禁用清单
本节列出的做法均属证据等级 D，实测无效，不要使用：
1. **单独写相机、胶片或工艺名**（`Contax T2`、`Fujifilm Pro 400H`、`Polaroid SX-70`、单独写的 `Wet plate collodion` 或 `CCTV footage`）：单变量替换后画面没有可辨认的差异；但只要改写成对应的物理显影与光学画面表现，例如相纸边缘药水黄斑折痕、荧光绿夜视反光白瞳加时间戳（见 §3.2），就能 100% 生成对应的媒介效果。

  > 证据等级 A

2. **单独写渲染引擎商标或材质术语名**（`Unreal Engine 5`、`Octane Render`、`Blender Cycles`、`Subsurface scattering`）：单独出现不会生成对应风格，最多拼出模型自己理解的近似质感。

  > 证据等级 A

3. **要求生成写实次世代 3A 游戏的 3D 渲染人物**（FF7 重制版 / 法环级 PBR 建模感）：这是底座模型的训练盲区，强行写 `3D render` / `CGI` 会退化成磨皮塑料人偶或 2.5D 厚涂；要生成写实游戏角色，只能按真人胶片实拍的写法来写。

  > 证据等级 A

4. **在正面提示词里把角色名、IP 与画师名一概删掉**：这种做法是错的。角色名应当按 §2 保留，只有画师名与版权大词需要删除。

  > 证据等级 A

5. **保留 150 词写实毛孔与闪光灯正文，只靠单个 `cel shading` / `CCTV footage` 切换画风**：只改这一个词不会起作用，因为会被后面 150 词的写实毛孔与闪光灯描述压制。如果主体描述保持画风中立，不写毛孔、闪光灯、胶片这类词，就不用修改正面提示词结尾与负面提示词，只替换开头 8~15 词的 Danbooru 画种、年代与技法词组，就能切换整张图的画风（见 §16）。

  > 证据等级 A

6. **用 SLG 填 `"22"` 还是置空 `""` 来切换插画与写实画风**：当提示词为二次元或动漫描述时，即使 `SLG=""`（置空）也会生成纯正的二次元插画，不会变成三次元真人。SLG 的真实作用是让高频几何边缘收敛、减少局部粘连，并不是二次元与三次元的切换开关。

  > 证据等级 A

7. **正文里留着强烈的画风或颜色词，却只改开头 1 个词来跨次元切换画风**：当正文已经包含写实毛孔或具体颜色词这类把画风、色彩定死的描述时，开头的单个词无法覆盖正文；要先删掉正文里互相冲突的词，例如生成黑白漫画或用 JSON `palette` 控制配色时，要删掉正文里的 `red eyes, navy` 等自然语言颜色词（见 §16.2）。

  > 证据等级 A

8. **对模型不认识的个人画师写 `XXX style`，或用五官与高光的几何描述硬凑个人画师的画风习惯**：模型对预训练未收录的个人画师名（如 `kawakami_rokkaku`、`yewang19`、`ciloranko`）不响应，不要写 `XXX style`；也不要用“四角星高光、斜杠腮红、贴纸白边、半垂眼”这类零碎的几何描述去硬凑某位画师的脸型，否则模型只会按字面意思机械拼贴成僵硬的“拼图脸”，还原不出画师的笔触（见 §16.1）。

  > 证据等级 A

## 12. 解释实验现象的原因推测
以下内容均为对实验现象的推测，尚无独立拆解证明，属于证据等级 C，不作为规则执行：

> 证据等级 C

- 模型只有一条文本条件通道：技术报告 Appendix A.2 指出 `Caption, structural, and metadata fields are combined before applying the Qwen chat template and tokenization.` 所谓“元数据字段”并不是独立入口，任何名词能否起作用，取决于它在训练 caption 里的共现频率（见 §14.3）。
- 训练 caption 由 VLM 批量重新打标生成，技术报告 §3.2 点名使用了 **gemma4**，词表高度同质化：打标器只写肉眼可见的颜色、明暗、反光、笔触，不写器材型号与软件名。因此直接写出肉眼可见的画面效果，比只写器材或工艺名称更有用。一手出处见 §13.2。
- 训练集里的人像默认偏向正面平光：如果不明确写出暗部位置，模型就会按训练集的平均情况给人物正面补光。
- 单个 token 在 150 词的长提示词中权重占比过低，只有约 2%，不足以改变整段提示词定下的画面走向；把正面提示词的开头、结尾与负面提示词成套替换才有效。

---

## 13. 对照官方规范并说明本文件的采样参数

### 13.1 先确认本文件规则所依赖的固定采样参数
阅读或使用本文件前请先看这一段：本文件的全部规则都建立在如下**固定采样点**上：面积固定 0.6 百万像素（比例按需求自拟）、Steps `16`、CFG `3.0`、Shift `5.0`、Euler、SLG（`single_layers`）**置空**。

| 参数 | 官方推荐 | 本文件固定值 | 与官方推荐的差异 |
|---|---|---|---|
| 分辨率 | 1M / 1.5M / 2M（可外推 0.25M–4M） | 0.6M | 低于推荐下限，速度优先 |
| Steps | ≥ 50 为最佳 | 16 | 减到约三分之一 |
| CFG | 3.0（区间 3.0–6.0） | 3.0 | 一致 |
| Shift / Sampler | 5.0 / Euler | 5.0 / Euler | 一致 |
| SLG | 开启 Two-Pass `uncond skip` layer 22（22/24/26 皆可） | 置空（关闭） | **有意关闭**换写实感 |
| Negative | `nsfw, worst quality, bad anatomy, watermark, jpeg artifact` | 数百词长负面提示词 | 远超官方建议量级 |

官方 USERGUIDE 原话：`Turning off SLG will make the output image look very realistic, but it may also cause some details to collapse.`
因此本文件里整套防止肢体与五官崩坏的词条（§5、§6）是**针对“关闭 SLG + 低步数”导致细节崩坏所做的人工补偿**，不适用于开启 SLG 的高分辨率、高步数档位。
`single_layers` 是 ComfyUI 端 SkipLayerGuidance 的跳层参数。官方只给出“关闭更写实、但可能细节崩坏”与推荐 layer `22`/`24`/`26`，**并未**说某个 layer 固定对应某种画风。在动漫提示词下，`SLG=""`（置空）同样生成纯 2D 动漫，所谓“置空 = 写实”只在写实提示词下成立；SLG `"22"` 的真实作用是改善窗框、书页、百褶裙等高频几何边缘的收敛度。

该节点另一栏 `double_layers` 对应「正面提示词那一路也跳层」的模式：官方源码 docstring 里叫 `slg_layers`，会额外增加一次跳过 block 的正向前向。官方 USERGUIDE 的用法是把 `double_layers` 留空、只填 `single_layers`，本文件每次出图也都把 `double_layers` 置空，所以这一栏的效果我们没有单独实测过。

> 证据等级 A

### 13.2 为什么直接写出肉眼可见的画面效果比只写器材或引擎名称更有用
本节为 B+ 级原因说明，官方打标规范的两处一手原文如下：

> 证据等级 B+

- USERGUIDE Image Captioning：`Include only information visible in the image.` 所列描述维度为 `composition` / `key subjects` / `actions` / `setting` / `colors` / `lighting` / `artistic style`。
- 技术报告 §3.2：`We caption images with gemma4 at different semantic granularities: a short sentence, a medium paragraph, or a detailed description of composition, subjects, scene attributes, and visible text.`
两份清单里**都没有器材型号、胶片型号、渲染器条目**——重新打标的模型（Gemma 4）从不被要求写这些内容，所以训练后的模型也没有学会靠这些名称生成对应质感。
- 相机型号、胶片型号、渲染引擎名属于画面上看不见的推断信息，不会进入训练 caption，因此模型没有学会把型号名与画面质感对应起来。§12 禁用清单第 1、2、5 条就是这个原因。
- 技术报告 §3.2 指出：存在原始 caption 时，训练会按 **80% 的比例采样 Gemma 4 生成的 caption**，20% 用原始 caption；Danbooru 画师 tag 不是可见信息，所以在训练语料中被系统性稀释，这也是 §2 要求删除画师名的原因。
- 技术报告 Table 12 显示：训练时 `meta` 按 p=0.1、`canvas`/`caption` 按 p=0.5、可选字段按 p=0.5 随机丢弃，而 `gen_brief` / `elements` 基本保留；因此元数据词对画面的控制力天然弱于正文描述，所以本文件要求把正面提示词的开头、结尾与负面提示词成套替换，而不是只改单个词。
- 技术报告 §3.1 指出：数据不做质量过滤，水印与伪影应当写进文本监督（`annotate rather than filter`）；这解释了为什么单靠负面提示词压不住问题，要在正面提示词里写出想要的画面效果。

### 13.3 对照官方结构化写法与本文件写法的差异
1. 本文件用 `[Image Composition]` 加自然语言方位词划分画面区域；官方部署规范是 JSON `summary/items` 加 `bbox_coord_space:"abs_centralized"` 与 `bbox_format:"yxyx"`（中心原点绝对像素）。技术报告 Appendix K 称中心原点坐标在未见分辨率上泛化更好。
2. 官方 `must_not` 上限为 3 条具体排除项；本文件使用数百词的负面提示词，长度未做消融测试。
3. 官方示例 `assets/images/004` 中防止多人撞脸的原句（`each appearing once` / `not a grid`）已在 §7 引用：原文属实，但我们未对该句式做单独对照。

  > 证据等级 B

4. 官方 `summary.meta` 可容纳作品名，例如 `assets/images/011` 中的 `["screenshot","video_game_capture","dark_fantasy","elden_ring"]`；但所有字段最终拼接后走同一条 Qwen chat template 文本通道（技术报告 Appendix A.2），不存在独立的元数据嵌入，因此任何名词能否起作用取决于它在训练 caption 语料中的共现频率。
5. 本文件的提示词长度落在模型训练的长度范围内：官方 caption 统计为 short mean 20.9 / medium mean 76.6 / long mean 235.1 tokens（P90 293，最长 3,126），条件上限 4,096 tokens；本文件分支 A（≤85 词）≈ medium 档，分支 B 分块长文 ≈ long 档。

---

## 14. 按官方 Layout JSON schema 生成结构化提示词
官方将这一接口标注为 `Experimental`，本节各条照搬官方 `USERGUIDE > Layout Prompts` 的原文与字段定义，未作改写。官方同时说明：`The model supports combining natural language and structured input.`，即自然语言与结构化输入可以混用。

### 14.1 照搬官方示例的 JSON 结构并按需替换字段值
以下是官方给出的示例，生成提示词时保持这一 JSON 结构，按实际需求替换各字段的值：
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

### 14.2 按官方定义填写各字段并换算中心原点坐标
以下是官方文档里的字段定义原文：
- `summary.caption`：`A description of the overall scene and style.`，即整体场景与风格描述。
- `items`：每个元素的类别（`label`）、描述（`caption`）、边界框（`bbox`）；渲染文字时用 `text` 指定内容。
- `bbox_format: "yxyx"`：坐标顺序为 `[y_min, x_min, y_max, x_max]`。
- `bbox_coord_space: "abs_centralized"`：绝对像素坐标，以画布中心为原点，`x` 向右、`y` 向下。例如 1024×1024 画布在两个坐标轴上的范围均为 −512…+512。
- 从常规的左上角原点坐标换算时，官方原句为 `subtract W/2 and H/2 from conventional top-left-origin x and y coordinates, respectively`，换算公式为 `[y_min, x_min, y_max, x_max] = [y′_min − H/2, x′_min − W/2, y′_max − H/2, x′_max − W/2]`。
- 官方明确说明选用绝对像素坐标的原因：`We use absolute pixel coordinates, which we believe are better suited to layout control across different resolutions and aspect ratios.`

### 14.3 按官方规定限制各字段的填写范围
以下是官方 skill 文档给出的各字段取值范围：
- `summary.canvas`：填 `single` / `multi_panel` / `collage` 三值之一。
- `items[].label`：填 `object` / `character` / `text` 三值之一。训练语料里还包含 `panels`、`characters`、`watermarks` 等类别，但公开编译器只接受上述三个；论文推理增强器的 `elements` 属另一套 schema，官方 skill 原文要求不要写入：`Do not add ... scene-schema elements`。
- 官方支持 `summary.palette` 与 `items[].palette`（HEX 字符串数组），官方原文说明为 `palette on summary is also supported`。

### 14.4 处理坐标取整、画布尺寸与负面提示词参数
- 分数边缘按官方规则对左上坐标作 `floor`、对右下坐标作 `ceil`，最终坐标为整数且宽高为正。
- 编译产物中删去 `width/height` 等画布尺寸字段，尺寸在生成参数中另外设置，与本文件 §13 的采样参数一致；`W`、`H` 取正的 16 倍数。
- 负面提示词填入 CFG 的 `negative_prompt` 参数，官方 `Prompt Guidelines` 给出的 5 条已由本文件 §10 的负面提示词覆盖并扩充。论文 `planner` 的 `anchors` / `must_not` 是推理增强器内部字段，公开 schema 中不存在，不要写入。

### 14.5 将本文件其余规则填入对应的 JSON 字段
- `summary.caption` 按官方定义用于描述整体场景与风格，因此 §3 确定图片媒介与场景光影的开头句写在它开头；不要把画风词挪进 `meta`，因为所有字段拼接后都会进入同一条文本通道，详见 §13.3 第 4 条。
- 防止肢体结构崩坏、控制人物年龄的词条写进各个 `item` 的 `caption`，对应 §5、§6；`bbox` 只约束位置，不能替代这些词条。
- 图内文字内容仍按 §8 的规则处理，包括用英文双引号括起要渲染的文字、在换行处直接换行，以及防止图内印字把人物画成幼童或把身材画胖。

---

## 15. 控制动漫与二次元画面风格，以及避开 Gemma 4 重打标丢失的信息

> 证据等级 A

### 15.1 Gemma 4 重打标会丢失个人画师特征，不能靠写画师名或拆解五官还原

> 证据等级 A

1. **重打标阶段只保留肉眼可见内容，会丢失画师个人特征**：
   - 官方训练语料有 80% 采样自 Gemma 4 按 `USERGUIDE.md` 规范生成的 `caption.long/medium/short`，该规范要求只描述画面中肉眼可见的信息：`Include only information visible in the image`。
   - 取个人画师的原图，严格按官方 Gemma 4 规则写出 1:1 的 `caption.long` 再重新生图，实测哪些内容能还原、哪些会丢失如下：
     - **能够还原的内容（画面内容与构图）**：机位、人物动作、服装道具、环境光影与画面文字均能准确还原；
     - **会丢失的内容（画师手癖与特殊骨相）**：画师独有的笔压曲线、水彩混色边缘、铅笔灰调线稿以及偏离主流均值的个人骨相，在转写成英语描述时已经不可逆地丢失，生成的画面会退回常见的通用动漫画风与大众脸型。
2. **根据这一原因确定的两条编写要求**：
   - **不要写未收录的个人画师名**：除 §2 所列的 5 组知名厂牌与画师名（`Studio Ghibli / Hayao Miyazaki`、`Hirohiko Araki / JoJo`、`Akira Toriyama / Dragon Ball`、`Ilya Kuvshinov`、`WLOP`）外，不要在提示词中写任何个人画师名或 `XXX style`，例如 `kawakami_rokkaku`、`yewang19`、`ciloranko`、`rella`、`liduke`、`mika pikazo`、`ask`。
   - **不要用细碎的五官几何词硬凑个人画师脸型**：如果把画师手癖拆解成字面上的几何形状词，例如 `four-point-star white hair highlights, diagonal blush hash lines, droopy half-lidded eyes, white sticker rim`，模型只会按字面意思生硬拼贴出僵硬的脸型，反而破坏画面美感。要还原特定个人画师的笔触与脸型，只能训练该画师的 LoRA，不能靠提示词硬凑。

### 15.2 通过四大动漫风格调动通道切换二次元画风
当主体描述保持**画风中立**，即不写三次元的毛孔、闪光灯、胶片等词条时，不需要修改正面提示词结尾与负面提示词，利用训练语料中保留下来的信息，直接通过以下通道就能在 16 步、0.6 百万像素、`SLG=""` 参数下切换动漫风格：

> 证据等级 A

#### 通道一：在正面提示词开头写入 Danbooru 画种、年代与技法分类词
Gemma 4 重新打标时参考了原图的 `{{reference}}`（Danbooru Tags），并按规范详细描述了 `artistic style, colors, lighting`，因此模型对以下表示画种、年代与技法的 Danbooru 分类词反应明显：

| 动漫风格 | 放在正面提示词开头的词组 | 关键画面要素与注意事项 |
| :--- | :--- | :--- |
| **1990s 复古赛璐璐 OVA** | `1990s (style), retro artstyle, anime screencap, cel shading, vintage 1990s OVA anime aesthetic, VHS tape color bleed, hand-painted cel animation` | 蓬松 90s 复古刘海、暖橘胶片肤色、经典大白高光眼、手绘水粉背景 |
| **纯黑白日漫网点纸** | `manga page, monochrome, greyscale, screentones, inked lineart, bold black ink outlines, pen crosshatching, Japanese manga illustration` | 要同时**删掉正文里所有具体颜色词**（如删 `red, silver, navy`），否则正文里的颜色词会让黑白画面变成局部留色漫画 |
| **透明水彩晕染绘本** | `watercolor (medium), traditional media, ink and wash, soft pastel watercolor bleed on rough textured paper, delicate hand-painted illustration` | 冷压水彩纸纹理、边缘水渍自然扩散、柔和淡彩 |
| **高饱和扁平矢量波普** | `flat color, vector illustration, bold thick colored outlines, vibrant pop art palette, hard geometric cel shading, flat graphic artstyle` | 无渐变色块硬切、纯色暗部、潮酷平面插画感 |
| **厚涂刮刀油画插画** | `impasto, oil painting (medium), heavy visible brushstrokes, thick palette knife paint texture on canvas, painterly lineless illustration` | 抹除黑线，全图呈现立体堆叠的油画刮刀笔触肌理 |
| **日本动画原画手稿 (Genga)** | `anime genga, production art, rough pencil sketch, uncolored animation layout sheet, blue and red colored pencil construction lines on paper` | 白纸铅笔线稿 + 动画原画常用的红蓝彩铅结构辅助线与排线阴影 |
| **二头身 Q 版 (Chibi / SD)** | `chibi, super deformed, cute 2-head-tall chibi style, oversized head and tiny body, kawaii deformed anime illustration, bold outlines` | 直接将人体头身比固定为二头身粘土人比例 |
| **浮世绘 / 日本画和纸木版** | `ukiyo-e, traditional Japanese woodblock print, nihonga, flat mineral pigments on aged washi paper, sumi ink contour lines` | 泛黄和纸肌理、矿物颜料平涂、古典丹凤细眼 |
| **PC-98 16-bit 像素抖动** | `pixel art, 16-bit retro PC-98 Japanese adventure game graphic, crisp visible square pixel grid, checkerboard dithering shading, limited 16-color palette` | 清晰方形像素网格 + 天空与阴影处的棋盘格抖动网点 |
| **无描边油画棒/蜡笔绘本** | `lineless anime illustration, soft oil pastel and crayon drawing on rough textured paper, no black outlines, grainy chalk and pastel texture` | 消除黑色外轮廓线，呈粗糙粉彩颗粒感 |
| **狂放水墨重彩宣纸风** | `Chinese wild ink and wash painting, expressive dry-brush sumi calligraphy outlines with flying-white texture, splashed black ink mixed with traditional mineral pigments of cinnabar red and indigo on coarse yellowed xuan paper` | 枯笔焦墨飞白、泼墨重彩、粗宣纸纹理 |

#### 通道二：写出 4 种二次元界面边框或版式来带出对应画风
原理与 §9 用四角 HUD 界面带出 3D 游戏实机画面相同：特定的界面容器、版式与对应的二次元商业成图画风在训练语料中经常同时出现，在提示词里写出这些容器与具体文字，就能反过来带出对应的主体画风：
1. **Galgame / 视觉小说 CG**：
   - 开头写 `visual novel cg, Japanese galgame screenshot, glossy anime coloring`；
   - 结尾写出底部对话框：`At the bottom of the frame, a semi-transparent dark-blue visual novel dialogue box with a pink nameplate reading "[角色名]" and crisp white text reading "[台词]"`。
   - 效果：机位与人物上色会自动切换为正对镜头的 2010s 商业 Galgame 攻略立绘。
2. **TV 动画番剧正片平涂**：
   - 开头写 `anime screencap, TV anime broadcast screenshot, cel shading, flat anime coloring`；
   - 结尾写出底部黑边黄字字幕：`At the bottom center, crisp yellow anime subtitle text with black outline reading "[台词]"`。
   - 效果：去掉插画厚涂感，变成只有单层硬阴影的 TV 动画播出截帧。
3. **官方角色设定集三视图**：
   - 开头写 `anime character reference sheet, official character design turnaround sheet on clean white background, multiple views, full-body front view and 3 facial expression headshots, clean cel-shaded lineart, color palette swatches`。
   - 效果：画面会呈现同一角色的多角度立绘、表情差分头像与排列整齐的配色色卡方块。
4. **SSR 集换式卡牌框**：
   - 开头写 `SSR anime trading card game item, ornate golden holographic card frame border, top title bar reading "[标题]", 5 gold stars at top right, bottom stat box reading "ATK 2500 / DEF 2100", dazzling anime card illustration`。

#### 通道三：用高保留率 JSON 字段 `palette` 控制配色、用 `multi_panel` 同图切换画风，以及合成跨次元画面
技术报告 Appendix C.3 Table 12 显示，`scene_type`、`gen_brief`、`palette` 与保留率达 90% 的 `summary.meta` 在训练中保留率最高：
1. **用 `palette` 十六进制色值数组替换全图配色**：
   - 当需要迁移某张参考图的配色方案时，直接从参考图用吸管取 4~5 个主色 HEX 填入 JSON 的 `"palette": ["#HEX1", "#HEX2", ...]`；
   - **使用前提**：要从 `gen_brief` / `caption` 正文中**删去自然语言具体颜色词**，例如删掉 `silver, red, navy`，并在末尾加一句 `dominated strictly by the specified palette`。这样可以在保持人物线条、构图、动作不变的前提下，将全图色彩固定为目标 HEX 色板。
2. **用 `canvas: "multi_panel"` 在同一张图里分格画出不同画风**：
   - 使用 §14 的 `summary/items` JSON，设 `"canvas": "multi_panel"`，在每个 `item` 的 `bbox`（`abs_centralized` `yxyx`）内分别写入不同画风的 `caption` 与 `tags`，例如上格写 `["1990s (style)", "cel shading", "full color"]`、下格写 `["manga", "monochrome", "screentones"]`，就能在单张图像内做出上下分格完全不同的动漫时代画风，并且互不串色。
3. **合成跨次元 2D 与 3D 混合画面**：

  > 证据等级 A

   - **基础硬切剪贴画风格（Roger Rabbit 强反差风）**：在提示词中写明背景为 `unretouched photorealistic 35mm photograph with shallow depth of field`，主体为 `flat 2D cel-shaded anime character illustration with crisp black ink outlines and flat anime coloring`。注意：如果不加光影合成描述，角色会呈现带白边的剪贴画感；如果用 `<|foreground_photo|>` 与 `<|background_manga|>` 分块包裹，则会把画面切成上下两格漫画分镜。
   - **85mm 电影实景与极简线稿人物的光影合成写法**：
     - **需要同时写明的 3 个部分**：① **背景**：`85mm 专业镜头拍摄的写实风格背景，浅景深焦外光斑，强烈光线照下` + `荷兰角倾斜构图、仰拍、侧面靠后视角` + `150cm 娇小身高比例，四肢不要画得过长`，用来防止写实摄影词汇把动漫少女画成真人比例的大长腿；② **角色线稿与上色**：`角色画风【线条极简】，简单的手绘线稿，线条清晰明确简单潦草，只绘制最低限度的单层阴影和极简上色`；③ **跨次元光影融合**：`结合后期的光影效果、逆光发丝金边和打光调色合成，让动漫画风少女完美地和写实背景融合`。
     - **3 种上色浓度的写法**：
       1. **清透淡彩唯美档（推荐主力）**：同时写入 `风格：二次元梦幻、极致唯美插画` 与 `角色画风【线条极简】，简单的线稿，只绘制最低限度的阴影和极简上色，结合后期的光影效果和打光合成`；前者提供清透柔和的固有色，后者去掉厚涂油腻感并让真实逆光穿透发梢与衣摆。
       2. **纯白描速写手稿档**：删去“极致唯美插画”与具体颜色词，把 `【线条极简】简单的手绘线稿、最低限度单层阴影` 放在前面，画面会呈现纯白底铅笔速写少女处在真实三维场景中、发丝被真实阳光打出金边的效果。
       3. **暖金环境光浸染档**：用英文写入 `warm golden sunlight rim-lighting and ambient color grading bleed naturally across her simple sketched lines`，让实景夕阳色温均匀浸染松散速写线条。
     - **近景特写防止人物变成三次元真人的写法**：当画面为**近景特写**时，如果句首写 `生成一张 85mm 镜头拍摄的电影感人物照片` 且只拍后脑勺或肩背而不露五官，模型会顺着 Qwen3.5 从左到右的单向注意力把人物完全画成**三次元真人**。因此近景特写要做到两点：① 在句首写明 `跨次元融合：2D动漫画风少女与85mm写实摄影背景合成`；② 明确写出 `侧头回眸露出清晰的2D动漫侧脸线条与明亮动漫眼眸`，同时删掉下半身或全身动作词，防止镜头自动拉远。

## 16. 用 `<|概念|>...<|/概念|>` 闭合标签分块生成提示词与标注训练集
本节汇总中英文出图提示词与 LoRA / SFT 训练集标注的闭合标签分块写法，可独立查阅。

> 证据等级 A

### 16.1 用 `<|概念|>...<|/概念|>` 替代单段长句能提高信噪比并防止特征互串
1. **文本编码器的分词与向量机理**：
   - Qwen3.5-9B 的原生聊天模板基于 `<|im_start|>`、`<|im_end|>` 等 `<|...|>` 结构。虽然自定义的 `<|style|>` 或 `<|镜头画风|>` 并非词表预注册的单个 special token，在 BPE 分词中会被切分为 `["<", "|", "概念", "|", ">"]`，但 Qwen3.5 仍会按聊天模板的习惯将其识别为隔开不同模块的边界标记。
   - 向量实测证实：在 22 种候选分隔格式中，`<|...|>`...`<|/...|>` 结构的词法背景噪音最低，进入 DiT 前的净有效信噪比是传统逗号拼接的 **12.7 倍**、是普通 `Style: ...\nScene: ...` 换行格式的 **3.2 倍**。
2. **出图实测证实的 4 项画面作用**：
   - **防止中英文长句连写多主体时出现多头或多肢粘连**：用逗号一路连写多个人物与怀抱动物、道具时，容易出现不同主体的特征粘连，例如猫长出两个头、前景道具重复出现并挡住人脸；用 `<|概念|>「内容」<|/概念|>` 按模块切开后，不同模块的特征不再互相粘连。
   - **防止多层电影景深画面上下出现大面积黑边**：当画面含前景虚化遮挡、中景人物、远景街道等四层空间时，单段长句容易把画面上下 2/3 涂成黑色信箱黑边；用 `<|foreground_blur|>` / `<|midground_focus|>` / `<|background_street|>` 分块后，各层景深会自动填满整张画布。
   - **双人贴脸时隔开各自的冷暖灯光**：当左侧人物受冷青色霓虹照射、右侧人物受暖黄色台灯照射，或者左窗照入闪电蓝光、右脸映着烛火暖光时，闭合标签可以把各自的色温限制在对应的人物块内，不会互相染色。
   - **单独修改某一块时不牵连其余模块的构图**：修改灯光只需改动 `<|lighting|>` 行，修改字幕只需改动 `<|subtitle|>` 行，不会牵连其余模块的构图。

### 16.2 使用闭合标签分块要遵守五条编写规则，否则会导致隔离失效或画面切分错误
1. **按人物或空间层次分块，不要按零散属性分块**：
   - **正确写法（一人一块，写全该角色的所有属性）**：同一个角色的**方位、发色、瞳色、服装、配饰、手中道具、专属灯光**要全部写在同一个 `<|left_subject|>...<|/left_subject|>` 块内。
   - **错误写法（把不同角色的属性按部位拆开）**：不要写成 `<|hair|>左女白发，右女黑发<|/hair|>\n<|clothes|>左女红衣，右女蓝衣<|/clothes|>`，按部位拆开混写会让不同角色的颜色交叉错位。
2. **每个块的内部正文仍要写明空间方位词**：
   - 标签名 `<|left_girl|>` 只负责隔开不同模块的边界，**不能替代块内的空间介词**。块内正文开头仍要明确写出 `On the left stands...`、`In the immediate foreground...` 或 `画面左侧...`。
3. **按从左到右的定调顺序排列各个块**：
   - 前面的块会给后面的块定调（机理见 §4），块与块的排列顺序固定为：
     - **第 1 块**：全局画风、镜头与基础光影，例如 `<|style_and_camera|>` 或 `<|环境氛围|>`。
     - **第 2 块**：主要人物或前景遮挡层，用来给全图的画风、人种特征与主色温定调。
     - **第 3 块**：次要人物或中远景层；描写次要人物时，句首要用 `In sharp contrast, NO...` 直接写出与主要人物相反的特征，切断前一块传下来的属性。
     - **第 4 块**：画面内可见文字、底部字幕或界面元素，例如 `<|subtitle|>` 或 `<|bottom_ui|>`。
4. **保持一行一块、成对闭合且不嵌套，总数控制在 3~5 块以内**：
   - 每个块单独占一行，行尾用真实换行符 `\n` 换行，首尾要成对闭合 `<|tag|>...<|/tag|>`，**不要在块内嵌套子块**。
   - 英文提示词直接写 `<|tag|>English content.<|/tag|>`；中文提示词推荐写成 `<|中文概念|>「中文内容。」<|/中文概念|>`。
5. **不同画风分块会把画面切成上下漫画分镜，跨次元融合不要分块**：
   - 如果在同一个提示词里把不同画风拆成两个块，例如写成 `<|foreground_photo|>三次元全彩实拍咖啡桌...<|/foreground_photo|>\n<|background_manga|>二次元黑白网点漫画少女...<|/background_manga|>`，模型会在画面正中切出一条笔直的**上下漫画分镜线**，把上半部分画成纯黑白漫画、下半部分画成全彩照片。
   - **选用方式**：需要生成上下分镜对比图时，用 `<|...|>` 分块；需要生成跨次元画面、让 2D 动漫少女坐在真实 3D 咖啡桌后并融在同一个三维空间里时，要把两者合在同一个块内用连贯自然语言描述。

### 16.3 按场景选用三套可直接套用的闭合标签模板

#### 模板 1：用于多层电影景深与双人特征对比的英文提示词模板
```text
<|camera_and_lighting|>Gritty 1990s Hong Kong cinema widescreen shot inside a dimly lit night diner, heavy 35mm film grain.<|/camera_and_lighting|>
<|foreground_blur|>In the immediate left foreground, an out-of-focus dark silhouette of a man's shoulder and a rain-streaked glass partition with neon reflections partially frame the view.<|/foreground_blur|>
<|left_subject|>In sharp focus on the left under cold cyan neon light sits a European teenage girl with curly blonde hair, round gold glasses, and light freckles, wearing a dark denim jacket and holding a silver camera.<|/left_subject|>
<|right_subject|>In sharp contrast, on the right under warm amber lamp light sits an East Asian teenage girl with straight jet-black bob hair, NO glasses, and spotless porcelain skin with zero freckles, wearing a cream knit sweater and stirring a glass of iced lemon tea.<|/right_subject|>
<|background_street|>Through the window in the far background, a blurry wet asphalt street glows with deep blue traffic lights and a passing red taxi.<|/background_street|>
```

#### 模板 2：用于影视剧照与多主体同框的中文提示词模板
中文提示词按 `<|概念|>「内容」<|/概念|>` 格式逐行分块编写：
```text
<|环境氛围|>「黄昏时分的复古理发店室内实拍剧照，35mm胶片质感，暖金夕阳与室内霓虹交织。」<|/环境氛围|>
<|前景柜台|>「前景的玻璃柜台左下角放着一把银色剪刀和一瓶蓝色洗发水，景深微虚。」<|/前景柜台|>
<|左侧人物|>「柜台后方左边坐着一位满头银发、戴着老花镜、穿褐色背心的老理发师，正低头安静看报纸。」<|/左侧人物|>
<|右侧人物|>「柜台后方右边站着一位染着亮粉色短发、不戴眼镜、穿黑色皮夹克的年轻女孩，怀里抱着一只正常的单头橘猫，正对着镜子吹粉色泡泡糖。」<|/右侧人物|>
<|背景与文字|>「后墙上挂着红色的霓虹灯牌，清晰亮着“OPEN 24H”字样，画面底部印着黄色中文字幕：“今夜无人入眠”。」<|/背景与文字|>
```

#### 模板 3：用于标注 LoRA 与 SFT 画师训练集的 `.txt` 双行模板
当制作 `MOSS-Image` 的画师风格 LoRA 或 SFT 微调数据集时，每张训练图的 `.txt` 标注文件固定采用以下两行结构：第一行单独写画师名标签，不要加 `@`，不要在同一行混入 `anime` 等大类词，也不要手写 `<|im_start|>` 等聊天模板标签；第二行写画面描述正文：
```text
<|artist|>kawakami_rokkaku<|/artist|>
A warm, softly lit 2D anime illustration inside a rustic wooden cabin. Holo from Spice and Wolf, a slender young girl with long reddish-brown hair, pointed wolf ears, and glowing crimson eyes, sits on a white bedsheet brushing her fluffy white-tipped wolf tail with a wooden comb...
```

使用 `MOSS-Image`（Qwen3.5-9B + 5B DiT）生成二次元厚涂插画时，如果不区分画面各部位的精修程度，画面容易显得像未画完的半成品草稿，或退化成死板塑料的 10 岁页游立绘。要同时保证商业插画的完成度与手绘笔触，应当把精细描写集中在五官与发丝等视觉焦点，同时让衣物与背景保留松弛笔触。生成高完成度插画提示词时，可按以下 4 项要点与配套骨架编写：

## 17. 不区分画面主次会导致整图沦为半成品草稿或死板塑料立绘
在 `MOSS-Image` 中，如果不对画面各部位做主次分级，模型就会出现以下两种极端情况：
1. **全图笔触过于松散，导致画面像未画完的半成品**：
   - 在提示词开头写下 `loose sketch lines, visible flat-brush strokes`（松散速写线、可见平头画笔笔触） 却没有限定作用部位时，Qwen3.5 的**单向因果注意力（§4）**会把这种松散潦草的笔触延续到后面的**脸部、眼睛和手指**。
   - **后果**：一旦人物的五官、瞳孔高光和手指关节出现草稿线或未涂满的色块，人眼就会直接将其看作没画完的半成品。
2. **全图所有部位都过度锐化精修，导致画面产生塑料页游感与 10 岁 球关节娃娃感**：
   - 如果为了避免半成品感而写下 `ultra-high-completion, clean refined edges, intricate golden embroidery` 并开启 `SLG="22"`，模型会把**衣服折痕与背景石头**等次要部位全部按最高锐度画得过于生硬。
   - **后果**：全图缺乏留白与透气感，手绘笔触的松弛感消失；同时因为过度追求精致，`jewel-like eyes` 会触发二次元常见的大头大眼萌系脸型，把 15 岁少女画成 10 岁塑料娃娃。

### 17.1 用四个分段标签把精修集中在视觉焦点并让边缘保持松弛笔触
生成提示词时，可用 4 个 `<|...|>` 标签分别控制不同部位的精修程度与笔触：

#### 1. 第 1 块 `<|style_and_aesthetic|>`：用选择性精修划分五官发丝与衣物的笔触
> **核心写法**：`selective focus: finely rendered, luminous facial features and delicate hair strands contrasted against expressive, breathable painterly gouache brushwork on the clothing and softly blurred background. Muted, sophisticated low-saturation color palette...`
- **作用机理**：直接要求模型把精细刻画集中在五官和发丝上，同时对衣服和背景采用透气、松弛的不透明水粉笔触。
- **插画主次关系**：在 fkey、ask、wlop 等插画师的常见处理中，画面显得完成度高，不是因为背景的每块石头都画得像照片，而是因为**眼睛、嘴唇、鼻尖腮红和发丝足够精致水润**，能撑起整体完成度，同时衣摆和背景保留松弛的大笔触；再配合 `Muted, low-saturation color palette`（低饱和高级灰调色盘），即可去掉廉价的荧光塑料感。

#### 2. 第 2 块 `<|teenage_girl_aura|>`：用双向年龄排除、少女骨相与微表情稳定 15 岁少女特征
> **核心写法**：`slender 15-to-16-year-old teenage girl with true adolescent grace—neither a big-headed 10-year-old child nor a mature 20-year-old woman. Balanced, delicate teenage facial proportions, a softly tapered chin, and a slender, graceful neck. Tilting her head slightly back at a three-quarter angle... half-lidded, languid olive-green eyes filled with a quiet, wistful teenage aloofness.`
- **把年龄特征控制在 10 岁与 20 岁之间的三个要点**：
  1. **用自然语言同时排除幼态与成熟女性特征**：Qwen3.5-9B 作为大语言模型，能够理解 `neither a big-headed 10-year-old child nor a mature 20-year-old woman` 这类双向排除约束。
  2. **写出少女期的下巴与颈部骨相**：10 岁幼童的特征是 `round babyface + short neck`，而 15～16 岁少女的骨相特征是 **`softly tapered chin`（微尖柔和的下巴）** 与 **`slender, graceful neck`（抽条期修长的脖子）**。
  3. **避开正脸瞪眼，改用侧颜微仰与半垂眼神**：幼童常表现为正脸瞪大眼睛（`wide-open eyes`），而少女感来自 **`three-quarter angle tilted slightly back`（3/4 侧颜微仰头）** 配合 **`half-lidded, languid eyes with wistful teenage aloofness`（半垂慵懒、带着一丝清冷心事的眼神）**。

#### 3. 第 3 块 `<|hair_and_pose|>`：加入微松发髻与半透明碎发，避免头发像死板假发
> **核心写法**：`slightly loose round buns... airy, translucent wisps of bangs falling naturally across her forehead`
- 只写 `neat round twin buns`（整齐的丸子头）时，模型容易将其画成两个死板的黑球；改为 `slightly loose`（微松） 并加上 `airy, translucent wisps of bangs`（透气半透明的空气感碎发丝） 后，发丝的高光和边缘会呈现出细碎自然的层次。

#### 4. 第 4 块 `<|hanfu_and_background|>`：主动省略繁琐刺绣，保持服装色块干净
> **核心写法**：`simple, elegant traditional Hanfu without gaudy embroidery... plain matte dark-red borders... softly out-of-focus twilight garden background.`
- **画面完成度不足时，不要往衣服上加复杂花纹**：一旦让模型去画 `golden floral embroidery`（金线花卉刺绣），模型会分出大量注意力去画细碎花纹，反而显得杂乱。明确写出 `without gaudy embroidery, plain matte borders`（无俗气刺绣、纯色哑光暗红边），更容易保持大色块的干净质感。

### 17.2 生成高完成度二次元厚涂插画时可直接套用四段式提示词骨架
无论使用 `<|artist|>fkey<|/artist|>` 还是其他厚涂画师，要同时兼顾画面完成度、松弛笔触与 15 岁少女特征，生成正面提示词时可直接套用以下四段式骨架：
```text
<|artist|>[画师名]<|/artist|>
<|style_and_aesthetic|>High-taste artistic 2D digital painting with selective focus: finely rendered, luminous facial features and delicate hair strands contrasted against expressive, breathable painterly gouache brushwork on the clothing and softly blurred background. Muted, sophisticated low-saturation color palette with [填入2-3个主色调，如 cool greys, dusty blue, and warm browns].<|/style_and_aesthetic|>
<|teenage_girl_aura|>[景别如 Medium three-quarter portrait] of a slender 15-to-16-year-old teenage girl with true adolescent grace—neither a big-headed 10-year-old child nor a mature 20-year-old woman. She has balanced teenage facial proportions, a softly tapered chin, and a slender neck. [填入头颈微动态，如 Tilting her head slightly at a three-quarter angle], she gazes through [眼神如 half-lidded, languid eyes with a quiet teenage aloofness]. Soft warm peach-pink blush dusts her nose tip and cheeks, with softly parted, glossy pale-pink lips.<|/teenage_girl_aura|>
<|hair_and_pose|>Her silky [发色] hair is styled into [发型，加上 slightly loose / airy, translucent wisps of bangs]. [填入手部与道具的简洁优雅动作].<|/hair_and_pose|>
<|outfit_and_background|>She wears a simple, elegant [服装描述，加上 without gaudy patterns, clean matte fabric blocks], set against an atmospheric, softly out-of-focus [背景环境] background.<|/outfit_and_background|>
```
生成时的采样参数建议配合 `SLG=""`（置空）、`28~30` 步与 `CFG 3.3` 使用。

负面提示词：
```text
<|texture_and_style_neg|>photorealistic, real human, 3d render, cgi, photograph, sketch, rough sketch, uncolored patches, unpolished, camera flash, microscopic pores, film grain, green skin, zombie skin, blotchy skin patches, harsh unblended skin blocks, over-sharpened hard edges, vector art, plastic skin, oversaturated neon colors, gaudy gold embroidery, wet long hair covering eyes, thick black outlines<|/texture_and_style_neg|>
<|age_and_body_neg|>child, toddler, preteen, loli, 6 years old, 8 years old, 10 years old, oversized head, big round babyface, short stubby neck, wide-open staring eyes, doll-like stiff face, flat chest, mature 20-year-old woman, adult woman, milf, old, 25 years old, 30 years old, curvy, plump, chubby, fat, wide adult hips, large breasts, huge breasts, C-cup, hard muscle planes, six-pack abs, gaunt ribs, heavy makeup, thick black eyelashes<|/age_and_body_neg|>
<|universal_anatomy_neg|>blurry, out of focus, lowres, censored, mosaic censoring, bar censor, ugly face, distorted face, squished cheek, worst quality, bad anatomy, fused bodies, fused legs, extra limbs, extra arms, extra legs, extra hands, three hands, extra fingers, mutated hands, fused fingers, six fingers, watermark, text, signature<|/universal_anatomy_neg|>
```
以下针对 `MOSS-Image`（Qwen3.5-9B 单向因果编码器 + 5B DiT），按画质与色调词、多分镜 Layout JSON 构图与排版、负面提示词固定写法、跨分镜与跨页角色一致性四个部分逐条说明。

## 18. 生成全彩多分镜漫画时的质量词、构图、负面提示词与人设一致性

### 18.1 特定质量词与色调词决定画质、笔触与清透色调
在同一个 Seed 下，`summary.caption` 前 60 个词哪怕漏掉一个词，整张图的色调就会变黄、变油腻，或者退化成未涂满的水彩草稿。以下是决定全彩水润厚涂效果的每条核心词及其作用：

#### 1. 用全局画种与分级渲染消除草图感与死板塑料感
| 核心英文词条 | 作用与机理 |
| :--- | :--- |
| **`High-taste, fully colored 2-panel vertical 2D digital painting comic (2koma)`** | **固定画种与完成度**：`fully colored 2D digital painting` 让画面脱离黑白漫草稿；哪怕在 4~7 格分镜页中，故意保留 `2-panel vertical... (2koma)` 这几个字不删，是为了**保持前 20 个 Token 的位置不变**，防止固定 Seed 的初始色彩空间发生偏移。 |
| **`with selective focus:`** | **分配画面主次**：让 Qwen3.5 不要平均用力，改成中心精修、边缘写意的主次分级渲染。 |
| **`finely rendered, luminous facial features, glistening highlights / teardrops, and delicate hair strands`** | **精修视觉焦点**：把 80% 的高频细节集中在五官、眼神光、泪珠和发丝上。只要脸和眼睛足够水润精致，人眼就会认为整张画是高完成度成品。 |
| **`contrasted against expressive, breathable painterly gouache brushwork on the clothing`** | **限制水粉笔触范围**：将水粉厚涂笔触（`painterly gouache brushwork`）限制在衣服和背景上，既保留手绘笔触感，又不会波及脸部导致五官像没画完的草稿。 |
| **`zero rough sketch lines`** | **去除草稿线**：在正面提示词中显式声明零草稿线，防止全身镜头或留白较多时模型露出铅笔结构线。 |

#### 2. 用奶油蜜桃色调与室内日光词控制肤色与通透度
| 核心英文词条 | 作用与漏掉后的后果 |
| :--- | :--- |
| **`Muted, sophisticated warm peach skin and deep navy-blue color palette`** | **低饱和冷暖对比色盘**：用低饱和暖蜜桃肤色配深海军蓝与纯白构成冷暖对比。**不要改成 `vibrant saturation` 或 `warm-tan / wooden`**，否则在单向因果染色下全图会变成焦黄油腻的木头色。 |
| **`with soft bright indoor daylight`** | **清透室内日光**：画面变灰暗油腻，往往是因为把 `soft bright indoor daylight` 简写成了 `soft indoor lighting`。加上 `bright... daylight` 才能打出日系插画清透、干净的漫反射亮面。 |
| **`smooth creamy skin tones`** + **`fair creamy peach-flushed skin`** | **白里透粉的细腻肤质**：决定皮肤呈现水润丝滑的奶油蜜桃感，而不是粗糙厚重的油画块面感。 |
| **`crisp Chinese comic dialogue text in clean speech bubbles`** | **保持对话框与台词清晰**：保证图内中文台词笔画清晰、气泡干净，不出现脏乱的灰底色块。 |

### 18.2 按七项做法控制多分镜构图、空间隔离与气泡
在 `1024 × 1536` 分辨率、`Steps 50`、`SLG="22"` 下，使用官方 `abs_centralized`（中心原点 `[0,0]`，Y轴 `-768 ~ +768` 上负下正，X轴 `-512 ~ +512` 左负右正）与 `yxyx`（`[上Y, 左X, 下Y, 右X]`）格式时，有以下 7 项分镜做法：

1. **主画面用 `character`、局部特写用 `object` 防止多画人头**
   - **问题**：一页里有主立绘，同时右上角有眼睛微距、嘴唇微距、手部微距或衣褶特写时，如果全部写 `"label": "character"`，模型会在每个特写框里多画一颗人头。
   - **做法**：全图只有带完整头身的格子用 `"label": "character"`；所有局部放大特写（眼睛、嘴唇、手部、衣褶、文字招牌）应当设为 `"label": "object"`，并在 `caption` 开头写 `Extreme close-up inset panel showing ONLY...`。
2. **用两个独立 BBox 拉开双人距离**
   - **问题**：把两个角色写在同一个 `character` 框里，模型默认画成肩贴肩站在一起。
   - **做法**：在同一个分镜内，将左角色和右角色**拆成两个独立的 `"label": "character"` 项**，并在两者的 X 坐标之间留出 **30 ~ 140 像素的间隔**（例如左框 `X: -450 ~ -70`，右框 `X: 70 ~ 450`），配合描述词 `standing far apart on opposite ends of the panel, keeping a wide distance`，即使做击掌动作也会隔空伸长胳膊单手拍掌。
3. **并排小分镜对齐上边界 Y 坐标防止人物跨出分镜框**
   - 当画面是上半部大图加左下、右下两个并排小分镜时，左下格和右下格的 `y_min`（上边界）应当设为同一个数值（如 `360`），不要让右下格的 `y_min` 伸到 `20`，否则右下格的人物会半截身子跨出分镜框踩到上格画面里。
4. **多格小分镜用 `Waist-up / Thigh-up Medium Shot` 避免人物画成四头身小人**
   - 在一页切分 6~7 个小格子的页面里，小格子里不要写 `full-body`，应当改写为 `waist-up medium shot`（腰部以上中景）或 `thigh-up medium shot`（大腿以上中近景），把躯干放大填满格子，避免模型因为格子小而把人物画成 Q 版小人。
5. **不写“旁白”二字，靠几何形状描述区分旁白方框与角色对话气泡**
   - **旁白框写法**：`"caption": "Clean vertical Chinese narration text inside a sharp-cornered white rectangular narration box (no bubble tail)"`（直角白色矩形竖排旁白框，无气泡尾巴）。
   - **角色开口气泡写法**：`"caption": "Clean vertical Chinese spoken dialogue inside a rounded oval comic speech bubble with a pointer tail pointing to [Character]"`（带指向尾巴的圆角椭圆漫画对话气泡）。
   - **剧烈情绪/广播框写法**：`jagged burst speech bubble`（爆炸锯齿气泡，用于怒吼）或 `octagonal monitor frame`（八角形屏幕广播框）。
6. **在留白区添加漫画标题 Logo 与草稿标记**
   - 在画面的斜向留白区或天际留白区（如顶部中央 `[-745, -180, -610, 280]` 或左下角留白）新增一个 `"label": "text"`，描述写为 `Ornate, stylized Japanese manga title logo in bold dark-navy and gold typography with a decorative border`，或写手写草稿标号 `Casual handwritten-style Chinese draft marker`。
7. **生成 SVG 矢量线框分镜示意图或纯几何体木偶占位图**
   - 若要生成带分镜框标线的分镜示意图：将全局描述改为 `Digital manga storyboard layout wireframe diagram with visible SVG vector rectangular bounding-box overlays in bright red, cyan-blue, and green with corner anchor points and crosshairs`；
   - 若要抹除人物五官、只画空间占位几何体：将所有 `"character"` 改为 `"object"`，正面提示词写 `faceless geometric primitive dummy made ONLY of simple empty circles with crosshairs for heads, rectangular boxes for torsos, and plain cylinders for limbs`，负面提示词加入 `eyes, nose, mouth, face, hair, clothing details`。

### 18.3 固定负面提示词以保持采样方向稳定
在固定 Seed 下，CFG 采样方向由 `Positive - Negative` 共同决定。一旦在负面提示词开头增删词条（例如临时加入 `orange skin`），Qwen3.5 的单向因果注意力就会改变整条负面提示词的编码向量，导致全图画风和人脸发生明显偏移。

因此，同一系列漫画应当**从头到尾 100% 保持同一套三段式负面提示词不变，连标点都不改**，所有单页特殊要求（如遮住页码、去掉页码）全部在正面提示词内解决。

#### 负面提示词采用固定三段式结构
```text
<|texture_and_style_neg|>photorealistic, real human, 3d render, cgi, photograph, monochrome, greyscale, black and white, screentones, manga draft, lineart only, sketch, rough sketch, messy construction lines, uncolored patches, unpolished, camera flash, microscopic pores, film grain, green skin, zombie skin, blotchy skin patches, harsh unblended skin blocks, over-sharpened hard edges, vector art, plastic skin, oversaturated neon colors, gaudy gold embroidery, wet long hair covering eyes, thick black outlines<|/texture_and_style_neg|>
<|age_and_body_neg|>child, toddler, preteen, loli, 6 years old, 8 years old, 10 years old, oversized head, big round babyface, short stubby neck, wide-open staring eyes, doll-like stiff face, flat chest, mature 20-year-old woman, adult woman, milf, old, 25 years old, 30 years old, curvy, plump, chubby, fat, wide adult hips, large breasts, huge breasts, C-cup, hard muscle planes, six-pack abs, gaunt ribs, heavy makeup, thick black eyelashes<|/age_and_body_neg|>
<|universal_anatomy_neg|>blurry, out of focus, lowres, censored, mosaic censoring, bar censor, ugly face, distorted face, overly squished grotesque cheek, worst quality, bad anatomy, fused bodies, fused legs, extra limbs, extra arms, extra legs, extra hands, three hands, extra fingers, mutated hands, fused fingers, six fingers, extra penis, two penises, detached penis, floating hand, watermark, signature<|/universal_anatomy_neg|>
```
- **渲染中文台词时的限制**：当需要渲染中文台词时，`<|universal_anatomy_neg|>` 末尾**不要写 `text, speech bubble, comic dialogue`**。
- **双侧年龄与体型排除**：`<|age_and_body_neg|>` 同时排除幼龄特征（`child, loli, 10 years old, oversized head, flat chest`）和成年丰满特征（`mature 20-year-old woman, curvy, C-cup`），把生成结果收敛在 **15~17 岁抽条期 A+ 罩杯少女** 的范围内。

### 18.4 跨页与跨分镜保持同一角色人设、瞳色与年龄骨相有三项做法
要让同一个角色在 10 页不同分镜、或 1 页 7 个小分镜里保持 **95% 以上的面部、瞳色、发型与年龄体态一致性**，可采用以下四项做法：

#### 1. 在顶部写出角色总表并在下方分镜中只写动作与核心特征
- **原理**：在顶部 `summary.caption` 写出角色总表并在下方 `items` 中只写分镜动作，利用 Qwen3.5 单向因果编码器从左到右传递信息的特性，把主角的完整外貌特征直接定义在 JSON 开头的 `summary.caption` 中（如 `Character 1 (Name): [完整骨相+瞳色+发色+标志物+服饰]. Character 2 (Name): [完整外貌].`）。
- **分镜内重复核心特征词**：在下方每个 `character` 分镜框里，**不要省略骨相词与瞳色词**，否则省略 `emerald-green eyes` 时容易被 `blush` 染成粉红眼；每个分镜框都带上短描述 `our tall, slender 16-to-17-year-old [发色] girl [名字] with elegant adolescent proportions, softly tapered chin, long graceful neck, and sharp [瞳色] eyes`，后面紧跟该格子的简洁动作。

#### 2. 用固定骨相词并禁用幼龄词防止名著角色按 11 岁童星印象幼化或多格自动幼化
- **多格漫画正面提示词不要出现的词**：
  - 在多格分镜的正面提示词中，**不要使用 `petite`（娇小）、`child`、`10-year-old`、`little`**。哪怕写否定句 `neither a 10-year-old child`，在小分镜里也会因为注意力分散被模型误提取出 `10-year-old child`。
- **防止幼化、大头与熟女化的 15~17 岁抽条期修长少女骨相词**：
  > `a tall, slender, long-legged 16-to-17-year-old older teenage girl with elegant 7.5-head-tall bishoujo proportions, mature adolescent facial bone structure, softly tapered chin, long graceful neck, fair creamy peach-flushed skin, small perky A-cup adolescent breasts`
- **防止正太化的 16~17 岁高挑少年骨相词**：
  > `a tall, lean, broad-shouldered 16-to-17-year-old older teenage boy with a defined adolescent jawline, long neck, and tall bishounen proportions`

#### 3. 将图内印字、角色设定年龄与正面提示词的骨相年龄分开处理
- **图内印字避免触发幼化**：如果原图名牌写 `"12岁"` 或 `"13岁"`，直接印在图上会把角色画成幼童；应当将图内文字改为 `"15岁"`。同理，遇到三泽 咲这类 28 岁设定角色时，**台词气泡里写 `"确实你是二十八岁的大人了"`，但英文 `character` 描述里依然 100% 保持 15~16 岁少女外貌描述**，不要在英文里写 `28 years old`。
- **性转角色写法**：画女版马尔福时，正面提示词**不要写 `genderswap / mtf / male to female`**，否则会画出男性骨骼；应当直接写 `draco malfoy, 1girl`，再配合 16 岁修长少女骨相与斯莱特林校服特征。

### 18.5 生成全彩多分镜漫画时可直接套用以下母版
生成 `MOSS-Image` 全彩多分镜漫画提示词时，可直接套用以下母版，按需替换方括号中的角色、分镜与台词信息：
```json
{
  "summary": {
    "meta": ["comic", "multi_panel", "digital_painting"],
    "canvas": "multi_panel",
    "caption": "High-taste, fully colored 2-panel vertical 2D digital painting comic (2koma) with selective focus: finely rendered, luminous facial features, glistening highlights, and delicate hair strands contrasted against expressive, breathable painterly gouache brushwork on the clothing. Muted, sophisticated warm peach skin and deep navy-blue color palette with soft bright indoor daylight, smooth creamy skin tones, crisp Chinese comic dialogue text in clean speech bubbles, zero rough sketch lines. Character 1 ([角色名/Tag]): a tall, slender, long-legged 16-to-17-year-old older teenage girl with elegant 7.5-head-tall bishoujo proportions, mature adolescent facial bone structure, softly tapered chin, long graceful neck, fair creamy peach-flushed skin, sharp luminous [瞳色] eyes, and [发型发色与标志配饰], wearing [核心服装]. Character 2 ([男主名/Tag]): [男主固定的年龄、骨相与服装写法，每个分镜都照这一份写]. [全局核心Tags], uncensored."
  },
  "items": [
    {
      "label": "character",
      "caption": "[分镜方位与景别，优先用 Waist-up / Thigh-up medium shot]: Our tall, slender 16-to-17-year-old [发色] girl [角色名] with elegant adolescent proportions, softly tapered chin, long graceful neck, and sharp [瞳色] eyes in her [服装简写], [该分镜的简洁动作与傲娇/害羞微表情].",
      "bbox": ["y_min", "x_min", "y_max", "x_max"]
    },
    {
      "label": "object",
      "caption": "[局部特写、道具或背景的分镜，这一格不要写 characters，否则会长出多余的头]: Extreme close-up inset panel showing...",
      "bbox": ["y_min", "x_min", "y_max", "x_max"]
    },
    {
      "label": "text",
      "caption": "Clean vertical Chinese narration text inside a sharp-cornered white rectangular narration box (no bubble tail).",
      "text": "这里写旁白内容，画面里不要出现「旁白」两个字",
      "bbox": ["y_min", "x_min", "y_max", "x_max"]
    },
    {
      "label": "text",
      "caption": "Clean vertical Chinese spoken dialogue inside a rounded oval comic speech bubble with a pointer tail pointing to [Character].",
      "text": "这里是角色对话台词",
      "bbox": ["y_min", "x_min", "y_max", "x_max"]
    }
  ],
  "bbox_coord_space": "abs_centralized",
  "bbox_format": "yxyx"
}
```

## 19. 明确漫画对话气泡的说话人归属，防止气泡尾巴指错角色

> 证据等级 A

在三格漫画与单页漫画的对照实测中，当画面存在多个角色或画面外的第一人称视角说话人时，如果只写笼统的 `comic speech bubble`，`MOSS-Image` 的 DiT 空间注意力会自动在气泡边缘生成尖尾巴，并就近指向距离该 BBox 最近的任意人物头部或肩部，导致台词指错说话人。

### 19.1 气泡尾巴指错人与修气泡时人物走样的三种情况
1. **气泡离其他角色头部太近时，尾巴会自动连到最近的人头上**：
   - 当原本属于中景主角的气泡 A 的 `BBox` 坐标放在配角 B 的头顶正上方 `50px` 内，例如把主角的 `“蠢货～♡”` 气泡放在左下角配角头顶上方，即使写 `pointing to [主角]`，模型仍会因为几何距离过近，将气泡尾巴直接连在配角 B 头上。
2. **画面外的第一人称视角角色使用带尾气泡时，尾巴会连到画面内人物脸上**：
   - 当台词属于**画面外的第一人称视角角色**（如柜台对面的收银员、镜头外的提问者）时，如果使用普通带尾气泡（`speech bubble`），模型因在画面内找不到对应说话人的脸，会把尾巴连在画面内的女主或背景路人脸上。
3. **在靠前位置突然增加长描述会使后方角色的位置编码后移，导致人物走样**：
   - 在固定 `Seed` 微调气泡归属时，如果在靠前的 `items` 中突然增加数十个单词的长描述，`Qwen3.5` 从左到右的单向因果注意力会将后方角色的位置编码整体后移，导致固定 `Seed` 下原本固定的角色发型和微表情发生变化，例如中分变成齐刘海。

### 19.2 按四类说话人身份分别选用对应的气泡句式
编写 `Layout JSON` 的 `"label": "text"` 时，要按以下 4 类说话人身份分别选用句式，不要混用笼统的 `comic speech bubble`：

#### 类别 1：画面内可见角色开口说话
- **气泡位置要求**：气泡的 `bbox` 要优先贴近**该说话人自身的头部上方或左右肩侧**，并与画面内其他角色的头部保持 **≥ 80 像素** 的距离；不要把角色 A 的气泡悬在角色 B 的脑门正上方。
- **句式模板**（写明说话人、尾巴起点边与指向方向）：
  ```text
  "Clean vertical Chinese spoken dialogue spoken by [角色名] inside a rounded oval comic speech bubble, with a clear pointer tail on its [bottom / left / right] edge pointing straight [DOWN / LEFT / RIGHT] to [角色名]'s head."
  ```

#### 类别 2：画面外的第一人称视角角色说话或内心吐槽
- **气泡位置要求**：气泡应当放在画布左右两侧边缘或靠近前景第一人称视角手部的位置，远离画面中心主角的脸颊轮廓。
- **去掉气泡尾巴并写明不指向画面内角色**：要明确写出 **`tail-less oval bubble（无尾椭圆框）`**，并加上排除指向的描述 **`with NO pointer tail pointing to [画面内角色]`**。
- **句式模板**：
  ```text
  "Clean vertical Chinese [cashier / POV] inner-monologue text inside a tail-less vertical oval thought bubble (smooth oval outline with NO pointer tail pointing to the girl or the man)."
  ```

#### 类别 3：客观旁白与人物姓名年龄设定牌
- **边框形状要求**：不要使用圆角椭圆框，以免与对话气泡混淆；应当使用**直角矩形白底黑边方框**（`Sharp-cornered rectangular box`）。
- **句式模板**：
  ```text
  "Clean vertical Chinese text inside a sharp-cornered white rectangular character profile label box with a thin black border (no bubble tail)."
  ```

#### 类别 4：咆哮、情绪失控或惊叫台词
- **按字数选用气泡边框，防止吞字**：使用 **`jagged burst comic speech bubble（爆炸锯齿框）`** 时，锯齿边缘会向内侵占约 30% 的文字排版面积。
  - 如果为**短促惊叫**（2~6 个字，如 `"别！！"`、`"闭嘴！！"`）：直接使用 `jagged burst comic speech bubble`；
  - 如果为**超过 15 个字的多列长句**：不要使用 `jagged burst`，否则锯齿会吞字截断；应当改用加宽的 `wide rounded comic speech bubble`，并将 `BBox` 宽度拓宽至 `220px` 以上。

### 19.3 在日式竖排气泡中把数字改为汉字并去掉姓名空格，防止字符重叠或变成圆圈

> 证据等级 A

- **原因与现象**：在 `vertical Chinese text`（竖排汉字）气泡中，直接输入半角多位阿拉伯数字（如 `"720日元"`、`"28岁"`），会触发日漫竖排的“縦中横”挤压，导致 `7`、`2`、`0` 在同一个汉字格内重叠成乱码；人名中间如果有半角空格（如 `"白石 凛"`），空格会被误画成圆圈 `"O"`。
- **具体写法**：
  1. **竖排对话气泡或名牌框内的金额、年龄、数量**：应当改写为**中文汉字数字**，例如 `"720日元"`：改用 `"七百二十日元"`，`"28岁"`：改用 `"二十八岁"`，使每个字独占一个标准方块网格，避免数字粘连或错字；
  2. **印在商品包装盒或电子屏幕上的横排标识**：保留阿拉伯数字；
  3. **中文姓名中间不要留半角空格**：应当紧贴连写，例如 `"一色凛"`、`"白石凛"`。

### 19.4 固定随机种子修气泡时只改动出错项并控制词数变化，防止人物走样
固定 `Seed` 生成的画面、光影与人物发型已经符合预期（如 `ref_image_04` / `ref_image_08`），只需修正某几个气泡的尾巴归属或错别字时：
1. **不要修改 `summary.caption` 与负面提示词的任何字符**；
2. **不要在无需修改的前序 `items` 中成批加入长句**；
3. 只针对出错的那 1~2 个 `"label": "text"` 项，微调其 `bbox` 坐标并将 `regular speech bubble` 替换为 `tail-less vertical oval bubble (NO pointer tail)` 或指定尾巴方向，将全篇 `Token` 长度差控制在 **`±15 tokens` 以内**，就能在保持原图人脸与发型不变的同时修正气泡。
