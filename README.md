# Deploy and Host VoiceStudio on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/new/template/voicestudio?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=voicestudio)

[VoiceStudio](https://github.com/debpalash/VoiceStudio) is an open-source alternative to ElevenLabs. It clones a voice from a short sample, designs new voices from a description, generates speech in hundreds of languages, dubs videos, transcribes audio and builds audiobooks. This template runs the official VoiceStudio server image, so you get the full Studio web UI in your browser plus an OpenAI-compatible speech API. The models run inside the container, so your text and voice samples never go to a third-party API.

## About Hosting VoiceStudio

It's one service and a Railway volume.

- **Locked from the first boot.** A random admin key (`OMNIVOICE_API_KEY`) is generated at deploy time. The web UI asks for it once and swaps it for a short-lived browser session; API clients send it as a Bearer token. Without it, everything except the page shell is refused.
- **HTTPS out of the box.** Railway terminates TLS, so the key and session cookie never travel in plain text.
- **Models and voices survive redeploys.** The project database, your cloned voices, settings and the Hugging Face model cache all live on the volume at `/app/omnivoice_data`.
- **CPU inference, tuned.** Railway has no GPUs, so VoiceStudio runs on CPU. The template caps PyTorch at your plan's vCPUs (`OMP_NUM_THREADS`, default 8). Without that cap PyTorch sees the host's 48 cores, oversubscribes the 8 it's allowed, and a single sentence took over 15 minutes in our tests; with it, a short sentence takes about 20 seconds.

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

**First run.** The first generation downloads the default model (about 3.4 GB) into the volume and loads it, so it takes a minute or two. After that, a 4-second clip takes about 20 seconds on the Hobby plan's 8 vCPUs. Long texts, audiobooks and video dubbing take proportionally longer, so run them from the web UI rather than through a single HTTP request (Railway closes HTTP requests after 5 minutes).

**Use the API.** Send the key as a Bearer token:

```bash
curl https://<your-app>.up.railway.app/v1/audio/speech \
  -H "Authorization: Bearer <OMNIVOICE_API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"model": "tts-1", "input": "Hello from Railway", "voice": "alloy", "response_format": "wav"}' -o hello.wav
```

Apps in the same Railway project can call `http://${{VoiceStudio.RAILWAY_PRIVATE_DOMAIN}}:3900` instead, with the same key. That traffic isn't billed as egress.

**Resources and cost.** The image is about 6 GB (it ships PyTorch), so it needs the Hobby plan or higher. Before the first generation the service uses about 2 GB of RAM. Once a model is loaded, Railway meters about 6 GB: roughly 2.3 GB for the process plus 3.7 GB of model files the kernel keeps cached. At Railway's memory pricing that's around $60 a month if left running, so remove the service (or scale it down) when you're not using it. The default model takes about 3.4 GB of the 5 GB Hobby volume; grow the volume (Pro) or remove models you don't use from the Model Catalogue before installing more.

**Updates.** The template pins a VoiceStudio release. To upgrade, change the image tag on the service to a newer `ghcr.io/debpalash/voicestudio:<version>` and redeploy; your data stays on the volume.

**Telemetry.** Product analytics are opt-in and off unless you accept the first-run prompt. Set `OMNIVOICE_ANALYTICS_DISABLED=1` to remove the prompt entirely.

**Use voice cloning responsibly.** Only clone voices you own or have explicit permission to use.

## Why Deploy VoiceStudio on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying VoiceStudio on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
