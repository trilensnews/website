# TriLens n8n Architecture

**Last Updated:** October 20, 2025  
**Workflow ID:** `5rRZ28uyPS03HKkZ`  
**Status:** Testing Phase

---

## Architecture Overview

TriLens uses n8n as the core automation platform with Claude Sonnet 4.5 as an AI Agent to discover and match political news stories across LEFT/CENTER/RIGHT perspectives.

```
┌─────────────────────────────────────────────────────────┐
│         n8n Workflow: Story Discovery & Curation        │
└─────────────────────────────────────────────────────────┘
                          │
                          ▼
              ┌───────────────────────┐
              │  Claude AI Agent      │
              │  (Sonnet 4.5)         │
              └───────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
    ┌─────────┐     ┌─────────┐     ┌─────────┐
    │  LEFT   │     │ CENTER  │     │  RIGHT  │
    │  Tool   │     │  Tool   │     │  Tool   │
    └─────────┘     └─────────┘     └─────────┘
          │               │               │
          │   NewsAPI     │   NewsAPI     │   NewsAPI
          │   (100 items) │   (100 items) │   (100 items)
          │               │               │
          └───────────────┼───────────────┘
                          ▼
              ┌───────────────────────┐
              │  Story Matching       │
              │  (AI-powered)         │
              └───────────────────────┘
                          ▼
              ┌───────────────────────┐
              │  Structured Output    │
              │  {headline, left,     │
              │   center, right}      │
              └───────────────────────┘
```

---

## Workflow Nodes

### 1. Manual Trigger
- **Type:** `n8n-nodes-base.manualTrigger`
- **Purpose:** Start workflow on demand for testing
- **Future:** Replace with Schedule Trigger (hourly)

---

### 2. Claude AI Agent
- **Type:** `@n8n/n8n-nodes-langchain.agent`
- **Model:** `claude-sonnet-4-5-20250929`
- **Configuration:**
  - Thinking mode: ✅ Enabled
  - Memory: Buffer window
  - Output parser: Structured JSON

**Agent Prompt:**
```
You are TriLens' AI news curator. Your goal: find ONE story covered by 
LEFT, CENTER, and RIGHT outlets, then create an unbiased summary showing 
how each side frames it differently.

YOUR TOOLS:
- Left - Returns LEFT-leaning articles
- Center - Returns CENTER articles  
- Right - Returns RIGHT articles

YOUR PROCESS:
1. Call all 3 tools (Left, Center, Right)
2. Match stories by:
   - Common keywords in title
   - Same publishedAt date (within 24 hours)
   - Similar entities (people, bills, events)
3. Select best story with coverage from ALL 3 sides
4. Extract 1 article per perspective (most recent)
5. Return structured JSON matching schema

RULES:
✅ Use exact values from API (don't paraphrase)
✅ Write neutral_summary like wire service
✅ Match stories by keywords + date
❌ Don't editorialize
❌ Don't return schema structure
❌ Don't include stories without all 3 perspectives
```

---

### 3. Left Tool (NewsAPI)
- **Type:** `n8n-nodes-base.httpRequestTool`
- **URL:** `https://newsapi.org/v2/everything`
- **Auth:** HTTP Bearer (NewsAPI key)
- **Query Parameters:**
  - `q`: `politics`
  - `sources`: `cnn,nbc-news,msnbc,the-guardian-us`
  - `sortBy`: `publishedAt`
  - `pageSize`: `100`

**Response Structure:**
```json
{
  "status": "ok",
  "totalResults": 150,
  "articles": [
    {
      "source": {"id": "cnn", "name": "CNN"},
      "title": "Article headline",
      "description": "Brief summary",
      "url": "https://cnn.com/...",
      "publishedAt": "2025-10-19T22:27:48Z"
    }
  ]
}
```

---

### 4. Center Tool (NewsAPI)
- **Type:** `n8n-nodes-base.httpRequestTool`
- **URL:** `https://newsapi.org/v2/everything`
- **Auth:** HTTP Bearer (NewsAPI key)
- **Query Parameters:**
  - `q`: `politics`
  - `sources`: `reuters,bbc-news,axios,associated-press`
  - `sortBy`: `publishedAt`
  - `pageSize`: `100`

---

