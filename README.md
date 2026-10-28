# Twitter Video to GIF

Convert a Twitter / X video into a GIF **right in your browser** — no signup, no upload, no install.

**→ Try it: [twittergif.com](https://twittergif.com)**

## What it does

1. Paste a post link from x.com or twitter.com (mobile links, `?s=` share links and `/video/1` links all work).
2. Trim the clip, pick the frame rate (10–30 fps) and width (480–1080px). A live estimate shows the GIF size before you convert.
3. Click **Convert to GIF** and download the file.

It also turns X's "GIFs" — which X actually stores as MP4 videos — back into real `.gif` files.

## How it works (and why nothing is uploaded)

```
paste link ──► tiny Cloudflare Worker ──► public video.twimg.com MP4 links
                                              │
your browser ◄── downloads the MP4 directly ──┘   (X's CDN sends CORS headers)
     │
     └─► ffmpeg.wasm converts it on your device ──► GIF you download
```

- **Link lookup:** a small Cloudflare Worker turns the post ID into the public MP4 links. It never caches or stores anything.
- **Video download:** your browser fetches the MP4 straight from X's CDN, so the video never passes through our server.
- **Conversion:** [ffmpeg.wasm](https://github.com/ffmpegwasm/ffmpeg.wasm) (FFmpeg compiled to WebAssembly) does the work on your own device. The ~9 MB engine is only downloaded when you click convert, then cached by your browser.

## The FFmpeg recipe

This is the filter graph used for a 480px / 15 fps GIF. It's handy with desktop `ffmpeg` too:

```bash
ffmpeg -ss 0 -t 5 -i input.mp4 \
  -filter_complex "[0:v]fps=15,scale=480:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle" \
  -loop 0 output.gif
```

- `fps` before `scale`, so frames that get dropped are never resized.
- `palettegen` builds the best 256-colour palette for this clip, and `paletteuse` maps every frame onto it.
- Ordered (`bayer`) dithering plus `diff_mode=rectangle` made our GIFs roughly 20–65% smaller than the default dithering on real tweet videos, with very little visible difference.
- For long or large clips, run the palette as a separate first pass (`palettegen` → `palette.png`, then `paletteuse`) so the frames don't all sit in memory at once.

Typical size: a 5-second clip at 480px / 15 fps comes out around 2–3 MB.

## Free and paid

The free plan converts up to 5 clips a day (10 seconds max, 480px, 15 fps) with a small watermark in the corner. A one-time GIF Pack or Pro plan removes the watermark and unlocks longer, sharper GIFs — see [pricing](https://twittergif.com/#pricing).

## Feedback

A post that won't convert, a link format that isn't recognised, a GIF that looks wrong: please [open an issue](../../issues) or email hello@twittergif.com.

---

Not affiliated with X Corp. "Twitter" and "X" are trademarks of X Corp.
