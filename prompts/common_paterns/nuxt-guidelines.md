# Nuxt 4 Fullstack Development Guide

## Core Philosophy

Nuxt 4 is a fullstack framework with clear separation between app (client/universal) and server code. Embrace auto-imports, file-based conventions, and TypeScript-first development for maximum productivity.

## Directory Structure

```
my-nuxt-app/
├── app/                    # client & universal code
│   ├── assets/            # processed assets (images, styles)
│   ├── components/        # auto-imported vue components
│   ├── composables/       # auto-imported composition api functions
│   ├── layouts/           # page layout wrappers
│   ├── middleware/        # route middleware
│   ├── pages/             # file-based routing
│   ├── plugins/           # app plugins
│   ├── utils/             # auto-imported utility functions
│   ├── app.vue            # root component
│   ├── app.config.ts      # runtime app configuration
│   └── error.vue          # error page
├── server/                # server-only code
│   ├── api/               # api routes (prefixed with /api)
│   ├── routes/            # server routes (no /api prefix)
│   ├── middleware/        # server middleware (runs on every request)
│   ├── plugins/           # nitro plugins
│   └── utils/             # auto-imported server utilities
├── shared/                # code shared between app & server
│   ├── utils/             # auto-imported shared utilities
│   └── types/             # auto-imported shared types
├── public/                # static assets (served as-is)
├── content/               # content files (with @nuxt/content)
├── .nuxt/                 # generated files (gitignore)
├── .output/               # production build output
└── nuxt.config.ts         # nuxt configuration
```

## Code Style and Structure

### General Principles

- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns; avoid classes
- Favor iteration and modularization over code duplication
- Use descriptive variable names with auxiliary verbs (e.g., `isLoading`, `hasError`)
- Use lowercase with dashes for directory names (e.g., `components/auth-wizard`)
- Use lowercase comments and log messages

### File Organization

