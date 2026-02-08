---
description: "Media processing recipes reference (internal)"
when_to_use: "Receives prepared arguments from task-media with file paths and context included"
when-to-use: "Receives prepared arguments from task-media with file paths and context included"
context: fork
model: inherit
allowed-tools: Read
---

# Media Processing Recipes

**Tools:** ffmpeg, ffprobe, yt-dlp, imagemagick (convert, identify, mogrify, composite), exiftool, gifsicle, optipng, jpegoptim, pngquant, cwebp, sox, whisper, pandoc, weasyprint, unoconv, graphviz (dot), mermaid-cli, qpdf, img2pdf

---

**Task**: $ARGUMENTS

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Convert video format | ffmpeg | `ffmpeg -i input.mp4 output.webm` |
| Extract audio | ffmpeg | `ffmpeg -i video.mp4 -vn audio.mp3` |
| Resize video | ffmpeg | `ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4` |
| Trim video | ffmpeg | `ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 output.mp4` |
| Resize image | convert | `convert input.jpg -resize 800x600 output.jpg` |
| Convert image format | convert | `convert input.png output.jpg` |
| Create thumbnail | convert | `convert input.jpg -thumbnail 150x150^ -gravity center -extent 150x150 thumb.jpg` |
| Get media info | ffprobe | `ffprobe -v quiet -print_format json -show_format input.mp4` |
| Read EXIF | exiftool | `exiftool image.jpg` |
| Strip EXIF | exiftool | `exiftool -all= image.jpg` |
| Convert to PDF | pandoc | `pandoc input.md -o output.pdf` |
| Optimize GIF | gifsicle | `gifsicle -O3 --lossy=80 input.gif -o output.gif` |
| HTML to PDF | weasyprint | `weasyprint input.html output.pdf` |
| Office convert | unoconv | `unoconv -f pdf document.docx` |

---

## When to Use What

| Task | Best Tool | Why |
|------|-----------|-----|
| Video conversion/editing | ffmpeg | Universal, most powerful |
| Audio effects/conversion | sox | Audio-specific operations |
| Image resize/convert | imagemagick (convert) | Batch with mogrify |
| GIF optimization | gifsicle | Reduce size, edit frames |
| PNG optimization | optipng | Lossless compression |
| JPEG optimization | jpegoptim | Lossless/lossy JPEG compression |
| Document conversion | pandoc | Markdown/HTML/PDF/DOCX |
| Video download | yt-dlp | Any video site |
| Metadata read/write | exiftool | Most comprehensive |
| Audio transcription | whisper | OpenAI speech-to-text |

---

## Video Operations (ffmpeg)

### Format Conversion
```bash
# Basic conversion (auto-detect codecs)
ffmpeg -i input.mp4 output.webm
ffmpeg -i input.mov output.mp4
ffmpeg -i input.avi output.mkv

# Specify codecs
ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4
ffmpeg -i input.mp4 -c:v libvpx-vp9 -c:a libopus output.webm

# Copy streams (no re-encoding, fast)
ffmpeg -i input.mp4 -c copy output.mkv

# Web-optimized MP4 (fast start)
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -movflags +faststart output.mp4
```

### Quality Control
```bash
# CRF (Constant Rate Factor): lower = better quality, bigger file
# H.264: 18-28 typical, 23 default
ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4

# VP9: 15-35 typical, 31 default
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 31 -b:v 0 output.webm

# Two-pass encoding (better quality for target bitrate)
ffmpeg -i input.mp4 -c:v libx264 -b:v 2M -pass 1 -f null /dev/null
ffmpeg -i input.mp4 -c:v libx264 -b:v 2M -pass 2 output.mp4
```

### Resize/Scale
```bash
# Specific resolution
ffmpeg -i input.mp4 -vf scale=1280:720 output.mp4

# Maintain aspect ratio (width auto)
ffmpeg -i input.mp4 -vf scale=-1:720 output.mp4

# Maintain aspect ratio (divisible by 2)
ffmpeg -i input.mp4 -vf "scale=-2:720" output.mp4

# Scale down only (don't upscale)
ffmpeg -i input.mp4 -vf "scale='min(1280,iw)':'min(720,ih)':force_original_aspect_ratio=decrease" output.mp4
```

