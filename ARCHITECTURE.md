# Architecture & Design

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      Streamlit Frontend                          │
│                     (frontend/app.py)                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                  HTTP POST /analyze
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                    FastAPI Backend                               │
│                   (backend/main.py)                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │        Context Enrichment Pipeline                       │   │
│  │      (backend/context_enricher.py)                       │   │
│  │                                                           │   │
│  │  1. GitHub API Client                                    │   │
│  │     ├─ Fetch issue + comments                            │   │
│  │     ├─ Get linked PRs/issues                             │   │
│  │     ├─ Extract changed files                             │   │
│  │     └─ Recent commits                                    │   │
│  │                                                           │   │
│  │  2. Context Enrichment                                   │   │
│  │     ├─ Link extraction (#123 references)                 │   │
│  │     ├─ File gathering from PRs                           │   │
│  │     ├─ Stack trace extraction                            │   │
│  │     └─ Repository metadata                               │   │
│  └──────────────────────────────────────────────────────────┘   │
│                         │                                        │
│                   Enriched Context                               │
│                         │                                        │
│  ┌──────────────────────▼──────────────────────────────────┐   │
│  │      LLM Analysis Pipeline                              │   │
│  │      (backend/llm_analyzer.py)                          │   │
│  │                                                           │   │
│  │  1. Prompt Building                                      │   │
│  │     ├─ Issue title/body                                  │   │
│  │     ├─ Comments summary                                  │   │
│  │     ├─ Linked items                                      │   │
│  │     ├─ Files changed                                     │   │
│  │     └─ Stack traces & commits                            │   │
│  │                                                           │   │
│  │  2. OpenAI API Call                                      │   │
│  │     └─ JSON response with structured analysis            │   │
│  │                                                           │   │
│  │  3. Response Validation                                  │   │
│  │     ├─ Type classification                               │   │
│  │     ├─ Priority score bounds                             │   │
│  │     └─ Label generation                                  │   │
│  └──────────────────────┬──────────────────────────────────┘   │
│                         │                                        │
│                   Structured Analysis                            │
└────────────────────────┬────────────────────────────────────────┘
                         │
                  HTTP 200 + JSON
                         │
┌────────────────────────▼────────────────────────────────────────┐
│                   Streamlit Frontend                             │
│              (Display Analysis Results)                          │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. GitHub Client (`backend/github_client.py`)
- **Purpose:** Abstraction layer for GitHub API interactions
- **Key Methods:**
  - `get_issue()`: Fetch issue with comments
  - `get_linked_issues_and_prs()`: Extract linked items
  - `get_files_from_pr()`: Get changed files
  - `get_file_content()`: Retrieve file content
  - `get_recent_commits()`: Find recent commits
  - `get_repository_info()`: Get repo metadata

### 2. Context Enricher (`backend/context_enricher.py`)
- **Purpose:** Orchestrates comprehensive context gathering
- **Pipeline:**
  1. Fetch issue + comments
  2. Extract linked items from text
  3. Gather files from linked PRs
  4. Extract stack traces and errors
  5. Get recent commits
  6. Collect repository metadata

### 3. LLM Analyzer (`backend/llm_analyzer.py`)
- **Purpose:** Analyzes context using OpenAI's API
- **Process:**
  1. Build comprehensive prompt from context
  2. Call OpenAI Chat Completions API
  3. Request JSON response format
  4. Validate and sanitize response
  5. Return structured analysis

### 4. FastAPI Backend (`backend/main.py`)
- **Purpose:** HTTP API server
- **Endpoints:**
  - `POST /analyze`: Single issue analysis
  - `POST /batch-analyze`: Multiple issues
  - `GET /health`: Health check
  - `GET /`: API documentation

### 5. Streamlit Frontend (`frontend/app.py`)
- **Purpose:** User interface for analysis
- **Features:**
  - Repository URL & issue number input
  - Real-time backend connection status
  - Analysis result visualization
  - Raw JSON export
  - Configuration sidebar

## Data Flow

### 1. User Input
```
User enters:
  - Repository URL: "owner/repo"
  - Issue Number: 123
```

### 2. Request Processing
```
FastAPI receives POST /analyze
  → Validates input format
  → Calls ContextEnricher
```

### 3. Context Enrichment
```
ContextEnricher:
  → GitHub API: Fetch issue #123 + comments
  → GitHub API: Find linked items (#456, #789)
  → GitHub API: Get files from linked PRs
  → Text Processing: Extract stack traces
  → GitHub API: Get recent commits
  → Result: Enriched context object
```

### 4. LLM Analysis
```
LLMAnalyzer:
  → Build detailed prompt with all context
  → OpenAI API: Send prompt + request JSON
  → Validate response format
  → Sanitize values (priority 1-5, type validation)
  → Result: Structured analysis
```

### 5. Response Formatting
```
FastAPI returns:
{
  "summary": "...",
  "type": "bug|feature_request|...",
  "priority_score": {"score": 1-5, "justification": "..."},
  "suggested_labels": ["label1", "label2", "label3"],
  "potential_impact": "..."
}
```

### 6. Frontend Display
```
Streamlit renders:
  → Summary section
  → Issue type badge
  → Priority score with emoji
  → Impact warning
  → Suggested labels
  → Raw JSON viewer
```

## Key Design Decisions

### 1. Context Enrichment First
- **Why:** LLM accuracy depends on context richness
- **Benefit:** More accurate analysis with linked information
- **Trade-off:** Slower but comprehensive

### 2. Separation of Concerns
- **GitHub Client:** Isolated API interactions
- **Context Enricher:** Orchestrates context gathering
- **LLM Analyzer:** Focuses on analysis logic
- **Frontend:** User interface only

### 3. JSON Response Format
- **Why:** Structured, parseable, easy to integrate
- **Benefit:** Can be used in other tools/workflows
- **Validation:** Ensures consistent response format

### 4. Configurable Backend
- **Why:** Works with different hosts/ports
- **Benefit:** Flexible deployment options
- **Frontend:** Sidebar configuration

### 5. Error Handling
- **GitHub API:** Graceful degradation if PR fetch fails
- **LLM API:** Validation + fallback values
- **Frontend:** User-friendly error messages

## Extensibility

### Adding New Analysis Features
1. Extend `ContextEnricher` with new context gathering
2. Update LLM prompt in `LLMAnalyzer`
3. Modify response model in `models.py`

### Adding New Endpoints
1. Create function in `backend/main.py`
2. Define request/response models
3. Add route decorator

### Customizing UI
1. Edit `frontend/app.py` for layout
2. Update `frontend/styles.py` for styling
3. Add new sections for analysis display

## Performance Optimization

### Current Bottlenecks
1. **GitHub API Calls:** Multiple sequential requests
   - Solution: Batch requests or implement caching
2. **LLM Response Time:** Depends on model/context size
   - Solution: Use faster models for quick analysis
3. **Context Size:** Large prompts take longer
   - Solution: Summarize context before sending

### Suggested Improvements
1. **Caching:** Cache GitHub data for 1 hour
2. **Async Processing:** Queue background tasks
3. **Batch Operations:** Analyze multiple issues in parallel
4. **Context Summarization:** Abstract context before LLM

## Security Considerations

1. **API Keys:** Stored in `.env`, never committed
2. **GitHub Token:** Scoped to minimal permissions (repo, read:user)
3. **Input Validation:** Repository format verified
4. **Response Sanitization:** LLM output validated
5. **Error Messages:** No sensitive data in responses

## Testing Strategy

### Unit Tests
- GitHub client methods
- Context enricher functions
- LLM response validation

### Integration Tests
- End-to-end analysis pipeline
- API endpoint responses
- Frontend UI interactions

### Performance Tests
- Context enrichment speed
- LLM response time
- API throughput

---

**Document Version:** 1.0
**Last Updated:** December 2025
