# Testing Instructions for TypeScript Node.js Applications

> General testing best practices for TypeScript Node.js development. These instructions are project-agnostic and apply to any new application or service.

## Overview

This document provides comprehensive testing guidelines for TypeScript Node.js applications. Aim for high test coverage (minimum 80%, ideally 100%) and follow **Test-Driven Development (TDD)** principles.

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

### Tools and Libraries

#### Recommended Setup

- **Test Runner**: [Vitest](https://vitest.dev/) (recommended) or Jest
- **Coverage Provider**: `@vitest/coverage-v8` or Jest coverage
- **Assertion Library**: Built-in with test runner (Chai-compatible for Vitest)
- **Mocking**: Test runner utilities
  - `vi.mock()` / `jest.mock()` - Module mocking
  - `vi.fn()` / `jest.fn()` - Function mocking
  - `vi.spyOn()` / `jest.spyOn()` - Spy on methods
  - `vi.useFakeTimers()` / `jest.useFakeTimers()` - Time manipulation
- **Node Version**: Latest LTS or stable version
- **Package Manager**: pnpm, npm, or yarn

#### Configuration

**vitest.config.ts**:
```typescript
import { defineConfig } from 'vitest/config';
import * as path from 'node:path';

export default defineConfig({
  test: {
    globals: true,            // Enable global test APIs
    environment: 'node',      // Node.js environment (not browser)
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@services': path.resolve(__dirname, './src/services'),
      '@utils': path.resolve(__dirname, './src/utils'),
      // Add more path aliases as needed
    },
  },
});
```

**package.json scripts**:
```json
{
  "scripts": {
    "test": "vitest run",              // Single run (CI)
    "test:watch": "vitest watch",      // Watch mode (development)
    "test:coverage": "vitest --coverage" // Coverage report
  }
}
```

## Test File Organization

### Naming Convention

- **Test files**: `{feature}.test.ts` or `{feature}.spec.ts`
- **Location**: Collocated with implementation files
- **Example**:
  ```
  src/
  ├── services/
  │   └── user/
  │       ├── userService.ts        # Implementation
  │       ├── userService.test.ts   # Tests
  │       ├── userRepository.ts
  │       └── userRepository.test.ts
  ```

### Test Structure Template

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { functionToTest } from './module';

describe('ModuleName', () => {
  describe('functionToTest', () => {
    beforeEach(() => {
      // Setup: runs before each test
      vi.clearAllMocks();
    });

    afterEach(() => {
      // Teardown: runs after each test
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
- **Critical Paths**: 100% coverage REQUIRED
  - Authentication/authorization
  - Data persistence
  - External API integrations
  - Error handling

### Coverage Types

1. **Statement Coverage**: Every line executed
2. **Branch Coverage**: Every if/else path tested
3. **Function Coverage**: Every function called
4. **Line Coverage**: Every line executed at least once

### Running Coverage Reports

```bash
# Generate coverage report
npm run test:coverage
# or with pnpm
pnpm test:coverage
# or with yarn
yarn test:coverage

# View HTML report
open coverage/index.html  # macOS
xdg-open coverage/index.html  # Linux
start coverage/index.html  # Windows
```

### Coverage Output Example

```
 Test Files  3 passed (3)
      Tests  45 passed (45)
   Start at  10:30:00
   Duration  1.23s (transform 234ms, setup 0ms, collect 456ms, tests 543ms, environment 0ms, prepare 123ms)

 % Coverage report from v8
-----------------|---------|----------|---------|---------|-------------------
File             | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-----------------|---------|----------|---------|---------|-------------------
All files        |     100 |      100 |     100 |     100 |
 userService.ts  |     100 |      100 |     100 |     100 |
 apiClient.ts    |     100 |      100 |     100 |     100 |
 validator.ts    |     100 |      100 |     100 |     100 |
-----------------|---------|----------|---------|---------|-------------------
```

## Mocking Best Practices

### Node.js Built-in Modules

Mock Node.js modules at the top level:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import * as fs from 'node:fs/promises';
import { saveData } from './saveData';

// Mock the entire module
vi.mock('node:fs/promises');

describe('saveData', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should write file successfully', async () => {
    // Arrange
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));

    // Act
    const result = await saveData({ id: 1, name: 'test' });

    // Assert
    expect(fs.writeFile).toHaveBeenCalledWith(
      expect.stringContaining('.json'),
      expect.any(String),
      'utf-8'
    );
  });
});
```

### Global Fetch API

Mock the global `fetch` function:

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { fetchUserData } from './fetchUserData';

describe('fetchUserData', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    process.env.API_KEY = 'test-api-key';
    process.env.API_BASEURL = 'https://api.test.com';
  });

  it('should fetch data successfully', async () => {
    // Arrange
    const mockResponse = {
      totalItems: 100,
      items: [{ id: 1, name: 'Test User' }],
    };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockResponse,
    });

    // Act
    const result = await fetchUserData();

    // Assert
    expect(result).toEqual(mockResponse);
    expect(global.fetch).toHaveBeenCalledWith(
      expect.stringContaining('api.test.com'),
      expect.objectContaining({
        headers: {
          'Authorization': 'Bearer test-api-key',
        },
      })
    );
  });

  it('should handle API errors', async () => {
    // Arrange
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 500,
    });

    // Act & Assert
    await expect(fetchUserData()).rejects.toThrow('Failed to fetch data: 500');
  });
});
```

### Environment Variables

Save and restore environment variables:

```typescript
describe('fetchUserData', () => {
  const originalEnv = {
    API_KEY: process.env.API_KEY,
    API_BASEURL: process.env.API_BASEURL,
  };

  beforeEach(() => {
    vi.clearAllMocks();
    process.env.API_KEY = 'test-api-key';
    process.env.API_BASEURL = 'https://api.test.com';
  });

  afterEach(() => {
    process.env.API_KEY = originalEnv.API_KEY;
    process.env.API_BASEURL = originalEnv.API_BASEURL;
  });

  it('should throw error when API key is missing', async () => {
    // Arrange
    delete process.env.API_KEY;

    // Act & Assert
    await expect(fetchUserData()).rejects.toThrow(
      'API_KEY environment variable is not set'
    );
  });
});
```

### Timers and Time-Based Logic

Mock timers for scheduler and time-dependent tests:

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { startScheduler } from './scheduler';

describe('Scheduler', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    vi.useRealTimers();
    vi.restoreAllMocks();
  });

  it('should execute task at specified interval', async () => {
    // Arrange
    const mockTask = vi.fn().mockResolvedValue(undefined);
    const interval = 10 * 60 * 1000; // 10 minutes

    // Act
    startScheduler(mockTask, interval);

    // Wait for immediate execution
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    // Advance time and verify interval execution
    await vi.advanceTimersByTimeAsync(interval);

    // Assert
    expect(mockTask).toHaveBeenCalledTimes(2);
  });
});
```

### Mock Timestamps for Filename Generation

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { writeProjects } from './writeProjects';

describe('writeProjects', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2025-11-25T10:30:45.123Z'));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should create filename with ISO timestamp', async () => {
    // Arrange
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);

    // Act
    const result = await writeProjects({ totalHits: 0, items: [] });

    // Assert
    expect(result).toContain('2025-11-25T10-30-45-123Z.json');
  });
});
```

