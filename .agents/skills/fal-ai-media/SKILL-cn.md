---
name: fal-ai-media
description: 通过 fal.ai MCP 实现统一的媒体生成能力，涵盖图片、视频和音频。支持文生图（Nano Banana）、文/图生视频（Seedance、Kling、Veo 3）、文生语音（CSM-1B）以及视频生音频（ThinkSound）。当用户需要使用 AI 生成图片、视频或音频时使用此技能。
---

# fal.ai 媒体生成

通过 MCP 调用 fal.ai 模型生成图片、视频和音频。

## 何时激活

- 用户想要通过文本 prompt 生成图片
- 从文本或图片创建视频
- 生成语音、音乐或音效
- 任何媒体生成任务
- 用户提到 "generate image"、"create video"、"text to speech"、"make a thumbnail" 或类似表述

## MCP 要求

必须配置 fal.ai MCP 服务器。添加到 `~/.claude.json`：

```json
"fal-ai": {
  "command": "npx",
  "args": ["-y", "fal-ai-mcp-server"],
  "env": { "FAL_KEY": "YOUR_FAL_KEY_HERE" }
}
```

在 [fal.ai](https://fal.ai) 获取 API 密钥。

## MCP 工具

fal.ai MCP 提供以下工具：
- `search` — 按关键词查找可用模型
- `find` — 获取模型详情和参数
- `generate` — 使用指定参数运行模型
- `result` — 检查异步生成状态
- `status` — 检查任务状态
- `cancel` — 取消运行中的任务
- `estimate_cost` — 预估生成成本
- `models` — 列出热门模型
- `upload` — 上传文件作为输入使用

---

## 图片生成

### Nano Banana 2（快速版）
适用场景：快速迭代、草稿、文生图、图片编辑。

```
generate(
  model_name: "fal-ai/nano-banana-2",
  input: {
    "prompt": "a futuristic cityscape at sunset, cyberpunk style",
    "image_size": "landscape_16_9",
    "num_images": 1,
    "seed": 42
  }
)
```

### Nano Banana Pro（高保真版）
适用场景：生产级图片、写实风格、文字排版、细节丰富的 prompts。

```
generate(
  model_name: "fal-ai/nano-banana-pro",
  input: {
    "prompt": "professional product photo of wireless headphones on marble surface, studio lighting",
    "image_size": "square",
    "num_images": 1,
    "guidance_scale": 7.5
  }
)
```

### 通用图片参数

| 参数 | 类型 | 可选值 | 说明 |
|-------|------|---------|-------|
| `prompt` | string | 必填 | 描述你想要生成的内容 |
| `image_size` | string | `square`, `portrait_4_3`, `landscape_16_9`, `portrait_16_9`, `landscape_4_3` | 宽高比 |
| `num_images` | number | 1-4 | 生成数量 |
| `seed` | number | 任意整数 | 可复现性 |
| `guidance_scale` | number | 1-20 | 遵循 prompt 的程度（数值越高 = 越严格匹配文本） |

### 图片编辑
使用 Nano Banana 2 配合输入图片实现局部重绘、外绘或风格迁移：

```
# 首先上传源图片
upload(file_path: "/path/to/image.png")

# 然后使用图片输入进行生成
generate(
  model_name: "fal-ai/nano-banana-2",
  input: {
    "prompt": "same scene but in watercolor style",
    "image_url": "<uploaded_url>",
    "image_size": "landscape_16_9"
  }
)
```

---

## 视频生成

### Seedance 1.0 Pro（字节跳动出品）
适用场景：具备高运动质量的文生视频、图生视频。

```
generate(
  model_name: "fal-ai/seedance-1-0-pro",
  input: {
    "prompt": "a drone flyover of a mountain lake at golden hour, cinematic",
    "duration": "5s",
    "aspect_ratio": "16:9",
    "seed": 42
  }
)
```

### Kling Video v3 Pro
适用场景：支持原生音频生成的文/图生视频。

```
generate(
  model_name: "fal-ai/kling-video/v3/pro",
  input: {
    "prompt": "ocean waves crashing on a rocky coast, dramatic clouds",
    "duration": "5s",
    "aspect_ratio": "16:9"
  }
)
```

### Veo 3（Google DeepMind 出品）
适用场景：带生成音效的视频，视觉质量高。

```
generate(
  model_name: "fal-ai/veo-3",
  input: {
    "prompt": "a bustling Tokyo street market at night, neon signs, crowd noise",
    "aspect_ratio": "16:9"
  }
)
```

### 图生视频
从现有图片开始生成：

```
generate(
  model_name: "fal-ai/seedance-1-0-pro",
  input: {
    "prompt": "camera slowly zooms out, gentle wind moves the trees",
    "image_url": "<uploaded_image_url>",
    "duration": "5s"
  }
)
```

### 视频参数

| 参数 | 类型 | 可选值 | 说明 |
|-------|------|---------|-------|
| `prompt` | string | 必填 | 描述视频内容 |
| `duration` | string | `"5s"`, `"10s"` | 视频时长 |
| `aspect_ratio` | string | `"16:9"`, `"9:16"`, `"1:1"` | 帧比例 |
| `seed` | number | 任意整数 | 可复现性 |
| `image_url` | string | URL | 图生视频的源图片 |

---

## 音频生成

### CSM-1B（对话式语音）
具备自然对话质感的文生语音能力。

```
generate(
  model_name: "fal-ai/csm-1b",
  input: {
    "text": "Hello, welcome to the demo. Let me show you how this works.",
    "speaker_id": 0
  }
)
```

### ThinkSound（视频生音频）
根据视频内容生成匹配的音频。

```
generate(
  model_name: "fal-ai/thinksound",
  input: {
    "video_url": "<video_url>",
    "prompt": "ambient forest sounds with birds chirping"
  }
)
```

### ElevenLabs（通过 API 调用，无需 MCP）
如需专业语音合成，直接使用 ElevenLabs：

```python
import os
import requests

resp = requests.post(
    "https://api.elevenlabs.io/v1/text-to-speech/<voice_id>",
    headers={
        "xi-api-key": os.environ["ELEVENLABS_API_KEY"],
        "Content-Type": "application/json"
    },
    json={
        "text": "Your text here",
        "model_id": "eleven_turbo_v2_5",
        "voice_settings": {"stability": 0.5, "similarity_boost": 0.75}
    }
)
with open("output.mp3", "wb") as f:
    f.write(resp.content)
```

### VideoDB 生成式音频
如果已配置 VideoDB，可使用其生成式音频能力：

```python
# 语音生成
audio = coll.generate_voice(text="Your narration here", voice="alloy")

# 音乐生成
music = coll.generate_music(prompt="upbeat electronic background music", duration=30)

# 音效
sfx = coll.generate_sound_effect(prompt="thunder crack followed by rain")
```

---

## 成本预估
生成前可查看预估成本：

```
estimate_cost(model_name: "fal-ai/nano-banana-pro", input: {...})
```

## 模型查找
查找适合特定任务的模型：

```
search(query: "text to video")
find(model_name: "fal-ai/seedance-1-0-pro")
models()
```

## 提示
- 迭代 prompt 时使用 `seed` 获得可复现的结果
- prompt 迭代阶段优先使用成本更低的模型（Nano Banana 2），最终产出时切换到 Pro 版本
- 生成视频时，prompt 要描述清晰但简洁——重点关注动作和场景
- 图生视频比纯文生视频的结果更可控
- 运行成本较高的视频生成任务前先检查 `estimate_cost`

## 相关技能
- `videodb` — 视频处理、编辑和流媒体
- `video-editing` — AI 驱动的视频编辑 workflow
- `content-engine` — 社交平台内容创作