### Trim/Cut
```bash
# By start time and duration
ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 -c copy output.mp4

# By start and end time
ffmpeg -i input.mp4 -ss 00:01:00 -to 00:01:30 -c copy output.mp4

# Accurate seek (slower, re-encodes)
ffmpeg -i input.mp4 -ss 00:01:00 -t 00:00:30 output.mp4
```

### Extract Audio
```bash
# To MP3
ffmpeg -i video.mp4 -vn -c:a mp3 -ab 192k audio.mp3

# To AAC
ffmpeg -i video.mp4 -vn -c:a aac audio.m4a

# To WAV (uncompressed)
ffmpeg -i video.mp4 -vn audio.wav

# Copy audio codec
ffmpeg -i video.mp4 -vn -c:a copy audio.aac
```

### Extract Frames
```bash
# Single frame at timestamp
ffmpeg -i input.mp4 -ss 00:00:05 -frames:v 1 frame.jpg

# Frame every second
ffmpeg -i input.mp4 -vf fps=1 frames/frame_%04d.jpg

# Frame every 10 seconds
ffmpeg -i input.mp4 -vf fps=1/10 frames/frame_%04d.jpg

# All frames (warning: many files)
ffmpeg -i input.mp4 frames/frame_%06d.jpg

# High quality PNG
ffmpeg -i input.mp4 -ss 00:00:05 -frames:v 1 frame.png
```

### Create GIF
```bash
# Simple GIF
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1" output.gif

# High quality GIF (two-pass with palette)
ffmpeg -i input.mp4 -vf "fps=10,scale=320:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -lavfi "fps=10,scale=320:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif

# GIF from portion
ffmpeg -ss 00:00:05 -t 3 -i input.mp4 -vf "fps=10,scale=320:-1" output.gif
```

### Concatenate Videos
```bash
# Create file list (files.txt):
# file 'video1.mp4'
# file 'video2.mp4'

ffmpeg -f concat -safe 0 -i files.txt -c copy output.mp4
```

### Add Audio to Video
```bash
# Replace audio
ffmpeg -i video.mp4 -i audio.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 output.mp4

# Mix audio
ffmpeg -i video.mp4 -i audio.mp3 -filter_complex amix=inputs=2:duration=first output.mp4
```

### Video Speed
```bash
# 2x speed
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" -filter:a "atempo=2.0" output.mp4

# 0.5x speed (slow motion)
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" -filter:a "atempo=0.5" output.mp4
```

### Rotate/Flip
```bash
# Rotate 90 clockwise
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4

# Rotate 90 counter-clockwise
ffmpeg -i input.mp4 -vf "transpose=2" output.mp4

# Rotate 180
ffmpeg -i input.mp4 -vf "transpose=2,transpose=2" output.mp4

# Horizontal flip
ffmpeg -i input.mp4 -vf "hflip" output.mp4

# Vertical flip
ffmpeg -i input.mp4 -vf "vflip" output.mp4
```

### Get Video Info
```bash
# Basic info
ffprobe input.mp4

# JSON format
ffprobe -v quiet -print_format json -show_format -show_streams input.mp4

# Just duration
ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 input.mp4

# Just resolution
ffprobe -v error -select_streams v:0 -show_entries stream=width,height -of csv=p=0 input.mp4
```

### Crop
```bash
# Crop to width:height starting at x:y
ffmpeg -i input.mp4 -vf "crop=640:480:100:50" output.mp4

# Crop center (output 640x480 from center)
ffmpeg -i input.mp4 -vf "crop=640:480" output.mp4

# Auto-detect and remove letterbox/pillarbox bars
ffmpeg -i input.mp4 -vf "cropdetect=24:16:0" -f null - 2>&1 | tail -5
# Then apply detected values:
ffmpeg -i input.mp4 -vf "crop=1920:800:0:140" output.mp4
```

