# Aggregation and Rollup

**Version:** 1.0.0
**Status:** Draft

<!--semcontext:segment start key="overview" type="overview" audience="developer,engineering"-->
## Overview

Aggregation enables automatic rollup of segment metadata to page level and project level. This creates a hierarchical view of semantic metadata without manual duplication.

**Hierarchy:**
```
Project
  ├─ Page 1
  │   ├─ Segment 1
  │   ├─ Segment 2
  │   └─ Segment 3
  └─ Page 2
      ├─ Segment 1
      └─ Segment 2
```

**Aggregation levels:**
1. **Segment** - Original metadata (source of truth)
2. **Page** - Aggregated from all segments in page
3. **Project** - Aggregated from all pages in project

**Use cases:**
- Generate page-level concept index
- Create project-wide concept taxonomy
- Calculate total token budgets
- Build aggregated semantic entity maps
- Project-level search/filtering
<!--semcontext:segment end-->

<!--semcontext:segment start key="page-aggregation" type="reference" audience="developer"-->
## Page-Level Aggregation

### Aggregation Rules

| Field | Aggregation Method | Example |
|-------|-------------------|---------|
| `concepts` | **Union** (deduplicate) | [seg1: auth, oauth] + [seg2: oauth, jwt] = [auth, oauth, jwt] |
| `audience_role` | **Union** (deduplicate) | [seg1: developer] + [seg2: developer, operator] = [developer, operator] |
| `semantics.*` | **Union** (deduplicate) | All tasks/problems/solutions from all segments |
| `return.max_tokens` | **Sum** | seg1: 800 + seg2: 600 = 1400 total |
| `return.min_tokens` | **Sum** | seg1: 200 + seg2: 150 = 350 minimum |
| `boost` | **Weighted average** by segment token count | (seg1.boost × seg1.tokens + seg2.boost × seg2.tokens) / total_tokens |
| `segment_checksums` | **Array** of all segment checksums | [sha256:seg1, sha256:seg2, sha256:seg3] |
| `page_checksum` | **Combined** hash of all segment checksums | sha256(seg1_checksum + seg2_checksum + seg3_checksum) |

### Page Aggregation Format

```json
{
  "version": "1",
  "source_file": "docs/authentication.md",
  "source_checksum": "sha256:file123",

  "page_aggregation": {
    "concepts": ["authentication", "oauth", "jwt", "security"],
    "audience_role": ["developer", "operator"],

    "semantics": {
      "tasks": ["task:@stsk_setup_auth", "task:@stsk_configure_oauth"],
      "problems": ["problem:@sprb_auth_complexity"],
      "solutions": ["solution:@ssol_simplified_auth"],
      "features": ["feature:@sftr_oauth", "feature:@sftr_jwt"]
    },

    "token_budget": {
      "total_return_max": 2400,
      "total_return_min": 700,
      "total_generate_max": 3000,
      "total_generate_min": 900
    },

    "average_boost": 1.35,

    "segment_count": 3,
    "segment_checksums": [
      "sha256:seg1abc",
      "sha256:seg2def",
      "sha256:seg3ghi"
    ],
    "page_checksum": "sha256:combined_page_hash"
  },

  "segments": [
    {
      "id": "overview",
      "concepts": ["authentication"],
      "semantics": {
        "tasks": ["task:@stsk_understand_auth"]
      },
      "return": {"max_tokens": 800, "min_tokens": 200},
      "boost": 1.5
    },
    {
      "id": "oauth-setup",
      "concepts": ["oauth", "security"],
      "semantics": {
        "tasks": ["task:@stsk_setup_auth", "task:@stsk_configure_oauth"],
        "problems": ["problem:@sprb_auth_complexity"],
        "features": ["feature:@sftr_oauth"]
      },
      "return": {"max_tokens": 1000, "min_tokens": 300},
      "boost": 1.4
    },
    {
      "id": "jwt-tokens",
      "concepts": ["jwt", "security"],
      "semantics": {
        "solutions": ["solution:@ssol_simplified_auth"],
        "features": ["feature:@sftr_jwt"]
      },
      "return": {"max_tokens": 600, "min_tokens": 200},
      "boost": 1.2
    }
  ]
}
```

