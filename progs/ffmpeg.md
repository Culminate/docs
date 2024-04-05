---
title: FFMPEG
description: 
published: true
date: 2024-04-05T20:31:01.246Z
tags: 
editor: markdown
dateCreated: 2023-08-28T14:22:13.121Z
---

# ffmpeg

# video to gif

```bash
ffmpeg -i input.mp4 \
-vf "fps=10,scale=-1:540:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 \
output.gif
```

* [fps](https://ffmpeg.org/ffmpeg-filters.html#fps) filter sets the frame rate. A rate of 10 frames per second is used in the example.
* [scale](https://ffmpeg.org/ffmpeg-filters.html#scale) filter will resize the output to 320 pixels wide and automatically determine the height while preserving the aspect ratio. The lanczos [scaling algorithm](https://ffmpeg.org/ffmpeg-scaler.html) is used in this example.
* [palettegen](https://ffmpeg.org/ffmpeg-filters.html#palettegen) and [paletteuse](https://ffmpeg.org/ffmpeg-filters.html#paletteuse) filters will generate and use a custom palette generated from your input. These filters have many options, so refer to the links for a list of all available options and values. Also see the **Advanced options** section below.
* [split](https://ffmpeg.org/ffmpeg-filters.html#split_002c-asplit) filter will allow everything to be done in one command and avoids having to create a temporary PNG file of the palette.
* Control looping with `-loop` output option but the values are confusing. A value of `0` is infinite looping, `-1` is no looping, and `1` will loop once meaning it will play twice. So a value of 10 will cause the GIF to play 11 times.

See [High quality GIF with FFmpeg](http://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html) for explanations, example images, and more detailed info for advanced usage.

## links

https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality
https://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html

# high efficient encoding

Выбираем для декодирования эффективный кодек av1. Кодировать будем в него декодером svtav1 от intel.

```shell
ffmpeg -i input_video.mp4 \
-c:v libsvtav1 -svtav1-params fast-decode=1 -crf 45 -preset 6 -map 0 \
-ss 00:05:20 -to 00:10:00  output_video.mp4
```

## links

https://trac.ffmpeg.org/wiki/Encode/AV1

# denoise

```
ffmpeg -i input_video.mp4 \
-filter:a "highpass=f=300,asendcmd=0.0 afftdn sn start,asendcmd=1.5 afftdn sn stop,afftdn=nf=-20,dialoguenhance,lowpass=f=3000, arnndn=m=lq.rnnn" \
-ss 00:00:00 -to 00:00:10 -map 0:a:1 -f matroska - | ffplay -i -
```


- First everything below 300Hz is thrown away;
- the Denoise with FFT filter is used with a noise profile of the first 1.5 seconds of the audio and a noise floor of -20dB;
- then we enhance the dialogue;
- finally remove frequencies above 3000Hz


## links

https://github.com/GregorR/rnnoise-models
https://stackoverflow.com/questions/44159621/how-to-denoise-audio-with-sox
https://odan.github.io/2023/08/20/reducing-background-audio-noise.html