# Backend Dockerfile
# 臺北市立復興高級中學 學分補考查詢系統 - 後端服務
# 使用 uv 進行套件管理

FROM python:3.13-slim

# 設定工作目錄
WORKDIR /app

# 設定環境變數
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONPATH=/app \
    UV_SYSTEM_PYTHON=1

# 安裝系統依賴
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 安裝 uv
RUN curl -LsSf https://astral.sh/uv/install.sh | sh
ENV PATH="/root/.local/bin:$PATH"

# 複製 pyproject.toml（用於安裝依賴）
COPY backend/pyproject.toml /app/backend/

# 安裝 Python 依賴
WORKDIR /app/backend
RUN uv pip install -r pyproject.toml

# 複製後端程式碼
WORKDIR /app
COPY backend /app/backend

# 暴露端口
EXPOSE 8000

# 啟動命令
CMD ["uvicorn", "backend.main:app", "--host", "0.0.0.0", "--port", "8000"]
