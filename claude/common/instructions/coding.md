# Coding Instructions

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should write
> and review code when assisting with any project. They are not designed or supported for use with other AI agents or
> models.

## Core Principles

### 1. Type Safety and Correctness

- **Use the strongest type system available** for your language
- **Avoid dynamic or untyped constructs** when static alternatives exist
- **Use explicit types** for function signatures and public APIs
- **Leverage language-specific type features** (generics, sum types, discriminated unions)
- **Prefer immutability** to prevent unintended mutations

### 2. Modern Language Patterns

#### Type Inference

- Let the compiler/interpreter infer types when obvious
- Be explicit for public APIs and complex return types

#### Generics and Parametric Polymorphism

- Use generics for reusable, type-safe functions and data structures
- Constrain type parameters when needed

#### Error Representation

- Use language-appropriate error types (Result, Either, Optional, exceptions)
- Make error states explicit in function signatures where possible

### 3. Asynchronous Patterns

- **Prefer async/await or equivalent** over raw callbacks or manual promise handling
- **Always handle errors** in async operations
- **Never ignore failed operations** - handle or propagate errors

#### Concurrent Operations

- Use parallel execution for independent operations
- Use sequential execution when order matters
- Implement proper cancellation and timeout patterns

### 4. Error Handling Strategy

#### Custom Error Types

**Recommended Pattern**:

- Create domain-specific error types with context
- Include error codes, messages, and relevant metadata
- Preserve error chains for debugging
- Capture stack traces where available

**Key Points**:

- Include meaningful error identifiers
- Add context properties relevant to the error type
- Use consistent error codes across the application

#### Error Propagation

- Create domain-specific error types
- Include context in errors for debugging
- Use error cause chains when supported
- Log errors at boundaries, not everywhere

### 5. Code Organization

#### Module Structure

**Recommended Structure**:

```
src/
├── errors/                       # Custom error types
├── services/                     # Business logic
│   └── {feature}/
│       ├── types/               # Type definitions
│       ├── {feature}.{ext}      # Implementation
│       └── {feature}.test.{ext} # Tests
├── utils/                        # Shared utilities
│   ├── logger.{ext}
│   └── config.{ext}
├── models/                       # Data models
├── controllers/                  # API controllers (if applicable)
└── main.{ext}                    # Entry point
```

**Organization Principles**:

- Group related functionality in feature directories
- Keep types, implementation, and tests colocated
- Separate concerns (errors, services, utils, models)

#### Dependency Injection

- Use constructor injection for dependencies
- Avoid tight coupling to concrete implementations
- Use interfaces/protocols for dependencies

#### Single Responsibility

- Each module should have one reason to change
- Keep functions small and focused (< 50 lines)
- Extract complex logic into separate functions

### 6. Testing Strategy

> **[Testing Instructions](testing.md)**

**Quick Reference**:

- Use Test-Driven Development (TDD) for all features
- Maintain minimum 80% code coverage (100% for critical paths)
- Mock all external dependencies
- Tests are collocated with implementation files

### 7. Logging and Monitoring

#### Structured Logging

- Use structured logging with context (key-value pairs)
- Include correlation IDs for request tracing
- Never log sensitive data (passwords, tokens, PII)

#### Log Levels

- **error**: Failures requiring attention
- **warn**: Potential issues
- **info**: Significant events (production default)
- **debug**: Detailed troubleshooting (development default)

### 8. Environment Configuration

#### Environment Variables

- Validate environment variables at startup
- Use environment files for local development
- Never commit secrets to version control
- Use different configs for different environments
- Fail fast on missing required configuration

### 9. Database Patterns

#### Connection Management

- Use connection pooling
- Implement graceful shutdown
- Handle connection errors with retries

#### Query Patterns

- Use parameterized queries to prevent injection attacks
- Use transactions for multi-step operations
- Implement proper error handling and rollback

#### Migrations

- Version control all schema changes
- Use migration tools appropriate to your stack
- Never modify existing migrations
- Test migrations with rollback

### 10. API Design (if applicable)

#### RESTful Conventions

- Use proper HTTP methods: GET, POST, PUT, PATCH, DELETE
- Return appropriate status codes
- Use consistent response formats
- Version APIs: `/api/v1/...`

#### Input Validation

- Validate all external input at boundaries
- Use schema validation libraries
- Return clear error messages

### 11. Performance Considerations

#### Optimization