Structure files with this order:
1. Imports (auto-imports don't need explicit import statements)
2. Types and interfaces
3. Composables and reactive state
4. Functions and computed properties
5. Lifecycle hooks
6. Template (in .vue files)

## Auto-Imports

### What Gets Auto-Imported

**From app/ directory:**
- `app/components/` → Vue components (with path-based naming)
- `app/composables/` → Composables (top-level files only)
- `app/utils/` → Utility functions (top-level files only)

**From server/ directory:**
- `server/utils/` → Server utilities (only available in server context)

**From shared/ directory:**
- `shared/utils/` → Shared utilities (available in both app & server)
- `shared/types/` → Shared types (available in both app & server)

**Built-in:**
- Vue APIs: `ref`, `computed`, `watch`, `onMounted`, etc.
- Nuxt composables: `useFetch`, `useAsyncData`, `useRoute`, `useRouter`, etc.
- H3 utilities (server): `defineEventHandler`, `readBody`, `getQuery`, etc.

### Auto-Import Best Practices

```typescript
// ✅ good: leverage auto-imports
const count = ref(0)
const doubled = computed(() => count.value * 2)
const route = useRoute()

// ❌ avoid: unnecessary explicit imports (unless needed for clarity)
import { ref, computed } from 'vue'
import { useRoute } from '#app'

// ✅ good: explicit import when needed for clarity or to avoid naming conflicts
import { formatDate } from '#imports'

// ✅ good: nested composables need re-export or explicit import
// app/composables/index.ts
export { useAuth } from './auth/useAuth'

// ✅ good: types can be auto-imported from shared/types/
// shared/types/user.ts
export interface User {
  id: string
  name: string
}

// usage (no import needed)
const user = ref<User>()
```

### Preventing Client/Server Import Issues

```typescript
// ❌ avoid: using client-only utils in server
// server/api/data.ts
export default defineEventHandler(() => {
  const data = useLodash() // error: lodash is client-only
})

// ✅ good: use shared/ for code needed in both contexts
// shared/utils/format.ts
export const formatCurrency = (amount: number) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(amount)
}

// works in both app/ and server/
```

## API Routes & Server Code

### File Naming Convention (Dot Method)

Use HTTP method suffixes for endpoints:

```
server/
├── api/
│   ├── posts.get.ts              # GET /api/posts
│   ├── posts.post.ts             # POST /api/posts
│   ├── posts/
│   │   ├── [id].get.ts           # GET /api/posts/:id
│   │   ├── [id].patch.ts         # PATCH /api/posts/:id
│   │   ├── [id].delete.ts        # DELETE /api/posts/:id
│   │   └── [id]/
│   │       └── comments.get.ts   # GET /api/posts/:id/comments
│   └── users/
│       ├── index.get.ts          # GET /api/users
│       ├── index.post.ts         # POST /api/users
│       └── [id]/
│           ├── index.get.ts      # GET /api/users/:id
│           └── profile.get.ts    # GET /api/users/:id/profile
```

### Server Route Best Practices

```typescript
// server/api/posts.get.ts
export default defineEventHandler(async (event) => {
  // validate query params
  const query = await getValidatedQuery(event, z.object({
    page: z.coerce.number().positive().default(1),
    limit: z.coerce.number().positive().max(100).default(10)
  }).parse)

  // fetch data
  const posts = await fetchPosts(query)

  // return json automatically
  return posts
})

// server/api/posts.post.ts
export default defineEventHandler(async (event) => {
  // validate request body
  const body = await readValidatedBody(event, z.object({
    title: z.string().min(1).max(200),
    content: z.string().min(1),
    published: z.boolean().default(false)
  }).parse)

  // create post
  const post = await createPost(body)

  // set response status
  setResponseStatus(event, 201)

  return post
})

// server/api/posts/[id].get.ts
export default defineEventHandler(async (event) => {
  // get route params
  const id = getRouterParam(event, 'id')

  // or validate params
  const params = await getValidatedRouterParams(event, z.object({
    id: z.string().uuid()
  }).parse)

  const post = await getPost(params.id)

  if (!post) {
    throw createError({
      statusCode: 404,
      statusMessage: 'Post not found'
    })
  }

  return post
})

// server/api/posts/[id].delete.ts
export default defineEventHandler(async (event) => {
  // require authentication
  const session = await requireUserSession(event)

  const id = getRouterParam(event, 'id')
  await deletePost(id, session.user.id)

  // return no content
  return sendNoContent(event)
})
```

### Server Utilities

```typescript
// server/utils/db.ts
import { drizzle } from 'drizzle-orm/d1'

// auto-imported in server context only
export const useDB = () => {
  return drizzle(hubDatabase())
}

// server/utils/auth.ts
export const requireAdmin = async (event: H3Event) => {
  const session = await requireUserSession(event)
  
  if (session.user.role !== 'admin') {
    throw createError({
      statusCode: 403,
      statusMessage: 'Forbidden'
    })
  }
  
  return session
}

// usage in server routes
// server/api/admin/users.get.ts
export default defineEventHandler(async (event) => {
  await requireAdmin(event)
  // only admins reach here
  return await getUsers()
})
```

### Error Handling

```typescript
// ✅ good: structured error handling
export default defineEventHandler(async (event) => {
  try {
    const body = await readValidatedBody(event, schema.parse)
    return await processData(body)
  } catch (error) {
    // zod validation errors
    if (error instanceof z.ZodError) {
      throw createError({
        statusCode: 400,
        statusMessage: 'Validation Error',
        data: error.errors
      })
    }
    
    // database errors
    if (error.code === 'SQLITE_CONSTRAINT') {
      throw createError({
        statusCode: 409,
        statusMessage: 'Resource already exists'
      })
    }
    
    // generic server error
    throw createError({
      statusCode: 500,
      statusMessage: 'Internal Server Error'
    })
  }
})
```

## Data Fetching

### When to Use Which

- **`useFetch`**: Default choice for fetching data in components (SSR-friendly)
- **`useAsyncData`**: When you need more control or custom fetcher logic
- **`$fetch`**: Only for client-side event handlers (e.g., button clicks, form submissions)

### useFetch Best Practices

```typescript
// ✅ good: basic data fetching
const { data, pending, error, refresh } = await useFetch('/api/posts')

// ✅ good: with options
const { data: posts } = await useFetch('/api/posts', {
  query: { 
    page: 1, 
    limit: 10 
  },
  // only fetch needed fields
  pick: ['id', 'title', 'createdAt'],
  // refresh when route params change
  watch: [() => route.params.id],
  // don't block navigation
  lazy: true,
  // provide unique cache key
  key: 'posts-list',
  // enable client-side caching
  getCachedData(key) {
    return useNuxtData(key).data.value
  }
})

// ✅ good: reactive url
const id = computed(() => route.params.id)
const { data: post } = await useFetch(() => `/api/posts/${id.value}`)

// ✅ good: with transform
const { data: postTitles } = await useFetch('/api/posts', {
  transform: (posts) => posts.map(p => p.title)
})

// ❌ avoid: using $fetch in setup (causes double fetch)
const posts = await $fetch('/api/posts') // fetches on server AND client
```

### useAsyncData Best Practices

```typescript
// ✅ good: complex data fetching
const { data, pending, error } = await useAsyncData(
  'posts-with-users',
  async () => {
    const [posts, users] = await Promise.all([
      $fetch('/api/posts'),
      $fetch('/api/users')
    ])
    return { posts, users }
  },
  {
    watch: [searchQuery],
    lazy: false
  }
)

// ✅ good: third-party API with custom logic
const { data: weather } = await useAsyncData(
  'weather',
  async () => {
    const response = await $fetch('https://api.weather.com/forecast', {
      query: { city: 'London' }
    })
    return parseWeatherData(response)
  }
)
```

### $fetch Usage

```typescript
// ✅ good: user-triggered actions
const handleSubmit = async () => {
  try {
    loading.value = true
    const result = await $fetch('/api/posts', {
      method: 'POST',
      body: formData.value
    })
    toast.success('Post created!')
    navigateTo(`/posts/${result.id}`)
  } catch (error) {
    toast.error('Failed to create post')
  } finally {
    loading.value = false
  }
}

// ✅ good: optimistic updates
const deletePost = async (id: string) => {
  // optimistically remove from UI
  posts.value = posts.value.filter(p => p.id !== id)
  
  try {
    await $fetch(`/api/posts/${id}`, { method: 'DELETE' })
  } catch (error) {
    // revert on error
    await refresh()
  }
}
```

## Validation with Zod

### Shared Validation Schemas

```typescript
// shared/utils/validators.ts
import { z } from 'zod'

// constants for validation limits
export const POST_TITLE_MIN = 1
export const POST_TITLE_MAX = 200
export const POST_CONTENT_MIN = 1

// schema for creating posts
export const createPostSchema = z.object({
  title: z.string()
    .min(POST_TITLE_MIN, 'Title is required')
    .max(POST_TITLE_MAX, 'Title too long'),
  content: z.string()
    .min(POST_CONTENT_MIN, 'Content is required'),
  published: z.boolean().default(false),
  tags: z.array(z.string()).max(5).optional()
})

export type CreatePost = z.infer<typeof createPostSchema>

// schema for query params
export const postQuerySchema = z.object({
  page: z.coerce.number().positive().default(1),
  limit: z.coerce.number().positive().max(100).default(10),
  search: z.string().optional(),
  tag: z.string().optional()
})

export type PostQuery = z.infer<typeof postQuerySchema>
```

### Server-Side Validation

```typescript
// server/api/posts.post.ts
import { createPostSchema } from '~/shared/utils/validators'

export default defineEventHandler(async (event) => {
  // validate with automatic error handling
  const body = await readValidatedBody(event, createPostSchema.parse)
  
  // or use safeParse for custom error handling
  const result = await readValidatedBody(
    event, 
    body => createPostSchema.safeParse(body)
  )
  
  if (!result.success) {
    throw createError({
      statusCode: 400,
      statusMessage: 'Validation failed',
      data: result.error.errors
    })
  }
  
  const post = await createPost(result.data)
  return post
})

// server/api/posts.get.ts
import { postQuerySchema } from '~/shared/utils/validators'

export default defineEventHandler(async (event) => {
  // validate query parameters
  const query = await getValidatedQuery(event, postQuerySchema.parse)
  
  return await getPosts(query)
})

// server/api/posts/[id].patch.ts
export default defineEventHandler(async (event) => {
  // validate route params
  const params = await getValidatedRouterParams(event, z.object({
    id: z.string().uuid()
  }).parse)
  
  // validate body with partial schema
  const body = await readValidatedBody(
    event,
    createPostSchema.partial().parse
  )
  
  return await updatePost(params.id, body)
})
```

### Client-Side Validation

```vue
<!-- app/components/PostForm.vue -->
<script setup lang="ts">
import { createPostSchema, type CreatePost } from '~/shared/utils/validators'

const form = ref<CreatePost>({
  title: '',
  content: '',
  published: false
})

const errors = ref<Record<string, string>>({})
const pending = ref(false)

const validateForm = () => {
  const result = createPostSchema.safeParse(form.value)
  
  if (!result.success) {
    errors.value = result.error.errors.reduce((acc, err) => {
      acc[err.path[0]] = err.message
      return acc
    }, {} as Record<string, string>)
    return false
  }
  
  errors.value = {}
  return true
}

const handleSubmit = async () => {
  if (!validateForm()) return
  
  pending.value = true
  try {
    const post = await $fetch('/api/posts', {
      method: 'POST',
      body: form.value
    })
    navigateTo(`/posts/${post.id}`)
  } catch (error) {
    toast.error('Failed to create post')
  } finally {
    pending.value = false
  }
}

// validate on blur
const validateField = (field: keyof CreatePost) => {
  const result = createPostSchema.shape[field].safeParse(form.value[field])
  if (!result.success) {
    errors.value[field] = result.error.errors[0].message
  } else {
    delete errors.value[field]
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit">
    <div>
      <label for="title">Title</label>
      <input
        id="title"
        v-model="form.title"
        @blur="validateField('title')"
      >
      <span v-if="errors.title" class="error">
        {{ errors.title }}
      </span>
    </div>

    <div>
      <label for="content">Content</label>
      <textarea
        id="content"
        v-model="form.content"
        @blur="validateField('content')"
      />
      <span v-if="errors.content" class="error">
        {{ errors.content }}
      </span>
    </div>

    <button type="submit" :disabled="pending">
      {{ pending ? 'Creating...' : 'Create Post' }}
    </button>
  </form>
</template>
```

## Components

### Component Organization

```
app/components/
├── base/              # base ui components
│   ├── Button.vue
│   └── Input.vue
├── layout/            # layout components
│   ├── Header.vue
│   └── Footer.vue
└── features/          # feature-specific components
    └── posts/
        ├── PostCard.vue
        ├── PostList.vue
        └── PostForm.vue
```

### Component Best Practices

```vue
<!-- ✅ good: composables pattern -->
<script setup lang="ts">
interface Props {
  postId: string
}

const props = defineProps<Props>()

const { data: post, pending, error, refresh } = await useFetch(
  () => `/api/posts/${props.postId}`
)

const handleDelete = async () => {
  if (!confirm('Delete this post?')) return
  
  await $fetch(`/api/posts/${props.postId}`, {
    method: 'DELETE'
  })
  
  navigateTo('/posts')
}
</script>

<template>
  <div v-if="pending">
    Loading...
  </div>
  <div v-else-if="error">
    Error: {{ error.message }}
  </div>
  <article v-else-if="post">
    <h1>{{ post.title }}</h1>
    <div v-html="post.content" />
    <button @click="handleDelete">
      Delete
    </button>
  </article>
</template>

<style scoped>
/* scoped styles */
</style>
```

### Component Naming (Path-Based)

```
app/components/
├── base/
│   └── Button.vue       → <BaseButton>
├── form/
│   └── Input.vue        → <FormInput>
└── posts/
    └── Card.vue         → <PostsCard>
```

To disable path-based naming:

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: {
    dirs: [
      {
        path: '~/components',
        pathPrefix: false  // disable path prefix
      }
    ]
  }
})
```

## State Management

### Composables Pattern (Recommended)

```typescript
// app/composables/useAuth.ts
export const useAuth = () => {
  const user = useState<User | null>('user', () => null)
  const loading = useState('auth-loading', () => false)

  const login = async (credentials: LoginCredentials) => {
    loading.value = true
    try {
      const { data } = await useFetch('/api/auth/login', {
        method: 'POST',
        body: credentials
      })
      user.value = data.value
    } finally {
      loading.value = false
    }
  }

  const logout = async () => {
    await $fetch('/api/auth/logout', { method: 'POST' })
    user.value = null
  }

  return {
    user: readonly(user),
    loading: readonly(loading),
    login,
    logout
  }
}

