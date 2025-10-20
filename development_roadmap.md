# Political News App - Development Roadmap

**UPDATED: October 2025**  
**Major Pivot:** Custom code → n8n workflows + Claude AI Agent

## Phase Overview

**MVP Timeline: 6-8 weeks** (reduced from 10 weeks)
- Week 1-2: n8n setup + AI Agent workflow ✅ DONE
- Week 3-4: Database integration + manual review dashboard
- Week 5-6: Frontend UI
- Week 7: Integration + testing
- Week 8: Polish + soft launch

---

## ✅ COMPLETED: Week 1-2 (n8n + AI Agent)

### What We Built

**n8n Workflow: "TriLens - Story Discovery & Curation"**
- **Workflow ID:** `5rRZ28uyPS03HKkZ`
- **Status:** Testing phase
- **Architecture:** Claude Sonnet 4.5 AI Agent with 3 tools

### Current Workflow Structure

```
Manual Trigger
    ↓
Claude AI Agent (Sonnet 4.5)
    ├→ Left Tool (NewsAPI: CNN, NBC, MSNBC, Guardian)
    ├→ Center Tool (NewsAPI: Reuters, BBC, Axios, AP)
    └→ Right Tool (NewsAPI: Fox News, National Review)
    ↓
Story Matching (AI-powered)
    ↓
Structured Output Parser
    ↓
JSON Output: {headline, left, center, right, summary}
```

### Key Components

**1. AI Agent Node**
- Model: Claude Sonnet 4.5
- Thinking mode: Enabled
- Memory: Buffer window
- Output: Structured JSON

**2. Three HTTP Request Tools**
```javascript
// Left Tool
URL: https://newsapi.org/v2/everything
Sources: cnn,nbc-news,msnbc,the-guardian-us
Query: politics
Page Size: 100

// Center Tool  
Sources: reuters,bbc-news,axios,associated-press

// Right Tool
Sources: fox-news,national-review-online
```

**3. Structured Output Schema**
```json
{
  "headline": "string",
  "date_published": "ISO timestamp",
  "perspectives": {
    "left": {"source": "", "headline": "", "url": "", "description": ""},
    "center": {"source": "", "headline": "", "url": "", "description": ""},
    "right": {"source": "", "headline": "", "url": "", "description": ""}
  },
  "neutral_summary": "string",
  "reasoning": "string"
}
```

### API Usage
- **Current:** 3 calls per execution (Left + Center + Right)
- **If hourly:** 72 calls/day
- **NewsAPI limit:** 100 calls/day ✅ Under limit

---

## WEEK 3-4: Database Integration + Review Dashboard

### Day 1-2: PostgreSQL Setup

**What to do:**
1. Set up PostgreSQL database
2. Create tables for stories, articles, outlets
3. Connect n8n to database

**Database schema:**
```sql
-- Outlets table
CREATE TABLE outlets (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  bias_rating VARCHAR(50), -- LEFT, CENTER, RIGHT
  url VARCHAR(255),
  rss_feed_url VARCHAR(255),
  newsapi_source_id VARCHAR(100),
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Articles table
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  outlet_id INT NOT NULL REFERENCES outlets(id),
  headline VARCHAR(500) NOT NULL,
  description TEXT,
  url VARCHAR(500) UNIQUE NOT NULL,
  published_at TIMESTAMP,
  scraped_at TIMESTAMP DEFAULT NOW()
);

-- Stories table
CREATE TABLE stories (
  id SERIAL PRIMARY KEY,
  headline VARCHAR(500) NOT NULL,
  date_published TIMESTAMP NOT NULL,
  left_article_id INT REFERENCES articles(id),
  center_article_id INT REFERENCES articles(id),
  right_article_id INT REFERENCES articles(id),
  neutral_summary TEXT,
  ai_reasoning TEXT,
  status VARCHAR(50) DEFAULT 'pending', -- pending, approved, rejected, published
  confidence_score FLOAT,
  likes_count INT DEFAULT 0,
  views INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  approved_at TIMESTAMP,
  published_at TIMESTAMP
);

-- Government data sources (NEW)
CREATE TABLE government_sources (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255),
  source_type VARCHAR(100), -- congress_bill, executive_order, scotus_opinion, federal_register
  url VARCHAR(500),
  api_endpoint VARCHAR(500),
  is_active BOOLEAN DEFAULT true
);
```

**Populate outlets:**
```sql
INSERT INTO outlets (name, bias_rating, newsapi_source_id) VALUES
  ('CNN', 'LEFT', 'cnn'),
  ('NBC News', 'LEFT', 'nbc-news'),
  ('MSNBC', 'LEFT', 'msnbc'),
  ('The Guardian', 'LEFT', 'the-guardian-us'),
  ('Reuters', 'CENTER', 'reuters'),
  ('BBC News', 'CENTER', 'bbc-news'),
  ('Axios', 'CENTER', 'axios'),
  ('Associated Press', 'CENTER', 'associated-press'),
  ('Fox News', 'RIGHT', 'fox-news'),
  ('National Review', 'RIGHT', 'national-review-online');
```

