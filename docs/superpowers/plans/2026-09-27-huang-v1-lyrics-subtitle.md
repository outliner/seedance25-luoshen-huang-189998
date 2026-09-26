# 《荒》二次元 V1 歌词字幕版 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 为任务 `189998` 的30秒二次元母带制作可编辑 ASS、烧录歌词字幕版、视觉验收材料和生成记录，并上传现有 GitHub 项目。

**Architecture:** 先把原歌曲的六句歌词整体后移4秒写成独立 ASS，以 Windows 宋体和安全区样式烧录到 H.264 视频，同时复制原 AAC 音轨。随后完整解码成片，以前奏、六句歌词和尾奏代表时点生成联系表，记录技术规格、哈希和视觉结论后提交。

**Tech Stack:** ASS、FFmpeg/libass、FFprobe、PowerShell、Git。

## Global Constraints

- 输入固定为 `Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4`，不得覆盖或改写。
- 输出固定为 `Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4`。
- 六句歌词时间固定为4.00–7.18、7.18–9.30、9.30–13.46、14.06–17.14、17.14–19.80、20.18–23.48秒。
- 歌词保留“林霄”和“何道”；不加标题、对白、Logo或卡拉OK逐字效果。
- 字幕为底部居中冷调浅蓝白宋体（RGB `#E2EEF3`，ASS `&H00F3EEE2`）、暗青黑描边与轻阴影，底边距150像素；此颜色经用户审阅后批准。
- 输出必须保持720×1280、30.000秒、AAC 48kHz双声道；烧录视频允许重新编码，音频流复制。
- 原无字幕母带必须保持原SHA-256不变。

---

### Task 1: 创建并预检 ASS 字幕

**Files:**
- Read: `Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4`
- Create: `荒_V1_歌词字幕.ass`
- Create: `任务189998_歌词字幕版_字幕预检.jpg`

**Interfaces:**
- Consumes: 720×1280、30秒无字幕母带和批准的六句歌词时间。
- Produces: UTF-8 ASS文件，供Task 2的FFmpeg `subtitles`滤镜读取。

- [ ] **Step 1: 记录输入母带哈希和媒体规格**

```powershell
Get-FileHash -Algorithm SHA256 '.\Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4'
ffprobe -v error -show_entries format=duration,size:stream=codec_type,codec_name,width,height,r_frame_rate,sample_rate,channels -of json '.\Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4'
```

预期：视频720×1280，音频AAC 48kHz双声道，总时长30.000秒。保存输入哈希供Task 3复核。

- [ ] **Step 2: 验证字幕尚未存在**

```powershell
Test-Path -LiteralPath '.\荒_V1_歌词字幕.ass'
```

预期：新工作树中返回`False`，证明后续验证不是复用旧字幕文件。

- [ ] **Step 3: 使用 `apply_patch` 创建完整 ASS**

```ass
[Script Info]
Title: Huang V1 Lyric Subtitles
ScriptType: v4.00+
PlayResX: 720
PlayResY: 1280
ScaledBorderAndShadow: yes
WrapStyle: 2

[V4+ Styles]
Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
Style: Lyric,SimSun,42,&H00F3EEE2,&H00F3EEE2,&H00251F18,&H78000000,0,0,0,0,100,100,1.5,0,1,3.2,1.2,2,54,54,150,1

[Events]
Format: Layer, Start, End, Style, Name, MarginL, MarginR, MarginV, Effect, Text
Dialogue: 0,0:00:04.00,0:00:07.18,Lyric,,0,0,0,,一念间天荒仙灭
Dialogue: 0,0:00:07.18,0:00:09.30,Lyric,,0,0,0,,我仍立八荒间
Dialogue: 0,0:00:09.30,0:00:13.46,Lyric,,0,0,0,,踏碎林霄又葬下了天
Dialogue: 0,0:00:14.06,0:00:17.14,Lyric,,0,0,0,,我欲横剑天啸
Dialogue: 0,0:00:17.14,0:00:19.80,Lyric,,0,0,0,,此去沧海落扶摇
Dialogue: 0,0:00:20.18,0:00:23.48,Lyric,,0,0,0,,万古梦回涅槃何道
```

- [ ] **Step 4: 机械检查ASS结构和时间**

```powershell
$ass = Get-Content -Raw -LiteralPath '.\荒_V1_歌词字幕.ass'
if (($ass | Select-String -Pattern '^Dialogue:' -AllMatches).Matches.Count -ne 6) { throw 'Expected six dialogue lines' }
rg -n '4\.00|7\.18|9\.30|13\.46|14\.06|17\.14|19\.80|20\.18|23\.48|林霄|何道' '.\荒_V1_歌词字幕.ass'
```

预期：六条Dialogue；所有批准时间和字词均命中。

- [ ] **Step 5: 渲染第三句代表帧验证字体链路**

```powershell
ffmpeg -hide_banner -loglevel error -y -ss 10.5 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4' -vf "subtitles='荒_V1_歌词字幕.ass':fontsdir='C\:/Windows/Fonts'" -frames:v 1 -q:v 2 '.\任务189998_歌词字幕版_字幕预检.jpg'
```

