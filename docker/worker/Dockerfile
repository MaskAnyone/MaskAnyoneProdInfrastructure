FROM nvidia/cuda:11.7.1-cudnn8-runtime-ubuntu22.04

ARG DEBIAN_FRONTEND=noninteractive
ARG MASK_ANYONE_VERSION

WORKDIR /app

RUN apt-get update \
    && apt-get -y install \
    python3 \
    python3-pip \
    python-is-python3 \
    python3-opencv \
    libgl1 \
    libglib2.0-0 \
    libpq-dev \
    gcc \
    git \
    ffmpeg \
    wget \
    unzip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN pip install \
    mediapipe \
    ultralytics \
    requests \
    torch \
    torchvision \
    torchaudio \
    huggingface_hub \
    gdown \
    ffmpeg-python \
    moviepy

# RVC
COPY scripts/ /scripts/

RUN cd / \
    && git clone https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI.git \
    && cd Retrieval-based-Voice-Conversion-WebUI \
    && git checkout 3e455ece57a627cb29252d7c6de2b86ace9aff85 \
    && pip install -r requirements.txt

RUN python3 /scripts/download_voice_models.py

# STTN
RUN cd / \
    && git clone https://github.com/researchmm/STTN.git \
    && cd STTN \
    && mkdir ./checkpoints \
    && cd ./checkpoints \
    && gdown https://drive.google.com/uc?id=1ZAMV8547wmZylKRt5qR_tC5VlosXD4Wv \
    && sed -i 's/cuda:1/cuda:0/' /STTN/test.py \
    && sed -i 's/mp4v/vp09/' /STTN/test.py

RUN cd / \
    && wget https://github.com/MaskAnyone/MaskAnyone/archive/refs/tags/${MASK_ANYONE_VERSION}.zip \
    && unzip ${MASK_ANYONE_VERSION}.zip -d MaskAnyone \
    && mv MaskAnyone/MaskAnyone-${MASK_ANYONE_VERSION#?}/workers/* /app \
    && rm -rf MaskAnyone

CMD ["python3", "/app/basic_masking_worker.py"]