### Overlay / Watermark / Picture-in-Picture
```bash
# Image watermark bottom-right with padding
ffmpeg -i input.mp4 -i logo.png \
  -filter_complex "overlay=W-w-10:H-h-10" output.mp4

# Watermark with transparency (50%)
ffmpeg -i input.mp4 -i logo.png \
  -filter_complex "[1:v]format=rgba,colorchannelmixer=aa=0.5[logo];[0:v][logo]overlay=W-w-10:H-h-10" output.mp4

# Picture-in-picture (small video in corner)
ffmpeg -i main.mp4 -i pip.mp4 \
  -filter_complex "[1:v]scale=320:240[pip];[0:v][pip]overlay=W-w-10:10" output.mp4

# Watermark with fade in/out
ffmpeg -i input.mp4 -i logo.png \
  -filter_complex "[1:v]format=rgba,fade=in:st=0:d=1:alpha=1,fade=out:st=8:d=1:alpha=1[logo];[0:v][logo]overlay=W-w-10:H-h-10:enable='between(t,0,9)'" output.mp4
```

### Text Overlay (drawtext)
```bash
# Static text
ffmpeg -i input.mp4 -vf "drawtext=text='Hello World':fontsize=48:fontcolor=white:x=10:y=10" output.mp4

# Text with background box
ffmpeg -i input.mp4 -vf "drawtext=text='Title':fontsize=36:fontcolor=white:box=1:boxcolor=black@0.5:boxborderw=5:x=(w-text_w)/2:y=50" output.mp4

# Timestamp overlay
ffmpeg -i input.mp4 -vf "drawtext=text='%{pts\:hms}':fontsize=24:fontcolor=white:x=10:y=h-40" output.mp4

# Timecode burn-in
ffmpeg -i input.mp4 -vf "drawtext=timecode='00\:00\:00\:00':rate=30:fontsize=32:fontcolor=white:x=10:y=10" output.mp4
```

### Color Correction
```bash
# Brightness and contrast (eq filter)
ffmpeg -i input.mp4 -vf "eq=brightness=0.06:contrast=1.5" output.mp4

# Saturation adjustment
ffmpeg -i input.mp4 -vf "eq=saturation=1.5" output.mp4

# Curves preset (increase contrast)
ffmpeg -i input.mp4 -vf "curves=preset=increase_contrast" output.mp4

# Color balance (boost reds in shadows)
ffmpeg -i input.mp4 -vf "colorbalance=rs=0.3" output.mp4

# Convert to grayscale
ffmpeg -i input.mp4 -vf "format=gray" output.mp4
```

### Stack / Split Videos
```bash
# Side by side (horizontal stack)
ffmpeg -i left.mp4 -i right.mp4 -filter_complex hstack output.mp4

# Vertical stack
ffmpeg -i top.mp4 -i bottom.mp4 -filter_complex vstack output.mp4

# 2x2 grid
ffmpeg -i tl.mp4 -i tr.mp4 -i bl.mp4 -i br.mp4 \
  -filter_complex "[0:v][1:v]hstack[top];[2:v][3:v]hstack[bottom];[top][bottom]vstack" output.mp4

# xstack 2x2 with explicit layout
ffmpeg -i a.mp4 -i b.mp4 -i c.mp4 -i d.mp4 \
  -filter_complex "xstack=inputs=4:layout=0_0|w0_0|0_h0|w0_h0" output.mp4
```

### Crossfade Transitions
```bash
# Crossfade between two videos (1s transition)
ffmpeg -i first.mp4 -i second.mp4 \
  -filter_complex "xfade=transition=fade:duration=1:offset=4" output.mp4

# Available transitions: fade, wipeleft, wiperight, wipeup, wipedown,
# slideleft, slideright, slideup, slidedown, circlecrop, rectcrop,
# distance, fadeblack, fadewhite, radial, smoothleft, smoothright,
# smoothup, smoothdown, circleopen, circleclose, dissolve, pixelize, diagtl
```

### Frame Rate Conversion
```bash
# Change frame rate (drop/duplicate frames)
ffmpeg -i input.mp4 -vf "fps=30" output.mp4

# Motion-interpolated frame rate conversion (slow, high quality)
ffmpeg -i input.mp4 -vf "minterpolate=fps=60:mi_mode=mci:mc_mode=aobmc:me_mode=bidir:vsbmc=1" output.mp4

# Decimate (remove duplicate frames, e.g., 30fps telecine to 24fps)
ffmpeg -i input.mp4 -vf "decimate" output.mp4
```

