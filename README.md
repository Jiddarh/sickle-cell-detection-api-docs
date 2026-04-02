# Sickle Cell Detection API

The Sickle Cell Detection API is an AI-powered medical image classification API built on a deep learning model trained to detect sickle cell disease from blood smear images. It accepts blood smear images as input and returns a structured prediction indicating whether sickle cell disease is present.

This document is intended for developers integrating automated sickle cell detection into healthcare applications, diagnostic tools, and medical imaging platforms.

> ⚠️ **Medical Disclaimer:** This API is intended to assist medical professionals and researchers. It should not be used as a substitute for professional medical diagnosis. Always consult a qualified healthcare provider for medical decisions.

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Quick Start](#quick-start)
- [Endpoints](#endpoints)
  - [Analyze Image](#analyze-image)
  - [Batch Analysis](#batch-analysis)
  - [Model Info](#model-info)
- [Request & Response Examples](#request--response-examples)
- [Error Reference](#error-reference)
- [Troubleshooting](#troubleshooting)
- [Understanding the Model](#understanding-the-model)
- [Final Notes](#final-notes)

---

## Overview

Sickle cell disease is a hereditary blood disorder characterized by abnormally shaped red blood cells. Early and accurate detection is critical for effective treatment. This API automates the detection process using a TensorFlow-based deep learning model trained on blood smear images sourced from Kaggle.

### Key Features

- Analyzes blood smear images in seconds
- Returns clear positive or negative predictions
- Supports both single and batch image analysis
- Built on a model with **93.91% accuracy**

### Typical Use Cases

- Integrating automated screening into diagnostic platforms
- Supporting medical researchers analyzing large datasets
- Building healthcare tools for underserved communities with limited lab access
- Accelerating sickle cell screening in clinical workflows

---

## Authentication

All requests require an API key.

### Getting Your API Key

1. Create an account at the developer portal
2. Navigate to **Settings → API Keys**
3. Click **Generate New Key**
4. Copy your key — it starts with `scd_`

### Authorization Header Format

```
Authorization: Bearer YOUR_API_KEY
```

> ⚠️ Never expose your API key in client-side code or commit it to version control. Always use environment variables.

---

## Quick Start

Make your first API call in minutes.

### cURL

```bash
curl -X POST https://api.sicklecelldection.com/v1/analyze \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": "BASE64_ENCODED_IMAGE",
    "image_format": "jpeg"
  }'
```

### Python

```python
import requests
import base64
import os

API_KEY = os.environ["SCD_API_KEY"]
API_URL = "https://api.sicklecelldection.com/v1/analyze"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

def analyze_image(image_path: str) -> dict:
    with open(image_path, "rb") as image_file:
        encoded_image = base64.b64encode(image_file.read()).decode("utf-8")
    
    payload = {
        "image": encoded_image,
        "image_format": "jpeg"
    }
    
    response = requests.post(API_URL, headers=headers, json=payload)
    response.raise_for_status()
    return response.json()

result = analyze_image("blood_smear_sample.jpg")
print(result)
# Output: {"prediction": "positive", "request_id": "req_9a2b3c"}
```

🎉 You're now successfully using the Sickle Cell Detection API!

---

## Endpoints

### Analyze Image

Analyzes a single blood smear image and returns a sickle cell prediction.

**Endpoint:**
```
POST /v1/analyze
```

**Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `image` | string | Yes | Base64-encoded blood smear image |
| `image_format` | string | Yes | Image format: `jpeg`, `png`, or `tiff` |

**Example Request:**

```json
{
  "image": "BASE64_ENCODED_IMAGE",
  "image_format": "jpeg"
}
```

**Example Response:**

```json
{
  "prediction": "positive",
  "request_id": "req_9a2b3c"
}
```

---

### Batch Analysis

Analyzes multiple blood smear images in a single request. Useful for processing large datasets or multiple patient samples at once.

**Endpoint:**
```
POST /v1/analyze/batch
```

**Request Body:**

| Field | Type | Required | Description |
|---|---|---|---|
| `images` | array | Yes | Array of image objects to analyze |
| `images[].image` | string | Yes | Base64-encoded blood smear image |
| `images[].image_format` | string | Yes | Image format: `jpeg`, `png`, or `tiff` |
| `images[].sample_id` | string | No | Optional identifier for tracking samples |

**Example Request:**

```json
{
  "images": [
    {
      "image": "BASE64_ENCODED_IMAGE_1",
      "image_format": "jpeg",
      "sample_id": "patient_001"
    },
    {
      "image": "BASE64_ENCODED_IMAGE_2",
      "image_format": "jpeg",
      "sample_id": "patient_002"
    }
  ]
}
```

**Example Response:**

```json
{
  "results": [
    {
      "sample_id": "patient_001",
      "prediction": "positive",
      "request_id": "req_9a2b3c"
    },
    {
      "sample_id": "patient_002",
      "prediction": "negative",
      "request_id": "req_9a2b4d"
    }
  ],
  "total_analyzed": 2
}
```

---

### Model Info

Returns technical details about the underlying deep learning model including version, accuracy, and input requirements.

**Endpoint:**
```
GET /v1/model/info
```

**No request body required.**

**Example Response:**

```json
{
  "model_name": "SickleCell-DetectNet",
  "version": "1.0.0",
  "framework": "TensorFlow",
  "accuracy": 93.91,
  "dataset": "Kaggle Blood Smear Dataset",
  "input": {
    "type": "image",
    "formats": ["jpeg", "png", "tiff"],
    "recommended_resolution": "224x224"
  },
  "labels": ["positive", "negative"],
  "last_updated": "2025-01-15"
}
```

---

## Request & Response Examples

### Interpreting the Response

| Field | Value | Meaning |
|---|---|---|
| `prediction` | `positive` | Sickle cell disease indicators detected |
| `prediction` | `negative` | No sickle cell disease indicators detected |
| `request_id` | `req_xxxxx` | Unique identifier for tracking and support |

### Image Requirements

| Requirement | Specification |
|---|---|
| Format | JPEG, PNG, or TIFF |
| Recommended resolution | 224 x 224 pixels |
| Image type | Blood smear microscopy image |
| Encoding | Base64 |

---

## Error Reference

| HTTP Status | Error | Cause | Resolution |
|---|---|---|---|
| `400` | Bad Request | Missing or invalid fields | Check required fields and image encoding |
| `401` | Unauthorized | Missing or invalid API key | Verify your `Authorization` header |
| `413` | Payload Too Large | Image file size exceeds limit | Compress or resize the image |
| `415` | Unsupported Media Type | Invalid image format | Use `jpeg`, `png`, or `tiff` only |
| `429` | Too Many Requests | Rate limit exceeded | Slow down requests or upgrade your plan |
| `500` | Server Error | Internal processing error | Retry the request or contact support |

### Error Response Format

```json
{
  "error": {
    "code": 400,
    "message": "The 'image' field is required."
  }
}
```

---

## Troubleshooting

| Issue | Possible Cause | What to Do |
|---|---|---|
| `400` on every request | Image not properly Base64 encoded | Re-encode the image using a Base64 library |
| `413` Payload Too Large | Image resolution too high | Resize image to 224x224 before encoding |
| Unexpected predictions | Poor image quality | Use high-quality microscopy images |
| Unexpected predictions | Non-blood-smear image submitted | Ensure only blood smear images are submitted |
| `401` Unauthorized | Missing `Bearer` prefix | Ensure header is `Bearer scd_xxxx` not just `scd_xxxx` |

---

## Understanding the Model

### About the Model

The Sickle Cell Detection API is powered by a convolutional neural network (CNN) built with TensorFlow and trained on a labeled dataset of blood smear images sourced from Kaggle. The model classifies images into two categories: **positive** (sickle cell indicators present) and **negative** (no indicators detected).

### Model Performance

| Metric | Value |
|---|---|
| Accuracy | 93.91% |
| Framework | TensorFlow |
| Architecture | Convolutional Neural Network (CNN) |
| Training Dataset | Kaggle Blood Smear Dataset |
| Input Size | 224 x 224 pixels |

### Model Limitations

- Performs best on high-quality microscopy images
- May produce inaccurate results on low-resolution or blurry images
- Not trained to detect other blood disorders
- Should not be used as the sole basis for a medical diagnosis
- Performance may vary on images from different microscopy equipment

> **Important:** This model achieves 93.91% accuracy, meaning approximately 1 in 17 predictions may be incorrect. Always combine API results with professional medical judgment.

---

## Final Notes

- Always store your API key in environment variables — never hardcode it
- Resize images to 224x224 before encoding for best performance and faster responses
- For large-scale screening, use the batch endpoint to reduce the number of API calls
- This API is a demo project built on a deep learning model developed as a final year research project
- For questions or feedback, open an issue in the repository
