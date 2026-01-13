# Reference Implementations

Official parsers and validators for Semwerk specifications.

## Maintained by Semwerk

### semspec-ts (TypeScript/npm)

**Repository:** https://github.com/semwerk/semspec-ts
**Package:** [@semwerk/semspec](https://www.npmjs.com/package/@semwerk/semspec)
**Status:** Active development
**Version:** 0.1.0

**Features:**
- Parse linkage.yaml
- Extract segment markers from markdown
- Parse YAML frontmatter
- Validate against JSON schemas
- Bidirectional consistency checking
- CLI validator

**Installation:**
```bash
npm install @semwerk/semspec
```

**Documentation:** [README](https://github.com/semwerk/semspec-ts/blob/main/README.md)

---

### semspec-python (Python/pip)

**Repository:** https://github.com/semwerk/semspec-python
**Package:** [semspec](https://pypi.org/project/semspec/)
**Status:** Active development
**Version:** 0.1.0

**Features:**
- Parse linkage.yaml
- Extract segment markers from markdown
- Parse YAML frontmatter
- Dataclass-based models
- Directory-wide validation
- CLI validator

**Installation:**
```bash
pip install semspec
```

**Documentation:** [README](https://github.com/semwerk/semspec-python/blob/main/README.md)

---

### semspec-go (Go module)

**Repository:** https://github.com/semwerk/semspec-go
**Module:** github.com/semwerk/semspec-go
**Status:** Active development
**Version:** 0.1.0

**Features:**
- Parse linkage.yaml
- Extract segment markers from markdown
- Parse YAML frontmatter
- Struct-based models
- Bidirectional consistency checking
- Stdlib dependencies only

**Installation:**
```bash
go get github.com/semwerk/semspec-go
```

**Documentation:** [README](https://github.com/semwerk/semspec-go/blob/main/README.md)

---

## Community Implementations

None yet. Want to contribute an implementation in another language? See [CONTRIBUTING](../CONTRIBUTING.md).

### Potential Languages

- **Rust** - Great fit for high-performance tooling
- **Java** - For JVM ecosystem integration
- **C#** - For .NET developers
- **Ruby** - For Jekyll/static site generators

## Specification Support Matrix

| Specification | semspec-ts | semspec-python | semspec-go |
|--------------|-------------|-----------------|-------------|
| Linkage Mapping | ✓ | ✓ | ✓ |
| Segment Markers | ✓ | ✓ | ✓ |
| Frontmatter | ✓ | ✓ | ✓ |
| Segment Taxonomy | - | - | - |
| Symbol Taxonomy | - | - | - |
| Audience Roles | - | - | - |
| Change Types | - | - | - |
| Provenance | - | - | - |
| Classification Config | - | - | - |

**Note:** Taxonomy and model specs are informational; parsers implement the formats that use them.

## Contributing an Implementation

See our [contribution guidelines](../CONTRIBUTING.md) for:
- Implementation requirements
- Testing expectations
- Documentation standards
- Review process

We welcome implementations in new languages!
