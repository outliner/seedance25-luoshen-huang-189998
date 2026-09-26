# 《荒·折天开海》Seedance 第二版生成执行计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 使用四张参考图和10镜节奏重构生成《荒·折天开海》第二版30秒Seedance成片，并验证故事是否在规定时间进入日出开海和人物收束。

**Architecture:** 先根据身份图与开海场景图制作一张带洛神的终场参考图，再把已批准的10镜设计转换为紧凑Seedance执行提示词。通过Guagu原生OpenAPI顺序上传四张图片，以前三张为`reference_image`、终场图为`last_frame`，估价通过后只创建一个固定幂等任务；成功后下载原片、替换30秒母带并做关键时点抽帧验收。

**Tech Stack:** 内置ImageGen、PowerShell、Python 3.10、`guagu_video.py`、Guagu `/openapi/v1`、FFmpeg/FFprobe、Git。

## Global Constraints

- 模型固定为渠道Y `seedance55`；若实时模型目录不再返回该模型，停止并报告，不自动换挡位。
- 输出固定为720p、9:16、30秒；生成前必须实时查询余额并执行estimate。
- 参考图固定为四张：人物身份、墨沙穹顶、竖海长堤、新终场图。
- `01_luoshen_identity.png`、`02_ink_vault.png`、`05_vertical_sea.png`使用`reference_image`；`10_open_sea_luoshen_final.png`使用`last_frame`。
- 第二版幂等键固定为`luoshen-huang-zhetian-kaihai-v2-20260927-channel-y-v1`；结果不确定时仅用相同参数复用此键。
- 渠道Y不上传参考音频；下载后以`audio/MV_30s_master.wav`替换音轨。
- 不自动发起第三次付费生成。第二版若不达标，记录偏差并等待用户决定。
- 不输出、记录或提交`GUAGU_API_KEY`。

---

### Task 1: 制作终场参考图

**Files:**
- Read: `assets/01_luoshen_identity.png`
- Read: `assets/07_open_sea_dawn.png`
- Create: `assets_v2/10_open_sea_luoshen_final.png`

**Interfaces:**
- Consumes: 人物身份图与无人物日出开海场景图。
- Produces: 一张9:16终场画板，供API以`last_frame`角色上传。

- [ ] **Step 1: 检查两张输入图**

使用`view_image`分别确认：身份图确有唯一成年洛神三视图；场景图具有青黑岩台、开阔海面与琥珀日出，前景留有完整站立平台。

- [ ] **Step 2: 调用ImageGen制作终场图**

以两张本地图片作为参考，使用以下完整指令：

```text
创建一张竖屏9:16东方手绘动画插画终场画板。严格继承第一张参考图中唯一成年洛神的成熟柔和五官、黑色超长半挽发、白花头饰、浅青发带与金色流苏、象牙白和浅青层叠广袖汉服、白花刺绣、浅青绣鞋；画面只能出现一个洛神，不能出现三视图、复制人物或文字。严格继承第二张参考图的日出开海环境：青黑岩石平台、广阔海湾、远山、两侧云幕与中央琥珀色日出。洛神完整站稳在前景平台中央偏下，身体侧前朝向日出，双脚与石面接触清楚，裙摆落在平台上且不穿地；她刚经历压迫后抵达开放天地，姿态放松但不软弱，头部轻微回望镜头旁，嘴角有克制而明亮的笑意。中近全身构图，人物约占画高45%，肩后海平线与日出可辨，发带和长发向后轻摆。保持青黑、浅青、象牙白、琥珀金统一色板，空间透视清楚，不转真人摄影或塑料3D，不要武器、翅膀、文字、水印、灰色占位人或无关角色。
```

- [ ] **Step 3: 保存并目视验收**

保存为`assets_v2/10_open_sea_luoshen_final.png`。验收必须同时满足：唯一角色、身份和服饰正确、全身站稳、平台完整、日出海面可辨、无文字水印、无灰色占位。

- [ ] **Step 4: 提交终场图**

```powershell
git add -- 'assets_v2/10_open_sea_luoshen_final.png'
git commit -m "Add Huang v2 final-frame reference"
```

### Task 2: 编写并验证第二版Seedance提示词

**Files:**
- Read: `docs/superpowers/specs/2026-09-27-huang-v2-seedance-design.md`
- Create: `Seedance2.5_30秒完整提示词_V2.txt`

**Interfaces:**
- Consumes: 已批准的10镜时间轴和四图职责。
- Produces: 可直接作为API `prompt`参数的UTF-8纯文本。

- [ ] **Step 1: 创建完整执行提示词**

提示词必须按以下结构写入文件，不加入第11镜或第五张参考图：

