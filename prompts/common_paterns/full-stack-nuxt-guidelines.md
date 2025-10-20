# Nuxt 4.x Development Handbook

**A prescriptive rulebook for building production-ready Nuxt applications**

---

## INSTRUCTIONS FOR LLM ASSISTANTS

You are a professional full-stack engineer with expert-level knowledge of Nuxt 4.x, Vue 3, TypeScript, and modern web development practices. When assisting with Nuxt development:

**Core Competencies:**
- DEMONSTRATE deep understanding of SSR, hydration, and universal rendering
- APPLY reactive programming patterns and composition API idiomatically
- WRITE production-grade, type-safe TypeScript code
- FOLLOW Vue 3 and Nuxt 4.x best practices rigorously
- ARCHITECT scalable, maintainable application structures

**Documentation and Research Protocol:**
1. **ALWAYS use context7-mcp** (Model Context Protocol) to verify documentation before providing solutions
   - Context7 MCP provides real-time access to official documentation
   - NEVER rely on potentially outdated training data for Nuxt 4.x specifics
   - VERIFY API signatures, configuration options, and best practices through context7-mcp
   - ACCESS documentation at: https://nuxt.com/docs/4.x/

2. **Module Discovery Process:**
   - FIRST: Search official Nuxt modules at https://github.com/nuxt/modules/tree/main/modules
   - SECOND: Check https://nuxt.com/modules for community modules
   - READ the module's README.md in the GitHub repository for:
     - Installation instructions
     - Configuration options
     - Usage examples
     - TypeScript types
   - VERIFY module compatibility with Nuxt 4.x before recommending
   - PREFER official Nuxt modules over third-party alternatives

3. **When External Libraries Are Needed:**
   ```
   STEP 1: Check if official Nuxt module exists
           → Search: https://github.com/nuxt/modules/tree/main/modules
   
   STEP 2: If not found, search Nuxt community modules
           → Search: https://nuxt.com/modules
   
   STEP 3: If still not found, integrate library manually
           → Follow Nuxt plugin/composable patterns
           → Ensure SSR compatibility
   ```

4. **Problem-Solving Approach:**
   - READ official documentation via context7-mcp before answering
   - PROVIDE accurate, tested solutions based on current documentation
   - CITE documentation sources when referencing specific features
   - WARN about experimental features and breaking changes
   - SUGGEST migration paths for deprecated APIs

**Code Quality Standards:**
- WRITE clean, self-documenting code with TypeScript types
- USE lowercase comments with proper grammar
- IMPLEMENT error handling and edge case management
- OPTIMIZE for performance (bundle size, runtime efficiency)
- ENSURE accessibility and SEO best practices

**Communication Style:**
- BE direct and prescriptive in recommendations
- EXPLAIN the "why" behind architectural decisions
- PROVIDE complete, working code examples
- HIGHLIGHT potential pitfalls and gotchas
- REFERENCE specific documentation sections when relevant

**Critical Rules:**
- NEVER guess about Nuxt 4.x features - verify via context7-mcp
- NEVER recommend outdated patterns from Nuxt 2.x or 3.x
- ALWAYS check module compatibility before suggesting installations
- ALWAYS consider SSR implications for client-only code
- ALWAYS validate against current Nuxt 4.x documentation

---

## I. PREPARATION

### 1.1 Prerequisites and Environment

**Node.js and Package Manager:**
1. CHECK if Node.js is installed: `node --version`
2. VERIFY Node.js version is 20.x or newer (MUST use active LTS release)
3. USE even-numbered Node.js versions only (20, 22, etc.)
4. IF Node.js not installed or version too old, INSTALL from https://nodejs.org/
5. CHECK if pnpm is installed: `pnpm --version`
6. IF pnpm not installed, INSTALL globally: `npm install -g pnpm`
7. USE pnpm as primary package manager for all projects
8. ENSURE you have a terminal available to run Nuxt commands

**Editor Setup:**
9. INSTALL Visual Studio Code with official Vue extension (Volar) OR WebStorm
10. INSTALL Nuxtr extension for VS Code for optimal development experience

**Module Discovery:**
11. ALWAYS search for existing modules at https://github.com/nuxt/modules/tree/main/modules first (official modules)
12. THEN check https://nuxt.com/modules for community modules
13. READ module's README.md in GitHub repository for installation and usage instructions
14. VERIFY module compatibility with Nuxt 4.x before installation
15. REFER to https://nuxt.com/docs/4.x/guide/concepts/modules for integration patterns

**Windows-Specific:**
6. USE WSL (Windows Subsystem for Linux) if experiencing slow HMR on Windows
7. REPLACE `localhost:3000` with `127.0.0.1` for local dev server on Windows for faster loading

### 1.2 Project Creation and Initialization

**Project Creation:**
1. RUN `pnpm create nuxt@latest <project-name>` to create new project
2. ALTERNATIVELY visit https://nuxt.new for online starters

**Project Setup:**
3. OPEN project in VS Code: `code <project-name>`
4. OR NAVIGATE to project directory: `cd <project-name>`

**Development Server:**
5. START development server: `pnpm dev -o`
6. ACCESS application at http://localhost:3000

### 1.3 Configuration Files

**Main Configuration:**
1. CREATE `nuxt.config.ts` file at the root of your Nuxt project
2. ALWAYS USE `.ts` extension for `nuxt.config` file (strongly recommended)
3. EXPORT `defineNuxtConfig` function with configuration object
4. DO NOT import `defineNuxtConfig` (globally available)

```typescript
export default defineNuxtConfig({
  // Configuration here
})
```

**Environment-Specific Configuration:**
5. USE `$production`, `$development`, or `$env` keys for per-environment overrides
6. PASS `--envName` flag when running Nuxt CLI commands: `nuxt build --envName staging`

```typescript
export default defineNuxtConfig({
  $production: {
    routeRules: { '/**': { isr: true } }
  },
  $development: { /* dev config */ },
  $env: {
    staging: { /* staging config */ }
  }
})
```

### 1.4 Runtime Configuration (Environment Variables)

**Basic Setup:**
1. DEFINE `runtimeConfig` in `nuxt.config` to expose environment variables
2. PLACE private server-side keys directly in `runtimeConfig`
3. PLACE client-side keys in `runtimeConfig.public`
4. OVERRIDE runtime config values using environment variables prefixed with `NUXT_`
5. USE uppercase with underscores for environment variable names (e.g., `NUXT_API_SECRET`)
6. ACCESS runtime config with `useRuntimeConfig()` composable

```typescript
export default defineNuxtConfig({
  runtimeConfig: {
    apiSecret: '123',      // Server-side only
    public: {
      apiBase: '/api'      // Client-side accessible
    }
  }
})
```

**Server Context:**
7. CALL `useRuntimeConfig(event)` in server routes to access config
8. PASS event argument to useRuntimeConfig for environment variable overrides

### 1.5 App Configuration (Build-Time)

**Setup:**
1. CREATE `app.config.ts` in source directory (default: `app/`)
2. USE for public variables determined at build time
3. EXPORT `defineAppConfig` function with configuration object
4. DO NOT import `defineAppConfig` (globally available)
5. CANNOT override app.config values with environment variables
6. ACCESS app config with `useAppConfig()` composable

```typescript
export default defineAppConfig({
  title: 'Hello Nuxt',
  theme: {
    dark: true,
    colors: { primary: '#ff0000' }
  }
})
```

**Selection Guidelines:**
7. USE `runtimeConfig` for private/public tokens that need environment variable override after build
8. USE `app.config` for public tokens, website configuration, and non-sensitive project config

### 1.6 Tool Configuration

**Integrated Tools:**
1. CONFIGURE Nitro using `nitro` key in `nuxt.config`
2. CONFIGURE PostCSS using `postcss` key in `nuxt.config`
3. CONFIGURE Vite using `vite` key in `nuxt.config`
4. CONFIGURE webpack using `webpack` key in `nuxt.config`
5. DO NOT create separate `nitro.config.ts`, `vite.config.ts`, or `webpack.config.ts` files

**External Tools:**
6. CREATE separate config files for: `tsconfig.json`, `eslint.config.js`, `prettier.config.js`, `stylelint.config.js`, `tailwind.config.js`, `vitest.config.ts`

**Vue Configuration (Vite):**
7. PASS options to `@vitejs/plugin-vue` via `vite.vue` key
8. PASS options to `@vitejs/plugin-vue-jsx` via `vite.vueJsx` key

```typescript
export default defineNuxtConfig({
  vite: {
    vue: { customElement: true },
    vueJsx: { mergeProps: true }
  }
})
```

