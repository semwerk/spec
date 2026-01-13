# Field Usage Guide

**Version:** 1.0.0
**Status:** Draft

<!--semcontext:segment start key="overview" type="overview" audience="developer,product"-->
## Overview

Comprehensive guide to every field in Semwerk specifications: what it is, why you need it, how to use it, what outcome you achieve, and when to use it.

**Audience:**
- Developers implementing Semwerk parsers
- Product teams authoring content
- Engineering teams integrating Semwerk

**Structure:**
- **Concept** - What the field represents
- **Task** - What specific job it helps you accomplish
- **Why** - The problem it solves
- **How** - Practical implementation examples
- **Outcome** - What you achieve by using it
- **Situations** - All possible scenarios where you'd use it
<!--semcontext:segment end-->

<!--semcontext:segment start key="segment-id" type="reference" audience="developer"-->
## Segment ID

### Concept
Unique identifier for a content segment within a document.

### Task
**Primary:** Uniquely identify and reference specific sections of content
**Secondary:** Enable precise linking from code to specific doc sections

### Why You Need It
**Problem:** Without IDs, you can only reference entire documents
**Solution:** Granular references to specific sections (e.g., "OAuth setup section" not "entire auth guide")

### How to Use It

**Inline marker:**
```markdown
<!--semcontext:segment start key="oauth-setup"-->
## OAuth Setup
...
<!--semcontext:segment end-->
```

**Frontmatter:**
```yaml
semcontext:
  segments:
    - id: oauth-setup
```

**External annotation:**
```json
{
  "segments": [
    {"id": "oauth-setup"}
  ]
}
```

**Referencing:**
```yaml
# In journey node
content_refs:
  - content:@doc-auth-guide/oauth-setup

# In linkage mapping
code_to_assets:
  "oauth.go:Authenticate":
    assets:
      - path: docs/auth.md
        segments:
          - id: oauth-setup
```

### Outcome
- **Precise documentation linking** - Code references exact section, not whole doc
- **Stable references** - ID doesn't change if heading text changes
- **Programmatic access** - Tools can fetch specific segments
- **Context-aware retrieval** - RAG systems retrieve relevant section only

### Situations

**When to use:**
1. **Code-to-doc linking** - Link `Login()` function to "Authentication" section
2. **Multi-audience docs** - Same doc, different sections for different roles
3. **RAG/LLM context** - Retrieve specific section for AI context
4. **Documentation updates** - Track which sections changed
5. **Translation** - Translate section-by-section
6. **A/B testing** - Test different versions of specific sections
7. **Analytics** - Track which sections users read
8. **Search** - Return section-level results, not entire docs
9. **Versioning** - Version specific sections independently
10. **Reuse** - Include same section in multiple documents

**When NOT to use:**
- Document has no logical sections
- Entire document is single concept (no subdivision needed)
- Temporary scratch notes

**Best practices:**
- Use kebab-case: `oauth-setup`, `getting-started`, `api-reference`
- Be specific: `stripe-webhook-setup` not `setup`
- Keep stable: Don't change IDs when refactoring content
- Use hierarchy: `auth.oauth.setup` for nested concepts (in concepts field, not ID)
<!--semcontext:segment end-->

<!--semcontext:segment start key="segment-type" type="reference" audience="developer"-->
## Segment Type

### Concept
Classification of content segment (overview, howto, reference, tutorial, etc.)

### Task
**Primary:** Classify content for appropriate retrieval and presentation
**Secondary:** Filter content by type, apply type-specific rendering

### Why You Need It
**Problem:** All content treated the same, can't optimize for use case
**Solution:** Tutorials get step-by-step UI, reference gets compact display, troubleshooting gets problem-solution format

### How to Use It

**Inline marker:**
```markdown
<!--semcontext:segment start key="setup" type="howto"-->
```

**Frontmatter:**
```yaml
segments:
  - id: setup
    type: howto
```

