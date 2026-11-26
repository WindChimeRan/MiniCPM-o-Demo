# Apple Silicon Backend (vLLM-Metal)

On a Mac, vLLM-Metal serves MiniCPM-o 4.5 through vLLM-Omni, and the desktop
Audio Full-Duplex and Omnimodal Full-Duplex pages connect to it. The model
server and the gateway each run in their own terminal.

## Terminal 1: model server

Install full Xcode and select it, as in vLLM-Metal's
[development setup](https://github.com/vllm-project/vllm-metal/blob/v0.34.0/docs/CONTRIBUTING.md#development-setup).
Then install vLLM-Metal v0.34.0 with vLLM-Omni and ONNX Runtime, and cache the
Silero model that server VAD uses for voice interruption:

```bash
git clone --branch v0.34.0 https://github.com/vllm-project/vllm-metal.git
cd vllm-metal
./install.sh
source .venv-vllm-metal/bin/activate
VLLM_OMNI_TARGET_DEVICE=cpu uv pip install -e '.[omni]' onnxruntime
hf download istupakov/silero-vad-onnx silero_vad.onnx \
  --revision 8b14476858ef240c50b3884bb38cc67290c1cc70
```

Start the server in the same directory and environment:

```bash
VLLM_METAL_BACKEND=mps PYTORCH_ENABLE_MPS_FALLBACK=0 vllm-omni serve \
  openbmb/MiniCPM-o-4_5 --omni \
  --deploy-config vllm_metal/pytorch_backend/deploy/minicpmo_4_5_mps.yaml \
  --host 127.0.0.1 --port 8097
```

It is ready when it prints `Application startup complete`, a few minutes
later. The `trust_remote_code` errors it logs at startup are harmless.

## Terminal 2: gateway

In an environment with the gateway dependencies installed (see
[docker/Dockerfile.gateway](../../docker/Dockerfile.gateway)), run from this repository:

```bash
python gateway.py --host 127.0.0.1 --port 8006 --http \
  --vllm-omni-url ws://127.0.0.1:8097/v1/realtime
```

Open http://127.0.0.1:8006/audio_duplex for voice or
http://127.0.0.1:8006/omni for camera and voice. The browser connects directly
to the given WebSocket URL; use a browser-accessible `wss://` URL when serving
the page over HTTPS.

Independent TTS reference audio and server-side save/share are unavailable.
Turn-based and mobile clients use the provider backend.
