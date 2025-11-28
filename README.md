# On-Device Low-Latency Dictation Engine

FastAPI-based realtime speech dictation pipeline fully on-device. No LLMs. Modules: ASR (Vosk), fillers, repetitions, grammar (LanguageTool via JPype), tone transform (T5-small ONNX), auto-format. Real-time WebSocket streaming with VAD and pause detection; all stages run in-memory.

## Quick start (Docker)
1. Prepare models on host:
   - Vosk model: `./models/vosk-en/` (download e.g. `vosk-model-small-en-us-0.15` and rename folder)
   - LanguageTool jar: `./models/LanguageTool/languagetool.jar`
   - T5-small ONNX folder: `./models/t5-small-onnx/` containing `encoder.onnx`, `decoder_init.onnx`, `decoder_step.onnx`, `spiece.model`
2. Run:
```
docker compose up --build
```
- Core realtime API: http://localhost:8080
- UI Gateway: http://localhost:3000

## Run locally (Windows, without Docker)
- Install Python 3.11 and Java 11+ (for LanguageTool).
- In `services/core_realtime`:
```
pip install -r requirements.txt
set VOSK_MODEL=./models/vosk-en
set LT_JAR=./models/LanguageTool/languagetool.jar
set T5_ONNX_DIR=./models/t5-small-onnx
uvicorn app.main:app --host 0.0.0.0 --port 8080
```
- Serve the UI gateway:
```
cd services/ui_gateway
npm install
npm start
```

## Key endpoints
- WebSocket: `ws://localhost:8080/asr/stream?lang=en&tone=neutral`
- REST: `/process/full`, `/process/fillers`, `/process/repetition`, `/process/grammar`, `/process/tone`
- Models: `/models/list`, `/models/reload`
- Metrics: `/metrics`

## Notes
- All fast-path processing is in one process; no internal REST.
- If ONNX or LanguageTool models are missing, tone/grammar steps gracefully no-op.
- To use GPU on Windows, install `onnxruntime-directml` instead of `onnxruntime`.