### Deinterlace
```bash
# yadif (Yet Another DeInterlacing Filter) - good default
ffmpeg -i input.mp4 -vf "yadif" output.mp4

# bwdif (Bob Weaver Deinterlacing Filter) - better quality
ffmpeg -i input.mp4 -vf "bwdif" output.mp4
```

### Video Stabilization (two-pass)
```bash
# Pass 1: Detect motion
ffmpeg -i shaky.mp4 -vf "vidstabdetect=shakiness=5:accuracy=15:result=transforms.trf" -f null -

# Pass 2: Apply stabilization
ffmpeg -i shaky.mp4 -vf "vidstabtransform=input=transforms.trf:zoom=1:smoothing=30,unsharp=5:5:0.8:3:3:0.4" stabilized.mp4
```

### Subtitles
```bash
# Burn subtitles into video (hardcode) - SRT
ffmpeg -i input.mp4 -vf "subtitles=subs.srt" output.mp4

# Burn subtitles - ASS/SSA (preserves styling)
ffmpeg -i input.mp4 -vf "ass=subs.ass" output.mp4

# Add soft subtitles (can be toggled by player)
ffmpeg -i input.mp4 -i subs.srt -c copy -c:s mov_text output.mp4
ffmpeg -i input.mp4 -i subs.srt -c copy -c:s srt output.mkv

# Extract subtitles from video
ffmpeg -i input.mkv -map 0:s:0 subs.srt

# List subtitle streams
ffprobe -v error -select_streams s -show_entries stream=index,codec_name,codec_type -of csv=p=0 input.mkv
```

### HLS / DASH Streaming
```bash
# Basic HLS output
ffmpeg -i input.mp4 -c:v libx264 -c:a aac -hls_time 10 -hls_list_size 0 \
  -hls_segment_filename "segment_%03d.ts" playlist.m3u8

# Multi-bitrate HLS (adaptive)
ffmpeg -i input.mp4 \
  -map 0:v -map 0:v -map 0:a \
  -c:v:0 libx264 -b:v:0 800k -s:v:0 640x360 \
  -c:v:1 libx264 -b:v:1 2400k -s:v:1 1280x720 \
  -c:a aac -b:a 128k \
  -var_stream_map "v:0,a:0 v:1,a:0" \
  -hls_time 6 -hls_list_size 0 \
  -master_pl_name master.m3u8 \
  stream_%v/playlist.m3u8

# DASH output
ffmpeg -i input.mp4 -c:v libx264 -c:a aac \
  -f dash -seg_duration 4 output.mpd
```

### Hardware Acceleration
```bash
# List available encoders
ffmpeg -encoders 2>/dev/null | grep -E "nvenc|vaapi|qsv|videotoolbox"

# NVIDIA NVENC (H.264)
ffmpeg -hwaccel cuda -i input.mp4 -c:v h264_nvenc -preset p7 -cq 23 output.mp4

# NVIDIA NVENC (H.265/HEVC)
ffmpeg -hwaccel cuda -i input.mp4 -c:v hevc_nvenc -preset p7 -cq 28 output.mp4

# Intel Quick Sync (QSV)
ffmpeg -hwaccel qsv -i input.mp4 -c:v h264_qsv -global_quality 23 output.mp4

# VAAPI (Linux, AMD/Intel)
ffmpeg -hwaccel vaapi -hwaccel_output_format vaapi -i input.mp4 \
  -c:v h264_vaapi -qp 23 output.mp4
```

### Video Metadata
```bash
# Copy all metadata
ffmpeg -i input.mp4 -c copy -map_metadata 0 output.mp4

# Strip all metadata
ffmpeg -i input.mp4 -c copy -map_metadata -1 output.mp4

# Set title and artist
ffmpeg -i input.mp4 -c copy -metadata title="My Video" -metadata artist="Author" output.mp4

# Read metadata with ffprobe
ffprobe -v quiet -print_format json -show_format input.mp4 | jq '.format.tags'
```

