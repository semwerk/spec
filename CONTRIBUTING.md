# Contributing to Semwerk Specification

Thank you for your interest in contributing to Semwerk specification!

## How to Contribute

### Reporting Issues
- Use GitHub Issues to report problems or suggest improvements
- Include specification name and version
- Provide concrete examples

### Proposing Changes

1. **Small Changes** (typos, clarifications)
   - Open a PR directly

2. **Significant Changes** (new fields, breaking changes)
   - Open an issue first to discuss
   - Get consensus before implementation
   - Consider backwards compatibility

### Specification Writing Guidelines

#### Structure
```markdown
# Specification Name

## Overview
Brief description of what this specifies.

## Format
Detailed specification with examples.

## Schema
Link to JSON schema.

## Examples
Minimal and complex examples.

## Compatibility
Version history and migration notes.
```

#### Style
- Use clear, unambiguous language
- Provide both minimal and complex examples
- Define all terms
- Link to related specifications
- Include JSON schema for validation

#### Schema Guidelines
- Use JSON Schema Draft-07
- Provide examples that validate
- Document all fields with descriptions
- Mark required vs. optional fields

### Adding Examples

Examples should:
- Be self-contained and runnable
- Cover common use cases
- Show best practices
- Validate against the schema

### Versioning

- Backwards-compatible additions → MINOR version bump
- Breaking changes → MAJOR version bump
- Clarifications, typos → PATCH version bump

### Review Process

1. Submit PR with changes
2. Automated checks run (schema validation, link checking)
3. Maintainer review (usually within 1 week)
4. Address feedback
5. Merge + version bump

## Development Setup

```bash
# Clone repository
git clone https://github.com/semwerk/spec
cd specs

# Validate schemas
npm install -g ajv-cli
ajv validate -s formats/linkage-mapping/schema.json -d formats/linkage-mapping/examples/*.yaml

# Check links
npm install -g markdown-link-check
find . -name "*.md" -exec markdown-link-check {} \;
```

## Adding a New Specification

1. Create directory under appropriate category (formats/, taxonomies/, models/)
2. Write `spec.md` following the template
3. Create `schema.json` if applicable
4. Add examples/ directory with at least 2 examples
5. Update main README.md to link to new spec
6. Add entry to CHANGELOG.md

## Questions?

Open an issue or discussion for questions about:
- Specification interpretation
- Implementation guidance
- Compatibility concerns

## Code of Conduct

Be respectful, constructive, and welcoming. We're building open standards together.
