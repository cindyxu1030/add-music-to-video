# Ducking and speech preservation: music under dialogue

Interview footage, vlogs, talking-head videos, and tutorials all share the same problem: music
generated without speech awareness ends up competing with the voice. This recipe covers two
approaches — one that prevents the conflict during generation, and one that fixes it after.
Music is **licensed and safe for commercial use (terms apply)**.

## Two approaches

| Scenario | Approach |
|---|---|
| Generating music for footage with dialogue | Use `preserve_speech` at generation time |
| You already have a music track | Use the `audio_ducking` endpoint to duck it |

Both return a final audio file ready to mux. Use one or both in sequence.

---

## Approach A: `preserve_speech` at generation time

When calling `video_to_music`, set `preserve_speech: true`. Sonilo detects speech regions in the
video and composes the music bed to sit below them — the generated track already has natural
ducking baked in. No post-processing needed.

**MCP:**

```
video_to_music({
  video_path: "interview.mp4",
  preserve_speech: true,
  style_prompt: "warm acoustic background"   // optional
})
// → "music_bed.m4a" — quiet under speech, full volume in pauses
```

**REST:**

```bash
curl -s -X POST https://api.sonilo.com/v1/video-to-music \
  -H "Authorization: Bearer $SONILO_API_KEY" \
  -F "file=@interview.mp4" \
  -F "preserve_speech=true" \
  -F "style_prompt=warm acoustic background" \
  -F "output_format=m4a"
```

Mux it:

```bash
ffmpeg -y -i interview.mp4 -i music_bed.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -b:a 320k -shortest final.mp4
```

**When to use:** footage where dialogue is the primary audio — interviews, narration, instructional
video, vlog talking segments.

---

## Approach B: duck an existing music track

If you already have a music file (library track, previously generated, client-supplied), use the
`audio_ducking` endpoint. It takes your video + music and returns a ducked version of the music
that automatically quiets under speech.

**REST:**

```bash
# Submit
RESPONSE=$(curl -s -X POST https://api.sonilo.com/v1/audio-ducking \
  -H "Authorization: Bearer $SONILO_API_KEY" \
  -F "video=@interview.mp4" \
  -F "audio=@music_track.m4a")

TASK_ID=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['task_id'])")

# Poll
while true; do
  S=$(curl -s "https://api.sonilo.com/v1/tasks/$TASK_ID" \
    -H "Authorization: Bearer $SONILO_API_KEY")
  STATE=$(echo "$S" | python3 -c "import sys,json; print(json.load(sys.stdin)['status'])")
  echo "$STATE"; [ "$STATE" = "succeeded" ] && break; sleep 5
done

AUDIO_URL=$(echo "$S" | python3 -c "import sys,json; print(json.load(sys.stdin)['audio_url'])")
curl -s -L -o ducked_music.m4a "$AUDIO_URL"
```

Mux:

```bash
ffmpeg -y -i interview.mp4 -i ducked_music.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -shortest final.mp4
```

**When to use:** you own or licensed the music already and need it to sit under speech without
manual keyframing. Also useful for matching a Sonilo-generated track to a second edit where speech
placement has changed.

---

## Combining both: generate then duck

For maximum control — generate a music bed from the video, then run ducking as a final pass:

```python
# Step 1: generate music (no preserve_speech — full-volume track first)
video_to_music(video_path="interview.mp4", style_prompt="warm cinematic documentary")
# → music_bed.m4a

# Step 2: duck the generated track against the same video
audio_ducking(video="interview.mp4", audio="music_bed.m4a")
# → ducked_music_bed.m4a
```

```bash
ffmpeg -y -i interview.mp4 -i ducked_music_bed.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -shortest final.mp4
```

This gives you a fully original, licensed track that's also precisely ducked to your exact edit.

---

## Handling existing dialogue audio in the video

If the video file already has a dialogue track you want to keep, preserve it and add the ducked
music as an additional layer:

```bash
ffmpeg -y -i interview.mp4 -i ducked_music.m4a \
  -filter_complex "[0:a][1:a]amix=inputs=2:duration=first:weights=1 0.8[aout]" \
  -map 0:v -map "[aout]" -c:v copy -c:a aac -shortest final.mp4
```

`weights=1 0.8` keeps the original dialogue at full volume and blends the music at 80%. Adjust the
music weight lower (`0.5`–`0.7`) for heavier dialogue scenes.

---

## Tips

- **`preserve_speech` vs ducking endpoint:** `preserve_speech` is one parameter added to the
  generation call — cheapest, fastest, best for new generations. The ducking endpoint is a
  separate task that costs an additional generation credit but works on any existing music file.
- **Caps:** `preserve_speech` inherits video_to_music's 6-minute cap. The ducking endpoint accepts
  videos up to 6 minutes and audio up to the same length.
- **Auto-editor and Descript users:** export your assembled cut (with embedded dialogue) before
  calling either approach — they both need the finished video to detect speech timing.
- **No speech in your video?** Skip both approaches — standard `video_to_music` without
  `preserve_speech` already fills the full dynamic range and sounds fuller.