### TypeScript Implementation

```typescript
interface PageAggregation {
  concepts: string[];
  audience_role: string[];
  semantics: {
    tasks?: string[];
    problems?: string[];
    solutions?: string[];
    outcomes?: string[];
    validations?: string[];
    next_steps?: string[];
    impacts?: string[];
    features?: string[];
  };
  token_budget: {
    total_return_max: number;
    total_return_min: number;
    total_generate_max?: number;
    total_generate_min?: number;
  };
  average_boost: number;
  segment_count: number;
  segment_checksums: string[];
  page_checksum: string;
}

function aggregatePageMetadata(
  annotations: ExternalAnnotations
): PageAggregation {
  const concepts = new Set<string>();
  const audienceRoles = new Set<string>();
  const semantics: Record<string, Set<string>> = {
    tasks: new Set(),
    problems: new Set(),
    solutions: new Set(),
    outcomes: new Set(),
    validations: new Set(),
    next_steps: new Set(),
    impacts: new Set(),
    features: new Set(),
  };

  let totalReturnMax = 0;
  let totalReturnMin = 0;
  let totalGenerateMax = 0;
  let totalGenerateMin = 0;
  let weightedBoostSum = 0;
  let totalTokens = 0;
  const segmentChecksums: string[] = [];

  for (const segment of annotations.segments) {
    // Aggregate concepts
    if (segment.concepts) {
      const conceptList = Array.isArray(segment.concepts)
        ? segment.concepts
        : [segment.concepts];
      conceptList.forEach(c => concepts.add(c));
    }

    // Aggregate audience roles
    if (segment.audience_role) {
      const roleList = Array.isArray(segment.audience_role)
        ? segment.audience_role
        : [segment.audience_role];
      roleList.forEach(r => audienceRoles.add(r));
    }

    // Aggregate semantics
    if (segment.semantics) {
      Object.entries(segment.semantics).forEach(([key, values]) => {
        if (values && Array.isArray(values)) {
          values.forEach(v => semantics[key]?.add(v));
        }
      });
    }

    // Aggregate token budgets
    if (segment.return?.max_tokens) {
      totalReturnMax += segment.return.max_tokens;
      totalTokens += segment.return.max_tokens;
    }
    if (segment.return?.min_tokens) {
      totalReturnMin += segment.return.min_tokens;
    }
    if (segment.generate?.max_tokens) {
      totalGenerateMax += segment.generate.max_tokens;
    }
    if (segment.generate?.min_tokens) {
      totalGenerateMin += segment.generate.min_tokens;
    }

    // Weighted boost
    const segmentTokens = segment.return?.max_tokens || 0;
    const boost = segment.boost || 1.0;
    weightedBoostSum += boost * segmentTokens;

    // Collect checksums
    if (segment.segment_checksum) {
      segmentChecksums.push(segment.segment_checksum);
    }
  }

  // Calculate combined page checksum
  const pageChecksum = segmentChecksums.length > 0
    ? `sha256:${sha256(segmentChecksums.join('|'))}`
    : '';

  return {
    concepts: Array.from(concepts),
    audience_role: Array.from(audienceRoles),
    semantics: Object.fromEntries(
      Object.entries(semantics).map(([key, set]) => [key, Array.from(set)])
    ),
    token_budget: {
      total_return_max: totalReturnMax,
      total_return_min: totalReturnMin,
      total_generate_max: totalGenerateMax || undefined,
      total_generate_min: totalGenerateMin || undefined,
    },
    average_boost: totalTokens > 0 ? weightedBoostSum / totalTokens : 1.0,
    segment_count: annotations.segments.length,
    segment_checksums: segmentChecksums,
    page_checksum: pageChecksum,
  };
}
```

