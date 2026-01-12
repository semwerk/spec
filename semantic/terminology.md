# Terminology Definition

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Terminology entries define domain-specific terms and their meanings. Used for glossaries, tooltips, and semantic understanding.

## Format

```yaml
version: "1"
kind: terminology

terms:
  - id: <string>                  # Unique ID (term_*)
    key: <string>                 # Kebab-case key
    term: <string>                # The term itself
    definition: <string>          # Full definition
    aliases: [<string>]           # Alternative names
    abbreviation: <string>        # Common abbreviation

    # Context
    domain: <string>              # Domain (technical | business | legal)
    category: <string>            # Category within domain

    # Relationships
    related_terms: [<term-ref>]   # Related terminology
    concepts: [<concept-ref>]     # Associated concepts

    # References
    content_refs: [<content-coord>]  # Documentation references
    code_refs: [<code-coord>]     # Code usage examples

metadata:
  source: <enum>                  # manual | discovered | imported
  status: <enum>                  # active | deprecated
```

## Examples

### Technical Terms

```yaml
version: "1"
kind: terminology

terms:
  - id: term_jwt
    key: jwt
    term: "JWT"
    definition: "JSON Web Token - A compact, URL-safe means of representing claims to be transferred between two parties"
    abbreviation: JWT
    aliases:
      - "JSON Web Token"
      - "Bearer Token"

    domain: technical
    category: authentication

    related_terms:
      - term:@term_oauth
      - term:@term_bearer_auth

    concepts:
      - concept:@cpt_authentication
      - concept:@cpt_jwt

    content_refs:
      - content:@doc-auth@v2.0.0/jwt.overview
      - content:@doc-api@v2.0.0/authentication.jwt

    code_refs:
      - code:@cfn_jwt_handler@v2.0.0/GenerateToken
      - code:@cfn_jwt_handler@v2.0.0/ValidateToken

    metadata:
      source: manual
      status: active

  - id: term_oauth
    key: oauth
    term: "OAuth 2.0"
    definition: "An authorization framework that enables applications to obtain limited access to user accounts"
    abbreviation: OAuth
    aliases:
      - "OAuth"
      - "Open Authorization"

    domain: technical
    category: authentication

    related_terms:
      - term:@term_jwt
      - term:@term_pkce

    concepts:
      - concept:@cpt_oauth
      - concept:@cpt_authentication

    content_refs:
      - content:@doc-auth@v2.0.0/oauth

    metadata:
      source: manual
      status: active
```

### Business Terms

```yaml
version: "1"
kind: terminology

terms:
  - id: term_arr
    key: arr
    term: "ARR"
    definition: "Annual Recurring Revenue - The value of recurring subscription revenue normalized to a single calendar year"
    abbreviation: ARR
    aliases:
      - "Annual Recurring Revenue"
      - "Annualized Revenue"

    domain: business
    category: metrics

    related_terms:
      - term:@term_mrr
      - term:@term_revenue

    concepts:
      - concept:@cpt_revenue
      - concept:@cpt_saas_metrics

    metadata:
      source: manual
      status: active

  - id: term_churn
    key: churn
    term: "Churn Rate"
    definition: "The percentage of customers who stop using your product during a given time period"
    aliases:
      - "Customer Churn"
      - "Attrition Rate"

    domain: business
    category: metrics

    related_terms:
      - term:@term_retention
      - term:@term_arr

    concepts:
      - concept:@cpt_retention
      - concept:@cpt_saas_metrics

    metadata:
      source: manual
      status: active
```

### Legal Terms

```yaml
version: "1"
kind: terminology

terms:
  - id: term_pci_dss
    key: pci-dss
    term: "PCI DSS"
    definition: "Payment Card Industry Data Security Standard - Security standards for organizations that handle credit cards"
    abbreviation: PCI DSS
    aliases:
      - "PCI Data Security Standard"
      - "Payment Card Industry Standard"

    domain: legal
    category: compliance

    related_terms:
      - term:@term_gdpr
      - term:@term_compliance

    concepts:
      - concept:@cpt_pci_compliance
      - concept:@cpt_security

    content_refs:
      - content:@doc-compliance@v2.0.0/pci-dss

    metadata:
      source: manual
      status: active
```

## Terminology as First-Party Concept

Terminology is elevated to a first-party concept with bidirectional linking to content assets and segments.

### Domain Models

#### Terminology