### Screen Recording
```bash
# Record X11 screen (full screen)
ffmpeg -f x11grab -framerate 30 -i :0.0 -c:v libx264 -preset ultrafast screen.mp4

# Record region (offset+size)
ffmpeg -f x11grab -framerate 30 -video_size 1280x720 -i :0.0+100,200 region.mp4

# Screen recording with audio (PulseAudio)
ffmpeg -f x11grab -framerate 30 -i :0.0 \
  -f pulse -i default \
  -c:v libx264 -preset ultrafast -c:a aac screen_audio.mp4
```

### Stream Mapping
```bash
# Select specific video and audio streams
ffmpeg -i input.mkv -map 0:v:0 -map 0:a:1 -c copy output.mkv

# Exclude subtitles
ffmpeg -i input.mkv -map 0 -map -0:s -c copy output.mkv

# Copy all streams from first input, video only from second
ffmpeg -i main.mkv -i overlay.mp4 -map 0 -map 1:v -c copy output.mkv

# Map multiple audio tracks
ffmpeg -i input.mkv -map 0:v:0 -map 0:a:0 -map 0:a:1 -c copy output.mkv
```

### Gotchas & Common Pitfalls
```bash
# -ss BEFORE -i = fast seek (may be inaccurate with -c copy)
ffmpeg -ss 00:01:00 -i input.mp4 -t 30 -c copy output.mp4
# -ss AFTER -i = accurate seek (slower, decodes from start)
ffmpeg -i input.mp4 -ss 00:01:00 -t 30 output.mp4
# Best of both: fast seek + accurate output
ffmpeg -ss 00:00:55 -i input.mp4 -ss 5 -t 30 -c copy output.mp4

# -c copy cannot apply filters (scale, crop, etc.)
# This FAILS: ffmpeg -i input.mp4 -vf scale=720:-1 -c copy output.mp4
# Fix: remove -c copy to re-encode, or use -c:a copy to copy only audio

# Pixel format compatibility (common error: "incompatible pixel format")
ffmpeg -i input.mp4 -c:v libx264 -pix_fmt yuv420p output.mp4

# Always use -y to overwrite without prompting (for scripts)
ffmpeg -y -i input.mp4 output.mp4

# Codec compatibility matrix (container → codecs)
# MP4:  H.264/H.265 + AAC/MP3        (most compatible)
# WebM: VP8/VP9/AV1 + Vorbis/Opus    (web-optimized)
# MKV:  nearly any codec              (most flexible)
# MOV:  H.264/ProRes + AAC            (Apple ecosystem)

# Common -preset values (libx264): ultrafast, superfast, veryfast,
# faster, fast, medium (default), slow, slower, veryslow
# Slower = better compression (smaller file), same quality
```

---

## Image Operations (ImageMagick)

### Resize
```bash
# Specific size
convert input.jpg -resize 800x600 output.jpg

# Fit within bounds (maintain aspect)
convert input.jpg -resize 800x600 output.jpg

# Force exact size (may distort)
convert input.jpg -resize 800x600! output.jpg

# Resize by percentage
convert input.jpg -resize 50% output.jpg

# Only if larger
convert input.jpg -resize '800x600>' output.jpg

# Only if smaller
convert input.jpg -resize '800x600<' output.jpg
```

### Thumbnails
```bash
# Simple thumbnail
convert input.jpg -thumbnail 150x150 thumb.jpg

# Crop to exact square
convert input.jpg -thumbnail 150x150^ -gravity center -extent 150x150 thumb.jpg

# With padding instead of crop
convert input.jpg -thumbnail 150x150 -gravity center -extent 150x150 -background white thumb.jpg
```

### Format Conversion
```bash
# PNG to JPG
convert input.png output.jpg

# JPG to PNG
convert input.jpg output.png

# To WebP
convert input.jpg output.webp

# Batch conversion
mogrify -format jpg *.png
```

### Quality/Compression
```bash
# JPG quality (1-100)
convert input.jpg -quality 85 output.jpg

# PNG compression (0-9)
convert input.png -quality 95 output.png

# Strip metadata for smaller files
convert input.jpg -strip -quality 85 output.jpg
```