### Python Implementation

```python
from typing import Dict, List, Set
import hashlib

def aggregate_page_metadata(annotations: ExternalAnnotations) -> dict:
    """Aggregate segment metadata to page level."""
    concepts: Set[str] = set()
    audience_roles: Set[str] = set()
    semantics = {
        'tasks': set(),
        'problems': set(),
        'solutions': set(),
        'outcomes': set(),
        'validations': set(),
        'next_steps': set(),
        'impacts': set(),
        'features': set(),
    }

    total_return_max = 0
    total_return_min = 0
    total_generate_max = 0
    total_generate_min = 0
    weighted_boost_sum = 0.0
    total_tokens = 0
    segment_checksums = []

    for segment in annotations.segments:
        # Aggregate concepts
        if segment.concepts:
            concept_list = segment.concepts if isinstance(segment.concepts, list) else [segment.concepts]
            concepts.update(concept_list)

        # Aggregate audience roles
        if segment.audience_role:
            role_list = segment.audience_role if isinstance(segment.audience_role, list) else [segment.audience_role]
            audience_roles.update(role_list)

        # Aggregate semantics
        if segment.semantics:
            for key in semantics.keys():
                values = getattr(segment.semantics, key, [])
                if values:
                    semantics[key].update(values)

        # Aggregate token budgets
        if segment.return_config:
            total_return_max += segment.return_config.get('max_tokens', 0)
            total_return_min += segment.return_config.get('min_tokens', 0)
            total_tokens += segment.return_config.get('max_tokens', 0)

        if segment.generate_config:
            total_generate_max += segment.generate_config.get('max_tokens', 0)
            total_generate_min += segment.generate_config.get('min_tokens', 0)

        # Weighted boost
        segment_tokens = segment.return_config.get('max_tokens', 0) if segment.return_config else 0
        boost = segment.boost or 1.0
        weighted_boost_sum += boost * segment_tokens

        # Collect checksums
        if segment.segment_checksum:
            segment_checksums.append(segment.segment_checksum)

    # Calculate combined checksum
    page_checksum = ''
    if segment_checksums:
        combined = '|'.join(segment_checksums)
        page_checksum = f"sha256:{hashlib.sha256(combined.encode()).hexdigest()}"

    return {
        'concepts': list(concepts),
        'audience_role': list(audience_roles),
        'semantics': {k: list(v) for k, v in semantics.items() if v},
        'token_budget': {
            'total_return_max': total_return_max,
            'total_return_min': total_return_min,
            'total_generate_max': total_generate_max if total_generate_max > 0 else None,
            'total_generate_min': total_generate_min if total_generate_min > 0 else None,
        },
        'average_boost': weighted_boost_sum / total_tokens if total_tokens > 0 else 1.0,
        'segment_count': len(annotations.segments),
        'segment_checksums': segment_checksums,
        'page_checksum': page_checksum,
    }
```

### Go Implementation

