FROM python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="/root/.local/bin:${PATH}"

# 시스템 패키지
RUN apt-get update && apt-get install -y \
  curl \
  wget \
  git \ 
  vim \ 
  unzip \ 
  build-essential \ 
  procps \ 
  htop \ 
  net-tools \ 
  && rm -rf /var/lib/apt/lists/* 
  
# uv 설치 RUN curl -LsSf https://astral.sh/uv/install.sh | sh 

# code-server 설치 
RUN curl -fsSL https://code-server.dev/install.sh | sh 

# Python 패키지 
RUN pip install --no-cache-dir --upgrade pip setuptools wheel && \ 
  pip install --no-cache-dir \ 
  fastapi \ 
  "uvicorn[standard]" \ 
  pydantic \ 
  python-multipart \ 
  httpx \ 
  requests \ 
  gradio \ 
  numpy \ 
  pandas \ 
  matplotlib \ 
  scipy \ 
  seaborn \ 
  scikit-learn \ 
  xgboost \ 
  torch \ 
  torchvision \ 
  torchaudio \ 
  transformers \ 
  datasets \ 
  accelerate \ 
  sentence-transformers \ 
  jupyterlab \ 
  ipykernel 

WORKDIR /workspace 
EXPOSE 8080 
CMD ["code-server", \ 
  "--bind-addr", "0.0.0.0:8080", \ 
  "--auth", "password", 
  \ "/workspace"]