### Reusable Mock Factories

Create factories for complex mock objects:

```typescript
// Helper to create mock User
function createMockUser(overrides: Partial<User> = {}): User {
  return {
    id: 'test-id',
    email: 'test@example.com',
    name: 'Test User',
    status: 'active',
    // ... other default properties
    ...overrides,
  };
}

describe('saveUsers', () => {
  it('should handle large datasets', async () => {
    const largeData: UsersResponse = {
      totalItems: 1000,
      items: new Array(1000)
        .fill(null)
        .map((_, i) => createMockUser({
          id: `user-${i + 1}`,
          name: `User ${i + 1}`,
        })),
    };

    // ... test implementation
  });
});
```

## Test Patterns by Feature Type

### Testing Data Fetching (API Integration)

**Pattern**: Mock `fetch`, test pagination, error handling, and data transformation.

```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { fetchUserData } from './fetchUserData';

describe('fetchProjects', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    process.env.OBJEKTBANKEN_API_KEY = 'test-api-key';
    process.env.OBJEKTBANKEN_API_BASEURL = 'https://api.test.com';
  });

  it('should fetch all projects with pagination', async () => {
    // Arrange
    const mockPage1 = {
      totalHits: 150,
      items: new Array(100).fill(null).map((_, i) => ({ id: i + 1 })),
    };
    const mockPage2 = {
      totalHits: 150,
      items: new Array(50).fill(null).map((_, i) => ({ id: i + 101 })),
    };

    global.fetch = vi.fn()
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockPage1,
      })
      .mockResolvedValueOnce({
        ok: true,
        json: async () => mockPage2,
      });

    // Act
    const result = await fetchProjects();

    // Assert
    expect(result.totalHits).toBe(150);
    expect(result.items).toHaveLength(150);
    expect(global.fetch).toHaveBeenCalledTimes(2);
    expect(global.fetch).toHaveBeenNthCalledWith(
      1,
      'https://api.test.com/projects?skip=0&take=100',
      expect.objectContaining({
        headers: { 'Api-Key': 'test-api-key' },
      })
    );
  });

  it('should handle empty response', async () => {
    // Arrange
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ totalHits: 0, items: [] }),
    });

    // Act
    const result = await fetchProjects();

    // Assert
    expect(result.totalHits).toBe(0);
    expect(result.items).toHaveLength(0);
  });

  it('should throw FetchError with context on API failure', async () => {
    // Arrange
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 404,
    });

    // Act & Assert
    try {
      await fetchProjects();
      expect.fail('Should have thrown an error');
    } catch (error) {
      expect(error).toBeInstanceOf(Error);
      if (error instanceof Error) {
        expect((error as any).name).toBe('FetchError');
        expect((error as any).statusCode).toBe(404);
        expect((error as any).code).toBe('API_REQUEST_FAILED');
        expect((error as any).context).toBeDefined();
      }
    }
  });

  it('should handle network errors', async () => {
    // Arrange
    global.fetch = vi.fn().mockRejectedValue(new Error('Network error'));

    // Act & Assert
    await expect(fetchProjects()).rejects.toThrow('Network error');
  });

  it('should handle JSON parsing errors', async () => {
    // Arrange
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => {
        throw new Error('Invalid JSON');
      },
    });

    // Act & Assert
    await expect(fetchProjects()).rejects.toThrow('Invalid JSON');
  });
});
```

