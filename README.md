# 🎧 STEM — AI Audio Source Separation Platform

A full-stack machine learning application that separates audio files into individual stems and classifies each stem by genre or environmental sound category. The platform supports two distinct modes — one for music and one for nature recordings — each backed by a custom-trained ResNet-18 model.

> Built as part of Applied AI Mini Project — Team 5

---

🔗 **Live Demo:** https://stem-seperation.vercel.app  
🤗 **Backend API:** https://zen-4011-audio-separation-model.hf.space

---

## 🖼️ Preview

<img width="1280" height="718" alt="Home Page" src="https://github.com/user-attachments/assets/d81576fb-9789-4324-80a0-ee338874ae2b" />
<img width="1280" height="756" alt="Results Page" src="https://github.com/user-attachments/assets/3f0fca09-36be-427f-8b7c-c0912e6487a2" />
<img width="1280" height="750" alt="Stems View" src="https://github.com/user-attachments/assets/126dc48f-5014-4a84-9fc3-8f68a4c37c33" />

---

## 🚀 Key Features

- 🎼 **Audio Source Separation** — Split music into vocals, drums, bass, and other using Demucs
- 🧠 **AI Genre Classification** — Custom-trained ResNet-18 models for music and nature sounds
- 🌊 **Interactive Waveforms** — Powered by WaveSurfer.js
- 📊 **Mel Spectrograms** — Visual frequency representation of each stem
- ⚡ **Real-time Upload Preview** — Web Audio API visualization before submission
- 🟢 **Backend Health Monitoring** — Auto-detects model readiness before processing
- 📥 **Downloadable Stems** — Export each separated track as WAV

---

## 🧩 What It Does

Upload a WAV, MP3, FLAC, or OGG file and the platform will:

**🎵 Music Mode** — Run the audio through Facebook's Demucs (htdemucs model) to separate it into four stems: vocals, drums, bass, and other. Each stem is then passed through a custom ResNet-18 classifier trained on the GTZAN dataset to predict its genre. A mel spectrogram is generated for each stem and displayed alongside an interactive waveform player.

**🌿 Nature Mode** — Skip the separation step and classify the full recording using a ResNet-18 model trained on the ESC-50 dataset across 50 environmental sound categories. Returns the top 3 predictions with confidence scores and a full-clip spectrogram.

In both modes, individual stems or the full recording can be played back in the browser and downloaded as WAV files.

---

## 🤖 AI Models

Both classifiers share the same base architecture: a ResNet-18 pretrained on ImageNet, adapted for single-channel mel spectrogram input by modifying the first convolutional layer to accept one channel instead of three. The RGB weights are averaged across channels to preserve the pretrained feature representations. The final fully connected layer is replaced with a dropout layer (p=0.3) followed by a linear layer sized to the target number of classes.

| Model | Dataset | Classes | Task |
|---|---|---|---|
| ResNet-18 (modified) | GTZAN | 10 music genres | Music genre classification |
| ResNet-18 (modified) | ESC-50 | 50 sound categories | Environmental sound classification |

**Music model (GTZAN)**
- 10 output classes: blues, classical, country, disco, hiphop, jazz, metal, pop, reggae, rock
- Input: 30-second audio clip at 22050 Hz, converted to a 64-band mel spectrogram
- Confidence threshold for a recognised result: 0.25
- Training data: GTZAN genre dataset, 30 epochs, batch size 32, Adam optimizer

**Nature model (ESC-50)**
- 50 output classes covering environmental sounds such as rain, dog bark, engine, chainsaw, and so on
- Input: 5-second audio clip at 22050 Hz, converted to a 128-band mel spectrogram
- Confidence threshold for a recognised result: 0.25
- Training data: ESC-50 dataset with SpecAugment (frequency and time masking) for augmentation

---

## 🏗️ Architecture

```
User uploads audio (WAV / MP3 / FLAC / OGG)
              ↓
    React Frontend (Vercel)
              ↓  HTTP POST /api/separate
    Flask Backend (Hugging Face)
              ↓
    Demucs htdemucs — splits song into 4 stems
              ↓
    ResNet-18 — converts each stem to Mel Spectrogram → predicts genre
              ↓
    Returns: labels + confidence scores + spectrograms + audio URLs
              ↑
    React displays interactive stem cards with waveform players
```

