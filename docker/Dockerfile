# ベースイメージ
FROM nvidia/cuda:11.8.0-devel-ubuntu20.04

# 環境変数設定
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH \
    BNB_CUDA_VERSION=118 \
    NVIDIA_VISIBLE_DEVICES=all \
    NVIDIA_DRIVER_CAPABILITIES=graphics,utility,compute

# Python 3.10のインストール (最適化版)
RUN apt-get update && apt-get install -y \
    software-properties-common \
    build-essential \
    git \
    curl \
    wget \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get update \
    && apt-get install -y \
    python3.10 \
    python3.10-dev \
    python3.10-distutils \
    python3.10-venv \
    python3-pip \
    python3-apt \
    ffmpeg \
    libsm6 \
    libxext6 \
    protobuf-compiler \
    libprotobuf-dev \
    && curl -sS https://bootstrap.pypa.io/get-pip.py | python3.10 \
    && pip install --no-cache-dir protobuf \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Python設定
RUN ln -sf /usr/bin/python3.10 /usr/bin/python \
    && ln -sf /usr/bin/python3.10 /usr/bin/python3

# OpenCvの設定
RUN apt-get update && apt-get install -y \
    libopencv-dev \
    python3-opencv \
    && pip install --no-cache-dir \
    opencv-python-headless==4.9.0.80 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
    
# 作業ディレクトリ設定
WORKDIR /app

# 依存関係のインストール (キャッシュを活用するため先にコピー)
COPY requirements.txt .
COPY requirements-torch.txt .

# PyTorchと主要依存関係のインストール
RUN pip install --no-cache-dir --upgrade pip setuptools ninja packaging \
    && pip install --no-cache-dir \
    torch==2.1.2 \
    torchvision==0.16.2 \
    torchaudio==2.1.2 \
    --index-url https://download.pytorch.org/whl/cu118

# Flash Attentionのインストール (Python 3.10用)
RUN wget -q https://github.com/Dao-AILab/flash-attention/releases/download/v2.4.0/flash_attn-2.4.0+cu118torch2.1cxx11abiFALSE-cp310-cp310-linux_x86_64.whl \
    && pip install --no-cache-dir flash_attn-2.4.0+cu118torch2.1cxx11abiFALSE-cp310-cp310-linux_x86_64.whl \
    && rm flash_attn-2.4.0+cu118torch2.1cxx11abiFALSE-cp310-cp310-linux_x86_64.whl

# その他の依存関係
RUN pip install --no-cache-dir \
    transformers==4.36.0 \
    peft==0.6.0 \
    accelerate==0.21.0 \
    deepspeed \
    wandb \
    tensorboard

# アプリケーションコードのコピー
COPY . .

# LLaVAのインストール
WORKDIR /app/LLaVA
RUN pip install --no-cache-dir .

# メイン作業ディレクトリに戻る
WORKDIR /app

# 不要ファイルの削除
RUN find /app -type f -name '*.pyc' -delete \
    && find /app -type d -name '__pycache__' -exec rm -rf {} +

# シェル設定
RUN echo 'PS1="(DLV) \[\]\[\e]0;\u@\h: \w\a\]${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ \[\]"' >> /root/.bashrc

# デフォルトコマンド
CMD ["bash"]