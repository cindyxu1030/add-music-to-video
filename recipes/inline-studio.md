# Inline Studio: the Sonilo nodes on the canvas

[Inline Studio](https://github.com/inlineresearch/Inline-Studio) is a free, open-source app for
AI filmmaking on a node canvas, and its hosted-model registry now carries three Sonilo nodes.
This recipe builds the smallest useful graph with them: a finished cut goes in, a soundtrack
matched to that cut comes back as an audio take, and the take wires straight into a Video
Director cut — all without leaving the canvas. Tracks are **licensed and safe for commercial
use (terms apply)**.

Disclosure up front, same as in the upstream PRs: Sonilo maintains this cookbook and
contributed the three nodes.

## Prerequisites

- **Inline Studio installed and running.** Follow the
  [getting-started guide](https://inlinestudio.art/getting-started); once the app is up you'll
  have the node canvas open in your browser.
- **A [fal.ai API key](https://fal.ai/dashboard/keys).** The Sonilo nodes are hosted fal
  endpoints (`sonilo/v1.1/*`), so they run through the same fal key as every other hosted model
  in the app — no separate account or key is needed for the canvas nodes. (The
  [MCP and REST surfaces](../README.md#two-ways-to-use-sonilo) covered in the other recipes use
  a Sonilo key instead; that's a different path.)
- **A video file** for the two video-driven nodes. Any finished cut works — a stitched set of
  generated clips, an export from your editor, stock footage.

## The three nodes at a glance

| Node (as named in the app) | In → out | Prompt | Reach for it when |
|---|---|---|---|
| **Sonilo · Video → Music** | video frame → audio take | optional style hint | You want a track matched to your cut, and you want to place and mix it yourself — in the Video Director, or exported into your own pipeline. |
| **Sonilo · Text → Music** | prompt → audio take | **required** — it carries the whole instruction | There's no video yet, or you just need a bed from a description (mood, instrumentation, tempo). |
| **Sonilo · Video → Video** | video frame → video take | optional style hint | You want the finished clip back with the track already mixed in, one step. A **Keep original speech** toggle preserves the source vocals over the new music. |

The two video nodes are prompt-optional on purpose: the video is the context. Sonilo reads the
cut's pacing and mood and writes its own musical direction; a wired prompt only steers the
style. Text → Music has no video to read, so there the prompt is the instruction and stays
required.

(Sound effects — a royalty-free SFX track timed to the picture — are not a canvas node yet;
they run through the [MCP or REST surface](../RECIPES.md#add-sound-effects-with-sonilo) for now.)

## Step 1 — Add your fal key

Open **Settings** and paste your key into the **fal.ai API key** field. The key stays on the
machine that runs the engine; it's never echoed back to the browser.

[FILL: screenshot — Settings panel with the fal.ai API key field]

## Step 2 — Add the node

Open the Add-node menu (the **+** button in the toolbar, or double-click empty canvas) and pick
**Sonilo · Video → Music** from the **Sonilo** group of Generate models. You can also add any
Generate node and switch it from the model picker in its footer.

[FILL: screenshot — Add-node menu open, Sonilo group visible]

## Step 3 — Wire the minimal graph

1. Get your cut onto the canvas: drag the video file in, or add a **Load Assets** node. You now
   have a video frame.
2. Wire the video frame into the node's **Video** input dot.
3. That's the whole required graph. Optionally, add a **Prompt** node and wire it into the amber
   **Prompt (optional)** dot with a style hint like `warm, cinematic, building to a swell` — or
   leave it unwired and let Sonilo read the cut.

[FILL: screenshot — video frame → Sonilo · Video → Music, with an optional Prompt node]

The node's settings panel holds the remaining parameters: **Samples**, **Start offset (s)**, and
**Duration (s, 0 = full video)** — the latter two select a segment of the cut to score, and the
default scores the whole thing.

## Step 4 — Run

Select the graph and press **Run** (the control floats above the graph's output node). The node
reads the clip straight from the connected frame and sends it with the request — you don't have
to host the file anywhere. Progress runs Queued → Generating → Done, and the result lands on the
node as an audio take with a waveform preview. Every run adds a new take; nothing is overwritten.

[FILL: screenshot — the node mid-generation, then the finished waveform take]

## Step 5 — Wire the track into a cut

Add a **Video Director** node. Wire your video frames into its **V** inputs and the Sonilo audio
frame into an **A** input: the videos' own sound sits on the AUDIO L1 layer, and your generated
track lands on AUDIO L2, with per-input and per-layer volume. Scrub the in-node preview, then
export the cut.

[FILL: screenshot — Video Director with the Sonilo audio frame wired into A1]

### The other two nodes

**Sonilo · Text → Music** needs no media input at all: wire a **Prompt** node describing the
track (the node won't run without one — it shows *Connect a Prompt node* until you do), set
**Duration (s)** (default 90, up to 600), and Run. **Sonilo · Video → Video** wires exactly like
Video → Music but hands back a video take with the mix done; flip **Keep original speech** on to
carry the source vocals over the generated track.

## Tips

- **Duration caps.** Text → Music clamps its duration to 600 seconds (default 90). The two
  video nodes score the full clip by default; **Start offset** + **Duration** select a segment
  instead. The Sonilo music endpoint itself takes cuts up to six minutes — the node doesn't
  enforce that, so longer inputs fail at the endpoint rather than on the canvas.
- **Output formats.** Music takes come back as AAC in an `.m4a` container; Video → Video returns
  an `.mp4`. Takes are saved into the project's `takes/` folder, so you can pull the file into
  any external pipeline (the [FFmpeg recipes](../RECIPES.md) all apply).
- **Price estimates.** Generation is billed by fal at roughly $0.009 per second of output per
  sample, and each node shows a rough estimate before you run. On the two video nodes the
  estimate only appears once you set a **Duration** — the full video's length isn't known at
  estimate time.
- **Leave Samples at 1 for now.** The app currently saves one take per run for audio and video
  outputs even when more samples are requested, so a higher count can bill without surfacing
  extra takes. A fix is tracked upstream; until it lands, run again for another take instead —
  each run lands as a new take anyway.

## More control: structured briefs

The nodes are deliberately video-first: no prompt required, the cut is the context. When you do
want control, the difference between "good" and "exactly what I meant" is a short structured
brief — genre, energy arc, the moment that must hit, the sounds that must not appear.

[**sonilo-ai/skills (music/prompting.md + sound-effects/prompting.md)**](https://github.com/sonilo-ai/skills) is a set of skills
that teach an agent (or you) to write those briefs: `/sonilo:sonilo-prompting` for the
pre-flight checks, `/sonilo:video-to-music` for style-prompt craft, `/sonilo:video-to-sfx` for
sound-effects action maps. The briefs are plain text, so the technique transfers directly to the
canvas: write the brief, paste it into a **Prompt** node, wire it in. The skills also cover the
MCP and REST surfaces for everything beyond the canvas nodes.

## Troubleshooting

- ***"Add a fal API key in Settings to generate."*** — no key is saved. Step 1: Settings →
  **fal.ai API key**. If generation fails with *"fal rejected the API key"* instead, the saved
  key is invalid or revoked — paste a fresh one from the fal dashboard.
- **The node shows *Connect a Prompt node* or *Wire video* and won't run.** Text → Music
  requires a prompt (it's the whole instruction); the two video nodes require a video frame on
  the **Video** input. Only Video → Music and Video → Video run prompt-free.
- ***"The model returned no output."*** or a rate-limit message — transient endpoint states.
  Wait a moment and run again; the request log in your fal dashboard shows the raw response if
  it persists.

## Where to go next

- [Render-to-music workflow](./render-to-music-workflow.md) — the provider-neutral shape of this
  same flow, for pipelines outside the canvas
- [RECIPES.md](../RECIPES.md) — FFmpeg commands for muxing, fades, ducking, and sound effects on
  exported takes
- [DEMOS.md](../DEMOS.md) — hear the before/after pairs