The project is split into two independently deployed services:

**Frontend** — React 18, built with Vite, deployed on Vercel. Uses React Router for page navigation, WaveSurfer.js for waveform rendering, Three.js for the animated background, and Axios for API communication.

**Backend** — Flask, served with Gunicorn, containerised with Docker, deployed on Hugging Face. Handles file ingestion, runs Demucs separation as a subprocess, runs inference through the custom PyTorch models, generates spectrogram images with Librosa and Matplotlib, and serves all resulting files as static endpoints.

---

## ⚙️ Tech Stack

| Layer | Technology |
|---|---|
| Frontend framework | React 18 |
| Build tool | Vite |
| Routing | React Router v7 |
| Audio visualisation | WaveSurfer.js |
| 3D background | Three.js |
| HTTP client | Axios |
| Backend framework | Flask |
| WSGI server | Gunicorn |
| Audio separation | Demucs (htdemucs) by Meta Research |
| Deep learning | PyTorch, TorchAudio |
| Audio processing | Librosa |
| Containerisation | Docker |
| Frontend hosting | Vercel |
| Backend hosting | Hugging Face Spaces |

---

## 📁 Project Structure

```
Stem-Seperation/
├── frontend/
│   ├── public/
│   │   ├── music-bg.jpg
│   │   └── nature-bg.jpg
│   └── src/
│       ├── pages/
│       │   ├── Home.jsx               # Landing page with domain selection
│       │   ├── UploadPage.jsx         # File upload, validation, health check
│       │   ├── ProcessingPage.jsx     # Loading state while backend processes
│       │   └── ResultsPage.jsx        # Stem cards and results display
│       ├── components/
│       │   ├── StemCard.jsx           # Individual stem with player and spectrogram
│       │   ├── FileUpload.jsx         # Drag and drop file input
│       │   ├── HealthBanner.jsx       # Backend status indicator
│       │   ├── WaveformPlayer.jsx     # WaveSurfer audio player
│       │   ├── Waveform.jsx           # Upload page preview player
│       │   ├── LiquidBackground.jsx   # Animated Three.js background
│       │   └── DomainToggle.jsx       # Music / Nature / Auto selector
│       ├── hooks/
│       │   └── useHealth.js           # Backend health polling hook
│       ├── services/
│       │   └── api.js                 # All API calls in one place
│       └── App.jsx                    # Root component and routing
│
└── backend/
    ├── Api/
    │   └── app.py                     # Flask routes and request handling
    ├── Core/
    │   ├── inference.py               # Model loading and prediction logic
    │   ├── resnet_model.py            # AudioResNet architecture definition
    │   ├── esc50_dataset.py           # ESC-50 PyTorch dataset class
    │   ├── gtzan_dataset.py           # GTZAN PyTorch dataset class
    │   └── separator.py               # Demucs separation wrapper
    ├── Training/
    │   ├── train_esc50.py             # Training script for the nature model
    │   └── train_gtzan.py             # Training script for the music model
    ├── Models/
    │   ├── esc50_resnet_v1.pth        # Trained nature classification weights
    │   └── gtzan_resnet_v1.pth        # Trained music classification weights
    ├── data/
    │   └── esc50.csv                  # ESC-50 class label metadata
    ├── Dockerfile
    └── requirements.txt
```

---

## 📡 API Reference

Base URL: `https://zen-4011-audio-separation-model.hf.space`

### GET /api/health
Returns the service status and model load state.
```json
{
  "status": "ok",
  "models_loaded": true
}
```

### POST /api/separate
Accepts a multipart form upload and returns stems with predictions and URLs.

| Field | Type | Description |
|---|---|---|
| `audio` | File | WAV, MP3, FLAC, or OGG file |
| `domain` | string | `"music"` or `"nature"` |

