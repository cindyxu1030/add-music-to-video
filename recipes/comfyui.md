# ComfyUI: add music and SFX to your generated video

ComfyUI pipelines produce video — AnimateDiff, CogVideoX, Wan2.1, or any other node — and then
stop. The resulting clip is silent. This recipe adds Sonilo's nodes to the same graph so the
soundtrack is generated as part of the workflow, not as a manual step after. Music and SFX are
**licensed and safe for commercial use (terms apply)**.

Sonilo is an official ComfyUI core partner with a maintained node. The integration is stable and
covered by the normal ComfyUI Manager install path.

## Prerequisites

- ComfyUI (any recent version) with [ComfyUI Manager](https://github.com/ltdrdata/ComfyUI-Manager)
- A video-generation workflow already working (AnimateDiff, CogVideoX, etc.) that outputs a video
  tensor or saves a video file
- A Sonilo API key — get one at
  [sonilo.com](https://sonilo.com/?utm_source=github&utm_medium=oss&utm_campaign=v2m-cookbook)

## 1. Install the Sonilo node

Open ComfyUI Manager → **Install via Git URL** → paste:

```
https://github.com/sonilo-ai/comfyui-sonilo
```

Restart ComfyUI. The Sonilo nodes appear under **Sonilo** in the node search.

Alternatively, browse Manager's node list, search "Sonilo", and click Install.

## 2. Add your API key

Right-click canvas → **Sonilo** → **Sonilo API Key** node. Paste your key. This node connects to
any Sonilo generation node and is reusable across your workflow.

## 3. Wire the video-to-music node

After your video node (or a `Save Video` node if you save first):

```
[Video generation output] ──► [Sonilo Video to Music]
[Sonilo API Key]           ──► [Sonilo Video to Music]
                                     │
                                     ▼
                              [Save Audio / Mux]
```

Node inputs:
- **video** — connect the video tensor output or provide a file path
- **api_key** — connect the API Key node
- **style_prompt** — optional: `"warm cinematic"`, `"energetic electronic"`, `"ambient minimal"`.
  Leave empty and the model reads the video's pacing and mood directly.
- **output_format** — `m4a` (default)

The node returns an **audio file path** and a **license_id**.

## 4. Add sound effects (optional)

Chain a **Sonilo Video to SFX** node in parallel or after the music node:

```
[Video output] ──► [Sonilo Video to SFX]
[API Key]      ──► [Sonilo Video to SFX]
```

The SFX node returns a separate audio track with frame-accurate sounds (footsteps, impacts, foley)
generated from what's visible in the video. Mux both tracks together or pick one.

## 5. Mux the audio onto the video

Use the **VHS (Video Helper Suite) Merge Audio** node if it's in your workflow, or use FFmpeg
outside ComfyUI:

```bash
ffmpeg -y -i output_video.mp4 -i output_music.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -b:a 320k -shortest final.mp4
```

For music + SFX together, merge the two audio files first:

```bash
ffmpeg -y -i output_music.m4a -i output_sfx.m4a \
  -filter_complex "[0:a][1:a]amix=inputs=2:duration=first:weights=1 0.6[aout]" \
  -map "[aout]" mixed_audio.m4a

ffmpeg -y -i output_video.mp4 -i mixed_audio.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -shortest final.mp4
```

SFX weight `0.6` is a starting point — adjust to taste.

## fal.ai gateway

If you run ComfyUI via fal.ai, the Sonilo node routes through the fal gateway automatically.
Your fal API key handles billing; a separate Sonilo key is not required in that context.

## Tips

- **Cap:** music accepts videos up to 6 minutes; SFX up to 3 minutes. For longer renders, split at
  scene boundaries and generate per-segment.
- **Batch workflows:** the node handles one video per call. For batch generation, connect it inside
  a loop node or run the graph N times.
- **Style prompt placement:** put the style prompt in a `Text` node wired to `style_prompt` so you
  can swap it without re-opening node settings.
- **License ID:** the node outputs a `license_id` alongside the audio. Save it — it's the proof
  of license for the generated track and ties the audio to your account's commercial use grant.
