---
name: Render a single video with the legacy Wideo replace/encode flow
description: Substitute values into a Wideo template with the legacy /automation/replace call, then encode the result into a finished MP4 with /automation/encode.
api: openapi/wideo-automation-api-openapi.yml
operations:
- replaceVariables
- encodeVideo
---

# Render a single video with the legacy Wideo replace/encode flow

Wideo's original Video Automation flow is two calls against
`https://automationapi.wideo.co`. Use it when you need one video from one set of
values. For many videos from one template, use the batch flow instead — see
`wideo-batch-video-generation.md`.

## Before you start

- API access is not self-serve. Keys are issued through the "Request API access"
  form on <https://wideo.co/api-documentation/>.
- Templates are authored inside Wideo, not through the API. You work with Wideo to
  create the template and to define which content areas are variable; the API only
  references the template by `TEMPLATE_ID`.
- Send your key in the `x-api-key` header (UUID form). Wideo's published legacy
  code samples show only `Content-Type: application/json`, but the host is an AWS
  API Gateway that answers unauthenticated requests with
  `403 {"message":"Missing Authentication Token"}` — so send the key.

## Steps

1. **Substitute the values** — `replaceVariables` (POST `/automation/replace`).
   - Body is a single `data` object whose keys are the variable names defined in
     your template:
     ```json
     { "data": { "name": "1", "desc": "description", "image": "https://yourdomain.co/1.png" } }
     ```
   - The response carries the replace object id:
     ```json
     { "statusCode": "200", "body": { "id": "REPLACE_OBJECT_ID" } }
     ```
   - Keep that `id`. It is the only handle to the substituted content.

2. **Encode the video** — `encodeVideo` (POST `/automation/encode`).
   - Body pairs the template with the replace object. `clientId` is a free-text
     label Wideo uses for tracking only:
     ```json
     { "clientId": "YourName",
       "videos": [ { "templateId": "TEMPLATE_ID", "replaceId": "REPLACE_OBJECT_ID" } ] }
     ```
   - Read the finished MP4 URL from the response. Wideo's published documentation
     is inconsistent about where it sits: the documented response body shows `url`
     at the top level, while the accompanying JavaScript sample reads
     `response.videos[0].url`. Handle both.

## Conventions and cautions

- Everything is `application/json`.
- **No idempotency key exists.** Re-running `encodeVideo` with the same
  `replaceId` renders and bills a second video. Persist the returned URL against
  your own job id before retrying anything.
- Errors are a bare `{"message": "..."}` envelope from AWS API Gateway; the real
  failure class is in the `x-amzn-errortype` response header. There is no error
  reference — see `errors/wideo-problem-types.yml`.
- Consumption is billed in credits, where one credit is 60 seconds of generated
  video, against a monthly allowance — see `plans/wideo-plans-pricing.yml`.
- This is the older of two live generations. Wideo publishes no deprecation policy
  and no statement about how long `/automation/*` will be supported, so treat its
  lifetime as unknown and prefer `/batch` for new work.
