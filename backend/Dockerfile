FROM python:3.11-slim

RUN apt-get update && apt-get install -y \
    ffmpeg \
    libsndfile1 \
    git \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir "numpy<2"

RUN pip install --no-cache-dir \
    torch==2.2.2 \
    torchaudio==2.2.2 \
    torchvision==0.17.2 \
    --index-url https://download.pytorch.org/whl/cpu

RUN pip install --no-cache-dir flask flask-cors werkzeug demucs \
    pandas librosa soundfile scipy scikit-learn matplotlib \
    Pillow requests tqdm omegaconf einops julius PyYAML gunicorn

RUN python -c "from demucs.pretrained import get_model; get_model('htdemucs')"

ARG CACHE_BUST=6
RUN echo "Cache bust: $CACHE_BUST"

COPY . .

EXPOSE $PORT

CMD ["sh", "-c", "PYTHONUNBUFFERED=1 gunicorn Api.app:app --bind 0.0.0.0:${PORT} --timeout 300 --workers 1 --log-level debug --capture-output"]