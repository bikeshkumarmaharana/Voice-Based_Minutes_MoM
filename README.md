# Voice-Based Minutes of Meeting (MoM) Pipeline

An end-to-end, locally-run pipeline that processes meeting recordings (audio/video), performs speaker diarization, transcribes multilingual speech (English, Hindi, and Odia), translates for summarization, extracts key discussion points, decisions, and action items, and exports structured results into JSON, DOCX, and PDF formats.

## Features
- **Fully Local / Open Source**: Runs without relying on paid proprietary LLM APIs.
- **Multilingual Support**: Handles English, Hindi, and features a dedicated model path for Odia.
- **Speaker Diarization**: Identifies and attributes dialogue to distinct speakers (e.g., Person 1, Person 2).
- **Automated Summarization & Extraction**: Generates meeting summaries, key discussion points, formal decisions, and assigned action items using transformer models.
- **Multiple Export Formats**: Automatically outputs structured JSON, formatted DOCX, and PDF documents.
- **Interactive UI**: Powered by Gradio for easy drag-and-drop file processing.

---

## Architecture & Technology Choices

The pipeline is structured into clear sequential modules:
1. **Preprocessing**: Uses `ffmpeg` to standardize input audio to 16kHz mono WAV, accompanied by defensive sample cleanup.
2. **Speaker Diarization**: Uses `pyannote/speaker-diarization-3.1` (pinned to CPU for CUDA stability) to segment audio by unique speakers.
3. **Speech-to-Text (ASR)**:
   - *English & Hindi*: Uses `faster-whisper` (CTranslate2 port of OpenAI Whisper) running on GPU with VAD filtering.
   - *Odia*: Uses Meta's Multilingual Massively Scaled Speech model (`facebook/mms-1b-all`) with the Odia (`ory`) adapter, as Whisper does not natively support Odia.
4. **Speaker Attribution & Merging**: Maps Whisper/MMS text segments to diarization time-turns and merges consecutive fragments from the same speaker.
5. **Translation & Summarization**: Non-English segments are translated to English via `facebook/nllb-200-distilled-600M` to feed a robust summarization pipeline using `facebook/bart-large-cnn`.
6. **Information Extraction**: Uses `google/flan-t5-large` with targeted prompts to extract bulleted key points, decisions, and action items.
7. **Export**: Compiles final outputs into a structured JSON file, a python-docx generated document, and an fpdf2-generated PDF.

---

## Setup and Execution Instructions

### Prerequisites
- Google Colab with a GPU runtime (`Runtime -> Change runtime type -> T4 GPU`) is recommended.
- A free Hugging Face account and an Access Token (with permissions to accept gated model licenses for Pyannote).

### Step-by-Step Execution
1. Open the project Jupyter/Colab notebook (`Copy of Voice-Based_Minutes_MoM.ipynb`)[cite: 7].
2. Run **Cell 1** to install all required Python libraries and system dependencies (ffmpeg). The runtime will automatically restart once upon first installation[cite: 7].
3. Run **Cell 2** to authenticate with Hugging Face using your access token[cite: 7].
4. Run **Cell 3** to load all core AI models up-front into memory[cite: 7].
5. Run **Cell 4** to initialize the underlying processing functions[cite: 7].
6. Run **Cell 5** to launch the Gradio web interface, then use the provided public URL to upload an audio/video recording and generate your Minutes of Meeting[cite: 7].

---

## Test Results and Known Limitations

### Test Results
- Successfully tested on multi-speaker English, Hindi, and Odia meeting snippets.
- Speaker statistics and conversation tracking accurately compute speaking percentages and timelines.
- Clean JSON, DOCX, and PDF artifacts are successfully generated and downloadable directly through the UI.

### Known Limitations
- **Odia Transcription**: Relies on a lower-resource MMS adapter, which exhibits lower accuracy than English/Hindi and lacks automatic punctuation due to CTC constraints.
- **Language Auto-Detection**: Whisper samples only the initial ~30 seconds of audio; accents or quiet openings can occasionally cause misidentification (mitigated by the manual language dropdown).
- **Overlapping Speech**: Heavy cross-talk or simultaneous interruptions reduce diarization precision.
- **Speaker Consistency**: Speaker labels are scoped strictly to the uploaded file and do not persist across separate meetings.