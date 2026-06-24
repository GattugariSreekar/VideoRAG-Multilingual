# VideoRAG-Multilingual — Multimodal Video Question Answering System

Ask questions about any YouTube video in any language and receive answers in the same language the question was asked.

Built with: Whisper · CLIP · GPT-4o · LlamaIndex · LanceDB · Gradio

---

## What This Project Does

**VideoRAG-Multilingual** is an end-to-end **Multimodal Retrieval-Augmented Generation (RAG)** system that allows users to:

- Paste any YouTube URL (video in any language)
- Ask questions about the video content in any language (English, Telugu, Hindi, French, Japanese, etc.)
- Get answers automatically delivered in the same language the question was asked
- See visual evidence — the exact video frames that support the AI's answer

The system combines **computer vision (frames)**, **speech-to-text (audio)**, and **large language model reasoning (GPT-4o)** into a unified pipeline, making it capable of understanding both what is spoken and what is visually shown in a video.

---

## System Architecture

```
+---------------------------------------------------------------------+
|                        USER (Any Language)                          |
|                     Gradio Web Interface (UI)                       |
+----------------------------+----------------------------------------+
                             | YouTube URL
                             v
+---------------------------------------------------------------------+
|                    VIDEO INGESTION LAYER                            |
|                                                                     |
|   yt-dlp --> MP4 Video File (best quality, auto-format selection)  |
+----------------------+----------------------+-----------------------+
                       |                      |
          +------------+--------+   +---------+----------------+
          |  VISUAL PIPELINE    |   |    AUDIO PIPELINE        |
          |                     |   |                          |
          |  MoviePy            |   |  MoviePy (audio strip)   |
          |  |                  |   |  |                       |
          |  30 evenly-spaced   |   |  MP3 @ 32kbps            |
          |  JPEG frames        |   |  |                       |
          |  (512x288 px)       |   |  OpenAI Whisper-1        |
          |  |                  |   |  (Auto lang detection)   |
          |  CLIP Embeddings    |   |  |                       |
          |                     |   |  transcript.txt          |
          +------------+--------+   +---------+----------------+
                       |                      |
                       +----------+-----------+
                                  |
              +-------------------v---------------------------+
              |        MULTIMODAL INDEXING LAYER              |
              |                                               |
              |   LlamaIndex MultiModalVectorStoreIndex       |
              |   +---------------+  +------------------+    |
              |   |  Text Store   |  |   Image Store    |    |
              |   |  (LanceDB)    |  |   (LanceDB)      |    |
              |   +---------------+  +------------------+    |
              +-------------------+---------------------------+
                                  |
                                  | User Query
              +-------------------v---------------------------+
              |          RETRIEVAL LAYER                      |
              |                                               |
              |   Similarity Search                           |
              |   top-k=3 text chunks + top-k=3 images       |
              +-------------------+---------------------------+
                                  |
              +-------------------v---------------------------+
              |      MULTILINGUAL GENERATION LAYER            |
              |                                               |
              |   langdetect  -->  Detect user query language |
              |   |                                           |
              |   GPT-4o Multimodal                           |
              |   (Text context + Retrieved images)           |
              |   |                                           |
              |   Answer in USER'S language                   |
              +-------------------+---------------------------+
                                  |
              +-------------------v---------------------------+
              |            OUTPUT LAYER                       |
              |                                               |
              |   Text Answer (in user's language)            |
              |   Visual Evidence (retrieved frames)          |
              +-----------------------------------------------+
```

---

## Key Features

