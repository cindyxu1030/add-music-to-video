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

Approach A returns audio tracks to mux yourself; Approach B returns the finished mix (a new `.mp4`
when the voice input is a video). Use one or both in sequence.

---

## Approach A: `preserve_speech` at generation time

When calling `video_to_music` on footage with dialogue, set `preserve_speech: true` to keep the
source speech audible in the result (you also get a `vocals` speech stem and a ready-mixed `mux`),
and `ducking: true` to duck the generated music under the voice — this adds a `ducked` track
alongside the clean `audio` track. No post-processing needed.

**MCP:**

```
video_to_music({
  video_path: "interview.mp4",
  preserve_speech: true,
  ducking: true,
  prompt: "warm acoustic background"   // optional
})
// → clean music + a ducked track (quiet under speech), plus a mux with the speech already mixed in
```

**REST:**

```bash
curl -s -X POST https://api.sonilo.com/v1/video-to-music \
  -H "Authorization: Bearer $SONILO_API_KEY" \
  -F "video=@interview.mp4" \
  -F "preserve_speech=true" \
  -F "ducking=true" \
  -F "prompt=warm acoustic background" \
  -F "mode=async" \
  -F "output_format=m4a"
```

This returns `{"task_id": ...}` — poll `GET /v1/tasks/{task_id}` as in the
[FFmpeg-only recipe](./ffmpeg-only.md). The clean music is `audio[0].url`; the ducked track is
`ducked[0].url`. Save the ducked one as `music_bed.m4a`, then mux it:

```bash
ffmpeg -y -i interview.mp4 -i music_bed.m4a \
  -map 0:v -map 1:a -c:v copy -c:a aac -b:a 320k -shortest final.mp4
```

**When to use:** footage where dialogue is the primary audio — interviews, narration, instructional
video, vlog talking segments.

---

## Approach B: duck an existing music track

If you already have a music file (library track, previously generated, client-supplied), use the
`audio_ducking` endpoint. It takes a voice input (audio, or a video — its audio track is used as the
voice) plus a music file, lowers the music wherever the voice speaks, and returns the mixed result.
With a video as the voice input, the ducked mix comes back already muxed into a new `.mp4`.

**REST:**

```bash
# Submit
RESPONSE=$(curl -s -X POST https://api.sonilo.com/v1/audio-ducking \
  -H "Authorization: Bearer $SONILO_API_KEY" \
  -F "voice_file=@interview.mp4" \
  -F "music_file=@music_track.m4a")

TASK_ID=$(echo "$RESPONSE" | python3 -c "import sys,json; print(json.load(sys.stdin)['task_id'])")

# Poll
while true; do
  S=$(curl -s "https://api.sonilo.com/v1/tasks/$TASK_ID" \
    -H "Authorization: Bearer $SONILO_API_KEY")
  STATE=$(echo "$S" | python3 -c "import sys,json; print(json.load(sys.stdin)['status'])")
  echo "$STATE"; [ "$STATE" = "succeeded" ] && break; sleep 5
done

OUT_URL=$(echo "$S" | python3 -c "import sys,json; print(json.load(sys.stdin)['output_url'])")
curl -s -L -o final.mp4 "$OUT_URL"
```

No ffmpeg step: because the voice input was a video, `final.mp4` already carries the ducked mix
(`output_type` is `video`). Pass an audio file as `voice_file` instead to get a `.wav` mix back.

**When to use:** you own or licensed the music already and need it to sit under speech without
manual keyframing. Also useful for matching a Sonilo-generated track to a second edit where speech
placement has changed.

---

## Combining both: generate then duck

For maximum control — generate a music bed from the video, then run ducking as a final pass:

```python
# Step 1: generate music (no preserve_speech — full-volume track first)
video_to_music(video_path="interview.mp4", prompt="warm cinematic documentary")
# → music_bed.m4a

# Step 2: duck the generated track against the same video
audio_ducking(voice_path="interview.mp4", music_path="music_bed.m4a")
# → final .mp4 — the voice input was a video, so the ducked mix comes back muxed in
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
