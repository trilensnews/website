# Political News App - Development Roadmap

## Phase Overview

**MVP Timeline: 8-10 weeks**
- Week 1-2: Backend foundation + database
- Week 3-4: n8n workflows
- Week 5-6: Frontend UI
- Week 7: Integration + testing
- Week 8: Polish + soft launch
- Week 9-10: Feedback + iterate

---

## WEEK 1-2: Backend Foundation & Database

### Day 1-2: Project Setup

**What to do:**
1. Create GitHub repo
2. Set up project structure
3. Initialize backend
4. Set up version control

**Commands:**
```bash
# Create repo
mkdir political-news-app
cd political-news-app
git init
git remote add origin https://github.com/YOUR_USERNAME/political-news-app.git

# Initialize backend (choose one)
# Option 1: Node.js + Express
npm init -y
npm install express dotenv cors axios nodemon

# Option 2: Python + FastAPI
pip install fastapi uvicorn python-dotenv httpx
```

**Project structure:**
```
political-news-app/
├── backend/
│   ├── src/
│   │   ├── app.js (or main.py)
│   │   ├── config/
│   │   │   └── database.js
│   │   ├── routes/
│   │   ├── models/
│   │   └── utils/
│   ├── .env
│   ├── .gitignore
│   └── package.json
├── frontend/
│   ├── src/
│   ├── public/
│   └── package.json
├── n8n-workflows/
│   └── [n8n workflow files]
├── database/
│   └── schema.sql
├── docs/
│   └── API.md
└── README.md
```

**First commit:**
```bash
git add .
git commit -m "Initial project setup"
git push origin main
```

---

### Day 3-4: Database Schema

**What to do:**
1. Design PostgreSQL schema
2. Create migrations
3. Set up database locally

**Database schema:**
```sql
-- Outlets table
CREATE TABLE outlets (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  bias_rating VARCHAR(50), -- LEFT, CENTER, RIGHT
  url VARCHAR(255),
  rss_feed_url VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Articles table
CREATE TABLE articles (
  id SERIAL PRIMARY KEY,
  outlet_id INT NOT NULL REFERENCES outlets(id),
  headline VARCHAR(500) NOT NULL,
  summary TEXT,
  body TEXT,
  url VARCHAR(500) UNIQUE NOT NULL,
  published_at TIMESTAMP,
  scraped_at TIMESTAMP DEFAULT NOW(),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Stories table
CREATE TABLE stories (
  id SERIAL PRIMARY KEY,
  headline_neutral VARCHAR(500) NOT NULL,
  post_date DATE NOT NULL,
  left_article_id INT REFERENCES articles(id),
  center_article_id INT REFERENCES articles(id),
  right_article_id INT REFERENCES articles(id),
  status VARCHAR(50) DEFAULT 'draft', -- draft, published, archived
  likes_count INT DEFAULT 0,
  comments_count INT DEFAULT 0,
  views INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  published_at TIMESTAMP
);

-- Comments table
CREATE TABLE comments (
  id SERIAL PRIMARY KEY,
  story_id INT NOT NULL REFERENCES stories(id),
  user_id VARCHAR(255), -- anonymous user ID for MVP
  text TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Engagement table
CREATE TABLE engagement (
  id SERIAL PRIMARY KEY,
  story_id INT NOT NULL REFERENCES stories(id),
  user_id VARCHAR(255),
  action VARCHAR(50), -- like, save, share, view
  created_at TIMESTAMP DEFAULT NOW()
);
```

**Create database:**
```bash
# Install PostgreSQL if not already installed
# macOS: brew install postgresql
# Linux: sudo apt install postgresql postgresql-contrib
# Windows: Download from postgresql.org

# Start PostgreSQL
pg_ctl start

# Create database
createdb political_news_app

# Load schema
psql political_news_app < database/schema.sql
```

**Create .env file:**
```
DATABASE_URL=postgresql://localhost/political_news_app
NODE_ENV=development
PORT=5000
NEWSAPI_KEY=your_key_here
OPENAI_API_KEY=your_key_here
```

---

### Day 5: Backend API Setup

**What to do:**
1. Create Express/FastAPI server
2. Set up database connection
3. Create basic CRUD endpoints