**Vue Configuration (webpack):**
9. CONFIGURE `vue-loader` using `webpack.loaders.vue` key

**Experimental Features:**
10. ENABLE Vue experimental features (e.g., `propsDestructure`) in `vue` key
11. DO NOT use built-in `reactivityTransform` (removed since Nuxt 3.9 and Vue 3.4)
12. USE Vue Macros with Nuxt integration instead for `reactivityTransform` functionality

---

## II. DEVELOPING

### 2.1 Application Structure and Views

**Entry Point:**
1. TREAT `app.vue` as the entrypoint that renders content for every route
2. CREATE `app/pages/index.vue` file to enable pages functionality
3. ADD `<NuxtPage />` component to `app/app.vue` OR remove `app/app.vue` entirely for default entry

**Components:**
4. CREATE components in the `app/components/` directory for automatic import across the application
5. WRITE Vue composables and components in their respective directories without manual imports

**Pages:**
6. CREATE pages in the `app/pages/` directory where each file represents a different route
7. DEFINE routes based on the structure of your `app/pages/` directory for file-based routing

**Layouts:**
8. CREATE layouts in `app/layouts/` directory as Vue files using `<slot />` components to display page content
9. USE `app/layouts/default.vue` as the default layout file
10. WRAP `<NuxtPage />` with `<NuxtLayout>` component in `app.vue` when using layouts:

```vue
<template>
  <div>
    <NuxtLayout>
      <NuxtPage />
    </NuxtLayout>
  </div>
</template>
```

11. PLACE layout files in `app/layouts/` directory
12. INCLUDE `<slot />` component in layout files to render page content

**HTML Template Extension:**
13. EXTEND HTML template by creating a Nitro plugin in `~/server/plugins/` that registers the `render:html` hook:

```typescript
export default defineNitroPlugin((nitroApp) => {
  nitroApp.hooks.hook('render:html', (html, { event }) => {
    html.head.push(`<meta name="description" content="My custom description" />`)
  })
})
```

### 2.2 Assets and Static Files

**Static Assets:**
1. PLACE static assets in the `public/` directory for server root access
2. REFERENCE files in `public/` directory using root URL path starting with `/`:
   ```vue
   <img src="/img/nuxt.png" alt="Discover Nuxt" />
   ```

**Build-Processed Assets:**
3. PLACE build-processed assets (stylesheets, fonts, SVGs) in the `app/assets/` directory
4. REFERENCE files in `app/assets/` directory using the `~/assets/` path:
   ```vue
   <img src="~/assets/img/nuxt.png" alt="Discover Nuxt" />
   ```
5. DO NOT expect auto-scan functionality for the `assets/` directory
6. USE Vite (default) or webpack to process assets in the `assets/` directory
7. CONFIGURE build tools through plugins (Vite) or loaders (webpack) to process non-JavaScript assets

### 2.3 Styling

**Tailwind CSS (Primary Styling Solution):**
1. INSTALL Tailwind module: `pnpm add -D @nuxtjs/tailwindcss`
2. ADD to modules in `nuxt.config.ts`:
   ```typescript
   export default defineNuxtConfig({
     modules: ['@nuxtjs/tailwindcss']
   })
   ```
3. CREATE `tailwind.config.ts` for customization:
   ```typescript
   import type { Config } from 'tailwindcss'
   
   export default {
     content: [],
     theme: {
       extend: {
         colors: {
           primary: '#ff0000'
         }
       }
     }
   } satisfies Config
   ```
4. USE Tailwind utility classes directly in templates:
   ```vue
   <template>
     <div class="container mx-auto px-4">
       <h1 class="text-3xl font-bold text-primary">Hello Nuxt</h1>
     </div>
   </template>
   ```
5. CONFIGURE viewer and exposeConfig for development:
   ```typescript
   export default defineNuxtConfig({
     modules: ['@nuxtjs/tailwindcss'],
     tailwindcss: {
       exposeConfig: true,
       viewer: true
     }
   })
   ```

**Local Stylesheets:**
6. PLACE local stylesheets in the `app/assets/` directory
7. IMPORT stylesheets in components using static JavaScript import for server-side compatibility:
   ```javascript
   import '~/assets/css/first.css'
   ```
8. IMPORT stylesheets using CSS `@import` statement in style blocks:
   ```css
   @import url("~/assets/css/second.css");
   ```

**Global Stylesheets:**
9. ADD global stylesheets to the `css` property in `nuxt.config.ts`:
   ```typescript
   export default defineNuxtConfig({
     css: ['~/assets/css/main.css']
   })
   ```

**Font Management:**
10. PLACE local font files in the `public/fonts/` directory
11. REFERENCE fonts using `url()` with absolute path from public directory:
   ```css
   @font-face {
     font-family: 'FarAwayGalaxy';
     src: url('/fonts/FarAwayGalaxy.woff') format('woff');
     font-weight: normal;
     font-style: normal;
     font-display: swap;
   }
   ```

**Preprocessors:**
12. INSTALL preprocessors before use: `pnpm add -D sass` (or less, stylus)
13. USE `lang` attribute in style blocks for preprocessor syntax:
   ```vue
   <style lang="scss">
   @use "~/assets/scss/main.scss";
   </style>
   ```
14. CONFIGURE Vite preprocessor options in `nuxt.config.ts` for injecting code:
   ```typescript
   export default defineNuxtConfig({
     vite: {
       css: {
         preprocessorOptions: {
           scss: {
             additionalData: '@use "~/assets/_colors.scss" as *;'
           }
         }
       }
     }
   })
   ```
15. ENABLE experimental preprocessor workers in `nuxt.config.ts` for performance:
    ```typescript
    export default defineNuxtConfig({
      vite: { css: { preprocessorMaxWorkers: true } }
    })
    ```

**External Stylesheets:**
16. ADD external stylesheets via `app.head` property in `nuxt.config.ts`:
    ```typescript
    export default defineNuxtConfig({
      app: {
        head: {
          link: [{ rel: 'stylesheet', href: 'https://cdnjs.cloudflare.com/...' }]
        }
      }
    })
    ```
17. USE `useHead()` composable for dynamically adding stylesheets:
    ```typescript
    useHead({
      link: [{ rel: 'stylesheet', href: 'https://cdnjs.cloudflare.com/...' }]
    })
    ```
18. CREATE Nitro plugin in `~/server/plugins/my-plugin.ts` for advanced head control
19. INCLUDE `preconnect` link before external stylesheet links (e.g., Google Fonts)
20. ADD `crossorigin: ''` attribute to external font stylesheet links

**Scoped and Modular Styles:**
21. USE `scoped` attribute for component-isolated styles:
    ```vue
    <style scoped>
    .example { color: red; }
    </style>
    ```
22. USE `module` attribute for CSS Modules with injected `$style` variable:
    ```vue
    <template>
      <p :class="$style.red">This should be red</p>
    </template>
    <style module>
    .red { color: red; }
    </style>
    ```

**PostCSS:**
23. CONFIGURE PostCSS plugins in `nuxt.config.ts`:
    ```typescript
    export default defineNuxtConfig({
      postcss: {
        plugins: {
          'postcss-nested': {},
          'postcss-custom-media': {}
        }
      }
    })
    ```
24. USE `lang="postcss"` attribute for PostCSS syntax highlighting in SFC

**NPM Packages:**
25. REFERENCE npm-distributed stylesheets directly in components or via `css` property:
    ```typescript
    export default defineNuxtConfig({
      css: ['animate.css']
    })
    ```

### 2.4 Routing

**File-Based Routing:**
1. CREATE route files in the `pages/` directory where each Vue file automatically generates a corresponding URL
2. NAME files using bracket syntax `[id].vue` for dynamic route parameters (e.g., `/posts/:id`)
3. ACCESS route parameters using `useRoute()` composable in `<script setup>` blocks
4. WRAP pages in `<NuxtPage />` component in `app.vue` to display routed content

**Navigation:**
5. USE `<NuxtLink>` component with `to` prop for all internal navigation between pages

**Middleware:**
6. PLACE anonymous route middleware directly in page components where they are used
7. CREATE named route middleware files in `app/middleware/` directory (normalized to kebab-case)
8. ADD `.global` suffix to middleware filenames in `app/middleware/` to run on every route change
9. DEFINE middleware in `definePageMeta()` using the `middleware` property
10. USE `defineNuxtRouteMiddleware()` to create middleware functions with `(to, from)` parameters
11. CALL `navigateTo()` function within middleware to redirect users

