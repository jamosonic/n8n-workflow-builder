# Custom API Integration Guide for Groq-Based Content Arbitrage

## Your Stack Foundation

### 1. AI Content Generation - Groq (Primary)

```javascript
// Groq API configuration for n8n HTTP Request
{
  "method": "POST",
  "url": "https://api.groq.com/openai/v1/chat/completions",
  "headers": {
    "Authorization": "Bearer {{$credentials.groqApi.key}}",
    "Content-Type": "application/json"
  },
  "body": {
    "model": "mixtral-8x7b-32768", // Best for content, 32k context
    "messages": [{"role": "system", "content": "{{$json.systemPrompt}}"}],
    "temperature": 0.7,
    "max_tokens": 1000
  }
}
```

**Groq Model Recommendations:**

- `mixtral-8x7b-32768` - Best quality for scripts/content
- `llama2-70b-4096` - Faster, good for ideation
- `gemma-7b-it` - Ultra-fast for simple tasks

**Cost: ~$0.27/million tokens** (much cheaper than GPT-4)

### 2. Content Discovery Strategy

Since you mentioned Gummysearch and Ahrefs but aren't currently paying:

**Free Tier Alternatives:**

```yaml
Reddit Discovery:
  - Reddit JSON API (free, no auth): Add .json to any Reddit URL
  - Pushshift.io (if still available)
  - EnsembleData (generous free tier)

Trend Analysis:
  - Google Trends (free, native n8n node)
  - Reddit's own trending API
  - Twitter/X Advanced Search (via web scraping)
```

**If You Decide on Gummysearch ($49/month):**

- Excellent for finding pain points/content ideas
- API access for automation
- Worth it once you're generating revenue

### 3. Essential Services to Add

**Voice Synthesis (Pick One):**

```yaml
Option A - ElevenLabs:
  - Free: 10k characters/month
  - Starter: $5/month (30k characters)
  - Best quality for viral content

Option B - Google Cloud TTS:
  - Free: 4M characters/month
  - Then $4/million characters
  - Good enough for volume

Option C - Edge TTS (Free):
  - Microsoft Edge's TTS API
  - Completely free, decent quality
  - Requires more setup
```

**Video Creation (Pick One):**

```yaml
Option A - Creatomate:
  - $41/month starter
  - Template-based, easy automation
  - Great n8n integration

Option B - Remotion (Self-hosted):
  - Free, React-based
  - More technical but unlimited
  - Full control over rendering

Option C - JSON2Video:
  - $39/month
  - Simple API, good templates
  - Limited but sufficient
```

**Distribution:**

```yaml
Ayrshare:
  - $25/month for 500 posts
  - Covers all major platforms
  - Essential for automation

Alternative: Direct APIs
  - More complex setup
  - Platform-specific limits
  - Requires more maintenance
```

### 4. Proxy Infrastructure (Critical)

**For new social accounts:**

```yaml
Webshare:
  - $30/month for starter residential
  - Essential for account creation
  - Rotate per platform

IPRoyal:
  - Pay as you go ($1.75/GB)
  - Good for testing
  - Scale as needed
```

### 5. Raindrop.io Integration Ideas

```javascript
// Save trending content to Raindrop
{
  "method": "POST",
  "url": "https://api.raindrop.io/rest/v1/raindrop",
  "headers": {
    "Authorization": "Bearer {{$credentials.raindropApi.token}}"
  },
  "body": {
    "link": "{{$json.sourceUrl}}",
    "tags": ["trending", "{{$json.platform}}", "score_{{$json.viralScore}}"],
    "collection": "{{$env.CONTENT_IDEAS_COLLECTION_ID}}"
  }
}
```

Use Raindrop as your:

- Content idea archive
- Competitor analysis storage
- Performance tracking (tag with metrics)

## Recommended Service Progression

### Phase 1 (Start Here - ~$50/month):

- Groq API (pay as you go, ~$10/month)
- Edge TTS or Google Cloud TTS (free)
- Webshare basic proxy ($30/month)
- Reddit JSON API (free)

### Phase 2 (After First Revenue - ~$120/month):

- Add ElevenLabs ($22/month)
- Add Ayrshare ($25/month)
- Upgrade proxy allocation

### Phase 3 (Scaling Up - ~$200/month):

- Add Creatomate ($41/month)
- Consider Gummysearch ($49/month)
- Multiple proxy providers for redundancy

## Groq-Specific Optimization Tips

### 1. Content Generation Prompt

```javascript
const systemPrompt = `You are a viral content scriptwriter. 
Output format: JSON with these exact fields:
{
  "hook": "3-second attention grabber",
  "script": "Main content 20-25 seconds",
  "cta": "Call to action 5 seconds",
  "hashtags": ["relevant", "trending", "tags"]
}
Rules:
- Use simple 5th-grade language
- One idea per sentence
- Create curiosity gaps
- End with a question`;
```

### 2. Speed Optimization

```javascript
// Parallel Groq requests for different platforms
const platforms = ["tiktok", "reels", "shorts"];
const requests = platforms.map((platform) => ({
  model: "gemma-7b-it", // Faster model for variations
  prompt: `Adapt this script for ${platform}: ${originalScript}`,
}));
```

### 3. Cost Management

```javascript
// Track token usage per workflow
const tokenCount = $json.content.length / 4; // Rough estimate
$node["Postgres"].executeQuery({
  query: "INSERT INTO api_usage (service, tokens, cost) VALUES ($1, $2, $3)",
  params: ["groq", tokenCount, tokenCount * 0.00000027],
});
```

## Account Creation & Warming Protocol

Since you're starting fresh:

### Week 1-2: Account Setup

1. Use residential proxy (different IP per platform)
2. Create accounts with unique device fingerprints
3. Complete all profile sections manually
4. Follow 5-10 accounts in your niche
5. Like/comment manually (2-3 per day)

### Week 3-4: Soft Automation

1. Schedule first posts (1 per day)
2. Use Raindrop to track what works
3. Gradually increase to 2-3 posts
4. Maintain manual engagement

### Week 5+: Full Automation

1. Deploy complete n8n workflows
2. 3-5 posts per day per platform
3. A/B test different hooks
4. Scale winning formulas

## Next Steps Action Plan

1. **This Week:**

   - Set up Groq API in n8n
   - Create Reddit scraper workflow
   - Test Edge TTS integration
   - Get Webshare proxy trial

2. **Next Week:**

   - Create first content generation workflow
   - Set up fresh social accounts (with proxy)
   - Build Raindrop integration for tracking

3. **Week 3:**
   - Add video creation (start with free tools)
   - Begin manual posting routine
   - Track metrics in PostgreSQL

Want me to create a specific workflow template for your first Groq + Reddit → Content pipeline?
