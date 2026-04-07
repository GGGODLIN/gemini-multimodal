# gemini-multimodal

A Claude Code skill for multimodal generation via Gemini API. Zero dependencies — pure `curl` + `python3`.

## Capabilities

| Capability | Model | Output |
|---|---|---|
| Image Generation | Nano Banana (gemini-2.5-flash-image) | PNG |
| Image Generation | Imagen 4 (imagen-4.0-fast/ultra) | PNG |
| Video Generation | Veo 3.1 | MP4 |
| Text-to-Speech | Gemini 2.5 Flash/Pro TTS | WAV |
| Music Generation | Lyria 3 | MP3 |
| Image Editing | Nano Banana (multi-turn) | PNG |

## Prerequisites

- **GEMINI_API_KEY** environment variable with a **Paid Tier 1+** key
- Free tier keys have 0 quota for all generation models
- Get a key at [Google AI Studio](https://aistudio.google.com/apikey) — billing must be enabled

## Install

```bash
npx skills add GGGODLIN/gemini-multimodal -g -y
```

Or install for current project only:

```bash
npx skills add GGGODLIN/gemini-multimodal -y
```

## Usage

Once installed, Claude Code will automatically use this skill when you ask it to generate images, videos, speech, or music.

**Examples:**

- "Generate an image of a sunset over mountains"
- "Create a 4-second video of a cat walking on a beach"
- "Convert this text to speech using the Puck voice"
- "Generate a lo-fi hip hop beat"
- "Edit this image: remove the background and make it transparent"

## How It Works

This skill teaches Claude Code the exact `curl` commands to call Gemini API endpoints. No MCP server, no npm packages, no extra tooling — just shell commands that work on any macOS/Linux system.

| Capability | API Endpoint | Method |
|---|---|---|
| Nano Banana | `generateContent` | Synchronous |
| Imagen 4 | `predict` | Synchronous |
| Veo | `predictLongRunning` | Async (poll + download) |
| TTS | `generateContent` | Synchronous |
| Lyria | `generateContent` | Synchronous |
| Image Editing | `generateContent` | Synchronous (temp file for base64) |

## Pricing Note

All generation models require Paid Tier 1+ billing. As of April 2026:

| Model | Approximate Cost |
|---|---|
| Nano Banana | ~$0.02-0.04 / image |
| Imagen 4 | $0.02-0.06 / image |
| Veo 3.1 | $0.05-0.60 / second |
| TTS | $10 / 1M output tokens |
| Lyria | $0.04-0.08 / clip |

Check your quota at [AI Studio Rate Limit](https://aistudio.google.com/rate-limit).

## License

MIT

---

# gemini-multimodal（繁體中文）

Claude Code 的多模態生成技能，透過 Gemini API 實現。零依賴 — 純 `curl` + `python3`。

## 功能

| 功能 | 模型 | 輸出 |
|---|---|---|
| 圖片生成 | Nano Banana (gemini-2.5-flash-image) | PNG |
| 圖片生成 | Imagen 4 (imagen-4.0-fast/ultra) | PNG |
| 影片生成 | Veo 3.1 | MP4 |
| 語音合成 | Gemini 2.5 Flash/Pro TTS | WAV |
| 音樂生成 | Lyria 3 | MP3 |
| 圖片編輯 | Nano Banana（多輪對話） | PNG |

## 前置需求

- 設定 **GEMINI_API_KEY** 環境變數，需要 **Paid Tier 1+** 的金鑰
- 免費金鑰的生成模型配額為 0，無法使用
- 至 [Google AI Studio](https://aistudio.google.com/apikey) 取得金鑰，需啟用帳單功能

## 安裝

```bash
npx skills add GGGODLIN/gemini-multimodal -g -y
```

或僅安裝至目前專案：

```bash
npx skills add GGGODLIN/gemini-multimodal -y
```

## 使用方式

安裝後，當你要求 Claude Code 生成圖片、影片、語音或音樂時，它會自動使用此技能。

**範例：**

- 「幫我生成一張夕陽下的山景圖」
- 「建立一段 4 秒的貓在沙灘上走路的影片」
- 「把這段文字轉成語音，用 Puck 的聲音」
- 「生成一段 lo-fi 嘻哈節拍」
- 「編輯這張圖片：把背景移除」

## 運作原理

此技能教 Claude Code 如何用 `curl` 呼叫 Gemini API 端點。不需要 MCP server、不需要 npm 套件、不需要額外工具 — 只用 macOS/Linux 內建的 shell 指令。

| 功能 | API 端點 | 方式 |
|---|---|---|
| Nano Banana | `generateContent` | 同步 |
| Imagen 4 | `predict` | 同步 |
| Veo | `predictLongRunning` | 非同步（輪詢 + 下載） |
| TTS | `generateContent` | 同步 |
| Lyria | `generateContent` | 同步 |
| 圖片編輯 | `generateContent` | 同步（用暫存檔傳 base64） |

## 計費說明

所有生成模型需要 Paid Tier 1+ 帳單。截至 2026 年 4 月：

| 模型 | 大約費用 |
|---|---|
| Nano Banana | ~$0.02-0.04 / 張 |
| Imagen 4 | $0.02-0.06 / 張 |
| Veo 3.1 | $0.05-0.60 / 秒 |
| TTS | $10 / 1M 輸出 tokens |
| Lyria | $0.04-0.08 / 段 |

可至 [AI Studio Rate Limit](https://aistudio.google.com/rate-limit) 查看配額。

## 授權

MIT
