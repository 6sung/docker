FROM python:3.12-slim

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH="/root/.local/bin:${PATH}"

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
  "--auth", "password", \
  "/workspace"]