**14 standard types:**
- `overview` - High-level introduction
- `howto` - Task-oriented guide
- `reference` - API/technical reference
- `tutorial` - Learning-oriented walkthrough
- `troubleshooting` - Problem-solution guide
- `faq` - Frequently asked questions
- `example` - Code example
- `conceptual` - Concept explanation
- `requirements` - Requirements specification
- `rfc` - Design proposal
- `meeting` - Meeting notes
- `policy` - Policy documentation
- `review` - Review/postmortem
- `roadmap` - Future planning

**See:** [content:@doc-segments/types](#) for full taxonomy

### Outcome
- **Better search results** - Return tutorials for "how to" queries, reference for API lookups
- **Appropriate UI** - Tutorials get interactive elements, reference gets search/filter
- **Role-based filtering** - Show beginners tutorials, advanced users reference
- **Content strategy** - Measure coverage (do we have enough tutorials?)
- **LLM prompting** - Different prompts for different types

### Situations

**When to use:**
1. **Beginner onboarding** - Type: `tutorial`, walk through step-by-step
2. **API documentation** - Type: `reference`, structured lookup tables
3. **Troubleshooting errors** - Type: `troubleshooting`, symptom → solution
4. **Quick answers** - Type: `faq`, common questions answered
5. **Learning concepts** - Type: `conceptual`, explain the "why"
6. **Product decisions** - Type: `rfc`, document architecture choices
7. **Compliance** - Type: `policy`, formal policies and procedures
8. **Project planning** - Type: `roadmap`, future direction
9. **Code examples** - Type: `example`, copy-paste snippets
10. **High-level intro** - Type: `overview`, what is this product

**Type determines presentation:**
- **tutorial** → Interactive checklist UI, progress tracking
- **howto** → Step numbers, prerequisites, estimated time
- **reference** → Search box, filterable tables, quick navigation
- **troubleshooting** → Accordion with symptoms, search by error message
- **example** → Syntax highlighting, copy button, run in playground
- **faq** → Accordion, search, vote helpful/not helpful
- **overview** → Hero section, key features, quick links
- **conceptual** → Diagrams, illustrations, progressive disclosure

**When NOT to use:**
- Content is unstructured (use generic type or none)
- Mixing multiple types in one segment (split into multiple segments)

**Anti-patterns:**
- Using `reference` for tutorials (confuses users)
- Using `tutorial` for API reference (wrong UI)
- Inconsistent types (same content, different types in different docs)
<!--semcontext:segment end-->

<!--semcontext:segment start key="audience-role" type="reference" audience="developer"-->
## Audience Role

### Concept
Target user role(s) for content segment.

### Task
**Primary:** Filter content by user role for personalization
**Secondary:** Measure content coverage per role, identify gaps

### Why You Need It
**Problem:** Developers see end-user content, beginners see advanced content, everyone confused
**Solution:** Show each role only their relevant content

### How to Use It

**Inline marker:**
```markdown
<!--semcontext:segment start key="api-ref" type="reference" audience="developer"-->
```

**Multiple audiences:**
```markdown
<!--semcontext:segment start key="deploy" type="howto" audience="developer,operator"-->
```

**Frontmatter:**
```yaml
segments:
  - id: api-ref
    audience_role: developer
  - id: deployment
    audience_role: [developer, operator]
```

**10 standard roles:**
- `user` - End users
- `developer` - Software developers
- `operator` - DevOps/SRE
- `admin` - System administrators
- `security` - Security engineers
- `product` - Product managers
- `engineering` - Engineering managers
- `executive` - C-level
- `legal` - Legal/compliance
- `internal` - Internal only

**See:** [content:@doc-audience-roles/taxonomy](#)

### Outcome
- **Personalized portals** - Developer portal shows only dev content
- **Reduced cognitive load** - Users only see what's relevant
- **Better search** - Filter by "show me developer docs only"
- **Gap analysis** - "We have 50 dev docs but only 5 for operators"
- **Access control** - Internal content hidden from external users
- **RAG optimization** - Only retrieve content for user's role

### Situations

**When to use:**

1. **Multi-role product (SaaS platform)**
   - Task: Different roles need different docs
   - Outcome: Personalized documentation experience
   - Example: Stripe docs (developers vs. business owners vs. compliance)

2. **API documentation**
   - Task: Separate API reference from end-user guides
   - Outcome: Developers find technical docs, users find how-tos
   - Example: `audience: developer` for API ref, `audience: user` for UI guide

3. **Enterprise software**
   - Task: Role-based content access
   - Outcome: Operators see deployment guides, users see feature guides
   - Example: Kubernetes docs (cluster admins vs. app developers)

4. **Internal tooling**
   - Task: Hide internal docs from external users
   - Outcome: Secrets/internal processes stay private
   - Example: `audience: internal` for runbooks, `audience: user` for public docs

5. **Compliance documentation**
   - Task: Legal needs compliance docs, devs need technical specs
   - Outcome: Each role gets their required reading
   - Example: `audience: legal` for GDPR, `audience: developer` for API security

6. **Onboarding flows**
   - Task: Different paths for different experience levels
   - Outcome: Beginners get tutorials, advanced get reference
   - Example: Use with journey decision nodes (experience_level routing)

7. **Support documentation**
   - Task: Support team needs troubleshooting, users need how-tos
   - Outcome: Appropriate content for each support scenario
   - Example: `audience: support` for internal procedures

8. **Sales enablement**
   - Task: Sales needs product positioning, users need features
   - Outcome: Sales gets battlecards, users get feature docs
   - Example: `audience: product` for positioning docs

9. **Multi-tenant SaaS**
   - Task: Different customer tiers see different features
   - Outcome: Enterprise customers see advanced features
   - Example: `audience: [developer, enterprise-admin]`

10. **Localized content**
    - Task: Different regions have different roles
    - Outcome: EU admins see GDPR, US admins see HIPAA
    - Example: Combined with content versioning

**Multiple audiences:**
```yaml
# Both developers and operators need this
audience_role: [developer, operator]

# Use case: Deployment guide relevant to both
```

**Role hierarchy:**
```
Technical roles:
  - developer → operator → admin → security

Business roles:
  - user → product → engineering → executive
```

**When NOT to use:**
- Content is universal (everyone needs it)
- Single-role product (only developers use it)
- Public marketing site (no role distinction)

**Anti-patterns:**
- Too many roles on one segment (probably needs splitting)
- Inconsistent role naming (use taxonomy)
- Using roles for authorization (use separate access control system)
<!--semcontext:segment end-->

<!--semcontext:segment start key="concepts" type="reference" audience="developer"-->
## Concepts

### Concept
Semantic tags representing topics, technologies, or domains covered in content.

### Task
**Primary:** Enable concept-based search and retrieval
**Secondary:** Build knowledge graph of how topics relate

### Why You Need It
**Problem:** Keyword search misses synonyms, related topics, semantic meaning
**Solution:** Tag content with concepts, find all "authentication" content even if keyword isn't present

### How to Use It

**Inline (not directly supported, use frontmatter/external):**
- Inline markers only support key, type, audience
- For concepts, use frontmatter or external annotations

**Frontmatter:**
```yaml
segments:
  - id: oauth-setup
    concepts: [oauth, authentication, security]
    # or single concept
    concepts: authentication
```

**External annotation:**
```json
{
  "segments": [
    {
      "id": "oauth-setup",
      "concepts": ["oauth", "authentication", "security"]
    }
  ]
}
```

**Concept taxonomy:**
```yaml
# Create concept definitions
concepts:
  - id: cpt_oauth
    key: oauth
    name: "OAuth 2.0"
    relationships:
      - to: concept:@cpt_authentication
        kind: parent
```

**See:** [content:@doc-concept-definition/overview](#)

### Outcome
- **Semantic search** - Find all auth content even if "authentication" keyword not present
- **Related content** - "If reading about OAuth, also show JWT content"
- **Concept coverage** - "We have 20 docs about payments, only 2 about refunds"
- **Knowledge graph** - Visualize how topics connect
- **Contextual RAG** - Retrieve based on semantic similarity, not just keywords
- **Content strategy** - Identify concept gaps, plan new content

### Situations

**When to use:**

1. **Large documentation sites (100+ docs)**
   - Task: Help users discover related content
   - Outcome: Users find answers across related topics
   - Example: Searching "JWT" also shows "OAuth", "authentication", "security"

2. **Multi-product platforms**
   - Task: Tag content with product features
   - Outcome: "Show me all docs related to payments feature"
   - Example: Concepts: `[payments, webhooks, api, stripe-integration]`

3. **Technical documentation**
   - Task: Connect technology stack topics
   - Outcome: Reading about Postgres triggers docs about caching, indexing
   - Example: Concepts: `[postgres, database, sql, caching, indexing]`

4. **Compliance and security**
   - Task: Group all security-related content
   - Outcome: Audit shows all GDPR/PCI/SOC2 docs
   - Example: Concepts: `[gdpr, pci-dss, compliance, security]`

5. **Onboarding journeys**
   - Task: Progressive disclosure based on concepts learned
   - Outcome: Show advanced concepts only after basics
   - Example: Journey checks if user has seen `[authentication]` concept

6. **Content recommendations**
   - Task: Suggest related reading
   - Outcome: "Since you read about X, you might like Y"
   - Example: Read OAuth → Suggest JWT, API keys, SAML

7. **RAG context building**
   - Task: Retrieve semantically related content
   - Outcome: LLM gets comprehensive context about topic
   - Example: Query "How do I authenticate?" → Retrieve all `[authentication, oauth, jwt]` segments

8. **Documentation coverage analysis**
   - Task: Measure concept coverage
   - Outcome: Report showing which concepts well-documented vs. sparse
   - Example: "Payments: 25 segments, Refunds: 2 segments (gap!)"

9. **Multi-language documentation**
   - Task: Tag language-agnostic concepts
   - Outcome: Find "database migration" docs regardless of tech stack
   - Example: Concept `database-migration` applies to Go, Python, Node.js docs

10. **Feature flag documentation**
    - Task: Tag content by feature
    - Outcome: Show/hide docs based on enabled features
    - Example: Concept `[feature-oauth-pkce]` only shown if feature enabled

**Concept granularity:**
```yaml
# Too broad (not useful)
concepts: [documentation]

# Good (specific, actionable)
concepts: [oauth, pkce-flow, authorization-code-grant]

# Too specific (over-tagging)
concepts: [oauth, oauth-2.0, oauth2, authorization, delegated-auth, ...]
```

**Concept relationships:**
```yaml
# Parent-child
oauth (parent) ← jwt, saml, api-keys

# Related
authentication ←→ authorization ←→ security

# Implements
jwt (implements) → oauth

# Depends on
payment-processing (depends on) → authentication
```

**When NOT to use:**
- Content is too general (whole site about one concept)
- Concepts are obvious from title/content
- Over-tagging (10+ concepts on one segment)

**Anti-patterns:**
- Using concepts as keywords (duplicating title words)
- Inconsistent concept naming (`oauth` vs. `OAuth` vs. `oauth2`)
- Not using concept relationships (flat taxonomy)
<!--semcontext:segment end-->

<!--semcontext:segment start key="boost" type="reference" audience="developer"-->
## Boost

### Concept
Relevance multiplier for ranking/scoring in search and retrieval (0-10, default: 1.0).

### Task
**Primary:** Prioritize important content in search results
**Secondary:** Control RAG retrieval ranking

### Why You Need It
**Problem:** All content ranked equally, critical sections buried in results
**Solution:** Boost critical sections (e.g., 1.5×), downrank deprecated content (e.g., 0.5×)

### How to Use It

**Frontmatter:**
```yaml
segments:
  - id: critical-security-warning
    type: troubleshooting
    boost: 2.0        # 2× more important
    concepts: [security, vulnerability]

  - id: deprecated-api-v1
    type: reference
    boost: 0.3        # 70% less relevant
    concepts: [api, deprecated]
```

**External annotation:**
```json
{
  "segments": [
    {
      "id": "getting-started",
      "boost": 1.8,
      "concepts": ["onboarding"]
    }
  ]
}
```

**Scoring calculation:**
```typescript
// Base semantic similarity score
const baseScore = cosineSimilarity(query, segment.content);

// Apply boost
const finalScore = baseScore * (segment.boost || 1.0);

// Rank by final score
segments.sort((a, b) => b.finalScore - a.finalScore);
```

### Outcome
- **Critical content surfaces first** - Security warnings, breaking changes prioritized
- **Deprecated content downranked** - Old APIs shown last
- **Better user experience** - Most relevant answers first
- **Cost optimization** - Send highest-boost content to LLM (best signal/token ratio)
- **A/B testing** - Temporarily boost new content to measure engagement

### Situations

**When to use:**

1. **Security warnings**
   - Task: Ensure users see critical security info
   - Outcome: Security warnings appear first in search
   - Example: `boost: 2.5` for "SQL injection prevention" section

2. **Breaking changes**
   - Task: Highlight migration guides
   - Outcome: Developers find migration path immediately
   - Example: `boost: 2.0` for "Migrating from v1 to v2"

3. **Getting started**
   - Task: Surface onboarding content for new users
   - Outcome: New users see quick start before advanced topics
   - Example: `boost: 1.8` for "5-minute quick start"

4. **Deprecated content**
   - Task: Downrank old APIs while keeping docs available
   - Outcome: Old API docs findable but not prominent
   - Example: `boost: 0.3` for "API v1 (deprecated)"

5. **Seasonal content**
   - Task: Temporarily boost relevant content
   - Outcome: Holiday promotions surfaced in December
   - Example: `boost: 1.5` for "Holiday API rate limits" (Dec only)

6. **Popular questions**
   - Task: Surface frequently asked questions
   - Outcome: Reduce support tickets
   - Example: `boost: 1.6` for "Why am I getting 401 errors?"

7. **New features**
   - Task: Promote newly launched features
   - Outcome: Drive adoption of new capabilities
   - Example: `boost: 1.7` for "New: Webhooks v2" (first 30 days)

8. **Context-specific importance**
   - Task: Different boosts for different user contexts
   - Outcome: Advanced users see advanced content first
   - Example: Boost "Advanced OAuth" for users who completed basics

9. **Error documentation**
   - Task: Prioritize common error messages
   - Outcome: Faster troubleshooting
   - Example: `boost: 1.8` for "Error: Invalid API key"

10. **Compliance requirements**
    - Task: Ensure required reading surfaces first
    - Outcome: Users see mandatory policies
    - Example: `boost: 2.2` for "PCI DSS Requirements (Required)"

**Boost ranges:**
```yaml
2.5 - 3.0:  Critical (security, breaking changes)
1.8 - 2.2:  Very important (onboarding, common errors)
1.3 - 1.7:  Important (new features, popular content)
1.0:        Normal (default)
0.6 - 0.9:  Less important (edge cases, advanced topics)
0.3 - 0.5:  Deprecated (keep for search, but don't prioritize)
< 0.3:      Hidden (consider removing)
```

**Dynamic boosting:**
```typescript
// Adjust boost based on context
function getDynamicBoost(segment, userContext) {
  let boost = segment.boost || 1.0;

  // Boost beginner content for new users
  if (userContext.daysActive < 7 && segment.type === 'tutorial') {
    boost *= 1.5;
  }

  // Downrank for experienced users
  if (userContext.daysActive > 90 && segment.type === 'tutorial') {
    boost *= 0.7;
  }

  // Boost seasonal content
  if (isHolidaySeason() && segment.concepts.includes('holiday-api')) {
    boost *= 1.4;
  }

  return boost;
}
```

**When NOT to use:**
- All content equally important
- Single-segment documents
- Can't justify why one section more important

**Anti-patterns:**
- Everything boost 2.0 (defeats purpose)
- Boost 0.0 (just delete it)
- Inconsistent boosting (similar sections, vastly different boosts)
<!--semcontext:segment end-->

<!--semcontext:segment start key="max-tokens" type="reference" audience="developer"-->
## Max Tokens (return.max_tokens)

### Concept
Maximum tokens to retrieve/return when fetching segment for LLM context.

### Task
**Primary:** Prevent any single segment from consuming entire context window
**Secondary:** Control costs (fewer tokens = lower cost)

### Why You Need It
**Problem:** One huge segment uses 6K of your 8K context, no room for other segments
**Solution:** Limit each segment to 800 tokens, fit 10 segments in 8K context

### How to Use It

**Frontmatter:**
```yaml
segments:
  - id: overview
    return:
      max_tokens: 800      # Truncate if longer

  - id: detailed-api-ref
    return:
      max_tokens: 1500     # Allow longer for comprehensive reference
```

**External annotation:**
```json
{
  "segments": [
    {
      "id": "overview",
      "return": {"max_tokens": 800}
    }
  ]
}
```

**Retrieval logic:**
```typescript
function retrieveSegment(segmentId: string, maxTokens: number) {
  let content = getSegmentContent(segmentId);

  const tokenCount = countTokens(content);
  if (tokenCount > maxTokens) {
    // Truncate to max_tokens
    content = truncateToTokens(content, maxTokens);
  }

  return content;
}
```

### Outcome
- **Predictable context usage** - Know exactly how much context each segment uses
- **Cost control** - Limit token consumption = control API costs
- **Fit more segments** - More segments in context = better answers
- **No context overflow** - Never exceed model's max context
- **Fair distribution** - Each segment gets proportional share of context

### Situations

**When to use:**

1. **RAG systems with limited context**
   - Task: Fit multiple segments in 8K context window
   - Outcome: 10 segments × 800 tokens = 8K, comprehensive context
   - Example: GPT-3.5 (8K context) with 10 relevant segments

2. **Cost optimization**
   - Task: Reduce API costs for high-volume queries
   - Outcome: Smaller context = lower cost per query
   - Example: 500 tokens/segment instead of 2000 = 4× cheaper

3. **Quick answers**
   - Task: Fast responses with minimal context
   - Outcome: Low-latency, focused answers
   - Example: FAQ segments with 300 token limit

4. **Multi-segment queries**
   - Task: Retrieve many segments for comprehensive answer
   - Outcome: 15 small segments vs. 3 large segments
   - Example: "How does authentication work?" → 15 auth-related segments

5. **Mobile/edge deployment**
   - Task: Smaller models with tiny context windows
   - Outcome: Works with 2K context models
   - Example: Edge LLM with 2K context, 400 token segments

6. **Streaming responses**
   - Task: Start responding quickly
   - Outcome: Shorter segments = faster time-to-first-token
   - Example: 500 token segments stream faster than 2000

7. **Content quality control**
   - Task: Enforce segment size limits
   - Outcome: Segments stay focused and concise
   - Example: Max 1000 tokens = forces clear, concise writing

8. **A/B testing context sizes**
   - Task: Test different context amounts
   - Outcome: Find optimal segment size for quality vs. cost
   - Example: Test 500 vs. 1000 vs. 1500 token segments

9. **Role-based context limits**
   - Task: Give different context amounts per role
   - Outcome: Developers get detailed (1500), users get concise (500)
   - Example: Same segment, different max_tokens by audience

10. **Summarization pipelines**
    - Task: Limit input to summarization model
    - Outcome: Consistent summary lengths
    - Example: Max 2000 tokens input → 200 token summary

**Typical ranges:**
```yaml
300-500:   Quick answers, FAQs, simple how-tos
600-800:   Standard segments, balanced detail
1000-1500: Comprehensive guides, API reference
1500-2000: In-depth tutorials, complex topics
2000+:     Full specifications, detailed RFCs
```

**Dynamic limits:**
```typescript
// Adjust based on context
function getEffectiveMaxTokens(segment, context) {
  let maxTokens = segment.return.max_tokens;

  // Reduce for mobile
  if (context.device === 'mobile') {
    maxTokens = Math.min(maxTokens, 500);
  }

  // Reduce for quick answers
  if (context.queryType === 'quick-answer') {
    maxTokens = Math.min(maxTokens, 400);
  }

  // Increase for research mode
  if (context.mode === 'research') {
    maxTokens = Math.min(maxTokens * 1.5, 3000);
  }

  return maxTokens;
}
```

**When NOT to use:**
- Segment already very short (< 100 tokens)
- Unlimited context budget
- Non-LLM use cases (display only)

**Anti-patterns:**
- max_tokens < segment actual size (constant truncation, missing context)
- max_tokens too large (defeats purpose)
- Same max_tokens for all segments (some need more, some less)
<!--semcontext:segment end-->

<!--semcontext:segment start key="min-tokens" type="reference" audience="developer"-->
## Min Tokens (return.min_tokens)

### Concept
Minimum tokens required when retrieving segment - skip if segment is too short.

### Task
**Primary:** Enforce quality threshold - reject segments with insufficient content
**Secondary:** Filter out stub sections, placeholder content

### Why You Need It
**Problem:** RAG retrieves tiny, useless snippets that waste context
**Solution:** Skip segments < 200 tokens, ensure meaningful content only

### How to Use It

**Frontmatter:**
```yaml
segments:
  - id: comprehensive-guide
    return:
      max_tokens: 1500
      min_tokens: 400      # Must have at least 400 tokens

  - id: quick-tip
    return:
      max_tokens: 300
      min_tokens: 50       # Can be as short as 50 tokens
```

**Retrieval logic:**
```typescript
function retrieveSegment(segmentId: string) {
  let content = getSegmentContent(segmentId);
  const tokenCount = countTokens(content);

  const minTokens = segment.return.min_tokens || 0;
  const maxTokens = segment.return.max_tokens || Infinity;

  // Skip if too short
  if (tokenCount < minTokens) {
    console.log(`Skipping ${segmentId}: ${tokenCount} < ${minTokens} tokens`);
    return null;
  }

  // Truncate if too long
  if (tokenCount > maxTokens) {
    content = truncateToTokens(content, maxTokens);
  }

  return content;
}
```

### Outcome
- **Higher quality results** - No stub sections in LLM context
- **Better signal/noise** - Only substantive content retrieved
- **Fewer wasted tokens** - Don't pay for 50-token useless snippets
- **Consistent minimum detail** - All retrieved content has sufficient depth
- **Filter incomplete content** - Identify sections needing expansion

### Situations

**When to use:**

1. **Quality control for RAG**
   - Task: Ensure LLM gets meaningful context
   - Outcome: No "TODO: Write this section" in results
   - Example: `min_tokens: 200` filters out stubs

2. **Comprehensive answers only**
   - Task: User queries need detailed responses
   - Outcome: Return complete explanations, not snippets
   - Example: `min_tokens: 500` for architecture docs

3. **Filter work-in-progress**
   - Task: Hide incomplete sections from public retrieval
   - Outcome: Only finished sections appear in search
   - Example: Stub has 50 tokens, `min_tokens: 200` hides it

4. **Long-form content**
   - Task: Ensure tutorials have enough detail
   - Outcome: No superficial tutorials in results
   - Example: `min_tokens: 600` for tutorials (forces depth)

5. **API reference validation**
   - Task: Detect incomplete API docs
   - Outcome: Report shows which endpoints under-documented
   - Example: Reference sections with < 300 tokens flagged

6. **Translation quality**
   - Task: Ensure translations are complete
   - Outcome: Reject machine translations that are too terse
   - Example: Original 800 tokens, translation 200 → rejected

7. **Content audits**
   - Task: Find thin content needing expansion
   - Outcome: Report of segments below threshold
   - Example: Query segments with < min_tokens, flag for writers

8. **Premium vs. free content**
   - Task: Premium content must be comprehensive
   - Outcome: Premium = min 800 tokens, free = no minimum
   - Example: Tiered access based on content depth

9. **Expert vs. beginner content**
   - Task: Beginner content must be detailed
   - Outcome: Beginners get comprehensive guides, not terse refs
   - Example: Beginner tutorials: `min_tokens: 600`, expert reference: `min_tokens: 200`

10. **Compliance documentation**
    - Task: Regulatory docs must be thorough
    - Outcome: Incomplete compliance docs rejected
    - Example: `min_tokens: 1000` for GDPR sections

**Typical ranges:**
```yaml
50-100:    Quick tips, one-liners acceptable
150-300:   Standard quality threshold
400-600:   Comprehensive guides required
800-1000:  In-depth tutorials, detailed specs
1000+:     Regulatory, legal, critical documentation
```

**Combined with max_tokens:**
```yaml
# Wide range (flexible)
return:
  min_tokens: 100
  max_tokens: 2000

# Narrow range (strict)
return:
  min_tokens: 600
  max_tokens: 800
```

**When NOT to use:**
- Content is naturally short (code snippets, quick commands)
- Retrieval is for display, not LLM
- Quality doesn't correlate with length

**Anti-patterns:**
- `min_tokens > max_tokens` (impossible to satisfy)
- `min_tokens` too high (excludes valid short content)
- Same `min_tokens` for all types (FAQs can be short, tutorials should be long)
<!--semcontext:segment end-->

## See Also

- [content:@doc-external-annotations/overview](#) - Complete field reference
- [content:@doc-frontmatter/overview](#) - Frontmatter schema
- [content:@doc-aggregation/overview](#) - Rollup and aggregation

<!--semcontext:segment start key="byte-line-range" type="reference" audience="developer"-->
## Byte Range / Line Range

### Concept
Location of segment within source file (byte offsets or line numbers).

### Task
**Primary:** Extract segment content from source file
**Secondary:** Validate ranges, detect content shifts

### Why You Need It
**Problem:** External annotations need to know WHERE segment is in file
**Solution:** Line range for text files, byte range for binary files

### How to Use It

**External annotation (required):**
```json
{
  "segments": [
    {
      "id": "overview",
      "line_range": {"start": 1, "end": 25}
    },
    {
      "id": "binary-data",
      "byte_range": {"start": 1250, "end": 3480}
    }
  ]
}
```

**Note:** At least ONE of byte_range or line_range MUST be present in external annotations.

**Extraction:**
```typescript
// Line range (text files)
const lines = source.split('\n');
const content = lines.slice(lineRange.start - 1, lineRange.end).join('\n');

// Byte range (binary files)
const content = source.substring(byteRange.start, byteRange.end);
```

### Outcome
- **Content extraction** - Get segment from file
- **Format independence** - Works with text and binary
- **Human maintainability** - Line numbers easier than byte offsets for text
- **Precision** - Exact byte ranges for binary formats

### Situations
1. **Text files** - Use line_range (easier to maintain when file changes)
2. **Binary files** - Use byte_range (only option)
3. **JSON/XML** - Use byte_range (no concept of "lines")
4. **Generated files** - Line ranges from parser output
5. **Large files** - Byte ranges for performance (no line splitting)

**When NOT to use:**
- Inline markers or frontmatter (location implicit)
- Entire file is one segment

**Prefer line_range for text** - More maintainable, survives minor edits better.
<!--semcontext:segment end-->

## Complete Field Matrix

| Field | Inline | Frontmatter | External | Task | Use When |
|-------|--------|-------------|----------|------|----------|
| **id/key** | ✓ | ✓ | ✓ | Identify segment | Always |
| **type** | ✓ | ✓ | ✓ | Classify content | Content has distinct type |
| **audience** | ✓ | ✓ | ✓ | Filter by role | Multi-role product |
| **concepts** | ✗ | ✓ | ✓ | Semantic search | Large doc site, knowledge graph |
| **boost** | ✗ | ✓ | ✓ | Prioritize results | Some content more important |
| **semantics.*** | ✗ | ✓ | ✓ | Link to entities | SpecBook integration |
| **return.max** | ✗ | ✓ | ✓ | Limit context | RAG, cost control |
| **return.min** | ✗ | ✓ | ✓ | Quality threshold | Filter stubs |
| **generate.*** | ✗ | ✓ | ✓ | Control generation | AI-generated content |
| **segment_checksum** | ✗ | ✗ | ✓ | Detect drift | External docs, monitoring |
| **byte_range** | ✗ | ✗ | ✓ | Extract content | Binary files, external annotations |
| **line_range** | ✗ | ✗ | ✓ | Extract content | Text files, external annotations |

**Total fields documented:** 9 core fields + aggregation + all sub-fields