### 5. Right Tool (NewsAPI)
- **Type:** `n8n-nodes-base.httpRequestTool`
- **URL:** `https://newsapi.org/v2/everything`
- **Auth:** HTTP Bearer (NewsAPI key)
- **Query Parameters:**
  - `q`: `politics`
  - `sources`: `fox-news,national-review-online`
  - `sortBy`: `publishedAt`
  - `pageSize`: `100`

---

### 6. Structured Output Parser
- **Type:** `@n8n/n8n-nodes-langchain.outputParserStructured`
- **Purpose:** Ensures Claude returns valid JSON matching our schema

**Output Schema:**
```json
{
  "headline": "Trump commutes George Santos sentence",
  "date_published": "2025-10-17T22:27:48Z",
  "perspectives": {
    "left": {
      "source": "CNN",
      "headline": "Trump says he has commuted sentence of former Rep. George Santos",
      "url": "https://www.cnn.com/...",
      "description": "President Donald Trump said on social media..."
    },
    "center": {
      "source": "Associated Press",
      "headline": "Trump commutes prison sentence for former Rep. George Santos",
      "url": "https://apnews.com/...",
      "description": "Former Representative George Santos will be released..."
    },
    "right": {
      "source": "Fox News",
      "headline": "President Donald Trump commutes former New York GOP Rep. George Santos' prison sentence",
      "url": "https://www.foxnews.com/...",
      "description": "President Donald Trump announced he had signed..."
    }
  },
  "neutral_summary": "President Donald Trump commuted the prison sentence of former Representative George Santos on October 17, 2025. Santos had been serving a seven-year sentence for fraud charges related to his time in Congress.",
  "reasoning": "This story was covered across all three political perspectives with clear framing differences. LEFT outlets emphasized Santos as 'disgraced,' CENTER provided straightforward factual reporting, and RIGHT highlighted Trump's action. Keywords matched: Santos, commute, Trump, sentence. All articles published on October 17, 2025."
}
```

---

### 7. Simple Memory
- **Type:** `@n8n/n8n-nodes-langchain.memoryBufferWindow`
- **Purpose:** Maintains conversation context for AI agent
- **Session:** Custom key based

---

## Data Flow

### Execution Flow

```
1. Manual Trigger
   ↓
2. Claude Agent calls LEFT tool
   ← Returns 100 articles from CNN, NBC, MSNBC, Guardian
   ↓
3. Claude Agent calls CENTER tool
   ← Returns 100 articles from Reuters, BBC, Axios, AP
   ↓
4. Claude Agent calls RIGHT tool
   ← Returns 100 articles from Fox News, National Review
   ↓
5. Claude processes 300 total articles
   - Identifies common stories
   - Matches by keywords + date + entities
   - Ranks by coverage volume
   ↓
6. Claude selects TOP story with all 3 perspectives
   ↓
7. Structured Output Parser validates JSON
   ↓
8. Return final story object
```

### Processing Logic (Inside Claude)

Claude automatically:
1. **Extracts keywords** from each headline
2. **Groups similar articles** using natural language understanding
3. **Checks date proximity** (within 24 hours)
4. **Identifies entities** (people, bills, events)
5. **Filters for cross-partisan coverage** (must have L+C+R)
6. **Ranks candidates** by coverage volume
7. **Selects best match** with clearest framing differences
8. **Generates neutral summary** (wire service style)

**No manual code needed** - Claude handles all matching logic!

---

## API Usage & Rate Limits

### Current Usage
- **3 API calls per execution** (Left + Center + Right)
- **300 articles fetched** (100 per tool)
- **1 execution = 3 NewsAPI requests**

### If Running Hourly
```
3 calls/hour × 24 hours = 72 calls/day
NewsAPI free tier limit = 100 calls/day
✅ Under limit with 28 calls/day buffer
```

### Cost Breakdown
```
NewsAPI:    Free tier (100 calls/day)
Claude API: ~$0.015 per execution
            ($0.36/day if hourly)
            (~$11/month)

Total:      $11/month for MVP
```

---

## Next: Database Integration

### Add PostgreSQL Nodes

After the Structured Output Parser, add:

```
Structured Output Parser
    ↓
[Function Node] - Parse JSON
    ↓
[PostgreSQL] - Insert LEFT article
[PostgreSQL] - Insert CENTER article
[PostgreSQL] - Insert RIGHT article
    ↓
[PostgreSQL] - Insert Story (link to 3 articles)
    ↓
[IF Node] - Check confidence score
    ├→ > 80% → [Update status: approved]
    └→ 50-80% → [Update status: pending review]
```

