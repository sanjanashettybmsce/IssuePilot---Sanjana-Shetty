# IssueSense AI - Quick Reference Card

## 🚀 Start Here

```bash
# 1. Setup
cd /Users/sanjana/Desktop/IssuePilot
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# Edit .env with your GitHub token and OpenAI API key

# 2. Run Backend (Terminal 1)
python -m uvicorn backend.main:app --reload

# 3. Run Frontend (Terminal 2)
streamlit run frontend/app.py

# 4. Access
# Frontend: http://localhost:8501
# Backend API Docs: http://localhost:8000/docs
# Health Check: curl http://localhost:8000/health
```

## 📋 API Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/analyze` | Analyze single issue |
| POST | `/batch-analyze` | Analyze multiple issues |
| GET | `/health` | Health check |
| GET | `/` | API info |
| GET | `/docs` | Swagger UI |

## 💻 API Call Examples

### Analyze Issue
```bash
curl -X POST http://localhost:8000/analyze \
  -H "Content-Type: application/json" \
  -d '{"repo_url":"torvalds/linux","issue_number":12345}'
```

### Python
```python
import requests
response = requests.post("http://localhost:8000/analyze", json={
    "repo_url": "pallets/flask",
    "issue_number": 4789
})
print(response.json())
```

## 📁 Project Structure

```
IssuePilot/
├── backend/          # FastAPI server
│   ├── main.py       # API endpoints
│   ├── config.py     # Configuration
│   ├── github_client.py    # GitHub API wrapper
│   ├── context_enricher.py # Context gathering
│   ├── llm_analyzer.py     # OpenAI integration
│   └── models.py     # Data models
├── frontend/         # Streamlit UI
│   ├── app.py        # Main UI
│   └── styles.py     # Styling
├── requirements.txt  # Dependencies
├── .env.example     # Config template
├── README.md        # Overview
├── SETUP.md         # Installation
├── ARCHITECTURE.md  # Design details
└── EXAMPLES.md      # Code examples
```

## ⚙️ Environment Variables

```bash
# Required
GITHUB_TOKEN=ghp_xxx...          # GitHub personal access token
OPENAI_API_KEY=sk-xxx...         # OpenAI API key

# Optional (defaults shown)
GITHUB_API_BASE_URL=https://api.github.com
OPENAI_MODEL=gpt-4-turbo-preview
BACKEND_HOST=localhost
BACKEND_PORT=8000
```

## 🔍 What It Does

```
GitHub Issue + Comments
    ↓
GitHub API: Fetch details, linked items, files, commits
    ↓
Text Processing: Extract stack traces, error messages
    ↓
Context Enrichment: Compile comprehensive context
    ↓
OpenAI API: Analyze with LLM
    ↓
Response Validation: Ensure correct format
    ↓
Return: Structured JSON analysis
    {
      "summary": "Issue description",
      "type": "bug|feature|documentation|question|other",
      "priority_score": {"score": 1-5, "justification": "..."},
      "suggested_labels": ["label1", "label2", "label3"],
      "potential_impact": "Impact description"
    }
```

## 🎨 UI Features

- **Input Section**: Repository URL + Issue number
- **Status Indicator**: Backend connection status
- **Results Display**: 
  - Summary
  - Issue type with emoji
  - Priority score with justification
  - Potential impact warning
  - Suggested labels
  - Raw JSON export
- **Sidebar**: Configuration, links, info

## 📊 Response Format

```json
{
  "summary": "Single sentence problem description",
  "type": "bug",
  "priority_score": {
    "score": 4,
    "justification": "Affects many users, blocking feature"
  },
  "suggested_labels": ["bug", "authentication", "critical"],
  "potential_impact": "Users cannot log in, blocking all access"
}
```

## 🔧 Troubleshooting

| Problem | Solution |
|---------|----------|
| Cannot connect to backend | Start backend: `python -m uvicorn backend.main:app --reload` |
| GitHub token not set | Create `.env` file with `GITHUB_TOKEN=your_token` |
| OpenAI key not set | Add `OPENAI_API_KEY=your_key` to `.env` |
| Import errors | Run `pip install -r requirements.txt` |
| Repository not found | Check format is `owner/repo` |

## 📚 Documentation

| File | Purpose |
|------|---------|
| README.md | Quick overview |
| SETUP.md | Detailed installation |
| ARCHITECTURE.md | System design |
| EXAMPLES.md | API usage examples |
| PROJECT_SUMMARY.md | Complete summary |

## 🌐 Testing Repositories

- `torvalds/linux` - Linux kernel
- `pallets/flask` - Web framework
- `django/django` - Django framework
- `nodejs/node` - Node.js runtime
- `rust-lang/rust` - Rust language

## ⏱️ Performance

- GitHub context: 10-20s
- LLM analysis: 15-30s
- **Total**: ~30-60s per issue

## 🔒 Security

- API keys in `.env` only (never commit)
- GitHub token scoped to minimal permissions
- Input validation on all endpoints
- No sensitive data in logs

## 💡 Tips

1. Start with well-known repos (torvalds/linux, pallets/flask)
2. Test with recent issues (more context available)
3. Check backend logs for debugging
4. Use health endpoint to verify connection
5. Review raw JSON for integration needs

## 🚀 Next Steps

1. ✅ Install dependencies
2. ✅ Configure `.env`
3. ✅ Start backend & frontend
4. ✅ Analyze first issue
5. ✅ Explore UI and API
6. ✅ Integrate into workflows
7. ✅ Deploy to production

## 📞 Quick Help

```bash
# Check health
curl http://localhost:8000/health

# View API docs
open http://localhost:8000/docs

# View frontend
open http://localhost:8501

# Check Python version
python3 --version

# List installed packages
pip list | grep -E "(fastapi|streamlit|openai)"
```

---

**Everything is ready to go!** 🎉

See SETUP.md for detailed instructions.