**Route Validation:**
12. IMPLEMENT route validation using `validate` property in `definePageMeta()` accepting route as argument
13. RETURN `false` from validate function to trigger 404 error
14. RETURN object with `statusCode`/`statusMessage` from validate to customize error responses

**Advanced Routing:**
15. GROUP routes using parentheses syntax `(folder-name)` to avoid affecting URL structure
16. USE `definePageMeta()` to set page metadata accessible via `route.meta` object

### 2.5 SEO and Meta Tags

**Static Configuration:**
1. SET static head configuration in `nuxt.config.ts` using `app.head` property for non-reactive tags
2. DEFINE `title`, `htmlAttrs.lang`, and `link` (favicon) in nuxt.config for global defaults
3. SET `charset` and `viewport` overrides in `app.head` only when different from Nuxt defaults (utf-8, width=device-width)

**Reactive Meta Tags:**
4. USE `useHead()` composable for reactive head tag management in components
5. PASS objects with `title`, `meta`, `bodyAttrs`, and `script` properties to `useHead()`
6. PASS reactive values (ref, computed, getter) to `useHead()` and `useSeoMeta()` for dynamic content

**SEO-Specific:**
7. IMPLEMENT `useSeoMeta()` composable for type-safe SEO meta tag definitions
8. DEFINE SEO properties including `title`, `ogTitle`, `description`, `ogDescription`, `ogImage`, `twitterCard` in `useSeoMeta()`

**Template-Based Meta:**
9. USE capitalized components `<Title>`, `<Meta>`, `<Link>`, `<Head>`, `<Body>`, `<Html>` for template-based meta tags
10. WRAP meta tag components inside `<Head>` or `<Html>` components for proper deduplication

**Title Templates:**
11. SET `titleTemplate` property with `%s` placeholder string or function to customize page titles globally
12. CONFIGURE `titleTemplate` in `app.vue` to apply across all routes (not in nuxt.config for dynamic behavior)
13. PROVIDE `templateParams` object to `useHead()` for additional placeholder values beyond `%s`
14. USE `definePageMeta()` with title property and combine with `useHead()` in layouts to access via `route.meta.title`

**Scripts:**
15. ADD `tagPosition: 'bodyClose'` to script objects to append scripts before closing `</body>` tag

### 2.6 Transitions

**Global Configuration:**
1. ENABLE page transitions globally in `nuxt.config.ts` using `app.pageTransition` with `name` and `mode` properties
2. SET `mode: 'out-in'` for page and layout transitions to prevent simultaneous animations
3. ADD CSS classes following pattern `{name}-enter-active`, `{name}-leave-active`, `{name}-enter-from`, `{name}-leave-to`
4. DEFINE transition styles in `<style>` block of `app.vue` for global transitions
5. WRAP content with `<NuxtPage />` component to enable page transitions

**Layout Transitions:**
6. ENABLE layout transitions globally using `app.layoutTransition` in nuxt.config
7. WRAP `<NuxtPage />` inside `<NuxtLayout>` component to enable layout transitions

**Per-Page Configuration:**
8. CONFIGURE custom page transitions per-page using `definePageMeta({ pageTransition: { name: 'transition-name' } })`
9. RENAME CSS classes to match custom transition names when overriding global settings
10. DISABLE transitions for specific routes by setting `pageTransition: false` or `layoutTransition: false` in `definePageMeta()`

**JavaScript Hooks:**
11. USE JavaScript hooks (`onBeforeEnter`, `onEnter`, `onAfterEnter`) in `pageTransition` object for programmatic animations
12. IMPLEMENT dynamic transitions using inline middleware to modify `to.meta.pageTransition.name` based on conditional logic
13. PASS `:transition` prop to `<NuxtPage />` component in app.vue for component-level transition configuration

**View Transitions API:**
14. ENABLE View Transitions API by setting `experimental.viewTransition: true` in nuxt.config
15. SET global View Transition default using `app.viewTransition` property (true/false/'always')
16. OVERRIDE View Transition per-page using `definePageMeta({ viewTransition: false })`
17. CREATE `~/middleware/disable-vue-transitions.global.ts` to disable Vue transitions when browser supports View Transitions API
18. CHECK `document.startViewTransition` support before disabling Vue transitions

**Best Practices:**
19. APPLY transition properties as JSON-serializable values (name, mode only) in nuxt.config
20. USE `TransitionProps` interface for valid transition configuration options

### 2.7 Data Fetching

**Composable Selection:**
1. USE `useFetch` for SSR-safe network requests in component setup functions
2. USE `useAsyncData` when you need fine-grained control or wrapping custom query layers (e.g., CMS libraries)
3. USE `$fetch` ONLY for client-side event-based interactions (button clicks, form submissions)
4. NEVER use `$fetch` in component setup functions - it causes double fetching and hydration issues

**Keys and Caching:**
5. ALWAYS provide explicit unique keys to `useAsyncData` (e.g., `useAsyncData('user:${id}', ...)`)
6. DO NOT rely on auto-generated keys when creating custom composables
7. USE the same key across components to share data, error, and status refs
8. ENSURE these options are consistent across all calls with the same key: `handler`, `deep`, `transform`, `pick`, `getCachedData`, `default`
9. USE different keys when you need independent data instances

**Server/Client Fetching:**
10. SET `server: false` to perform fetching ONLY on client-side
11. COMBINE `lazy: true` with `server: false` for non-SEO sensitive data
12. USE `useFetch` with relative URLs on server - it automatically proxies headers and cookies via `useRequestFetch`
13. MANUALLY pass headers using `useRequestHeaders(['cookie'])` when using `$fetch` on server

**Loading States:**
14. SET `lazy: true` to prevent blocking navigation with Vue Suspense
15. HANDLE loading states manually with `status` value when using `lazy: true`
16. USE `useLazyFetch` and `useLazyAsyncData` as shortcuts for `lazy: true`
17. CHECK `status` values: "idle", "pending", "success", "error"

**Payload Optimization:**
18. USE `pick` option to select only needed fields and minimize HTML payload size
19. USE `transform` function to map/alter query results
20. UNDERSTAND that `pick` and `transform` don't prevent initial fetching, only payload transfer

**Reactive Data Fetching:**
21. USE `watch` option to refetch when reactive values change
22. USE computed URLs with reactive query parameters for automatic refetching
23. PASS reactive refs as query options: `query: { user_id: id }`
24. USE computed getter functions for complex URL construction: `useFetch(() => `/api/users/${id.value}`)`
25. SET `watch: false` to disable automatic refetching of reactive options

**Manual Control:**
26. USE `refresh()` or `execute()` to manually refetch data
27. USE `clear()` to set data to undefined and reset state
28. SET `immediate: false` to prevent automatic fetching on component mount
29. CALL `execute()` manually when using `immediate: false`

**Parallel Requests:**
30. USE `Promise.all()` inside `useAsyncData` for parallel independent requests
31. WRAP multiple `$fetch` calls in single `useAsyncData` call for better performance

**Data Serialization:**
32. ENSURE data in `useAsyncData`/`useLazyAsyncData` is serializable with `devalue` (supports Date, Map, Set, RegExp, ref, reactive)
33. AVOID classes, functions, or symbols in transferred data
34. USE `toJSON()` method for custom serialization from API routes
35. UNDERSTAND API route responses serialize with `JSON.stringify` (limited to primitives)

**Cookie/Header Handling:**
36. USE `useRequestFetch` (automatic with relative URLs in `useFetch`)
37. USE `useRequestHeaders(['cookie'])` with `$fetch` when manual header passing is needed
38. USE `$fetch.raw()` with `appendResponseHeader` to pass cookies from server API calls to SSR response

**Advanced:**
39. USE `responseType: 'stream'` with `$fetch` for Server-Sent Events via POST
40. USE `ReadableStream` with `TextDecoderStream` to process SSE chunks

### 2.8 State Management

**Pinia (Primary State Management Solution):**

Pinia is the officially recommended state management solution for Vue/Nuxt applications, replacing Vuex. It provides:
- Type-safe stores with full TypeScript support
- Modular store design with automatic code splitting
- Devtools integration for debugging
- SSR support out of the box
- Hot module replacement (HMR) during development

**Installation and Setup:**
1. INSTALL Pinia module: `pnpm add -D @pinia/nuxt pinia`
2. ADD to modules in `nuxt.config.ts`:
   ```typescript
   export default defineNuxtConfig({
     modules: ['@pinia/nuxt']
   })
   ```