### Crop
```bash
# Crop to size from top-left
convert input.jpg -crop 800x600+0+0 output.jpg

# Crop from center
convert input.jpg -gravity center -crop 800x600+0+0 output.jpg

# Auto-crop (remove borders)
convert input.jpg -trim output.jpg
```

### Rotate
```bash
# Rotate by degrees
convert input.jpg -rotate 90 output.jpg
convert input.jpg -rotate -90 output.jpg

# Auto-rotate based on EXIF
convert input.jpg -auto-orient output.jpg
```

### Get Image Info
```bash
# Basic info
identify image.jpg

# Verbose info
identify -verbose image.jpg

# Just dimensions
identify -format "%wx%h" image.jpg

# Just format
identify -format "%m" image.jpg
```

### Batch Operations (mogrify)
```bash
# Resize all JPGs in place
mogrify -resize 800x600 *.jpg

# Convert and rename
mogrify -format png -path output/ *.jpg

# Add suffix
for f in *.jpg; do convert "$f" -resize 50% "${f%.jpg}_small.jpg"; done
```

### Composite/Overlay
```bash
# Watermark
composite -gravity southeast -geometry +10+10 watermark.png input.jpg output.jpg

# Combine images side-by-side
convert +append image1.jpg image2.jpg combined.jpg

# Combine images vertically
convert -append image1.jpg image2.jpg combined.jpg
```

---

## Image Optimization

### JPEG
```bash
# jpegoptim (lossy)
jpegoptim --max=85 image.jpg
jpegoptim --max=85 --strip-all image.jpg

# jpegoptim (lossless)
jpegoptim image.jpg
```

### PNG
```bash
# optipng (lossless)
optipng image.png
optipng -o7 image.png              # Maximum compression

# pngquant (lossy, smaller files)
pngquant --quality=65-80 image.png
pngquant --force --output optimized.png image.png
```

### WebP
```bash
# Convert to WebP
cwebp -q 80 input.jpg -o output.webp
cwebp -q 80 input.png -o output.webp

# Lossless WebP
cwebp -lossless input.png -o output.webp
```

---

## Metadata (exiftool)

### Read Metadata
```bash
# All metadata
exiftool image.jpg

# Common fields
exiftool -common image.jpg

# Specific fields
exiftool -DateTimeOriginal -GPSLatitude -GPSLongitude image.jpg

# JSON output
exiftool -json image.jpg
exiftool -json *.jpg > metadata.json

# Compact format
exiftool -s image.jpg

# GPS coordinates
exiftool -gps:all image.jpg
```

### Modify Metadata
```bash
# Set description
exiftool -Description="My Photo" image.jpg

# Set date
exiftool -DateTimeOriginal="2024:01:15 10:30:00" image.jpg

# Copy metadata from one file to another
exiftool -TagsFromFile source.jpg target.jpg
```

### Remove Metadata
```bash
# Remove all metadata
exiftool -all= image.jpg

# Remove GPS only
exiftool -gps:all= image.jpg

# Remove all but preserve orientation
exiftool -all= -TagsFromFile @ -Orientation image.jpg

# Batch remove
exiftool -all= *.jpg
```

---

## Audio Operations

### FFmpeg Audio
```bash
# Convert format
ffmpeg -i input.wav output.mp3
ffmpeg -i input.flac -c:a mp3 -ab 320k output.mp3

# Change bitrate
ffmpeg -i input.mp3 -ab 192k output.mp3

# Trim audio
ffmpeg -i input.mp3 -ss 00:00:30 -t 00:01:00 output.mp3

# Merge audio files
# Create list file: echo -e "file 'input1.mp3'\nfile 'input2.mp3'" > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp3

# Change volume
ffmpeg -i input.mp3 -af "volume=2.0" output.mp3

# Normalize audio
ffmpeg -i input.mp3 -af "loudnorm=I=-16:TP=-1.5:LRA=11" output.mp3
```

