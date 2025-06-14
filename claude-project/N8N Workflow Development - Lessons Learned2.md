# N8N Workflow Development - Lessons Learned

## Project Context

Building automated content arbitrage workflows: Reddit discovery → Groq AI analysis → Multi-platform distribution

- **User**: Staff software engineer, intermediate n8n knowledge
- **Goal**: Income generation through automated short-form video content
- **Stack**: Groq (not OpenAI), Supabase (not PostgreSQL), self-hosted n8n

## Critical Technical Lessons

### 1. N8N Set Node Structure (Major Issue Fixed)

**Problem**: Set node assignments appearing blank when importing JSON workflows.

**Root Cause**: Incorrect JSON structure for Set node parameters.

**Correct Structure** (n8n Set node v3.4+):

```json
{
  "parameters": {
    "assignments": {
      "assignments": [
        {
          "id": "unique-identifier",
          "name": "fieldName",
          "value": "fixedValue", // or "={{ expression }}" for expressions
          "type": "string|number|array|object"
        }
      ]
    },
    "includeOtherFields": false,
    "options": {}
  },
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4
}
```

**Key Requirements**:

- Each assignment needs unique `id` field
- Must include `includeOtherFields` and `options` parameters
- Use current `typeVersion` (3.4+ as of 2025)
- Arrays/expressions: `"value": "=[...]"` or `"value": "={{ expression }}"`
- Fixed strings: `"value": "simpleString"` (no expression wrapper)
- Numbers: `"value": "=50"` (expression format)

### 2. Environment Variables vs Workflow Configuration

**Problem**: User correctly identified that hardcoded environment variables are too rigid.

**Better Approach**: Store configuration in workflow using Set node:

- ✅ No server restarts needed
- ✅ Multiple environments/versions possible
- ✅ Self-documenting workflows
- ✅ Version controlled with workflow

**Implementation**: Use Set Config node with assignments for:

- API endpoints
- Collection IDs
- Workflow IDs
- Thresholds and filters

### 3. N8N Workflow Template Research Strategy

**Effective Sources**:

1. **n8n.io/workflows** - Official community templates
2. **GitHub issues** - Real-world node structure examples
3. **n8n documentation** - Current node parameters

**Search Patterns That Work**:

- `site:n8n.io/workflows [technology]` - Find relevant templates
- `"n8n-nodes-base.[nodetype]" JSON` - Find structure examples
- n8n GitHub issues for complex node configurations

### 4. Groq API Integration Patterns

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

### 5. Database Schema Design for Content Pipeline

**Key Tables**:

- `content_pipeline` - Main workflow tracking
- `api_usage` - Cost monitoring from day one

**Critical Fields**:

- `content_status` - Pipeline stage tracking
- `performance_metrics` - JSONB for flexible metrics
- Proper indexing on status, scores, timestamps

### 6. Workflow Architecture Patterns

**Modular Design**:

- Separate workflows for: Discovery → Generation → Distribution
- Use Execute Workflow nodes for communication
- Store intermediate data in database, not memory

**Error Handling**:

- Add IF nodes for conditional execution
- Implement try/catch in Code nodes
- Log failures to database for monitoring

## Development Workflow Best Practices

### 1. Template Adaptation Process

1. **Research existing templates** on n8n.io/workflows first
2. **Identify closest match** to required functionality
3. **Verify node structures** against current n8n version
4. **Adapt step-by-step** rather than building from scratch
5. **Test individual nodes** before full workflow execution

### 2. Configuration Management

- **Always use Set nodes** for workflow configuration
- **Avoid environment variables** for workflow-specific settings
- **Make configuration visible** in the workflow itself
- **Use meaningful field names** and comments

### 3. Testing Strategy

- **Disable schedule triggers** during development
- **Add manual triggers** for testing
- **Test node-by-node** before connecting full pipeline
- **Verify data structure** at each step

### 4. JSON Import/Export Issues

**Common Problems**:

- Node versions outdated in exported JSON
- Missing required parameters in node configurations
- Incorrect value formatting (expressions vs fixed)

**Solutions**:

- Always check typeVersion against current n8n
- Verify all required parameters using documentation
- Test import immediately after creation

## Stack-Specific Configurations

### Groq API (Not OpenAI)

```json
{
  "url": "https://api.groq.com/openai/v1/chat/completions",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "groqApi"
}
```

### Supabase (Not PostgreSQL)

```json
{
  "type": "n8n-nodes-base.supabase",
  "operation": "insert",
  "table": { "value": "table_name", "mode": "list" }
}
```

### Raindrop.io Integration

```json
{
  "url": "https://api.raindrop.io/rest/v1/raindrop",
  "authentication": "predefinedCredentialType",
  "nodeCredentialType": "raindropApi"
}
```

## Next Session Preparation

### What to Remember:

1. **User prefers workflow-level configuration** over environment variables
2. **Always verify Set node structure** against current n8n version
3. **Research n8n.io/workflows first** for similar templates
4. **Test imports immediately** - don't assume JSON will work
5. **User is building for income generation** - focus on practical implementation

### Common Mistakes to Avoid:

- ❌ Using outdated node typeVersions
- ❌ Missing required node parameters
- ❌ Hardcoding values that should be configurable
- ❌ Not testing individual nodes before full workflow
- ❌ Assuming JSON exports will import cleanly

### Files in This Project:

- `N8N Content Arbitrage Project - Progress Summary & Next Steps.md` - Overall project status
- `Reddit Viral Scanner - Groq + Supabase Workflow.txt` - Working workflow JSON
- `Reddit Viral Scanner Setup Checklist.md` - Implementation guide
- This file - Technical lessons and patterns

## Success Metrics for Next Phase:

- Workflow imports without errors
- All nodes execute successfully
- Configuration is easily modifiable
- Cost tracking works from day one
- Ready to build content generation workflow