---

### Day 3-4: Update n8n Workflow

**Add PostgreSQL nodes to workflow:**

```
Claude AI Agent (returns story JSON)
    ↓
Save Articles to DB
    ↓
Create Story Record
    ↓
Check Confidence Score
    ├→ > 80% → Auto-approve → Publish
    └→ 50-80% → Flag for manual review
```

**PostgreSQL Insert Nodes:**

1. **Insert Articles** (3 nodes: left, center, right)
2. **Insert Story** (links to 3 articles)
3. **Update Story Status** (based on confidence)

---

### Day 5: Manual Review Dashboard

**Two options:**

**Option 1: n8n Form Trigger (Quick)**
- n8n sends email with story details
- Email has "Approve" and "Reject" links
- Links trigger n8n webhooks
- Webhooks update database

**Option 2: Simple Web Dashboard (Better)**
```bash
# Create minimal React app
npx create-react-app review-dashboard
cd review-dashboard
npm install axios
```

**Dashboard.jsx:**
```jsx
import React, { useState, useEffect } from 'react';
import axios from 'axios';

export default function Dashboard() {
  const [pendingStories, setPendingStories] = useState([]);

  useEffect(() => {
    // Fetch from n8n webhook or PostgreSQL API
    axios.get('http://localhost:5678/webhook/pending-stories')
      .then(res => setPendingStories(res.data));
  }, []);

  const handleApprove = async (id) => {
    await axios.post(`http://localhost:5678/webhook/approve-story`, { id });
    setPendingStories(pendingStories.filter(s => s.id !== id));
  };

  const handleReject = async (id) => {
    await axios.post(`http://localhost:5678/webhook/reject-story`, { id });
    setPendingStories(pendingStories.filter(s => s.id !== id));
  };

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-4">Story Review</h1>
      {pendingStories.map(story => (
        <div key={story.id} className="border p-4 mb-4 rounded">
          <h2 className="text-xl font-bold">{story.headline}</h2>
          <p className="text-sm text-gray-600">Confidence: {story.confidence_score}%</p>
          
          <div className="mt-4 space-y-2">
            <div className="border-l-4 border-blue-500 pl-2">
              <strong>LEFT:</strong> {story.perspectives.left.headline}
            </div>
            <div className="border-l-4 border-gray-500 pl-2">
              <strong>CENTER:</strong> {story.perspectives.center.headline}
            </div>
            <div className="border-l-4 border-red-500 pl-2">
              <strong>RIGHT:</strong> {story.perspectives.right.headline}
            </div>
          </div>

          <div className="mt-4 flex gap-2">
            <button 
              onClick={() => handleApprove(story.id)}
              className="bg-green-500 text-white px-4 py-2 rounded"
            >
              ✓ Approve
            </button>
            <button 
              onClick={() => handleReject(story.id)}
              className="bg-red-500 text-white px-4 py-2 rounded"
            >
              ✗ Reject
            </button>
          </div>
        </div>
      ))}
    </div>
  );
}
```

---

## WEEK 5-6: Frontend UI

### Mobile-First Design (Instagram Reels Style)

**React Setup:**
```bash
npx create-react-app frontend
cd frontend
npm install axios tailwindcss react-router-dom
npx tailwindcss init -p
```

**Key Components:**

1. **StoryCard.jsx** - Full-screen story display
2. **ArticlePerspective.jsx** - LEFT/CENTER/RIGHT sections
3. **Feed.jsx** - Vertical scroll feed
4. **Navigation.jsx** - Bottom nav (future: saved, profile)

**Reference the original roadmap for component code** (still valid)

---

## NEW: Government Data Integration

### Week 7-8: Expand Data Sources

**Why:** Government sources = no bias classification needed, authoritative

**Sources to add:**

1. **Congress.gov API** (free)
   - Bills introduced
   - Roll call votes
   - Committee hearings

2. **Federal Register API** (free)
   - Executive orders
   - New regulations
   - Agency rules

3. **Supreme Court Opinions** (free via SCOTUS API or Court Listener)
   - Recent decisions
   - Oral arguments

4. **White House Briefings** (RSS feed)
   - Press releases
   - Statements

### Implementation in n8n

**New Workflow: "Government Data Aggregator"**

```
Schedule Trigger (daily at 6am)
    ↓
Split in Batches
    ├→ Fetch Congress.gov Bills
    ├→ Fetch Federal Register
    ├→ Fetch SCOTUS Opinions
    └→ Fetch White House RSS
    ↓
Merge Results
    ↓
Filter: Only items from last 24 hours
    ↓
For Each Item:
    ├→ Check if news outlets covered it
    ├→ If yes: Find LEFT/CENTER/RIGHT articles
    └→ Create story card
    ↓