**Node.js/Express (src/app.js):**
```javascript
const express = require('express');
const dotenv = require('dotenv');
const cors = require('cors');
const pool = require('./config/database');

dotenv.config();
const app = express();

// Middleware
app.use(express.json());
app.use(cors());

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'OK' });
});

// Get all stories
app.get('/api/stories', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT * FROM stories WHERE status = $1 ORDER BY published_at DESC',
      ['published']
    );
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Get single story with articles
app.get('/api/stories/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const result = await pool.query(`
      SELECT 
        s.*,
        left_a.headline as left_headline,
        left_a.summary as left_summary,
        left_a.url as left_url,
        left_o.name as left_outlet,
        center_a.headline as center_headline,
        center_a.summary as center_summary,
        center_a.url as center_url,
        center_o.name as center_outlet,
        right_a.headline as right_headline,
        right_a.summary as right_summary,
        right_a.url as right_url,
        right_o.name as right_outlet
      FROM stories s
      LEFT JOIN articles left_a ON s.left_article_id = left_a.id
      LEFT JOIN outlets left_o ON left_a.outlet_id = left_o.id
      LEFT JOIN articles center_a ON s.center_article_id = center_a.id
      LEFT JOIN outlets center_o ON center_a.outlet_id = center_o.id
      LEFT JOIN articles right_a ON s.right_article_id = right_a.id
      LEFT JOIN outlets right_o ON right_a.outlet_id = right_o.id
      WHERE s.id = $1
    `, [id]);
    res.json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Create story (for approval dashboard)
app.post('/api/stories', async (req, res) => {
  try {
    const { headline_neutral, post_date, left_article_id, center_article_id, right_article_id } = req.body;
    const result = await pool.query(
      'INSERT INTO stories (headline_neutral, post_date, left_article_id, center_article_id, right_article_id) VALUES ($1, $2, $3, $4, $5) RETURNING *',
      [headline_neutral, post_date, left_article_id, center_article_id, right_article_id]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Like story
app.post('/api/stories/:id/like', async (req, res) => {
  try {
    const { id } = req.params;
    await pool.query(
      'UPDATE stories SET likes_count = likes_count + 1 WHERE id = $1',
      [id]
    );
    res.json({ success: true });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

**Test endpoints:**
```bash
# Start server
npm start

# In another terminal, test
curl http://localhost:5000/health
```

---

## WEEK 3-4: n8n Workflows

### Day 1-2: n8n Setup

**What to do:**
1. Install n8n locally
2. Create dashboard

**Install n8n:**
```bash
# Install globally
npm install -g n8n

# Start n8n
n8n start

# Access at http://localhost:5678
```

---

### Day 3-5: Build Core Workflows

**Workflow 1: Hourly News Pull**
- Trigger: Every hour
- Steps:
  1. Fetch from NewsAPI
  2. Parse headlines
  3. Store in database

**Workflow 2: Story Matching**
- Trigger: After news pull
- Steps:
  1. Get recent articles
  2. Match same story across outlets
  3. Rank by coverage

**Workflow 3: Daily Flagging**
- Trigger: Every morning (9am)
- Steps:
  1. Get top story candidate
  2. Fetch full articles
  3. Generate summaries via OpenAI
  4. Send email to you with story preview

**Workflow 4: Story Publishing**
- Trigger: Webhook (you approve in dashboard)
- Steps:
  1. Save story to database
  2. Mark as published
  3. Post to social (future)

**n8n workflow structure:**
```
n8n-workflows/
├── 1-hourly-news-pull.json
├── 2-story-matching.json
├── 3-daily-flagging.json
└── 4-story-publishing.json
```

---

## WEEK 5-6: Frontend UI

### Day 1-3: React Setup & Components

**Create React app:**
```bash
npx create-react-app frontend
cd frontend
npm install axios tailwindcss
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

**File structure:**
```
frontend/
├── src/
│   ├── components/
│   │   ├── StoryCard.jsx      # Main story display
│   │   ├── ArticlePreview.jsx # LEFT/CENTER/RIGHT articles
│   │   ├── Navbar.jsx
│   │   └── Comments.jsx        # For future
│   ├── pages/
│   │   ├── Feed.jsx           # Main feed (Reels-style)
│   │   ├── Story.jsx          # Single story detail
│   │   └── Dashboard.jsx      # Admin approval dashboard
│   ├── services/
│   │   └── api.js             # Backend API calls
│   ├── App.jsx
│   └── App.css
└── public/
```

---

### Day 4-5: Build Core Components

**StoryCard.jsx (Main story - Reels style):**
```jsx
import React, { useState } from 'react';
import ArticlePreview from './ArticlePreview';

export default function StoryCard({ story }) {
  const [liked, setLiked] = useState(false);

  return (
    <div className="w-full h-screen bg-white flex flex-col justify-between p-4">
      {/* Header */}
      <div>
        <h2 className="text-2xl font-bold mb-2">{story.headline_neutral}</h2>
        <p className="text-gray-600 text-sm">📅 {story.post_date}</p>
      </div>

      {/* Articles */}
      <div className="flex-1 overflow-y-auto my-4">
        <ArticlePreview side="LEFT" article={story.left_article} outlet={story.left_outlet} />
        <ArticlePreview side="CENTER" article={story.center_article} outlet={story.center_outlet} />
        <ArticlePreview side="RIGHT" article={story.right_article} outlet={story.right_outlet} />
      </div>

      {/* Actions */}
      <div className="flex justify-around border-t pt-4">
        <button onClick={() => setLiked(!liked)} className="text-2xl">
          {liked ? '❤️' : '🤍'} Like
        </button>
        <button className="text-2xl">💬 Comment</button>
        <button className="text-2xl">🔖 Save</button>
        <button className="text-2xl">Share</button>
      </div>

      {/* Scroll hints */}
      <div className="text-center text-gray-400 text-xs mt-2">
        ↑ Newer | ↓ Older
      </div>
    </div>
  );
}
```