Response shape (music mode):
```json
{
  "job_id": "a1b2c3d4",
  "domain_detected": "music",
  "processing_time_seconds": 42.1,
  "stems": [
    {
      "id": "a1b2c3d4_vocals",
      "name": "Vocals",
      "label": "Hip-Hop",
      "confidence": 0.81,
      "recognised": true,
      "top3": [
        { "label": "Hip-Hop", "confidence": 0.81 },
        { "label": "Pop",     "confidence": 0.11 },
        { "label": "Reggae",  "confidence": 0.05 }
      ],
      "audio_url": "/api/stems/a1b2c3d4/vocals.wav",
      "spectrogram_url": "/api/spectrograms/a1b2c3d4_vocals.png"
    }
  ]
}
```

### GET /api/stems/\<job_folder\>/\<stem_name\>
Streams a separated stem WAV file.

### GET /api/spectrograms/\<filename\>
Returns a mel spectrogram PNG image.

### GET /api/audio/\<filename\>
Returns the original uploaded audio file.

### GET /api/download-all/\<job_id\>
Returns a ZIP archive containing all separated stems for a job.

---

## 🛠️ Running Locally

**Prerequisites:** Python 3.11, Node.js 18+, ffmpeg (`brew install ffmpeg` on macOS)

### Backend
```bash
cd backend
pip install -r requirements.txt
gunicorn Api.app:app --bind 0.0.0.0:5001 --timeout 300 --workers 1
```
API available at `http://localhost:5001`

> Model weights at `Models/gtzan_resnet_v1.pth` and `Models/esc50_resnet_v1.pth` must be present.

### Frontend
```bash
cd frontend
npm install
```
Create a `.env` file inside `frontend/`:
```
VITE_API_URL=http://localhost:5001
```
```bash
npm run dev
```
App runs at `http://localhost:3000`

---

## 🏋️ Training the Models

Training scripts are in `backend/Training/`. Datasets must be downloaded separately.

**GTZAN (music model)**

Download GTZAN and place genre folders at `backend/data/gtzan_data/genres_original/`
```bash
cd backend
python Training/train_gtzan.py
```
Saves weights to `Models/gtzan_resnet_v1.pth`

**ESC-50 (nature model)**

Download ESC-50 from [github.com/karolpiczak/ESC-50](https://github.com/karolpiczak/ESC-50) and place audio files at `backend/data/audio/`
```bash
cd backend
python Training/train_esc50.py
```
Saves weights to `Models/esc50_resnet_v1.pth`

---

## 🚀 Deployment

### Backend — Hugging Face Spaces
1. Create a new Space with **Docker** SDK
2. Upload all files from the `backend/` folder
3. Hugging Face builds automatically from the `Dockerfile`
4. Build time: ~15–20 minutes (downloads PyTorch + Demucs weights)
5. Requires at least **2GB RAM** (CPU Basic tier)

### Frontend — Vercel
1. Import the GitHub repo into Vercel
2. Set **Root Directory** to `frontend`
3. Set **Framework Preset** to Vite
4. Add environment variable:
```
VITE_API_URL = https://your-huggingface-space-url.hf.space
```
5. Deploy — the `vercel.json` handles SPA routing automatically

---

## ⚠️ Known Limitations

- Processing time for music separation ranges from **30 to 90 seconds** depending on file length, as Demucs runs on CPU
- Files larger than approximately **50MB** may cause request timeouts
- The nature classifier covers only the **50 categories** present in the ESC-50 dataset — sounds outside this set will be returned as "Unrecognised Sound"
- Temporary files (uploads, separated stems, spectrograms) are stored on the container filesystem and **not persisted** across deployments
- The backend runs on Hugging Face free tier and may take **20–30 seconds to wake up** after inactivity — the health banner handles this automatically

---

## 🙌 Acknowledgements

- [Demucs](https://github.com/facebookresearch/demucs) — audio source separation by Meta Research
- [WaveSurfer.js](https://wavesurfer.xyz) — waveform visualization
- [Three.js](https://threejs.org) — 3D background rendering
- [Hugging Face](https://huggingface.co) — model and backend hosting
- [GTZAN Dataset](http://marsyas.info/downloads/datasets.html) — music genre classification benchmark
- [ESC-50 Dataset](https://github.com/karolpiczak/ESC-50) — environmental sound classification benchmark

---

## 📄 License

MIT License