```yaml
terminology:
  id: <uuid>                    # Internal ID
  public_id: <string>           # term_* prefixed ID
  tenant_id: <uuid>             # Tenant scope
  term: <string>                # The canonical term
  definition: <string>          # Full definition
  description: <string>         # Extended description (optional)
  aliases: [<string>]           # Alternative names/spellings
  tags: [<string>]              # Categorization tags
  usage_context: <string>       # When/how to use this term
  status: <enum>                # active | deprecated | draft | review
  date_created: <timestamp>
  date_updated: <timestamp>
```

#### TerminologyAssetLink

```yaml
terminology_asset_link:
  id: <uuid>
  terminology_id: <uuid>
  asset_id: <uuid>
  link_type: <enum>             # manual | auto_detected | suggested
  confidence: <float>           # 0.0-1.0, null for manual
  date_created: <timestamp>
```

#### TerminologySegmentLink

```yaml
terminology_segment_link:
  id: <uuid>
  terminology_id: <uuid>
  segment_id: <uuid>
  link_type: <enum>             # manual | auto_detected | suggested
  confidence: <float>           # 0.0-1.0, null for manual
  date_created: <timestamp>
```

#### TerminologyMention

```yaml
terminology_mention:
  id: <uuid>
  terminology_id: <uuid>
  asset_id: <uuid>
  segment_id: <uuid>            # Optional, null for asset-level
  matched_text: <string>        # Exact text that matched
  byte_offset_start: <int>      # Position in content
  byte_offset_end: <int>
  content_hash: <string>        # Hash for drift detection
  date_created: <timestamp>
```

### Link Types

| Type | Description | Confidence |
|------|-------------|------------|
| `manual` | Explicitly created by user | null |
| `auto_detected` | Discovered by scanning content | 0.0-1.0 |
| `suggested` | AI-suggested, pending review | 0.0-1.0 |

### Content Linking

Terminology can be explicitly linked to content:

| Link Type | Direction | Description |
|-----------|-----------|-------------|
| `terminology_asset` | term -> asset | Term defines a concept used in this asset |
| `terminology_segment` | term -> segment | Term is relevant to this segment |
| `terminology_mention` | auto-detected | Term appears in content (auto-detected) |

### API Interface

#### gRPC Service (semantic.v1.TerminologyService)

```protobuf
service TerminologyService {
  // CRUD
  rpc CreateTerminology(CreateTerminologyRequest) returns (Terminology);
  rpc GetTerminology(GetTerminologyRequest) returns (Terminology);
  rpc UpdateTerminology(UpdateTerminologyRequest) returns (Terminology);
  rpc DeleteTerminology(DeleteTerminologyRequest) returns (DeleteTerminologyResponse);
  rpc ListTerminologies(ListTerminologiesRequest) returns (ListTerminologiesResponse);
  rpc SearchTerminologies(SearchTerminologiesRequest) returns (SearchTerminologiesResponse);

  // Aliases
  rpc AddAlias(AddAliasRequest) returns (Terminology);
  rpc RemoveAlias(RemoveAliasRequest) returns (Terminology);

  // Asset Linking
  rpc LinkTerminologyToAsset(LinkTerminologyToAssetRequest) returns (TerminologyAssetLink);
  rpc UnlinkTerminologyFromAsset(UnlinkTerminologyFromAssetRequest) returns (UnlinkResponse);
  rpc GetTerminologyForAsset(GetTerminologyForAssetRequest) returns (TerminologyAssetLinksResponse);
  rpc GetAssetsForTerminology(GetAssetsForTerminologyRequest) returns (TerminologyAssetLinksResponse);

  // Segment Linking
  rpc LinkTerminologyToSegment(LinkTerminologyToSegmentRequest) returns (TerminologySegmentLink);
  rpc UnlinkTerminologyFromSegment(UnlinkTerminologyFromSegmentRequest) returns (UnlinkResponse);
  rpc GetTerminologyForSegment(GetTerminologyForSegmentRequest) returns (TerminologySegmentLinksResponse);
  rpc GetSegmentsForTerminology(GetSegmentsForTerminologyRequest) returns (TerminologySegmentLinksResponse);

  // Mentions
  rpc GetMentionsForAsset(GetMentionsForAssetRequest) returns (TerminologyMentionsResponse);
  rpc GetMentionsForTerminology(GetMentionsForTerminologyRequest) returns (TerminologyMentionsResponse);

  // Auto-Detection
  rpc DetectTerminologyMentions(DetectTerminologyMentionsRequest) returns (DetectTerminologyMentionsResponse);
}
```

