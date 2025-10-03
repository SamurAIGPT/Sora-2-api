# Sora 2 api
Sora 2 api access using http://muapi.ai

<img width="1536" height="1024" alt="s2api" src="https://github.com/user-attachments/assets/91432155-4152-4a43-b9f4-6824b2184e5b" />

```
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

    print(f"✅ Task submitted successfully. Request ID: {request_id}")

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
            print("🎉 Task completed!")
            print(f"Video URL: {video_url}")
            break
        elif status == "failed":
            print("❌ Task failed:", poll_data.get("error"))
            break
        else:
            print(f"⏳ Task still processing... Status: {status}")
            time.sleep(1)  # Poll every 1s

if __name__ == "__main__":
    main()
```
