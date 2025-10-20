# Playwright: Complete Tutorial and Advanced Guide

## Table of Contents
1. [Introduction to Playwright](#introduction-to-playwright)
2. [Installation and Setup](#installation-and-setup)
3. [Basic Usage](#basic-usage)
4. [Locators and Selectors](#locators-and-selectors)
5. [Advanced Features](#advanced-features)
6. [Testing Strategies](#testing-strategies)
7. [Best Practices](#best-practices)
8. [Troubleshooting](#troubleshooting)
9. [Advanced Examples](#advanced-examples)

## Introduction to Playwright

Playwright is a Node.js library that provides a high-level API to control Chrome, Firefox, Safari, and other browsers. It's designed for end-to-end testing, automation, and web scraping.

### Key Features
- **Cross-browser support**: Works with Chromium, Firefox, and WebKit
- **Auto-waiting**: Automatically waits for elements to be ready
- **Network interception**: Control network requests and responses
- **Mobile emulation**: Test responsive designs
- **Parallel execution**: Run tests in parallel
- **Multiple contexts**: Support for multiple browser contexts

## Installation and Setup

### Prerequisites
- Node.js 16+ (LTS recommended)

### Installation

```bash
# Install Playwright
npm init playwright@latest

# Or install manually
npm install @playwright/test
npm install -D @playwright/test
```

### Install Browser Binaries

```bash
# Install all browsers
npx playwright install

# Install specific browsers
npx playwright install chromium firefox webkit
```

### Basic Configuration

Create `playwright.config.ts`:

```typescript
import { defineConfig } from '@playwright/test'

export default defineConfig({
  // Global setup
  timeout: 30000,
  expect: {
    timeout: 5000
  },
  
  // Global teardown
  globalSetup: require.resolve('./global-setup'),
  globalTeardown: require.resolve('./global-teardown'),
  
  // Test environment
  testDir: './tests',
  testMatch: /.*\.e2e\.spec\.ts/,
  
  // Reporter options
  reporter: [
    ['html'],
    ['json', { outputFile: 'test-results.json' }]
  ],
  
  // Browser configuration
  use: {
    // Browser options
    headless: true,
    viewport: { width: 1280, height: 720 },
    ignoreHTTPSErrors: true,
    video: 'on-first-retry',
    screenshot: 'only-on-failure',
    
    // Context options
    baseURL: 'http://localhost:3000',
    
    // Action options
    actionTimeout: 0, // No timeout for actions
  },
  
  // Workers (parallel execution)
  workers: process.env.CI ? 1 : undefined,
  
  // Retry failed tests
  retries: 1,
  
  // Project configurations
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],
  
  // Web server configuration
  webServer: {
    command: 'npm run serve',
    port: 3000,
    timeout: 120 * 1000,
    reuseExistingServer: !process.env.CI,
  },
})
```

## Basic Usage

### Simple Test Example

```typescript
// example.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Basic Example', () => {
  test('should navigate to page and check title', async ({ page }) => {
    // Navigate to URL
    await page.goto('https://example.com')
    
    // Get title
    const title = await page.title()
    expect(title).toBe('Example Domain')
    
    // Check for specific text
    await expect(page.locator('h1')).toContainText('Example Domain')
  })
})
```

### Page Object Model

```typescript
// pages/home.page.ts
import { Page, Locator } from '@playwright/test'

export class HomePage {
  readonly page: Page
  readonly searchInput: Locator
  readonly searchButton: Locator
  readonly results: Locator

  constructor(page: Page) {
    this.page = page
    this.searchInput = page.locator('#search-input')
    this.searchButton = page.locator('button[type="submit"]')
    this.results = page.locator('.search-results')
  }

  async goto() {
    await this.page.goto('/')
  }

  async search(query: string) {
    await this.searchInput.fill(query)
    await this.searchButton.click()
  }

  async getResultsCount(): Promise<number> {
    return await this.results.count()
  }
}
```

```typescript
// tests/search.spec.ts
import { test, expect } from '@playwright/test'
import { HomePage } from '../pages/home.page'

test.describe('Search Functionality', () => {
  test('should search and return results', async ({ page }) => {
    const homePage = new HomePage(page)
    await homePage.goto()
    
    await homePage.search('playwright')
    const count = await homePage.getResultsCount()
    
    expect(count).toBeGreaterThan(0)
  })
})
```

## Locators and Selectors

### Locator Strategies

```typescript
// Text-based locators
page.locator('text=Login')
page.locator('text=/Log.*in/') // Regex

// Role-based locators (recommended for accessibility)
page.getByRole('button', { name: 'Submit' })
page.getByRole('link', { name: 'Home' })
page.getByRole('textbox', { name: 'Username' })

// Label-based locators
page.getByLabel('Username')
page.getByLabel('Password')

// Placeholder-based locators
page.getByPlaceholder('Enter username')

// Alt text for images
page.getByAltText('Logo')

// Title attribute
page.getByTitle('Help')

// Test ID (recommended for stable selectors)
page.getByTestId('submit-button')
```

### CSS Selectors

```typescript
// Basic CSS selectors
page.locator('#myId')
page.locator('.myClass')
page.locator('button')
page.locator('input[type="text"]')

// Complex CSS selectors
page.locator('form > input[name="username"]')
page.locator('div.container > ul li:nth-child(2)')
page.locator('a[href*="example.com"]')

// Attribute selectors
page.locator('[data-testid="my-element"]')
page.locator('[aria-label="Close"]')
```

### Advanced Locators

```typescript
// Chained locators
page.locator('form').locator('input').first()
page.locator('nav').getByRole('link').nth(2)

// Frame locators
const frame = page.frameLocator('#myFrame')
await frame.locator('#input-in-frame').fill('text')

// Filter locators
page.locator('li').filter({ hasText: 'Active' })
page.locator('button').filter({ has: page.locator('.icon') })

// Multiple conditions
page.locator('button', { hasText: 'Submit', visible: true })
```

## Advanced Features

### Network Interception

```typescript
import { test, expect } from '@playwright/test'

test('network interception example', async ({ page }) => {
  // Mock API response
  await page.route('**/api/users', route => {
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, name: 'John Doe' },
        { id: 2, name: 'Jane Smith' }
      ])
    })
  })

  // Block specific requests
  await page.route('**/*.(png|jpg|jpeg)', route => route.abort())

  // Wait for specific request
  const [response] = await Promise.all([
    page.waitForResponse('**/api/login'),
    page.locator('#login-button').click()
  ])

  expect(response.status()).toBe(200)
})
```

### File Upload and Download

```typescript
// File upload
test('file upload', async ({ page }) => {
  // Single file upload
  await page.locator('#file-input').setInputFiles('path/to/file.pdf')
  
  // Multiple file upload
  await page.locator('#file-input').setInputFiles([
    'path/to/file1.pdf',
    'path/to/file2.pdf'
  ])
})

// File download
test('file download', async ({ page }) => {
  // Wait for download
  const [download] = await Promise.all([
    page.waitForEvent('download'),
    page.locator('#download-button').click()
  ])
  
  // Save download
  await download.saveAs('downloads/my-file.pdf')
  
  // Check download details
  expect(download.suggestedFilename()).toBe('report.pdf')
})
```

### Authentication

```typescript
// Basic authentication
test.use({
  httpCredentials: {
    username: 'user',
    password: 'password'
  }
})

// API token authentication
test('API token auth', async ({ page }) => {
  await page.context().setExtraHTTPHeaders({
    'Authorization': 'Bearer my-token'
  })
  
  await page.goto('/protected-page')
})

// Cookie-based authentication
test('cookie auth', async ({ page }) => {
  await page.context().addCookies([{
    name: 'session',
    value: 'my-session-id',
    domain: 'example.com',
    path: '/'
  }])
  
  await page.goto('/dashboard')
})
```

### Mobile and Responsive Testing

```typescript
import { test, expect } from '@playwright/test'

test.describe('Mobile Testing', () => {
  test.use({
    viewport: { width: 375, height: 667 }, // iPhone SE
    userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X)'
  })

  test('mobile navigation', async ({ page }) => {
    await page.goto('/')
    
    // Test mobile-specific interactions
    await page.locator('.mobile-menu-toggle').click()
    await expect(page.locator('.mobile-menu')).toBeVisible()
    
    // Simulate touch events
    await page.locator('#touch-element').tap()
  })
})
```

### Visual Regression Testing

```typescript
test('visual regression', async ({ page }) => {
  await page.goto('/dashboard')
  
  // Take screenshot
  await expect(page).toHaveScreenshot('dashboard.png')
  
  // Take screenshot of specific element
  await expect(page.locator('#header')).toHaveScreenshot('header.png')
  
  // Custom screenshot options
  await expect(page).toHaveScreenshot('dashboard-full.png', {
    fullPage: true,
    omitBackground: true,
    animations: 'disabled'
  })
})
```

## Testing Strategies

### Page Object Model (POM)

```typescript
// Base page class
export class BasePage {
  protected page: Page

  constructor(page: Page) {
    this.page = page
  }

  async navigateTo(url: string) {
    await this.page.goto(url)
  }

  async waitForLoad() {
    await this.page.waitForLoadState('networkidle')
  }

  async takeScreenshot(name: string) {
    await this.page.screenshot({ path: `screenshots/${name}.png` })
  }
}

// Specific page class
export class LoginPage extends BasePage {
  private usernameInput: Locator
  private passwordInput: Locator
  private loginButton: Locator

  constructor(page: Page) {
    super(page)
    this.usernameInput = page.locator('#username')
    this.passwordInput = page.locator('#password')
    this.loginButton = page.locator('#login-button')
  }

  async login(username: string, password: string) {
    await this.usernameInput.fill(username)
    await this.passwordInput.fill(password)
    await this.loginButton.click()
  }

  async isLoginErrorVisible(): Promise<boolean> {
    return await this.page.locator('.error-message').isVisible()
  }
}
```

### Data-Driven Testing

```typescript
test.describe('Data-driven tests', () => {
  const testData = [
    { username: 'user1', password: 'pass1', expected: 'Dashboard' },
    { username: 'user2', password: 'pass2', expected: 'Dashboard' },
    { username: 'invalid', password: 'wrong', expected: 'Error' }
  ]

  for (const data of testData) {
    test(`login test for ${data.username}`, async ({ page }) => {
      await page.goto('/login')
      
      await page.locator('#username').fill(data.username)
      await page.locator('#password').fill(data.password)
      await page.locator('#login-button').click()
      
      if (data.expected === 'Dashboard') {
        await expect(page).toHaveURL('/dashboard')
      } else {
        await expect(page.locator('.error')).toBeVisible()
      }
    })
  }
})
```

### Parallel Execution

```typescript
// playwright.config.ts
export default defineConfig({
  // Run tests in parallel
  workers: 3, // Number of parallel workers
  
  // Or based on available CPU cores
  workers: process.env.CI ? 2 : undefined,
  
  // Test sharding
  shard: {
    total: 4, // Total number of shards
    current: 1 // Current shard (1-4)
  }
})
```

## Best Practices

### 1. Use Semantic Locators

```typescript
// ❌ Avoid - Fragile CSS selectors
page.locator('div:nth-child(2) > span:first-child')

// ✅ Prefer - Semantic locators
page.getByRole('button', { name: 'Submit' })
page.getByLabel('Username')
page.getByTestId('submit-button')
```

### 2. Implement Proper Waits

```typescript
// ❌ Avoid - Hardcoded waits
await page.waitForTimeout(2000)

// ✅ Prefer - Explicit waits
await expect(page.locator('#result')).toBeVisible()
await page.waitForURL('/dashboard')
await page.waitForResponse('**/api/data')
```

### 3. Organize Tests with Describes

```typescript
test.describe('User Authentication', () => {
  test.describe('Login', () => {
    test('valid credentials', async ({ page }) => {
      // Test implementation
    })
    
    test('invalid credentials', async ({ page }) => {
      // Test implementation
    })
  })
  
  test.describe('Logout', () => {
    // Logout tests
  })
})
```

### 4. Use Fixtures for Common Setup

```typescript
// fixtures/auth.fixture.ts
import { test as base } from '@playwright/test'

type AuthFixtures = {
  authenticatedPage: Page
}

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ browser }, use) => {
    const context = await browser.newContext()
    const page = await context.newPage()
    
    // Perform authentication
    await page.goto('/login')
    await page.locator('#username').fill('testuser')
    await page.locator('#password').fill('password')
    await page.locator('#login-button').click()
    
    await use(page)
    await context.close()
  }
})

export { expect } from '@playwright/test'
```

### 5. Handle Test Dependencies

```typescript
test.describe('Dependent Tests', () => {
  let userId: string

  test('create user', async ({ page }) => {
    await page.goto('/users/create')
    await page.locator('#name').fill('John Doe')
    await page.locator('#email').fill('john@example.com')
    await page.locator('#submit').click()
    
    // Get created user ID
    userId = await page.locator('#user-id').textContent()
    expect(userId).not.toBeNull()
  })

  test('edit user', async ({ page }) => {
    test.skip(!userId, 'User not created')
    
    await page.goto(`/users/${userId}/edit`)
    await page.locator('#name').fill('Jane Doe')
    await page.locator('#save').click()
    
    await expect(page.locator('#success-message')).toBeVisible()
  })
})
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Element Not Found

```typescript
// Debug element visibility
await page.locator('#my-element').waitFor()
await expect(page.locator('#my-element')).toBeVisible()

// Check if element is in iframe
const frame = page.frameLocator('#my-frame')
await frame.locator('#element-in-frame').click()

// Wait for dynamic content
await page.waitForSelector('#dynamic-element', { state: 'visible' })
```

#### 2. Timing Issues

```typescript
// Wait for network idle
await page.waitForLoadState('networkidle')

// Wait for specific response
await page.waitForResponse('**/api/data')

// Wait for navigation
await Promise.all([
  page.waitForNavigation(),
  page.locator('#link').click()
])
```

#### 3. Cross-Origin Issues

```typescript
// Configure context for cross-origin
const context = await browser.newContext({
  permissions: ['notifications'],
  extraHTTPHeaders: {
    'Accept-Language': 'en-US,en;q=0.9'
  }
})
```

### Debugging Techniques

```typescript
// Enable debug mode
PWDEBUG=1 npm run test

// Open browser in headed mode
headless: false,

// Slow down execution
slowMo: 1000, // 1 second between actions

// Record video for debugging
video: 'on'

// Take screenshots on failure
screenshot: 'only-on-failure'
```

## Advanced Examples

### API Testing with Playwright

```typescript
import { test, expect } from '@playwright/test'

test('API testing with Playwright', async ({ request }) => {
  // GET request
  const response = await request.get('/api/users')
  expect(response.status()).toBe(200)
  
  const users = await response.json()
  expect(users.length).toBeGreaterThan(0)
  
  // POST request
  const createUserResponse = await request.post('/api/users', {
    data: {
      name: 'John Doe',
      email: 'john@example.com'
    }
  })
  
  expect(createUserResponse.status()).toBe(201)
  
  // DELETE request
  const deleteResponse = await request.delete(`/api/users/${users[0].id}`)
  expect(deleteResponse.status()).toBe(200)
})
```

### Custom Test Reporter

```typescript
// custom-reporter.ts
import { Reporter, TestCase, TestResult } from '@playwright/test/reporter'

class CustomReporter implements Reporter {
  onTestEnd(test: TestCase, result: TestResult): void {
    console.log(`${test.title}: ${result.status}`)
  }

  onEnd(): void {
    console.log('All tests completed')
  }
}

export default CustomReporter
```

### Environment-Specific Configuration

```typescript
// playwright.config.ts
const config = defineConfig({
  use: {
    baseURL: process.env.ENVIRONMENT === 'production' 
      ? 'https://prod.example.com'
      : 'https://staging.example.com',
  },
  
  projects: [
    {
      name: 'production',
      use: { 
        baseURL: 'https://prod.example.com',
        ...devices['Desktop Chrome']
      },
      testIgnore: process.env.SKIP_PROD ? '*' : undefined
    },
    {
      name: 'staging',
      use: { 
        baseURL: 'https://staging.example.com',
        ...devices['Desktop Chrome']
      }
    }
  ]
})
```

### Custom Assertions

```typescript
import { expect, Page } from '@playwright/test'

// Extend expect with custom matchers
expect.extend({
  async toHaveValidForm(page: Page, selector: string) {
    const form = page.locator(selector)
    const isValid = await form.evaluate(form => (form as HTMLFormElement).checkValidity())
    
    return {
      pass: isValid,
      message: () => `Expected form to be valid but it was invalid`
    }
  }
})

// Usage
await expect(page).toHaveValidForm('#registration-form')
```

## Performance Optimization

### 1. Efficient Selectors

```typescript
// Use stable selectors
// ✅ Data attributes
page.locator('[data-testid="submit-button"]')

// ✅ Semantic selectors
page.getByRole('button', { name: 'Submit' })

// ❌ Fragile selectors
page.locator('div:nth-child(2) > span:first-child')
```

### 2. Minimize Browser Contexts

```typescript
// Reuse contexts when possible
const context = await browser.newContext()
const page1 = await context.newPage()
const page2 = await context.newPage() // Reuse same context

// Instead of creating new contexts
const context1 = await browser.newContext()
const context2 = await browser.newContext()
```

### 3. Batch Operations

```typescript
// Use Promise.all for parallel operations
const [response1, response2] = await Promise.all([
  page.waitForResponse('**/api/users'),
  page.waitForResponse('**/api/posts'),
  page.locator('#load-data').click()
])
```

This comprehensive guide covers Playwright from basic usage to advanced techniques, providing practical examples and best practices for effective end-to-end testing.