### Sox (Sound eXchange)
```bash
# Convert format
sox input.wav output.mp3

# Get info
soxi input.mp3

# Trim
sox input.mp3 output.mp3 trim 30 60   # Start at 30s, 60s duration

# Concatenate
sox input1.mp3 input2.mp3 output.mp3

# Change speed
sox input.mp3 output.mp3 speed 1.5

# Add silence
sox input.mp3 output.mp3 pad 1 2      # 1s before, 2s after
```

### yt-dlp (Video/Audio Downloads)
```bash
# Download video (best quality)
yt-dlp "https://www.youtube.com/watch?v=VIDEO_ID"

# Download audio only (best quality)
yt-dlp -x --audio-format mp3 "URL"
yt-dlp -x --audio-format opus "URL"

# List available formats
yt-dlp -F "URL"

# Download specific format by ID
yt-dlp -f 137+140 "URL"    # Video 137 + audio 140

# Download with subtitles
yt-dlp --write-subs --sub-langs en "URL"
yt-dlp --write-auto-subs --sub-langs en --convert-subs srt "URL"

# Download thumbnail and metadata
yt-dlp --write-thumbnail --write-info-json "URL"

# Download playlist
yt-dlp -o "%(playlist_index)s-%(title)s.%(ext)s" "PLAYLIST_URL"

# Limit quality (save bandwidth)
yt-dlp -f "bestvideo[height<=720]+bestaudio" "URL"

# Download with rate limit
yt-dlp --limit-rate 5M "URL"

# Batch download from file
yt-dlp -a urls.txt

# Extract audio, post-process with ffmpeg
yt-dlp -x --audio-format mp3 --audio-quality 0 --postprocessor-args "-ar 44100" "URL"
```

### whisper (Speech-to-Text)
```bash
# Basic transcription (auto-detect language)
whisper audio.mp3

# Specify language
whisper audio.mp3 --language en

# Choose model size (tiny, base, small, medium, large)
whisper audio.mp3 --model medium

# Output specific format
whisper audio.mp3 --output_format txt
whisper audio.mp3 --output_format srt
whisper audio.mp3 --output_format vtt
whisper audio.mp3 --output_format json

# Transcribe with timestamps
whisper audio.mp3 --output_format srt --model small

# Combined workflow: extract audio then transcribe
ffmpeg -i video.mp4 -vn -c:a pcm_s16le -ar 16000 audio.wav
whisper audio.wav --model medium --output_format srt

# Batch transcribe
for f in *.mp3; do whisper "$f" --model small --output_format srt; done
```

---

## Document Conversion (Pandoc)

### Basic Conversions
```bash
# Markdown to PDF
pandoc input.md -o output.pdf

# Markdown to HTML
pandoc input.md -o output.html

# Markdown to DOCX
pandoc input.md -o output.docx

# DOCX to Markdown
pandoc input.docx -o output.md

# HTML to Markdown
pandoc input.html -o output.md
```

### With Styling
```bash
# PDF with custom margins
pandoc input.md -V geometry:margin=1in -o output.pdf

# HTML with CSS
pandoc input.md -c style.css -o output.html

# Standalone HTML (includes headers)
pandoc -s input.md -o output.html
```

### PDF Options
```bash
# With table of contents
pandoc input.md --toc -o output.pdf

# Numbered sections
pandoc input.md --number-sections -o output.pdf

# Custom PDF engine
pandoc input.md --pdf-engine=xelatex -o output.pdf
```

---

## Diagrams

### Graphviz (dot)
```bash
# PNG output
dot -Tpng input.dot -o output.png

# SVG output
dot -Tsvg input.dot -o output.svg

# PDF output
dot -Tpdf input.dot -o output.pdf
```

### Mermaid
```bash
# Using mermaid-cli
mmdc -i input.mmd -o output.png
mmdc -i input.mmd -o output.svg
mmdc -i input.mmd -o output.pdf
```

---

## PDF Operations

### Combine PDFs
```bash
# qpdf
qpdf --empty --pages file1.pdf file2.pdf -- output.pdf

# pdftk
pdftk file1.pdf file2.pdf cat output combined.pdf
```

### Extract Pages
```bash
# qpdf
qpdf input.pdf --pages . 1-5 -- output.pdf

# pdftk
pdftk input.pdf cat 1-5 output output.pdf
```

