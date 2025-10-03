# API Documentation

## Docs generated from api server code

## Overview

This API provides image and video generation capabilities using Google's AI models. The service supports various operations including image generation, image transformation, prompt extraction, and video generation.

## Authentication

All requests require an API key, which can be provided in two ways:

**Option 1: Header (Recommended)**

```text
X-API-Key: your_api_key_here
```

**Option 2: Query Parameter**

```text
?api_key=your_api_key_here
```

## Base URL

```text
https://app.recrafter.fun
```

## Available Endpoints

### 1. List Providers

Get information about available providers and their capabilities.

**Endpoint:** `GET /api/v1/providers`

**Response:**

```json
{
  "providers": {
    "google_fx": {
      "images": ["generate", "transform", "extract_prompt"],
      "videos": ["generate"]
    }
  }
}
```

---

### 2. Image Generation

Generate images from text prompts or transform existing images.

**Endpoint:** `POST /api/v1/images`

#### Operation: Generate Image

Create a new image from a text prompt.

**Request:**

```json
{
  "provider": "google_fx",
  "operation": "generate",
  "parameters": {
    "prompt": "A serene mountain landscape at sunset",
    "aspect_ratio": "IMAGE_ASPECT_RATIO_LANDSCAPE",
    "seed": 12345
  }
}
```

**Parameters:**

- `prompt` (required): Text description of the image you want to generate
- `aspect_ratio` (optional): One of:
  - `IMAGE_ASPECT_RATIO_LANDSCAPE` (default)
  - `IMAGE_ASPECT_RATIO_PORTRAIT`
  - `IMAGE_ASPECT_RATIO_SQUARE`
- `seed` (optional): Integer for reproducible results

**Response:**

```json
{
  "success": true,
  "provider": "google_fx",
  "operation": "generate",
  "result": "data:image/jpg;base64,/9j/4AAQSkZJRg...",
  "task_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

#### Operation: Transform Image

Modify an existing image based on a prompt.

**Request:**

```json
{
  "provider": "google_fx",
  "operation": "transform",
  "parameters": {
    "prompt": "Make it look like a watercolor painting",
    "input_image": "data:image/png;base64,iVBORw0KGgo..."
  }
}
```

**Parameters:**

- `prompt` (required): How you want to transform the image

- `input_image` (required): Base64-encoded data URI of the input image

**Response:**

```json
{
  "success": true,
  "provider": "google_fx",
  "operation": "transform",
  "result": "data:image/png;base64,iVBORw0KGgo...",
  "task_id": "550e8400-e29b-41d4-a716-446655440001"
}
```

#### Operation: Extract Prompt

Extract a text description from an image.

**Request:**

```json
{
  "provider": "google_fx",
  "operation": "extract_prompt",
  "parameters": {
    "input_image": "data:image/png;base64,iVBORw0KGgo..."
  }
}
```

**Response:**

```json
{
  "success": true,
  "provider": "google_fx",
  "operation": "extract_prompt",
  "result": "A serene mountain landscape at sunset with purple and orange hues",
  "task_id": "550e8400-e29b-41d4-a716-446655440002"
}
```

---

### 3. Video Generation

Generate videos from images with text prompts.

**Endpoint:** `POST /api/v1/videos`

**Request:**

```json
{
  "provider": "google_fx",
  "operation": "generate",
  "parameters": {
    "prompt": "Camera slowly pans across the landscape",
    "input_image": "data:image/png;base64,iVBORw0KGgo...",
    "loop": true
  }
}
```

**Parameters:**

- `prompt` (required): Description of how the video should animate
- `input_image` (required): Base64-encoded data URI of the starting image
- `loop` (required): Boolean - whether the video should loop seamlessly

**Response:**

```json
{
  "success": true,
  "provider": "google_fx",
  "operation": "generate",
  "result": "data:video/mp4;base64,AAAAIGZ0eXBpc29t...",
  "task_id": "550e8400-e29b-41d4-a716-446655440003"
}
```

---

### 4. Usage Statistics

Check your current usage and rate limits.

**Endpoint:** `GET /api/v1/usage`

**Response:**

```json
{
  "api_key": "your_key_here",
  "account_limits": {
    "img_gen_per_hour_limit": 2000,
    "video_gen_per_hour_limit": 60,
    "img_generation_threads_allowed": 25,
    "video_generation_threads_allowed": 2
  },
  "current_usage": {
    "hourly_usage": {
      "image_generation": {
        "current_usage": 45,
        "window_start": 1696348800.0
      },
      "video_generation": {
        "current_usage": 3,
        "window_start": 1696348800.0
      }
    },
    "active_threads": {
      "image_threads": 2,
      "video_threads": 0
    }
  },
  "usage_window": "per_hour"
}
```

---

## Error Responses

All errors follow this format:

```json
{
  "error": "Error description here",
  "task_id": "550e8400-e29b-41d4-a716-446655440004"
}
```

**Common Error Codes:**

- `400` - Bad Request (invalid parameters)
- `401` - Unauthorized (invalid or missing API key)
- `403` - Forbidden (feature not allowed on your license)
- `429` - Rate Limit Exceeded
- `500` - Internal Server Error
- `503` - Service Unavailable

---

## Rate Limits

Rate limits are enforced per hour and are license-specific. Typical limits:

- **Image Generation:** 2,000 successful requests per hour
- **Video Generation:** 60 successful requests per hour
- **Concurrent Threads:** Limited based on your license

**Note:** Only successful generations count toward rate limits. Failed requests do not consume your quota.

---

## Code Examples

### Python

```python
import requests
import base64

