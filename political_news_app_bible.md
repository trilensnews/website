### Automated Selection Path (MVP)
- **AI scans news hourly** across LEFT/CENTER/RIGHT outlets
- **Identifies stories appearing on all three sides** (algorithmic match)
- **Ranks by coverage volume** (which story appears most across outlets?)
- **Flags top candidate daily** with metrics:
  - How many outlets covered it?
  - Which sides covered it?
  - Headline differences (preview of framing)
- **YOU review in < 1 minute:** "Is this a good story to feature?"
  - If YES → approve, story posts
  - If NO → skip, try next candidate
- **Your job:** Quality gate ONLY, not story selection
- **Decision criteria for approval:** "Do the headlines show obvious framing differences?"
  - Not: "Do I think this is important?"
  - Not: "Do I want people to know this?"
  - Only: "Do multiple outlets cover this with different angles?"# TriLens - The Bible
**Version 1.1 - October 2025**

**App Name:** TriLens  
**Tagline:** "See political news from all perspectives"  
**Domain:** trilens.app (reserved)  
**Email:** trilens@gmail.com  
**GitHub:** github.com/yourusername/trilens

This is our source of truth. All decisions reference this document. If something doesn't fit here, we discuss and update this before proceeding.

---

## 1. MISSION STATEMENT

**What we're building:** A news aggregation app that shows the same story from LEFT, CENTER, and RIGHT outlets side-by-side, so users understand how different outlets frame the same event differently.

**Why:** Most news aggregators (Google News, Apple News) are biased toward left-leaning outlets. We show all three sides transparently.

**Who we're building for:** Anyone who wants to understand different perspectives without being manipulated. Could be: students, educators, politically curious adults, people trying to break out of echo chambers. We don't care about their existing political leanings.

**What we're NOT:**
- A fact-checker
- A "neutral" platform claiming to be unbiased
- A social network
- A news source (we link out to real outlets)