| Feature | Description |
|---|---|
| **True Multilingual** | Supports 20+ languages — English, Hindi, Telugu, Tamil, Spanish, French, German, Japanese, Korean, Chinese, and more |
| **Video Intelligence** | Understands both visual content (frames) and spoken content (audio) |
| **Multimodal RAG** | Retrieves the most relevant text chunks and video frames to answer each question |
| **GPT-4o Reasoning** | Uses OpenAI's most capable multimodal model for grounded, accurate answers |
| **Visual Evidence** | Returns the actual video frames that support the AI's answer |
| **Auto Language Detection** | Detects the user's question language and responds in the same language automatically |
| **Secure API Handling** | API keys loaded from environment variables / Colab Secrets — never hardcoded |
| **Interactive UI** | Clean Gradio interface — no coding required to use |

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Video Download** | `yt-dlp` |
| **Frame Extraction** | `MoviePy`, `Pillow` |
| **Audio Transcription** | `OpenAI Whisper-1` |
| **Image Embeddings** | `OpenAI CLIP` |
| **RAG Framework** | `LlamaIndex` (MultiModalVectorStoreIndex) |
| **Vector Database** | `LanceDB` (separate text + image stores) |
| **LLM** | `GPT-4o` (OpenAI) |
| **Language Detection** | `langdetect` |
| **Frontend / UI** | `Gradio` |
| **Runtime** | `Google Colab` (GPU-accelerated) |
| **Media Processing** | `ffmpeg` |

---

## Project Structure

```
VideoRAG-Multilingual/
|
+-- Video_RAG_Multilingual_GPT4o.ipynb   # Main Colab notebook (full pipeline)
|
+-- README.md                             # Project documentation
|
+-- requirements.txt                      # Python dependencies
|
+-- assets/
    +-- architecture.png                  # Architecture diagram (optional)
```

**Runtime folders (auto-created during execution):**

```
video_data/       <-- Downloaded YouTube video (MP4)
mixed_data/       <-- Extracted frames (JPEG) + audio transcript (TXT)
lancedb/          <-- LanceDB vector database (text + image tables)
```

---

## How to Run (Google Colab)

### Step 1: Open the Notebook

