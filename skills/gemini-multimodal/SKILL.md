---
name: gemini-multimodal
description: Generate images (Nano Banana + Imagen 4), videos (Veo), speech (TTS), and music (Lyria) using Gemini API via curl. Zero dependencies. Requires GEMINI_API_KEY (Paid Tier 1+).
---

# Gemini Multimodal Generation

Generate images, videos, speech, and music from Claude Code using the Gemini API. Zero dependencies — uses only `curl` and `python3` (macOS/Linux built-in).

## Prerequisites

- `GEMINI_API_KEY` environment variable set with a **Paid Tier 1+** key
- Free tier keys have 0 quota for all generation models
- Get a key at https://aistudio.google.com/apikey — billing must be enabled

## When to Use This Skill

- User asks to generate, create, or draw an image
- User asks to generate a video from text or image
- User asks to convert text to speech / generate audio
- User asks to generate music
- User asks to edit an image iteratively

## Available Models

| Capability | Model ID | Endpoint | Quality |
|---|---|---|---|
| Image gen (fast) | `gemini-2.5-flash-image` | generateContent | Good, fast |
| Image gen (pro) | `gemini-3-pro-image-preview` | generateContent | High quality |
| Image gen (latest) | `gemini-3.1-flash-image-preview` | generateContent | Nano Banana 2 |
| Image gen (Imagen) | `imagen-4.0-fast-generate-001` | predict | Photorealistic |
| Image gen (Imagen Ultra) | `imagen-4.0-ultra-generate-001` | predict | Best quality |
| Video gen | `veo-3.1-generate-preview` | predictLongRunning | Up to 8s, 4K |
| TTS | `gemini-2.5-flash-preview-tts` | generateContent | 30 voices |
| Music | `lyria-3-clip-preview` | generateContent | 30s clips |

---

## 1. Nano Banana Image Generation

Use for general image generation with text-and-image interleaving.

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents":[{"parts":[{"text":"PROMPT_HERE"}]}],
    "generationConfig":{"responseModalities":["TEXT","IMAGE"]}
  }' | python3 -c "
import sys,json,base64
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img=base64.b64decode(p['inlineData']['data'])
        path='OUTPUT_PATH'
        with open(path,'wb') as f: f.write(img)
        print(f'Saved: {path} ({len(img)} bytes)')
    elif 'text' in p and p['text'].strip():
        print(p['text'])
"
```

**Replace:** `PROMPT_HERE` with the image description, `OUTPUT_PATH` with the desired file path (e.g., `/tmp/output.png`).

**Model substitution:** Replace `gemini-2.5-flash-image` with `gemini-3-pro-image-preview` or `gemini-3.1-flash-image-preview` for different quality/speed tradeoffs.

---

## 2. Imagen 4 Image Generation

Use for highest quality photorealistic images. Uses the `predict` endpoint (not `generateContent`).

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/imagen-4.0-fast-generate-001:predict?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "instances":[{"prompt":"PROMPT_HERE"}],
    "parameters":{"sampleCount":1}
  }' | python3 -c "
import sys,json,base64
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
for i,pred in enumerate(d['predictions']):
    img=base64.b64decode(pred['bytesBase64Encoded'])
    path='OUTPUT_PATH'
    with open(path,'wb') as f: f.write(img)
    print(f'Saved: {path} ({len(img)} bytes)')
"
```

**Parameters:**
- `sampleCount`: 1-4 (number of image variants)
- Model variants: `imagen-4.0-fast-generate-001` (fast), `imagen-4.0-generate-001` (standard), `imagen-4.0-ultra-generate-001` (ultra)

---

## 3. Veo Video Generation

Asynchronous — submit job, poll for completion, download result.

### Step 1: Submit

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/veo-3.1-generate-preview:predictLongRunning?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "instances":[{"prompt":"PROMPT_HERE"}],
    "parameters":{"aspectRatio":"16:9","durationSeconds":4}
  }' | python3 -c "
import sys,json
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
print(d['name'])
"
```

Save the operation name (e.g., `models/veo-3.1-generate-preview/operations/xxxxx`).

**Parameters:**
- `durationSeconds`: 4, 6, or 8
- `aspectRatio`: `"16:9"` or `"9:16"`

### Step 2: Poll

```bash
curl -s "https://generativelanguage.googleapis.com/v1beta/OPERATION_NAME?key=$GEMINI_API_KEY" | python3 -c "
import sys,json
d=json.load(sys.stdin)
done=d.get('done',False)
if done and 'response' in d:
    samples=d['response'].get('generateVideoResponse',{}).get('generatedSamples',[])
    for s in samples:
        uri=s.get('video',{}).get('uri','')
        if uri: print(f'READY:{uri}')
elif done and 'error' in d:
    print(f'ERROR: {d[\"error\"]}')
else:
    print('PENDING')
