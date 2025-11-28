# Research Instructions for TypeScript Node.js Applications

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should
> conduct research when assisting with TypeScript Node.js projects. They are not designed or supported for use with other
> AI agents or models.

This document extends the general [research.md](../common/instructions/research.md) with TypeScript and Node.js
ecosystem-specific guidance.

---

## Foundation

All research should follow the core principles defined in the general research instructions:

- **Research-Driven Development workflow**
- **Verification from multiple sources**
- **Documentation of findings with sources**
- **No assumptions or hallucinations**

This document adds TypeScript/Node.js-specific research patterns, templates, and workflows.

---

## TypeScript Ecosystem Research

### Package Research Checklist

When evaluating an npm package:

1. **Package quality indicators**
    - Weekly downloads and trend
    - Last publish date
    - Open issues vs. closed ratio
    - Maintenance activity (commits in last 6 months)
    - TypeScript support (built-in types vs. @types/*)

2. **Type safety**
    - Native TypeScript or DefinitelyTyped
    - Type coverage and accuracy
    - Generic support quality
    - Strict mode compatibility

3. **Compatibility**
    - Node.js version requirements
    - ESM vs. CommonJS support
    - Peer dependency requirements
    - Bundle size (for shared libraries)

4. **Security**
    - npm audit results
    - Snyk vulnerability database
    - Dependency tree depth
    - Known CVEs

### Package Research Template

```markdown
# [Package Name] Research

## Package Info

- npm: https://www.npmjs.com/package/[name]
- Repository:
- Documentation:
- TypeScript: Native / @types/[name] / None

## Quality Metrics

- Weekly downloads:
- Last published:
- License:
- Bundle size:

## TypeScript Support

- Type definitions: Built-in / DefinitelyTyped / Community / None
- Strict mode compatible: Yes / No / Partial
- Generic support: Good / Limited / None
- Type accuracy: High / Medium / Low

## Node.js Compatibility

- Minimum Node version:
- ESM support: Yes / No / Dual
- CommonJS support: Yes / No

## Dependencies

- Direct dependencies:
- Peer dependencies:
- Dependency tree depth:

## Security

- npm audit: Clean / Issues found
- Known vulnerabilities:
- Last security review:

## API Overview

```typescript
// Key exports and usage patterns
import { Feature } from '[package]';
```

## Pros
- 

## Cons
- 

## Alternatives

| Package | TypeScript | Downloads | Last Update |
|---------|------------|-----------|-------------|
|         |            |           |             |

## Recommendation

- Use / Avoid / Conditional

## Sources

1.
2.

```

---

## Node.js Runtime Research

### Node.js Version Research

When researching Node.js version requirements:

1. **LTS status and EOL dates**
2. **Feature availability by version**
3. **Performance characteristics**
4. **Security support timeline**
5. **Compatibility with key dependencies**

### Node.js Feature Research Template

```markdown
# [Feature Name] Research

## Availability
- Introduced in: Node.js vX.X.X
- Stable since: Node.js vX.X.X
- Current LTS support: Yes / No

## TypeScript Support
- Type definitions accurate: Yes / No
- @types/node version required:

## Usage
```typescript
// Example with proper types
```

## Considerations

- Performance:
- Compatibility:
- Edge cases:

## Sources

1. Node.js docs:
2. TypeScript types:

```

---

## TypeScript Configuration Research

### tsconfig.json Research

When researching TypeScript compiler options:

1. **Option purpose and behavior**
2. **Compatibility implications**
3. **Performance impact**
4. **Interaction with other options**
5. **Migration path from current config**

### Compiler Option Research Template

```markdown
# tsconfig: [Option Name] Research

## Option
- Name: `[optionName]`
- Category: [Type Checking / Module Resolution / Emit / etc.]
- Default: [value]

## Purpose
[What this option does]

## Values
| Value | Behavior |
|-------|----------|
|       |          |

## Recommended Setting
- Value: 
- Rationale:

## Interactions
- Requires: [other options]
- Conflicts with: [other options]
- Affected by: [other options]

## Migration Notes
[How to migrate existing code]

## Sources
1. TypeScript docs:
2. TypeScript GitHub:
```

---

## API Client Research (TypeScript-Specific)

### API Client Library Evaluation

When researching API client libraries or generating clients:

1. **Type generation quality**
    - OpenAPI/Swagger support
    - Type inference accuracy
    - Runtime validation options

2. **Error handling patterns**
    - Typed error responses
    - Discriminated unions for errors
    - Exception vs. Result patterns

3. **Request/response typing**
    - Generic request builders
    - Response type inference
    - Middleware type safety

### API Client Research Template

```markdown
# [API Name] TypeScript Client Research

## Official SDK

- Package:
- TypeScript support: Native / Partial / None
- Version:

## Type Definitions

```typescript
// Key interfaces
interface RequestParams {
}

interface ResponseData {
}
```

## Authentication Types

```typescript
// Auth configuration
interface AuthConfig {
}
```

## Error Types

```typescript
// Error discrimination
type ApiError =
  | { code: 'NOT_FOUND'; message: string }
  | { code: 'UNAUTHORIZED'; message: string };
```

## Client Initialization

```typescript
// Recommended setup
import {Client} from '[package]';

const client = new Client({
  // config
});
```

## Type Safety Gaps

- [Areas where types are incomplete or incorrect]

## Workarounds

```typescript
// Type fixes or augmentations
declare module '[package]' {
  // ...
}
```

## Sources

1.
2.

```

---

## Database Research (TypeScript-Specific)

### ORM/Query Builder Evaluation

When researching database tools for TypeScript:

1. **Type inference quality**
   - Schema-to-type generation
   - Query result typing
   - Migration type safety

2. **TypeScript integration**
   - Strict mode compatibility
   - Generic support
   - Type narrowing in queries

3. **Developer experience**
   - IDE autocompletion
   - Error messages quality
   - Documentation with TypeScript examples

### Database Library Research Template

```markdown
# [Library Name] Research

## Overview
- Type: ORM / Query Builder / Driver
- Database support:
- TypeScript: Native / Augmented

## Type Safety

### Schema Definition
```typescript
// How schemas/models are defined
```

### Query Typing

```typescript
// How queries are typed
const result = await db.query<ResultType>(...);
```

### Migration Types

```typescript
// Migration type safety
```

## Type Inference Quality

- Select queries: Full / Partial / None
- Insert/Update: Full / Partial / None
- Joins: Full / Partial / None
- Aggregations: Full / Partial / None

## Strict Mode Compatibility

- strictNullChecks: Yes / No
- noImplicitAny: Yes / No
- Known issues:

## Performance Considerations

- Connection pooling:
- Query compilation:
- Type overhead at runtime:

## Comparison

| Feature        | [Library] | Alternative A | Alternative B |
|----------------|-----------|---------------|---------------|
| Type inference |           |               |               |
| Migrations     |           |               |               |
| Performance    |           |               |               |

## Recommendation

[Use / Avoid / Conditional with rationale]

## Sources

1.
2.

```

---

## Testing Library Research

### Test Framework Evaluation

When researching testing tools for TypeScript:

1. **TypeScript support**
   - Native vs. transformer required
   - Type definitions quality
   - Mock typing support

2. **Integration with ecosystem**
   - ESM support
   - Coverage tools
   - IDE integration

### Testing Library Research Template

```markdown
# [Library Name] Testing Research

## Overview
- Type: Test Runner / Assertion / Mocking / E2E
- TypeScript support: Native / Transformer

## Configuration
```typescript
// vitest.config.ts or jest.config.ts
```

## Type Support

### Test Definitions

```typescript
// How tests are typed
describe('Feature', () => {
  it('should work', () => {
    // ...
  });
});
```

### Mock Typing

```typescript
// How mocks are typed
const mockFn = vi.fn<[string], number>();
```

### Assertion Typing

```typescript
// Type-safe assertions
expect(value).toBe<ExpectedType>(expected);
```

## Coverage Integration

- Provider:
- Configuration:

## Comparison with [testing.md](testing.md)

- Aligns with testing instructions: Yes / No
- Gaps:

## Sources

1.
2.

```

---

## Build Tool Research

### Bundler/Compiler Evaluation

When researching build tools:

1. **TypeScript compilation**
   - Type checking integration
   - Declaration file generation
   - Source map support

2. **Module handling**
   - ESM/CJS output
   - Path alias resolution
   - Tree shaking

3. **Development experience**
   - Watch mode performance
   - Hot reload support
   - Error reporting

### Build Tool Research Template

```markdown
# [Tool Name] Research

## Overview
- Purpose: Bundler / Compiler / Both
- TypeScript handling: Native / Plugin

## TypeScript Integration

### Type Checking
- During build: Yes / No
- Separate step required: Yes / No

### Configuration
```typescript
// esbuild.config.ts, vite.config.ts, etc.
```

### Output Formats

- ESM: Yes / No
- CJS: Yes / No
- Declaration files: Yes / No

## Path Aliases

```typescript
// Configuration for @/ paths
```

## Performance

- Cold start:
- Incremental:
- Watch mode:

## Comparison

| Feature           | [Tool] | Alternative |
|-------------------|--------|-------------|
| Type checking     |        |             |
| Build speed       |        |             |
| Config complexity |        |             |

## Integration with Project

- Aligns with [coding.md](coding.md): Yes / No
- Docker integration ([docker.md](docker.md)): Yes / No

## Sources

1.
2.

```

---

## Type Definition Research

### @types/* Package Research

When researching DefinitelyTyped packages:

1. **Version alignment with library**
2. **Type accuracy and completeness**
3. **Maintenance status**
4. **Known issues**

### Type Definition Research Template

```markdown
# @types/[package] Research

## Package Info
- Library version:
- Types version:
- DefinitelyTyped: https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/[package]

## Type Coverage
- Core API: Full / Partial / Missing
- Optional features: Full / Partial / Missing
- Callbacks/Events: Full / Partial / Missing

## Known Issues
- Open issues:
- Type inaccuracies:
- Missing declarations:

## Augmentation Needed
```typescript
// Custom type augmentations
declare module '[package]' {
  // fixes
}
```

## Alternative Approaches

- Use `any` for: [specific cases]
- Custom types for: [specific cases]

## Sources

1.
2.

```

---

## Research Workflows (TypeScript-Specific)

### Workflow: New Dependency Addition

1. **Identify the need**
   - What problem does this solve?
   - Are there existing solutions in the project?

2. **Research candidates**
   - Use Package Research Template
   - Evaluate at least 2-3 alternatives

3. **Verify TypeScript support**
   - Check type definitions
   - Test with strict mode
   - Review type accuracy

4. **Security check**
   - Run `npm audit`
   - Check Snyk database
   - Review dependency tree

5. **Integration assessment**
   - Compatibility with existing packages
   - Alignment with [coding.md](coding.md) patterns
   - Testing approach per [testing.md](testing.md)

6. **Document decision**
   - Rationale for selection
   - Alternatives considered
   - Known limitations

### Workflow: TypeScript Upgrade Research

1. **Review changelog**
   - Breaking changes
   - New features
   - Deprecations

2. **Assess impact**
   - Incompatible patterns in codebase
   - Required tsconfig changes
   - Affected dependencies

3. **Test compatibility**
   - Run type checking
   - Check @types/* compatibility
   - Verify build tools

4. **Plan migration**
   - Incremental approach
   - Testing strategy
   - Rollback plan

### Workflow: API Integration Research

1. **Follow general API Research** (from common research.md)

2. **TypeScript-specific evaluation**
   - Official SDK quality
   - Type definitions accuracy
   - Error type completeness

3. **Client implementation approach**
   - Use SDK vs. custom client
   - Type generation options
   - Validation layer needs

4. **Integration with project patterns**
   - Error handling per [coding.md](coding.md#4-error-handling-strategy)
   - Testing approach per [testing.md](testing.md)

---

## Source Priority (TypeScript-Specific)

### Tier 1: Official Sources
1. TypeScript documentation (typescriptlang.org)
2. Node.js documentation (nodejs.org)
3. Package official documentation
4. GitHub release notes

### Tier 2: Verified Technical Sources
1. TypeScript GitHub issues (confirmed by maintainers)
2. Node.js GitHub issues (confirmed)
3. DefinitelyTyped discussions
4. Package maintainer responses

### Tier 3: Community Sources
1. Stack Overflow (accepted answers, verify)
2. Dev.to / Medium (verify against Tier 1)
3. Reddit r/typescript, r/node (verify)

Always verify Tier 2 and 3 against official documentation.

---

## Integration with Other Instructions

| Research Area | Implementation Reference |
|---------------|-------------------------|
| Package selection | [coding.md - Dependency Injection](coding.md#5-code-organization) |
| API client patterns | [coding.md - Asynchronous Patterns](coding.md#3-asynchronous-patterns) |
| Database tools | [coding.md - Database Patterns](coding.md#9-database-patterns) |
| Testing libraries | [testing.md](testing.md) |
| Build configuration | [docker.md - Dockerfile Template](docker.md#dockerfile-template) |

---

## Research Quality Checklist (TypeScript-Specific)

Before completing research:

### Type Safety
- [ ] TypeScript support verified
- [ ] Strict mode compatibility confirmed
- [ ] Type definitions reviewed for accuracy
- [ ] Generic patterns evaluated

### Ecosystem Fit
- [ ] Node.js version compatibility confirmed
- [ ] ESM/CJS handling understood
- [ ] Integration with existing packages verified
- [ ] Build tool compatibility confirmed

### Documentation
- [ ] TypeScript examples included
- [ ] Type definitions documented
- [ ] Known type issues listed
- [ ] Workarounds provided where needed

### Project Alignment
- [ ] Follows [coding.md](coding.md) patterns
- [ ] Testing approach aligns with [testing.md](testing.md)
- [ ] Docker considerations per [docker.md](docker.md) addressed

---

## Summary

This document extends the general research instructions with TypeScript/Node.js-specific guidance for:

- **Package evaluation** with type safety focus
- **TypeScript configuration** research
- **API client** type quality assessment
- **Database tools** type inference evaluation
- **Build tools** and compilation research
- **Type definitions** accuracy verification

Always reference the general [research.md](../common/instructions/research.md) for foundational research methodology, then apply TypeScript-specific patterns from this document.