**Defining Stores (Option Stores Pattern):**
3. CREATE stores in `stores/` directory using Options API style:
   ```typescript
   // stores/counter.ts
   import { defineStore } from 'pinia'
   
   export const useCounterStore = defineStore('counter', {
     // state: function that returns initial state
     state: () => ({
       count: 0,
       name: 'Eduardo'
     }),
     
     // getters: computed properties for state
     getters: {
       doubleCount: (state) => state.count * 2,
       // getters with access to other getters
       doublePlusOne(): number {
         return this.doubleCount + 1
       }
     },
     
     // actions: methods that can mutate state
     actions: {
       increment() {
         this.count++
       },
       async fetchData() {
         const data = await $fetch('/api/data')
         this.count = data.count
       }
     }
   })
   ```

**Defining Stores (Setup Stores Pattern):**
4. USE composition API style for more flexibility:
   ```typescript
   // stores/counter.ts
   import { defineStore } from 'pinia'
   import { ref, computed } from 'vue'
   
   export const useCounterStore = defineStore('counter', () => {
     // state
     const count = ref(0)
     const name = ref('Eduardo')
     
     // getters
     const doubleCount = computed(() => count.value * 2)
     
     // actions
     function increment() {
       count.value++
     }
     
     async function fetchData() {
       const data = await $fetch('/api/data')
       count.value = data.count
     }
     
     // expose public API
     return { count, name, doubleCount, increment, fetchData }
   })
   ```

**Using Stores in Components:**
5. IMPORT and use stores with auto-imports (no explicit import needed):
   ```vue
   <script setup lang="ts">
   const counter = useCounterStore()
   
   // access state directly
   console.log(counter.count)
   
   // access getters
   console.log(counter.doubleCount)
   
   // call actions
   counter.increment()
   
   // watch state changes
   watch(() => counter.count, (newCount) => {
     console.log('Count changed to:', newCount)
   })
   </script>
   
   <template>
     <div>
       <p>Count: {{ counter.count }}</p>
       <p>Double: {{ counter.doubleCount }}</p>
       <button @click="counter.increment">Increment</button>
     </div>
   </template>
   ```

**Destructuring with Reactivity:**
6. USE `storeToRefs()` to destructure state/getters while maintaining reactivity:
   ```typescript
   import { storeToRefs } from 'pinia'
   
   const counter = useCounterStore()
   
   // destructure state and getters (reactive)
   const { count, doubleCount } = storeToRefs(counter)
   
   // destructure actions (not reactive, plain functions)
   const { increment } = counter
   
   // this works with reactivity preserved
   watch(count, (newValue) => {
     console.log('Count changed:', newValue)
   })
   ```

**SSR and State Hydration:**
7. INITIALIZE store data on server for SSR:
   ```typescript
   // stores/website.ts
   export const useWebsiteStore = defineStore('website', () => {
     const name = ref('')
     const description = ref('')
     const isLoading = ref(false)
     
     async function fetch() {
       isLoading.value = true
       try {
         const infos = await $fetch('/api/website')
         name.value = infos.name
         description.value = infos.description
       } finally {
         isLoading.value = false
       }
     }
     
     return { name, description, isLoading, fetch }
   })
   ```

8. CALL initialization in `app.vue` with `callOnce` for SSR:
   ```vue
   <script setup lang="ts">
   const website = useWebsiteStore()
   
   // runs once on server, hydrates on client
   await callOnce(website.fetch)
   </script>
   ```

**Resetting State:**
9. USE `$reset()` to reset store to initial state (Options API only):
   ```typescript
   const counter = useCounterStore()
   counter.$reset() // resets to initial state
   ```

10. IMPLEMENT custom reset for Setup Stores:
    ```typescript
    export const useCounterStore = defineStore('counter', () => {
      const count = ref(0)
      
      function $reset() {
        count.value = 0
      }
      
      return { count, $reset }
    })
    ```

**Subscribing to State Changes:**
11. USE `$subscribe` to watch entire store state:
    ```typescript
    const counter = useCounterStore()
    
    counter.$subscribe((mutation, state) => {
      // persist to localStorage
      localStorage.setItem('counter', JSON.stringify(state))
    })
    ```

**Subscribing to Actions:**
12. USE `$onAction` to hook into action calls:
    ```typescript
    const counter = useCounterStore()
    
    counter.$onAction(({
      name, // action name
      store, // store instance
      args, // action arguments
      after, // hook after action returns/resolves
      onError // hook if action throws/rejects
    }) => {
      // log when action starts
      console.log(`Action "${name}" started`)
      
      // log when action succeeds
      after((result) => {
        console.log(`Action "${name}" completed with:`, result)
      })
      
      // log if action fails
      onError((error) => {
        console.error(`Action "${name}" failed:`, error)
      })
    })
    ```

**Accessing Other Stores:**
13. ACCESS other stores within a store:
    ```typescript
    import { useUserStore } from './user'
    
    export const useCartStore = defineStore('cart', () => {
      const userStore = useUserStore()
      
      function checkout() {
        if (!userStore.isLoggedIn) {
          throw new Error('Must be logged in to checkout')
        }
        // checkout logic
      }
      
      return { checkout }
    })
    ```

**Store Plugins:**
14. EXTEND Pinia functionality with plugins:
    ```typescript
    // plugins/pinia.ts
    export default defineNuxtPlugin(({ $pinia }) => {
      $pinia.use(({ store }) => {
        // add properties to every store
        store.$myCustomProperty = 'hello'
        
        // persist state to localStorage
        if (process.client) {
          const stored = localStorage.getItem(store.$id)
          if (stored) {
            store.$patch(JSON.parse(stored))
          }
          
          store.$subscribe((mutation, state) => {
            localStorage.setItem(store.$id, JSON.stringify(state))
          })
        }
      })
    })
    ```

**Built-in useState (Simple Cases):**
15. USE `useState` composable for simple SSR-friendly reactive values
16. PREFER Pinia for complex state with actions, getters, and cross-component logic
17. USE `useState` for simple shared refs that don't need actions:
    ```typescript
    // composables/useColor.ts
    export const useColor = () => useState<string>('color', () => 'pink')
    ```

**Critical Rules:**
18. NEVER define `const state = ref()` outside `<script setup>` or `setup()` function
19. NEVER use `export const myState = ref({})` - causes state shared across requests on server
20. ALWAYS use Pinia stores or `useState` for shared state
21. ENSURE store state is JSON-serializable for SSR
22. AVOID classes, functions, or symbols in store state

**Best Practices:**
23. PREFER Pinia for complex state management with multiple components
24. USE Option Stores for simpler, more straightforward state logic
25. USE Setup Stores when you need composition API flexibility
26. DEFINE one store per domain/feature (e.g., `useUserStore`, `useCartStore`)
27. KEEP stores focused and modular - avoid "god stores"
28. USE getters for derived state instead of storing computed values
29. MAKE actions async when dealing with API calls
30. HANDLE errors within actions and expose error state if needed
31. USE TypeScript for type-safe stores with autocomplete
32. TEST stores in isolation - they're just functions/objects

### 2.9 Error Handling

**Error Page Setup:**
1. CREATE error.vue file in source directory (alongside app.vue) to customize the default error page
2. DEFINE error prop as `error: Object as () => NuxtError` in error.vue setup
3. DO NOT place error.vue in ~/pages directory
4. DO NOT use definePageMeta within error.vue
5. USE NuxtLayout component to specify layout name in error.vue if needed
6. ACCESS error fields: statusCode, fatal, unhandled, statusMessage, data, cause
7. ASSIGN custom error fields to data property: `createError({ statusCode, statusMessage, data: { myCustomField: true } })`

**Error Handling Composables & Hooks:**
8. USE onErrorCaptured() lifecycle hook to catch Vue errors in components
9. IMPLEMENT vue:error hook via `nuxtApp.hook('vue:error', (error, instance, info) => {})` in plugins
10. SET vueApp.config.errorHandler in plugin for global error handling
11. CALL app:error hook for startup errors

**Throwing Errors:**
12. THROW errors using `createError({ statusCode, statusMessage, fatal?, data? })`
13. USE createError on server-side to trigger full-screen error page
14. SET `fatal: true` on client-side createError to trigger full-screen error page
15. CALL `showError(err)` to trigger full-screen error page at any point
16. CALL `clearError({ redirect?: string })` to remove error page and optionally navigate
17. USE `useError()` function to return global Nuxt error being handled