### Testing Data Persistence (File/Database Writing)

**Pattern**: Mock file system or database, test file creation, conflict detection, and error handling.

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import * as fs from 'node:fs/promises';
import { writeProjects } from './writeProjects';

vi.mock('node:fs/promises');

describe('writeProjects', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    vi.useFakeTimers();
    vi.setSystemTime(new Date('2025-11-25T10:30:45.123Z'));
  });

  afterEach(() => {
    vi.useRealTimers();
  });

  it('should write data to JSON file', async () => {
    // Arrange
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);

    const mockData = { totalHits: 2, items: [{ id: 1 }, { id: 2 }] };

    // Act
    const result = await writeProjects(mockData);

    // Assert
    expect(result).toContain('2025-11-25T10-30-45-123Z.json');
    expect(fs.writeFile).toHaveBeenCalledWith(
      expect.any(String),
      JSON.stringify(mockData, null, 2),
      'utf-8'
    );
  });

  it('should throw FileExistsError if file exists', async () => {
    // Arrange
    vi.mocked(fs.access).mockResolvedValue(undefined);

    // Act & Assert
    try {
      await writeProjects({ totalHits: 0, items: [] });
      expect.fail('Should have thrown an error');
    } catch (error) {
      expect((error as any).name).toBe('FileExistsError');
      expect((error as any).filePath).toBeDefined();
      expect((error as any).code).toBe('FILE_EXISTS');
    }
  });

  it('should create directory if it does not exist', async () => {
    // Arrange
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));
    vi.mocked(fs.mkdir).mockResolvedValue(undefined);
    vi.mocked(fs.writeFile).mockResolvedValue(undefined);

    // Act
    await writeProjects({ totalHits: 0, items: [] });

    // Assert
    expect(fs.mkdir).toHaveBeenCalledWith(
      expect.stringContaining('data'),
      { recursive: true }
    );
  });

  it('should handle write errors', async () => {
    // Arrange
    vi.mocked(fs.access).mockRejectedValue(new Error('File does not exist'));
    vi.mocked(fs.writeFile).mockRejectedValue(new Error('Write failed'));

    // Act & Assert
    await expect(writeProjects({ totalHits: 0, items: [] }))
      .rejects.toThrow('Write failed');
  });
});
```

### Testing Schedulers and Timers

**Pattern**: Use fake timers, test immediate execution, interval-based execution, and error recovery.

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from 'vitest';
import { startScheduler, stopScheduler } from './scheduler';

describe('Scheduler', () => {
  beforeEach(() => {
    vi.useFakeTimers();
  });

  afterEach(() => {
    stopScheduler();
    vi.restoreAllMocks();
  });

  it('should execute task immediately on start', async () => {
    // Arrange
    const mockTask = vi.fn().mockResolvedValue(undefined);

    // Act
    startScheduler(mockTask, 1000);

    // Assert
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));
  });

  it('should execute task at specified interval', async () => {
    // Arrange
    const mockTask = vi.fn().mockResolvedValue(undefined);
    const interval = 10 * 60 * 1000;

    // Act
    startScheduler(mockTask, interval);
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    await vi.advanceTimersByTimeAsync(interval);
    await vi.advanceTimersByTimeAsync(interval);

    // Assert
    expect(mockTask).toHaveBeenCalledTimes(3);
  });

  it('should continue execution even if task throws error', async () => {
    // Arrange
    const mockTask = vi.fn()
      .mockRejectedValueOnce(new Error('Task failed'))
      .mockResolvedValue(undefined);
    const interval = 1000;

    // Act
    startScheduler(mockTask, interval);
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    await vi.advanceTimersByTimeAsync(interval);

    // Assert
    expect(mockTask).toHaveBeenCalledTimes(2);
  });

  it('should stop executing after stopScheduler is called', async () => {
    // Arrange
    const mockTask = vi.fn().mockResolvedValue(undefined);
    const interval = 1000;

    // Act
    startScheduler(mockTask, interval);
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    stopScheduler();
    await vi.advanceTimersByTimeAsync(interval * 3);

    // Assert
    expect(mockTask).toHaveBeenCalledTimes(1);
  });

  it('should throw error when starting scheduler twice', async () => {
    // Arrange
    const mockTask = vi.fn().mockResolvedValue(undefined);

    // Act
    startScheduler(mockTask, 1000);
    await vi.waitFor(() => expect(mockTask).toHaveBeenCalledTimes(1));

    // Assert
    expect(() => startScheduler(mockTask, 1000))
      .toThrow('Scheduler is already running');
  });
});
```

