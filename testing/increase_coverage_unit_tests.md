Prompt1: 
run yarn jest --coverage=true to execute the tests, by default it will also return coverage per files, with the coverage report we want to create a file currentUnitTestCoverageTasks.md that includes each file/component that has under 80% coverage based on the report, each files will also have a md checkbox [] so we can then start implementing unit tests one by one

Prompt2: 
go through #Current File  and check each file one by one, check if the file does not need to have tests written, in this case, we need to add to jest ignore in jest config, please think about patterns so we can ignore using regex, ex: ignore all files that end with .styled.ts or are named styled.ts, you smart, you do it, if you gitginore a file, you can remove its row from the tasks

Prompt3: (if the project has already uni tests written)
Please check all unit tests written and identify patterns that would be great to have as rules for unit tests, this guidance should help future AI agents, add them in CLAUDE.md or ./cursor/rules/unit-tests.mdc or ./kiro/steering/unit-tests.mdc ruls
Also think if any of the logic is repeated in multiple files, and decide to refactor and create utility functions that would help to write tests easier and not duplicate code (add details about the helper if the unit tests guidance file)
Example:
Always use this helper function to render the component with providers when the test requires it
export const renderWithProviders = (component: React.ReactNode) => {
    return render(
        <QueryClientProvider client={new QueryClient()}>
            <Theme>
                <ToastContainer containerId="defaultToast" />
                <BrowserRouter>{component}</BrowserRouter>
            </Theme>
        </QueryClientProvider>
    );
};

Prompt4:
check #unit-tests.md  and then start implementing unit tests in #Current File  one by one
- write tests for a file
- check test is passing
- fix typescript/lint errors
- mark texts as complete, add x between [. ]
- re-iterate with next file
- lets implement first 5 files in first chunk