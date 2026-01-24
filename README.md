# Semwerk Specification

Open specifications for code-documentation linkage and intelligent documentation systems.

## Vision

Enable third-party developers to build tools that work with Semwerk data formats, driving ecosystem adoption through open standards.

## What's Here

### 🔗 Data Interchange Formats
- **[Linkage Mapping](formats/linkage-mapping.md)** - Bidirectional code↔doc mapping
- **[Segment Markers](formats/segment-markers.md)** - HTML comment syntax for marking doc sections
- **[Segment Versioning](formats/segment-versioning.md)** - Version metadata for segments (inline-first, semver/latest)
- **[Frontmatter](formats/frontmatter.md)** - YAML metadata schema for documents
- **[Specbook Transform](formats/specbook-transform.md)** - Portable spec folder layout + run artifacts for turning raw business inputs into specbook pages

### 📚 Standard Vocabularies
- **[Segment Taxonomy](taxonomies/segments.md)** - Document types (overview, howto, reference, api, etc.)
- **[Symbol Taxonomy](taxonomies/symbols.md)** - Language-agnostic code symbols
- **[Audience Roles](taxonomies/audience-roles.md)** - Content targeting roles
- **[Change Types](taxonomies/change-types.md)** - Code and surface change classification

### 🏗️ Data Models
- **[Provenance Model](models/provenance.md)** - Origin tracking (inferred vs. asserted)
- **[Classification Config](models/classification.md)** - Customizable classification rules

### 📦 Projects & Versions
- **[Project Definition](projects/project-definition.md)** - Project format (products, services, libraries, platforms)
- **[Project Types](projects/project-types.md)** - Project type taxonomy
- **[Version Specification](versions/version-specification.md)** - Semver + freeform versioning
- **[Lifecycle States](versions/lifecycle-states.md)** - draft → alpha → beta → rc → supported → deprecated → eol
- **[Version Lineage](versions/version-lineage.md)** - Version ancestry graph (ancestor, fork, merge, branch, port)

### 🗺️ User Journeys
- **[Journey Graph](journeys/journey-graph.md)** - DAG format for user journeys
- **[Node Types](journeys/node-types.md)** - stage, milestone, decision, jump_off
- **[Entry/Exit Points](journeys/entry-exit-points.md)** - Journey boundaries
- **[Success Metrics](journeys/success-metrics.md)** - Measurement and KPIs

### 🧠 Semantic Concepts
- **[Concept Definition](concepts/concept-definition.md)** - Semantic concept format
- **[Relationship Types](concepts/relationship-types.md)** - parent, related, implements, documents, depends_on
- **[Concept Graph](concepts/concept-graph.md)** - Knowledge graph representation

### 👥 Semantic Entities (SpecBook)
- **[Persona](semantic/persona.md)** - User persona archetypes
- **[SpecBook Entities](semantic/specbook-entities.md)** - task, problem, solution, outcome, validation, next_step, impact, feature
- **[Terminology](semantic/terminology.md)** - Domain-specific glossary terms

### 🎯 Unified Coordinates
- **[Coordinate System](coordinates/COORDINATE_SPEC.md)** - `type:@id[@version][/segment[.subheading]][#checksum]` format

### 📖 Usage Guides
- **[Field Usage Guide](guides/field-usage-guide.md)** - Complete guide: why, how, task, outcome, and all situations for each field

## Reference Implementations

- **[@semwerk/semspec](https://github.com/semwerk/semspec-ts)** - TypeScript/npm
- **[semspec](https://github.com/semwerk/semspec-python)** - Python/pip
- **[semspec-go](https://github.com/semwerk/semspec-go)** - Go module

## Quick Start

### Parse Linkage Mapping (TypeScript)
```typescript
import { parseLinkage } from '@semwerk/semspec';

const linkage = parseLinkage(yamlContent);
console.log(linkage.code_to_assets);
```

### Parse Segment Markers (Python)
```python
from semspec import parse_segments

segments = parse_segments(markdown_content)
for segment in segments:
    print(f"{segment.id}: {segment.type}")
```

### Parse Frontmatter (Go)
```go
import "github.com/semwerk/semspec-go/frontmatter"

fm, err := frontmatter.Parse(content)
fmt.Println(fm.Semcontext.Segments)
```

## Ecosystem

These specifications enable:

- ✅ **VS Code Extensions** - Show code↔doc mappings in editor
- ✅ **Static Site Generators** - Extract segment markers (Hugo, Jekyll, Docusaurus)
- ✅ **AI Doc Tools** - Generate Semwerk-compatible markdown
- ✅ **GitHub Actions** - Validate linkage integrity in CI
- ✅ **Analytics** - Third-party doc coverage analysis

## Versioning

Specifications use [Semantic Versioning](https://semver.org/):
- **MAJOR** - Backwards-incompatible changes
- **MINOR** - New features, backwards-compatible
- **PATCH** - Bug fixes, clarifications

Current version: **0.1.0** (pre-release)

See [compatibility matrix](reference/compatibility.md) for version support.

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for:
- How to propose spec changes
- Specification writing guidelines
- Review process

## License

MIT License - See [LICENSE](LICENSE) for details.

## Learn More

- [Use Cases](reference/use-cases.md) - Example integrations
- [Known Implementations](reference/implementations.md) - Tools using these specs
- [Migration Guides](reference/migration-guides/) - Upgrade guides

## Status

🚧 **Pre-release** - Specifications are under active development. Expect changes before 1.0.0.
