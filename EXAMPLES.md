# API Examples & Testing

## Quick Start Examples

### 1. Test Health Check

```bash
curl http://localhost:8000/health
```

**Response:**
```json
{
  "status": "healthy",
  "service": "IssueSense AI",
  "version": "1.0.0"
}
```

### 2. Single Issue Analysis

```bash
curl -X POST "http://localhost:8000/analyze" \
  -H "Content-Type: application/json" \
  -d '{
    "repo_url": "pallets/flask",
    "issue_number": 4789
  }'
```

**Response:**
```json
{
  "summary": "Users report template rendering fails with unicode characters in filenames.",
  "type": "bug",
  "priority_score": {
    "score": 4,
    "justification": "Affects template system, impacting international users with unicode filenames."
  },
  "suggested_labels": ["bug", "templates", "unicode"],
  "potential_impact": "Template files with unicode names cannot be rendered, breaking applications serving international audiences."
}
```

### 3. Batch Analysis

```bash
curl -X POST "http://localhost:8000/batch-analyze" \
  -H "Content-Type: application/json" \
  -d '[
    {"repo_url": "pallets/flask", "issue_number": 4789},
    {"repo_url": "django/django", "issue_number": 35400},
    {"repo_url": "torvalds/linux", "issue_number": 8843}
  ]'
```

**Response:**
```json
[
  {
    "repo": "pallets/flask",
    "issue": 4789,
    "analysis": {
      "summary": "...",
      "type": "bug",
      ...
    },
    "status": "success"
  },
  {
    "repo": "django/django",
    "issue": 35400,
    "analysis": { ... },
    "status": "success"
  },
  {
    "repo": "torvalds/linux",
    "issue": 8843,
    "error": "GitHub API rate limit exceeded",
    "status": "failed"
  }
]
```

## Python Client Example

```python
import requests
import json

# Configuration
BACKEND_URL = "http://localhost:8000"
REPO = "torvalds/linux"
ISSUE_NUM = 12345

# Make request
response = requests.post(
    f"{BACKEND_URL}/analyze",
    json={
        "repo_url": REPO,
        "issue_number": ISSUE_NUM
    },
    timeout=120
)

# Handle response
if response.status_code == 200:
    analysis = response.json()
    
    print(f"📊 Analysis for {REPO}#{ISSUE_NUM}")
    print(f"Summary: {analysis['summary']}")
    print(f"Type: {analysis['type']}")
    print(f"Priority: {analysis['priority_score']['score']}/5")
    print(f"Labels: {', '.join(analysis['suggested_labels'])}")
    print(f"Impact: {analysis['potential_impact']}")
else:
    print(f"Error: {response.status_code}")
    print(response.json())
```

## JavaScript/Node.js Client Example

```javascript
const axios = require('axios');

async function analyzeIssue(repo, issueNumber) {
  try {
    const response = await axios.post(
      'http://localhost:8000/analyze',
      {
        repo_url: repo,
        issue_number: issueNumber
      },
      { timeout: 120000 }
    );
    
    const analysis = response.data;
    console.log(`📊 Analysis for ${repo}#${issueNumber}`);
    console.log(`Summary: ${analysis.summary}`);
    console.log(`Type: ${analysis.type}`);
    console.log(`Priority: ${analysis.priority_score.score}/5`);
    console.log(`Labels: ${analysis.suggested_labels.join(', ')}`);
    console.log(`Impact: ${analysis.potential_impact}`);
    
    return analysis;
  } catch (error) {
    console.error('Error analyzing issue:', error.message);
    throw error;
  }
}

// Usage
analyzeIssue('nodejs/node', 45678);
```

## Testing with Postman

### Setup
1. Import from URL:
   ```
   http://localhost:8000/openapi.json
   ```

### Tests

**Test 1: Analyze Single Issue**
- Method: POST
- URL: `http://localhost:8000/analyze`
- Body:
  ```json
  {
    "repo_url": "pallets/flask",
    "issue_number": 4789
  }
  ```

**Test 2: Batch Analysis**
- Method: POST
- URL: `http://localhost:8000/batch-analyze`
- Body:
  ```json
  [
    {"repo_url": "pallets/flask", "issue_number": 4789},
    {"repo_url": "django/django", "issue_number": 35400}
  ]
  ```

## Popular Repositories for Testing

| Repository | Stars | Common Issues |
|---|---|---|
| `torvalds/linux` | 180K+ | Kernel bugs, performance |
| `pallets/flask` | 67K+ | Web framework issues |
| `django/django` | 77K+ | Framework bugs, features |
| `nodejs/node` | 108K+ | Runtime issues |
| `rust-lang/rust` | 98K+ | Language features |
| `python/cpython` | 63K+ | Core language issues |
| `kubernetes/kubernetes` | 109K+ | Container orchestration |
| `tensorflow/tensorflow` | 185K+ | Machine learning issues |

## Error Handling

### GitHub Repository Not Found
```json
{
  "detail": "Error analyzing issue. Please check the repository and issue number."
}
```

### Invalid Repository Format
```json
{
  "detail": "Invalid repository format. Use 'owner/repo'"
}
```

### API Key Issues
```json
{
  "detail": "Error analyzing issue. Please check the repository and issue number."
}
```

### Rate Limiting
```json
{
  "detail": "Error analyzing issue. Please check the repository and issue number."
}
```

## Performance Benchmarks

| Step | Typical Time | Range |
|---|---|---|
| GitHub context fetch | 10-20s | 5-40s |
| Stack trace extraction | 1-2s | 0-5s |
| Recent commits fetch | 5-10s | 2-20s |
| LLM analysis | 15-30s | 10-60s |
| **Total** | **31-62s** | **17-125s** |

## Optimization Tips

### 1. Filter by Label
Pre-filter issues by label in GitHub to reduce context:
```python
response = requests.post(
    f"{BACKEND_URL}/analyze",
    json={
        "repo_url": "torvalds/linux",
        "issue_number": 12345,
        "filter_by_label": "bug"  # Future feature
    }
)
```

### 2. Use Faster Model
Edit `.env` to use faster models:
```
OPENAI_MODEL=gpt-3.5-turbo  # Faster but less accurate
OPENAI_MODEL=gpt-4          # More accurate but slower
```

### 3. Batch Processing
Analyze multiple issues in one request:
```bash
POST /batch-analyze with 10+ issues
```

## Debugging

### Enable Verbose Logging
Backend:
```bash
LOGLEVEL=DEBUG python -m uvicorn backend.main:app --reload
```

### Check Backend Status
```bash
curl -v http://localhost:8000/health
```

### Inspect API Docs
```
http://localhost:8000/docs (Swagger UI)
http://localhost:8000/redoc (ReDoc)
```

## Integration Examples

### GitHub Actions Workflow
```yaml
name: Analyze Issues

on:
  issues:
    types: [opened]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - name: Analyze Issue
        run: |
          curl -X POST "http://your-server:8000/analyze" \
            -H "Content-Type: application/json" \
            -d '{
              "repo_url": "${{ github.repository }}",
              "issue_number": ${{ github.event.issue.number }}
            }'
```

### Git Hook
```bash
#!/bin/bash
# .git/hooks/post-commit

ISSUE_NUM=$(git log -1 --pretty=%B | grep -oP '#\K\d+')
if [ ! -z "$ISSUE_NUM" ]; then
  curl -X POST "http://localhost:8000/analyze" \
    -H "Content-Type: application/json" \
    -d "{\"repo_url\": \"owner/repo\", \"issue_number\": $ISSUE_NUM}"
fi
```

---

**Last Updated:** December 2025
