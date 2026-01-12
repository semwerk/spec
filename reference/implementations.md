# Reference Implementations

Official parsers and validators for Semwerk specifications.

## Maintained by Semwerk

### werkspec-ts (TypeScript/npm)

**Repository:** https://github.com/semwerk/werkspec-ts
**Package:** [@semwerk/werkspec](https://www.npmjs.com/package/@semwerk/werkspec)
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
npm install @semwerk/werkspec
```

**Documentation:** [README](https://github.com/semwerk/werkspec-ts/blob/main/README.md)

---

### werkspec-python (Python/pip)

**Repository:** https://github.com/semwerk/werkspec-python
**Package:** [werkspec](https://pypi.org/project/werkspec/)
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
pip install werkspec
```

**Documentation:** [README](https://github.com/semwerk/werkspec-python/blob/main/README.md)

---

### werkspec-go (Go module)

**Repository:** https://github.com/semwerk/werkspec-go
**Module:** github.com/semwerk/werkspec-go
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
go get github.com/semwerk/werkspec-go
```

**Documentation:** [README](https://github.com/semwerk/werkspec-go/blob/main/README.md)

---

## Community Implementations

None yet. Want to contribute an implementation in another language? See [CONTRIBUTING](../CONTRIBUTING.md).

### Potential Languages

- **Rust** - Great fit for high-performance tooling
- **Java** - For JVM ecosystem integration
- **C#** - For .NET developers
- **Ruby** - For Jekyll/static site generators

## Specification Support Matrix

| Specification | werkspec-ts | werkspec-python | werkspec-go |
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
