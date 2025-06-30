---
title: FFMPEG
description: 
published: 1
date: 2025-06-30T18:47:33.475Z
tags: 
editor: markdown
dateCreated: 2025-01-09T22:05:30.101Z
---

# ffmpeg

# Basic

```shell
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url}
```

## Time duration

There are two accepted syntaxes for expressing time duration. 

```
[-][HH:]MM:SS[.m...]
```

`HH` expresses the number of hours, `MM` the number of minutes for a maximum of 2 digits, and `SS` the number of seconds for a maximum of 2 digits. The `m` at the end expresses decimal value for `SS`.

or

```
[-]S+[.m...][s|ms|us]
```

`S` expresses the number of seconds, with the optional decimal part `m`. The optional literal suffixes `s`, `ms` or `us` indicate to interpret the value as seconds, milliseconds or microseconds, respectively.

In both expressions, the optional `-` indicates negative duration.

**examples:**
- `55` 55 seconds
- `0.2` 0.2 seconds
- `200ms` 200 milliseconds, that’s 0.2s
- `200000us` 200000 microseconds, that’s 0.2s
- `12:03:45` 12 hours, 03 minutes and 45 seconds
- `23.189` 23.189 seconds 

## Stream Selection

```
-map [-]input_file_id[:stream_specifier][:view_specifier][:?] | [linklabel] (output)
```

- `-map 0` map everything
- `-map 0:1` select specific stream. If you have two audio streams in the first input file, these streams are identified by 0:0 and 0:1
- `-map 0:1 -map 1:2` select multiple stream. select second stream from first input and third stream from second input
- `-map 0:v -map 0:a:2` select all video stream and third audio stream
- `-map 0 -map -0:a:1` select all stream except second audio stream.
- `-map 0 -c:v libx264 -c:a copy` encodes all video streams with libx264 and copies all audio streams. 
- `-map 0 -c copy -c:v:1 libx264 -c:a:137 libvorbis` will copy all the streams except the second video, which will be encoded with libx264, and the 138th audio, which will be encoded with libvorbis.

## Cut video

- `-ss` start [position](#time-duration) (input/output)
  - When used as an input option (before -i), seeks nearest keyframe in input.
  - When used as an output option (before an output url), decodes but discards input until the timestamps reach position.
  
  Note: when used first option copying a stream, it leads to artifacts.
- `-to` end [position](#time-duration) (input/output)
- `-t` [duration](#time-duration) (input/output)

# Cut without reencoding

```shell
ffmpeg -ss 00:00:25 -i input.ext -to 00:01:00 -c:v copy -c:a copy -map 0 output.ext
```

https://superuser.com/questions/377343/cut-part-from-video-file-from-start-position-to-end-position-with-ffmpeg/377407#377407

# Video to gif

```bash
-vf "fps=10,scale=-1:540:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0
```

* [fps](https://ffmpeg.org/ffmpeg-filters.html#fps) filter sets the frame rate. A rate of 10 frames per second is used in the example.
* [scale](https://ffmpeg.org/ffmpeg-filters.html#scale) filter will resize the output to 320 pixels wide and automatically determine the height while preserving the aspect ratio. The lanczos [scaling algorithm](https://ffmpeg.org/ffmpeg-scaler.html) is used in this example.
* [palettegen](https://ffmpeg.org/ffmpeg-filters.html#palettegen) and [paletteuse](https://ffmpeg.org/ffmpeg-filters.html#paletteuse) filters will generate and use a custom palette generated from your input. These filters have many options, so refer to the links for a list of all available options and values. Also see the **Advanced options** section below.
* [split](https://ffmpeg.org/ffmpeg-filters.html#split_002c-asplit) filter will allow everything to be done in one command and avoids having to create a temporary PNG file of the palette.
* Control looping with `-loop` output option but the values are confusing. A value of `0` is infinite looping, `-1` is no looping, and `1` will loop once meaning it will play twice. So a value of 10 will cause the GIF to play 11 times.

See [High quality GIF with FFmpeg](http://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html) for explanations, example images, and more detailed info for advanced usage.

https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality
https://blog.pkh.me/p/21-high-quality-gif-with-ffmpeg.html

# HEVC(H.265) Encoding

```shell
-c:v libx265 -crf 35 -preset medium
```

https://trac.ffmpeg.org/wiki/Encode/H.265

# AV1 Encoding

Select for encoding High efficient codec av1. Use libstav1 by intel.

```shell
-c:v libsvtav1 -svtav1-params fast-decode=1 -crf 45 -preset 6
```

https://trac.ffmpeg.org/wiki/Encode/AV1

# Audio compose

Увеличивает громкость на первой аудиодорожке. Сливает в одну 0 и 1 аудиодорожку.

```shell
-filter_complex "[0:a:1]volume=1.2[vol];[0:a:0][vol]amix=inputs=2[out]" -map 0:v:0 -map [out]
```

# Denoise audio

```shell
ffmpeg -i input_video.mp4 \
-filter:a "highpass=f=300,asendcmd=0.0 afftdn sn start,asendcmd=1.5 afftdn sn stop,afftdn=nf=-20,dialoguenhance,lowpass=f=3000, arnndn=m=lq.rnnn" \
-ss 00:00:00 -to 00:00:10 -map 0:a:1 -f matroska - | ffplay -i -
```

```shell
ffmpeg -i in.mp4 -c:v libsvtav1 -svtav1-params fast-decode=1 -crf 42 -preset 6 -filter_complex \
"[0:a:1]highpass=f=200,lowpass=f=3000,afftdn=nf=-20,arnndn=m=../lq.rnnn[denoise];[0:a:0][denoise]amix[out]" \
-map 0 -map "[out]" out.mp4
```


- First everything below 300Hz is thrown away;
- the Denoise with FFT filter is used with a noise profile of the first 1.5 seconds of the audio and a noise floor of -20dB;
- then we enhance the dialogue;
- finally remove frequencies above 3000Hz

https://github.com/GregorR/rnnoise-models
https://stackoverflow.com/questions/44159621/how-to-denoise-audio-with-sox
https://forum.videohelp.com/threads/413220-Remove-noise-having-a-noise-file

# links

https://www.ffmpeg.org/ffmpeg.html
[habr.com Кодирование с кодеком HEVC простым языком](https://habr.com/ru/companies/ruvds/articles/845202/)