```go
type PageAggregation struct {
	Concepts          []string          `json:"concepts"`
	AudienceRole      []string          `json:"audience_role"`
	Semantics         map[string][]string `json:"semantics"`
	TokenBudget       TokenBudgetSummary `json:"token_budget"`
	AverageBoost      float64           `json:"average_boost"`
	SegmentCount      int               `json:"segment_count"`
	SegmentChecksums  []string          `json:"segment_checksums"`
	PageChecksum      string            `json:"page_checksum"`
}

type TokenBudgetSummary struct {
	TotalReturnMax    int `json:"total_return_max"`
	TotalReturnMin    int `json:"total_return_min"`
	TotalGenerateMax  int `json:"total_generate_max,omitempty"`
	TotalGenerateMin  int `json:"total_generate_min,omitempty"`
}

func AggregatePageMetadata(anno *ExternalAnnotations) *PageAggregation {
	concepts := make(map[string]bool)
	audienceRoles := make(map[string]bool)
	semantics := make(map[string]map[string]bool)

	var totalReturnMax, totalReturnMin, totalGenerateMax, totalGenerateMin int
	var weightedBoostSum, totalTokens float64
	var segmentChecksums []string

	for _, segment := range anno.Segments {
		// Aggregate concepts, audience roles, semantics...
		// (Similar logic to TypeScript/Python)

		// Aggregate token budgets
		if segment.Return != nil {
			totalReturnMax += segment.Return.MaxTokens
			totalReturnMin += segment.Return.MinTokens
			totalTokens += float64(segment.Return.MaxTokens)
		}

		// Weighted boost
		segmentTokens := 0
		if segment.Return != nil {
			segmentTokens = segment.Return.MaxTokens
		}
		boost := segment.Boost
		if boost == 0 {
			boost = 1.0
		}
		weightedBoostSum += boost * float64(segmentTokens)

		if segment.SegmentChecksum != "" {
			segmentChecksums = append(segmentChecksums, segment.SegmentChecksum)
		}
	}

	// Calculate combined checksum
	pageChecksum := ""
	if len(segmentChecksums) > 0 {
		combined := strings.Join(segmentChecksums, "|")
		hash := sha256.Sum256([]byte(combined))
		pageChecksum = fmt.Sprintf("sha256:%x", hash)
	}

	averageBoost := 1.0
	if totalTokens > 0 {
		averageBoost = weightedBoostSum / totalTokens
	}

	return &PageAggregation{
		Concepts:         mapKeysToSlice(concepts),
		AudienceRole:     mapKeysToSlice(audienceRoles),
		Semantics:        mapToStringSlices(semantics),
		TokenBudget: TokenBudgetSummary{
			TotalReturnMax:   totalReturnMax,
			TotalReturnMin:   totalReturnMin,
			TotalGenerateMax: totalGenerateMax,
			TotalGenerateMin: totalGenerateMin,
		},
		AverageBoost:     averageBoost,
		SegmentCount:     len(anno.Segments),
		SegmentChecksums: segmentChecksums,
		PageChecksum:     pageChecksum,
	}
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="project-aggregation" type="reference" audience="developer"-->
## Project-Level Aggregation

### Project Aggregation Format

```json
{
  "version": "1",
  "project": "project:@prj_payment_platform",
  "project_version": "version:@v2.0.0",

  "project_aggregation": {
    "concepts": ["authentication", "payments", "webhooks", "api", "security"],
    "audience_role": ["developer", "operator", "user"],

    "semantics": {
      "tasks": ["task:@stsk_integrate_payments", "task:@stsk_setup_auth"],
      "problems": ["problem:@sprb_integration_complexity"],
      "solutions": ["solution:@ssol_sdk", "solution:@ssol_api_wrapper"],
      "outcomes": ["outcome:@sout_payment_integration"],
      "features": ["feature:@sftr_payments", "feature:@sftr_oauth"],
      "personas": ["persona:@per_backend_dev", "persona:@per_frontend_dev"]
    },

    "token_budget": {
      "total_return_max": 12500,
      "total_return_min": 3800,
      "total_generate_max": 15000,
      "total_generate_min": 5000
    },

    "average_boost": 1.25,
    "page_count": 8,
    "segment_count": 45,

    "page_checksums": [
      "sha256:page1_combined",
      "sha256:page2_combined",
      "sha256:page3_combined"
    ],
    "project_checksum": "sha256:project_combined_hash"
  },

  "pages": [
    {
      "source_file": "docs/authentication.md",
      "page_aggregation": {
        "concepts": ["authentication", "oauth", "jwt"],
        "segment_count": 3,
        "page_checksum": "sha256:auth_page"
      }
    },
    {
      "source_file": "docs/payments.md",
      "page_aggregation": {
        "concepts": ["payments", "webhooks"],
        "segment_count": 5,
        "page_checksum": "sha256:payment_page"
      }
    }
  ]
}
```

### TypeScript Implementation

```typescript
interface ProjectAggregation {
  project: string;
  project_version?: string;
  project_aggregation: {
    concepts: string[];
    audience_role: string[];
    semantics: Record<string, string[]>;
    token_budget: {
      total_return_max: number;
      total_return_min: number;
      total_generate_max?: number;
      total_generate_min?: number;
    };
    average_boost: number;
    page_count: number;
    segment_count: number;
    page_checksums: string[];
    project_checksum: string;
  };
  pages: PageSummary[];
}