```text
《荒·折天开海》成年洛神MV第二版，30秒，竖屏9:16，目标720p，东方手绘动画插画质感。

【四图职责】
@图片1只锁定唯一成年洛神的身份、成熟五官、黑色超长半挽发、白花头饰、浅青发带、金色流苏、象牙白与浅青层叠广袖汉服、白花刺绣和浅青绣鞋；三视图是同一人物，不生成多人。@图片2只锁定开场墨沙穹顶、六片暗金弧板和青黑金色板。@图片3只锁定竖海长堤的明确水墙边界、干燥堤面和空间尺度。@图片4是27–30秒必须到达的唯一终场构图与尾帧目标。四图不是依次轮播；八荒环台、倒悬天宫和云层天井由模型在统一画风下自行设计。

【全局故事与节奏】
唯一故事：洛神从封闭压迫中以稳定承重触发世界逐层打开，经过八荒、天宫和竖海，最终抵达开放海天。共10镜，切点严格为2、5、8、12、15、18、20、23、27秒。5秒前开天完成；12秒前离开天宫；18秒前触水、水拱完成并离开竖海；23秒进入开海；27秒进入最终人物镜头。中间空间若来不及，宁可缩短八荒、天宫或云井，绝不推迟23–30秒终场。

0–2秒，S01：侧前中近景，洛神已坐稳干燥黑石台，第一帧同时看见她的脸与身后快速掠动的六片弧板。人物清晰、背景快，她抬眼迎向一束金光，相机锁定。

2–5秒，S02：硬切低位中全景，承接坐姿与右掌支撑。右掌已压实，金光沿石缝外扩；六片弧板随即依次绕外侧轴翻开，露出倒悬山河。相机后退揭示尺度，5秒前全部打开；人物不站起，石台不动。

5–8秒，S03：以前景弧板全遮硬切八荒环台。洛神开场已站稳完整宽台，身体面向画右；相机向右侧移，揭示上方倒悬山河和下方荒原。8秒前完整读到人、台和天地夹层，她不迈步。

8–12秒，S04：硬切倒悬天宫大全景。洛神已站在水平宽廊前方；相机沿廊道中轴高速穿过两个真实空梁框，最后一秒减速。远处非承重宫阙向外分层，脚下地面不转动。12秒前近处深色立柱扫满画面。

12–15秒，S05：同色立柱遮挡切入竖海。洛神已半跪干燥堤面，左膝稳定，右手接近数十米高的竖直水墙；相机短距离侧移后停，15秒前看清人、堤、水三者关系，指尖仍差少量距离。

15–18秒，S06：同一侧前中远景，不切局部特写。右手向前极短距离，指尖触水形成圆形波纹；水墙上缘立即向远离她的一侧卷成巨大水拱，相机同步后退。水不扑向人物，18秒前水拱成形并立即切离竖海。

18–20秒，S07：在水拱弧线上硬切同位置的云层天井弧边，水不变成云。无人物短镜，相机沿中央空隙快速上升，三圈云环向外打开，20秒前中央金色天光清楚可见。

20–23秒，S08：硬切中全景，洛神已站稳天井底部完整石台，右手迎光抬至肩高。相机从平视升到略俯视，云隙打开为通道。23秒前相机进入短段不透明灰青云，准备切入终场。

23–27秒，S09：从灰青云中直接穿出进入日出开海。洛神已站在青黑平台中央；相机越过近处低云后转为升高后退，依次揭示海湾、远山、两侧云幕和中央日出。27秒前必须完整到达@图片4的终场空间，人物不复制、不飞起。

27–30秒，S10：明确硬切侧前中近景，仍在同一终场平台。洛神把视线从日出移向镜头旁，下颌放松，露出克制而明亮的笑意；相机固定，肩后海平线稳定，最后一秒只保留表情、呼吸和逐渐减弱的发带摆动，最终画面收束到@图片4。

【统一禁止】
禁止新增人物、三视图多人化、灰色占位、分身、换装、武器、翅膀、异常手指、脚滑或浮空、袖裙穿模、随机建筑液化、手水融合、反射分身、字幕、歌词、文字、水印、频闪和白闪。歌曲只作节奏依据，无演唱、无对白，音乐后期铺设。
```

- [ ] **Step 2: 验证提示词长度与时间轴**

```powershell
$prompt = Get-Content -Raw -LiteralPath '.\Seedance2.5_30秒完整提示词_V2.txt'
$prompt.Length
rg -n '0–2秒|2–5秒|5–8秒|8–12秒|12–15秒|15–18秒|18–20秒|20–23秒|23–27秒|27–30秒' '.\Seedance2.5_30秒完整提示词_V2.txt'
```

预期：字符数不超过2,500；10个时间段各出现一次并连续覆盖0–30秒。

- [ ] **Step 3: 提交提示词**

```powershell
git add -- 'Seedance2.5_30秒完整提示词_V2.txt'
git commit -m "Add Huang v2 Seedance prompt"
```

### Task 3: API预检并创建唯一第二版任务

**Files:**
- Read: `Seedance2.5_30秒完整提示词_V2.txt`
- Read: `assets/01_luoshen_identity.png`
- Read: `assets/02_ink_vault.png`
- Read: `assets/05_vertical_sea.png`
- Read: `assets_v2/10_open_sea_luoshen_final.png`