**What we ARE:**
- A curation + display service
- Transparent about our process
- Focused on showing media framing differences
- Ad-supported (users don't pay)

---

## 2. CORE PRODUCT: HOW IT WORKS

### User Experience (MVP)

**Mobile-first design inspired by Instagram Reels/TikTok Shorts:**

Each story displays FULL SCREEN (vertical scroll):

```
┌─────────────────────────────────────────┐
│  HEADLINE                   │
│  "Trump Announces $100K     │
│   H-1B Visa Fee"            │
│                             │
│  📅 Oct 19, 2025            │ ← Date news broke (world time, not app time)
│                             │
├─────────────────────────────────────────┤
│ [LEFT]  CNN                 │
│ Headline: "Critics warn..." │
│ Summary: 2-3 sentences      │
│ → Read full article         │
│                             │
├─────────────────────────────────────────┤
│ [CENTER] Reuters            │
│ Headline: "Trump imposes..." │
│ Summary: 2-3 sentences      │
│ → Read full article         │
│                             │
├─────────────────────────────────────────┤
│ [RIGHT] Fox News            │
│ Headline: "Trump protects..." │
│ Summary: 2-3 sentences      │
│ → Read full article         │
│                             │
├─────────────────────────────────────────┤
│ ❤️ Like  💬 Comment         │
│ 📖 Save  Share              │
│                             │
│ ↑ Scroll up (newer)         │
│ ↓ Scroll down (older)       │
└─────────────────────────────────────────┘
```

**Key UX Details:**
- **Full screen card** - Takes up entire viewport
- **Post date visible** - Shows WHEN the news actually happened/broke (not app publish date)
- **Vertical infinite scroll** - Swipe/scroll up = newer stories, down = older stories
- **Smooth transitions** - Like Instagram Reels
- **All actions at bottom** - Like/comment/save/share buttons
- **One story per view** - No horizontal scrolling, no multiple stories visible
- **Mobile responsive** - Optimized for phones, works on desktop but not primary focus

---

## 3. CONTENT SELECTION STRATEGY

### What Stories We Cover

**Must meet ONE of these criteria:**
- Major federal legislation (bills passed, executive orders, SCOTUS decisions)
- Regulatory announcements
- Congressional votes on controversial bills
- Breaking political news covered across left AND right outlets
- Major events where left/center/right frames differ significantly

**Must be covered by outlets on multiple sides of spectrum.**

### Which Outlets We Use

**LEFT-leaning outlets:**
- CNN
- MSNBC
- New York Times
- Washington Post
- NBC News
- The Guardian

**CENTER outlets:**
- Reuters (most important)
- BBC News
- Axios
- Wall Street Journal (News section only, not Opinion)
- Associated Press (note: now Lean Left, but included for baseline)
- Christian Science Monitor
- C-SPAN

**RIGHT-leaning outlets:**
- Fox News
- National Review
- Wall Street Journal (Opinion section)
- The Dispatch
- Washington Examiner

### Outlet Selection Philosophy

We don't rate or judge outlets. We just use established, credible, nationally recognized news sources. AllSides has already done the bias classification work—we use their ratings as reference but don't rely on them.

**Coverage scope:**
- US Federal Level ONLY (for MVP)
- No state/local politics
- No international news
- No opinion pieces (unless they're the story itself, e.g., "Congressman says X")
- No podcasts, video, or broadcast (written news only)

### How We Find Stories

1. **Pull headlines hourly** from NewsAPI + RSS feeds from major outlets
2. **Identify cross-partisan coverage** - story appears on both left and right outlets
3. **Dedup same story** - "Trump H-1B fee" is same story whether CNN or Fox reports it
4. **Gather statements** - find 1-3 articles from each side covering this story
5. **AI generates summaries** of each article (2-3 sentences, close paraphrasing)
6. **Check for quality** - if AI is uncertain or summary looks bad, flag for manual review
7. **Post if confident** - automatically post to app

---

## 4. POSTING WORKFLOW

### Automated Path (AI Decides)
- AI detects story on cross-partisan outlets
- AI gathers articles from left/center/right
- AI writes summaries
- If confidence > 80%, **AUTO-POST** (no human needed)
- Story goes live immediately

### Manual Review Path (Edge Cases)
- AI flags as "uncertain" (confidence 50-80%)
- YOU review in dashboard (< 1 minute check)
- "Approve" or "Reject"
- If approved, post goes live

### Update Frequency
- **MVP:** ONE story per day (manually curated by you)
- **Selection rule:** Must be "no brainer" - obvious cross-partisan coverage with clear framing differences
- **Post-MVP:** Increase to 2-3 stories/day once you validate demand and automation works
- **Never:** Auto-post in MVP. You approve every story.

---

## 5. HOW WE HANDLE TRUTH

### Core Principle: We Show What They Say, Not Whether It's True

**RULE:** We DO NOT fact-check, verify, or judge official statements.

**Example:**
- If Fox News says: "Trump's fee protects American workers"
- And CNN says: "Trump's fee harms innovation"
- We show BOTH as stated by the outlets
- We DON'T add labels, fact-checks, or our opinion

**Why:** The framing itself reveals bias. Users can fact-check themselves.

**Exception:** If an outlet's official statement is demonstrably false AND the opposing party's official statement directly contradicts it with facts, showing both perspectives IS the story (the conflict itself is newsworthy).

---

## 6. MONETIZATION

### Revenue Model
- **Primary:** Google AdSense (display ads)
- **Ad placement:** Between perspectives, below story cards
- **No political ads** initially
- **Users never pay** (free forever)

### Future Options (Not MVP)
- B2B licensing: API for schools, universities, news organizations
- Premium tier: Ad-free, advanced features (if needed)
- Partnerships: Civics education nonprofits, media literacy orgs

### Cost Model
- Servers: ~$50-200/month (start cheap)
- APIs: NewsAPI (~$50-100/month), Congress.gov (free)
- Domain/hosting: ~$10-50/month
- Total: ~$150-300/month initially
- Ads should cover this + growing costs

---

## 7. TECH STACK

| Component | Technology |
|-----------|------------|
| **Frontend** | React (web) or React Native (mobile) |
| **Backend** | Node.js + Express OR Python + FastAPI |
| **Database** | PostgreSQL (structured data) + Redis (cache) |
| **APIs** | NewsAPI, Congress.gov, RSS feeds |
| **Summarization** | OpenAI GPT-4 OR open-source alternative |
| **Automation** | n8n (self-hosted or cloud) |
| **Scraping/Workflows** | n8n for all data pipelines |
| **Version Control** | GitHub |

### Data Schema (High Level)

**Stories table:**
- id, headline, summary, date_published
- left_article_id, center_article_id, right_article_id
- likes, comments_count, views

**Articles table:**
- id, outlet_id, url, headline, summary, body, published_at

**Outlets table:**
- id, name, bias_rating (LEFT/CENTER/RIGHT), url, rss_feed_url

**Comments table:**
- id, story_id, user_id, text, created_at

---

## 8. MVP LAUNCH CHECKLIST

### Must-Have (MVP doesn't launch without these)
- [ ] Outlet database (50-100 outlets tagged LEFT/CENTER/RIGHT)
- [ ] NewsAPI + RSS feed integration
- [ ] Story matching algorithm (find same story across outlets)
- [ ] AI summarization pipeline
- [ ] Manual review dashboard (you approve posts)
- [ ] Simple UI (story card with LEFT/CENTER/RIGHT)
- [ ] Daily cron job (automated pull schedule)
- [ ] Google AdSense integration
- [ ] Database schema + basic backend API
- [ ] Mobile-responsive web design
- [ ] 1-click "Read full article" links

### Nice-to-Have (Post-MVP)
- [ ] Native mobile apps (iOS + Android)
- [ ] User accounts & saved stories
- [ ] Comments system
- [ ] Share to social media
- [ ] Search/filter by date or topic
- [ ] Email digest
- [ ] Analytics dashboard (for us)
- [ ] Like/reaction counts
- [ ] "Lopsided stories" (when all sides ignore something)

### What We're NOT Building in MVP
- ❌ User accounts (can add later)
- ❌ Comments (can add later)
- ❌ Video/podcasts/broadcast (written only)
- ❌ International news (US only)
- ❌ State/local politics (federal only)
- ❌ Our own bias ratings (use AllSides)
- ❌ Fact-checking layer (just show what they say)
- ❌ Automatic story detection (you manually select 1/day)
- ❌ Auto-posting (you approve every story)

---

## 9. SUCCESS METRICS (MVP PHASE)

**Week 1-2:**
- Can we reliably pull stories from all outlets? (cross-outlet matching)
- Does story matching algorithm work? (accuracy > 80%)
- Do we identify 1-2 candidates per day?

**Week 3-4:**
- Does AI rank stories by coverage volume correctly?
- Are flagged stories actually "no brainers"?
- Can you approve/reject in < 1 minute?

**Week 5-8:**
- Do users click through to articles? (target: 20%+ CTR)
- Do users return? (DAU > 0)
- Do ads load? (AdSense integration working)

**Week 9-12 (Soft Launch)**
- 50-500 beta users
- Daily active users (DAU) > 50
- Session length > 3 minutes
- Return rate > 20%

**3 months:**
- 10K+ downloads/signups
- 1K+ DAU
- 30%+ engagement rate
- Ad revenue > $100/month

---

## 10. WHAT WE LEARNED FROM ALLSIDES

### What They Did Right
- Simple core concept (Left | Center | Right side-by-side)
- Daily updates, consistent presence
- Transparency about process
- Rotate story order daily (avoids left bias)
- Partnered with schools (smart for education market)

### What They Did Wrong
- Tried to rate outlet bias (becomes disputed and becomes weakness)
- Claims to be "neutral" while also saying "no one is unbiased" (contradictory)
- Poor UX (slow scrolling, no reader mode, dated interface)
- Low engagement (users read articles and leave)
- Took 12+ years to gain traction (not viral, slow education curve)
- Bias ratings shifted over time (AP moved from Center to Lean Left)

### Our Advantages
1. **Simpler model:** Just show outlets, don't rate them
2. **Better UX:** Mobile-first, fast, clean design
3. **Engagement:** Add comments, reactions, saves from day 1
4. **Honesty:** "We show different perspectives. You decide. We're not neutral either."
5. **Ad monetization:** From day 1, not relying on eventual premium users
6. **Smaller scope:** Fed-only, written news only = faster to launch

### What We're NOT Doing Like AllSides
- ❌ Don't have a bias-rating panel
- ❌ Don't claim neutrality
- ❌ Don't do blind bias surveys
- ❌ Don't rate outlets ourselves
- ❌ Don't expand to TV/podcasts/radio

---

## 11. RED FLAGS TO AVOID

1. ❌ **Editorializing:** If we add commentary, opinions, or interpret statements, we've lost neutrality
2. ❌ **Showing only 1-2 perspectives:** Core value is seeing all three
3. ❌ **Cherry-picking which outlets:** If we inconsistently include/exclude outlets, it's biased
4. ❌ **Hiding our process:** Be transparent about story selection, outlet choices
5. ❌ **Over-updating:** More than daily updates in MVP wastes resources and adds complexity
6. ❌ **Fact-checking stories:** Not our job. Users can do it.
7. ❌ **Making it look neutral:** Be honest about inherent bias in curation
8. ❌ **Too many features at launch:** Keep MVP simple. Add features based on user feedback.
9. ❌ **Relying on user payments:** Ads should work, not premium tier
10. ❌ **Ignoring coverage gaps:** If all three sides ignore a story, we should note it (transparency)

---

## 12. DESIGN PHILOSOPHY

### Visual Principles
- **Equal real estate** - All three perspectives get equal space/prominence
- **Clear source labeling** - Always show which outlet each article is from
- **Neutral tone** - Summaries don't interpret or editorialize
- **Mobile-first** - Design for phones, then desktop
- **Fast & snappy** - No slow scrolling, optimized performance
- **Color coding optional** - LEFT (blue), CENTER (gray), RIGHT (red) - only if it helps, don't overdo

### UX Principles
- **Minimal friction** - Users can skim stories in < 1 minute
- **One-click to full article** - "Read full article" link goes directly to outlet
- **No registration required to view** (can add later for saves/comments)
- **Dark mode option** (nice-to-have)
- **Accessible** - WCAG AA minimum (good contrast, readable fonts)

---

## 13. TEAM & ROLES (MVP Solo Build)

**You:** Everything
- Story curation (manual approval for edge cases)
- Product decisions
- Marketing/distribution

**AI/Automation:**
- Story matching
- Summary generation
- Posting (automated if confident)
- Ad serving

**Tools we'll use:**
- GitHub (code)
- Heroku/DigitalOcean (hosting)
- NewsAPI (data)
- OpenAI (summarization)
- Google AdSense (monetization)

---

## 14. LAUNCH TIMELINE

### Week 1-2: Backend & Data
- Set up GitHub repo
- Database schema (stories, articles, outlets, comments)
- Outlet database (tag 80 outlets LEFT/CENTER/RIGHT)
- Backend API structure (endpoints for frontend)
- n8n instance setup (self-hosted or cloud)

### Week 3-4: AI Pipeline
- Summarization setup (headline + article summary)
- Manual review dashboard
- Auto-posting logic
- Cron job scheduler

### Week 5-6: Frontend & UI
- React app setup
- Story card component
- Article display
- Ad integration
- Mobile responsive

### Week 7: Integration & Testing
- Connect frontend to backend
- Test story pulling → posting → display
- Test ad serving
- Manual QA

### Week 8: Soft Launch (100-500 beta users)
- Deploy to production
- Manual story posting only (you curate)
- Gather feedback
- Monitor for bugs

### Week 9+: Polish & Full Launch
- Fix bugs from beta
- Enable auto-posting if confident
- Monitor metrics
- Plan marketing push

---

## 15. MARKETING & GROWTH (Post-MVP)

### Target Audience
- Teachers/educators (civics classes)
- Students (high school, college)
- Civic engagement nonprofits
- People trying to break echo chambers
- News-literate adults

### Acquisition Channels
- Partnerships with schools/universities
- Posts on education subreddits
- Hacker News (if we build it well)
- ProductHunt (maybe)
- Media literacy organizations
- Twitter/X (tech community)
- Email to journalists, educators

### Growth Metrics to Track
- DAU/MAU
- Article click-through rate
- Session length
- Return rate
- Ad impressions/revenue
- Shares (how viral)
- Comments per story

### Don't Expect
- Viral growth (this is slow, educational growth)
- Mainstream adoption overnight (AllSides took 12 years)
- High willingness to pay (keep it free)

---

## 16. KNOWN LIMITATIONS & TRADEOFFS

### Limitation 1: "Center" Outlets Are Hard to Define
- Solution: Use Reuters + BBC primarily, acknowledge they're somewhat left-of-center
- We're transparent that "center" doesn't mean neutral

### Limitation 2: We Only Show Major Outlets
- Solution: Start with 80 outlets, can expand later
- Tradeoff: Some important smaller voices won't be covered
- Why: Manageable scope for MVP, data aggregation is easier

### Limitation 3: We Can't Prevent User Bias
- Solution: We're just providing options; users still form their own opinions
- This is OK—we're not trying to "fix" polarization, just inform it

### Limitation 4: Limited to Federal US News
- Solution: Start small, expand post-MVP
- Why: More manageable, easier to curate well

### Limitation 5: Some Stories Won't Have All Three Perspectives
- Solution: Post "lopsided roundups" when only 1-2 sides cover it
- Note: Be transparent when this happens

---

## 17. DECISION-MAKING FRAMEWORK

**When we disagree on something, ask:**

1. **Does it align with our mission?** (Show different perspectives transparently)
2. **Does it add value for users?** (Can they understand framing differences better?)
3. **Can we build it in MVP?** (Is it in scope for launch?)
4. **Does it maintain neutrality?** (Are we editorializing or just showing?)
5. **Will users pay/use for it?** (Is it worth our effort?)

If yes to all 5 → build it
If no to any → defer to MVP or don't build

---

## 18. WHAT SUCCESS LOOKS LIKE (1 YEAR)

**User Perspective:**
- Open app daily
- See 2-3 stories
- Read at least 1-2 full articles
- Understand how different outlets framed the same story
- Share with friends who are curious about different perspectives

**Metrics Perspective:**
- 50K+ downloads/users
- 5K+ DAU
- 4-5 minute average session length
- 30%+ article click-through rate
- $2-5K/month in ad revenue
- Net positive unit economics (ad revenue > costs)

**Business Perspective:**
- Self-sustaining through ads
- No need for VC funding
- Can support 1-2 people part-time
- B2B partnerships starting (schools, nonprofits)
- Ready to expand to state/international

---

## 19. QUESTIONS WE'VE ANSWERED

**Q: How do we get neutral sides?**
A: We don't. We show left/center/right outlets side-by-side and let users see framing differences.

**Q: Where do we get center opinions?**
A: Reuters, BBC, Axios, AP. They're imperfect but least obviously biased.

**Q: Who decides what stories matter?**
A: We do (curation), but we're transparent about it. Stories must be covered cross-partisan.

**Q: Will people actually use this?**
A: Unknown. But AllSides proves the concept works. Growth is slow but real.

**Q: How do we monetize?**
A: Ads. Users never pay. B2B licensing is future revenue.

**Q: Can we fact-check?**
A: No. Not our job. We show what outlets say, not whether it's true.

---

## 20. VERSION CONTROL

**Version:** 1.0 (October 2025)

**Changes from this point forward:**
- Any major decision documented as VERSION X.Y
- Always link back to original decision
- Keep changelog at bottom of this document

**To update this document:**
1. Identify what changed
2. Update relevant section
3. Increment version number
4. Add date + summary to changelog
5. Commit to GitHub with reference to this document

---

## CHANGELOG

**v1.0 - October 2025**
- Initial spec created
- Based on AllSides research
- MVP scope defined
- Timeline established

---

## THIS IS THE BIBLE

Everything we build should trace back to this document. If something doesn't fit, we discuss it, update this document, and move forward as a team.

Go build. 🚀