#### GraphQL Schema

```graphql
# Types
type Terminology {
  id: ID!
  term: String!
  definition: String!
  description: String
  aliases: [String!]!
  tags: [String!]!
  usageContext: String
  status: TerminologyStatus!
  dateCreated: Time!
  dateUpdated: Time!
  linkedAssets: [TerminologyAssetLink!]!
  linkedSegments: [TerminologySegmentLink!]!
  mentions: [TerminologyMention!]!
}

type TerminologyAssetLink {
  id: ID!
  terminology: Terminology!
  asset: Artifact!
  linkType: TerminologyLinkType!
  confidence: Float
  dateCreated: Time!
}

type TerminologySegmentLink {
  id: ID!
  terminology: Terminology!
  segment: Segment!
  linkType: TerminologyLinkType!
  confidence: Float
  dateCreated: Time!
}

type TerminologyMention {
  id: ID!
  terminology: Terminology!
  asset: Artifact!
  segment: Segment
  matchedText: String!
  byteOffsetStart: Int!
  byteOffsetEnd: Int!
  contentHash: String!
  dateCreated: Time!
}

enum TerminologyStatus {
  ACTIVE
  DEPRECATED
  DRAFT
  REVIEW
}

enum TerminologyLinkType {
  MANUAL
  AUTO_DETECTED
  SUGGESTED
}

# Queries
type Query {
  terminology(id: ID!): Terminology
  terminologyByTerm(term: String!): Terminology
  terminologies(
    tags: [String!]
    status: TerminologyStatus
    first: Int
    after: String
  ): TerminologyConnection!
  searchTerminologies(query: String!, limit: Int): [TerminologySearchResult!]!
  terminologyForAsset(assetId: ID!): [TerminologyAssetLink!]!
  terminologyForSegment(segmentId: ID!): [TerminologySegmentLink!]!
  terminologyMentionsForAsset(assetId: ID!): [TerminologyMention!]!
}

# Mutations
type Mutation {
  createTerminology(input: CreateTerminologyInput!): Terminology!
  updateTerminology(id: ID!, input: UpdateTerminologyInput!): Terminology!
  deleteTerminology(id: ID!): Boolean!
  addTerminologyAlias(terminologyId: ID!, alias: String!): Terminology!
  removeTerminologyAlias(terminologyId: ID!, alias: String!): Terminology!
  linkTerminologyToAsset(input: LinkTerminologyToAssetInput!): TerminologyAssetLink!
  unlinkTerminologyFromAsset(terminologyId: ID!, assetId: ID!): Boolean!
  linkTerminologyToSegment(input: LinkTerminologyToSegmentInput!): TerminologySegmentLink!
  unlinkTerminologyFromSegment(terminologyId: ID!, segmentId: ID!): Boolean!
  detectTerminologyMentions(input: DetectTerminologyMentionsInput!): DetectTerminologyMentionsResult!
}

# Inputs
input CreateTerminologyInput {
  term: String!
  definition: String!
  description: String
  aliases: [String!]
  tags: [String!]
  usageContext: String
  relatedConceptIds: [ID!]
  sourceSegmentIds: [ID!]
}

input UpdateTerminologyInput {
  term: String
  definition: String
  description: String
  tags: [String!]
  status: TerminologyStatus
  usageContext: String
}

input LinkTerminologyToAssetInput {
  terminologyId: ID!
  assetId: ID!
  linkType: TerminologyLinkType!
  confidence: Float
}

input LinkTerminologyToSegmentInput {
  terminologyId: ID!
  segmentId: ID!
  linkType: TerminologyLinkType!
  confidence: Float
}

input DetectTerminologyMentionsInput {
  assetId: ID!
  content: String!
  contentHash: String!
  segmentId: ID
  storeMentions: Boolean
}

type DetectTerminologyMentionsResult {
  mentions: [TerminologyMention!]!
  termsScanned: Int!
  matchCount: Int!
}
```

### Bidirectional Queries

```graphql
# Get terminology for an asset
query {
  terminologyForAsset(assetId: "ast_xyz") {
    terminology {
      id
      term
      definition
    }
    linkType
    confidence
  }
}

# Get assets for a term
query {
  terminology(id: "term_jwt") {
    linkedAssets {
      asset { id name }
      linkType
    }
    mentions {
      asset { id }
      matchedText
      byteOffsetStart
    }
  }
}
```

