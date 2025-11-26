# Coding Instructions for TypeScript Node.js Applications

> General best practices for TypeScript Node.js development. These instructions are project-agnostic and apply to any new application or service.

## Core Principles

### 1. Type Safety First
- **Always enable strict mode** in `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "strict": true,
      "strictNullChecks": true,
      "strictFunctionTypes": true,
      "noImplicitAny": true,
      "noImplicitThis": true,
      "alwaysStrict": true
    }
  }
  ```
- **Avoid `any` type** - use `unknown` if type is truly unknown, then narrow with type guards
- **Use explicit return types** for all functions, especially public APIs
- **Leverage union types and discriminated unions** for complex state modeling
- **Use `readonly` and `const` assertions** to prevent unintended mutations

### 2. Modern TypeScript Patterns

#### Type Inference
- Let TypeScript infer types when obvious, but be explicit for public APIs
- Use type inference for variable declarations: `const user = getUser()` instead of `const user: User = getUser()`

#### Generics
- Use generics for reusable, type-safe functions and classes
- Constrain generics with `extends` when needed: `<T extends BaseType>`

#### Utility Types
- Leverage built-in utility types: `Partial<T>`, `Pick<T, K>`, `Omit<T, K>`, `Record<K, V>`
- Create custom utility types for domain-specific transformations

#### Type Guards
- Implement custom type guards using `is` keyword:
  ```typescript
  function isError(value: unknown): value is Error {
    return value instanceof Error;
  }
  ```

### 3. Asynchronous Patterns

#### Async/Await
- **Prefer async/await** over raw promises for better readability
- **Always handle errors** with try/catch blocks
- **Never ignore promise rejections** - handle or propagate errors

#### Error Handling
```typescript
async function fetchData(): Promise<Data> {
  try {
    const response = await api.getData();
    return response;
  } catch (error) {
    if (error instanceof ApiError) {
      logger.error('API error', { error: error.message, code: error.code });
      throw new DataFetchError('Failed to fetch data', { cause: error });
    }
    throw error;
  }
}
```

#### Concurrent Operations
- Use `Promise.all()` for parallel operations
- Use `Promise.allSettled()` when some failures are acceptable
- Use `Promise.race()` for timeout patterns

### 4. Error Handling Strategy

#### Custom Error Classes
```typescript
class AppError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}
```

#### Error Propagation
- Create domain-specific error classes
- Include context in errors for debugging
- Use error cause chains (ES2022): `{ cause: originalError }`
- Log errors at boundaries, not everywhere

### 5. Code Organization

#### Module Structure

**Recommended Structure**:
```
src/
├── errors/                       # Custom error classes
│   ├── ApiError.ts
│   ├── ValidationError.ts
│   └── DatabaseError.ts
├── services/                     # Business logic services
│   └── {feature}/
│       ├── types/               # TypeScript interfaces
│       ├── {feature}.ts         # Implementation
│       └── {feature}.test.ts    # Tests
├── utils/                        # Shared utilities
│   ├── logger.ts
│   └── config.ts
├── models/                       # Data models (if applicable)
├── controllers/                  # API controllers (if applicable)
└── index.ts                      # Entry point
```

**Organization Principles**:
- Group related functionality in feature directories
- Keep types, implementation, and tests colocated
- Tests live alongside implementation files
- Separate concerns (errors, services, utils, models)

#### Dependency Injection
- Use constructor injection for dependencies
- Avoid tight coupling to concrete implementations
- Use interfaces for dependencies

#### Single Responsibility
- Each module should have one reason to change
- Keep functions small and focused (< 50 lines)
- Extract complex logic into separate functions

### 6. Testing Strategy

#### Test-Driven Development (TDD)
**For every new feature or bug fix:**
1. **Write the test first** that describes the expected behavior
2. **Run the test** and watch it fail
3. **Write minimal code** to make the test pass
4. **Refactor** while keeping tests green
5. **Commit** with tests

