# N8N Content Arbitrage: Essential Architecture Patterns

## Core Architecture Pattern

```
[Trend Discovery] → [Content Generation] → [Distribution]
       ↓                    ↓                    ↓
  Sub-workflow         Sub-workflow         Sub-workflow
```

## Workflow Communication

Configure Execute Sub-workflow Trigger with typed inputs:

```json
{
  "inputDataMode": "Define using fields below",
  "fields": [
    { "name": "trend_data", "type": "object" },
    { "name": "target_platforms", "type": "array" },
    { "name": "content_type", "type": "string" }
  ]
}
```

## Essential Node Patterns

### 1. API Request Pattern (with Proxy)

```javascript
// HTTP Request node - Headers
{
  "Proxy-Authorization": "Basic {{$credentials.proxy.token}}",
  "User-Agent": "{{$json.randomUserAgent}}"
}

// Function node - Add jitter
const delay = 5000 + Math.random() * 10000; // 5-15 seconds
return { delay };
```

### 2. Batch Processing Pattern

```javascript
// Split In Batches node
{
  "batchSize": 10,
  "options": {
    "reset": false
  }
}
```

### 3. Error Handling Pattern

```javascript
// Error Workflow - Exponential backoff
const attempt = $item(0).$node["Error Trigger"].json.attempt || 1;
const delay = Math.min(1000 * Math.pow(2, attempt), 30000);
return { delay, attempt: attempt + 1 };
```

## Database Schema

```sql
CREATE TABLE content_pipeline (
  id SERIAL PRIMARY KEY,
  trend_id VARCHAR(255),
  content_title TEXT,
  content_status VARCHAR(50), -- 'discovered', 'generated', 'published'
  platforms JSONB,
  generated_at TIMESTAMP DEFAULT NOW(),
  published_at TIMESTAMP,
  performance_metrics JSONB
);

CREATE INDEX idx_status ON content_pipeline(content_status);
CREATE INDEX idx_trend ON content_pipeline(trend_id);
```

## Workflow Templates

### Template: Trend Monitor

```yaml
trigger: schedule_15min
nodes:
  - fetch_trends: HTTP_REQUEST
  - score_virality: FUNCTION
  - filter_threshold: IF
  - store_database: POSTGRES
  - trigger_generation: EXECUTE_WORKFLOW
```

### Template: Content Generator

```yaml
trigger: execute_workflow
nodes:
  - receive_trend: WORKFLOW_TRIGGER
  - generate_script: AI_NODE
  - create_media: HTTP_REQUEST
  - quality_check: FUNCTION
  - update_status: POSTGRES
```

### Template: Multi-Platform Distributor

```yaml
trigger: execute_workflow
nodes:
  - receive_content: WORKFLOW_TRIGGER
  - platform_router: SWITCH
  - platform_publishers: HTTP_REQUEST[]
  - update_metrics: POSTGRES
  - notify_complete: WEBHOOK
```

## Performance Optimizations

### Memory Management

- Process max 1000 items per workflow execution
- Use `.getBinaryData()` for media files
- Clear variables with `delete $input.item.json.largeData`

### API Rate Limiting

```javascript
// Wait node with dynamic timing
const platform = $json.platform;
const limits = {
  tiktok: 300000, // 5 minutes
  instagram: 180000, // 3 minutes
  youtube: 60000, // 1 minute
};
return { ms: limits[platform] || 120000 };
```

### Parallel Processing

- Use Split In Batches → Execute Workflow for parallel sub-workflows
- Limit concurrent executions: max 3 sub-workflows

## Compliance Patterns

### Human Behavior Simulation

```javascript
// Add to every social interaction
const baseDelay = 5000;
const jitter = Math.random() * 10000;
const hourOfDay = new Date().getHours();
const nightMultiplier = hourOfDay < 6 || hourOfDay > 23 ? 3 : 1;
return {
  waitTime: (baseDelay + jitter) * nightMultiplier,
};
```

### Content Uniqueness

```javascript
// Function node - Content variation
const variations = [
  { opening: "Did you know", closing: "Follow for more" },
  { opening: "Here's something wild", closing: "What do you think?" },
  { opening: "I just learned", closing: "Mind = blown" },
];
const selected = variations[Math.floor(Math.random() * variations.length)];
return { ...selected, timestamp: Date.now() };
```

## Quick Reference

**Node Execution Order:**

1. Trigger → 2. Validate → 3. Process → 4. Store → 5. Distribute

**Error Priority:**

1. API failures → Retry with backoff
2. Content violations → Skip and log
3. Rate limits → Queue for later

**Success Metrics:**

- Workflow execution time < 30s
- Error rate < 5%
- Content uniqueness > 70%
- Platform compliance 100%