---

## Planned Enhancements

### Phase 1: Automation (Week 3-4)
- [ ] Replace Manual Trigger with Schedule Trigger (hourly)
- [ ] Add PostgreSQL integration
- [ ] Add Redis caching (prevent duplicate API calls)
- [ ] Add error handling (email alerts on failure)

### Phase 2: Government Data (Week 5-6)
- [ ] Add HTTP Request nodes for Congress.gov API
- [ ] Add HTTP Request nodes for Federal Register API
- [ ] Add RSS Read node for White House feed
- [ ] Create separate workflow for government data

### Phase 3: Manual Review (Week 7)
- [ ] Add Webhook trigger for approval dashboard
- [ ] Create "Approve Story" workflow
- [ ] Create "Reject Story" workflow
- [ ] Add email notifications

### Phase 4: Publishing (Week 8)
- [ ] Add "Publish Story" workflow
- [ ] Connect to frontend API
- [ ] Add analytics tracking
- [ ] Add social media posting (Twitter/X)

---

## Government Data Workflow (Future)

```
Schedule Trigger (daily 6am)
    ↓
[Split in Batches]
    ├→ [HTTP Request] Congress.gov API
    ├→ [HTTP Request] Federal Register API
    ├→ [RSS Read] White House feed
    └→ [HTTP Request] SCOTUS opinions
    ↓
[Merge]
    ↓
[Filter] - Only last 24 hours
    ↓
[For Each] government item:
    ├→ [HTTP Request] Search NewsAPI for coverage
    ├→ [Claude Agent] Match to LEFT/CENTER/RIGHT articles
    └→ [PostgreSQL] Save story
```

---

## Troubleshooting

### Issue: Claude returns incomplete JSON
**Solution:** Update prompt to be more explicit about schema requirements

### Issue: No stories found with all 3 perspectives
**Solution:** 
1. Lower similarity threshold in prompt
2. Expand time window (24h → 48h)
3. Add more news sources

### Issue: API rate limit exceeded
**Solution:**
1. Reduce execution frequency (hourly → every 2 hours)
2. Implement Redis caching
3. Switch to RSS feeds for some outlets

### Issue: Duplicate stories detected
**Solution:** Add PostgreSQL check before inserting new stories

---

## Monitoring

### n8n Built-in Monitoring
- Execution history (last 100 runs)
- Error logs
- Execution time tracking
- Success/failure rate

### External Monitoring (Future)
- Sentry for error tracking
- Custom dashboard for story metrics
- Uptime monitoring (UptimeRobot)
- Cost tracking (API usage)

---

## Export/Backup

### Export Workflow JSON
```bash
# In n8n UI:
Workflows → TriLens Story Discovery → ... → Download
# Saves as: trilens-story-discovery.json
```

### Import Workflow
```bash
# In n8n UI:
Workflows → Import from File → Select JSON
```

### Backup Strategy
1. Export workflow JSON weekly
2. Commit to GitHub
3. Store credentials separately (environment variables)

---

## Security

### API Keys (Stored in n8n)
- ✅ NewsAPI key (HTTP Bearer Auth credential)
- ✅ Anthropic API key (Anthropic API credential)
- 🚧 PostgreSQL connection string (coming soon)

### Best Practices
- Never commit credentials to GitHub
- Use n8n credential manager
- Rotate API keys quarterly
- Monitor API usage for anomalies

---

## Testing

### Manual Test
1. Open n8n: http://localhost:5678
2. Navigate to: "TriLens - Story Discovery & Curation"
3. Click: "Execute Workflow"
4. Check output for valid JSON

### Expected Output
```json
{
  "headline": "...",
  "date_published": "2025-10-XX...",
  "perspectives": {
    "left": {...},
    "center": {...},
    "right": {...}
  },
  "neutral_summary": "...",
  "reasoning": "..."
}
```

### Error Cases to Test
- No stories with all 3 perspectives
- API rate limit exceeded
- Network timeout
- Invalid JSON from Claude
- Missing articles in NewsAPI response

---

## Resources

### Documentation
- n8n Docs: https://docs.n8n.io
- Claude API: https://docs.anthropic.com
- NewsAPI: https://newsapi.org/docs

### Support
- n8n Community: https://community.n8n.io
- GitHub Issues: https://github.com/trilensnews/website/issues

---

**Status:** ✅ Working prototype | 🚧 Database integration next | 📋 Government data planned