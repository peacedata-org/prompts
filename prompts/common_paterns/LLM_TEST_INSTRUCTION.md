## Phase 1: Project Structure

```text
my-nuxt-app/
├── tests/
│   ├── user-stories/              # Markdown user stories
│   │   ├── login/
│   │   │   └── user_story.md
│   │   └── purchase/
│   │       └── user_story.md
│   ├── unit/                      # Minimal mocks
│   ├── integration/
│   │   ├── backend/
│   │   │   ├── login/
│   │   │   │   └── login.test.ts
│   │   │   ├── purchase/
│   │   │   │   └── purchase.test.ts
│   │   │   └── utils/             # Backend test utilities
│   │   │       └── test-helpers.ts
│   │   └── frontend/
│   │       ├── login/
│   │       │   └── login-form.test.ts
│   │       ├── purchase/
│   │       │   └── checkout-form.test.ts
│   │       └── utils/             # Frontend test utilities
│   │           └── test-helpers.ts
│   └── e2e/
│       ├── login/
│       │   └── login.e2e.ts
│       └── purchase/
│           └── purchase.e2e.ts
├── docker-compose.test.yml        # Self-hosted test environment
└── .github/workflows/
    └── ci.yml                     # Self-hosted runner config
```

## Phase 2: TDD Algorithm

The TDD Workflow Algorithm Rules:

### Glossary:

#### SCOPE:

- BACKEND
- FRONTEND
- END2END

#### USER_STORY

A Markdown document detailing a specific user feature, outlining the desired behavior and ACCEPTANCE_CRITERIA for that feature.

#### ACCEPTANCE_CRITERIA

Specific, measurable conditions that must be met for a user story or feature to be considered complete and working as expected from the user's perspective.

##### Best Practices:

    - Use clear, simple language.
    - Focus on what the user can do, not how it's built.
    - Make them testable and verifiable.
    - Include both positive (happy path) and negative (error cases) scenarios.
    - Align with the user story's goal.

#### TEST_TEMPLATE

A Vitest/Playwright file containing describe and it.todo() blocks that outline the test scenarios derived from a USER_STORY, placed in the appropriate tests/integration/[scope]/[feature-name]/ directory.

### TDD Workflow Algorithm:

1. Check and Prepare CI/CD & Environment:
   - Check Existing CI/CD Pipeline: Verify if a GitHub Actions workflow (e.g., .github/workflows/ci.yml) is already configured for the project.
   - Evaluate Pipeline Needs: If a pipeline exists, assess if it requires modifications to correctly automate the upcoming test sequence (Backend -> Frontend -> E2E) and manage the test environment. Modifications should preserve existing, unrelated workflow functionality.
   - Modify or Create Pipeline: If modifications are needed, update the existing pipeline. If no suitable pipeline exists, create a new GitHub Actions workflow file (e.g., .github/workflows/ci.yml) for your self-hosted runner.
   - Check Test Environment: Verify if docker-compose.test.yml exists and defines the necessary services (app, test db, etc.) for the isolated test environment.
   - Evaluate Environment Needs: Assess if the existing docker-compose.test.yml requires modifications to correctly provision the test environment matching production requirements for the new feature's tests.
   - Modify or Create Environment: If modifications are needed, update the existing docker-compose.test.yml. If it doesn't exist, create it to define the required test services.
   - Ensure Pipeline Integration: Confirm that the CI/CD pipeline correctly utilizes the docker-compose.test.yml to set up, run tests within, and tear down the isolated test environment
2. Write USER_STORY: Create a Markdown file in tests/user-stories/[feature-name]/user_story.md describing the desired behavior.
3. You SHOULD create TEST_TEMPLATES in files, that includes test scenario for each scope: END2END, FRONTEND, BACKEND:
   - Tests should be based on the USER_STORY of that feature
   - Tests SHOULD USE the comprehensive `describe` and `it` blocks without any logic that should cover all the scenarios.
   - Tests SHOULD be marked as `it.todo()` initially
   - Tests SHOULD be places in tests/integration/[scope]/[feature-name].test.ts
   - Writing test spec SHOULD be started in this order: END2END, FRONTEND, BACKEND
   - Tests SHOULD include positive (first) and negative scenarios
4. After writing TEST_TEMPLATES, they SHOULD be double checked for that type of logical errors in context of USER_STORY:
   - Test SHOULD be fixed if it has logical errors.
   - Test SHOULD be added if (other tests are missing) and (there are not enough checks provided).
   - Test SHOULD be removed if they are redundant.
5. After reviewing the TEST_TEMPLATES, the BACKEND TESTS logic MUST be developed.
6. Once BACKEND TESTS logic is established, the BACKEND implementation SHOULD proceed.
7. Upon completing BACKEND development, BACKEND TESTS MUST be executed.
8. If failures occur, the BACKEND requires correction until all BACKEND TESTS pass.
9. When BACKEND TESTS complete successfully, FRONTEND TESTS logic MUST be constructed.
10. Following FRONTEND TESTS logic creation, the FRONTEND implementation SHOULD commence.
11. After concluding FRONTEND development, FRONTEND TESTS MUST be executed.
12. Should errors arise, the FRONTEND needs fixing until all FRONTEND TESTS pass.
13. Once FRONTEND TESTS execute successfully, E2E TESTS logic MUST be established.
14. After E2E TESTS logic is in place, E2E TESTS SHOULD be executed.
15. If issues emerge, E2E TESTS require debugging until all E2E TESTS pass.