- Profile before optimizing
- Use appropriate data structures
- Implement caching where beneficial
- Batch database operations
- Use streaming for large datasets

#### Resource Management

- Avoid resource leaks from unclosed connections
- Clear timers and scheduled tasks
- Remove event listeners when done
- Use appropriate lifecycle hooks

### 12. Code Quality

#### Linting and Formatting

- Use language-appropriate linters
- Use automatic formatters for consistency
- Run linters in CI/CD pipeline
- Configure pre-commit hooks

#### Code Review Checklist

- [ ] Functions have explicit types where appropriate
- [ ] Error handling is comprehensive
- [ ] Tests cover new functionality
- [ ] No hardcoded values (use constants/config)
- [ ] Logging is appropriate and structured
- [ ] No commented-out code
- [ ] Documentation updated if needed

### 13. Documentation

#### Code Comments

- Write self-documenting code (clear names)
- Comment the "why", not the "what"
- Use doc comments for public APIs
- Keep comments up to date

#### Doc Comment Guidelines

Document:

- Purpose of the function/method
- Parameters and their constraints
- Return values
- Exceptions/errors that may be thrown
- Usage examples for complex APIs

## Iterative Development Workflow

> This workflow details the code change process. For high-level task execution,
> see [CLAUDE.md Section 6](CLAUDE.md#6-task-execution-flow). For test-specific TDD cycles,
> see [testing.md](testing.md#test-driven-development-tdd).

### For Every Code Change:

1. **Understand Requirements**
    - Clarify the feature or bug
    - Identify affected components
    - Consider edge cases

2. **Write Tests First**
    - Follow TDD workflow (see [testing.md](testing.md))
    - Write failing tests, implement minimal solution, refactor

3. **Implement Minimal Solution**
    - Write just enough code to pass tests
    - Keep it simple and readable
    - Follow YAGNI (You Aren't Gonna Need It)

4. **Run All Tests**
    - Verify coverage meets requirements

5. **Refactor**
    - Improve code structure while tests pass
    - Extract duplicated code
    - Improve naming
    - Commit: `refactor: improve [component]`

6. **Code Review Self-Check**
    - Review your own changes
    - Check against style guide
    - Ensure documentation is updated

7. **Commit with Conventional Commits**
    - `feat: add user authentication`
    - `fix: resolve memory leak in connection pool`
    - `test: add integration tests for API`
    - `docs: update README with setup instructions`
    - `refactor: simplify error handling logic`

## Security Best Practices

### Input Validation

- Validate all external input
- Sanitize data before database operations
- Use parameterized queries

### Authentication & Authorization

- Never store passwords in plain text
- Use appropriate hashing algorithms for credentials
- Implement rate limiting
- Use secure token management with expiration

### Dependency Management

- Regularly update dependencies
- Audit for security vulnerabilities
- Use lock files for reproducible builds
- Review dependency licenses

### Secrets Management

- Use environment variables for secrets
- Never log secrets
- Rotate credentials regularly
- Use secret management tools in production

## Production Readiness

### Health Checks

- Implement health check endpoints
- Check all critical dependencies (database, external services)
- Return appropriate status codes

### Graceful Shutdown

- Stop accepting new requests
- Complete in-flight requests
- Close database connections
- Exit cleanly

### Monitoring

- Implement application metrics
- Track error rates
- Monitor response times
- Set up alerts for critical issues

## Naming Conventions

**Files**:

- Use consistent casing for your language (PascalCase, snake_case, kebab-case)
- Test files mirror implementation names

**Functions**:

- Use verb prefixes: `fetch`, `save`, `validate`, `create`
- Async functions should indicate asynchronous nature if language conventions require

**Variables**:

- Use consistent casing for your language
- Use ALL_CAPS for constants
- Prefer descriptive names over brevity

## Summary

When Claude Code generates code, always:

- ✅ Write tests first
- ✅ Use the strongest available type system
- ✅ Handle all errors appropriately
- ✅ Use appropriate async patterns
- ✅ Use structured logging
- ✅ Validate environment configuration
- ✅ Document public APIs
- ✅ Keep functions small and focused
- ✅ Use meaningful variable and function names
- ✅ Implement proper error handling with custom error types
- ✅ Think about security implications
- ✅ Consider performance and scalability
- ✅ Make code reviewable and maintainable
- ✅ Follow testing best practices
- ✅ Follow consistent naming conventions
- ✅ Organize code by feature/domain

**Remember**: Code is read more often than it's written. Optimize for readability and maintainability.