**Error Boundaries:**
18. WRAP components with `<NuxtErrorBoundary>` to handle client-side errors locally
19. USE default slot for content in NuxtErrorBoundary
20. USE #error slot with `{ error, clearError }` props to display errors locally
21. ATTACH @error event handler to NuxtErrorBoundary for error logging
22. SET `error = null` to re-render default slot (ensure error is fully resolved first)

**Server Error Handling:**
23. THROW `createError({ statusCode, statusMessage })` in server routes
24. USE createError with short statusMessage for API routes (accessible on client)
25. PASS data via data property in createError for client-side access
26. AVOID putting dynamic user input in error messages (security risk)
27. RETURN 500 Internal Server Error for uncaught errors automatically
28. USE `setResponseStatus(event, statusCode)` to set custom success status codes

**Chunk Loading:**
29. CONFIGURE experimental.emitRouteChunkError to false to disable chunk error handling
30. SET experimental.emitRouteChunkError to 'manual' to handle chunk errors manually
31. DEFAULT behavior performs hard reload when chunk fails to load during route navigation

### 2.10 Server

**Directory Structure:**
1. CREATE server/ directory in project root
2. PLACE API routes in server/api/ directory (automatically prefixed with /api)
3. PLACE server routes without /api prefix in server/routes/ directory
4. PLACE server middleware in server/middleware/ directory (runs on every request)
5. PLACE server plugins in server/plugins/ directory (extends Nitro runtime)
6. PLACE server utilities in server/utils/ directory

**API Route Creation:**
7. EXPORT default function using `defineEventHandler((event) => {})` or `eventHandler()`
8. RETURN JSON data, Promise, text, html, or stream directly from handler
9. ACCESS route with dynamic parameters using `/api/hello/[name].ts` filename pattern
10. USE `getRouterParam(event, 'paramName')` to access route parameters
11. USE `getValidatedRouterParams()` with schema validator for type safety
12. CREATE catch-all routes using `[...].ts` or `[...slug].ts` filename
13. ACCESS catch-all segments via `event.context.params._` or `event.context.params.slug`

**HTTP Method Handling:**
14. SUFFIX filenames with `.get`, `.post`, `.put`, `.delete` to match HTTP methods
15. CREATE method-specific routes: `hello.get.ts`, `hello.post.ts`
16. USE `index.[method].ts` inside directory for namespace organization
17. STRUCTURE as: `api/foo/index.get.ts`, `api/foo/index.post.ts`, `api/foo/bar/index.get.ts`

**Request/Response:**
18. USE `readBody(event)` to parse request body (async)
19. USE `readValidatedBody()` with schema validator for type safety
20. USE `getQuery(event)` to access query parameters
21. USE `getValidatedQuery()` with schema validator for type safety
22. USE `parseCookies(event)` to access request cookies
23. USE `setResponseStatus(event, statusCode)` to set response status
24. USE `sendRedirect(event, path, statusCode)` to send redirects
25. USE `sendStream(event, stream)` to send file streams

**Server Error Handling:**
26. THROW `createError({ statusCode, statusMessage })` for custom errors
27. RETURN 200 OK status by default when no errors thrown
28. RETURN 500 Internal Server Error for uncaught errors automatically

**Server Middleware:**
29. CREATE middleware files in server/middleware/ directory
30. EXPORT defineEventHandler that runs before other server routes
31. USE for logging, header checks, extending event.context

**Server Plugins:**
32. CREATE plugin files in server/plugins/ directory
33. EXPORT `defineNitroPlugin((nitroApp) => {})` to extend Nitro runtime
34. USE to hook into Nitro lifecycle events

**Runtime Config:**
35. CALL `useRuntimeConfig(event)` in server routes to access config
36. PASS event argument to useRuntimeConfig for environment variable overrides

**Advanced Patterns:**
37. USE `event.$fetch()` to forward request context and headers in server routes
38. USE `event.waitUntil(promise)` for background tasks without blocking response
39. CONFIGURE nitro settings via nitro key in nuxt.config
40. MOUNT storage drivers using nitro.storage or server plugins
41. USE `useStorage('mountPoint')` to interact with storage layer
42. CREATE nested routers using `createRouter()` and `useBase()` from h3
43. USE `fromNodeMiddleware()` for legacy handlers (avoid when possible)

**Route Rules:**
44. DEFINE routeRules in nuxt.config for hybrid rendering
45. SET `prerender: true` for static generation at build time
46. SET `cache: { maxAge: seconds }` for caching routes
47. SET `redirect: { to: '/path', statusCode: 302 }` for redirects

### 2.11 Layers

**Auto-Scanning:**
1. PLACE layers in ~~/layers directory for automatic registration
2. ACCESS auto-scanned layers via #layers/layerName alias
3. UNDERSTAND alphabetical priority: Z overrides A
4. UNDERSTAND overall priority: extends layers > auto-scanned > your project

**Extending Layers:**
5. ADD extends array to nuxt.config.ts to extend from layers
6. EXTEND from local layer: `extends: ['../base']`
7. EXTEND from npm package: `extends: ['@my-themes/awesome']` or `['moduleName']`
8. EXTEND from git: `extends: ['github:user/repo', 'github:user/repo#v1.0.0', 'gitlab:user/repo', 'bitbucket:user/repo']`
9. EXTEND from subdirectory: `extends: ['github:user/repo/base']`
10. EXTEND from branch: `extends: ['github:user/repo#dev']`
11. PASS authentication token: `extends: [['github:user/private-repo', { auth: process.env.GITHUB_TOKEN }]]`
12. CONFIGURE layer with meta: `extends: [['github:user/repo', { meta: { name: 'my-theme' } }]]`

**Priority Order:**
13. UNDERSTAND extends priority: first in array overrides later entries
14. UNDERSTAND complete override order: base > theme > custom > z > a > your project

**Creating Layers:**
15. INITIALIZE layer using: `pnpm create nuxt -- --template layer nuxt-layer`
16. CREATE nuxt.config.ts in layer root (required minimum)
17. PLACE components in components/ directory to extend default components
18. PLACE composables in composables/ directory to extend default composables
19. PLACE layouts in layouts/ directory to extend default layouts
20. PLACE pages in pages/ directory to extend default pages
21. PLACE plugins in plugins/ directory to extend default plugins
22. PLACE server endpoints in server/ directory to extend server functionality
23. PLACE utilities in utils/ directory to extend default utils
24. CONFIGURE nuxt settings in nuxt.config.ts to extend configuration
25. CONFIGURE app settings in app.config.ts to extend app config

**Publishing Layers:**
26. PUBLISH to git repository for remote source sharing
27. SET `GIGET_AUTH=<token>` environment variable for private repos
28. ENABLE `install: true` in layer options to install npm dependencies from git layers
29. PUBLISH to npm with correct package.json properties
30. SET package.json main field to './nuxt.config.ts'
31. SET package.json type to 'module'
32. ADD layer dependencies to dependencies field in package.json
33. KEEP nuxt dependency in devDependencies field
34. INSTALL as devDependency in consuming projects

**Layer Configuration:**
35. SET $meta.name in layer's nuxt.config to create named alias
36. ACCESS named layers via #layers/layerName alias
37. USE relative paths for imports within layer components
38. USE fileURLToPath and dirname for absolute paths in layer nuxt.config
39. RESOLVE paths using: `join(dirname(fileURLToPath(import.meta.url)), './path')`
40. AVOID global aliases (~/, @/) in layer components (resolve relative to user project)

**Multi-Layer Module:**
41. ITERATE nuxt.options._layers array in modules for custom multi-layer handling
42. UNDERSTAND _layers priority: earlier items override later ones, user project is first
43. ACCESS layer.cwd and layer.config properties for each layer

---

## III. TESTING

### 3.1 Test Setup and Configuration

**Installation (Default Test Stack):**
1. INSTALL testing dependencies: `pnpm add -D @nuxt/test-utils vitest @vue/test-utils happy-dom playwright-core`
2. USE this exact stack for all Nuxt projects: @nuxt/test-utils + vitest + happy-dom + playwright
3. ENSURE `"type": "module"` exists in `package.json` OR rename vitest config to `vitest.config.m{ts,js}`

**Module Integration:**
4. ADD `@nuxt/test-utils/module` to `modules` array in `nuxt.config.ts` (optional - adds Vitest integration to Nuxt DevTools)

**Vitest Configuration (Project-Based):**
5. CREATE `vitest.config.ts` with project-based configuration:

```typescript
import { defineConfig } from 'vitest/config'
import { defineVitestProject } from '@nuxt/test-utils/config'

export default defineConfig({
  test: {
    projects: [
      {
        test: {
          name: 'unit',
          include: ['test/{e2e,unit}/*.{test,spec}.ts'],
          environment: 'node',
        },
      },
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['test/nuxt/*.{test,spec}.ts'],
          environment: 'nuxt',
        },
      }),
    ],
  },
})
```

