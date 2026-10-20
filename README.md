# Deploy and Host VoiceStudio on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/voicestudio?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=voicestudio)

[VoiceStudio](https://github.com/debpalash/VoiceStudio) is an open-source alternative to ElevenLabs. It clones a voice from a short sample, designs new voices from a description, generates speech in hundreds of languages, dubs videos, transcribes audio and builds audiobooks. This template runs the official VoiceStudio server image, so you get the full Studio web UI in your browser plus an OpenAI-compatible speech API. The models run inside the container, so your text and voice samples never go to a third-party API.

## About Hosting VoiceStudio

It's one service and a Railway volume.

- **Locked from the first boot.** A random admin key (`OMNIVOICE_API_KEY`) is generated at deploy time. The web UI asks for it once and swaps it for a short-lived browser session; API clients send it as a Bearer token. Without it, everything except the page shell is refused.
- **HTTPS out of the box.** Railway terminates TLS, so the key and session cookie never travel in plain text.
- **Models and voices survive redeploys.** The project database, your cloned voices, settings and the Hugging Face model cache all live on the volume at `/app/omnivoice_data`.
- **CPU inference.** Railway has no GPUs, so VoiceStudio runs on CPU. Short clips take seconds; long audiobooks and video dubbing are slow. Pick a plan with enough RAM (see below).

## Common Use Cases

- Voice-overs and narration for videos, courses and podcasts without per-character pricing
- Cloning your own voice (or a consenting speaker's) for consistent narration across projects
- A private, OpenAI-compatible text-to-speech endpoint for your own apps and agents

## Dependencies for VoiceStudio Hosting

- Nothing else: models download from Hugging Face on first use and are cached on the volume
- Optional: a Hugging Face token (`HF_TOKEN`) for gated models

### Deployment Dependencies

- [VoiceStudio on GitHub](https://github.com/debpalash/VoiceStudio)
- [Docker install guide](https://github.com/debpalash/VoiceStudio/blob/main/docs/install/docker.md)
- [API authentication](https://github.com/debpalash/VoiceStudio/blob/main/docs/api-auth.md)
- [Template source on GitHub](https://github.com/two-workspaces/voicestudio)

### Implementation Details

**Sign in.** Open the service's Railway domain. When the UI asks for a key, paste `OMNIVOICE_API_KEY` from the Variables tab. The browser keeps only a short-lived session, not the key.

**First run.** The first model you use downloads its weights (a few GB) into the volume, so the first generation takes a few minutes. After that, models load from the volume.

**Use the API.** Send the key as a Bearer token:

```bash
curl https://<your-app>.up.railway.app/v1/audio/speech \
  -H "Authorization: Bearer <OMNIVOICE_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"input": "Hello from Railway", "voice": "default"}' -o hello.wav
```

Apps in the same Railway project can call `http://${{VoiceStudio.RAILWAY_PRIVATE_DOMAIN}}:3900` instead, with the same key. That traffic isn't billed as egress.

**Resources.** The image is about 6 GB (it ships PyTorch), so it needs the Hobby plan or higher. Speech models use roughly 2–4 GB of RAM while loaded. Railway volumes start at 5 GB on Hobby; if you install several large models, grow the volume (Pro) or remove models you don't use from the Model Catalogue.

**Updates.** The template pins a VoiceStudio release. To upgrade, change the image tag on the service to a newer `ghcr.io/debpalash/voicestudio:<version>` and redeploy; your data stays on the volume.

**Telemetry.** Product analytics are opt-in and off unless you accept the first-run prompt. Set `OMNIVOICE_ANALYTICS_DISABLED=1` to remove the prompt entirely.

**Use voice cloning responsibly.** Only clone voices you own or have explicit permission to use.

## Why Deploy VoiceStudio on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying VoiceStudio on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
