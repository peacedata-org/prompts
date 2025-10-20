# Nuxt 4 Testing & TDD Guide

## Test Stack

- **Vitest**: Unit, component, integration tests (official Nuxt support via `@nuxt/test-utils`)
- **Playwright**: E2E browser tests only (via `@nuxt/test-utils/e2e`)
- **Avoid**: Jest (ESM incompatible), Cucumber (enterprise overhead)

## TDD Workflow

1. Run `vitest --ui` in watch mode continuously
2. Write failing test first (red)
3. Implement minimal code to pass (green)
4. Refactor with confidence (tests stay green)
5. Sub-second feedback loop via Vitest's smart re-running

## Vitest Configuration

```typescript
// vitest.config.ts
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
    // fast unit tests in Node, Nuxt-aware tests in nuxt env
  }
})
```

## Test Writing Guidelines

### Component Tests

```typescript
import { mountSuspended } from '@nuxt/test-utils/runtime'
import MyComponent from '~/components/MyComponent.vue'

describe('MyComponent', () => {
  it('renders with nuxt context', async () => {
    const wrapper = await mountSuspended(MyComponent, {
      props: { title: 'test' }
    })
    expect(wrapper.text()).toContain('test')
  })
})
```

### Composable Tests

```typescript
import { mockNuxtImport } from '@nuxt/test-utils/runtime'

mockNuxtImport('useRouter', () => ({
  push: vi.fn()
}))

// test composable directly
```

### E2E Tests (Playwright)

```typescript
import { test, expect } from '@nuxt/test-utils/playwright'

test('user can login', async ({ page, goto }) => {
  await goto('/login', { waitUntil: 'hydration' })
  await page.getByLabel('Email').fill('user@test.com')
  // ...
})
```

## Code Style

- lowercase comments and log messages
- descriptive test names in plain english
- one assertion focus per test
- arrange-act-assert pattern

## AI Generation Tips

- Vitest syntax matches standard testing conventions (describe/it/expect)
- Use `mountSuspended` for components with Nuxt context
- Request "vitest test for [feature]" - models know the patterns
- Playwright tests: "e2e test for [user journey]"

## Test Organization

```
tests/
├── unit/           # *.spec.ts - fast Vitest tests
├── components/     # *.nuxt.spec.ts - Nuxt environment
└── e2e/           # *.e2e.spec.ts - Playwright only
```

## CI/CD Scripts

```json
{
  "test": "vitest",
  "test:ui": "vitest --ui",
  "test:e2e": "playwright test"
}
```

## Key Principles

- **Fast feedback**: Vitest watch mode runs affected tests in ~20ms
- **Zero config TypeScript**: Native ESM, auto-imports work automatically
- **Test pyramid**: Many unit (Vitest) → Some integration → Few E2E (Playwright)
- **Playwright only for critical paths**: Browser tests are slow, use sparingly
- **No mocking gymnastics**: `@nuxt/test-utils` provides real Nuxt context

## When to Use Each

- **Vitest**: Functions, composables, components, server utils, Pinia stores
- **Playwright**: Login flows, checkout, navigation, SSR/hydration validation

## Debug Commands

- `vitest --ui` - visual test runner with time-travel
- `playwright test --ui` - E2E debugging with DOM snapshots
- `playwright codegen` - generate E2E tests from browser interaction