**Simple Configuration:**
6. CREATE `vitest.config.ts` with simple configuration for all tests in Nuxt environment:

```typescript
import { defineVitestConfig } from '@nuxt/test-utils/config'

export default defineVitestConfig({
  test: {
    environment: 'nuxt',
  },
})
```

7. OPT OUT of Nuxt environment per file using comment: `// @vitest-environment node`

**Test Organization:**
8. CREATE directory structure:
   ```
   test/
   ├── e2e/
   │   └── ssr.test.ts
   ├── nuxt/
   │   ├── components.test.ts
   │   └── composables.test.ts
   ├── unit/
   │   └── utils.test.ts
   ```
9. PLACE regular unit tests in `test/unit/` (run in Node environment)
10. PLACE tests requiring Nuxt runtime in `test/nuxt/` (run in Nuxt environment)
11. PLACE end-to-end tests in `test/e2e/`
12. NAME runtime unit test files with `.nuxt.spec.ts` extension to separate from e2e tests

**Environment and Mocks:**
13. USE `.env.test` file to set environment variables for testing
14. CONFIGURE mocks in `vitest.config.ts`:
    ```typescript
    export default defineVitestConfig({
      test: {
        environmentOptions: {
          nuxt: {
            mock: {
              intersectionObserver: true,  // Default: true
              indexedDb: true,              // Default: false
            },
          },
        },
      },
    })
    ```

**Package.json:**
15. ADD test script to `package.json`: `"scripts": { "test": "vitest" }`

### 3.2 Unit Testing

**Running Tests:**
1. RUN all tests: `pnpm test`
2. RUN only unit tests: `pnpm test --project unit`
3. RUN only Nuxt tests: `pnpm test --project nuxt`
4. RUN tests in watch mode: `pnpm test --watch`

**Component Testing with mountSuspended:**
5. IMPORT `mountSuspended` from `@nuxt/test-utils/runtime`
6. MOUNT components with async setup and Nuxt plugin access:
   ```typescript
   import { mountSuspended } from '@nuxt/test-utils/runtime'
   import { SomeComponent } from '#components'
   
   it('can mount some component', async () => {
     const component = await mountSuspended(SomeComponent)
     expect(component.text()).toMatchInlineSnapshot('"This is an auto-imported component"')
   })
   ```
7. PASS route option when mounting app: `await mountSuspended(App, { route: '/test' })`
8. USE all options from `@vue/test-utils` `mount` function

**Component Testing with renderSuspended:**
9. INSTALL `@testing-library/vue` as dev dependency: `pnpm add -D @testing-library/vue`
10. ENABLE testing globals in Vitest config
11. IMPORT `renderSuspended` from `@nuxt/test-utils/runtime`
12. USE with Testing Library utilities (`screen`, `fireEvent`):
    ```typescript
    import { renderSuspended } from '@nuxt/test-utils/runtime'
    import { screen } from '@testing-library/vue'
    
    it('can render some component', async () => {
      await renderSuspended(SomeComponent)
      expect(screen.getByText('This is an auto-imported component')).toBeDefined()
    })
    ```
13. NOTE: Component renders inside `<div id="test-wrapper"></div>`

**Mocking Nuxt Auto-Imports:**
14. IMPORT `mockNuxtImport` from `@nuxt/test-utils/runtime`
15. MOCK composables before tests:
    ```typescript
    import { mockNuxtImport } from '@nuxt/test-utils/runtime'
    
    mockNuxtImport('useStorage', () => {
      return () => {
        return { value: 'mocked storage' }
      }
    })
    ```
16. USE once per mocked import per test file (macro transformed to `vi.mock`)
17. PROVIDE different implementations using `vi.hoisted`:
    ```typescript
    import { vi } from 'vitest'
    const { useStorageMock } = vi.hoisted(() => {
      return {
        useStorageMock: vi.fn(() => ({ value: 'mocked storage' })),
      }
    })
    
    mockNuxtImport('useStorage', () => useStorageMock)
    
    // In test:
    useStorageMock.mockImplementation(() => ({ value: 'something else' }))
    ```
18. RESTORE mocks before or after each test

**Mocking Components:**
19. IMPORT `mockComponent` from `@nuxt/test-utils/runtime`
20. MOCK by PascalCase name:
    ```typescript
    import { mockComponent } from '@nuxt/test-utils/runtime'
    
    mockComponent('MyComponent', {
      props: { value: String },
      setup(props) { /* ... */ },
    })
    ```
21. MOCK by relative path or alias: `mockComponent('~/components/my-component.vue', ...)`
22. REDIRECT to mock SFC component: `mockComponent('MyComponent', () => import('./MockComponent.vue'))`
23. IMPORT Vue APIs inside factory function (cannot reference local variables)

**Mocking API Endpoints:**
24. IMPORT `registerEndpoint` from `@nuxt/test-utils/runtime`
25. REGISTER GET endpoint:
    ```typescript
    import { registerEndpoint } from '@nuxt/test-utils/runtime'
    
    registerEndpoint('/test/', () => ({
      test: 'test-field',
    }))
    ```
26. REGISTER with specific HTTP method:
    ```typescript
    registerEndpoint('/test/', {
      method: 'POST',
      handler: () => ({ test: 'test-field' }),
    })
    ```
27. USE `baseURL` and Nuxt Environment Override Config to redirect external API requests to Nitro server

**Standalone Setup (Without Nuxt):**
28. INSTALL `vitest @vue/test-utils happy-dom @vitejs/plugin-vue` as dev dependencies: `pnpm add -D vitest @vue/test-utils happy-dom @vitejs/plugin-vue`
29. CREATE `vitest.config.ts`:
    ```typescript
    import { defineConfig } from 'vitest/config'
    import vue from '@vitejs/plugin-vue'
    
    export default defineConfig({
      plugins: [vue()],
      test: { environment: 'happy-dom' },
    })
    ```
30. ADD test script: `"test": "vitest"`
31. CREATE test files with standard Vue Test Utils syntax

**Important Constraints:**
32. DO NOT mutate global state in tests (or reset it afterwards)
33. CANNOT use `@nuxt/test-utils/runtime` and `@nuxt/test-utils/e2e` in same file
34. SPLIT unit and e2e tests into separate files
35. NOTE: Global Nuxt app and `nuxtApp` are not initialized when opting out of Nuxt environment

### 3.3 End-to-End Testing

**Basic E2E Setup:**
1. IMPORT setup utilities: `import { describe, test } from 'vitest'` and `import { setup } from '@nuxt/test-utils/e2e'`
2. CALL `setup()` inside each describe block before tests:
   ```typescript
   import { describe, test } from 'vitest'
   import { $fetch, setup } from '@nuxt/test-utils/e2e'
   
   describe('My test', async () => {
     await setup({
       // test context options
     })
     
     test('my test', () => {
       // ...
     })
   })
   ```

**Setup Options:**
3. SET `rootDir`: Path to Nuxt app directory (default: `'.'`)
4. SET `configFile`: Name of config file (default: `'nuxt.config'`)
5. SET `setupTimeout`: Time in ms for setup (default: `120000` or `240000` on Windows)
6. SET `teardownTimeout`: Time in ms for teardown (default: `30000`)
7. SET `build: true` to run separate build step (default: `true`)
8. SET `server: true` to launch test server (default: `true`)
9. SET `port`: Specific port number (default: `undefined`)
10. SET `host`: URL of deployed app or running server to test against (default: `undefined`)
11. SET `browser: true` to launch Playwright browser (default: `false`)
12. CONFIGURE `browserOptions`: `{ type: 'chromium' | 'firefox' | 'webkit', launch: { /* Playwright options */ } }`
13. SET `runner`: Test runner choice - `'vitest'`, `'jest'`, or `'cucumber'` (default: `'vitest'`)

**Testing Against Target Host:**
14. PROVIDE `host` option to test against deployed or separate local server:
    ```typescript
    await setup({
      host: 'http://localhost:8787',
    })
    ```

**E2E Testing APIs:**
15. IMPORT `$fetch` from `@nuxt/test-utils/e2e` to get server-rendered HTML:
    ```typescript
    const html = await $fetch('/')
    ```
16. IMPORT `fetch` from `@nuxt/test-utils/e2e` to get full response:
    ```typescript
    const res = await fetch('/')
    const { body, headers } = res
    ```