**ArticlePreview.jsx:**
```jsx
export default function ArticlePreview({ side, article, outlet }) {
  return (
    <div className="border-l-4 border-gray-200 pl-4 mb-4">
      <p className="font-bold text-gray-700">[{side}] {outlet}</p>
      <h3 className="text-lg font-semibold mb-2">{article.headline}</h3>
      <p className="text-gray-600 text-sm mb-2">{article.summary}</p>
      <a href={article.url} target="_blank" rel="noopener noreferrer" className="text-blue-500 text-sm">
        → Read full article
      </a>
    </div>
  );
}
```

**Feed.jsx (Main page - infinite scroll):**
```jsx
import React, { useState, useEffect } from 'react';
import StoryCard from '../components/StoryCard';
import { getStories } from '../services/api';

export default function Feed() {
  const [stories, setStories] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchStories = async () => {
      try {
        const data = await getStories();
        setStories(data);
        setLoading(false);
      } catch (err) {
        console.error(err);
        setLoading(false);
      }
    };
    fetchStories();
  }, []);

  const handleScroll = (direction) => {
    if (direction === 'up' && currentIndex > 0) {
      setCurrentIndex(currentIndex - 1);
    } else if (direction === 'down' && currentIndex < stories.length - 1) {
      setCurrentIndex(currentIndex + 1);
    }
  };

  if (loading) return <div>Loading...</div>;

  return (
    <div 
      className="relative overflow-hidden"
      onWheel={(e) => handleScroll(e.deltaY > 0 ? 'down' : 'up')}
    >
      {stories[currentIndex] && <StoryCard story={stories[currentIndex]} />}
    </div>
  );
}
```

**Dashboard.jsx (Approval workflow):**
```jsx
import React, { useState } from 'react';
import { approveStory } from '../services/api';

export default function Dashboard() {
  const [candidate, setCandidate] = useState(null);
  const [loading, setLoading] = useState(false);

  const handleApprove = async () => {
    setLoading(true);
    try {
      await approveStory(candidate.id);
      setCandidate(null); // Clear for next one
    } catch (err) {
      console.error(err);
    }
    setLoading(false);
  };

  const handleSkip = () => {
    setCandidate(null);
  };

  if (!candidate) {
    return <div className="p-8">No pending stories</div>;
  }

  return (
    <div className="p-8 max-w-2xl">
      <h1 className="text-3xl font-bold mb-4">Story Approval</h1>
      
      <div className="bg-gray-50 p-6 rounded-lg">
        <h2 className="text-xl font-bold mb-2">{candidate.headline_neutral}</h2>
        <p className="text-gray-600 mb-4">📅 {candidate.post_date}</p>

        <div className="space-y-4">
          <div className="border-l-4 border-blue-500 pl-4">
            <p className="font-bold">LEFT: CNN</p>
            <p className="text-sm">{candidate.left_headline}</p>
          </div>
          <div className="border-l-4 border-gray-500 pl-4">
            <p className="font-bold">CENTER: Reuters</p>
            <p className="text-sm">{candidate.center_headline}</p>
          </div>
          <div className="border-l-4 border-red-500 pl-4">
            <p className="font-bold">RIGHT: Fox News</p>
            <p className="text-sm">{candidate.right_headline}</p>
          </div>
        </div>

        <div className="flex gap-4 mt-6">
          <button 
            onClick={handleApprove}
            disabled={loading}
            className="bg-green-500 text-white px-4 py-2 rounded"
          >
            {loading ? 'Approving...' : 'Approve & Post'}
          </button>
          <button 
            onClick={handleSkip}
            className="bg-gray-300 text-gray-800 px-4 py-2 rounded"
          >
            Skip
          </button>
        </div>
      </div>
    </div>
  );
}
```

---

## WEEK 7: Integration & Testing

### Day 1-3: Connect Frontend to Backend

