---
name: preserve-video-quality
description: Preserve source detail when cutting and rendering videos with HyperFrames. Use when an AI edit unexpectedly looks softer, text becomes blurry, or a final export needs a quality check.
---

# Preserve video quality

Protect visible detail from ingestion through the final MP4. A smaller file alone is not proof of lost quality; compare the same source and output frames. Never claim that increasing bitrate can restore detail already lost in an earlier encode.

## Before editing

1. Keep the camera or screen recording as an immutable source. Do not replace it with a proxy, chat upload, preview, or compressed transcript file.
2. Inspect source dimensions, frame rate, codec, and pixel format:

   ```bash
   ffprobe -v error -select_streams v:0 -show_entries stream=codec_name,width,height,avg_frame_rate,pix_fmt,bit_rate -of json "SOURCE.mp4"
   ```

3. Record the intended output dimensions and frame rate. Preserve the source aspect ratio. Do not upscale merely to report a larger resolution, and do not change 30 fps to 60 fps without a creative reason.

## Build the edit

- Keep silence and mistake decisions as timestamps or an EDL against the original recording. Avoid a chain of compressed MP4 intermediates. Each lossy re-encode can remove more detail, especially from fine text and screen recordings.
- If an intermediate is unavoidable, use a lossless or visually lossless format and document the storage cost. For a simple cut, consider stream copy only when keyframe boundaries and exact A/V sync make it safe; verify the actual result.
- In HyperFrames, set the composition's dimensions to the intended delivery size. Inspect source scaling and crop: readable text and faces matter more than a nominal export resolution.
- Use `draft` only for iteration. For a final render on HyperFrames versions supporting these flags:

   ```bash
   npx hyperframes render --quality high --output renders/final.mp4
   ```

  When the source includes UI, screen capture, or color-sensitive graphics, add `--video-frame-format png` to avoid JPEG extraction artifacts. If the final encode is visibly soft, test `--crf 18` and then a lower CRF on a short representative segment. Lower CRF usually means a larger file. Do not combine `--crf` with `--video-bitrate`. Check `npx hyperframes render --help` for the installed version before using optional flags.
- Do not add a separate FFmpeg compression pass just to reduce file size. When changing audio only, copy the video stream if the container and codec permit it (`-c:v copy`), then confirm A/V sync.

## Verify the encoded result

1. Probe the final MP4 with the same `ffprobe` command. Explain any intentional dimension, frame-rate, or pixel-format changes.
2. Compare matching moments from the original and final video at 100% display size. For edited footage, use the EDL to map each output moment to its source moment. Include a face close-up, fine on-screen text, motion, and a dark or high-contrast scene where available. Check sharpness, blockiness, color, banding, dropped/duplicated frames, and audio sync.
3. Decode a few frames from the **encoded output**, not just the browser preview:

   ```bash
   ffmpeg -ss 00:00:30 -i renders/final.mp4 -frames:v 1 -update 1 final-check.png
   ```

4. If a defect appears, identify the first stage where it appears: source, extraction, intermediate cut, composition scaling, final encode, or video-platform playback. Fix that stage and render again. A platform may transcode uploads; judge the local master separately from its streamed copy.
5. Report the final settings, source/output metadata, inspected timestamps, and any remaining visible defects. Do not certify quality solely from bitrate, file size, metadata, or a successful render command.