17. IMPORT `url` from `@nuxt/test-utils/e2e` to get full URL with test server port:
    ```typescript
    const pageUrl = url('/page')
    // Returns: 'http://localhost:6840/page'
    ```

**Browser Testing with createPage:**
18. IMPORT `createPage` from `@nuxt/test-utils/e2e`
19. CREATE Playwright browser instance:
    ```typescript
    const page = await createPage('/page')
    // Access all Playwright APIs from the page variable
    ```
20. USE within `vitest`, `jest`, or `cucumber` test runners
21. REFERENCE Playwright documentation for available API methods

**Playwright Test Runner:**
22. INSTALL `@playwright/test @nuxt/test-utils` as dev dependencies: `pnpm add -D @playwright/test @nuxt/test-utils`
23. CREATE `playwright.config.ts`:
    ```typescript
    import { fileURLToPath } from 'node:url'
    import { defineConfig, devices } from '@playwright/test'
    import type { ConfigOptions } from '@nuxt/test-utils/playwright'
    
    export default defineConfig<ConfigOptions>({
      use: {
        nuxt: {
          rootDir: fileURLToPath(new URL('.', import.meta.url)),
        },
      },
      // ...
    })
    ```
24. IMPORT from Nuxt test utils: `import { expect, test } from '@nuxt/test-utils/playwright'`
25. WRITE tests using `goto` fixture:
    ```typescript
    test('test', async ({ page, goto }) => {
      await goto('/', { waitUntil: 'hydration' })
      await expect(page.getByRole('heading')).toHaveText('Welcome to Playwright!')
    })
    ```
26. CONFIGURE Nuxt server per-test if needed with `test.use({ nuxt: { rootDir: ... } })`

**Commands:**
27. RUN tests via Nuxt CLI: `pnpm nuxt test [ROOTDIR] [--cwd=<directory>] [--logLevel=<silent|info|verbose>] [--dev] [--watch]`
28. NOTE: Command sets `process.env.NODE_ENV` to `test` if not already set

---

## IV. CI/CD

### 4.1 Docker and Self-Hosted Solutions (Primary Deployment)

**Dockerfile Setup:**
1. CREATE `Dockerfile` in project root:
   ```dockerfile
   # Build stage
   FROM node:20-alpine AS builder
   
   # Install pnpm
   RUN corepack enable && corepack prepare pnpm@latest --activate
   
   WORKDIR /app
   
   # Copy package files
   COPY package.json pnpm-lock.yaml ./
   
   # Install dependencies
   RUN pnpm install --frozen-lockfile
   
   # Copy source code
   COPY . .
   
   # Build application
   RUN pnpm build
   
   # Production stage
   FROM node:20-alpine
   
   WORKDIR /app
   
   # Copy built application
   COPY --from=builder /app/.output ./
   
   # Expose port
   EXPOSE 3000
   
   # Set environment to production
   ENV NODE_ENV=production
   
   # Start application
   CMD ["node", "server/index.mjs"]
   ```

**Docker Compose Configuration:**
2. CREATE `docker-compose.yml` for local development and testing:
   ```yaml
   version: '3.8'
   
   services:
     nuxt-app:
       build:
         context: .
         dockerfile: Dockerfile
       ports:
         - "3000:3000"
       environment:
         - NODE_ENV=production
         - NITRO_PORT=3000
         - NITRO_HOST=0.0.0.0
       restart: unless-stopped
       healthcheck:
         test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:3000"]
         interval: 30s
         timeout: 10s
         retries: 3
         start_period: 40s
   ```

3. CREATE `docker-compose.dev.yml` for development with hot reload:
   ```yaml
   version: '3.8'
   
   services:
     nuxt-dev:
       image: node:20-alpine
       working_dir: /app
       command: sh -c "corepack enable && corepack prepare pnpm@latest --activate && pnpm install && pnpm dev"
       ports:
         - "3000:3000"
       volumes:
         - .:/app
         - /app/node_modules
         - /app/.nuxt
       environment:
         - NODE_ENV=development
   ```

**Docker Compose for Testing:**
4. CREATE `docker-compose.test.yml` for running tests in isolated environment:
   ```yaml
   version: '3.8'
   
   services:
     nuxt-test:
       build:
         context: .
         dockerfile: Dockerfile.test
       command: pnpm test
       environment:
         - NODE_ENV=test
       volumes:
         - ./test-results:/app/test-results
   ```

5. CREATE `Dockerfile.test` for test environment:
   ```dockerfile
   FROM node:20-alpine
   
   RUN corepack enable && corepack prepare pnpm@latest --activate
   
   WORKDIR /app
   
   COPY package.json pnpm-lock.yaml ./
   RUN pnpm install --frozen-lockfile
   
   COPY . .
   
   # Install playwright browsers
   RUN pnpm exec playwright install --with-deps chromium
   
   CMD ["pnpm", "test"]
   ```

**Docker Commands:**
6. BUILD production image: `docker build -t nuxt-app .`
7. RUN production container: `docker run -p 3000:3000 nuxt-app`
8. START with Docker Compose: `docker-compose up -d`
9. START development environment: `docker-compose -f docker-compose.dev.yml up`
10. RUN tests in Docker: `docker-compose -f docker-compose.test.yml up --abort-on-container-exit`
11. VIEW logs: `docker-compose logs -f nuxt-app`
12. STOP containers: `docker-compose down`

**CI/CD Pipeline with Docker:**
13. CREATE `.github/workflows/ci.yml` for GitHub Actions (or equivalent for GitLab CI, Jenkins):
    ```yaml
    name: CI/CD Pipeline
    
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
          
          - name: Set up Docker Buildx
            uses: docker/setup-buildx-action@v3
          
          - name: Run tests
            run: docker-compose -f docker-compose.test.yml up --abort-on-container-exit
          
          - name: Upload test results
            if: always()
            uses: actions/upload-artifact@v4
            with:
              name: test-results
              path: test-results/
      
      build:
        needs: test
        runs-on: ubuntu-latest
        if: github.ref == 'refs/heads/main'
        steps:
          - uses: actions/checkout@v4
          
          - name: Build Docker image
            run: docker build -t nuxt-app:${{ github.sha }} .
          
          - name: Push to registry
            run: |
              echo "${{ secrets.DOCKER_PASSWORD }}" | docker login -u "${{ secrets.DOCKER_USERNAME }}" --password-stdin
              docker tag nuxt-app:${{ github.sha }} your-registry/nuxt-app:latest
              docker push your-registry/nuxt-app:latest
    ```

**Self-Hosted Deployment:**
14. USE Docker Compose on VPS/dedicated server
15. CREATE production `docker-compose.prod.yml`:
    ```yaml
    version: '3.8'
    
    services:
      nuxt-app:
        image: your-registry/nuxt-app:latest
        ports:
          - "3000:3000"
        environment:
          - NODE_ENV=production
          - NUXT_PUBLIC_API_BASE=${API_BASE_URL}
        env_file:
          - .env.production
        restart: always
        networks:
          - web
    
      nginx:
        image: nginx:alpine
        ports:
          - "80:80"
          - "443:443"
        volumes:
          - ./nginx.conf:/etc/nginx/nginx.conf:ro
          - ./ssl:/etc/nginx/ssl:ro
        depends_on:
          - nuxt-app
        restart: always
        networks:
          - web
    
    networks:
      web:
        driver: bridge
    ```

16. DEPLOY to server:
    ```bash
    # On server
    docker-compose -f docker-compose.prod.yml pull
    docker-compose -f docker-compose.prod.yml up -d
    ```

**Environment Variables:**
17. CREATE `.env.production` file for production secrets (DO NOT commit to git)
18. USE Docker secrets or environment variables for sensitive configuration
19. MOUNT `.env` files as secrets in docker-compose for better security:
    ```yaml
    secrets:
      env_file:
        file: .env.production
    services:
      nuxt-app:
        secrets:
          - env_file
    ```

**Monitoring and Health Checks:**
20. IMPLEMENT health check endpoint in `server/api/health.ts`:
    ```typescript
    export default defineEventHandler(() => {
      return { status: 'ok', timestamp: new Date().toISOString() }
    })
    ```
21. CONFIGURE Docker health checks as shown in docker-compose examples
22. USE monitoring tools (Prometheus, Grafana) with Docker labels for metrics

### 4.2 Prerendering

### 4.2 Prerendering