"
```

Poll every 10-15 seconds until output starts with `READY:`.

### Step 3: Download

```bash
curl -s -L "VIDEO_URI&key=$GEMINI_API_KEY" -o OUTPUT_PATH
```

**Replace:** `VIDEO_URI` with the URI from step 2, `OUTPUT_PATH` with desired path (e.g., `/tmp/output.mp4`).

---

## 4. TTS (Text-to-Speech)

Generate speech audio from text. Output is PCM 24kHz mono, saved as WAV.

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-tts:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents":[{"parts":[{"text":"Say exactly this: TEXT_HERE"}]}],
    "generationConfig":{
      "responseModalities":["AUDIO"],
      "speechConfig":{"voiceConfig":{"prebuiltVoiceConfig":{"voiceName":"VOICE_NAME"}}}
    }
  }' | python3 -c "
import sys,json,base64
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        audio=base64.b64decode(p['inlineData']['data'])
        path='OUTPUT_PATH'
        with open(path,'wb') as f: f.write(audio)
        print(f'Saved: {path} ({len(audio)} bytes)')
"
```

**Available voices:** Zephyr, Puck, Charon, Fenrir, Aoede, Kore, Leda, Orus, Vega, Altair (and 20+ more).

**Replace:** `TEXT_HERE` with the text to speak, `VOICE_NAME` with chosen voice, `OUTPUT_PATH` with desired path (e.g., `/tmp/speech.wav`).

**Pro model:** Replace `gemini-2.5-flash-preview-tts` with `gemini-2.5-pro-preview-tts` for higher quality.

---

## 5. Lyria Music Generation

Generate music clips from text descriptions.

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/lyria-3-clip-preview:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents":[{"parts":[{"text":"PROMPT_HERE"}]}],
    "generationConfig":{"responseModalities":["AUDIO"]}
  }' | python3 -c "
import sys,json,base64
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        audio=base64.b64decode(p['inlineData']['data'])
        path='OUTPUT_PATH'
        with open(path,'wb') as f: f.write(audio)
        print(f'Saved: {path} ({len(audio)} bytes, {p[\"inlineData\"][\"mimeType\"]})')
    elif 'text' in p and p['text'].strip():
        print(p['text'][:200])
"
```

**Replace:** `PROMPT_HERE` with music description (e.g., "A cheerful acoustic guitar melody, upbeat tempo"), `OUTPUT_PATH` with desired path (e.g., `/tmp/music.mp3`).

**Pro model:** Replace `lyria-3-clip-preview` (30s clips) with `lyria-3-pro-preview` (full songs with lyrics).

---

## 6. Multi-turn Image Editing

Edit an existing image through conversation. Uses a temp file for the request body because base64 image data exceeds shell argument limits.

### Step 1: Build request JSON

```bash
python3 -c "
import json,base64
with open('INPUT_IMAGE_PATH','rb') as f:
    b64=base64.b64encode(f.read()).decode()
payload=json.dumps({
    'contents':[{
        'role':'user',
        'parts':[
            {'inlineData':{'mimeType':'image/png','data':b64}},
            {'text':'EDIT_INSTRUCTION'}
        ]
    }],
    'generationConfig':{'responseModalities':['TEXT','IMAGE']}
})
with open('/tmp/gemini_edit_req.json','w') as f:
    f.write(payload)
"
```

### Step 2: Send request and save result

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d @/tmp/gemini_edit_req.json | python3 -c "
import sys,json,base64
d=json.load(sys.stdin)
if 'error' in d:
    print(f'ERROR: {d[\"error\"][\"message\"]}'); sys.exit(1)
for p in d['candidates'][0]['content']['parts']:
    if 'inlineData' in p:
        img=base64.b64decode(p['inlineData']['data'])
        path='OUTPUT_PATH'
        with open(path,'wb') as f: f.write(img)
        print(f'Saved: {path} ({len(img)} bytes)')
    elif 'text' in p and p['text'].strip():
        print(p['text'])
"
```

**Replace:** `INPUT_IMAGE_PATH` with the source image, `EDIT_INSTRUCTION` with the edit command, `OUTPUT_PATH` with the output file path.

**For multi-turn editing**, include previous turns in the `contents` array when building the JSON:
```python
payload = json.dumps({
    "contents": [
        {"role":"user", "parts":[{"inlineData":{"mimeType":"image/png","data":"..."}}, {"text":"Draw a red circle"}]},
        {"role":"model", "parts":[{"inlineData":{"mimeType":"image/png","data":"..."}}]},
        {"role":"user", "parts":[{"text":"Now make it blue"}]}
    ],
    "generationConfig":{"responseModalities":["TEXT","IMAGE"]}
})
```

---

## Error Handling

| Error Code | Meaning | Action |
|---|---|---|
| 429 | Rate limit exceeded | Wait and retry. Check tier at aistudio.google.com/rate-limit |
| 403 | API key invalid or billing not enabled | Verify key and billing status |
| 404 | Model not found | Check model ID spelling |
| 400 | Invalid request | Check request body format |

## Important Notes

- All generation models require **Paid Tier 1+** billing on the API key
- Free tier keys show RPM 0/0 for all generation models
- Check your quota at https://aistudio.google.com/rate-limit
- Generated files include SynthID watermarks (invisible)
- Veo video generation is asynchronous and may take 30-120 seconds
- All other generation types are synchronous (5-30 seconds typical)
