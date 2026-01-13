# Changelog

All notable changes to Semwerk specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

**Data Interchange Formats:**
- Linkage Mapping format specification
- Segment Markers syntax specification
- Frontmatter schema specification

**Taxonomies:**
- Segment Taxonomy vocabulary (14 types)
- Symbol Taxonomy vocabulary (9 kinds, 13 languages)
- Audience Role Taxonomy (10 roles)
- Change Type Taxonomy (code, API, CLI, config changes)
- Node Type Taxonomy (stage, milestone, decision, jump_off)
- Project Type Taxonomy (product, service, library, platform, internal)
- Lifecycle States (7 states: draft → eol)
- Relationship Types (5 types: parent, related, implements, documents, depends_on)

**Data Models:**
- Provenance Model specification
- Classification Config schema

**Project & Version:**
- Project Definition format
- Version Specification (semver + freeform)
- Version Lineage (ancestry graphs)

**User Journeys:**
- Journey Graph format (DAG)
- Entry/Exit Points
- Success Metrics schema

**Semantic Entities:**
- Concept Definition and Graph
- Persona format
- SpecBook Entities (task, problem, solution, outcome, validation, next_step, impact, feature)
- Terminology/Glossary format

**Coordinates:**
- Unified Coordinate System: `type:@id[@version][/segment[.subheading]][#checksum]`

**Reference Implementations:**
- semspec-ts (TypeScript/npm) with tests
- semspec-python (Python/pip) with tests
- semspec-go (Go module) with tests

## [0.1.0] - 2026-01-10

### Added
- Initial pre-release of Semwerk specification
- MIT License
- Contributing guidelines
- Repository structure

[Unreleased]: https://github.com/semwerk/spec/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/semwerk/spec/releases/tag/v0.1.0
