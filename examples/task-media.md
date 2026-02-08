---
description: "Media processing - video won't play, image too large, need smaller file size, format incompatible, audio out of sync, video/audio conversion, image manipulation, and document conversion (ffmpeg, imagemagick, yt-dlp, whisper, pandoc)"
when_to_use: "Use when: video won't play or has wrong format, image too large and needs resizing, file size too big for upload or sharing, format incompatible with target device, audio out of sync, need to extract audio from video, creating thumbnails, stripping or reading metadata, converting documents, downloading video or audio from the web, transcribing speech. Triggers: video won't play, image too large, file size too big, format incompatible, audio out of sync, wrong format, can't open video, reduce file size, optimize image, ffmpeg, video convert, audio extract, image resize, thumbnail, exif, pdf convert, diagram, gif, imagemagick, compress image, overlay, watermark, drawtext, subtitle, hls, stabilize, crop video, yt-dlp, download video, whisper, transcribe, transcode, document convert, metadata strip, batch resize, image optimize, gifsicle, weasyprint, unoconv, pandoc."
when-to-use: "Use when: video won't play or has wrong format, image too large and needs resizing, file size too big for upload or sharing, format incompatible with target device, audio out of sync, need to extract audio from video, creating thumbnails, stripping or reading metadata, converting documents, downloading video or audio from the web, transcribing speech. Triggers: video won't play, image too large, file size too big, format incompatible, audio out of sync, wrong format, can't open video, reduce file size, optimize image, ffmpeg, video convert, audio extract, image resize, thumbnail, exif, pdf convert, diagram, gif, imagemagick, compress image, overlay, watermark, drawtext, subtitle, hls, stabilize, crop video, yt-dlp, download video, whisper, transcribe, transcode, document convert, metadata strip, batch resize, image optimize, gifsicle, weasyprint, unoconv, pandoc."
argument-hint: "[describe what you want to do with media]"
context: inherit
model: inherit
allowed-tools: Skill, Read
---

# Media Processing Task Router

You have access to the **full conversation context**. Your job is to build a well-formed argument for the media processing recipes skill.

**User's request**: $ARGUMENTS

---

## Your Task

Analyze the conversation to identify:

### 1. Goal
What media operation is needed?
- Convert format, extract audio, resize, create thumbnail, read/write metadata, compress, transform
- Overlay/watermark, text overlay, color correction, subtitle, stabilize, crop, stack, stream (HLS/DASH)
- Download video/audio (yt-dlp), transcribe audio (whisper)

### 2. Input File(s)
What media files?
- Explicit paths mentioned
- Files created/downloaded earlier
- File type (video, audio, image, document)

### 3. Output Requirements
What should the result be?
- **Format**: mp4, webm, mp3, jpg, png, pdf, gif
- **Quality**: resolution, bitrate, compression level
- **Dimensions**: specific size, aspect ratio

### 4. Processing Options
Special handling needed?
- Trim/cut (start time, duration)
- Scale/resize
- Extract frames
- Strip/preserve metadata
- Batch processing

### 5. Output Location
- Same directory as input
- Specific output path
- Output naming pattern

---

## Goal-to-Tool Mapping

| Goal | Primary Tool | Notes |
|------|-------------|-------|
| Video conversion | ffmpeg | Universal format converter |
| Audio extraction | ffmpeg -vn | Strip video, keep audio |
| Image resize/convert | imagemagick (convert) | Batch with mogrify |
| Image metadata | exiftool | Read/write EXIF data |
| GIF optimization | gifsicle | Reduce GIF file size |
| Image optimization | optipng / jpegoptim | Lossless PNG/JPEG compression |
| Document conversion | pandoc | Markdown/HTML/PDF/DOCX |
| Audio processing | sox | Audio effects, format conversion |
| Video download | yt-dlp | Download from video sites |
| Audio transcription | whisper | Speech-to-text |
| Diagram generation | graphviz (dot) | Graph/flowchart rendering |

---

## Argument Format

Construct a structured argument string:

```
<goal> from <input_path> to <output> - options: <settings>
```

**Include only relevant components.**

### Examples

**Video conversion:**
```
convert /home/user/video.mov to webm - resolution: 1280x720, quality: good
```

**Extract audio:**
```
extract audio from /tmp/movie.mp4 to mp3 - bitrate: 192k
```

**Video trimming:**
```
trim /home/user/video.mp4 - start: 00:01:30, duration: 00:00:45, output: clip.mp4
```

**Image resize:**
```
resize /tmp/photo.jpg to 800x600 - maintain aspect ratio, output: photo_small.jpg
```

**Create thumbnail:**
```
create thumbnail from /home/user/photo.jpg - size: 150x150, crop to square
```

**Batch conversion:**
```
convert all PNG files in ./screenshots/ to JPG - quality: 85
```

**Extract video frame:**
```
extract frame at 00:00:05 from /tmp/video.mp4 - output: frame.jpg
```

**Read metadata:**
```
read EXIF metadata from /home/user/photos/*.jpg - output: JSON
```

**Strip metadata:**
```
strip all metadata from /tmp/upload.jpg - preserve image quality
```

---

## Invoke Recipes

Once you've built the argument, use the Skill tool:

```
Skill: task-media-recipes
Args: <your constructed argument>
```

### When to Ask Instead

**Ask the user if:**
- Input file path is unclear
- Quality/format preference not specified
- Batch operation scope is ambiguous

**Use Read tool first if:**
- Need to verify file exists
- Need to check input format before choosing conversion