### Testing Custom Error Classes

**Pattern**: Test error name, message, custom properties, and stack trace.

```typescript
import { describe, it, expect } from 'vitest';
import FetchError from './FetchError';

describe('FetchError', () => {
  it('should create error with message and status code', () => {
    // Arrange & Act
    const error = new FetchError('API request failed', 'API_ERROR', 500, {
      url: 'https://api.test.com',
    });

    // Assert
    expect(error).toBeInstanceOf(Error);
    expect(error.name).toBe('FetchError');
    expect(error.message).toBe('API request failed');
    expect(error.code).toBe('API_ERROR');
    expect(error.statusCode).toBe(500);
    expect(error.context).toEqual({ url: 'https://api.test.com' });
    expect(error.stack).toBeDefined();
  });

  it('should use default status code if not provided', () => {
    // Arrange & Act
    const error = new FetchError('Error', 'ERR_CODE');

    // Assert
    expect(error.statusCode).toBe(500);
  });

  it('should be throwable and catchable', () => {
    // Arrange
    const throwError = () => {
      throw new FetchError('Test error', 'TEST_ERROR', 400);
    };

    // Act & Assert
    expect(throwError).toThrow('Test error');
    expect(throwError).toThrow(FetchError);
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

### Example: Comprehensive Edge Case Testing

```typescript
describe('fetchProjects', () => {
  // 1. Happy path
  it('should fetch projects successfully', async () => {
    // ... test implementation
  });

  // 2. Empty response
  it('should handle empty response with zero items', async () => {
    const mockEmptyResponse = { totalHits: 0, items: [] };
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockEmptyResponse,
    });

    const result = await fetchProjects();

    expect(result.totalHits).toBe(0);
    expect(result.items).toHaveLength(0);
  });

  // 3. Boundary: exact page boundary
  it('should handle response at exact page boundary', async () => {
    const mockExactBoundary = {
      totalHits: 100,
      items: new Array(100).fill(null).map((_, i) => ({ id: i + 1 })),
    };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockExactBoundary,
    });

    const result = await fetchProjects();

    expect(result.totalHits).toBe(100);
    expect(result.items).toHaveLength(100);
    expect(global.fetch).toHaveBeenCalledTimes(1); // No extra page fetch
  });

  // 4. Invalid environment
  it('should throw error when API key is missing', async () => {
    delete process.env.OBJEKTBANKEN_API_KEY;

    await expect(fetchProjects()).rejects.toThrow(
      'OBJEKTBANKEN_API_KEY environment variable is not set'
    );
  });

  // 5. Error scenarios
  it('should handle network errors', async () => {
    global.fetch = vi.fn().mockRejectedValue(new Error('Network error'));

    await expect(fetchProjects()).rejects.toThrow('Network error');
  });

  it('should throw error when API returns non-ok response', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: false,
      status: 500,
    });

    await expect(fetchProjects()).rejects.toThrow('Failed to fetch projects: 500');
  });

  it('should handle JSON parsing errors', async () => {
    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => {
        throw new Error('Invalid JSON');
      },
    });

    await expect(fetchProjects()).rejects.toThrow('Invalid JSON');
  });

  // 6. Edge case: multiple pages with varying sizes
  it('should handle multiple pages with varying sizes', async () => {
    const mockPage1 = {
      totalHits: 250,
      items: new Array(100).fill(null).map((_, i) => ({ id: i + 1 })),
    };
    const mockPage2 = {
      totalHits: 250,
      items: new Array(100).fill(null).map((_, i) => ({ id: i + 101 })),
    };
    const mockPage3 = {
      totalHits: 250,
      items: new Array(50).fill(null).map((_, i) => ({ id: i + 201 })),
    };

    global.fetch = vi.fn()
      .mockResolvedValueOnce({ ok: true, json: async () => mockPage1 })
      .mockResolvedValueOnce({ ok: true, json: async () => mockPage2 })
      .mockResolvedValueOnce({ ok: true, json: async () => mockPage3 });

    const result = await fetchProjects();

    expect(result.totalHits).toBe(250);
    expect(result.items).toHaveLength(250);
    expect(global.fetch).toHaveBeenCalledTimes(3);
  });
});
```

## TDD Workflow Example

### Scenario: Adding a New Stream (Objektbanken Objects)

#### Step 1: Write Type Definitions

```typescript
// src/streams/objektbanken/objects/types/ObjectsResponse.ts
export default interface ObjectsResponse {
  totalHits: number;
  items: ObjectItem[];
}