#### Test Structure
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create a user with valid data', async () => {
      // Arrange
      const userData = { email: 'test@example.com', name: 'Test User' };
      const mockRepo = createMockUserRepository();
      const service = new UserService(mockRepo);

      // Act
      const user = await service.createUser(userData);

      // Assert
      expect(user).toBeDefined();
      expect(user.email).toBe(userData.email);
      expect(mockRepo.save).toHaveBeenCalledWith(expect.objectContaining(userData));
    });

    it('should throw ValidationError for invalid email', async () => {
      // Arrange
      const userData = { email: 'invalid-email', name: 'Test User' };
      const service = new UserService(createMockUserRepository());

      // Act & Assert
      await expect(service.createUser(userData)).rejects.toThrow(ValidationError);
    });
  });
});
```

#### Test Coverage Requirements
- **Minimum 80% code coverage** for all new code
- **100% coverage for critical paths**: authentication, payment, data integrity
- **Test all edge cases**: null, undefined, empty arrays, boundary values
- **Test error scenarios**: network failures, invalid input, timeouts

#### Test Types

**Unit Tests**
- Test individual functions and classes in isolation
- Mock all external dependencies
- Fast execution (< 1ms per test)
- File naming: `*.test.ts` or `*.spec.ts`

**Integration Tests**
- Test multiple components working together
- Use test database or in-memory implementations
- Test API endpoints, database operations
- File naming: `*.integration.test.ts`

**E2E Tests (when applicable)**
- Test complete user flows
- Use real or staging environment
- Limited number, focusing on critical paths

#### Testing Tools

**Recommended Setup**:
- **Test Runner**: Vitest or Jest
- **Coverage**: @vitest/coverage-v8 or Jest coverage
- **Assertion Library**: Built-in with test runner
- **Mocking**: `vi.mock()` (Vitest) or `jest.mock()` (Jest)
- **Target Coverage**: Minimum 80%, ideally 100%

**Test Runner Configuration**:
- Single run mode for CI/CD
- Watch mode for development
- Coverage reports with appropriate provider

#### Mocking Best Practices

**Node.js Module Mocking**:
```typescript
// Mock Node.js built-in modules
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import * as fs from 'node:fs/promises';

vi.mock('node:fs/promises');

describe('writeProjects', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should write file', async () => {
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));

    // Test implementation
  });
});
```

**Timers and Time-based Tests**:
```typescript
import { beforeEach, afterEach, vi } from 'vitest';

beforeEach(() => {
  vi.useFakeTimers();
  vi.setSystemTime(new Date('2025-11-25T10:30:45.123Z'));
});

afterEach(() => {
  vi.useRealTimers();
});
```

**External API Mocking**:
```typescript
// Mock global fetch
globalThis.fetch = vi.fn();

beforeEach(() => {
  vi.mocked(fetch).mockResolvedValue({
    ok: true,
    json: async () => ({ data: 'mock' }),
  } as Response);
});
```

**Reusable Mock Factories**:
```typescript
// Create reusable mock factories
function createMockDatabase(): Database {
  return {
    query: vi.fn(), // or jest.fn()
    transaction: vi.fn(),
    close: vi.fn(),
  };
}

// Use partial mocks when needed
const mockLogger = {
  error: vi.fn(), // or jest.fn()
  warn: vi.fn(),
  info: vi.fn(),
} as Logger;
```

**General Testing Patterns**:
1. **Mock external dependencies**: Always mock `fs`, `fetch`, and other I/O
2. **Test error paths**: Verify behavior when mocks reject/throw
3. **Use fake timers**: For schedulers and time-dependent logic
4. **Clear mocks**: Reset mock state between tests
5. **Verify calls**: Check that dependencies are called correctly

### 7. Logging and Monitoring

#### Structured Logging
```typescript
// Use structured logging with context
logger.info('User created', {
  userId: user.id,
  email: user.email,
  timestamp: new Date().toISOString(),
});

