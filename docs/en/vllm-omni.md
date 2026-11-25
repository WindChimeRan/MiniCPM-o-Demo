# vLLM-Omni backend

The desktop Audio Full-Duplex and Omni Full-Duplex pages can connect to an
existing MiniCPM-o 4.5 vLLM-Omni server. On Apple Silicon, serve it with
vLLM-Metal v0.34.0: follow the Setup and MiniCPM-o 4.5 sections of its
[Omni guide](https://github.com/vllm-project/vllm-metal/blob/v0.34.0/docs/omni.md).

Voice interruption uses Omni's server VAD. Install `onnxruntime` in the serving
environment and cache its Silero model before starting the server:

```bash
hf download istupakov/silero-vad-onnx silero_vad.onnx \
  --revision 8b14476858ef240c50b3884bb38cc67290c1cc70
```

In an environment with the gateway dependencies installed (see
[docker/Dockerfile.gateway](../../docker/Dockerfile.gateway)), run from this repository:

```bash
python gateway.py --host 127.0.0.1 --port 8006 --http \
  --vllm-omni-url ws://127.0.0.1:8097/v1/realtime
```

Open http://127.0.0.1:8006/audio_duplex for voice or
http://127.0.0.1:8006/omni for camera and voice. The browser connects directly
to the given WebSocket URL; use a browser-accessible `wss://` URL when serving
the page over HTTPS. The Omni server manages models and session capacity.

Independent TTS reference audio and server-side save/share are unavailable.
Turn-based and mobile clients use the provider backend.