Save to Database
```

**Example: Congress Bill Story**

```json
{
  "headline": "Senate Passes Infrastructure Bill",
  "date_published": "2025-10-18",
  "government_source": {
    "type": "congress_bill",
    "bill_number": "H.R. 3684",
    "url": "https://www.congress.gov/bill/117th-congress/house-bill/3684"
  },
  "perspectives": {
    "left": { "outlet": "CNN", "headline": "...", "url": "..." },
    "center": { "outlet": "Reuters", "headline": "...", "url": "..." },
    "right": { "outlet": "Fox News", "headline": "...", "url": "..." }
  }
}
```

### n8n Nodes Needed

1. **HTTP Request** → Congress.gov API
2. **HTTP Request** → Federal Register API  
3. **RSS Read** → White House feed
4. **Code** → Match government item to news articles
5. **PostgreSQL** → Save results

---

## Updated Tech Stack

| Component | Technology | Status |
|-----------|------------|--------|
| **Automation** | n8n (cloud or self-hosted) | ✅ Active |
| **AI Agent** | Claude Sonnet 4.5 | ✅ Active |
| **News API** | NewsAPI | ✅ Active |
| **Government APIs** | Congress.gov, Federal Register | 🚧 Next |
| **Database** | PostgreSQL + Redis | 🚧 Next |
| **Frontend** | React (mobile-first) | 📋 Planned |
| **Hosting** | Vercel (frontend) + Railway (DB) | 📋 Planned |
| **Monitoring** | n8n built-in + Sentry | 📋 Planned |

---

## What Changed from Original Roadmap

### ✅ Improvements

1. **No custom story matching code** - Claude handles it intelligently
2. **Faster iteration** - n8n visual workflows vs writing code
3. **Built-in scheduling** - n8n replaces cron jobs
4. **Better AI integration** - Native Claude support
5. **Easier to modify** - Non-technical users can adjust workflows

### ⚠️ Tradeoffs

1. **n8n dependency** - Locked into platform (but can export workflows)
2. **Less control** - Can't fine-tune algorithms as precisely
3. **Execution limits** - n8n cloud has execution limits (self-host to avoid)

---

## Current Status (October 2025)

### ✅ Completed
- [x] n8n workflow created
- [x] Claude AI Agent integration
- [x] Three NewsAPI tools (LEFT/CENTER/RIGHT)
- [x] Structured output parser
- [x] Story matching algorithm (AI-powered)
- [x] Under API rate limits (72 calls/day < 100 limit)

### 🚧 In Progress
- [ ] Testing cross-partisan story detection accuracy
- [ ] Refining AI prompt for better matching
- [ ] PostgreSQL integration

### 📋 Next Up
- [ ] Manual review dashboard
- [ ] Schedule workflow to run hourly
- [ ] Government data integration
- [ ] Frontend UI
- [ ] Soft launch (50-100 users)

---

## Immediate Next Steps

### This Week
1. **Set up PostgreSQL database** (1 day)
2. **Connect n8n to database** (1 day)
3. **Build manual review UI** (2 days)
4. **Test end-to-end workflow** (1 day)

### Next Week
1. **Add government data sources** (3 days)
2. **Schedule workflow to run hourly** (1 day)
3. **Start frontend build** (ongoing)

---

## Resources

### Already Have
- ✅ n8n workflow (`5rRZ28uyPS03HKkZ`)
- ✅ NewsAPI account
- ✅ Anthropic API key (Claude)
- ✅ GitHub repo: `trilensnews/website`

### Need to Set Up
- [ ] PostgreSQL database (Railway/Render free tier)
- [ ] Redis cache (optional, for performance)
- [ ] Congress.gov API key (free)
- [ ] Google AdSense account

---

## Commands Cheat Sheet

```bash
# Access n8n
# Go to: http://localhost:5678 (or your cloud URL)

# Test workflow
# Click "Execute Workflow" in n8n UI

# Check PostgreSQL
psql -d trilens_production -c "SELECT * FROM stories WHERE status='pending';"

# Start frontend
cd frontend && npm start

# Deploy frontend
vercel
```

---

## Questions Answered

**Q: Do we still need custom code?**
A: Minimal. Only for frontend UI and simple API endpoints. n8n handles all data pipelines.

**Q: How do we handle story matching without code?**
A: Claude AI Agent does it automatically using natural language understanding.

**Q: Can we still add government sources?**
A: Yes! n8n has HTTP Request and RSS Read nodes that make it easy.

**Q: What if n8n goes down?**
A: Self-host n8n or use n8n cloud with 99.9% uptime. Can export workflows as JSON backup.

**Q: How much does this cost?**
A: 
- n8n: Free (self-hosted) or $20/month (cloud)
- NewsAPI: Free tier works
- Claude API: ~$2-5/month for MVP usage
- Database: Free tier (Railway/Render)
- **Total: $0-30/month**

---

## Ready to Continue?

**Your immediate action items:**

1. **TODAY:** Set up PostgreSQL database
2. **TOMORROW:** Connect n8n to database (add PostgreSQL nodes)
3. **DAY 3:** Build manual review dashboard
4. **DAY 4:** Test full workflow (n8n → DB → dashboard → publish)
5. **DAY 5:** Add government data sources (Congress.gov)

Don't overthink it. Focus on getting database working first.