# Prompt templates

Sonilo works with **no prompt at all** — it reads the cut and composes to match. Use a
prompt when you want to steer the style. These templates are starting points: copy one,
swap the bracketed parts, keep the shape.

**The shape that works:** register + instrumentation / energy arc anchored to what's on
screen / exclusions. Spend words on style; anchor moments to on-screen action ("when the
product appears"), never exact timestamps. Full technique guide:
[music prompting](https://github.com/sonilo-ai/skills/blob/main/music/prompting.md).

## Templates by use case

| Use case | Template |
|---|---|
| Product film | Kinetic score for a [10-second] product film. Gritty live drums and distorted bass, builds tension over the first half and peaks hard [when the product appears]. Modern, confident, cinematic. No vocals, no EDM drop, no orchestral swell. |
| Launch teaser | Launch-trailer score. Sparse pulsing synth opening, tension layering in as the cuts accelerate, a drop to near-silence before [the reveal], then the biggest hit of the track from the reveal onward. No vocals, no orchestral swell. |
| Talking-head / interview bed | Warm, unobtrusive bed for a [founder interview]. Soft keys and muted guitar, slow pulse, stays out of the way of the voice. No drums entering mid-phrase, no melody hooks, no vocals. |
| Vlog outro | Cozy vlog outro bed. Soft felt piano and light acoustic guitar, gentle sustained warmth that eases down toward the end. Intimate, unhurried. No vocals, no lo-fi crackle, no percussion build. |
| Before / after reveal | Two-section piece: restrained minimal bed under the "before" half — muted keys, low pulse — then opening up warm and full when [the "after" appears], resolving confident from there to the end. No vocals, no EDM drop. |
| Action / sports edit | High-energy action score. Driving drum kit and bass, hits landing with the cuts, biggest impact [when the rider lands]. Aggressive but clean. No vocals. |
| Travel montage | Warm cinematic travel score. Acoustic guitar and light strings, easy mid-tempo lift, opens wider at [the landscape reveal], settles gently at the end. No vocals, no EDM drop. |
| Suspense / thriller cut | Low sustained strings and a heartbeat-style pulse, pressure building the whole way, one hard hit then sudden silence at [the scare]. No melody hooks, no vocals. |
| Tutorial / screencast bed | Neutral, lightly optimistic bed. Soft synth pads and a quiet pulse, flat energy that never distracts from the voice. No drums, no melody hooks, no vocals. |
| Not sure yet | Three creative directions for this cut. One: orchestral-cinematic, strings-led, building to a full swell. Two: electronic-kinetic, driving synth bass and tight percussion. Three: minimal-percussive, sparse hits and sub pulses with wide space. No vocals in any direction. (Run with `variants_num=3` — three named directions, not three rerolls.) |

## Rules baked into every template

1. **Mirror the picture.** Consistency with the edit beats eloquent writing — same
   subject, same rhythm, same emphasized moments.
2. **Anchor to actions, not timestamps.** "When the door slams" parses; "at 12.5s" doesn't.
3. **No mix directions.** "Quieter" / "louder drums" aren't promptable — state the role
   instead ("background bed", "drums carry the track").
4. **Exclusions are best-effort.** If an excluded element keeps returning, strengthen the
   positive description instead of stacking negatives. Verify by ear.
5. **Two-section language parses.** Structure-shaped prompts ("restrained until X, then
   open up") are planned as sections when the backend can parse them; otherwise they fall
   back to a whole-track style hint. Either output is usable.

## More

- [Sound Prompt Bank](https://github.com/sonilo-ai/sound-prompt-bank) — the full
  want → prompt lookup, music and SFX, with generated audio samples per entry
- [Music prompting techniques](https://github.com/sonilo-ai/skills/blob/main/music/prompting.md) —
  why these prompts work
- [`RECIPES.md`](./RECIPES.md) — wiring the generated track into your pipeline
