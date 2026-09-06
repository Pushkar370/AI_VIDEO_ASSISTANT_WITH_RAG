# 🎬 AI Video Assistant

Turn any YouTube video or local audio/video file into a searchable, summarized meeting-style report — transcribed locally, summarized with an LLM, and chattable via RAG.

## Features

- **Input**: YouTube URL or local audio/video file
- **Transcription**: local Whisper for English audio, Sarvam AI for Hindi/Hinglish (translates to English while transcribing)
- **Title generation**: short, professional auto-generated title from the transcript
- **Summarization**: map-reduce style — chunks the transcript, summarizes each chunk, then combines into one final bullet-point summary
- **Extraction**: action items (with owner + deadline), key decisions, and open/unresolved questions, each pulled out separately
- **RAG chat**: ask follow-up questions about the meeting/video, answered strictly from the transcript via a Chroma vector store
- **Two interfaces**: a CLI (`main.py`) and a Streamlit dashboard (`app.py`)

## Tech Stack

| Layer | Tool |
|---|---|
| Audio acquisition | `yt-dlp`, `pydub`, `ffmpeg` |
| Transcription (English) | OpenAI Whisper (local) |
| Transcription (Hinglish) | Sarvam AI STT-translate API |
| LLM orchestration | LangChain (LCEL) |
| LLM | Mistral AI (`mistral-small-2603`) |
| Vector store | ChromaDB |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`) |
| UI | Streamlit |

## Setup

**1. Install dependencies**
```bash
pip install -r Requirements.txt
```
FFmpeg must also be installed separately and available on your system PATH (required by `yt-dlp` and `pydub`).

**2. Configure environment variables**

Create a `.env` file in the project root:
```env
MISTRAL_API_KEY=your_mistral_api_key
WHISPER_MODEL=small
SARVAM_API_KEY=your_sarvam_api_key
SARVAM_STT_MODEL=saaras:v2.5
```

## Usage

**CLI**
```bash
python main.py
```
You'll be prompted for a YouTube URL (or local file path) and a language (`english` or `hinglish`). Once processing finishes, you can chat with the transcript directly in the terminal (`exit` to quit).

**Streamlit dashboard**
```bash
streamlit run app.py
```
Paste a URL or file path in the sidebar, choose a language, and hit **Analyse**. Results (title, summary, action items, decisions, open questions) render as cards, with a chat panel below for RAG-based Q&A.

## Project Structure

```
.
├── main.py                    # CLI entry point
├── app.py                     # Streamlit UI
├── Requirements.txt
├── .env                       # API keys (not committed)
└── core/
    ├── transcriber.py         # Whisper / Sarvam routing + transcription
    ├── summarizer.py          # Title generation + map-reduce summarization
    ├── extractor.py           # Action items / decisions / open questions
    ├── rag_engine.py          # RAG chain: retrieval + Q&A
    ├── vector_store.py        # Chroma vector store build/load
    └── audio_processor.py     # Download, convert, and chunk audio
```

## Known Limitations

- **Mistral free-tier rate limits**: the free tier caps requests at roughly 1 request/second and a limited token budget per minute/month. Each pipeline run fires several sequential LLM calls (title, per-chunk summaries, 3 extraction calls), so a 429 rate-limit error is possible on longer transcripts or repeated runs in quick succession. Retry-with-backoff is built into each LLM call to handle this automatically; persistent 429s likely mean the account's monthly quota needs checking at `admin.mistral.ai/plateforme/limits`.
- **Temp files**: downloaded/converted audio and per-chunk WAV files are written to disk during processing and are not automatically deleted afterward.
- **No YouTube JS runtime by default**: `yt-dlp` may show a warning about a missing JavaScript runtime (`deno`) — install one per [yt-dlp's guide](https://github.com/yt-dlp/yt-dlp/wiki/EJS) if you hit extraction issues on certain videos.

## Roadmap

- Export summary/report as PDF or plain text
- Automatic cleanup of temporary audio files after a run
- Resume a past session's RAG chat from a saved vector store collection