// Log levels: error, warn, info, debug
// Production: info and above
// Development: debug and above
```

#### Logging Libraries
- Use `pino` or `winston` for structured JSON logging
- Include correlation IDs for request tracing
- Never log sensitive data (passwords, tokens, PII)

### 8. Environment Configuration

#### Environment Variables
```typescript
// config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  LOG_LEVEL: z.enum(['error', 'warn', 'info', 'debug']).default('info'),
});

export const env = envSchema.parse(process.env);
```

#### Configuration Best Practices
- Validate environment variables at startup
- Use `.env` files for local development
- Never commit secrets to version control
- Use different configs for different environments

### 9. Database Patterns

#### Connection Management
- Use connection pooling
- Implement graceful shutdown
- Handle connection errors with retries

#### Query Patterns
```typescript
// Use parameterized queries to prevent SQL injection
const user = await db.query(
  'SELECT * FROM users WHERE email = $1',
  [email]
);

// Use transactions for multi-step operations
await db.transaction(async (trx) => {
  await trx.query('INSERT INTO users ...');
  await trx.query('INSERT INTO profiles ...');
});
```

#### Migrations
- Version control all schema changes
- Use migration tools (e.g., `node-pg-migrate`, `kysely`)
- Never modify existing migrations
- Test migrations with rollback

### 10. API Design (if applicable)

#### RESTful Conventions
- Use proper HTTP methods: GET, POST, PUT, PATCH, DELETE
- Return appropriate status codes
- Use consistent response formats
- Version APIs: `/api/v1/...`

#### Input Validation
```typescript
// Use Zod or similar for runtime validation
const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().positive().optional(),
});

type CreateUserInput = z.infer<typeof createUserSchema>;
```

### 11. Performance Considerations

#### Optimization
- Profile before optimizing
- Use appropriate data structures
- Implement caching where beneficial
- Batch database operations
- Use streaming for large datasets

#### Memory Management
- Avoid memory leaks from unclosed connections
- Clear timers and intervals
- Remove event listeners when done
- Use weak references when appropriate

### 12. Code Quality

#### Linting and Formatting
- Use ESLint with TypeScript rules
- Use Prettier for consistent formatting
- Run linters in CI/CD pipeline
- Configure pre-commit hooks

#### Code Review Checklist
- [ ] All functions have explicit return types
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
- Use JSDoc for public APIs
- Keep comments up to date

#### JSDoc Example
```typescript
/**
 * Fetches user data from the API with retry logic.
 *
 * @param userId - The unique identifier of the user
 * @param options - Optional configuration for the request
 * @returns A promise that resolves to the user data
 * @throws {ApiError} When the API request fails after retries
 * @throws {ValidationError} When the userId is invalid
 */