// usage in components
const { user, login, logout } = useAuth()
```

### Pinia (For Complex State)

```typescript
// app/stores/cart.ts
export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])
  
  const total = computed(() => 
    items.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
  )
  
  const addItem = (product: Product) => {
    const existing = items.value.find(i => i.id === product.id)
    if (existing) {
      existing.quantity++
    } else {
      items.value.push({ ...product, quantity: 1 })
    }
  }
  
  const removeItem = (id: string) => {
    items.value = items.value.filter(i => i.id !== id)
  }
  
  return {
    items: readonly(items),
    total,
    addItem,
    removeItem
  }
})
```

## Styling

### Tailwind CSS (Recommended)

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxtjs/tailwind'],
  tailwind: {
    cssPath: '~/assets/css/tailwind.css',
    configPath: 'tailwind.config'
  }
})
```

```vue
<!-- use utility classes directly -->
<template>
  <div class="container mx-auto px-4">
    <h1 class="text-3xl font-bold text-gray-900 dark:text-white">
      {{ title }}
    </h1>
    <button class="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700">
      Click me
    </button>
  </div>
</template>
```

## TypeScript Best Practices

### Type Safety

```typescript
// ✅ good: typed api responses
interface Post {
  id: string
  title: string
  content: string
  createdAt: string
}

const { data } = await useFetch<Post[]>('/api/posts')

// ✅ good: typed event handlers
export default defineEventHandler<{ body: CreatePost }>(async (event) => {
  const body = await readBody(event)
  // body is typed as CreatePost
})

// ✅ good: typed composables
export const useCounter = () => {
  const count = ref<number>(0)
  const increment = () => count.value++
  
  return {
    count: readonly(count),
    increment
  }
}

// ✅ good: infer types from zod
const schema = z.object({
  name: z.string(),
  age: z.number()
})

type User = z.infer<typeof schema>
```