**api.js (Frontend service):**
```javascript
import axios from 'axios';

const API_URL = 'http://localhost:5000/api';

export const getStories = async () => {
  const response = await axios.get(`${API_URL}/stories`);
  return response.data;
};

export const getStory = async (id) => {
  const response = await axios.get(`${API_URL}/stories/${id}`);
  return response.data;
};

export const approveStory = async (id) => {
  const response = await axios.post(`${API_URL}/stories/${id}/approve`);
  return response.data;
};

export const likeStory = async (id) => {
  const response = await axios.post(`${API_URL}/stories/${id}/like`);
  return response.data;
};

export const commentOnStory = async (id, text) => {
  const response = await axios.post(`${API_URL}/stories/${id}/comments`, { text });
  return response.data;
};
```

### Day 4-5: Test Everything

**Testing checklist:**
- [ ] Backend API endpoints return correct data
- [ ] Frontend loads stories correctly
- [ ] Scroll navigation works
- [ ] Like button increments counter
- [ ] Approval dashboard shows candidates
- [ ] n8n workflows execute without errors
- [ ] Database updates when stories are published

---

## WEEK 8: Polish & Soft Launch

### Day 1-2: UI/UX Polish

- [ ] Mobile responsiveness
- [ ] Dark mode option
- [ ] Loading states
- [ ] Error handling
- [ ] Empty states

### Day 3-4: Deploy

**Backend:**
```bash
# Deploy to Heroku or DigitalOcean
heroku create political-news-app
git push heroku main
```

**Frontend:**
```bash
# Deploy to Vercel
npm install -g vercel
vercel
```

**Database:**
- Set up managed PostgreSQL (AWS RDS, Render, Railway)
- Run migrations on production
- Backup strategy

### Day 5: Setup Monitoring

- [ ] Error logging (Sentry)
- [ ] Analytics (simple tracking)
- [ ] Uptime monitoring

---

## WEEK 9-10: Feedback & Iterate

- Launch to 50-100 beta users
- Gather feedback
- Fix bugs
- Measure engagement (DAU, CTR, session length)
- Plan improvements

---

## What to Start RIGHT NOW (Today/Tomorrow)

### Step 1: Create GitHub Repo (15 minutes)
```bash
mkdir political-news-app
cd political-news-app
git init
echo "# Political News App" > README.md
git add README.md
git commit -m "Initial commit"
```

### Step 2: Set Up Backend (30 minutes)
```bash
# Choose Node.js or Python
# Option 1: Node.js
npm init -y
npm install express dotenv cors axios

# Option 2: Python
python -m venv venv
source venv/bin/activate
pip install fastapi uvicorn python-dotenv httpx
```

### Step 3: Create .env File (5 minutes)
```
DATABASE_URL=postgresql://localhost/political_news_app
NODE_ENV=development
PORT=5000
NEWSAPI_KEY=get_from_newsapi.org
OPENAI_API_KEY=get_from_openai.com
```

### Step 4: Create Database Schema (20 minutes)
- Copy schema from above into `database/schema.sql`
- Create PostgreSQL database locally
- Run schema

### Step 5: First API Endpoint (30 minutes)
- Create `src/app.js` with Express hello world
- Add `/health` and `/api/stories` endpoints
- Test with curl

---

## Your First Week Deliverables

By end of Week 1, you should have:

✅ GitHub repo with basic structure  
✅ Express/FastAPI server running  
✅ PostgreSQL database set up with schema  
✅ 3-4 working API endpoints  
✅ Outlets database populated (80 outlets)  
✅ First commit pushed to GitHub  

---

## Commands Cheat Sheet

```bash
# Start backend
npm start  # Node.js
# or
python -m uvicorn main:app --reload  # Python

# Start frontend
cd frontend && npm start

# Start n8n
n8n start

# Start PostgreSQL
pg_ctl start

# Test API
curl http://localhost:5000/health
curl http://localhost:5000/api/stories
```

---

## Resources You'll Need

- NewsAPI account (newsapi.org) - Free tier
- OpenAI API key (platform.openai.com) - Pay as you go
- PostgreSQL (postgresql.org) - Free
- GitHub account - Free
- n8n account (n8n.io) - Free self-hosted
- Heroku or DigitalOcean for deployment

---

## What NOT to Do

❌ Don't build full frontend before backend works  
❌ Don't worry about mobile optimization yet  
❌ Don't create skills before MVP works  
❌ Don't add features beyond the spec  
❌ Don't deploy before testing locally  
❌ Don't hardcode API keys in code  

---

## Ready to Start?

**Your immediate action items:**

1. **TODAY:** Create GitHub repo + backend scaffolding
2. **TOMORROW:** Set up PostgreSQL + database schema
3. **DAY 3:** Build first 3 API endpoints
4. **DAY 4:** Test API with curl/Postman
5. **DAY 5:** Populate outlet database

Don't overthink it. Start with Week 1 only. Ship it by end of Week 8.