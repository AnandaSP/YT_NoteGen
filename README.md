# YT NoteGen

Turn a YouTube lecture, or a whole playlist, into notes that keep the spoken words next to the frame that was on screen.

There is no language model. Captions come from YouTube. Frames come from the video. The notebook lines the two up by time and writes a self-contained HTML page and a Word file.

Open [`Notebooks/YT_LectureNotes.ipynb`](Notebooks/YT_LectureNotes.ipynb) and run it top to bottom. It works in Jupyter and in Google Colab.

## What you get

For one video, under `OUTPUT_DIR`:

```text
output/
  index.html
  Lecture_Title_videoId/
    notes.html
    notes.docx
    frames/
```

`notes.html` embeds each frame as an image, so the pictures still show if you download only that file. Each block links back to that moment on YouTube.

When the video has chapters, both files open with a chapter list. In HTML each chapter is a collapsible section. In Word each chapter is a heading you can jump to from the list. Speech that falls outside any chapter stays in the normal sequence.

A playlist also writes `output/lecture_notes.docx`, one document with a section per video, and `index.html` links every video.

## How the notes are arranged

Words spoken between frame 9 and frame 10 are printed above frame 10. The opening frame has an empty window when it sits at the start of the video. The last frame keeps any speech through the end of the video.

If a cut lands in the middle of a sentence, the rest of that sentence is pulled up so it finishes before the image. A `.` `?` `!`, or a pause of about `paragraph_pause` seconds, ends the sentence.

YouTube’s automatic captions repeat earlier words in each new cue. Repeated words are dropped. New words keep the time of the cue they arrived in, so the lecture is not glued into one paragraph.

## Run it

1. Open `Notebooks/YT_LectureNotes.ipynb`.
2. Run the cells from the top through **Settings**. The setup cell installs `yt-dlp`, `pillow`, `python-docx`, and `imageio-ffmpeg`. That last package includes an ffmpeg binary, so a separate ffmpeg install is optional.
3. In the **Run** cell, set `SOURCE_URL` to a video or a `/playlist?list=...` URL.
4. Run that cell. On Colab, set `OUTPUT_DIR` to something like `/content/output` if you want the files on the Colab disk.

Node.js on `PATH` lets yt-dlp see more YouTube formats. Captions and scene cuts still work without it. The setup cell looks for Node on `PATH`, in nvm, and in the usual Homebrew locations.

For a long playlist, set `PLAYLIST_LIMIT` to a small number while you try the settings.

## Cookies

YouTube sometimes answers caption requests with HTTP 429. The notebook waits and retries, then tries another caption format.

If that still fails, export a Netscape `cookies.txt` from a browser where you are logged into YouTube, upload it, and set `cookies_file` to that path. Leave `cookies_from_browser` as `None`.

`cookies_from_browser` can read Chrome or Firefox on the same machine as the kernel. Safari works only when that kernel is running on macOS. On Linux or Colab, use `cookies_file`.

Set one of those, not both.

## Settings

| Setting | Default | What it does |
| --- | --- | --- |
| `max_height` | `720` | Tallest video downloaded. Frames are taken from that file. |
| `scene_threshold` | `0.30` | ffmpeg scene score, between 0 and 1. Lower it toward `0.15` when a small slide change is missed. |
| `min_scene_gap` | `1.0` | Drop a cut that arrives sooner than this many seconds after the previous kept frame. |
| `max_gap` | `40` | Insert an extra frame when the picture stays still this long, so a talking-head stretch is not one paragraph. `0` keeps only real scene cuts. |
| `duplicate_threshold` | `6` | Average color difference, 0–255. Near-identical frames are dropped. Raise it toward `12` if animations still leave duplicates. |
| `paragraph_pause` | `1.15` | Silence, in seconds, that starts a new paragraph and stops a sentence from being pulled across a frame. |
| `text_then_image` | `True` | Speech, then the frame. `False` puts the frame first. |
| `keep_video` | `False` | Keep the downloaded video next to the notes. |
| `sub_langs` | `("en",)` | Exact caption language codes. A manual track is used when YouTube has one; otherwise auto-captions are used. |
