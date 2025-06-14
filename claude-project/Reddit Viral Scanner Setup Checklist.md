# Reddit Viral Scanner - Setup Checklist

## Prerequisites ✅

### 1. API Accounts & Keys

- [ ] **Groq API Key** - Get from [console.groq.com](https://console.groq.com)
  - Sign up, verify email, get API key
  - Free tier: 14,400 requests/day (~$10/month at scale)
- [ ] **Supabase Project** - Create at [supabase.com](https://supabase.com)
  - Create new project (free tier sufficient to start)
  - Note down your URL and anon key
- [ ] **Raindrop.io API** - Get from [raindrop.io/dev](https://raindrop.io/dev)
  - Create app, get access token
  - Create collection for "Viral Content Ideas"

### 2. N8N Setup (Self-hosted)

- [ ] N8N instance running (Docker/local)
- [ ] Access to n8n web interface
- [ ] Ability to add custom credentials

## Step-by-Step Setup

### Step 1: Database Setup

1. **Create Supabase Tables**
   - Copy the SQL schema from the "Supabase Schema" artifact
   - Go to Supabase Dashboard → SQL Editor
   - Paste and execute the schema
   - Verify tables created: `content_pipeline`, `api_usage`

### Step 2: N8N Credentials

1. **Add Groq Credentials**

   - Go to Settings → Credentials in n8n
   - Add new credential type: "Generic Credential"
   - Name it "groqApi"
   - Add field: `key` with your Groq API key

2. **Add Supabase Credentials**

   - Add Supabase credential
   - URL: Your Supabase project URL
   - Key: Your service key (or anon key for read/write)

3. **Add Raindrop Credentials**
   - Add HTTP Header Auth credential
   - Name: "raindropApi"
   - Header: "Authorization"
   - Value: "Bearer YOUR_RAINDROP_TOKEN"

### Step 3: Environment Variables

Add these to your n8n environment:

```bash
RAINDROP_COLLECTION_ID=your_collection_id
CONTENT_GENERATION_WORKFLOW_ID=workflow_id_when_created
```

### Step 4: Import Workflow

1. Copy the JSON from "Reddit Viral Scanner" artifact
2. In n8n: New Workflow → Import from JSON
3. Paste the JSON and import
4. Verify all nodes loaded correctly

### Step 5: Configuration

1. **Update Subreddit List** (Set Config node)

   - Modify the subreddits array for your niche
   - Popular options: `["AskReddit", "todayilearned", "LifeProTips", "unpopularopinion"]`

2. **Adjust Filters**

   - `minUpvotes`: Start with 50, increase as needed
   - `timeFilter`: "hour" for fresh content, "day" for more volume
   - `viral_score` threshold: 3+ for moderate, 5+ for high potential

3. **Test Individual Nodes**
   - Start with "Fetch Reddit Posts"
   - Verify JSON structure matches expectations
   - Test Groq API call with sample data

## Testing Phase

### Manual Test Run

1. **Disable Schedule Trigger**
   - Right-click "Every 15 Minutes" → Disable
2. **Add Manual Trigger**
   - Add "Manual Trigger" node
   - Connect to "Set Config"
3. **Execute & Debug**
   - Click "Execute Workflow"
   - Check each node output
   - Verify data flows correctly

### Verify Data Storage

```sql
-- Check Supabase data
SELECT * FROM content_pipeline ORDER BY created_at DESC LIMIT 5;

-- Check API usage
SELECT service, COUNT(*), SUM(cost_usd) FROM api_usage GROUP BY service;
```

### Check Raindrop Integration

- Go to your Raindrop collection
- Verify bookmarks are being added with correct tags
- Check metadata is populating

## Go Live Checklist

### Pre-Production

- [ ] All test runs successful
- [ ] Database connections stable
- [ ] API rate limits understood
- [ ] Error handling tested (try invalid Reddit URLs)
- [ ] Cost tracking working

### Production Deployment

- [ ] Enable 15-minute schedule trigger
- [ ] Monitor first few hours closely
- [ ] Set up alerts for failures
- [ ] Document any custom modifications

## Monitoring & Optimization

### Daily Checks (First Week)

```sql
-- Daily discovery stats
SELECT
  DATE(created_at) as date,
  COUNT(*) as discovered,
  AVG(final_score) as avg_score,
  MAX(final_score) as top_score
FROM content_pipeline
WHERE created_at >= CURRENT_DATE
GROUP BY DATE(created_at);

-- Top performing subreddits
SELECT subreddit, COUNT(*), AVG(final_score)
FROM content_pipeline
WHERE created_at >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY subreddit
ORDER BY AVG(final_score) DESC;
```

### Cost Monitoring

```sql
-- Weekly API costs
SELECT
  service,
  COUNT(*) as requests,
  SUM(cost_usd) as weekly_cost
FROM api_usage
WHERE created_at >= CURRENT_DATE - INTERVAL '7 days'
GROUP BY service;
```

## Next Steps After Setup

1. **Create Content Generation Workflow** (Phase 2)

   - Use trends from this workflow
   - Generate scripts with Groq
   - Create videos with selected tool

2. **Add Performance Tracking**

   - Update `performance_metrics` as content goes live
   - Track which subreddits generate best ROI

3. **Optimize Subreddit Selection**
   - A/B test different subreddit combinations
   - Focus on highest-converting sources

## Common Issues & Solutions

### Issue: Too Many Low-Quality Results

**Solution**: Increase `minUpvotes` and `viral_score` thresholds

### Issue: Groq API Rate Limits

**Solution**: Add delay between requests, consider upgrading plan

### Issue: Supabase Connection Errors

**Solution**: Check credentials, verify table permissions

### Issue: No Results from Reddit

**Solution**: Check subreddit names, verify Reddit isn't blocking requests (add delays)

### Issue: Raindrop Bookmarks Not Saving

**Solution**: Verify collection ID and API token permissions

## Performance Benchmarks

### Expected Results (First Week)

- **Posts Discovered**: 50-100 per day
- **High-Quality Candidates**: 5-15 per day (AI rating ≥ 6)
- **API Costs**: $2-5 per day (Groq + minimal other services)
- **Processing Time**: 2-5 minutes per 15-minute cycle

### Scaling Thresholds

- **When to add more subreddits**: Success rate > 80% with current list
- **When to tighten filters**: > 50 candidates per day
- **When to expand timeframe**: < 5 candidates per day

## Advanced Configurations

### Custom Viral Scoring Formula

Modify the `Process & Filter Posts` node to adjust the viral score calculation:

```javascript
// Enhanced viral scoring
viral_score: Math.log(post.ups + 1) * 2 + // Upvotes weight
  Math.log(post.num_comments + 1) * 1.5 + // Comments weight
  post.ups / Math.max(hoursOld, 1) + // Velocity factor
  (post.num_comments / post.ups) * 0.5; // Engagement ratio
```

### Platform-Specific Filtering

Add platform preferences to the Groq prompt:

```json
"Prioritize content suitable for: TikTok (trends, challenges), Instagram Reels (lifestyle, tips), YouTube Shorts (educational, surprising facts). Rate each platform 1-10 for suitability."
```

## Ready to Launch! 🚀

Once you complete this checklist:

1. Your Reddit Viral Scanner will run every 15 minutes
2. High-potential content gets stored in Supabase
3. Ideas are bookmarked in Raindrop for easy access
4. You'll have data to build your content generation workflow

**Next chat topic**: "Create the Groq Script Generator workflow to convert discovered trends into platform-optimized video scripts"
