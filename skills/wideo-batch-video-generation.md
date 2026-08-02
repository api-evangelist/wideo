---
name: Generate videos in bulk with Wideo
description: Render a batch of personalized MP4 videos from a Wideo template and a list of per-video variable objects, then retrieve the finished signed URLs.
api: openapi/wideo-automation-openapi.yml
operations:
- createBatch
- getBatch
---

# Generate videos in bulk with Wideo

Use the Wideo Video Automation API to render many MP4 videos from one template.

## Authentication
Send your API key on every request in the `x-api-key` header (UUID form). Keys
are scoped to a single account; generated assets are isolated per account.

## Steps

1. **Create the batch** — `createBatch` (POST `/batch`).
   - Body: `templateId` (the Wideo template to render), an optional `webhook`
     URL for the completion callback, and a `videos` array with one object of
     template-variable values per output video.
   - The response returns a `batchId` immediately. Rendering is asynchronous.

2. **Wait for completion.**
   - Preferred: supply a `webhook` URL and handle the `batch.completed`
     callback Wideo POSTs when rendering finishes.
   - Or poll `getBatch` (GET `/batch/{batchId}`) until `status` is `SUCCEEDED`.

3. **Collect the assets** — from the batch status `videos` array, read each
   `videoUrl` (signed MP4) and `previewUrl` (signed preview image).

## Conventions
- All requests and responses are `application/json`.
- The model is async: create returns an id, not the finished video.
- No idempotency-key is documented, so avoid resubmitting the same batch blindly;
  reconcile against the `batchId` you already hold.
- Asset URLs are signed and time-limited — download or re-fetch via `getBatch`
  rather than caching indefinitely.