export interface ObjectItem {
  id: string;
  name: string;
  // ... other fields
}
```

#### Step 2: Write Failing Tests for Fetch Function

```typescript
// src/streams/objektbanken/objects/fetchObjects.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { fetchObjects } from './fetchObjects';

describe('fetchObjects', () => {
  beforeEach(() => {
    vi.clearAllMocks();
    process.env.OBJEKTBANKEN_API_KEY = 'test-api-key';
    process.env.OBJEKTBANKEN_API_BASEURL = 'https://api.test.com';
  });

  it('should fetch objects from API', async () => {
    // Arrange
    const mockResponse = {
      totalHits: 50,
      items: [{ id: '1', name: 'Object 1' }],
    };

    global.fetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => mockResponse,
    });

    // Act
    const result = await fetchObjects();

    // Assert
    expect(result.totalHits).toBe(50);
    expect(result.items).toHaveLength(1);
  });

  it('should throw error when API key is missing', async () => {
    delete process.env.OBJEKTBANKEN_API_KEY;

    await expect(fetchObjects()).rejects.toThrow(
      'OBJEKTBANKEN_API_KEY environment variable is not set'
    );
  });
});
```

#### Step 3: Run Tests (Should Fail)

```bash
pnpm --filter fish test

# Expected output:
# FAIL  src/streams/objektbanken/objects/fetchObjects.test.ts
#   ● fetchObjects › should fetch objects from API
#     Cannot find module './fetchObjects'
```

#### Step 4: Implement Minimal Code

```typescript
// src/streams/objektbanken/objects/fetchObjects.ts
import FetchError from '@errors/FetchError';
import ObjectsResponse from './types/ObjectsResponse';

