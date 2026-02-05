# Sora 2 API

Access OpenAI's Sora 2 text-to-video generation model with audio through the [muapi.ai](https://muapi.ai) API.

<img width="1536" height="1024" alt="Sora 2 API Demo" src="https://github.com/user-attachments/assets/91432155-4152-4a43-b9f4-6824b2184e5b" />

## Features

- **Text-to-Video Generation** - Generate videos from text prompts using OpenAI's Sora 2 model
- **Audio Support** - Generated videos include audio based on the prompt description
- **Configurable Output** - Choose resolution (720p, 1080p) and aspect ratio (16:9, 9:16, 1:1)
- **Async Processing** - Submit tasks and poll for results with simple status tracking

## Prerequisites

- Python 3.7+
- A [muapi.ai](https://muapi.ai) API key

## Installation

1. Clone the repository:
```bash
git clone https://github.com/SamurAIGPT/Sora-2-api.git
cd Sora-2-api
```

2. Install dependencies:
```bash
pip install requests python-dotenv
```

3. Create a `.env` file in the project root:
```bash
MUAPIAPP_API_KEY=your_api_key_here
```

## Usage

```python
import os
import time
import requests
from dotenv import load_dotenv

load_dotenv()

def main():
    API_KEY = os.getenv("MUAPIAPP_API_KEY")
    if not API_KEY:
        raise ValueError("MUAPIAPP_API_KEY not found in environment variables")

    # 1. Submit video generation task
    url = "https://api.muapi.ai/api/v1/openai-sora-2-text-to-video"
    headers = {
        "Content-Type": "application/json",
        "x-api-key": API_KEY,
    }
    payload = {
        "prompt": (
            "A cyclist rides through a lively European street at sunrise. "
            "The camera follows behind as warm golden light hits old stone buildings, "
            "people open café shops, and pigeons scatter. "
            "You hear bicycle wheels clicking, distant chatter, and a soft morning breeze."
        ),
        "resolution": "720p",
        "aspect_ratio": "16:9",
    }

    print("Submitting video generation task...")
    response = requests.post(url, headers=headers, json=payload)

    if response.status_code != 200:
        print(f"Error submitting task: {response.status_code} - {response.text}")
        return

    result = response.json()
    request_id = result.get("request_id")
    if not request_id:
        print("No request_id returned in response:", result)
        return

    print(f"Task submitted successfully. Request ID: {request_id}")

    # 2. Poll for results
    poll_url = f"https://api.muapi.ai/api/v1/predictions/{request_id}/result"
    while True:
        poll_response = requests.get(poll_url, headers={"x-api-key": API_KEY})
        if poll_response.status_code != 200:
            print(f"Error polling task: {poll_response.status_code} - {poll_response.text}")
            break

        poll_data = poll_response.json()
        status = poll_data.get("status")

        if status == "completed":
            video_url = poll_data["outputs"][0]
            print("Task completed!")
            print(f"Video URL: {video_url}")
            break
        elif status == "failed":
            print("Task failed:", poll_data.get("error"))
            break
        else:
            print(f"Task still processing... Status: {status}")
            time.sleep(1)  # Poll every 1s

if __name__ == "__main__":
    main()
```

## API Reference

### Submit Video Generation Task

**Endpoint:** `POST https://api.muapi.ai/api/v1/openai-sora-2-text-to-video`

**Headers:**
| Header | Description |
|--------|-------------|
| `Content-Type` | `application/json` |
| `x-api-key` | Your muapi.ai API key |

**Request Body:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `prompt` | string | Text description of the video to generate (include audio cues for sound) |
| `resolution` | string | Output resolution: `720p` or `1080p` |
| `aspect_ratio` | string | Video aspect ratio: `16:9`, `9:16`, or `1:1` |

**Response:**
```json
{
  "request_id": "unique-request-id"
}
```

### Poll for Results

**Endpoint:** `GET https://api.muapi.ai/api/v1/predictions/{request_id}/result`

**Headers:**
| Header | Description |
|--------|-------------|
| `x-api-key` | Your muapi.ai API key |

**Response:**
```json
{
  "status": "completed",
  "outputs": ["https://url-to-generated-video.mp4"]
}
```

**Status Values:**
- `processing` - Video is being generated
- `completed` - Video generation finished successfully
- `failed` - Video generation failed (check `error` field)

## Resources

- [Medium Article](https://medium.com/@anilmatcha/openai-sora-2-api-text-to-video-generation-with-audio-exclusive-on-muapi-ai-3223d3623c1d) - Detailed walkthrough and examples
- [muapi.ai](https://muapi.ai) - API provider

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