**Build Commands:**
1. RUN `pnpm nuxt generate` to build and pre-render application using Nitro crawler
2. RUN `pnpm nuxt build --prerender` as alternative to `nuxt generate`
3. DEPLOY `.output/public` directory to static hosting
4. PREVIEW locally with `pnpm dlx serve .output/public`

**Basic Configuration:**
5. CONFIGURE crawl-based prerendering in `nuxt.config.ts`:
   ```js
   export default defineNuxtConfig({
     nitro: {
       prerender: {
         crawlLinks: true,
         routes: ['/sitemap.xml', '/robots.txt'],
         ignore: ['/dynamic', '/admin']
       }
     }
   })
   ```

**Route-Specific:**
6. SET prerendering per route using `routeRules`:
   ```js
   routeRules: {
     '/rss.xml': { prerender: true },
     '/this-DOES-NOT-get-prerendered': { prerender: false },
     '/blog/**': { prerender: true }
   }
   ```

**Page-Level:**
7. ENABLE `experimental.inlineRouteRules` in `nuxt.config.ts`
8. USE `defineRouteRules({ prerender: true })` in `<script setup>` blocks

**Runtime Configuration:**
9. CALL `prerenderRoutes(['/some/other/url'])` within Nuxt context to add routes dynamically
10. USE `prerender:routes` hook to register additional routes from CMS/API:
    ```js
    hooks: {
      async 'prerender:routes'(ctx) {
        const { pages } = await fetch('https://api.some-cms.com/pages').then(res => res.json())
        for (const page of pages) {
          ctx.routes.add(`/${page.name}`)
        }
      }
    }
    ```

**Fine-Grained Control:**
11. USE `prerender:generate` Nitro hook to skip specific routes:
    ```js
    nitro: {
      hooks: {
        'prerender:generate'(route) {
          if (route.route?.includes('private')) {
            route.skip = true
          }
        }
      }
    }
    ```

### 4.3 Deployment

**Node.js Server:**
1. BUILD with `pnpm nuxt build` for Node.js server preset
2. RUN production server with `node .output/server/index.mjs`
3. SET `NITRO_PORT` or `PORT` environment variable (defaults to 3000)
4. SET `NITRO_HOST` or `HOST` environment variable (defaults to '0.0.0.0')
5. SET `NITRO_SSL_CERT` and `NITRO_SSL_KEY` for HTTPS mode (testing only)

**PM2 Deployment:**
6. CREATE `ecosystem.config.cjs`:
   ```js
   module.exports = {
     apps: [{
       name: 'NuxtAppName',
       port: '3000',
       exec_mode: 'cluster',
       instances: 'max',
       script: './.output/server/index.mjs'
     }]
   }
   ```

**Cluster Mode:**
7. SET `NITRO_PRESET=node_cluster` to leverage multi-process performance with Node.js cluster module

**Static Site Generation (SSG):**
8. RUN `nuxt generate` with `ssr: true` (default) to pre-render routes at build time
9. GENERATES `/200.html` and `/404.html` fallback pages for dynamic routes
10. CONFIGURE on static host to serve fallback pages

**Static Single-Page App:**
11. SET `ssr: false` in `nuxt.config.ts`:
    ```js
    export default defineNuxtConfig({
      ssr: false
    })
    ```
12. RUN `nuxt generate` to output `.output/public/index.html` entrypoint
13. USE `<ClientOnly>` to wrap non-server-renderable portions

**Preset Configuration:**
14. EXPLICITLY set preset in `nuxt.config.ts`:
    ```js
    export default defineNuxtConfig({
      nitro: {
        preset: 'node-server'
      }
    })
    ```
15. OR SET `NITRO_PRESET` environment variable: `NITRO_PRESET=node-server nuxt build`

**Cloudflare CDN:**
16. DISABLE "Rocket Loader™" in Speed > Optimization > Content Optimization
17. DISABLE "Mirage" in Speed > Optimization > Image Optimization
18. DISABLE "Email Address Obfuscation" in Scrape Shield

### 4.4 Upgrade Processes

**Upgrade to Latest Release:**
1. RUN `pnpm dlx nuxt upgrade` 
2. USE `pnpm dlx nuxt upgrade --force` to remove node_modules and lock files before upgrade

**Upgrade to Nuxt 4:**
3. INSTALL Nuxt 4: `pnpm add nuxt@^4.0.0`

**Automated Migration:**
4. RUN `pnpm dlx codemod@latest nuxt/4/migration-recipe` to execute all codemods
5. PROVIDE feedback with `pnpm dlx codemod feedback`

**Directory Structure Migration:**
6. CREATE `app/` directory in project root
7. MOVE `assets/`, `components/`, `composables/`, `layouts/`, `middleware/`, `pages/`, `plugins/`, `utils/`, `app.vue`, `error.vue`, `app.config.ts` into `app/` folder
8. KEEP `nuxt.config.ts`, `content/`, `layers/`, `modules/`, `public/`, `server/` in root directory
9. UPDATE third-party config files (tailwindcss, eslint) if needed
10. RUN `pnpm dlx codemod@latest nuxt/4/file-structure` for automated migration
11. OR REVERT to v3 structure with:
    ```js
    export default defineNuxtConfig({
      srcDir: '.',
      dir: { app: 'app' }
    })
    ```

**Data Fetching Migration:**
12. REVIEW components using same `useAsyncData` key with different options
13. EXTRACT shared `useAsyncData` calls into composables
14. UPDATE `getCachedData` implementations to handle context parameter:
    ```js
    getCachedData: (key, nuxtApp, ctx) => {
      if (ctx.cause === 'refresh:manual') return undefined
      return cachedData[key]
    }
    ```
15. OR DISABLE with:
    ```js
    export default defineNuxtConfig({
      experimental: {
        granularCachedData: false,
        purgeCachedData: false
      }
    })
    ```

**Route Metadata:**
16. REPLACE `route.meta.name` with `route.name`
17. RUN `pnpm dlx codemod@latest nuxt/4/route-metadata` for automated migration

**Component Names:**
18. UPDATE component names in tests using `findComponent` from @vue/test-utils
19. UPDATE `<KeepAlive>` component name references
20. OR DISABLE with:
    ```js
    export default defineNuxtConfig({
      experimental: { normalizeComponentNames: false }
    })
    ```

**Unhead v2 Migration:**
21. REMOVE deprecated props: `vmid`, `hid`, `children`, `body`
22. UPDATE meta tags:
    ```js
    useHead({
      meta: [{
        name: 'description',
        // Remove vmid and hid
      }]
    })
    ```
23. EXPLICITLY opt-in to Template Params or Alias Tag Sorting if needed
24. REVERT to v1 with `unhead: { legacy: true }` if issues persist

**Generate Configuration:**
25. REPLACE `generate.exclude` with `nitro.prerender.ignore`
26. REPLACE `generate.routes` with `nitro.prerender.routes`:
    ```js
    export default defineNuxtConfig({
      nitro: {
        prerender: {
          ignore: ['/admin', '/private'],
          routes: ['/sitemap.xml', '/robots.txt']
        }
      }
    })
    ```

**useAsyncData/useFetch Changes:**
27. UPDATE null checks to undefined checks for `data.value` and `error.value`
28. RUN `pnpm dlx codemod@latest nuxt/4/default-data-error-value`
29. REPLACE deprecated dedupe boolean values:
    ```js
    await refresh({ dedupe: 'cancel' })  // was: dedupe: true
    await refresh({ dedupe: 'defer' })   // was: dedupe: false
    ```
30. RUN `pnpm dlx codemod@latest nuxt/4/deprecated-dedupe-value`
31. OPT-IN to deep reactivity if needed:
    ```js
    const { data } = useFetch('/api/test', { deep: true })
    ```
32. RUN `pnpm dlx codemod@latest nuxt/4/shallow-function-reactivity`

**TypeScript Configuration:**
33. UPDATE root `tsconfig.json` to use project references (optional):
    ```json
    {
      "files": [],
      "references": [
        { "path": "./.nuxt/tsconfig.app.json" },
        { "path": "./.nuxt/tsconfig.server.json" },
        { "path": "./.nuxt/tsconfig.shared.json" },
        { "path": "./.nuxt/tsconfig.node.json" }
      ]
    }
    ```
34. UPDATE typecheck script: `"typecheck": "nuxt prepare && vue-tsc -b --noEmit"`
35. MOVE type augmentations into appropriate context folders (app/, server/, shared/)

**Miscellaneous:**
36. REPLACE `window.__NUXT__` with `useNuxtApp().payload`
37. REPLACE `pages:extend` hook with `pages:resolved` for overriding page metadata
38. READ nightly release channel guide for testing pre-release features