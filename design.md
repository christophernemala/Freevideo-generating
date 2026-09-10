# FreeVideo Studio — Design and Deployment Architecture

## Goal
Turn the existing `video-use` editing skill into a deployable web application while preserving its open-source editing workflow.

The hosted application must not pretend that GPU inference is free. The web application can run on a free hosting tier, while generation is routed through a configurable provider or a self-hosted GPU worker.

## Product flows

### Image to video
1. Upload a JPG/PNG/WebP start image.
2. Enter a motion prompt.
3. Select duration/aspect ratio/provider.
4. Submit a generation job.
5. Poll job status.
6. Preview/download the resulting MP4.

### Text to video
1. Enter a scene prompt.
2. Select duration/aspect ratio/provider.
3. Submit and monitor the job.
4. Preview/download the result.

### Existing video editing
The existing `video-use` skill, FFmpeg helpers, transcription and post-production workflow remain separate from generation. Generated MP4 files can later enter that editing pipeline.

## Architecture

```text
Browser
  |
  v
Next.js-compatible static UI / API client
  |
  v
/api/generate  ---> provider adapter ---> external API OR self-hosted GPU
/api/jobs/:id <--- normalized status <---- provider
  |
  v
MP4 result URL
```

## Provider contract
Every provider adapter exposes the same conceptual interface:

- `createJob({ prompt, image, duration, aspectRatio })`
- `getJob(id)`
- normalized states: `queued | processing | succeeded | failed`
- `outputUrl` only for successful jobs

Provider credentials are server-only. Never expose provider secrets to browser JavaScript.

## Deployment

### Frontend/API
Target: Vercel-compatible deployment.

### Generation compute
Vercel is orchestration/UI hosting, not the GPU inference runtime. Generation should use one of:
- configurable external video API,
- self-hosted GPU endpoint,
- local development/mock provider.

The repository must work without a paid provider in `mock` mode so UI and orchestration can be tested free of charge.

## Security
- Provider keys remain server-side.
- Validate MIME type, prompt length, duration and aspect ratio.
- Apply request size limits.
- Do not log API keys or raw authorization headers.
- Add rate limiting before public production use.
- Treat provider output URLs as untrusted remote content.
- No secrets committed to Git.

## UI direction
Professional dark video studio rather than a generic demo page. Primary workspace: upload/reference image on the left, prompt and generation controls in the center, job/result panel on the right. Responsive layout collapses to one column on mobile.

## Treasury Atom use case
The application should support the Treasury Atom workflow: upload a dashboard still and instruct the model to animate only the Atom robot and subtle UI micro-motion while preserving dashboard geometry and typography.

## Definition of done for hosted MVP
- Deployable web UI.
- Image upload and prompt form.
- Provider abstraction.
- Mock generation path for free testing.
- Secure environment-variable configuration for real generation.
- Job status and errors visible in UI.
- README deployment instructions.
- Existing `video-use` editing assets preserved.