async function fetchUser(
  userId: string,
  options?: FetchOptions
): Promise<User> {
  // Implementation
}
```

## Iterative Development Workflow

### For Every Code Change:

1. **Understand Requirements**
   - Clarify the feature or bug
   - Identify affected components
   - Consider edge cases

2. **Write Tests First**
   - Write failing test(s) for new behavior
   - Run tests to confirm they fail
   - Commit: `test: add tests for [feature]`

3. **Implement Minimal Solution**
   - Write just enough code to pass tests
   - Keep it simple and readable
   - Follow YAGNI (You Aren't Gonna Need It)

4. **Run All Tests**
   - Ensure all tests pass
   - Check code coverage
   - Fix any failing tests

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
- Use bcrypt or argon2 for password hashing
- Implement rate limiting
- Use JWT tokens with expiration

### Dependency Management
- Regularly update dependencies
- Audit for security vulnerabilities: `npm audit`
- Use lock files (`pnpm-lock.yaml`)
- Review dependency licenses

### Secrets Management
- Use environment variables for secrets
- Never log secrets
- Rotate credentials regularly
- Use secret management tools in production

## Production Readiness

### Health Checks
```typescript
// Implement health check endpoint
app.get('/health', async (req, res) => {
  const dbHealth = await checkDatabase();
  const apiHealth = await checkExternalAPI();

  const health = {
    status: dbHealth && apiHealth ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    checks: { database: dbHealth, api: apiHealth },
  };

  res.status(health.status === 'healthy' ? 200 : 503).json(health);
});
```

### Graceful Shutdown
```typescript
process.on('SIGTERM', async () => {
  logger.info('SIGTERM received, shutting down gracefully');

  // Stop accepting new requests
  server.close(() => {
    logger.info('HTTP server closed');
  });

  // Close database connections
  await database.close();

  // Exit process
  process.exit(0);
});
```

### Monitoring
- Implement application metrics
- Track error rates
- Monitor response times
- Set up alerts for critical issues

## General Development Guidelines

### Type Definitions
- **Use `interface` for**:
  - API response shapes
  - Data contracts and object structures
  - Types that may need extension or declaration merging
- **Use `type` for**:
  - Union types, intersections
  - Utility types and mapped types
  - Function signatures

**Example**:
```typescript
// Good: Interface for API response
export interface ApiResponse {
  totalItems: number;
  items: unknown[];
}

// Good: Type for union
type Status = 'pending' | 'in_progress' | 'completed';

// Good: Type for function signature
type TaskFunction = () => Promise<void>;
```

### Error Class Pattern

**Recommended Pattern**:
```typescript
export class CustomError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500,
    public readonly context?: Record<string, unknown>
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Example usage
export class ApiError extends CustomError {
  constructor(message: string, statusCode: number, context?: Record<string, unknown>) {
    super(message, 'API_ERROR', statusCode, context);
  }
}

export class ValidationError extends CustomError {
  constructor(message: string, context?: Record<string, unknown>) {
    super(message, 'VALIDATION_ERROR', 400, context);
  }
}
```

**Key Points**:
- Always set `this.name` using `this.constructor.name`
- Include `Error.captureStackTrace()` for better debugging
- Add context properties relevant to the error type
- Use consistent error codes across the application

### Environment Configuration

**Best Practices**:
- Store sensitive data in environment variables
- Use `.env` for local development (gitignored)
- Validate required variables at startup
- Document all required variables

**Example**:
```typescript
// config/env.ts
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  API_KEY: z.string().min(1),
  DATABASE_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

### Naming Conventions

**Files**:
- PascalCase for types/interfaces: `UserResponse.ts`
- camelCase for implementation: `fetchUser.ts`
- Test files mirror implementation: `fetchUser.test.ts`

**Functions**:
- Use verb prefixes: `fetchData`, `saveRecord`, `validateInput`
- Async functions return `Promise<T>`
- Export as default for single-purpose modules

**Variables**:
- camelCase for local variables: `userId`, `apiKey`
- UPPER_SNAKE_CASE for constants: `MAX_RETRIES`, `API_TIMEOUT_MS`
- Descriptive names over brevity: `userResponse` not `res`

## Summary

When generating code, always:
- ✅ Write tests first (TDD approach)
- ✅ Use strict TypeScript with explicit types
- ✅ Handle all errors appropriately
- ✅ Follow async/await patterns
- ✅ Use structured logging
- ✅ Validate environment configuration
- ✅ Document public APIs with JSDoc
- ✅ Keep functions small and focused
- ✅ Use meaningful variable and function names
- ✅ Implement proper error handling with custom error classes
- ✅ Think about security implications
- ✅ Consider performance and scalability
- ✅ Make code reviewable and maintainable
- ✅ Use interfaces for data contracts, types for utilities
- ✅ Mock all external dependencies in tests
- ✅ Maintain high test coverage (minimum 80%, ideally 100%)
- ✅ Follow consistent naming conventions
- ✅ Organize code by feature/domain

**Remember**: Code is read more often than it's written. Optimize for readability and maintainability.