API_KEY = "your_api_key_here"
BASE_URL = "https://app.recrafter.fun/api/v1"

response = requests.post(
    f"{BASE_URL}/images",
    headers={"X-API-Key": API_KEY},
    json={
        "provider": "google_fx",
        "operation": "generate",
        "parameters": {
            "prompt": "A futuristic city at night",
            "aspect_ratio": "IMAGE_ASPECT_RATIO_LANDSCAPE"
        }
    }
)

result = response.json()
if result["success"]:
    # Split data URI and get the base64 part
    header, encoded = result["result"].split(",", 1)
    
    # Decode and save
    image_bytes = base64.b64decode(encoded)
    with open("output.jpg", "wb") as f:
        f.write(image_bytes)
```

### JavaScript

```javascript
const API_KEY = "your_api_key_here";
const BASE_URL = "https://app.recrafter.fun/api/v1";

async function generateImage() {
  const response = await fetch(`${BASE_URL}/images`, {
    method: "POST",
    headers: {
      "X-API-Key": API_KEY,
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      provider: "google_fx",
      operation: "generate",
      parameters: {
        prompt: "A futuristic city at night",
        aspect_ratio: "IMAGE_ASPECT_RATIO_LANDSCAPE"
      }
    })
  });
  
  const result = await response.json();
  return result.result; // Data URI
}
```

### cURL

```bash
curl -X POST https://app.recrafter.fun/api/v1/images \
  -H "X-API-Key: your_api_key_here" \
  -H "Content-Type: application/json" \
  -d '{
    "provider": "google_fx",
    "operation": "generate",
    "parameters": {
      "prompt": "A futuristic city at night",
      "aspect_ratio": "IMAGE_ASPECT_RATIO_LANDSCAPE"
    }
  }'
```

---

## Best Practices

1. **Store API Keys Securely:** Never hardcode API keys in client-side code
2. **Handle Rate Limits:** Implement exponential backoff for 429 errors
3. **Check Usage:** Monitor your usage with the `/api/v1/usage` endpoint
4. **Validate Responses:** Always check the `success` field before processing results
5. **Data URIs:** Results are returned as base64-encoded data URIs for easy use

---

## Support

For issues or questions, please check the status of your API key and ensure it's active and not expired.
