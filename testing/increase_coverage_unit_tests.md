# Unit Test Coverage Improvement Workflow

## Overview
Systematic approach to improve unit test coverage with quality gates and efficiency patterns. Target: 80%+ coverage with meaningful tests.

---

## Phase 1: Coverage Analysis & Task Creation
**Objective**: Generate comprehensive coverage report and create actionable task list  
**Time Estimate**: 5-10 minutes

### Steps:
1. **Execute Coverage Analysis**
   ```bash
   yarn jest --coverage=true --verbose
   ```

2. **Create Task Tracking File**
   - Generate `currentUnitTestCoverageTasks.md`
   - Include files with <80% coverage
   - Add markdown checkboxes `[ ]` for progress tracking
   - Prioritize by:
     - Critical business logic (high priority)
     - User-facing components (medium priority)
     - Utility functions (medium priority)
     - Configuration files (low priority)

3. **Coverage Report Analysis**
   - Identify patterns in untested code
   - Note files with 0% coverage (highest priority)
   - Flag complex functions with low coverage

**Success Criteria**: Complete task list with prioritized coverage gaps

---

## Phase 2: File Classification & Ignore Patterns
**Objective**: Identify files that don't require testing and optimize jest configuration  
**Time Estimate**: 10-15 minutes

### Classification Rules:
- **Skip Testing**: 
  - Styled components (`*.styled.ts`, `*.styles.ts`)
  - Type definitions (`*.types.ts`, `*.d.ts`)
  - Configuration files (`*.config.ts`, `vite.config.ts`)
  - Build/deployment scripts
  - Mock files (`*.mock.ts`, `__mocks__/*`)
  - Generated files
  - Simple barrel exports (index files with only exports)

### Jest Configuration Updates:
```javascript
// Add to jest.config.js coveragePathIgnorePatterns
coveragePathIgnorePatterns: [
  "/node_modules/",
  "\\.styled\\.(ts|tsx)$",
  "\\.styles\\.(ts|tsx)$", 
  "\\.types\\.(ts|tsx)$",
  "\\.config\\.(ts|js)$",
  "\\.mock\\.(ts|tsx)$",
  "/mocks/",
  "index\\.(ts|tsx)$" // Only if simple barrel exports
]
```

### Process:
1. Review each file in coverage report
2. Apply classification rules
3. Update jest ignore patterns using regex
4. Remove ignored files from task list
5. Re-run coverage to validate configuration

**Success Criteria**: Optimized coverage report focused on testable code

---

## Phase 3: Test Pattern Analysis & Standardization
**Objective**: Extract reusable patterns and create testing utilities  
**Time Estimate**: 20-30 minutes  
**Prerequisite**: Existing unit tests in codebase

### Pattern Identification:
1. **Common Setup Patterns**
   - Provider wrappers (React Query, Router, Theme)
   - Mock configurations
   - Test data factories
   - Event simulation helpers

2. **Repeated Logic Detection**
   - Component rendering with providers
   - API response mocking
   - Event handler testing
   - Async operation testing

### Utility Creation:
Create `src/test-utils/` directory with:

```typescript
// renderWithProviders.tsx
import React from 'react';
import { render, RenderOptions } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BrowserRouter } from 'react-router-dom';
import { Theme } from '../components/Theme';
import { ToastContainer } from 'react-toastify';

interface CustomRenderOptions extends Omit<RenderOptions, 'wrapper'> {
  queryClient?: QueryClient;
  initialEntries?: string[];
}

export const renderWithProviders = (
  component: React.ReactElement,
  options: CustomRenderOptions = {}
) => {
  const { queryClient = new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  }), initialEntries = ['/'], ...renderOptions } = options;

  const Wrapper = ({ children }: { children: React.ReactNode }) => (
    <QueryClientProvider client={queryClient}>
      <Theme>
        <ToastContainer containerId="defaultToast" />
        <BrowserRouter>{children}</BrowserRouter>
      </Theme>
    </QueryClientProvider>
  );

  return render(component, { wrapper: Wrapper, ...renderOptions });
};

// mockApiResponses.ts
export const createMockApiResponse = <T>(data: T, status = 200) => ({
  data,
  status,
  statusText: 'OK',
  headers: {},
  config: {},
});

// testDataFactories.ts
export const createMockUser = (overrides = {}) => ({
  id: '1',
  username: 'testuser',
  email: 'test@example.com',
  groups: [],
  ...overrides,
});
```

### Documentation Updates:
Add patterns to appropriate guidance file (`CLAUDE.md`, `./cursor/rules/unit-tests.md`, etc.):
- Test helper usage guidelines
- Common mock patterns
- Testing anti-patterns to avoid
- Component-specific testing strategies

**Success Criteria**: Reusable test utilities and documented patterns

---

## Phase 4: Test Implementation
**Objective**: Implement unit tests following established patterns  
**Time Estimate**: Variable (5-15 min per file)

### Implementation Process:
1. **Pre-Implementation**
   - Review `#unit-tests.md` guidance
   - Check component dependencies and complexity
   - Identify required mocks and test scenarios

2. **Test Development Cycle**
   ```bash
   # For each file in currentUnitTestCoverageTasks.md:
   
   # 1. Write comprehensive tests
   # 2. Run specific test file
   yarn jest ComponentName.test.tsx --verbose
   
   # 3. Check coverage for specific file
   yarn jest ComponentName.test.tsx --coverage
   
   # 4. Fix TypeScript/lint errors
   yarn tsc --noEmit && yarn lint
   
   # 5. Mark as complete [x]
   ```

3. **Test Categories per File**:
   - **Component Rendering**: Basic render without errors
   - **Props Testing**: Required/optional props, prop validation
   - **User Interactions**: Click, type, keyboard events
   - **State Changes**: Internal state management
   - **Error Handling**: Error boundaries, invalid inputs
   - **Accessibility**: ARIA attributes, keyboard navigation
   - **Edge Cases**: Empty states, loading states, error states

### Quality Gates:
- ✅ All tests pass
- ✅ TypeScript compilation succeeds
- ✅ Lint rules pass
- ✅ Coverage improves meaningfully
- ✅ Tests are readable and maintainable

### Batch Processing:
- **First Iteration**: 5 highest-priority files
- **Subsequent Iterations**: 3-5 files per batch
- **Break Between Batches**: Review and refactor common patterns

**Success Criteria**: 80%+ coverage with meaningful, passing tests

### Tests Failing:
- Check mock configurations
- Verify provider setup
- Review async operation handling
- Validate test environment setup

### TypeScript Errors:
- Update type definitions
- Check import paths
- Review jest type configurations
- Validate test utility types

---

## Success Metrics

- **Coverage Improvement**: Reach 80%+ overall coverage
- **Test Quality**: All tests meaningful and maintainable
- **Developer Experience**: Faster test writing with utilities
- **Documentation**: Clear patterns for future development