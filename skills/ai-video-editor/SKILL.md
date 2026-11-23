---
name: ai-video-editor
description: Set up and run Auto-Cut AI (AI-Video-Editor), a Python command-line tool that turns a raw talking-head video into an edited social clip with animated captions, stock B-roll, background music, sound effects and transitions, by transcribing with faster-whisper and having an LLM write the editing script. Use when the user wants to auto-edit a talking-head or vlog video into a short clip with this tool, configure its LiteLLM, Pexels or Pixabay keys, or fix a failed run.
---

# AI Video Editor (Auto-Cut AI)

The pipeline runs in four steps:
1. Transcribe the audio with word-level timestamps (`transcriber.py`, faster-whisper).
2. Send the transcript to an LLM through LiteLLM, which returns a JSON editing script: scenes, transitions, caption styles and B-roll (`llm_handler.py`).
3. Download the B-roll from Pexels and the music and sound effects from Pixabay (`asset_manager.py`).
4. Render the video with moviepy and Pillow (`video_processor.py`).

## Set up

Needs Python 3.10 or newer. moviepy also needs FFmpeg.

```bash
python -m venv venv
venv\Scripts\activate          # macOS/Linux: source venv/bin/activate
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill it in:

| Variable | What it is |
|---|---|
| `LITELLM_MODEL` | Any LiteLLM model id. The default is `mistral/magistral-medium-2506`; `openai/gpt-4o` or another provider's model also works |
| `LITELLM_API_KEY` | The key for that model's provider |
| `PEXELS_API_KEY` | Free key from pexels.com/api, for B-roll |
| `PIXABAY_API_KEY` | Free key from pixabay.com/api/docs, for music and sound effects |

The user supplies the keys. Never invent them, print them or commit `.env`. The run stops with "LITELLM_MODEL or LITELLM_API_KEY not found." if either is missing.

## Run

```bash
python main.py --input_video my_video.mp4
python main.py --input_video my_video.mp4 --output_name awesome_clip.mp4
```

The output goes to the `output/` folder; the default name is `final_corrected_video.mp4`. Put the input in the project folder or pass its path.

## Tips and troubleshooting

- Each run sends the transcript to the chosen LLM provider and downloads stock media, which can cost API credits. Tell the user before running on long videos.
- **Rate limits:** the LLM call reports LiteLLM's `RateLimitError`. Wait and retry, or switch `LITELLM_MODEL`.
- **No B-roll or music:** check the Pexels and Pixabay keys, and whether the searches found anything.
- **Slow transcription or rendering:** both run on the CPU by default, so long or high-resolution videos take a while. Trim the input first for a quick test.
- **Odd edits:** the LLM's editing script decides the cuts. Try a stronger model, or rerun, since the result varies.
- Stock media keeps its own licence (Pexels and Pixabay terms). Remind the user before they publish.

The repository has no licence file, so its own terms of use aren't stated.

Report the output path and length when a run finishes. Never claim a video was made without seeing the file.
