FROM python:3.12-slim

# ─── 공통 환경 변수 ──────────────────────────────────────────────────────────
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# ─── 시스템 패키지 ───────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
    # 네트워크 / 다운로드
    curl \
    wget \
    git \
    # 에디터 / 유틸
    vim \
    nano \
    htop \
    procps \
    tree \
    jq \
    unzip \
    # 빌드 도구 (psycopg2, cryptography 등 C 확장 필요)
    build-essential \
    gcc \
    g++ \
    make \
    pkg-config \
    # SSL / 암호화
    libssl-dev \
    libffi-dev \
    # PostgreSQL 클라이언트 헤더
    libpq-dev \
    # 인증서 / OS
    ca-certificates \
    gnupg \
    lsb-release \
    sudo \
    # PID 1 안전 처리 (SIGTERM/SIGCHLD 정상 전달)
    dumb-init \
  && rm -rf /var/lib/apt/lists/*

# ─── code-server 설치 ────────────────────────────────────────────────────────
# 공식 install.sh 사용 → Debian 계열:
#   실제 바이너리: /usr/lib/code-server/bin/code-server
#   시스템 심볼릭: /usr/bin/code-server
#
# [이전 RunContainerError 원인]
#   CMD에서 "code-server" 이름만 사용 시 컨테이너 런타임 PATH와 불일치 발생
#   → 절대 경로(/usr/local/bin/code-server)로 추가 심볼릭 링크 고정
RUN curl -fsSL https://code-server.dev/install.sh | sh \
  # 절대 경로 심볼릭 링크 추가 (PATH 의존성 완전 제거)
  && ln -sf "$(which code-server)" /usr/local/bin/code-server \
  # 설치 검증 — 이 줄에서 실패하면 빌드 단계에서 즉시 확인 가능
  && echo ">>> code-server binary: $(which code-server)" \
  && echo ">>> symlink: $(ls -la /usr/local/bin/code-server)" \
  && /usr/local/bin/code-server --version

# ─── PyTorch CPU 빌드 ────────────────────────────────────────────────────────
# GPU 없는 환경 → whl/cpu 인덱스 사용 (CUDA 드라이버 없이 이미지 크기 절감)
RUN pip install --no-cache-dir \
    torch \
    torchvision \
    torchaudio \
    --index-url https://download.pytorch.org/whl/cpu

# ─── Python 라이브러리 ───────────────────────────────────────────────────────
RUN pip install --no-cache-dir \
    \
    # === Web Framework ===
    fastapi \
    "uvicorn[standard]" \
    pydantic \
    pydantic-settings \
    python-multipart \
    starlette \
    \
    # === HTTP Client ===
    httpx \
    requests \
    aiohttp \
    aiofiles \
    anyio \
    \
    # === 직렬화 / 유틸 ===
    orjson \
    pyyaml \
    msgpack \
    \
    # === Data / 시각화 ===
    gradio \
    numpy \
    pandas \
    matplotlib \
    scipy \
    seaborn \
    plotly \
    pillow \
    \
    # === ML / AutoML ===
    scikit-learn \
    xgboost \
    lightgbm \
    optuna \
    joblib \
    imbalanced-learn \
    \
    # === 이미지 처리 (headless: GUI 디스플레이 불필요) ===
    opencv-python-headless \
    \
    # === NLP / Transformers ===
    transformers \
    datasets \
    accelerate \
    sentence-transformers \
    tokenizers \
    einops \
    safetensors \
    \
    # === Notebook / IDE 보조 ===
    jupyterlab \
    ipykernel \
    nbformat \
    ipywidgets \
    \
    # === Database / Cache ===
    sqlalchemy \
    alembic \
    psycopg2-binary \
    pymongo \
    motor \
    redis \
    \
    # === Task Queue ===
    celery \
    \
    # === 환경 설정 / 로깅 ===
    python-dotenv \
    loguru \
    rich \
    typer \
    tqdm \
    \
    # === 재시도 / 안정성 ===
    tenacity \
    \
    # === 날짜 / 시간 ===
    pendulum \
    arrow \
    \
    # === 모니터링 (추후 Prometheus 연동 대비) ===
    prometheus-client \
    \
    # === Testing / Code Quality ===
    pytest \
    pytest-asyncio \
    pytest-cov \
    httpx \
    black \
    isort \
    flake8 \
    mypy \
    pre-commit

# ─── 디렉토리 & code-server 기본 설정 ───────────────────────────────────────
RUN mkdir -p /root/.config/code-server /workspace

# 기본 config.yaml
# ※ PASSWORD 환경 변수가 주입되면 이 파일의 password 값보다 우선 적용됨
RUN cat > /root/.config/code-server/config.yaml << 'EOF'
bind-addr: 0.0.0.0:8080
auth: password
password: changeme_via_k8s_secret
cert: false
EOF

WORKDIR /workspace

# ─── 포트 ───────────────────────────────────────────────────────────────────
EXPOSE 8080

# ─── 실행 ────────────────────────────────────────────────────────────────────
# dumb-init: 컨테이너 PID 1로 실행 → SIGTERM 정상 전달 및 좀비 프로세스 방지
# 절대 경로 사용 → RunContainerError(executable not found in PATH) 방지
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/usr/local/bin/code-server", "--bind-addr", "0.0.0.0:8080", "--user-data-dir", "/workspace/.vscode-data", "--extensions-dir", "/workspace/.vscode-extensions", "/workspace"]