interface PageSummary {
  source_file: string;
  page_aggregation: PageAggregation;
}

function aggregateProjectMetadata(
  pages: ExternalAnnotations[]
): ProjectAggregation {
  const concepts = new Set<string>();
  const audienceRoles = new Set<string>();
  const semantics: Record<string, Set<string>> = {};

  let totalReturnMax = 0;
  let totalReturnMin = 0;
  let totalGenerateMax = 0;
  let totalGenerateMin = 0;
  let weightedBoostSum = 0;
  let totalTokens = 0;
  let totalSegments = 0;
  const pageChecksums: string[] = [];

  const pageSummaries: PageSummary[] = [];

  for (const page of pages) {
    const pageAgg = aggregatePageMetadata(page);

    // Aggregate page-level data to project
    pageAgg.concepts.forEach(c => concepts.add(c));
    pageAgg.audience_role.forEach(r => audienceRoles.add(r));

    Object.entries(pageAgg.semantics).forEach(([key, values]) => {
      if (!semantics[key]) semantics[key] = new Set();
      values.forEach(v => semantics[key].add(v));
    });

    totalReturnMax += pageAgg.token_budget.total_return_max;
    totalReturnMin += pageAgg.token_budget.total_return_min;
    totalGenerateMax += pageAgg.token_budget.total_generate_max || 0;
    totalGenerateMin += pageAgg.token_budget.total_generate_min || 0;

    weightedBoostSum += pageAgg.average_boost * pageAgg.token_budget.total_return_max;
    totalTokens += pageAgg.token_budget.total_return_max;
    totalSegments += pageAgg.segment_count;

    pageChecksums.push(pageAgg.page_checksum);

    pageSummaries.push({
      source_file: page.source_file,
      page_aggregation: pageAgg,
    });
  }

  // Calculate project checksum
  const projectChecksum = `sha256:${sha256(pageChecksums.join('|'))}`;

  return {
    project: 'project:@prj_example',
    project_aggregation: {
      concepts: Array.from(concepts),
      audience_role: Array.from(audienceRoles),
      semantics: Object.fromEntries(
        Object.entries(semantics).map(([key, set]) => [key, Array.from(set)])
      ),
      token_budget: {
        total_return_max: totalReturnMax,
        total_return_min: totalReturnMin,
        total_generate_max: totalGenerateMax || undefined,
        total_generate_min: totalGenerateMin || undefined,
      },
      average_boost: totalTokens > 0 ? weightedBoostSum / totalTokens : 1.0,
      page_count: pages.length,
      segment_count: totalSegments,
      page_checksums: pageChecksums,
      project_checksum: projectChecksum,
    },
    pages: pageSummaries,
  };
}
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="use-cases" type="howto" audience="developer,product"-->
## Use Cases

### 1. Project-Wide Concept Index

```typescript
// Build project-wide concept taxonomy
const projectAgg = aggregateProjectMetadata(allPages);
const allConcepts = projectAgg.project_aggregation.concepts;

// Generate concept index
const conceptIndex = allConcepts.map(concept => ({
  concept,
  pages: findPagesWithConcept(allPages, concept),
  segments: findSegmentsWithConcept(allPages, concept),
}));
```

### 2. Token Budget Planning

