FROM python:3.11-slim

RUN apt-get update && apt-get install -y \
    build-essential \
    curl \
    git \
    libssl-dev \
    libffi-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace

RUN pip install --upgrade pip && pip install \
    ipykernel \
    fastapi==0.104.1 \
    uvicorn[standard]==0.24.0 \
    gunicorn==21.2.0 \
    pydantic[email]==2.5.0 \
    sqlalchemy==2.0.23 \
    alembic==1.13.0 \
    asyncpg==0.29.0 \
    redis==5.0.1 \
    python-jose[cryptography]==3.3.0 \
    passlib[bcrypt]==1.7.4 \
    python-multipart==0.0.6 \
    httpx==0.25.2 \
    structlog==23.2.0 \
    pytest==7.4.3 \
    pytest-asyncio==0.21.1 \
    pytest-cov==4.1.0 \
    black==23.11.0 \
    isort==5.12.0 \
    flake8==6.1.0 \
    mypy==1.7.1 \
    pre-commit==3.6.0 \
    sentry-sdk[fastapi]==1.38.0 \
    prometheus-client==0.19.0

RUN python -m ipykernel install --name py311 --display-name "Python 3.11"

CMD ["/bin/bash"]
