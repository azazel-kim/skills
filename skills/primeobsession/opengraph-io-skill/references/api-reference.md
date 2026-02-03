# API Reference

Quick reference for OpenGraph.io API endpoints.

## Base URL

```
https://opengraph.io/api/1.1/
```

All requests require `app_id` as a query parameter.

## Endpoints

### Site (Unfurl)

Extract OpenGraph data from a URL.

```
GET /site/{encoded_url}?app_id=XXX
```

**Response:**
```json
{
  "hybridGraph": {
    "title": "Page Title",
    "description": "Page description",
    "image": "https://...",
    "url": "https://...",
    "type": "website"
  },
  "openGraph": { ... },
  "htmlInferred": { ... }
}
```

### Scrape

Fetch HTML content with JavaScript rendering.

```
GET /scrape/{encoded_url}?app_id=XXX
```

**Options:**
- `accept_lang=auto` — Auto-detect language
- `use_proxy=true` — Route through residential proxy

### Screenshot

Capture webpage screenshot.

```
GET /screenshot/{encoded_url}?app_id=XXX
```

**Options:**
- `dimensions=sm|md|lg|xl` — Image size
- `quality=1-100` — JPEG quality
- `full_page=true` — Full page vs viewport

**Response:**
```json
{
  "screenshotUrl": "https://..."
}
```

### Extract

Extract specific HTML elements.

```
GET /extract/{encoded_url}?app_id=XXX&html_elements=h1,h2,p
```

**Response:**
```json
{
  "tags": [
    {"tag": "h1", "innerText": "...", "position": 0},
    {"tag": "h2", "innerText": "...", "position": 1}
  ]
}
```

### Query

Ask AI questions about a webpage.

```
POST /query/{encoded_url}?app_id=XXX
Content-Type: application/json

{
  "query": "What services does this company offer?",
  "responseStructure": { ... }  // optional JSON schema
}
```

**Response:**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "...",
  ...
}
```

## Image Generation API

### Base URL
```
https://opengraph.io/image-agent/
```

### Create Session

```
POST /sessions?app_id=XXX
Content-Type: application/json

{"name": "my-session"}
```

### Generate Image

```
POST /sessions/{sessionId}/generate?app_id=XXX
Content-Type: application/json

{
  "prompt": "A blue cloud icon",
  "kind": "icon",
  "quality": "medium",
  "stylePreset": "github-dark"
}
```

**Response:**
```json
{
  "sessionId": "...",
  "assetId": "...",
  "status": "succeeded",
  "url": "/assets/{assetId}/file",
  "width": 512,
  "height": 512,
  "usage": {
    "totalCostUsd": 0.05
  }
}
```

### Iterate Image

```
POST /sessions/{sessionId}/iterate?app_id=XXX
Content-Type: application/json

{
  "assetId": "...",
  "prompt": "Change the color to orange"
}
```

### Get Session

```
GET /sessions/{sessionId}?app_id=XXX
```

### Get Asset File

```
GET /assets/{assetId}/file?app_id=XXX
```

Returns the image binary (PNG/JPEG).

## Error Codes

| Code | Meaning |
|------|---------|
| -9 | API key required |
| 101 | Invalid API key |
| 102 | Rate limit exceeded |
| 103 | Invalid URL |
| 104 | Site unreachable |
| 105 | Timeout |

## Rate Limits

Limits depend on your plan. Check headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1706547200
```