```python
# Calculate total token budget for project
project_agg = aggregate_project_metadata(all_pages)
total_budget = project_agg['project_aggregation']['token_budget']

print(f"Total retrieval budget: {total_budget['total_return_max']} tokens")
print(f"Total generation budget: {total_budget['total_generate_max']} tokens")
print(f"Pages: {project_agg['project_aggregation']['page_count']}")
print(f"Segments: {project_agg['project_aggregation']['segment_count']}")
```

### 3. Semantic Entity Mapping

```typescript
// Find all tasks addressed by project
const projectAgg = aggregateProjectMetadata(allPages);
const allTasks = projectAgg.project_aggregation.semantics.tasks;

// Map tasks to pages
const taskMap = allTasks.map(task => ({
  task,
  pages: findPagesWithTask(allPages, task),
  segments: findSegmentsWithTask(allPages, task),
}));
```

### 4. Incremental Updates

```python
# Only re-aggregate pages that changed
def update_project_aggregation(
    old_project_agg: dict,
    changed_pages: List[ExternalAnnotations]
):
    # Remove old checksums
    old_checksums = set(old_project_agg['page_checksums'])

    # Add new checksums
    for page in changed_pages:
        page_agg = aggregate_page_metadata(page)
        # Update aggregation...

    return updated_project_agg
```

### 5. Search Index Generation

```typescript
// Generate search index with aggregated metadata
const projectAgg = aggregateProjectMetadata(allPages);

const searchIndex = {
  project: projectAgg.project,
  concepts: projectAgg.project_aggregation.concepts,
  audiences: projectAgg.project_aggregation.audience_role,
  pages: projectAgg.pages.map(p => ({
    file: p.source_file,
    concepts: p.page_aggregation.concepts,
    checksum: p.page_aggregation.page_checksum,
  })),
};
```
<!--semcontext:segment end-->

<!--semcontext:segment start key="caching" type="howto" audience="developer"-->
## Caching and Incremental Updates

### Page-Level Caching

Store page aggregations to avoid recomputation:

```typescript
interface PageCache {
  source_file: string;
  source_checksum: string;
  page_aggregation: PageAggregation;
  computed_at: string;
}

// Check if page aggregation is cached and valid
function getCachedPageAggregation(
  annoFile: string,
  cache: Map<string, PageCache>
): PageAggregation | null {
  const anno = parseExternalAnnotations(annoFile);
  const cached = cache.get(anno.source_file);

  if (!cached) return null;

  // Validate checksum hasn't changed
  if (cached.source_checksum !== anno.source_checksum) {
    return null; // Cache invalid
  }

  return cached.page_aggregation;
}

// Compute and cache
function computeAndCachePageAggregation(
  anno: ExternalAnnotations,
  cache: Map<string, PageCache>
): PageAggregation {
  const pageAgg = aggregatePageMetadata(anno);

  cache.set(anno.source_file, {
    source_file: anno.source_file,
    source_checksum: anno.source_checksum!,
    page_aggregation: pageAgg,
    computed_at: new Date().toISOString(),
  });

  return pageAgg;
}
```

### Project-Level Caching

```typescript
interface ProjectCache {
  project: string;
  project_version?: string;
  page_checksums: string[];
  project_aggregation: ProjectAggregation['project_aggregation'];
  computed_at: string;
}

// Incremental project aggregation
function incrementalProjectAggregation(
  oldCache: ProjectCache,
  changedPages: ExternalAnnotations[],
  unchangedPages: ExternalAnnotations[]
): ProjectAggregation {
  // Reuse aggregations for unchanged pages
  const unchangedAggs = unchangedPages.map(p =>
    getCachedPageAggregation(p.source_file, pageCache)
  );

  // Compute new aggregations for changed pages
  const changedAggs = changedPages.map(aggregatePageMetadata);

  // Combine all
  return aggregateProjectMetadata([...unchangedAggs, ...changedAggs]);
}
```
<!--semcontext:segment end-->

## See Also

- [content:@doc-external-annotations/overview](#) - External annotation format
- [content:@doc-frontmatter/overview](#) - Frontmatter format
- [content:@doc-project-definition/overview](#) - Project structure