## Performance Optimization

### Code Splitting

```vue
<!-- lazy load components -->
<script setup>
const HeavyComponent = defineAsyncComponent(
  () => import('~/components/HeavyComponent.vue')
)
</script>

<!-- lazy load with loading state -->
<script setup>
const AdminPanel = defineAsyncComponent({
  loader: () => import('~/components/AdminPanel.vue'),
  loadingComponent: LoadingSpinner,
  delay: 200,
  timeout: 3000
})
</script>
```

### Image Optimization

```vue
<template>
  <NuxtImg
    src="/images/hero.jpg"
    alt="Hero image"
    width="800"
    height="600"
    format="webp"
    loading="lazy"
    sizes="sm:100vw md:50vw lg:800px"
  />
</template>
```

### Caching Strategies

```typescript
// client-side caching with getCachedData
const { data } = await useFetch('/api/posts', {
  getCachedData(key) {
    const cached = useNuxtData(key)
    const age = Date.now() - (cached.data.value?._fetchedAt || 0)
    
    // cache for 5 minutes
    if (age < 5 * 60 * 1000) {
      return cached.data.value
    }
    
    return undefined
  }
})

// server-side caching
export default defineEventHandler(async (event) => {
  return await cachedFunction(
    () => expensiveOperation(),
    {
      maxAge: 60 * 5, // 5 minutes
      name: 'expensive-op',
      getKey: () => 'unique-key'
    }
  )
})
```