Upload `Video_RAG_Multilingual_GPT4o.ipynb` to [Google Colab](https://colab.research.google.com).

### Step 2: Set Your OpenAI API Key

In Google Colab, go to the **Secrets** tab (left sidebar) and add:

```
Name:  OPENAI_API_KEY
Value: sk-...your-key-here...
```

> Never paste your API key directly in the notebook code.

### Step 3: Run All Cells

Go to **Runtime > Run all** (or press `Ctrl+F9`).

The notebook will:
1. Install all dependencies automatically
2. Install `ffmpeg` for audio/video processing
3. Load all modules and configure the pipeline
4. Launch the Gradio interface with a public share link

### Step 4: Use the Interface

Once Gradio launches, a public URL will appear:
```
Running on public URL: https://xxxx.gradio.live
```

In the UI:
1. Paste a YouTube video URL into the **YouTube URL** box
2. Click **Process Video** and wait for the pipeline to complete (~1-3 minutes)
3. Once processing is confirmed, type your question in any language
4. Click **Ask AI** to receive the answer and supporting visual frames

---

## Supported Languages

The system auto-detects and responds in the user's question language:

| Code | Language | Code | Language |
|------|----------|------|----------|
| `en` | English | `hi` | Hindi |
| `te` | Telugu | `ta` | Tamil |
| `kn` | Kannada | `ml` | Malayalam |
| `mr` | Marathi | `bn` | Bengali |
| `gu` | Gujarati | `pa` | Punjabi |
| `ur` | Urdu | `es` | Spanish |
| `fr` | French | `de` | German |
| `it` | Italian | `pt` | Portuguese |
| `ru` | Russian | `ja` | Japanese |
| `ko` | Korean | `zh` | Chinese |

> The video can be in any language. The answer will always match the question's language.

---

## Pipeline Deep Dive

### 1. Video Download (`yt-dlp`)
- Downloads the best available MP4 format
- Supports optional `cookies.txt` for age-restricted or authenticated videos
- Extracts metadata: title, uploader, duration, view count

### 2. Frame Extraction (`MoviePy` + `Pillow`)
- Samples **30 evenly-spaced frames** across the full video duration
- Resizes each frame to **512x288 pixels** for efficient embedding
- Saves as JPEG at 80% quality to the `mixed_data/` folder

### 3. Audio Transcription (`OpenAI Whisper-1`)
- Strips audio from the video as **32kbps MP3**
- Sends to **OpenAI Whisper API** for cloud transcription
- Whisper auto-detects the spoken language (supports 90+ languages)
- Outputs `transcript.txt` saved alongside the extracted frames

### 4. Multimodal Indexing (`LlamaIndex` + `LanceDB` + `CLIP`)
- `SimpleDirectoryReader` loads both frames and transcript
- **CLIP embeddings** applied to image frames for visual semantic search
- **Text embeddings** applied to transcript chunks
- Two separate **LanceDB vector tables** created per session (text store + image store)
- UUID-suffixed table names prevent data collisions across runs

### 5. Multilingual Query Engine (`langdetect` + `GPT-4o`)
- `langdetect` identifies the language of the user's question
- Maps detected language code to a human-readable name (20+ languages supported)
- Constructs a multilingual system prompt instructing GPT-4o to:
  - Use transcript context as the primary information source
  - Translate internally when the video language differs from the answer language
  - Always reply in the user's detected language
- Retrieves top-3 text chunks + top-3 image frames from vector stores
- Passes both text context and image nodes to **GPT-4o multimodal**

### 6. Gradio UI
- Clean, no-code interface with URL input, video preview, and metadata display
- Session state management using `gr.State` for index and metadata objects
- Displays the AI-generated answer alongside a gallery of retrieved visual evidence frames

---

## Dependencies

```
openai
llama-index
llama-index-vector-stores-lancedb
llama-index-multi-modal-llms-openai
llama-index-embeddings-clip
yt-dlp
gradio
moviepy
Pillow
langdetect
lancedb
SpeechRecognition
openai-whisper (installed from GitHub)
CLIP (installed from GitHub)
ffmpeg (system package)
```

Full install commands are included at the top of the notebook.

---

## Configuration

The following constants can be adjusted at the top of the notebook to balance performance and cost:

| Parameter | Default | Description |
|---|---|---|
| `MAX_FRAMES` | `30` | Number of frames sampled from the video |
| `IMG_SIZE` | `(512, 288)` | Resolution of extracted frames |
| `similarity_top_k` | `3` | Number of text chunks retrieved per query |
| `image_similarity_top_k` | `3` | Number of image frames retrieved per query |
| `max_new_tokens` | `1024` | Maximum token length for GPT-4o response |

---

## Example Use Cases

| Scenario | Example Question |
|---|---|
| Education | "What is the main concept explained in this lecture?" |
| Telugu News | "ఈ వీడియోలో ప్రధాన విషయం ఏమిటి?" |
| Product Review | "What are the pros and cons mentioned?" |
| Hindi Tutorial | "इस वीडियो में कौन सी तकनीक बताई गई है?" |
| French Documentary | "Quel est le sujet principal de cette vidéo?" |
| Interview | "What skills does the speaker highlight?" |

---

## Security Notes

- API keys are loaded from **Colab Secrets** (`userdata.get`) or environment variables
- No hardcoded credentials anywhere in the codebase
- LanceDB tables use **UUID-based names** to prevent cross-session data leaks
- The `reset_environment()` function clears all temporary data before processing each new video

---

## Limitations and Future Improvements

**Current Limitations:**
- Processing time is approximately 1-3 minutes per video depending on length and Colab GPU availability
- Long videos (60+ minutes) may encounter Whisper API file size limits
- Some YouTube videos require cookie authentication via `cookies.txt`

**Planned Improvements:**
- Support for local video file uploads (not limited to YouTube URLs)
- Streaming responses for faster perceived response time
- Video timestamp citations embedded in answers
