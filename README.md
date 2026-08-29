# GIF SMASHER

A GIF file compressor that leverages ffmpeg and gifsicle to compress a video or GIF file to a GIF of a given size in bytes.

Give it a file and a number of megabytes. It encodes a GIF, measures it, gives up
some framerate, colour, resolution or fidelity, and encodes again — until the file
fits.

## In the browser

There is a port at **<https://e-z-g.github.io/gifsmash.html>** that needs no ffmpeg,
no gifsicle and no shell. It keeps every knob below and adds a preview, a live
iteration log, and the ability to keep whichever pass landed closest instead of
whichever happened to be last. Everything runs in the tab; nothing is uploaded.

The palette generation, dithering, resampling and LZW are reimplemented by hand
there, so its byte counts will not match a run of this script — the knobs mean the
same things, but the tools underneath are different.

## Running the script

Needs `bash`, `bc`, `awk`, `ffmpeg`/`ffprobe`, and a build of gifsicle with the
lossy patch. The paths to all three are hard-coded near the top of
`gif_smasher.sh` and will need changing for your machine.

```sh
./gif_smasher.sh -i clip.mov -t 2
```

The output lands beside the input as `clip.gif`, or `clip(1).gif` if that name is
taken.

## Options

| flag | | default | |
|---|---|---|---|
| `-i` | `--input` | — | source video or GIF (required) |
| `-t` | `--target` | — | target size in MB (required) |
| `-I` | `--maxIter` | 6 | how many encodes to spend |
| `-f` | `--fpsmin` | 0.5 | framerate floor |
| `-l` | `--lossymax` | 200 | ceiling on gifsicle's `--lossy` |
| `-c` | `--colormin` | 4 | fewest colours allowed |
| `-s` | `--scalemin` | 0.1 | smallest resolution scale |
| `-F` | `--fpswt` | 1 | how hard to lean on framerate |
| `-L` | `--lossywt` | 1 | how hard to lean on lossiness |
| `-C` | `--colorwt` | 1 | how hard to lean on colour count |
| `-S` | `--scalewt` | 1 | how hard to lean on resolution |

A weight of 0 pins that knob at its starting value. Weights are divided by 2
regardless of what the others are set to, so a weight of 1 closes half the gap
rather than all of it.

## The two modes

`targetMode` is set in the script and is `approx` as shipped.

**approx** feeds each result back as `target / actual` and moves every knob by its
own function of that one ratio — framerate proportionally, colours by the square
root, resolution by the square, lossiness by an additive step. It converges from
either side and runs the full iteration count.

**threshold** walks each knob steadily from its starting value towards its limit
in equal steps and stops at the first encode that fits. Before each gifsicle pass
it checks whether ffmpeg's own output already fits at the current framerate, and
takes that if it does.

The exponents come from measuring what each knob did to a file, not from theory:

```
FPS       ∝ SIZE
LOSSINESS ∝ (1/3) * SIZE^(-1/4)
COLOR     ∝ SIZE^(1/2)
SCALE     ∝ SIZE^2
```

## Reference

- <http://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html>
- <https://www.lcdf.org/gifsicle/man.html>
