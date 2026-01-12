# Project Definition

**Version:** 1.0.0
**Status:** Draft
**Format:** YAML

## Overview

Projects are the organizational units that group code repositories, business assets (documentation), and user journeys. A project represents a product, service, library, platform, or internal tool.

## Format

```yaml
version: "1"
kind: project

project:
  id: <string>                    # Unique project identifier (prj_*)
  key: <string>                   # Kebab-case key (unique within tenant)
  name: <string>                  # Display name
  type: <enum>                    # product | service | library | platform | internal
  description: <string>           # Project description
  status: <enum>                  # active | archived

  # Relationships
  parent_project: <project-ref>   # Optional parent project

  # Sources (code repositories)
  repositories:
    - id: <string>                # Repository identifier
      url: <string>               # Git URL
      is_primary: <boolean>       # Primary repo for this project
      path_filter: <string>       # Optional: filter to specific paths
      branch: <string>            # Default branch (default: main)

  # Assets (business docs, marketing, etc.)
  assets:
    - id: <string>                # Asset collection identifier
      type: <enum>                # documentation | marketing | operational | legal
      source: <enum>              # filesystem | gdocs | notion | confluence
      path: <string>              # Path or external ID
      is_primary: <boolean>       # Primary asset collection

metadata:
  owner: <string>                 # Owning team
  tech_stack: [<string>]          # Technologies used
  tags: [<string>]                # Classification tags
  components: [<string>]          # Sub-components
```

## Project Types

See [Project Types](./project-types.md) for taxonomy.

- `product` - Customer-facing product
- `service` - Backend microservice
- `library` - Reusable library/SDK
- `platform` - Multi-service platform
- `internal` - Internal tooling

## Examples

### Simple Service

```yaml
version: "1"
kind: project

project:
  id: prj_payment_api
  key: payment-api
  name: "Payment API"
  type: service
  description: "Core payment processing service"
  status: active

  repositories:
    - id: repo_payment_api_main
      url: github.com/acme/payment-api
      is_primary: true
      branch: main

  assets:
    - id: ast_payment_docs
      type: documentation
      source: filesystem
      path: docs/
      is_primary: true

metadata:
  owner: payments-team
  tech_stack:
    - Go
    - gRPC
    - PostgreSQL
  tags:
    - payments
    - fintech
```

### Multi-Repo Platform

```yaml
version: "1"
kind: project

project:
  id: prj_platform
  key: payment-platform
  name: "Payment Platform"
  type: platform
  description: "Complete payment processing platform"
  status: active

  repositories:
    - id: repo_api
      url: github.com/acme/payment-api
      is_primary: true
      branch: main

    - id: repo_sdk
      url: github.com/acme/payment-sdk
      is_primary: false
      branch: main

    - id: repo_dashboard
      url: github.com/acme/payment-dashboard
      is_primary: false
      branch: main

  assets:
    - id: ast_docs
      type: documentation
      source: filesystem
      path: docs/
      is_primary: true

    - id: ast_marketing
      type: marketing
      source: gdocs
      path: gdoc-abc123def456
      is_primary: false

    - id: ast_runbooks
      type: operational
      source: confluence
      path: confluence-789012
      is_primary: false

metadata:
  owner: platform-team
  tech_stack:
    - Go
    - React
    - PostgreSQL
    - Redis
  tags:
    - platform
    - microservices
  components:
    - api
    - sdk
    - dashboard
```

### Library with Parent

```yaml
version: "1"
kind: project

project:
  id: prj_auth_sdk
  key: auth-sdk
  name: "Authentication SDK"
  type: library
  description: "Reusable authentication library"
  status: active

  parent_project: project:@prj_platform

  repositories:
    - id: repo_auth_sdk
      url: github.com/acme/auth-sdk
      is_primary: true
      branch: main

  assets:
    - id: ast_sdk_docs
      type: documentation
      source: filesystem
      path: README.md
      is_primary: true

metadata:
  owner: auth-team
  tech_stack:
    - Go
  tags:
    - library
    - authentication
```

## Coordinates

Projects can be referenced using the coordinate system:

```yaml
project:@prj_payment_api                        # Basic reference
project:@prj_payment_api@v2.0.0                 # At specific version
```

## Relationships

### Parent-Child Hierarchy

```yaml
# Platform (parent)
project:
  id: prj_platform
  key: payment-platform

# Services (children)
project:
  id: prj_api
  key: payment-api
  parent_project: project:@prj_platform

project:
  id: prj_dashboard
  key: payment-dashboard
  parent_project: project:@prj_platform
```

### Multi-Project Journeys

Journeys can span multiple projects:

```yaml
journey:
  key: end-to-end-payment
  projects:
    - project:@prj_payment_api
    - project:@prj_payment_sdk
    - project:@prj_dashboard
  primary_project: project:@prj_payment_api
```

## Repository Mapping

### Code Sources

Projects map to one or more git repositories:

```yaml
repositories:
  - id: repo_backend
    url: github.com/acme/backend
    is_primary: true
    path_filter: "services/payment/**"    # Optional: only this path

  - id: repo_shared
    url: github.com/acme/shared-libs
    is_primary: false
    path_filter: "packages/payment/**"
```

**Path filters** allow large monorepos to be sliced into logical projects.

### Asset Mapping

Projects map to documentation/content collections:

```yaml
assets:
  - id: ast_dev_docs
    type: documentation
    source: filesystem
    path: docs/

  - id: ast_marketing_site
    type: marketing
    source: gdocs
    path: gdoc-folder-xyz789

  - id: ast_runbooks
    type: operational
    source: notion
    path: notion-database-abc123
```

## Validation

- Project ID MUST start with `prj_`
- Key MUST be kebab-case, unique within tenant
- Type MUST be from taxonomy
- At least one repository or asset collection required
- If `is_primary: true`, only one repo and one asset collection can be primary

## Use Cases

### 1. Multi-Repo Coordination

```yaml
# Platform with 5 repos
project:
  key: platform
  repositories:
    - api-repo
    - sdk-repo
    - dashboard-repo
    - docs-repo
    - infrastructure-repo
```

### 2. Multi-Source Documentation

```yaml
# Docs in multiple systems
project:
  key: product
  assets:
    - type: documentation
      source: filesystem
      path: technical-docs/
    - type: marketing
      source: gdocs
      path: gdoc-folder-xyz
    - type: operational
      source: confluence
      path: confluence-space-OPS
```

### 3. Version-Specific Projects

```yaml
# Reference project at specific version
journey:
  nodes:
    - code_refs:
        - code:@cfn_auth@v2.0.0/Login
      project: project:@prj_auth@v2.0.0
```

## See Also

- [Project Types](./project-types.md) - Project type taxonomy
- [Project-Repository Mapping](./project-repository-mapping.md) - Repo details
- [Project-Asset Mapping](./project-asset-mapping.md) - Asset details
- [Version Specification](../versions/version-specification.md) - Versioning
- [Coordinate System](../coordinates/COORDINATE_SPEC.md) - Reference format
