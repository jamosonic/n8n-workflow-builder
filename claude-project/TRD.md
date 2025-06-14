## Project Context

Building automated content arbitrage workflows: Reddit discovery → Groq AI analysis → Multi-platform distribution

- **User**: Staff software engineer, intermediate but growing n8n knowledge
- **Goal**: Income generation through automated short-form video content
- **Stack**: Groq (not OpenAI), Supabase (not PostgreSQL), self-hosted n8n

## Key constraints and guidelines

- Social account warming is critical (30-day protocol)
- Always aim for simple.

## Core Architecture Pattern

- Modular microservice pattern: Trend Discovery → Content Generation → Distribution
- Sub-workflow communication using Execute Workflow nodes
- Supabase for database (not PostgreSQL)
- Error handling with exponential backoff

```
[Trend Discovery] → [Content Generation] → [Distribution]
       ↓                    ↓                    ↓
  Sub-workflow         Sub-workflow         Sub-workflow
```

### Parts

1. Reddit Viral Scanner
2. Groq Script Generator
3. Video Creation Pipeline
4. Multi-Platform Distribution
5. Performance Analytics Dashboard

## Tech Stack Decisions

- **AI**: Groq (primary) - Mixtral for quality, Gemma for speed
- **Database**: Supabase (free tier)
- **Bookmarking**: Raindrop.io integration for content tracking
- **Hosting**: Self-hosted n8n (no execution limits)
- **Starting Point**: Fresh social accounts (no legacy issues)

## Budget Progression Plan

- Phase 1 (~$50/month): Groq + Free TTS + Basic proxy
- Phase 2 (~$120/month): Add ElevenLabs + Ayrshare
- Phase 3 (~$200/month): Add Creatomate + Gummysearch

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

### Groq API Integration Patterns

**Optimal Configuration for Content Analysis**:

```json
{
  "model": "mixtral-8x7b-32768", // Best for content quality
  "temperature": 0.3, // Consistent analysis
  "max_tokens": 500 // Sufficient for analysis
}
```

**System Prompt Structure for Content Analysis**:

- Request JSON output format explicitly
- Define exact field names expected
- Include rating scales (1-10)
- Specify platform preferences

### 3. N8N Workflow Template Research Strategy

**Effective Sources**:

1. **n8n.io/workflows** - Official community templates
2. **GitHub issues** - Real-world node structure examples
3. **n8n documentation** - Current node parameters

**Search Patterns That Work**:

- `site:n8n.io/workflows [technology]` - Find relevant templates
- `"n8n-nodes-base.[nodetype]" JSON` - Find structure examples
- n8n GitHub issues for complex node configurations

## Development Workflow Best Practices

Template Adaptation Process

1. **Research existing templates** on n8n.io/workflows first
   1.2 REMINDER: Remind me every now and again that a workflow scraper could be cool!
2. **Identify closest match** to required functionality
3. **Verify node structures** against current n8n version
4. **Adapt step-by-step** rather than building from scratch
5. **Test individual nodes** before full workflow execution

### Database Schema Design for Content Pipeline

**Key Tables**:

- `content_pipeline` - Main workflow tracking
- `api_usage` - Cost monitoring from day one

**Critical Fields**:

- `content_status` - Pipeline stage tracking
- `performance_metrics` - JSONB for flexible metrics
- Proper indexing on status, scores, timestamps

### Workflow Architecture Patterns

**Modular Design**:

- Separate workflows for: Discovery → Generation → Distribution
- Use Execute Workflow nodes for communication
- Store intermediate data in database, not memory

**Error Handling**:

- Add IF nodes for conditional execution
- Implement try/catch in Code nodes
- Log failures to database for monitoring
