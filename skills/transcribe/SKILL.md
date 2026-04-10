---
name: transcribe
description: Create clean, subtitle-ready `.srt` subtitles for a local audio or video file using Whisper CLI. Use this when the user asks to transcribe a local media file, generate subtitles, create captions, or make an `.srt` for a specific local file path.
---

# transcribe

## Input
Use the media file path the user provided. If it's unclear which file to transcribe, ask the user for the local media file path and stop.

## Instructions
When this skill is invoked, do the work immediately.

1. Run Whisper on the input file and write an `.srt` file next to the source media file.
2. Review the generated subtitles and make them clean and subtitle-ready rather than overly literal.
3. Reveal the final `.srt` file in Finder.
4. Report the output `.srt` path.

## Subtitle cleanup requirements
You must optimize the subtitles for readability while preserving meaning and approximate timing.

- Remove obvious filler words when they do not add meaning, such as `um`, `uh`, `like`, and similar verbal padding.
- Remove obvious false starts, accidental repetitions, and disfluent fragments when they make captions harder to read.
- Fix punctuation, capitalization, and obvious formatting problems.
- Preserve intentional wording, important emphasis, and technical terms.
- Do not rewrite the speaker's meaning or turn the transcript into a summary.
- Prefer concise, natural subtitle text over raw verbatim speech.

## Command
Run Whisper with the provided media file:

```bash
INPUT="<media file path>"
whisper "$INPUT" --output_format srt --output_dir "$(dirname "$INPUT")"
```

The generated subtitle path is:

```bash
SRT_PATH="$(dirname "$INPUT")/$(basename "${INPUT%.*}").srt"
```

After cleaning the subtitle text, reveal the file in Finder:

```bash
open -R "$SRT_PATH"
```

## Output
Report the final `.srt` path:

`$SRT_PATH`

If Whisper fails, report the error briefly and stop.
