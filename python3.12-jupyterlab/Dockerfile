FROM python:3.12-slim

# ─── 공통 환경 변수 ──────────────────────────────────────────────────────────
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# ─── 시스템 패키지 ───────────────────────────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl \
    wget \
    git \
    vim \
    nano \
    htop \
    procps \
    tree \
    jq \
    unzip \
    build-essential \
    gcc \
    g++ \
    make \
    pkg-config \
    libssl-dev \
    libffi-dev \
    libpq-dev \
    ca-certificates \
    gnupg \
    lsb-release \
    sudo \
    dumb-init \
  && rm -rf /var/lib/apt/lists/*

# ─── PyTorch CPU ─────────────────────────────────────────────────────────────
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
    # === 이미지 처리 ===
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
    # === Notebook / IDE ===
    jupyterlab \
    ipykernel \
    nbformat \
    ipywidgets \
    jupyterlab-git \
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
    # === 모니터링 ===
    prometheus-client \
    \
    # === Testing / Code Quality ===
    pytest \
    pytest-asyncio \
    pytest-cov \
    black \
    isort \
    flake8 \
    mypy \
    pre-commit

# ─── 설치 경로 검증 ──────────────────────────────────────────────────────────
# pip install 후 /usr/local/bin/jupyter 에 생성됨
# python:3.12-slim 은 /usr/local/bin 이 기본 PATH에 포함되어 있어
# code-server처럼 별도 심볼릭 링크가 불필요하지만, 아래에서 확인 후 고정
RUN JUPYTER_BIN="$(which jupyter)" \
  && echo ">>> jupyter binary  : $JUPYTER_BIN" \
  && echo ">>> jupyter version : $(jupyter --version)" \
  && echo ">>> jupyterlab      : $(jupyter lab --version)" \
  # 절대 경로 심볼릭 링크 고정 (/usr/local/bin/jupyter 가 이미 존재해도 덮어씀)
  && ln -sf "$JUPYTER_BIN" /usr/local/bin/jupyter \
  && echo ">>> symlink OK      : $(ls -la /usr/local/bin/jupyter)"

# ─── ipykernel 등록 ──────────────────────────────────────────────────────────
RUN python -m ipykernel install --sys-prefix --name python3 --display-name "Python 3.12"

# ─── JupyterLab 설정 파일 생성 ───────────────────────────────────────────────
# JUPYTER_TOKEN 환경 변수 우선순위:
#   k8s Secret → env var JUPYTER_TOKEN → Jupyter 자동 읽기
#   config에 token을 명시하지 않아야 env var가 정상 동작함
RUN jupyter lab --generate-config \
  && cat >> /root/.jupyter/jupyter_lab_config.py << 'EOF'
c.ServerApp.ip            = '0.0.0.0'
c.ServerApp.port          = 8888
c.ServerApp.open_browser  = False
c.ServerApp.allow_root    = True
c.ServerApp.root_dir      = '/workspace'
# token은 여기서 설정하지 않음 → JUPYTER_TOKEN 환경 변수로 주입 (k8s Secret)
# password 기반 인증 비활성화 (token 방식만 사용)
c.ServerApp.password      = ''
EOF

# ─── 작업 디렉토리 ───────────────────────────────────────────────────────────
RUN mkdir -p /workspace

WORKDIR /workspace

# ─── 포트 ───────────────────────────────────────────────────────────────────
EXPOSE 8888

# ─── 실행 ────────────────────────────────────────────────────────────────────
# dumb-init: PID 1 시그널 안전 처리
# 절대 경로: PATH 의존성 완전 제거
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/usr/local/bin/jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--no-browser", "--allow-root"]