### Auto-Detection

The system scans content for terminology mentions using trie-based matching:

1. **Matcher builds trie** from all active terminology (terms + aliases)
2. **Case-insensitive matching** with word boundary detection
3. **Returns positions** for highlighting and linking

#### Detection Request

```yaml
detect_request:
  asset_id: ast_auth_guide
  content: "Our API uses JWT for authentication..."
  content_hash: "sha256:abc123"
  segment_id: seg_overview       # Optional
  store_mentions: true           # Persist to DB
```

#### Detection Response

```yaml
detect_response:
  mentions:
    - terminology_id: term_jwt
      asset_id: ast_auth_guide
      segment_id: seg_overview
      matched_text: "JWT"
      byte_offset_start: 13
      byte_offset_end: 16
      content_hash: "sha256:abc123"
  terms_scanned: 150
  match_count: 1
```

### Database Schema

```sql
-- Terminology to Asset linking
CREATE TABLE semantic.terminology_asset (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    terminology_id uuid NOT NULL REFERENCES semantic.terminology(id) ON DELETE CASCADE,
    asset_id uuid NOT NULL REFERENCES assets.asset(id) ON DELETE CASCADE,
    link_type text NOT NULL DEFAULT 'manual',
    confidence double precision,
    date_created timestamptz NOT NULL DEFAULT now(),
    UNIQUE(terminology_id, asset_id)
);

-- Terminology to Segment linking
CREATE TABLE semantic.terminology_segment (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    terminology_id uuid NOT NULL REFERENCES semantic.terminology(id) ON DELETE CASCADE,
    segment_id uuid NOT NULL REFERENCES assets.segment(id) ON DELETE CASCADE,
    link_type text NOT NULL DEFAULT 'manual',
    confidence double precision,
    date_created timestamptz NOT NULL DEFAULT now(),
    UNIQUE(terminology_id, segment_id)
);

-- Auto-detected terminology mentions
CREATE TABLE semantic.terminology_mention (
    id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
    terminology_id uuid NOT NULL REFERENCES semantic.terminology(id) ON DELETE CASCADE,
    asset_id uuid NOT NULL REFERENCES assets.asset(id) ON DELETE CASCADE,
    segment_id uuid REFERENCES assets.segment(id) ON DELETE CASCADE,
    matched_text text NOT NULL,
    byte_offset_start int NOT NULL,
    byte_offset_end int NOT NULL,
    content_hash text NOT NULL,
    date_created timestamptz NOT NULL DEFAULT now()
);

-- Indexes
CREATE INDEX idx_terminology_asset_term ON semantic.terminology_asset(terminology_id);
CREATE INDEX idx_terminology_asset_asset ON semantic.terminology_asset(asset_id);
CREATE INDEX idx_terminology_segment_term ON semantic.terminology_segment(terminology_id);
CREATE INDEX idx_terminology_segment_segment ON semantic.terminology_segment(segment_id);
CREATE INDEX idx_terminology_mention_asset ON semantic.terminology_mention(asset_id);
CREATE INDEX idx_terminology_mention_term ON semantic.terminology_mention(terminology_id);
```

## Terminology in Content

Terms can be used inline with tooltips:

```markdown
Our API uses {{term:jwt|JWT}} for authentication.

All payment data is {{term:pci-dss|PCI DSS}} compliant.
```

Maps to:
```yaml
term:@term_jwt
term:@term_pci_dss
```

## Terminology in Glossaries

```yaml
# Auto-generate glossary
glossary:
  - term: term:@term_jwt
  - term: term:@term_oauth
  - term: term:@term_pci_dss
```

## Discovered Terminology

```yaml
terms:
  - id: term_circuit_breaker
    key: circuit-breaker
    term: "Circuit Breaker"
    definition: "Design pattern that prevents cascading failures"
    domain: technical
    category: patterns

    metadata:
      source: discovered
      confidence: 0.85                # Discovered from code/docs
```

## Coordinates

```yaml
term:@term_jwt                                  # Terminology reference
term:@term_oauth@v1                             # Versioned term
```

## Validation

- ID MUST start with `term_`
- Slug MUST be kebab-case
- Domain MUST be: technical | business | legal | product
- If `source: discovered`, confidence SHOULD be present

## See Also

- [Concept Definition](./concept-definition.md) - Semantic concepts
- [SpecBook Entities](./specbook-entities.md) - Related entities
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
