# Testing Instructions

> **Scope:** These instructions are intended only for use with **Claude Code**. They define how Claude Code should write
> and execute tests when assisting with any project. They are not designed or supported for use with other AI agents or
> models.

## Core Testing Philosophy

### Test-Driven Development (TDD)

> This is the test-specific TDD cycle. For the broader code change workflow,
> see [coding.md "Iterative Development Workflow"](coding.md#iterative-development-workflow). For high-level task
> execution, see [CLAUDE.md Section 6](CLAUDE.md#6-task-execution-flow).

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

- **Test Runner**: Use the standard test framework for your language/platform
- **Coverage Provider**: Use built-in or standard coverage tools
- **Assertion Library**: Use idiomatic assertions for your language
- **Mocking**: Use test framework utilities or standard mocking libraries

### Configuration

Configure your test runner with:

- Test file discovery patterns
- Coverage thresholds
- Environment settings (test mode)
- Path aliases if needed

## Test File Organization

### Naming Convention

- **Test files**: `{feature}.test.{ext}` or `{feature}_test.{ext}` (language-dependent)
- **Location**: Collocated with implementation files

### Naming Mirrors Implementation

Test files should mirror the structure and naming of implementation files:

- Implementation: `fetch_user.{ext}` → Test: `fetch_user.test.{ext}`
- Implementation: `services/auth/authenticate.{ext}` → Test: `services/auth/authenticate.test.{ext}`

**Rationale**:

- Easy to locate corresponding test file
- Consistent naming reduces cognitive load
- IDE tooling can easily pair implementation and tests

### Test Structure Template

```
describe('ModuleName', () => {
  describe('functionToTest', () => {
    beforeEach(() => {
      // Clear mocks, reset state
    });

    afterEach(() => {
      // Restore original implementations
    });

    it('should do expected behavior when given valid input', () => {
      // Arrange - Set up test data and mocks
      // Act - Execute the function under test
      // Assert - Verify expected outcomes
    });

    it('should throw error when given invalid input', () => {
      // Arrange
      // Act & Assert
    });
  });
});
```

## Coverage Requirements

### Target Coverage

- **Minimum (enforced)**: 80% for all code - this is the threshold that must pass CI
- **Goal**: 100% for new code - strive for complete coverage on new implementations
- **Critical Paths**: 100% coverage REQUIRED (non-negotiable) - auth, data persistence, external APIs, error handling

### Running Coverage Reports

Use your test runner's coverage command and review HTML reports when available.

## Mocking Best Practices

### External Dependencies

Mock all external dependencies:

- File system operations
- Network requests (HTTP, database)
- System time
- Environment variables
- Third-party services

### Mocking Patterns

#### Network/HTTP Calls

- Mock the HTTP client or fetch function
- Return controlled responses
- Simulate error conditions

#### File System

- Mock file read/write operations
- Control returned data
- Simulate permission errors

#### Environment Variables

- Save original values in setup
- Set test values
- Restore in teardown

#### Time-Based Logic

- Use fake timers when available
- Control system time for deterministic tests
- Advance time programmatically

### Mock Cleanup

Always clean up mocks:

- Clear mock call history between tests
- Restore original implementations after tests
- Reset any modified global state

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

```
// Bad: Testing internal implementation
it('should call helper function', () => {
  // Fragile - breaks when implementation changes
});
```

### ✅ DO: Test Behavior and Outcomes

```
// Good: Testing observable behavior
it('should return formatted data', () => {
  const result = doSomething({input: 'raw'});
  expect(result).toEqual({output: 'formatted'});
});
```

### ❌ DON'T: Use Real External Dependencies

```
// Bad: Making real API calls - slow, flaky
it('should fetch real data', async () => {
  const data = await fetch('https://real-api.com/data');
});
```

### ✅ DO: Mock External Dependencies

```
// Good: Mocking external dependencies - fast, deterministic
it('should fetch data', async () => {
  mockFetch({data: 'mock'});
  const data = await fetchData();
  expect(data).toEqual({data: 'mock'});
});
```

### ❌ DON'T: Share State Between Tests

```
// Bad: Shared mutable state - order-dependent
let counter = 0;
it('first test', () => { counter++; });
it('second test', () => { expect(counter).toBe(1); }); // Fragile
```

### ✅ DO: Isolate Test State

```
// Good: Each test is independent
it('first test', () => {
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

- Set up test environment in `beforeAll`
- Clean up in `afterAll`
- Use test databases or sandboxes
- Ensure tests are idempotent

## TDD Workflow in Practice

### Step 1: Write Tests First

- Write failing test(s) for new behavior
- Run tests to confirm they fail with expected message
- Define the API/interface through test usage
- Commit: `test: add tests for [feature]`

### Step 2: Implement Minimal Solution

- Write just enough code to pass tests
- Keep it simple and readable
- Follow YAGNI (You Aren't Gonna Need It)

### Step 3: Run All Tests

- Ensure all tests pass (not just the new ones)
- Check code coverage
- Fix any failing tests immediately
- Verify coverage meets requirements

### Step 4: Refactor

- Improve code structure while tests remain green
- Extract duplicated code
- Improve naming and clarity
- Run tests after each refactor
- Commit: `refactor: improve [component]`

## Code Review: Testing Guidelines

### What Reviewers Should Check

- [ ] **Tests included**: All new functionality has corresponding tests
- [ ] **Coverage maintained**: Coverage meets minimum 80% (100% for critical paths)
- [ ] **Edge cases tested**: Null, undefined, empty values, boundaries
- [ ] **Error scenarios tested**: Network failures, invalid input, exceptions
- [ ] **Mocks properly configured**: All external dependencies mocked
- [ ] **Tests are deterministic**: No flaky or timing-dependent tests
- [ ] **Test names descriptive**: Clearly describe expected behavior
- [ ] **Follows TDD patterns**: Test structure matches established conventions
- [ ] **No commented tests**: All tests should be active
- [ ] **Resources cleaned up**: Setup/teardown properly configured

## Checklist: Testing New Features

- [ ] All tests written following TDD approach
- [ ] Tests cover happy path, error scenarios, and edge cases
- [ ] All mocks properly configured
- [ ] Environment variables saved/restored in tests
- [ ] Time mocked if time-dependent logic exists
- [ ] Coverage meets minimum 80% (100% for critical paths)
- [ ] Tests run successfully
- [ ] No flaky or timing-dependent tests
- [ ] Test names clearly describe expected behavior
- [ ] All async operations properly handled
- [ ] All resources cleaned up in teardown

## Summary

**Key Principles**:

1. **Write tests first** (TDD approach)
2. **Maintain high coverage** for all code
3. **Mock all external dependencies**
4. **Test behavior, not implementation**
5. **Write deterministic tests** (no flakiness)
6. **Isolate test state** (no shared mutations)
7. **Test all paths**: happy path, errors, edge cases
8. **Use descriptive test names**: "should [expected] when [condition]"
9. **Clean up resources** in teardown
10. **Follow established patterns** from existing tests

**Remember**: Tests are documentation. Write tests that clearly communicate what the code does and why it matters.