## Testing Strategy

Follow the separate testing guide for comprehensive testing setup with Vitest and Playwright.

## Error Handling

### Global Error Handling

```vue
<!-- app/error.vue -->
<script setup lang="ts">
const props = defineProps<{
  error: {
    statusCode: number
    statusMessage: string
    message: string
  }
}>()

const handleError = () => clearError({ redirect: '/' })
</script>

<template>
  <div class="error-page">
    <h1>{{ error.statusCode }}</h1>
    <p>{{ error.statusMessage || error.message }}</p>
    <button @click="handleError">
      Go home
    </button>
  </div>
</template>
```

### Error Boundaries in Components

```vue
<script setup>
const { data, error, refresh } = await useFetch('/api/posts')
</script>

<template>
  <div v-if="error" class="error-state">
    <p>Failed to load posts</p>
    <button @click="refresh">Retry</button>
  </div>
  <div v-else>
    <!-- render data -->
  </div>
</template>
```

## Methodology

### Development Process

1. **System 2 Thinking**: Break down requirements into smaller parts and consider each step thoroughly
2. **Tree of Thoughts**: Evaluate multiple solutions and their consequences before implementation
3. **Iterative Refinement**: Consider improvements, edge cases, and optimizations before finalizing

### Implementation Flow

1. **Deep Dive Analysis**: Understand requirements and constraints
2. **Planning**: Design architecture using Nuxt 4 conventions
3. **Implementation**: Build features following best practices
4. **Review and Optimize**: Refactor for performance and maintainability
5. **Testing**: Write tests following TDD principles
6. **Finalization**: Ensure code is secure, performant, and documented

## Key Reminders

- Embrace auto-imports but understand their scope (app vs server vs shared)
- Use dot method for API routes (`posts.get.ts`, `posts.post.ts`)
- Validate on both client and server using shared Zod schemas
- Prefer `useFetch` for data fetching, `$fetch` for user actions
- Keep server and app code separate - use `shared/` for common code
- Leverage TypeScript for type safety throughout the stack
- Follow file-based conventions for routing, components, and API
- Write lowercase comments and commit messages
- Test with Vitest for unit/component tests, Playwright for E2E

---

*This guide reflects Nuxt 4 best practices as of 2025. Always refer to [official documentation](https://nuxt.com/docs/4.x) for latest updates.*