**Interfaces:**
- Consumes: 完整提示词和四张已验收图片。
- Produces: 一个任务ID及四个`asset://`句柄；不创建重复任务。

- [ ] **Step 1: 实时检查模型与余额**

```powershell
$env:GUAGU_API_KEY = [Environment]::GetEnvironmentVariable('GUAGU_API_KEY','User')
$cli = 'C:\Users\32780\.codex\skills\guagu-video-api\scripts\guagu_video.py'
python $cli models --type image_to_video
python $cli balance
```

预期：模型目录包含精确ID`seedance55`，余额至少3.5点，冻结余额不参与可用额度计算。

- [ ] **Step 2: 顺序上传四张图片**

```powershell
python $cli upload '.\assets\01_luoshen_identity.png' --kind image
python $cli upload '.\assets\02_ink_vault.png' --kind image
python $cli upload '.\assets\05_vertical_sea.png' --kind image
python $cli upload '.\assets_v2\10_open_sea_luoshen_final.png' --kind image
```

记录四个返回的`asset://`句柄，保持顺序为身份、穹顶、竖海、终场。

- [ ] **Step 3: 构造请求并执行estimate**

使用前三个句柄的角色`reference_image`，第四个句柄的角色`last_frame`；模型`seedance55`、分辨率`720p`、比例`9:16`、时长`30`、`--no-generate-audio`。estimate必须返回3.5点或实时目录给出的当前价格，且不产生任务。

- [ ] **Step 4: 创建唯一任务**

使用与estimate完全相同的参数，并加入：

```text
--idempotency-key luoshen-huang-zhetian-kaihai-v2-20260927-channel-y-v1
```

记录返回的`task_id`。若响应不确定，仅复用相同幂等键和相同参数重试一次；不得重新上传后换键创建另一任务。

### Task 4: 等待、下载、母带替换与节奏验收

**Files:**
- Create: `Seedance2.5_荒_V2_渠道Y_任务${taskId}_模型原片.mp4`
- Create: `Seedance2.5_荒_V2_渠道Y_任务${taskId}_母带成片.mp4`
- Create: `任务${taskId}_V2_审片联系表.jpg`
- Create: `任务${taskId}_V2_节奏闸门验收.jpg`
- Create: `生成记录_任务${taskId}_V2.md`

**Interfaces:**
- Consumes: Task 3返回的运行时`taskId`和`audio/MV_30s_master.wav`。
- Produces: 第二版原片、母带成片、两张QA图和完整生成记录。

- [ ] **Step 1: 轮询到终态**

```powershell
python $cli wait $taskId --interval 7
```

只接受`succeeded`后下载；`failed`时以`billing_status`与`final_cost`判断是否退款，不自动重提。

- [ ] **Step 2: 下载模型原片**

```powershell
$original = ".\Seedance2.5_荒_V2_渠道Y_任务${taskId}_模型原片.mp4"
python $cli download $taskId $original
```

下载完成后用`ffprobe`确认视频流存在且文件非空。

- [ ] **Step 3: 替换30秒母带**

```powershell
$final = ".\Seedance2.5_荒_V2_渠道Y_任务${taskId}_母带成片.mp4"
ffmpeg -hide_banner -loglevel error -i $original -i '.\audio\MV_30s_master.wav' -map 0:v:0 -map 1:a:0 -c:v copy -c:a aac -b:a 320k -shortest -movflags +faststart -- $final
ffprobe -v error -show_entries format=duration,size:stream=codec_type,codec_name,width,height,sample_rate,channels -of json -- $final
```

预期：H.264视频720×1280，AAC音频48kHz双声道，总时长30.000秒。

- [ ] **Step 4: 生成全片联系表**

```powershell
$sheet = ".\任务${taskId}_V2_审片联系表.jpg"
ffmpeg -hide_banner -loglevel error -i $final -vf "fps=0.4,scale=240:-1,tile=4x3" -frames:v 1 -q:v 2 -- $sheet
```

- [ ] **Step 5: 生成节奏闸门验收图**

分别抽取18秒、23秒、27秒、29.5秒并横向拼接为`任务${taskId}_V2_节奏闸门验收.jpg`。通过要求：18秒后不再停留于触水准备；23秒出现开海入口或已进入开海；27秒位于终场空间；29.5秒是人物收束而非竖海触水。

- [ ] **Step 6: 写生成记录**

记录任务ID、幂等键、模型、四个素材句柄、estimate与final_cost、billing_status、视频规格、母带处理和逐项QA结论。若关键闸门失败，明确写“第二版未通过完整故事验收”，不得仅因API状态`succeeded`而标记方案成功。

- [ ] **Step 7: 最终提交**

```powershell
git add -- "Seedance2.5_荒_V2_渠道Y_任务${taskId}_母带成片.mp4" "任务${taskId}_V2_审片联系表.jpg" "任务${taskId}_V2_节奏闸门验收.jpg" "生成记录_任务${taskId}_V2.md"
git commit -m "Add Huang Seedance v2 result"
```

提交仅保留本地，除非用户另行要求上传GitHub。
