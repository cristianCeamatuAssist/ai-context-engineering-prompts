---
description:
globs:
alwaysApply: true
---

-   Use `yarn jest -u <filename> <filename>` to check tests are passing
-   Whenever we write a test and want to mock the window object, use this exemple as a reference:let windowSpy;

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

-   Please use jest.mock() instead of jest.spyOn() to mock implementations:
    BAD:
    const updateAllServiceWorkersMock = jest > 39 | .spyOn(require('./service-workers.utils'), 'updateAllServiceWorkers')
    | ^
    40 | .mockResolvedValue(undefined);
    GOOD
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

-   After you write unit/snapshots tests, always run the test command to check the written tests are passing

-   This is bad way to mock non default exports:
    jest.mock('src/features/account/hooks/useUser', () => () => ({
    isSuperKeyUser: true,
    userData: { id: '1' }
    }));
    jest.mock('src/features/auth/hooks/useAuth', () => () => ({
    refreshSession: jest.fn(() => Promise.resolve()),
    decodedIdToken: {}
    }));
    Instead mock them like this:
    jest.mock('src/features/account/hooks/useUser', () => (useUser: () => ({
    isSuperKeyUser: true,
    userData: { id: '1' }
    })));
    jest.mock('src/features/auth/hooks/useAuth', () => ({
    useAuth: () => ({
    refreshSession: jest.fn(() => Promise.resolve()),
    decodedIdToken: {}
    })
    }));

-   never run `yarn lint --fix` command or commands that format all the project because it adds to many changes to the PR

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