预期：命令退出0；预检图显示完整“踏碎林霄又葬下了天”，无方框、乱码或截断。目视不通过时只调整字体、字号、边距、描边或阴影，不改歌词和时间轴。

- [ ] **Step 6: 提交ASS**

```powershell
git add -- '荒_V1_歌词字幕.ass'
git commit -m "Add Huang v1 lyric subtitles"
```

### Task 2: 烧录字幕并验证媒体

**Files:**
- Read: `荒_V1_歌词字幕.ass`
- Read: `Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4`
- Create: `Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4`

**Interfaces:**
- Consumes: Task 1已目视通过的ASS。
- Produces: H.264字幕烧录视频和复制的原AAC音轨，供Task 3视觉验收。

- [ ] **Step 1: 烧录字幕，不改音轨**

```powershell
ffmpeg -hide_banner -loglevel error -y -i '.\Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4' -vf "subtitles='荒_V1_歌词字幕.ass':fontsdir='C\:/Windows/Fonts'" -c:v libx264 -preset slow -crf 16 -pix_fmt yuv420p -c:a copy -movflags +faststart '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4'
```

- [ ] **Step 2: 完整解码输出**

```powershell
ffmpeg -hide_banner -loglevel error -v error -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -f null NUL
```

预期：退出0且无解码错误。

- [ ] **Step 3: 验证输出规格**

```powershell
ffprobe -v error -show_entries format=duration,size:stream=codec_type,codec_name,width,height,r_frame_rate,sample_rate,channels -of json '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4'
```

预期：H.264 720×1280；AAC 48kHz双声道；30.000秒。

### Task 3: 视觉验收、记录和Git交付

**Files:**
- Create: `任务189998_歌词字幕版_审片联系表.jpg`
- Create: `字幕生成记录_任务189998.md`
- Modify: Git repository history only; do not modify untracked user assets.

**Interfaces:**
- Consumes: Task 2成片和Task 1输入母带哈希。
- Produces: 可下载字幕版、QA证据、生成记录和Git链接。

- [ ] **Step 1: 生成九格字幕联系表**

在1.0、5.0、8.0、11.0、15.0、18.0、22.0、24.5、29.0秒抽帧，按3×3从左到右、从上到下排列。1.0与29.0秒必须无字幕，其余代表帧覆盖六句歌词和一句间空白。

```powershell
ffmpeg -hide_banner -loglevel error -y -ss 1.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 5.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 8.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 11.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 15.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 18.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 22.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 24.5 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -ss 29.0 -i '.\Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' -filter_complex "[0:v]scale=240:426[a];[1:v]scale=240:426[b];[2:v]scale=240:426[c];[3:v]scale=240:426[d];[4:v]scale=240:426[e];[5:v]scale=240:426[f];[6:v]scale=240:426[g];[7:v]scale=240:426[h];[8:v]scale=240:426[i];[a][b][c][d][e][f][g][h][i]xstack=inputs=9:layout=0_0|240_0|480_0|0_426|240_426|480_426|0_852|240_852|480_852" -frames:v 1 -q:v 2 '.\任务189998_歌词字幕版_审片联系表.jpg'
```

- [ ] **Step 2: 目视验收联系表**

检查：前奏和尾奏无字幕；六句各出现一次；用户审阅批准的冷调浅蓝白宋体（RGB `#E2EEF3`，ASS `&H00F3EEE2`）、暗色描边、位置和宽度一致；无缺字、乱码、裁切或遮脸。若失败，返回Task 1只改样式并重新执行Task 2与Task 3。

- [ ] **Step 3: 验证原母带未变化**

```powershell
Get-FileHash -Algorithm SHA256 '.\Seedance2.5_荒_渠道Y_任务189998_母带成片.mp4'
```

预期：与Task 1记录完全一致。

- [ ] **Step 4: 创建生成记录**

使用`apply_patch`创建`字幕生成记录_任务189998.md`，逐项记录：输入和输出文件、字幕时间来源、六句时间、字体与样式、FFprobe规格、完整解码结果、输入/输出SHA-256、联系表目视结论和已知字词边界。

- [ ] **Step 5: 最终验证并提交**

```powershell
git add -- '荒_V1_歌词字幕.ass' 'Seedance2.5_荒_渠道Y_任务189998_母带歌词字幕版.mp4' '任务189998_歌词字幕版_审片联系表.jpg' '字幕生成记录_任务189998.md'
git commit -m "Add Huang v1 lyric subtitle edition"
git status -sb
```

预期：只有已有用户未跟踪素材保持未跟踪；本任务四个交付文件均已提交。

- [ ] **Step 6: 上传现有GitHub仓库并确认远端提交**

```powershell
git push origin main
git ls-remote --heads origin main
git rev-parse HEAD
```

预期：远端`main`和本地HEAD为同一提交，随后返回仓库链接和字幕版直接下载链接。
