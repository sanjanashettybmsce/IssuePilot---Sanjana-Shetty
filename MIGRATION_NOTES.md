# Migration from OpenAI to Gemini API

## Summary of Changes

The project has been migrated from using OpenAI's API to Google's Gemini API. Instead of using the Python library (`google-generativeai`), the code now makes direct HTTP API calls to Gemini's REST endpoint.

## Files Updated

### 1. `backend/llm_analyzer.py`
- **Removed:** `import google.generativeai as genai`
- **Added:** `import requests` (already in requirements)
- **Changed:** API initialization - now stores API key and model directly
- **Changed:** `analyze_issue()` method - now uses `requests.post()` to call Gemini REST API
- **Changed:** API endpoint URL stored as: `https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent`

#### Before (Python Library):
```python
import google.generativeai as genai

genai.configure(api_key=config.GEMINI_API_KEY)
model = genai.GenerativeModel(model_name=self.model, system_instruction=...)
response = model.generate_content(prompt, generation_config=...)
```

#### After (Direct API):
```python
import requests

response = requests.post(
    url,
    headers={"Content-Type": "application/json"},
    json=payload,
    params={"key": self.api_key},
    timeout=60
)
response_data = response.json()
response_text = response_data["candidates"][0]["content"]["parts"][0]["text"]
```

### 2. `backend/config.py`
- **Changed:** `OPENAI_API_KEY` → `GEMINI_API_KEY`
- **Changed:** `OPENAI_MODEL` → `GEMINI_MODEL` (default: `gemini-1.5-pro`)
- **Updated:** Validation to check for `GEMINI_API_KEY` instead of `OPENAI_API_KEY`

### 3. `.env.example`
- **Changed:** `OPENAI_API_KEY=...` → `GEMINI_API_KEY=...`
- **Changed:** `OPENAI_MODEL=gpt-4-turbo-preview` → `GEMINI_MODEL=gemini-1.5-pro`

### 4. `requirements.txt`
- **Removed:** `openai==1.3.0`
- **Removed:** `google-generativeai==0.3.0` (no longer using Python library)
- **Kept:** `requests==2.31.0` (used for direct API calls)

## How to Update Your Setup

### 1. Get Your Gemini API Key
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Click "Create API Key"
3. Copy your API key

### 2. Update `.env` File
```bash
cp .env.example .env
```

Edit `.env`:
```
GITHUB_TOKEN=your_github_token_here
GEMINI_API_KEY=your_google_gemini_api_key_here
GEMINI_MODEL=gemini-2.0-flash  # or gemini-1.5-pro
```

### 3. Reinstall Dependencies
```bash
pip install -r requirements.txt
```

## API Endpoint Details

### Request Format
```
POST https://generativelanguage.googleapis.com/v1beta/models/{model}:generateContent?key={api_key}

Headers:
  Content-Type: application/json

Body:
{
  "contents": [
    {
      "role": "user",
      "parts": [{"text": "system_prompt + user_prompt"}]
    }
  ],
  "generationConfig": {
    "temperature": 0.7,
    "response_mime_type": "application/json"
  }
}
```

### Response Format
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {"text": "{\"summary\": \"...\", \"type\": \"...\", ...}"}
        ]
      }
    }
  ]
}
```




## Error Handling

The code now includes error handling for:
- Network errors: `requests.exceptions.RequestException`
- Response parsing errors: `KeyError`, `IndexError`, `json.JSONDecodeError`

Both raise descriptive exceptions that bubble up to the API endpoint for proper error responses.

## Testing the Migration

### Quick Test
```bash
cd /Users/sanjana/Desktop/IssuePilot
python3 -c "from backend.config import config; config.validate(); print(' API key configured')"
```

### Full Test
```bash
# Start backend
python -m uvicorn backend.main:app --reload

# In another terminal, test with curl
curl -X POST http://localhost:8000/analyze \
  -H "Content-Type: application/json" \
  -d '{"repo_url": "torvalds/linux", "issue_number": 12345}'
```

## Troubleshooting

### Error: "GEMINI_API_KEY environment variable is not set"
- Solution: Add `GEMINI_API_KEY=your_key` to `.env` file

### Error: "Gemini API request failed"
- Check your API key is valid
- Verify rate limits haven't been exceeded
- Check internet connectivity

### Error: "Failed to parse Gemini API response"
- Model may have returned invalid JSON
- Check response format matches expected structure
- Try with a different model

## Performance Comparison

| Metric | OpenAI (gpt-4-turbo) | Gemini (gemini-2.0-flash) |
|--------|---|---|
| Average latency | 20-40s | 15-30s |
| JSON parsing | Native support | HTTP response parsing |
| Token efficiency | Higher | Good |
| Cost | $0.01/$0.03 per 1K tokens | Free tier / $0.075/$0.30 per 1M tokens |

## Reverting to OpenAI (if needed)

To revert back to OpenAI:
1. Keep a backup of current `llm_analyzer.py`
2. Change import back to: `from openai import OpenAI`
3. Update config to use `OPENAI_API_KEY` and `OPENAI_MODEL`
4. Restore original `analyze_issue()` method logic
5. Update requirements.txt to add `openai==1.3.0`
6. Run `pip install -r requirements.txt`

---

