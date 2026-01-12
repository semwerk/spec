# Symbol Taxonomy

**Version:** 1.0.0
**Status:** Draft

## Overview

Language-agnostic taxonomy for classifying code symbols. Enables consistent symbol classification across programming languages.

## Symbol Kinds

### Callable Units

**function**
- Standalone function
- Top-level function
- Examples: `main()`, `calculateTotal()`, `handleRequest()`

**method**
- Function attached to class/struct
- Instance or class method
- Examples: `User.save()`, `Parser.parse()`, `Array.prototype.map()`

### Type Definitions

**class**
- Class definition
- OOP class construct
- Examples: `User`, `HttpServer`, `DatabaseConnection`

**struct**
- Struct/record type
- Data structure definition
- Examples: Go `type User struct`, C `struct user`

**interface**
- Interface/protocol definition
- Abstract contract
- Examples: `Serializer`, `Repository`, `Handler`

**type**
- Type alias or custom type
- Type definition
- Examples: TypeScript `type UserID = string`, Go `type Handler func()`

**enum**
- Enumeration type
- Enum definition
- Examples: `Status`, `Color`, `HttpMethod`

### Constants & Variables

**const**
- Constant declaration
- Immutable value
- Examples: `MAX_RETRIES`, `API_VERSION`, `DEFAULT_TIMEOUT`

**var**
- Variable declaration
- Module-level variable
- Examples: `globalConfig`, `defaultLogger`

**property**
- Class/object property
- Field/attribute
- Examples: `user.email`, `config.timeout`

### Modules & Packages

**module**
- Module definition
- Package/namespace
- Examples: Python module, JavaScript ES6 module, Go package

## Visibility Levels

**public**
- Exported/public symbol
- Accessible outside module
- Examples: Capitalized in Go, `export` in JS, `public` in Java

**private**
- Internal to module/class
- Not exported
- Examples: Lowercase in Go, no `export` in JS, `private` in Java

**protected**
- Accessible to subclasses
- OOP protected access
- Examples: `protected` in Java/C++/TypeScript

**internal**
- Package-private
- Accessible within package
- Examples: Go `internal/` package

**package**
- Package/module level visibility
- Not exported but accessible in package
- Examples: Java default visibility, Go lowercase

## Test Types

**unit**
- Unit test
- Tests single function/method
- Examples: `TestCalculateTotal`, `test_parse_user`

**integration**
- Integration test
- Tests multiple components
- Examples: `TestDatabaseIntegration`, `test_api_flow`

**benchmark**
- Performance benchmark
- Measures execution time
- Examples: `BenchmarkParser`, `benchmark_sort`

**example**
- Example test (Go)
- Demonstrates usage
- Examples: `ExampleUser_Save`

**fuzz**
- Fuzz test
- Random input testing
- Examples: `FuzzTokenizer`

## Language Support

| Language | function | method | class | struct | interface | type | enum | const | var |
|----------|----------|--------|-------|--------|-----------|------|------|-------|-----|
| Go | ✓ | ✓ | - | ✓ | ✓ | ✓ | - | ✓ | ✓ |
| Python | ✓ | ✓ | ✓ | - | - | - | ✓ | ✓ | ✓ |
| JavaScript | ✓ | ✓ | ✓ | - | - | - | - | ✓ | ✓ |
| TypeScript | ✓ | ✓ | ✓ | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| Java | ✓ | ✓ | ✓ | - | ✓ | - | ✓ | ✓ | ✓ |
| Rust | ✓ | ✓ | - | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| C++ | ✓ | ✓ | ✓ | ✓ | - | ✓ | ✓ | ✓ | ✓ |
| C | ✓ | - | - | ✓ | - | ✓ | ✓ | ✓ | ✓ |
| Ruby | ✓ | ✓ | ✓ | - | - | - | - | ✓ | ✓ |
| PHP | ✓ | ✓ | ✓ | - | ✓ | - | - | ✓ | ✓ |
| C# | ✓ | ✓ | ✓ | ✓ | ✓ | - | ✓ | ✓ | ✓ |
| Kotlin | ✓ | ✓ | ✓ | - | ✓ | ✓ | ✓ | ✓ | ✓ |
| Swift | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

## Symbol Naming Convention

Format: `<file_path>:<symbol_name>`

### Examples by Language

**Go:**
```
pkg/auth/service.go:Authenticate          # function
pkg/auth/service.go:Service.Login         # method
pkg/auth/types.go:User                    # struct
pkg/auth/types.go:Authenticator           # interface
```

**Python:**
```
src/auth/service.py:authenticate          # function
src/auth/service.py:AuthService.login     # method
src/auth/models.py:User                   # class
```

**TypeScript:**
```
src/auth/service.ts:authenticate          # function
src/auth/service.ts:AuthService.login     # method
src/auth/types.ts:User                    # interface
src/auth/types.ts:UserRole                # enum
```

**Java:**
```
com/acme/auth/AuthService.java:AuthService              # class
com/acme/auth/AuthService.java:AuthService.authenticate # method
com/acme/auth/UserRole.java:UserRole                    # enum
```

## Validation

- Symbol kind MUST be from this taxonomy
- Visibility MUST be from defined levels
- Test type MUST be from defined types (if applicable)
- Symbol naming MUST follow `file:symbol` convention

## See Also

- [Linkage Mapping](../formats/linkage-mapping.md) - How symbols link to docs
- [Change Types](./change-types.md) - How symbol changes are classified