export default async function fetchObjects(): Promise<ObjectsResponse> {
  const apiKey = process.env.OBJEKTBANKEN_API_KEY;
  const baseUrl = process.env.OBJEKTBANKEN_API_BASEURL;

  if (!apiKey) {
    throw new FetchError(
      'OBJEKTBANKEN_API_KEY environment variable is not set',
      'ENV_VAR_MISSING',
      500,
      { variable: 'OBJEKTBANKEN_API_KEY' }
    );
  }

  if (!baseUrl) {
    throw new FetchError(
      'OBJEKTBANKEN_API_BASEURL environment variable is not set',
      'ENV_VAR_MISSING',
      500,
      { variable: 'OBJEKTBANKEN_API_BASEURL' }
    );
  }

  const url = `${baseUrl}/objects?skip=0&take=100`;
  const response = await fetch(url, {
    headers: {
      'Api-Key': apiKey,
    },
  });

  if (!response.ok) {
    throw new FetchError(
      `Failed to fetch objects: ${response.status}`,
      'API_REQUEST_FAILED',
      response.status,
      { httpStatus: response.status }
    );
  }

  return await response.json();
}
```

#### Step 5: Run Tests (Should Pass)

```bash
pnpm --filter fish test

# Expected output:
# PASS  src/streams/objektbanken/objects/fetchObjects.test.ts
#   ✓ should fetch objects from API (25ms)
#   ✓ should throw error when API key is missing (5ms)
```

#### Step 6: Add More Tests and Refactor

Continue adding test cases (pagination, error handling, edge cases) and refactor the implementation while keeping tests green.

## Integration Testing

### When to Use Integration Tests

- Testing multiple components working together
- Database operations (with test database)
- External API integration (with real or staging endpoints)
- File system operations across directories

### Integration Test Setup

```typescript
// {feature}.integration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { setupTestDatabase, teardownTestDatabase } from './testUtils';