### Images to PDF
```bash
img2pdf image1.jpg image2.jpg -o output.pdf
convert image1.jpg image2.jpg output.pdf
```

---

## Common Workflows

### Web-Ready Video
```bash
ffmpeg -i input.mp4 -c:v libx264 -crf 23 -preset medium \
  -c:a aac -b:a 128k -movflags +faststart output.mp4
```

### Batch Thumbnail Generation
```bash
for f in *.jpg; do
  convert "$f" -thumbnail 200x200^ -gravity center -extent 200x200 "thumb_${f}"
done
```

### Extract All Audio from Videos
```bash
for f in *.mp4; do
  ffmpeg -i "$f" -vn -c:a mp3 "${f%.mp4}.mp3"
done
```

### Optimize All Images in Directory
```bash
# JPEGs
find . -name "*.jpg" -exec jpegoptim --max=85 {} \;

# PNGs
find . -name "*.png" -exec optipng -o5 {} \;
```

### GIF Optimization and Editing (gifsicle)

```bash
# Optimize GIF
gifsicle -O3 input.gif -o optimized.gif        # Maximum optimization
gifsicle --lossy=80 input.gif -o smaller.gif   # Lossy compression
gifsicle --resize 320x240 input.gif -o resized.gif  # Resize

# Speed change
gifsicle --delay=5 input.gif -o faster.gif     # Speed up (5 = 50ms per frame)
gifsicle --delay=20 input.gif -o slower.gif    # Slow down

# Frame manipulation
gifsicle input.gif '#0-9' -o first10.gif       # Extract first 10 frames
gifsicle input.gif --rotate-90 -o rotated.gif  # Rotate
gifsicle a.gif b.gif > combined.gif            # Concatenate GIFs
```

### HTML/CSS to PDF Conversion (weasyprint)

```bash
weasyprint input.html output.pdf               # Convert HTML to PDF
weasyprint https://example.com page.pdf        # URL to PDF
weasyprint input.html output.pdf -s style.css  # With custom stylesheet
weasyprint - output.pdf < input.html           # From stdin
```

### Document Format Conversion (unoconv / libreoffice)

```bash
# unoconv
unoconv -f pdf document.docx                   # DOCX → PDF
unoconv -f html presentation.pptx              # PPTX → HTML
unoconv -f csv spreadsheet.xlsx                # XLSX → CSV

# libreoffice headless (alternative)
libreoffice --headless --convert-to pdf document.docx
libreoffice --headless --convert-to csv spreadsheet.xlsx
```

### Chroma Key (Green Screen)

```bash
# Remove green background with ffmpeg
ffmpeg -i greenscreen.mp4 -vf "colorkey=0x00FF00:0.3:0.2" -c:a copy transparent.webm

# Replace green with background image
ffmpeg -i greenscreen.mp4 -i background.jpg \
  -filter_complex "[0:v]colorkey=0x00FF00:0.3:0.2[fg];[1:v][fg]overlay" \
  -c:a copy output.mp4
```

### Combined Workflows

```bash
# yt-dlp → ffmpeg → whisper pipeline
yt-dlp -x --audio-format wav -o "audio.wav" "URL" && whisper audio.wav --model base --output_format txt

# Batch image optimization
fdfind -e jpg | xargs -P4 -I{} jpegoptim --max=85 {}
fdfind -e png | xargs -P4 -I{} optipng -o3 {}

# Video thumbnail strip
ffmpeg -i video.mp4 -vf "fps=1/10,scale=160:-1,tile=10x1" -frames:v 1 thumbnails.jpg
```

---

## Installation

```bash
# Core tools
sudo apt install ffmpeg imagemagick exiftool

# Image optimization
sudo apt install jpegoptim optipng pngquant webp

# Audio tools
sudo apt install sox

# Document conversion
sudo apt install pandoc
sudo apt install texlive-xetex    # For PDF output

# PDF tools
sudo apt install qpdf img2pdf

# Diagrams
sudo apt install graphviz
npm install -g @mermaid-js/mermaid-cli

# Video/audio downloads
pip install yt-dlp

# Speech-to-text
pip install openai-whisper
```
