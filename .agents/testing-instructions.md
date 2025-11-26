# Testing Instructions for TypeScript Node.js Applications

> General testing best practices for TypeScript Node.js development. These instructions are project-agnostic and apply to any new application or service.

## Core Testing Philosophy

### Test-Driven Development (TDD)

**Every feature or bug fix MUST follow this workflow:**

1. **Write the test first** - Define expected behavior before implementation
2. **Watch it fail** - Run tests to confirm they fail as expected
3. **Write minimal code** - Implement just enough to pass the test
4. **Watch it pass** - Run tests to confirm they now pass
5. **Refactor** - Improve code structure while keeping tests green
6. **Commit** - Commit with tests included

### Testing Pyramid

```
                    ╱╲
                   ╱  ╲
                  ╱ E2E ╲          Few (critical user flows)
                 ╱────────╲
                ╱          ╲
               ╱Integration╲       Some (component interaction)
              ╱──────────────╲
             ╱                ╲
            ╱   Unit Tests     ╲   Many (individual functions/classes)
           ╱────────────────────╲
```

**Focus**: Unit tests form the foundation. Integration and E2E tests supplement.

## Testing Stack

### Recommended Setup

- **Test Runner**: [Vitest](https://vitest.dev/) (recommended) or Jest
- **Coverage Provider**: `@vitest/coverage-v8` or Jest coverage
- **Assertion Library**: Built-in with test runner
- **Mocking**: Test runner utilities (`vi.mock()`, `vi.fn()`, `vi.spyOn()`, `vi.useFakeTimers()`)

### Configuration

**vitest.config.ts**:
```typescript
import { defineConfig } from 'vitest/config';
import * as path from 'node:path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

**package.json scripts**:
```json
{
  "scripts": {
    "test": "vitest run",
    "test:watch": "vitest watch",
    "test:coverage": "vitest --coverage"
  }
}
```

## Test File Organization

### Naming Convention

- **Test files**: `{feature}.test.ts` or `{feature}.spec.ts`
- **Location**: Collocated with implementation files

### Test Structure Template

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { functionToTest } from './module';

describe('ModuleName', () => {
  describe('functionToTest', () => {
    beforeEach(() => {
      vi.clearAllMocks();
    });

    afterEach(() => {
      vi.restoreAllMocks();
    });

    it('should do expected behavior when given valid input', async () => {
      // Arrange - Set up test data and mocks
      const input = { value: 'test' };
      const mockDependency = vi.fn().mockResolvedValue({ result: 'success' });

      // Act - Execute the function under test
      const result = await functionToTest(input);

      // Assert - Verify expected outcomes
      expect(result).toBeDefined();
      expect(result.value).toBe('expected');
      expect(mockDependency).toHaveBeenCalledWith(input);
    });

    it('should throw error when given invalid input', async () => {
      // Arrange
      const invalidInput = null;

      // Act & Assert
      await expect(functionToTest(invalidInput)).rejects.toThrow('Invalid input');
    });
  });
});
```

## Coverage Requirements

### Target Coverage

- **Minimum**: 80% for all new code
- **Goal**: 100% for all files
- **Critical Paths**: 100% coverage REQUIRED (auth, data persistence, external APIs, error handling)

### Running Coverage Reports

```bash
npm run test:coverage

# View HTML report
open coverage/index.html  # macOS
xdg-open coverage/index.html  # Linux
```

## Mocking Best Practices

### Node.js Built-in Modules

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import * as fs from 'node:fs/promises';
import { saveData } from './saveData';

vi.mock('node:fs/promises');

describe('saveData', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should write file successfully', async () => {
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);

    const result = await saveData({ id: 1, name: 'test' });

    expect(fs.writeFile).toHaveBeenCalledWith(
      expect.stringContaining('.json'),
      expect.any(String),
      'utf-8'
    );
  });
});
```

### Global Fetch API

```typescript
describe('fetchData', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    process.env.API_KEY = 'test-api-key';
  });

  it('should fetch data successfully', async () => {
    const mockResponse = { data: [{ id: 1, name: 'Test' }] };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockResponse,
    });

    const result = await fetchData();

    expect(result).toEqual(mockResponse);
    expect(global.fetch).toHaveBeenCalledWith(
      expect.stringContaining('api.'),
      expect.objectContaining({
        headers: { 'Authorization': 'Bearer test-api-key' },
      })
    );
  });

  it('should handle API errors', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 500,
    });

    await expect(fetchData()).rejects.toThrow('Failed to fetch data: 500');
  });
});
```

### Environment Variables

```typescript
describe('fetchData', () => {
  const originalEnv = { API_KEY: process.env.API_KEY };

  beforeEach(() => {
    process.env.API_KEY = 'test-api-key';
  });

  afterEach(() => {
    process.env.API_KEY = originalEnv.API_KEY;
  });

  it('should throw error when API key is missing', async () => {
    delete process.env.API_KEY;

    await expect(fetchData()).rejects.toThrow(
      'API_KEY environment variable is not set'
    );
  });
});
```

### Timers and Time-Based Logic

```typescript
describe('Scheduler', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should execute task at specified interval', async () => {
    const mockTask = vi.fn().mockResolvedValue(undefined);
    const interval = 10 * 60 * 1000;

    startScheduler(mockTask, interval);
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    await vi.advanceTimersByTimeAsync(interval);

    expect(mockTask).toHaveBeenCalledTimes(2);
  });
});
```

### Mock Timestamps

```typescript
describe('generateFilename', () => {
  beforeEach(() => {
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2025-11-25T10:30:45.123Z'));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should create filename with ISO timestamp', () => {
    const filename = generateFilename('data');
    expect(filename).toContain('2025-11-25T10-30-45-123Z');
  });
});
```

## Edge Cases and Boundary Testing

### Required Test Cases for Every Function

1. **Happy Path**: Valid input produces expected output
2. **Empty Input**: Empty arrays, null, undefined, empty strings
3. **Boundary Values**: Min/max values, zero, negative numbers
4. **Invalid Input**: Wrong types, malformed data
5. **Error Scenarios**: Network failures, file system errors, exceptions
6. **Edge Cases**: Pagination boundaries, time zones, race conditions

## Common Testing Pitfalls

### ❌ DON'T: Test Implementation Details

```typescript
// Bad: Testing internal implementation
it('should call helper function', () => {
  const spy = vi.spyOn(module, 'internalHelper');
  doSomething();
  expect(spy).toHaveBeenCalled(); // Fragile
});
```

### ✅ DO: Test Behavior and Outcomes

```typescript
// Good: Testing observable behavior
it('should return formatted data', () => {
  const result = doSomething({ input: 'raw' });
  expect(result).toEqual({ output: 'formatted' });
});
```

### ❌ DON'T: Use Real External Dependencies

```typescript
// Bad: Making real API calls
it('should fetch real data', async () => {
  const data = await fetch('https://real-api.com/data'); // Slow, flaky
  expect(data).toBeDefined();
});
```

### ✅ DO: Mock External Dependencies

```typescript
// Good: Mocking external dependencies
it('should fetch data', async () => {
  global.fetch = vi.fn().mockResolvedValue({
    ok: true,
    json: async () => ({ data: 'mock' }),
  });

  const data = await fetchData();
  expect(data).toEqual({ data: 'mock' });
});
```

### ❌ DON'T: Share State Between Tests

```typescript
// Bad: Shared mutable state
let counter = 0;

it('should increment counter', () => {
  counter++;
  expect(counter).toBe(1);
});

it('should be 2', () => {
  counter++;
  expect(counter).toBe(2); // Breaks if tests run in different order
});
```

### ✅ DO: Isolate Test State

```typescript
// Good: Each test is independent
it('should start with zero', () => {
  let counter = 0;
  counter++;
  expect(counter).toBe(1);
});

it('should also start with zero', () => {
  let counter = 0;
  counter++;
  expect(counter).toBe(1);
});
```

## Integration Testing

### When to Use

- Testing multiple components working together
- Database operations (with test database)
- External API integration (with real or staging endpoints)

### Integration Test Setup

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest';

describe('Pipeline Integration', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  it('should fetch and store data', async () => {
    const data = await fetchData();
    await storeInDatabase(data);

    const stored = await queryDatabase('SELECT * FROM items');
    expect(stored).toHaveLength(data.length);
  });
});
```

## Quick Reference

### Common Vitest APIs

```typescript
// Test structure
describe('Group', () => {
  it('should do something', () => {});
});

// Lifecycle hooks
beforeEach(() => {});  // Runs before each test
afterEach(() => {});   // Runs after each test
beforeAll(() => {});   // Runs once before all tests
afterAll(() => {});    // Runs once after all tests

// Assertions
expect(value).toBe(expected);
expect(value).toEqual(expected);
expect(value).toBeDefined();
expect(value).toHaveLength(5);
expect(fn).toThrow('error message');
await expect(promise).rejects.toThrow('error');

// Mocking
vi.fn();                          // Create mock function
vi.mock('module-name');           // Mock entire module
vi.mocked(fn);                    // Get typed mock
vi.spyOn(object, 'method');       // Spy on method
vi.clearAllMocks();               // Clear mock history
vi.restoreAllMocks();             // Restore original implementations

// Timers
vi.useFakeTimers();                    // Use fake timers
vi.useRealTimers();                    // Restore real timers
vi.setSystemTime(new Date());          // Set current time
vi.advanceTimersByTimeAsync(1000);     // Advance async timers by milliseconds
vi.waitFor(() => expect(...));         // Wait for condition
```

### Running Tests

```bash
# Run all tests once
npm test

# Run in watch mode
npm run test:watch

# Run with coverage
npm run test:coverage

# Run specific test file
npm test path/to/test.test.ts
```

## Checklist: Testing New Features

- [ ] All tests written following TDD approach
- [ ] Tests cover happy path, error scenarios, and edge cases
- [ ] All mocks properly configured
- [ ] Environment variables saved/restored in tests
- [ ] Timers mocked if time-dependent logic exists
- [ ] 100% coverage achieved (check with `test:coverage`)
- [ ] Tests run successfully
- [ ] No flaky or timing-dependent tests
- [ ] Test names clearly describe expected behavior
- [ ] All async operations properly awaited
- [ ] All resources cleaned up in `afterEach` or `afterAll`

## Summary

**Key Principles**:

1. **Write tests first** (TDD approach)
2. **Maintain 100% coverage** for all code
3. **Mock all external dependencies** (fs, fetch, timers)
4. **Test behavior, not implementation**
5. **Write deterministic tests** (no flakiness)
6. **Isolate test state** (no shared mutations)
7. **Test all paths**: happy path, errors, edge cases
8. **Use descriptive test names**: "should [expected] when [condition]"
9. **Clean up resources** in afterEach/afterAll
10. **Follow established patterns** from existing tests

**Remember**: Tests are documentation. Write tests that clearly communicate what the code does and why it matters.