describe('ProjectsPipeline Integration', () => {
  beforeAll(async () => {
    await setupTestDatabase();
  });

  afterAll(async () => {
    await teardownTestDatabase();
  });

  it('should fetch and store projects in database', async () => {
    // Arrange
    const projects = await fetchProjects();

    // Act
    await storeProjectsInDatabase(projects);

    // Assert
    const storedProjects = await queryDatabase('SELECT * FROM projects');
    expect(storedProjects).toHaveLength(projects.items.length);
  });
});
```

### Test Database Best Practices

1. **Use in-memory database** for fast tests (SQLite `:memory:`)
2. **Reset database** between tests
3. **Seed test data** consistently
4. **Use transactions** for isolation
5. **Clean up** after all tests complete

## Common Testing Pitfalls

### ❌ DON'T: Test Implementation Details

```typescript
// Bad: Testing internal implementation
it('should call helper function', () => {
  const spy = vi.spyOn(module, 'internalHelper');
  doSomething();
  expect(spy).toHaveBeenCalled(); // Fragile, couples test to implementation
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
// Bad: Making real API calls in tests
it('should fetch real data', async () => {
  const data = await fetch('https://real-api.com/data'); // Slow, flaky, expensive
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

### ❌ DON'T: Write Flaky Tests

```typescript
// Bad: Test depends on timing
it('should complete within 100ms', async () => {
  const start = Date.now();
  await doAsyncWork();
  const duration = Date.now() - start;
  expect(duration).toBeLessThan(100); // Flaky: depends on machine speed
});
```

### ✅ DO: Use Deterministic Assertions

```typescript
// Good: Test deterministic behavior
it('should complete successfully', async () => {
  const result = await doAsyncWork();
  expect(result.status).toBe('completed');
  expect(result.errors).toHaveLength(0);
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

## Performance Testing

### Benchmarking with Vitest

```typescript
import { describe, bench } from 'vitest';
import { processLargeDataset } from './dataProcessor';

describe('Performance', () => {
  bench('should process 1000 items', () => {
    const data = generateMockData(1000);
    processLargeDataset(data);
  });

  bench('should process 10000 items', () => {
    const data = generateMockData(10000);
    processLargeDataset(data);
  });
});
```

### Running Benchmarks

```bash
pnpm --filter fish test --run --benchmark
```

## CI/CD Integration

### GitHub Actions Example

```yaml
name: Test and Coverage

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '25'

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 10

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Run tests with coverage
        run: pnpm --filter fish test:coverage

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          files: ./apps/fish/coverage/coverage-final.json
          flags: fish

      - name: Check coverage threshold
        run: |
          # Fail if coverage is below 100%
          pnpm --filter fish test:coverage --coverage.thresholds.lines=100
```

### Pre-commit Hook (Optional)

```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Running tests before commit..."

pnpm --filter fish test

if [ $? -ne 0 ]; then
  echo "Tests failed! Commit aborted."
  exit 1
fi

echo "Tests passed! Proceeding with commit."
```

## Checklist: Testing New Features

Before marking a feature as complete:

- [ ] All tests written following TDD approach
- [ ] Tests cover happy path
- [ ] Tests cover error scenarios
- [ ] Tests cover edge cases and boundary values
- [ ] All mocks properly configured
- [ ] Environment variables saved/restored in tests
- [ ] Timers mocked if time-dependent logic exists
- [ ] Custom error classes tested for name, code, statusCode, context
- [ ] 100% coverage achieved (check with `test:coverage`)
- [ ] Tests run successfully in CI/CD pipeline
- [ ] No flaky or timing-dependent tests
- [ ] Test names clearly describe expected behavior
- [ ] Test files collocated with implementation files
- [ ] All async operations properly awaited
- [ ] All resources cleaned up in `afterEach` or `afterAll`

## Quick Reference

### Common Vitest APIs

```typescript
// Test structure
describe('Group', () => {
  it('should do something', () => {});
  test('alternative to it', () => {});
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
expect(value).toBeNull();
expect(value).toBeTruthy();
expect(value).toHaveLength(5);
expect(array).toContain(item);
expect(string).toContain('substring');
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
vi.useFakeTimers();               // Use fake timers
vi.useRealTimers();               // Restore real timers
vi.setSystemTime(new Date());     // Set current time
vi.advanceTimersByTime(1000);     // Advance timers by ms
vi.advanceTimersByTimeAsync(1000); // Advance async timers
vi.waitFor(() => expect(...));    // Wait for condition
```

### Running Tests

```bash
# Run all tests once
pnpm --filter fish test

# Run in watch mode (re-runs on file changes)
pnpm --filter fish test:watch

# Run with coverage report
pnpm --filter fish test:coverage

# Run specific test file
pnpm --filter fish test fetchProjects.test.ts

# Run tests matching pattern
pnpm --filter fish test --grep "should fetch"

# Run in UI mode (experimental)
pnpm --filter fish test --ui
```

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
