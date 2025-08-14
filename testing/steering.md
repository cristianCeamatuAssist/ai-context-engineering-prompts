---
description: Comprehensive unit testing guidelines and patterns for consistent, high-quality test implementation
globs: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts", "**/*.spec.tsx"]
alwaysApply: true
---

# Unit Testing Standards & Best Practices

## Core Testing Commands

-   Use `yarn jest <filename> --verbose` to run specific tests with detailed output
-   Use `yarn jest --coverage <filename>` to check coverage for specific files
-   Use `yarn jest --detectOpenHandles` to debug hanging tests

## Essential Testing Patterns

### Window Object Mocking
When testing code that interacts with the window object:
```javascript
let windowSpy;

beforeEach(() => {
windowSpy = jest.spyOn(window, "window", "get");
});

afterEach(() => {
windowSpy.mockRestore();
});

it('should return https://example.com', () => {
windowSpy.mockImplementation(() => ({
location: {
origin: "https://example.com"
}
}));

expect(window.location.origin).toEqual("https://example.com");
});

it('should be undefined.', () => {
windowSpy.mockImplementation(() => undefined);

expect(window).toBeUndefined();
});

### Module Mocking Best Practices

**CRITICAL**: Use `jest.mock()` instead of `jest.spyOn()` for mocking implementations

❌ **BAD - Don't use jest.spyOn() for implementations:**
```javascript
const updateAllServiceWorkersMock = jest
  .spyOn(require('./service-workers.utils'), 'updateAllServiceWorkers')
  .mockResolvedValue(undefined);
```

✅ **GOOD - Use jest.mock() for clean mocks:**
```javascript
jest.mock('src/features/auth/hooks/useAuth', () => ({
  useAuth() {
    return {
      currentUser: { username: 'testuser', groups: [] },
      isCheckingAuth: false,
      signOut: jest.fn(),
      signIn: jest.fn(),
      signUp: jest.fn(),
      confirmSignUp: jest.fn(),
      forgotPassword: jest.fn(),
      forgotPasswordSubmit: jest.fn(),
      phoneSignIn: jest.fn(),
      socialSignIn: jest.fn(),
      verifySMSCode: jest.fn(),
      currentChallenge: null
    };
  }
}));
```

-   After you write unit/snapshots tests, always run the test command to check the written tests are passing

### Named Export Mocking Patterns

❌ **BAD - Incorrect named export mocking:**
```javascript
// This creates an unnamed default export, not a named export
jest.mock('src/features/account/hooks/useUser', () => () => ({
  isSuperKeyUser: true,
  userData: { id: '1' }
}));
```

✅ **GOOD - Correct named export mocking:**
```javascript
// Mock named exports with explicit naming
jest.mock('src/features/account/hooks/useUser', () => ({
  useUser: () => ({
    isSuperKeyUser: true,
    userData: { id: '1' }
  })
}));

jest.mock('src/features/auth/hooks/useAuth', () => ({
  useAuth: () => ({
    refreshSession: jest.fn(() => Promise.resolve()),
    decodedIdToken: {}
  })
}));
```

### Development Workflow Rules

-   **NEVER run `yarn lint --fix`** or project-wide formatting commands - creates excessive PR changes
-   **DO run** `yarn lint <specific-file>` to check individual files
-   **DO fix** only the specific lint errors in files you're testing
-   **DO run** `yarn jest --passWithNoTests` to verify test setup

## Common Testing Patterns and Solutions

### React Import Issues

-   **Problem**: `ReferenceError: React is not defined` when using JSX in tests
-   **Solution**: Add `import React from 'react';` at the top of test files, even if not explicitly used
-   **Root Cause**: The project uses React 18 but may not have automatic JSX transform configured for tests
-   **Global Fix**: Added `global.React = React;` in jest-setup.ts to make React available globally

### Theme and Styled Components Testing

-   **Problem**: `TypeError: Cannot read properties of undefined (reading 'fontFamily')` or similar theme-related errors
-   **Solution**: Always mock theme providers with complete theme objects including all required properties
-   **Common Pattern**:

```javascript
jest.mock('../Themes', () => ({
    useTheme: () => ({
        theme: {
            colors: {
                componentTokens: {
                    input: { background: '#ffffff', border: '#cccccc' },
                    text: { fontFamily: 'Arial, sans-serif' }
                }
            }
        }
    }),
    BaseTheme: {
        colors: {
            componentTokens: {
                input: { background: '#ffffff', border: '#cccccc' },
                text: { fontFamily: 'Arial, sans-serif' }
            }
        }
    }
}));
```

### Event Testing Patterns

-   **Keyboard Events**: Use `new KeyboardEvent('keydown', { key: 'Enter' })` and `document.dispatchEvent(event)`
-   **DOM Events**: Use `fireEvent.click()`, `fireEvent.change()`, etc. from @testing-library/react
-   **Event Handlers**: Always test with `expect(mockHandler).toHaveBeenCalledTimes(1)`
-   **Event Cleanup**: Test that event listeners are properly removed on unmount

### Hook Testing Best Practices

-   **Use renderHook**: Import from `@testing-library/react` for testing custom hooks
-   **Cleanup Pattern**:

```javascript
afterEach(() => {
    jest.clearAllMocks();
    // Clean up any spies
    addEventListenerSpy?.mockRestore();
    removeEventListenerSpy?.mockRestore();
});
```

-   **Test Lifecycle**: Always test hook setup, usage, and cleanup

### Component Props and Type Testing

-   **Required Props**: Test required props first, then optional props
-   **Type Literals**: Use `as const` for type literals: `type: 'number' as const`
-   **Prop Validation**: Test that components handle different prop combinations correctly
-   **Edge Cases**: Test with empty arrays, undefined values, null values

### Async Testing Guidelines

-   **waitFor**: Use for async operations that need to complete
-   **act**: Use when testing state updates that happen asynchronously
-   **Timers**: Use `jest.useFakeTimers()` in beforeEach and `jest.useRealTimers()` in afterEach
-   **Promises**: Always await async operations in tests

### Common Mock Patterns

-   **nanoid**: `jest.mock('nanoid', () => ({ nanoid: () => '1234' }));`
-   **ResizeObserver**:

```javascript
global.ResizeObserver = jest.fn().mockImplementation(() => ({
    observe: jest.fn(),
    unobserve: jest.fn(),
    disconnect: jest.fn()
}));
```

-   **localStorage**: Mock in beforeEach and restore in afterEach
-   **External Libraries**: Mock at the top of test files, not inside test blocks

### Coverage and Quality Guidelines

-   **Target**: Aim for 80%+ line coverage but prioritize meaningful tests over coverage percentage
-   **Focus Areas**: Test error boundaries, edge cases, user interactions, and critical business logic
-   **Avoid**: Don't test implementation details, test behavior and outcomes
-   **Integration**: Mock external dependencies but test integration points

### File Organization and Naming

-   **Location**: Keep test files next to the component they test (ComponentName.test.tsx)
-   **Naming**: Use descriptive test names that explain the behavior: `'should call onChange when input value changes'`
-   **Structure**: Group related tests with `describe` blocks
-   **Setup**: Use `beforeEach` and `afterEach` for consistent setup and cleanup

### Common Pitfalls to Avoid

-   **Don't**: Use `jest.spyOn()` for mocking implementations (use `jest.mock()` instead)
-   **Don't**: Test internal component state or implementation details
-   **Don't**: Forget to clean up spies, timers, and event listeners
-   **Don't**: Mock everything - test real behavior when possible
-   **Do**: Test user interactions and expected outcomes
-   **Do**: Test error states and edge cases
-   **Do**: Use meaningful assertions that verify the expected behavior

### Theme/Styled Components Mocking

-   **Problem**: `TypeError: Cannot read properties of undefined (reading 'fontFamily')` or similar theme-related errors
-   **Root Cause**: Styled components access nested theme properties that aren't included in incomplete theme mocks
-   **Solution**: When mocking theme hooks, ensure you mock the complete theme structure that components expect
-   **Critical**: Always include both `colorTokens` AND `componentTokens` in theme mocks

**Complete Theme Mock Pattern**:

```javascript
jest.mock('../Themes', () => ({
    useTheme: () => ({
        theme: {
            colors: {
                colorTokens: {
                    primary900: '#000000',
                    primary700: '#333333',
                    primary500: '#666666'
                    // Add all color tokens the component uses
                },
                componentTokens: {
                    input: { background: '#ffffff', border: '#cccccc' },
                    text: { fontFamily: 'Arial, sans-serif' }
                    // Add all component tokens the component uses
                }
            }
        }
    }),
    BaseTheme: {
        colors: {
            colorTokens: {
                primary900: '#000000',
                primary700: '#333333',
                primary500: '#666666'
            },
            componentTokens: {
                input: { background: '#ffffff', border: '#cccccc' },
                text: { fontFamily: 'Arial, sans-serif' }
            }
        }
    }
}));
```

**Debugging Theme Issues**:

1. Check the component's styles file to see what theme properties it accesses
2. Look for `theme.colors.colorTokens.X` or `theme.colors.componentTokens.X` patterns
3. Add ALL accessed properties to your theme mock
4. Include both `useTheme` return value AND `BaseTheme` fallback

**Complex Component Theme Issues**:

-   For components with many dependencies and complex theme requirements, consider:
    -   Creating a shared theme mock utility
    -   Using a more comprehensive theme mock that covers all common properties
    -   Testing simpler components first to establish working patterns

### Input Component Type Constraints

-   **Problem**: BaseInput component only accepts `type="number"` or `type="date"`, not `type="text"`
-   **Solution**: Use `type="number"` for BaseInput tests and provide numeric values
-   **Pattern**: Always check component prop types before writing tests

### Hook Testing with Event Listeners

-   **Problem**: Testing custom hooks that add/remove event listeners
-   **Solution**:
    -   Use `renderHook` from `@testing-library/react`
    -   Spy on `document.addEventListener` and `document.removeEventListener`
    -   Use `document.dispatchEvent()` to simulate events
    -   Always test cleanup (unmount behavior)
-   **Example**: See `useKeyBoardKeys.test.ts` for complete pattern

### Avoiding Unrealistic Test Cases

-   **Problem**: Testing edge cases that would never occur in real usage (like undefined functions)
-   **Solution**: Focus on realistic scenarios and actual error conditions that could happen
-   **Better approach**: Test empty arrays, missing keys, etc. instead of malformed data

---

## Advanced Testing Patterns

### Async Operations & State Management

#### Custom Hooks with Async Operations
```javascript
import { renderHook, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const createWrapper = () => {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } }
  });
  return ({ children }) => (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
};

it('should handle async data fetching', async () => {
  const { result } = renderHook(() => useAsyncData(), {
    wrapper: createWrapper()
  });

  expect(result.current.isLoading).toBe(true);
  
  await waitFor(() => {
    expect(result.current.isLoading).toBe(false);
  });

  expect(result.current.data).toEqual(expectedData);
});
```

#### Error Boundary Testing
```javascript
import { ErrorBoundary } from '../components/ErrorBoundary';

const ThrowError = ({ shouldThrow }) => {
  if (shouldThrow) throw new Error('Test error');
  return <div>No error</div>;
};

it('should catch and display errors', () => {
  const consoleSpy = jest.spyOn(console, 'error').mockImplementation();
  
  render(
    <ErrorBoundary fallback={<div>Error caught</div>}>
      <ThrowError shouldThrow />
    </ErrorBoundary>
  );

  expect(screen.getByText('Error caught')).toBeInTheDocument();
  consoleSpy.mockRestore();
});
```

### Performance & Accessibility Testing

#### Performance Budget Testing
```javascript
import { PerformanceObserver } from 'perf_hooks';

it('should render within performance budget', async () => {
  const startTime = performance.now();
  
  render(<ExpensiveComponent data={largeDataSet} />);
  
  await waitFor(() => {
    expect(screen.getByTestId('content')).toBeInTheDocument();
  });

  const endTime = performance.now();
  const renderTime = endTime - startTime;
  
  expect(renderTime).toBeLessThan(100); // 100ms budget
});
```

#### Accessibility Testing
```javascript
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

it('should have no accessibility violations', async () => {
  const { container } = render(<AccessibleComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});

it('should support keyboard navigation', () => {
  render(<NavigableComponent />);
  
  const firstButton = screen.getByRole('button', { name: 'First' });
  const secondButton = screen.getByRole('button', { name: 'Second' });
  
  firstButton.focus();
  fireEvent.keyDown(firstButton, { key: 'Tab' });
  
  expect(secondButton).toHaveFocus();
});
```

### Complex Component Integration

#### Multi-Provider Component Testing
```javascript
const AllProviders = ({ children }) => (
  <QueryClientProvider client={testQueryClient}>
    <ThemeProvider theme={testTheme}>
      <AuthProvider>
        <Router>
          {children}
        </Router>
      </AuthProvider>
    </ThemeProvider>
  </QueryClientProvider>
);

it('should handle complex integration scenarios', async () => {
  render(<ComplexComponent />, { wrapper: AllProviders });
  
  // Test initial state
  expect(screen.getByTestId('loading')).toBeInTheDocument();
  
  // Wait for async operations
  await waitFor(() => {
    expect(screen.getByTestId('content')).toBeInTheDocument();
  });
  
  // Test user interactions
  fireEvent.click(screen.getByRole('button', { name: 'Action' }));
  
  await waitFor(() => {
    expect(screen.getByTestId('success-message')).toBeInTheDocument();
  });
});
```

### Testing Patterns by Component Type

#### Form Component Testing
```javascript
it('should handle form submission with validation', async () => {
  const mockSubmit = jest.fn();
  render(<ContactForm onSubmit={mockSubmit} />);

  // Test required field validation
  fireEvent.click(screen.getByRole('button', { name: 'Submit' }));
  expect(screen.getByText('Name is required')).toBeInTheDocument();

  // Fill form
  fireEvent.change(screen.getByLabelText('Name'), {
    target: { value: 'John Doe' }
  });
  fireEvent.change(screen.getByLabelText('Email'), {
    target: { value: 'john@example.com' }
  });

  // Test successful submission
  fireEvent.click(screen.getByRole('button', { name: 'Submit' }));
  
  await waitFor(() => {
    expect(mockSubmit).toHaveBeenCalledWith({
      name: 'John Doe',
      email: 'john@example.com'
    });
  });
});
```

#### Modal/Dialog Testing
```javascript
it('should handle modal lifecycle', () => {
  const mockClose = jest.fn();
  render(<Modal isOpen onClose={mockClose} />);

  // Test modal is visible
  expect(screen.getByRole('dialog')).toBeInTheDocument();

  // Test close on escape key
  fireEvent.keyDown(document, { key: 'Escape' });
  expect(mockClose).toHaveBeenCalledTimes(1);

  // Test focus trap
  const firstButton = screen.getByRole('button', { name: 'Cancel' });
  const lastButton = screen.getByRole('button', { name: 'Confirm' });
  
  firstButton.focus();
  fireEvent.keyDown(firstButton, { key: 'Tab', shiftKey: true });
  expect(lastButton).toHaveFocus();
});
```

#### List/Table Component Testing
```javascript
it('should handle large data sets efficiently', () => {
  const largeDataSet = Array.from({ length: 1000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));

  render(<VirtualizedList items={largeDataSet} />);

  // Test initial render
  expect(screen.getByText('Item 0')).toBeInTheDocument();
  expect(screen.queryByText('Item 999')).not.toBeInTheDocument();

  // Test scrolling behavior
  const container = screen.getByTestId('list-container');
  fireEvent.scroll(container, { target: { scrollTop: 10000 } });

  waitFor(() => {
    expect(screen.getByText('Item 999')).toBeInTheDocument();
  });
});
```

### Internationalization (i18n) Testing
```javascript
import { I18nextProvider } from 'react-i18next';
import i18n from '../i18n/test-config';

const renderWithI18n = (component, { locale = 'en' } = {}) => {
  i18n.changeLanguage(locale);
  return render(
    <I18nextProvider i18n={i18n}>
      {component}
    </I18nextProvider>
  );
};

it('should display text in different languages', () => {
  // Test English
  renderWithI18n(<WelcomeMessage />);
  expect(screen.getByText('Welcome')).toBeInTheDocument();

  // Test Spanish
  renderWithI18n(<WelcomeMessage />, { locale: 'es' });
  expect(screen.getByText('Bienvenido')).toBeInTheDocument();
});
```

### Responsive Design Testing
```javascript
import { act } from '@testing-library/react';

const resizeWindow = (width, height) => {
  act(() => {
    global.innerWidth = width;
    global.innerHeight = height;
    global.dispatchEvent(new Event('resize'));
  });
};

it('should adapt to different screen sizes', () => {
  render(<ResponsiveComponent />);

  // Test desktop view
  resizeWindow(1024, 768);
  expect(screen.getByTestId('desktop-nav')).toBeInTheDocument();
  expect(screen.queryByTestId('mobile-nav')).not.toBeInTheDocument();

  // Test mobile view
  resizeWindow(375, 667);
  expect(screen.queryByTestId('desktop-nav')).not.toBeInTheDocument();
  expect(screen.getByTestId('mobile-nav')).toBeInTheDocument();
});
```

### Testing with Different User Permissions/Roles
```javascript
const renderWithUser = (component, userRole = 'user') => {
  const mockUser = {
    id: '1',
    role: userRole,
    permissions: getRolePermissions(userRole)
  };

  return render(
    <AuthProvider value={{ currentUser: mockUser }}>
      {component}
    </AuthProvider>
  );
};

describe('Dashboard access control', () => {
  it('should show admin features for admin users', () => {
    renderWithUser(<Dashboard />, 'admin');
    expect(screen.getByRole('button', { name: 'Admin Panel' })).toBeInTheDocument();
  });

  it('should hide admin features for regular users', () => {
    renderWithUser(<Dashboard />, 'user');
    expect(screen.queryByRole('button', { name: 'Admin Panel' })).not.toBeInTheDocument();
  });
});
```

---

## Quality Assurance Checklist

### Pre-Test Implementation
- [ ] Review component requirements and user stories
- [ ] Identify critical user paths and edge cases
- [ ] Check existing test patterns in the codebase
- [ ] Verify all dependencies are properly mocked

### During Test Implementation
- [ ] Test renders without crashing
- [ ] Test all required props and prop variations
- [ ] Test user interactions (clicks, form inputs, keyboard)
- [ ] Test loading, success, and error states
- [ ] Test accessibility requirements (ARIA, keyboard navigation)
- [ ] Test responsive behavior if applicable

### Post-Test Implementation
- [ ] All tests pass consistently
- [ ] Coverage targets are met (80%+ meaningful coverage)
- [ ] No TypeScript compilation errors
- [ ] No ESLint violations
- [ ] Tests run within performance budget (<5s per file)
- [ ] Test names clearly describe the behavior being tested

### Integration Testing Considerations
- [ ] Test component integration points
- [ ] Validate API contract adherence
- [ ] Test with realistic data variations
- [ ] Verify error handling across component boundaries
- [ ] Test performance with large data sets

---

## Common Troubleshooting Guide

### Test Performance Issues
```javascript
// Use fake timers for animations and delays
beforeEach(() => {
  jest.useFakeTimers();
});

afterEach(() => {
  jest.useRealTimers();
});

// Mock expensive operations
jest.mock('./expensiveUtility', () => ({
  expensiveFunction: jest.fn().mockReturnValue('mock-result')
}));
```

### Memory Leaks in Tests
```javascript
afterEach(() => {
  // Clean up timers
  jest.clearAllTimers();
  
  // Clean up mocks
  jest.clearAllMocks();
  
  // Clean up DOM
  cleanup();
  
  // Clean up any global state
  resetGlobalState();
});
```

### Debugging Test Failures
```javascript
// Add debug output
import { screen } from '@testing-library/react';

it('should debug failing test', () => {
  render(<ProblematicComponent />);
  
  // Output current DOM
  screen.debug();
  
  // Query for elements with detailed logging
  const element = screen.getByRole('button', { name: /submit/i });
  console.log('Element found:', element);
});
```
