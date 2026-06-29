# Complete Next.js Interview Preparation Guide

This guide compiles everything discussed about modern Next.js and the surrounding frontend topics that commonly appear in interviews. It is written for App Router–first Next.js projects, while also calling out older Pages Router concepts where they still matter in real production codebases.

---

## How to Study This Guide

For interviews, do not memorize file names only. Interviewers usually test whether you understand the trade-offs behind each concept:

- When should something render on the server versus the client?
- When should a route be static, dynamic, cached, or streamed?
- How do layouts, route groups, dynamic routes, and loading/error boundaries compose in a real app?
- Where should auth, validation, mutations, and caching live?
- How do you keep a Next.js app production-safe: fast, secure, observable, testable, and maintainable?

Use the following mental model:

> Next.js is not just a React router. It is a full-stack React framework that combines file-system routing, server rendering, React Server Components, server-side data access, route handlers, caching, streaming, optimization, and deployment conventions.

---

## Master Topic List

1. Next.js fundamentals
2. App Router vs Pages Router
3. File-system routing
4. `page.tsx`
5. `layout.tsx`, root layouts, nested layouts
6. Dynamic layouts
7. `template.tsx`
8. Route groups `(group)`
9. Dynamic routes `[id]`
10. Catch-all and optional catch-all routes
11. `generateStaticParams`
12. Parallel routes `@slot`
13. Intercepting routes
14. `loading.tsx` and streaming
15. `error.tsx` and `global-error.tsx`
16. `not-found.tsx` and `notFound()`
17. Redirects
18. Route Handlers
19. Server Components
20. Client Components and `'use client'`
21. Server Actions and `'use server'`
22. Forms and mutations
23. Data fetching
24. Caching, `use cache`, `cacheLife`, `cacheTag`
25. Revalidation, ISR, `revalidatePath`, `revalidateTag`
26. Rendering strategies: SSR, SSG, ISR, CSR, streaming
27. Partial prerendering / Cache Components
28. Metadata and SEO
29. Image optimization
30. Font optimization
31. Script optimization
32. Static assets and `public/`
33. Styling: global CSS, CSS Modules, Tailwind, Sass, CSS-in-JS
34. Navigation: `Link`, `useRouter`, `usePathname`, `useSearchParams`
35. Cookies, headers, and sessions
36. Proxy, formerly Middleware
37. Route segment config
38. Environment variables
39. `next.config.js`
40. TypeScript with Next.js
41. Authentication and authorization
42. API / Backend-for-Frontend pattern
43. Internationalization
44. MDX and content sites
45. Performance and Core Web Vitals
46. Bundle optimization
47. Deployment: Vercel, Node, Docker, static export
48. Observability and instrumentation
49. Security
50. Testing
51. Essential frontend topics: React, JavaScript, TypeScript, CSS, browser rendering, HTTP, accessibility, state management

---

# 1. Next.js Fundamentals

## Definition and Information

Next.js is a React framework for building production web applications. It provides:

- File-system routing
- Server-side rendering
- Static generation
- Incremental static regeneration
- React Server Components
- Client Components
- Server Actions
- Route Handlers
- Image, font, and script optimization
- Metadata and SEO support
- Built-in conventions for deployment and performance

In plain React, you usually need to manually assemble routing, data fetching, rendering strategy, API layer, SEO handling, image optimization, and build/deployment behavior. Next.js gives these as first-class primitives.

## Use Case

Use Next.js when you need:

- SEO-friendly pages
- Fast initial page loads
- Public marketing pages
- Blogs, docs, and content sites
- SaaS dashboards
- E-commerce sites
- Authenticated web apps
- Server-side data access
- Full-stack React apps
- AI apps with server-side tool calls, database access, and streaming UI

## Production Usage

In production, Next.js often works as a frontend plus backend-for-frontend layer. For example:

- Marketing pages are statically generated or incrementally regenerated.
- Dashboards are dynamically rendered based on session and user permissions.
- Server Components fetch directly from databases or internal APIs.
- Route Handlers handle webhooks, lightweight APIs, or BFF endpoints.
- Server Actions handle app-internal form mutations.
- Images, fonts, and scripts are optimized to improve Core Web Vitals.

## Example

```tsx
// app/page.tsx
export default function HomePage() {
  return <h1>Welcome to Crackedin</h1>
}
```

## Most Asked Interview Questions

### Q1. Why use Next.js instead of plain React?

**Answer:** React is mainly a UI library. Next.js adds production-level app architecture: routing, SSR, SSG, ISR, React Server Components, Route Handlers, metadata, image/font/script optimization, caching, streaming, and deployment conventions.

### Q2. Is Next.js frontend or backend?

**Answer:** It is both. It is primarily a React framework, but it can also run server-side code through Server Components, Route Handlers, Server Actions, and server-only utilities.

### Q3. What is the biggest production benefit of Next.js?

**Answer:** The biggest benefit is choosing the right rendering and caching strategy per route/component, which improves SEO, performance, scalability, and developer productivity.

---

# 2. App Router vs Pages Router

## Definition and Information

Next.js has two routing architectures:

| Router | Folder | Status | Key Ideas |
|---|---|---|---|
| App Router | `app/` | Modern/default for new apps | Server Components, layouts, streaming, Server Actions, Route Handlers |
| Pages Router | `pages/` | Older but still widely used | `getServerSideProps`, `getStaticProps`, `getStaticPaths`, API Routes |

The App Router is built around React Server Components and nested layouts. It gives more granular control over rendering, streaming, and data-fetching boundaries.

## Use Case

Use App Router for new projects. Learn Pages Router because many existing companies still run production apps on it.

## Production Usage

Many companies migrate gradually:

```txt
app/
  dashboard/
    page.tsx

pages/
  legacy-report.tsx
  api/legacy-webhook.ts
```

This hybrid state is common. The important thing is to keep conventions clear and not mix patterns randomly.

## Example

### App Router

```tsx
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const data = await getDashboardData()
  return <Dashboard data={data} />
}
```

### Pages Router

```tsx
// pages/dashboard.tsx
export async function getServerSideProps() {
  const data = await getDashboardData()
  return { props: { data } }
}

export default function DashboardPage({ data }) {
  return <Dashboard data={data} />
}
```

## Most Asked Interview Questions

### Q1. What is the main difference between App Router and Pages Router?

**Answer:** App Router uses Server Components, nested layouts, streaming, and file conventions like `layout.tsx`, `loading.tsx`, and `error.tsx`. Pages Router uses page files and data functions like `getServerSideProps` and `getStaticProps`.

### Q2. Should new projects use App Router?

**Answer:** Yes, generally. App Router is the modern architecture and unlocks newer React and Next.js features.

### Q3. Can App Router and Pages Router exist together?

**Answer:** Yes. This is common during migration. However, each route must be owned by one routing system clearly.

---

# 3. File-System Routing

## Definition and Information

Next.js creates routes from folders and files. In App Router:

- Folders define route segments.
- `page.tsx` makes a route publicly accessible.
- Special files like `layout.tsx`, `loading.tsx`, `error.tsx`, and `route.ts` define behavior for that segment.

## Use Case

File-system routing removes the need for manual route configuration.

## Production Usage

A real production app may look like this:

```txt
app/
  (marketing)/
    page.tsx
    pricing/page.tsx
  (auth)/
    login/page.tsx
    signup/page.tsx
  (dashboard)/
    dashboard/page.tsx
    settings/page.tsx
  api/
    health/route.ts
```

This structure keeps product areas separated without requiring a centralized router file.

## Example

```txt
app/products/page.tsx        -> /products
app/products/[id]/page.tsx   -> /products/123
app/settings/profile/page.tsx -> /settings/profile
```

## Most Asked Interview Questions

### Q1. What makes a folder routable in App Router?

**Answer:** A folder becomes publicly accessible when it contains a `page.tsx` file.

### Q2. Do all folders become URLs?

**Answer:** Normal folders become route segments, but route groups like `(marketing)` do not affect the URL.

### Q3. What are special files in App Router?

**Answer:** Files like `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`, `template.tsx`, and `route.ts` define route behavior.

---

# 4. `page.tsx`

## Definition and Information

`page.tsx` defines the UI for a route. It is the leaf entry point for a URL.

In App Router, pages are Server Components by default, so they can be async and fetch data directly on the server.

## Use Case

Use `page.tsx` for each public route:

- `/`
- `/pricing`
- `/dashboard`
- `/products/[id]`
- `/blog/[slug]`

## Production Usage

A production `page.tsx` should usually be thin. It should:

- Read route params/search params
- Fetch server data
- Compose feature components
- Handle route-level not-found/redirect decisions

Avoid dumping large business logic directly inside page files.

## Example

```tsx
// app/products/page.tsx
export default async function ProductsPage() {
  const products = await getProducts()

  return <ProductList products={products} />
}
```

## Most Asked Interview Questions

### Q1. Can `page.tsx` be async?

**Answer:** Yes. In App Router, pages are Server Components by default and can fetch data directly using `await`.

### Q2. Should a page be marked `'use client'`?

**Answer:** Only if the whole page needs client-side state, effects, event handlers, or browser APIs. In production, this is usually avoided. Prefer keeping the page server-side and moving interactivity into small Client Components.

### Q3. What should not be inside `page.tsx`?

**Answer:** Large domain logic, reusable business services, huge client-side state, and unrelated UI. Keep pages focused on route composition.

---

# 5. `layout.tsx`, Root Layouts, and Nested Layouts

## Definition and Information

A layout wraps child routes. Layouts persist across navigation within their route segment.

The root layout is required in App Router and must define the top-level HTML structure:

```tsx
<html>
  <body>{children}</body>
</html>
```

Nested layouts let you create shared UI for sections like dashboard, admin, docs, or auth.

## Use Case

Use layouts for:

- Navbar
- Sidebar
- Footer
- Dashboard shell
- Admin shell
- Auth page shell
- Providers
- Persistent route UI

## Production Usage

Production apps often use multiple nested layouts:

```txt
app/
  layout.tsx                    -> global providers/html/body
  (marketing)/layout.tsx         -> marketing navbar/footer
  (dashboard)/dashboard/layout.tsx -> dashboard sidebar/session shell
```

Layouts are useful because they avoid remounting shared UI on navigation. For example, a dashboard sidebar does not need to reload every time the user changes dashboard tabs.

## Example

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <section className="dashboard-shell">
      <Sidebar />
      <main>{children}</main>
    </section>
  )
}
```

## Most Asked Interview Questions

### Q1. What is the difference between `layout.tsx` and a normal wrapper component?

**Answer:** A layout is part of the routing tree and persists across navigations for its segment. A normal wrapper persists only if its parent remains mounted.

### Q2. What must the root layout include?

**Answer:** The root layout must include `<html>` and `<body>` tags.

### Q3. Why are nested layouts useful?

**Answer:** They allow shared UI and logic to be scoped to a section of the app, improving performance and maintainability.

---

# 6. Dynamic Layouts

## Definition and Information

“Dynamic layout” is a practical concept, not a separate official file convention. It means a layout changes based on runtime information such as:

- Route params
- Tenant/org slug
- User role
- Session state
- Feature flags
- Locale
- Theme/branding

## Use Case

Dynamic layouts are common in SaaS apps:

- `/[org]/dashboard`
- `/[team]/settings`
- Admin vs user layouts
- Tenant-specific branding
- Role-specific navigation

## Production Usage

For a multi-tenant SaaS app, a dynamic layout may:

- Load organization data
- Validate membership
- Load allowed nav items
- Apply tenant branding
- Provide org context to child components

## Example

```tsx
// app/[org]/layout.tsx
import { notFound, redirect } from 'next/navigation'

export default async function OrgLayout({
  children,
  params,
}: {
  children: React.ReactNode
  params: Promise<{ org: string }>
}) {
  const { org } = await params
  const session = await getSession()

  if (!session) redirect('/login')

  const orgData = await getOrgBySlug(org)
  if (!orgData) notFound()

  const hasAccess = await userHasOrgAccess(session.user.id, orgData.id)
  if (!hasAccess) redirect('/dashboard')

  return (
    <OrgProvider org={orgData}>
      <OrgSidebar org={orgData} />
      <main>{children}</main>
    </OrgProvider>
  )
}
```

## Most Asked Interview Questions

### Q1. How do you create dynamic layouts in Next.js?

**Answer:** Use dynamic route segments like `[org]`, nested layouts, route groups, and server-side checks based on params/session/permissions.

### Q2. Are dynamic layouts always dynamic-rendered?

**Answer:** Not necessarily. A layout becomes dynamic when it depends on runtime data such as cookies, headers, uncached fetches, or per-request session data.

### Q3. What is a common mistake with dynamic layouts?

**Answer:** Fetching heavy data in a top-level layout that runs for too many pages. Keep layout data minimal and push page-specific data into page or child components.

---

# 7. `template.tsx`

## Definition and Information

`template.tsx` is similar to `layout.tsx`, but it remounts on navigation. Layouts persist; templates create a new instance for each navigation.

## Use Case

Use templates when you intentionally want state reset or remount behavior:

- Restart page animations
- Reset form state between sibling routes
- Re-run effects
- Re-trigger analytics for segment transitions

## Production Usage

Use sparingly. Most shared UI should be in layouts. Templates are useful when persistent layout behavior becomes a problem.

## Example

```tsx
// app/dashboard/template.tsx
export default function DashboardTemplate({
  children,
}: {
  children: React.ReactNode
}) {
  return <div className="page-transition">{children}</div>
}
```

## Most Asked Interview Questions

### Q1. Difference between `layout.tsx` and `template.tsx`?

**Answer:** Layouts persist across navigation; templates remount on navigation.

### Q2. When should you use `template.tsx`?

**Answer:** When you need fresh component state, re-triggered animations, or effects on each navigation.

### Q3. Why not use templates everywhere?

**Answer:** Remounting can lose useful state and cause unnecessary work. Layout persistence is usually better for performance and UX.

---

# 8. Route Groups `(group)`

## Definition and Information

Route groups organize routes without affecting the URL. A folder wrapped in parentheses is ignored in the route path.

Example:

```txt
app/(marketing)/pricing/page.tsx -> /pricing
```

The `(marketing)` segment does not appear in the URL.

## Use Case

Use route groups to:

- Organize large apps
- Apply different layouts to different sections
- Separate marketing, auth, app, admin, docs
- Create multiple root layouts
- Avoid URL pollution

## Production Usage

A production app often uses route groups like this:

```txt
app/
  (marketing)/
    layout.tsx
    page.tsx
    pricing/page.tsx
  (auth)/
    layout.tsx
    login/page.tsx
  (app)/
    dashboard/page.tsx
    settings/page.tsx
```

This keeps code clean without changing public routes.

## Example

```txt
app/
  (public)/
    page.tsx          -> /
    about/page.tsx    -> /about
  (private)/
    dashboard/page.tsx -> /dashboard
```

## Most Asked Interview Questions

### Q1. Does a route group appear in the URL?

**Answer:** No. Route groups are ignored in the URL path.

### Q2. Why use route groups?

**Answer:** To organize routes and apply layouts to specific sections without changing URLs.

### Q3. Can route groups create conflicting routes?

**Answer:** Yes. If two route groups resolve to the same URL path, that creates a conflict.

---

# 9. Dynamic Routes `[id]`

## Definition and Information

Dynamic routes capture URL values using bracket syntax.

```txt
app/products/[id]/page.tsx -> /products/123
```

The captured value is available in `params`.

## Use Case

Use dynamic routes for:

- Product pages
- Blog posts
- User profiles
- Company pages
- Organization dashboards
- Interview experience detail pages

## Production Usage

Dynamic routes are central to production apps. Common examples:

- `/companies/[slug]`
- `/jobs/[id]`
- `/problems/[slug]`
- `/dashboard/[workspaceId]`

You usually combine dynamic routes with:

- `notFound()` for missing data
- `generateMetadata()` for SEO
- `generateStaticParams()` for prebuilt static pages
- Auth checks for protected resources

## Example

```tsx
// app/problems/[slug]/page.tsx
import { notFound } from 'next/navigation'

export default async function ProblemPage({
  params,
}: {
  params: Promise<{ slug: string }>
}) {
  const { slug } = await params
  const problem = await getProblemBySlug(slug)

  if (!problem) notFound()

  return <ProblemDetails problem={problem} />
}
```

## Most Asked Interview Questions

### Q1. What is `[slug]` in Next.js?

**Answer:** It is a dynamic route segment that captures a value from the URL and exposes it through `params`.

### Q2. How do you handle a missing dynamic resource?

**Answer:** Fetch the resource on the server and call `notFound()` if it does not exist.

### Q3. How do dynamic routes work with SEO?

**Answer:** Use `generateMetadata()` to produce title, description, and Open Graph metadata based on the dynamic params.

---

# 10. Catch-All and Optional Catch-All Routes

## Definition and Information

Catch-all routes capture multiple URL segments.

```txt
[...slug]    -> required catch-all
[[...slug]]  -> optional catch-all
```

Examples:

```txt
app/docs/[...slug]/page.tsx
/docs/a
/docs/a/b
/docs/a/b/c
```

`params.slug` becomes an array.

## Use Case

Use catch-all routes for:

- Documentation sites
- CMS-driven pages
- Nested categories
- Knowledge bases
- File-browser-like routes

## Production Usage

Docs and CMS systems often map arbitrary nested paths to content records.

Example:

```txt
/docs/nextjs/routing/layouts
```

could map to:

```ts
['nextjs', 'routing', 'layouts']
```

## Example

```tsx
// app/docs/[...slug]/page.tsx
export default async function DocsPage({
  params,
}: {
  params: Promise<{ slug: string[] }>
}) {
  const { slug } = await params
  const page = await getDocsPage(slug.join('/'))

  if (!page) notFound()

  return <Article content={page.content} />
}
```

## Most Asked Interview Questions

### Q1. Difference between `[...slug]` and `[[...slug]]`?

**Answer:** `[...slug]` requires at least one segment. `[[...slug]]` also matches the base route.

### Q2. What type is `params.slug`?

**Answer:** It is usually an array of strings for catch-all routes.

### Q3. Where are catch-all routes useful?

**Answer:** Docs, nested category pages, CMS pages, and flexible path-based content.

---

# 11. `generateStaticParams`

## Definition and Information

`generateStaticParams` tells Next.js which dynamic routes should be generated statically at build time.

It is the App Router equivalent of the older `getStaticPaths` concept from Pages Router.

## Use Case

Use it for known dynamic pages:

- Blog posts
- Documentation pages
- Product pages
- Course pages
- Company pages

## Production Usage

For large sites, you may statically generate the most popular pages and let less popular pages render later, depending on configuration and caching strategy.

## Example

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  const posts = await getAllPosts()

  return posts.map((post) => ({
    slug: post.slug,
  }))
}

export default async function BlogPostPage({ params }) {
  const { slug } = await params
  const post = await getPost(slug)

  return <Article post={post} />
}
```

## Most Asked Interview Questions

### Q1. What does `generateStaticParams` do?

**Answer:** It predefines params for dynamic routes that should be statically generated.

### Q2. What did it replace from Pages Router?

**Answer:** It replaces the role of `getStaticPaths` in App Router.

### Q3. Should every dynamic route use `generateStaticParams`?

**Answer:** No. Use it when pages are known and benefit from static generation. For user-specific or highly dynamic routes, server rendering may be better.

---

# 12. Parallel Routes `@slot`

## Definition and Information

Parallel routes let you render multiple independent route branches inside the same layout. Slots are defined with the `@folder` convention.

Example:

```txt
app/dashboard/
  layout.tsx
  @analytics/page.tsx
  @team/page.tsx
```

The slots are passed as props to the parent layout.

## Use Case

Use parallel routes for:

- Dashboards
- Split views
- Feeds
- Analytics panels
- Notification panels
- Conditional panels
- Modal systems with intercepting routes

## Production Usage

Parallel routes are useful when different parts of the UI need independent loading, error, and navigation states.

Example: a dashboard can show team data and analytics data at the same time, with separate loading states.

## Example

```tsx
// app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <main>
      <section>{children}</section>
      <aside>{analytics}</aside>
      <aside>{team}</aside>
    </main>
  )
}
```

## Most Asked Interview Questions

### Q1. What are parallel routes?

**Answer:** They allow multiple pages or route branches to render simultaneously in the same layout using named slots.

### Q2. Do parallel route slots affect the URL?

**Answer:** No. Slots are not URL segments.

### Q3. Where are parallel routes useful?

**Answer:** Dashboards, feeds, split views, modals, and complex layouts where different UI regions need independent routing behavior.

---

# 13. Intercepting Routes

## Definition and Information

Intercepting routes allow one route to be displayed inside another route’s layout while masking or preserving navigation context.

They are often used with parallel routes to implement modal routing.

## Use Case

Common example:

- User is on `/feed`.
- They click photo `/photo/123`.
- The photo opens as a modal over the feed.
- If they refresh or directly open `/photo/123`, it renders as a full page.

## Production Usage

Use for:

- Photo modals
- Product preview modals
- Login/signup modals
- Cart side panels
- Detail panels in dashboards

This gives better UX because browser back/forward buttons and deep links still work.

## Example

```txt
app/
  feed/
    page.tsx
    @modal/
      (..)photo/[id]/page.tsx
  photo/
    [id]/page.tsx
```

## Most Asked Interview Questions

### Q1. Why use intercepting routes instead of normal component modals?

**Answer:** Intercepting routes preserve routing behavior, deep links, browser back/forward behavior, and direct full-page rendering.

### Q2. What are intercepting routes commonly paired with?

**Answer:** Parallel routes.

### Q3. What is a production example?

**Answer:** Instagram-style photo modals where opening from feed shows a modal, but direct URL access shows a full page.

---

# 14. `loading.tsx` and Streaming

## Definition and Information

`loading.tsx` defines loading UI for a route segment. Next.js automatically wraps the segment in a Suspense boundary and shows the fallback while content loads.

Streaming means the server can send parts of the UI progressively instead of waiting for the entire page to finish rendering.

## Use Case

Use for:

- Dashboard skeletons
- Search pages
- Slow data panels
- AI-generated responses
- Reports
- Analytics pages

## Production Usage

In production, loading states should be meaningful, not generic spinners everywhere. Prefer skeletons that match the final layout.

Streaming improves perceived performance because users see the stable shell early while slower parts load later.

## Example

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />
}
```

Granular Suspense example:

```tsx
import { Suspense } from 'react'

export default function DashboardPage() {
  return (
    <>
      <Header />
      <Suspense fallback={<ChartSkeleton />}>
        <SlowAnalyticsChart />
      </Suspense>
    </>
  )
}
```

## Most Asked Interview Questions

### Q1. What does `loading.tsx` do?

**Answer:** It provides instant fallback UI for a route segment while that segment loads or streams.

### Q2. Why is streaming useful?

**Answer:** It improves perceived performance by sending available UI immediately and progressively revealing slower data-dependent parts.

### Q3. Spinner or skeleton?

**Answer:** Skeletons are usually better for production because they preserve layout and reduce perceived wait time.

---

# 15. `error.tsx` and `global-error.tsx`

## Definition and Information

`error.tsx` defines an error boundary for a route segment. It catches rendering errors in that segment and displays fallback UI.

`global-error.tsx` catches root-level errors.

`error.tsx` must be a Client Component because it uses client-side reset behavior.

## Use Case

Use for:

- Graceful route-level error handling
- Retry UI
- Segment-specific fallback
- Preventing one broken panel from crashing the whole app

## Production Usage

In production:

- Show user-friendly error messages.
- Log errors to Sentry, Datadog, or another monitoring system.
- Provide retry with `reset()` where valid.
- Avoid exposing internal stack traces to users.

## Example

```tsx
// app/dashboard/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error
  reset: () => void
}) {
  return (
    <div>
      <h2>Something went wrong.</h2>
      <button onClick={reset}>Try again</button>
    </div>
  )
}
```

## Most Asked Interview Questions

### Q1. Why does `error.tsx` need `'use client'`?

**Answer:** Because it uses client-side interactivity such as the `reset()` function.

### Q2. Difference between `error.tsx` and `global-error.tsx`?

**Answer:** `error.tsx` handles errors for a segment. `global-error.tsx` handles root-level errors.

### Q3. What should you avoid in production error UI?

**Answer:** Avoid showing raw stack traces, database errors, tokens, or internal implementation details.

---

# 16. `not-found.tsx` and `notFound()`

## Definition and Information

`notFound()` is a function that triggers a 404 response and renders the nearest `not-found.tsx` file.

## Use Case

Use it when a requested resource does not exist:

- Product not found
- Blog post deleted
- Invalid user profile
- Unknown organization slug
- Missing docs page

## Production Usage

Call `notFound()` after server-side validation or database lookup. This is better than manually returning a random error component because it produces correct 404 behavior.

## Example

```tsx
// app/products/[id]/page.tsx
import { notFound } from 'next/navigation'

export default async function ProductPage({ params }) {
  const { id } = await params
  const product = await getProduct(id)

  if (!product) notFound()

  return <ProductDetails product={product} />
}
```

```tsx
// app/products/[id]/not-found.tsx
export default function ProductNotFound() {
  return <h1>Product not found</h1>
}
```

## Most Asked Interview Questions

### Q1. When should you use `notFound()`?

**Answer:** When the route is valid but the requested resource does not exist.

### Q2. Is `notFound()` same as redirecting to `/404`?

**Answer:** No. `notFound()` triggers proper 404 behavior and renders the nearest `not-found.tsx` boundary.

### Q3. Can each route segment have its own not-found UI?

**Answer:** Yes. You can define `not-found.tsx` at different segment levels.

---

# 17. Redirects

## Definition and Information

Redirects send users from one URL to another. Next.js supports redirects through:

- `redirect()`
- `permanentRedirect()`
- `useRouter().push()`
- `next.config.js` redirects
- Proxy-level redirects

## Use Case

Use redirects for:

- Auth protection
- Post-submit navigation
- Old URL migration
- Role-based routing
- Canonical URLs

## Production Usage

Choose the correct redirect mechanism:

| Scenario | Best Tool |
|---|---|
| Server-side auth check | `redirect()` |
| Client button click | `useRouter().push()` |
| Old static URLs | `next.config.js` redirects |
| Request-level conditional logic | Proxy |
| Permanent resource move | `permanentRedirect()` |

## Example

```tsx
import { redirect } from 'next/navigation'

export default async function DashboardPage() {
  const session = await getSession()

  if (!session) redirect('/login')

  return <Dashboard />
}
```

## Most Asked Interview Questions

### Q1. Difference between `redirect()` and `router.push()`?

**Answer:** `redirect()` is used in server-side rendering/server logic. `router.push()` is client-side navigation from event handlers.

### Q2. When should you use `permanentRedirect()`?

**Answer:** When a URL has permanently moved and search engines should update the canonical location.

### Q3. What is an open redirect vulnerability?

**Answer:** It happens when user-controlled redirect URLs allow navigation to malicious external sites. Always validate redirect targets.

---

# 18. Route Handlers

## Definition and Information

Route Handlers define custom server endpoints inside `app/**/route.ts`. They use Web `Request` and `Response` APIs and support HTTP methods like `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.

## Use Case

Use Route Handlers for:

- API endpoints
- Webhooks
- Auth callbacks
- BFF endpoints
- Health checks
- File upload URLs
- Server-side integrations

## Production Usage

Route Handlers are good for small/medium server endpoints. For large business domains, move business logic into service modules so it can be reused by Route Handlers, Server Actions, jobs, and tests.

## Example

```ts
// app/api/health/route.ts
export async function GET() {
  return Response.json({ ok: true })
}
```

```ts
// app/api/jobs/route.ts
export async function POST(request: Request) {
  const body = await request.json()
  const job = await createJob(body)

  return Response.json(job, { status: 201 })
}
```

## Most Asked Interview Questions

### Q1. Route Handler vs API Route?

**Answer:** Route Handlers belong to App Router. API Routes belong to Pages Router.

### Q2. Should every mutation be a Route Handler?

**Answer:** Not necessarily. App-internal form mutations may be better as Server Actions. Public APIs, webhooks, and external clients should use Route Handlers.

### Q3. Why keep business logic outside Route Handlers?

**Answer:** So it is testable, reusable, and not tightly coupled to HTTP request handling.

---

# 19. Server Components

## Definition and Information

Server Components render on the server and do not ship their component code to the browser. In App Router, components are Server Components by default unless marked as Client Components.

Server Components can:

- Fetch data directly
- Access server-only services
- Use secrets safely
- Reduce client JavaScript
- Render HTML/RSC payload from the server

They cannot:

- Use `useState`
- Use `useEffect`
- Use browser-only APIs like `window`
- Attach event handlers directly

## Use Case

Use Server Components for:

- Data-heavy pages
- SEO content
- Server-side database access
- Static pages
- Secure logic
- Reducing JS bundle size

## Production Usage

In production, default to Server Components. Move only interactive parts into Client Components.

Bad pattern:

```tsx
'use client'

export default function HugeDashboardPage() {
  // Everything now goes to client bundle
}
```

Better pattern:

```tsx
export default async function DashboardPage() {
  const data = await getData()

  return (
    <>
      <ServerRenderedStats data={data.stats} />
      <InteractiveChart data={data.chart} />
    </>
  )
}
```

## Example

```tsx
// app/orders/page.tsx
export default async function OrdersPage() {
  const orders = await db.order.findMany()

  return <OrderTable orders={orders} />
}
```

## Most Asked Interview Questions

### Q1. What are Server Components?

**Answer:** Components that render on the server and do not send their component JavaScript to the browser.

### Q2. Why are Server Components useful?

**Answer:** They reduce client bundle size, allow server-side data access, protect secrets, and improve initial performance.

### Q3. Can Server Components use hooks?

**Answer:** They cannot use client hooks like `useState` or `useEffect`. They can be async and fetch data directly.

---

# 20. Client Components and `'use client'`

## Definition and Information

Client Components run in the browser. A file becomes a Client Component boundary when it starts with `'use client'`.

Client Components can use:

- `useState`
- `useEffect`
- Event handlers
- Browser APIs
- Client-side routing hooks
- Interactive libraries

## Use Case

Use Client Components for:

- Buttons with click handlers
- Forms with live validation
- Modals
- Tabs
- Dropdowns
- Charts
- Editors
- Drag-and-drop
- Browser API usage

## Production Usage

Use small Client Components as interactive islands. Do not mark large layouts/pages as client-side unless necessary.

## Example

```tsx
// app/ui/like-button.tsx
'use client'

import { useState } from 'react'

export function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes)

  return (
    <button onClick={() => setLikes(likes + 1)}>
      {likes} likes
    </button>
  )
}
```

## Most Asked Interview Questions

### Q1. What does `'use client'` do?

**Answer:** It defines a client boundary. That component and its client-side imports become part of the browser bundle.

### Q2. What is the biggest mistake with Client Components?

**Answer:** Adding `'use client'` too high in the tree, which increases bundle size and weakens server-rendering benefits.

### Q3. Can Server Components pass props to Client Components?

**Answer:** Yes, but props must be serializable.

---

# 21. Server Actions and `'use server'`

## Definition and Information

Server Actions are async functions that run on the server and can be called from forms or Client Components. They use the `'use server'` directive.

## Use Case

Use Server Actions for:

- Form submissions
- Creating/updating/deleting records
- Authenticated internal mutations
- Simple app-specific mutations

## Production Usage

Server Actions should still be treated as backend endpoints. Always:

- Validate input
- Check auth
- Check authorization
- Handle expected errors
- Revalidate affected cache
- Avoid trusting client data

## Example

```ts
// app/actions.ts
'use server'

import { revalidatePath } from 'next/cache'

export async function createPost(formData: FormData) {
  const title = String(formData.get('title') || '')

  if (!title.trim()) {
    return { error: 'Title is required' }
  }

  await db.post.create({ data: { title } })
  revalidatePath('/posts')

  return { success: true }
}
```

```tsx
// app/posts/new/page.tsx
import { createPost } from '@/app/actions'

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  )
}
```

## Most Asked Interview Questions

### Q1. Are Server Actions a replacement for APIs?

**Answer:** No. They are excellent for app-internal mutations, especially forms. Public APIs, mobile clients, webhooks, and third-party integrations still need Route Handlers or separate backend services.

### Q2. Are Server Actions secure automatically?

**Answer:** No. They run on the server, but you must still validate input and check permissions.

### Q3. What should happen after a mutation?

**Answer:** Usually revalidate affected cached data, redirect if needed, and return expected validation errors in a user-friendly way.

---

# 22. Forms and Mutations

## Definition and Information

Forms in App Router can submit directly to Server Actions. This allows mutations without manually creating client-side fetch calls.

## Use Case

Use for:

- Signup
- Login
- Create post
- Update profile
- Delete item
- Contact form
- Settings forms

## Production Usage

Production forms should include:

- Server-side validation
- Client-side convenience validation when helpful
- Pending state
- Error display
- CSRF/session protection depending on auth architecture
- Idempotency for payment/order-like operations

## Example

```tsx
// app/profile/ProfileForm.tsx
'use client'

import { useActionState } from 'react'
import { updateProfile } from './actions'

export function ProfileForm() {
  const [state, action, pending] = useActionState(updateProfile, null)

  return (
    <form action={action}>
      <input name="name" />
      {state?.error && <p>{state.error}</p>}
      <button disabled={pending}>Save</button>
    </form>
  )
}
```

## Most Asked Interview Questions

### Q1. Should validation happen only on the client?

**Answer:** No. Client validation improves UX, but server validation is mandatory because clients can be bypassed.

### Q2. How do you show pending state for forms?

**Answer:** Use React form/action hooks or client-side state around the action.

### Q3. What is idempotency in mutations?

**Answer:** It means repeating the same request should not create duplicate side effects, such as double orders or duplicate payments.

---

# 23. Data Fetching

## Definition and Information

In App Router, data can be fetched in Server Components, Client Components, Route Handlers, or Server Actions.

Server Components can fetch with:

- `fetch`
- ORM/database clients
- Internal services
- Server-only modules

## Use Case

Use server-side data fetching for:

- SEO pages
- Secure data
- Initial dashboard data
- Database queries
- Reduced client requests

Use client-side fetching for:

- Highly interactive filters
- Polling
- Browser-only APIs
- Client-owned state
- User-triggered requests after page load

## Production Usage

Avoid waterfalls. Fetch independent data in parallel.

Bad:

```tsx
const user = await getUser()
const invoices = await getInvoices(user.id)
const stats = await getStats(user.id)
```

Better when independent:

```tsx
const [user, stats] = await Promise.all([
  getUser(),
  getStats(),
])
```

## Example

```tsx
export default async function DashboardPage() {
  const [profile, recommendations, progress] = await Promise.all([
    getProfile(),
    getRecommendations(),
    getProgress(),
  ])

  return (
    <Dashboard
      profile={profile}
      recommendations={recommendations}
      progress={progress}
    />
  )
}
```

## Most Asked Interview Questions

### Q1. Where should you fetch data in App Router?

**Answer:** Prefer Server Components for initial secure data. Use Client Components for interactive client-driven data.

### Q2. What is a request waterfall?

**Answer:** Sequential requests that wait on each other unnecessarily, increasing latency.

### Q3. Why fetch on the server?

**Answer:** It reduces client JavaScript, protects secrets, improves SEO, and can be closer to the database/service.

---

# 24. Caching, `use cache`, `cacheLife`, and `cacheTag`

## Definition and Information

Caching stores the result of data fetching or rendering so future requests can reuse it.

Modern Next.js includes Cache Components and directives/functions such as:

- `'use cache'`
- `cacheLife()`
- `cacheTag()`
- `revalidateTag()`
- `updateTag()`

The idea is to explicitly decide what can be cached, for how long, and how it should be invalidated.

## Use Case

Cache:

- Blog posts
- Docs pages
- Product catalogs
- Public company pages
- Feature flags that do not change often
- Expensive non-user-specific queries

Do not globally cache:

- Private user dashboards
- Session-specific data
- Auth-specific permissions
- Sensitive account data
- Anything dependent on per-request cookies/headers unless using the correct private/request-aware pattern

## Production Usage

The main production risk is stale or leaked data. Caching is a performance feature, but wrong caching is a correctness/security bug.

Use this decision flow:

1. Is the data public and shared? Cache more aggressively.
2. Is the data user-specific? Be careful; often avoid global cache.
3. How stale can it be? Choose cache lifetime.
4. What event changes it? Use tag/path revalidation.
5. Can incorrect stale data harm users? Reduce cache or avoid it.

## Example

```ts
import { cacheLife, cacheTag } from 'next/cache'

export async function getPublishedPosts() {
  'use cache'
  cacheLife('hours')
  cacheTag('posts')

  return db.post.findMany({ where: { published: true } })
}
```

## Most Asked Interview Questions

### Q1. What should you cache?

**Answer:** Cache data that does not depend on runtime/private request state and that can tolerate being stale for some period.

### Q2. What is the difference between cache lifetime and invalidation?

**Answer:** Lifetime controls how long cached data can live. Invalidation forces cache refresh when something changes.

### Q3. What is the biggest caching mistake?

**Answer:** Accidentally caching user-specific or permission-specific data globally.

---

# 25. Revalidation and ISR

## Definition and Information

Revalidation refreshes cached/static content. ISR, or Incremental Static Regeneration, allows static pages to update after deployment without rebuilding the entire app.

Common tools:

- `revalidatePath('/path')`
- `revalidateTag('tag')`
- `export const revalidate = 60`

## Use Case

Use ISR/revalidation for:

- Blogs
- Docs
- CMS content
- Product pages
- Landing pages
- Public catalogs

## Production Usage

Prefer tag-based revalidation when possible because it is more precise. Path revalidation is useful but can over-invalidate.

Example production flow:

1. CMS updates a blog post.
2. CMS webhook hits a Route Handler.
3. Route Handler calls `revalidateTag('post:123')`.
4. Next request receives fresh content.

## Example

```tsx
// app/blog/page.tsx
export const revalidate = 60

export default async function BlogPage() {
  const posts = await getPublishedPosts()
  return <PostList posts={posts} />
}
```

```ts
// app/api/revalidate/route.ts
import { revalidateTag } from 'next/cache'

export async function POST(request: Request) {
  const { tag, secret } = await request.json()

  if (secret !== process.env.REVALIDATION_SECRET) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  revalidateTag(tag)
  return Response.json({ revalidated: true })
}
```

## Most Asked Interview Questions

### Q1. What is ISR?

**Answer:** ISR lets static pages be regenerated after deployment, either after a time interval or on demand.

### Q2. `revalidatePath` vs `revalidateTag`?

**Answer:** `revalidatePath` invalidates by route path. `revalidateTag` invalidates all cached data associated with a tag and is often more precise for data-driven apps.

### Q3. When is ISR not appropriate?

**Answer:** For highly personalized, sensitive, or real-time data where stale content is unacceptable.

---

# 26. Rendering Strategies: SSR, SSG, ISR, CSR, Streaming

## Definition and Information

| Strategy | Meaning | Best For |
|---|---|---|
| SSR | Server renders per request | Personalized/dynamic pages |
| SSG | Static generation at build time | Stable public pages |
| ISR | Static generation with regeneration | Public pages that change sometimes |
| CSR | Client-side rendering/fetching | Highly interactive browser-driven UI |
| Streaming | Progressive server rendering | Slow data sections, better perceived UX |

## Use Case

Choose based on:

- SEO needs
- Personalization
- Data freshness
- Performance
- Security
- Cacheability

## Production Usage

Good defaults:

- Marketing: SSG/ISR
- Blog/docs: SSG/ISR
- Dashboard: SSR/dynamic + streaming
- Search filters: server initial render + client interaction
- Admin pages: dynamic server rendering
- AI response pages: streaming

## Example

```tsx
// Static-ish page
export const revalidate = 3600

export default async function PricingPage() {
  const plans = await getPricingPlans()
  return <Pricing plans={plans} />
}
```

```tsx
// Dynamic page
export default async function AccountPage() {
  const session = await getSession()
  const account = await getAccount(session.user.id)
  return <Account account={account} />
}
```

## Most Asked Interview Questions

### Q1. SSR vs SSG?

**Answer:** SSR renders per request; SSG renders at build time. SSR is fresher and personalized; SSG is faster and more cacheable.

### Q2. When should you use CSR?

**Answer:** For browser-only, highly interactive, or user-triggered data that does not need initial SEO rendering.

### Q3. What is streaming good for?

**Answer:** Showing the shell immediately while slow sections load progressively.

---

# 27. Partial Prerendering / Cache Components

## Definition and Information

Modern Next.js moves toward component-level static/dynamic composition. The idea is that a page can have a static shell and dynamic streamed parts.

This is better than thinking only at the route level.

## Use Case

Use this when a page has both stable and dynamic parts:

- Static marketing hero + dynamic pricing experiment
- Dashboard shell + live user metrics
- Docs page + dynamic comments
- Product page + dynamic inventory

## Production Usage

This improves performance because stable UI can be cached/prerendered while dynamic pieces remain fresh.

## Example

```tsx
import { Suspense } from 'react'

export default function ProductPage() {
  return (
    <>
      <StaticProductDescription />
      <Suspense fallback={<InventorySkeleton />}>
        <LiveInventory />
      </Suspense>
    </>
  )
}
```

## Most Asked Interview Questions

### Q1. Why is component-level rendering useful?

**Answer:** It lets you cache stable parts while rendering dynamic parts only where needed.

### Q2. What is the risk of treating the whole route as dynamic?

**Answer:** You lose caching/performance benefits for parts that could have been static.

### Q3. What enables this pattern?

**Answer:** Server Components, Suspense, streaming, and explicit cache boundaries.

---

# 28. Metadata and SEO

## Definition and Information

Next.js Metadata APIs define page metadata such as:

- Title
- Description
- Open Graph tags
- Twitter cards
- Canonical URLs
- Robots directives
- Icons
- Sitemap-related metadata

## Use Case

Use for:

- SEO
- Social sharing previews
- Dynamic product/blog titles
- Search engine indexing behavior

## Production Usage

Use static metadata for simple pages and `generateMetadata()` for dynamic pages.

## Example

```tsx
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }) {
  const { slug } = await params
  const post = await getPost(slug)

  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  }
}
```

## Most Asked Interview Questions

### Q1. Static metadata vs `generateMetadata()`?

**Answer:** Static metadata is fixed. `generateMetadata()` computes metadata dynamically from route params or fetched data.

### Q2. Why is metadata important?

**Answer:** It affects SEO, link previews, indexing, and social sharing.

### Q3. Can metadata be dynamic?

**Answer:** Yes, especially for product pages, blog posts, user-generated content, and CMS-driven pages.

---

# 29. Image Optimization

## Definition and Information

Next.js provides `next/image` for optimized images. It supports responsive sizing, lazy loading, resizing, and better layout stability.

## Use Case

Use for:

- Hero images
- Product images
- Blog covers
- Avatars
- Thumbnails
- Marketing assets

## Production Usage

Best practices:

- Always provide `alt` text.
- Provide width and height to avoid layout shift.
- Use `priority` only for above-the-fold critical images.
- Configure remote image domains/patterns.
- Avoid sending massive original images to users.

## Example

```tsx
import Image from 'next/image'

export function HeroImage() {
  return (
    <Image
      src="/hero.png"
      alt="Dashboard preview"
      width={1200}
      height={700}
      priority
    />
  )
}
```

## Most Asked Interview Questions

### Q1. Why use `next/image` instead of `<img>`?

**Answer:** It improves performance through optimization, responsive sizing, lazy loading, and layout stability.

### Q2. What does `priority` do?

**Answer:** It marks an image as high priority for loading, usually for above-the-fold images.

### Q3. Why do width and height matter?

**Answer:** They reserve layout space and prevent Cumulative Layout Shift.

---

# 30. Font Optimization

## Definition and Information

Next.js font optimization helps load fonts efficiently and reduce layout shift. `next/font` can self-host fonts and avoid extra external requests.

## Use Case

Use for:

- Brand typography
- Google Fonts
- Local fonts
- Performance-sensitive sites

## Production Usage

Prefer `next/font` over manually loading fonts through external `<link>` tags.

## Example

```tsx
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  )
}
```

## Most Asked Interview Questions

### Q1. Why does font optimization matter?

**Answer:** Poor font loading can cause invisible text, flash of unstyled text, and layout shift.

### Q2. What is the advantage of `next/font`?

**Answer:** It can self-host and optimize fonts automatically, improving performance and privacy.

### Q3. What is CLS?

**Answer:** Cumulative Layout Shift, a Core Web Vital measuring unexpected layout movement.

---

# 31. Script Optimization

## Definition and Information

`next/script` controls how third-party scripts load.

Common strategies:

- `beforeInteractive`
- `afterInteractive`
- `lazyOnload`
- `worker` where supported/appropriate

## Use Case

Use for:

- Analytics
- Payment SDKs
- Chat widgets
- Ads
- Tag managers

## Production Usage

Third-party scripts are a major performance risk. Load only what is necessary and avoid blocking the main thread.

## Example

```tsx
import Script from 'next/script'

export function AnalyticsScript() {
  return (
    <Script
      src="https://example.com/analytics.js"
      strategy="afterInteractive"
    />
  )
}
```

## Most Asked Interview Questions

### Q1. Why not put all scripts in `<head>`?

**Answer:** Blocking scripts can delay page rendering and interactivity.

### Q2. Which scripts should load early?

**Answer:** Only scripts that are critical for the page to function.

### Q3. What is the risk of third-party scripts?

**Answer:** They can increase load time, block the main thread, break privacy expectations, or create security risks.

---

# 32. Static Assets and `public/`

## Definition and Information

Files inside `public/` are served from the root URL.

Example:

```txt
public/logo.svg -> /logo.svg
```

## Use Case

Use `public/` for:

- Favicons
- Static images
- Robots files
- Downloadable files
- Static JSON that must be publicly accessible

## Production Usage

Use `public/` for assets that do not need import-time processing. For images that need optimization, prefer `next/image`.

## Example

```tsx
export function Logo() {
  return <img src="/logo.svg" alt="Logo" />
}
```

## Most Asked Interview Questions

### Q1. When should you use `public/`?

**Answer:** For static files that should be directly URL-accessible.

### Q2. Are files in `public/` private?

**Answer:** No. Anything in `public/` is publicly accessible.

### Q3. Should secrets ever go in `public/`?

**Answer:** Absolutely not.

---

# 33. Styling in Next.js

## Definition and Information

Next.js supports multiple styling approaches:

- Global CSS
- CSS Modules
- Tailwind CSS
- Sass
- PostCSS
- CSS-in-JS, depending on compatibility and setup

## Use Case

Use based on team preference and app scale.

Common production choices:

- Tailwind for fast UI development and design systems
- CSS Modules for scoped component styles
- Global CSS for resets, variables, and base styles

## Production Usage

Recommended split:

- Global CSS: reset, root variables, typography base
- CSS Modules/Tailwind: component styling
- Design tokens: colors, spacing, typography
- Avoid uncontrolled global class collisions

## Example

```css
/* app/ui/button.module.css */
.primary {
  padding: 0.75rem 1rem;
  border-radius: 0.5rem;
  font-weight: 600;
}
```

```tsx
import styles from './button.module.css'

export function Button() {
  return <button className={styles.primary}>Save</button>
}
```

## Most Asked Interview Questions

### Q1. CSS Modules vs global CSS?

**Answer:** CSS Modules are scoped to components. Global CSS applies across the entire app.

### Q2. Why is uncontrolled global CSS risky?

**Answer:** It can cause style collisions and unpredictable changes across unrelated components.

### Q3. Is Tailwind good for production?

**Answer:** Yes, if used with design discipline. Without component abstraction and tokens, it can become messy.

---

# 34. Navigation APIs

## Definition and Information

Next.js provides:

- `Link` for declarative client-side navigation
- `useRouter()` for programmatic client-side navigation
- `usePathname()` for reading current path
- `useSearchParams()` for reading query params

These hooks are used in Client Components.

## Use Case

Use for:

- Nav links
- Active sidebar state
- Search filters
- Pagination
- Programmatic redirects after user actions

## Production Usage

Prefer `Link` for navigation. Use `router.push()` only when navigation is triggered by client-side logic.

## Example

```tsx
import Link from 'next/link'

export function Navbar() {
  return <Link href="/dashboard">Dashboard</Link>
}
```

```tsx
'use client'

import { useRouter } from 'next/navigation'

export function SaveButton() {
  const router = useRouter()

  return <button onClick={() => router.push('/settings')}>Go</button>
}
```

## Most Asked Interview Questions

### Q1. `Link` vs `router.push()`?

**Answer:** Use `Link` for normal navigation. Use `router.push()` for event-driven client-side navigation.

### Q2. Can `useRouter()` run in Server Components?

**Answer:** No. It is a client hook and requires a Client Component.

### Q3. Where should URL query state live?

**Answer:** If the state should be shareable/bookmarkable, put it in the URL using search params.

---

# 35. Cookies, Headers, and Sessions

## Definition and Information

Next.js provides server utilities to access request cookies and headers.

They are used for:

- Auth tokens
- Locale detection
- A/B testing
- Session lookup
- Request-specific rendering

## Use Case

Use cookies/headers when server rendering depends on the incoming request.

## Production Usage

Reading cookies or headers usually makes a route dynamic because the output can vary per request.

Use secure cookie settings:

- `httpOnly`
- `secure`
- `sameSite`
- appropriate expiration
- scoped path/domain

## Example

```tsx
import { cookies } from 'next/headers'

export default async function AccountPage() {
  const cookieStore = await cookies()
  const sessionToken = cookieStore.get('session')?.value

  const account = sessionToken
    ? await getAccountFromSession(sessionToken)
    : null

  return <AccountView account={account} />
}
```

## Most Asked Interview Questions

### Q1. Why can cookies make a route dynamic?

**Answer:** Because the rendered output may depend on each individual request.

### Q2. What is an `httpOnly` cookie?

**Answer:** A cookie that cannot be accessed by client-side JavaScript, reducing XSS token theft risk.

### Q3. Should sensitive session tokens be stored in `localStorage`?

**Answer:** Generally no. `httpOnly` secure cookies are safer for many web auth systems.

---

# 36. Proxy, Formerly Middleware

## Definition and Information

Modern Next.js uses `proxy.ts` for request interception before routes complete. The older `middleware.ts` convention has been deprecated/renamed to Proxy in recent docs.

Proxy can:

- Redirect
- Rewrite
- Modify headers
- Inspect cookies
- Return responses
- Apply request-level logic

## Use Case

Use Proxy for:

- Lightweight auth gates
- Locale redirects
- A/B test routing
- Bot blocking
- Request header manipulation
- Legacy URL rewrites

## Production Usage

Keep Proxy lightweight. Do not put heavy database work or complex business logic there.

Important production note: Proxy can be useful for broad checks, but sensitive authorization should still be enforced in the data-access/server layer. Do not rely only on Proxy for security.

## Example

```ts
// proxy.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function proxy(request: NextRequest) {
  const token = request.cookies.get('session')?.value

  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }

  return NextResponse.next()
}

export const config = {
  matcher: ['/dashboard/:path*'],
}
```

## Most Asked Interview Questions

### Q1. What is Proxy used for in Next.js?

**Answer:** Request-level logic before rendering, such as redirects, rewrites, lightweight auth gates, and header manipulation.

### Q2. Why should Proxy be lightweight?

**Answer:** It runs before matched requests and can add latency globally.

### Q3. Should authorization live only in Proxy?

**Answer:** No. Proxy can redirect unauthenticated users, but sensitive authorization must be enforced where data is accessed.

---

# 37. Route Segment Config

## Definition and Information

Route segment config lets you control behavior for a route segment, such as:

- Runtime
- Revalidation
- Dynamic/static behavior
- Preferred region
- Max duration
- Dynamic params

## Use Case

Use it when default behavior is not enough.

## Production Usage

Be explicit when correctness matters. Wrong segment config can cause:

- Stale data
- Slow dynamic rendering
- Unexpected static generation
- Runtime incompatibility

## Example

```tsx
export const runtime = 'nodejs'
export const revalidate = 60
export const dynamicParams = true

export default async function Page() {
  return <div>Configured route</div>
}
```

## Most Asked Interview Questions

### Q1. Why use route segment config?

**Answer:** To control rendering, caching, runtime, and deployment behavior of a route.

### Q2. What is `runtime = 'nodejs'` useful for?

**Answer:** Node runtime is needed for APIs/libraries that require Node-specific capabilities.

### Q3. What can go wrong with wrong config?

**Answer:** You may accidentally cache dynamic data, force expensive rendering, or use an incompatible runtime.

---

# 38. Environment Variables

## Definition and Information

Environment variables store configuration and secrets.

Important rule:

- Server-only env vars stay on the server.
- Variables prefixed with `NEXT_PUBLIC_` are exposed to the browser.

## Use Case

Use environment variables for:

- Database URLs
- API keys
- Auth secrets
- Public app URLs
- Feature flags
- Third-party service config

## Production Usage

Never put secrets in `NEXT_PUBLIC_` variables. Anything public-prefixed can be visible in the client bundle.

## Example

```ts
const databaseUrl = process.env.DATABASE_URL
const publicAppUrl = process.env.NEXT_PUBLIC_APP_URL
```

## Most Asked Interview Questions

### Q1. What does `NEXT_PUBLIC_` do?

**Answer:** It exposes the environment variable to client-side code.

### Q2. Where should database credentials live?

**Answer:** In server-only environment variables, never in client-exposed variables.

### Q3. Should `.env.local` be committed?

**Answer:** No. It usually contains secrets and local-only config.

---

# 39. `next.config.js` / `next.config.ts`

## Definition and Information

`next.config` controls framework-level configuration.

Common settings:

- Image remote patterns
- Redirects
- Rewrites
- Headers
- Transpiling packages
- Output mode
- Experimental features

## Use Case

Use it when app behavior must be configured globally.

## Production Usage

Keep config minimal and documented. Overcomplicated config becomes difficult to debug.

## Example

```ts
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'cdn.example.com',
      },
    ],
  },
  async redirects() {
    return [
      {
        source: '/old-pricing',
        destination: '/pricing',
        permanent: true,
      },
    ]
  },
}

export default nextConfig
```

## Most Asked Interview Questions

### Q1. Redirect vs rewrite?

**Answer:** Redirect changes the URL in the browser. Rewrite serves content from another path while keeping the visible URL unchanged.

### Q2. Why configure remote image patterns?

**Answer:** To allow `next/image` to optimize images from approved external sources.

### Q3. Should business logic go in `next.config`?

**Answer:** No. It should contain framework/build configuration, not domain logic.

---

# 40. TypeScript with Next.js

## Definition and Information

Next.js has strong TypeScript support. TypeScript helps catch issues in route params, component props, server responses, schemas, and config.

## Use Case

Use TypeScript for:

- Props
- API contracts
- Database types
- Server Action inputs/outputs
- Route params
- Shared domain models

## Production Usage

Use strict mode when possible. Combine TypeScript with runtime validation libraries like Zod because TypeScript does not validate runtime input.

## Example

```tsx
type PageProps = {
  params: Promise<{ id: string }>
}

export default async function ProductPage({ params }: PageProps) {
  const { id } = await params
  return <div>Product {id}</div>
}
```

## Most Asked Interview Questions

### Q1. Why use TypeScript with Next.js?

**Answer:** It catches data-shape, prop, route, and API contract mistakes before runtime.

### Q2. Does TypeScript replace runtime validation?

**Answer:** No. External input still needs runtime validation.

### Q3. What is a practical production pattern?

**Answer:** Use TypeScript for compile-time safety and Zod or similar tools for runtime validation.

---

# 41. Authentication and Authorization

## Definition and Information

Authentication answers: “Who is the user?”

Authorization answers: “What is this user allowed to do?”

## Use Case

Use for:

- Login/signup
- Protected routes
- Role-based access
- Tenant access
- Admin panels
- API protection

## Production Usage

Never rely only on client-side checks. Always check authorization on the server before reading or mutating protected data.

Good production layers:

1. Proxy: broad redirect for unauthenticated users
2. Server Component/layout: session checks
3. Data access layer: strict permission checks
4. Database: row-level security if applicable

## Example

```tsx
import { redirect } from 'next/navigation'

export default async function AdminPage() {
  const user = await requireUser()

  if (user.role !== 'admin') {
    redirect('/dashboard')
  }

  return <AdminPanel />
}
```

## Most Asked Interview Questions

### Q1. Authentication vs authorization?

**Answer:** Authentication verifies identity. Authorization verifies permissions.

### Q2. Why is client-only route protection unsafe?

**Answer:** Client code can be bypassed. Sensitive data and actions must be protected on the server.

### Q3. Where should permission checks live?

**Answer:** As close to data access/mutation as possible, not only in UI or Proxy.

---

# 42. API / Backend-for-Frontend Pattern

## Definition and Information

A Backend-for-Frontend, or BFF, adapts backend services for frontend needs. Next.js can act as a BFF using Server Components, Route Handlers, and Server Actions.

## Use Case

Use BFF when the frontend needs to:

- Aggregate multiple backend APIs
- Hide internal services
- Normalize data
- Add session-aware logic
- Reduce client complexity

## Production Usage

Keep the BFF thin. Do not turn route files into huge service layers. Put domain logic in reusable server modules.

## Example

```ts
// app/api/dashboard/route.ts
export async function GET() {
  const session = await requireSession()

  const [profile, progress, recommendations] = await Promise.all([
    getProfile(session.user.id),
    getProgress(session.user.id),
    getRecommendations(session.user.id),
  ])

  return Response.json({ profile, progress, recommendations })
}
```

## Most Asked Interview Questions

### Q1. Why use a BFF?

**Answer:** It shapes backend data for the frontend, reduces client complexity, and hides internal service details.

### Q2. Should the BFF contain all business logic?

**Answer:** No. It should orchestrate and adapt. Core domain logic should live in services.

### Q3. Server Component vs Route Handler for fetching data?

**Answer:** If only the page needs the data, Server Component fetching is often simpler. Use Route Handlers for reusable HTTP endpoints, external clients, or webhooks.

---

# 43. Internationalization

## Definition and Information

Internationalization, or i18n, means supporting multiple languages, locales, formats, and regional behavior.

It includes:

- Text translation
- Locale-aware routing
- Date formatting
- Number/currency formatting
- Localized metadata
- Directionality for RTL languages

## Use Case

Use for global products, multi-region SaaS, localized content, and SEO across countries/languages.

## Production Usage

Common route structure:

```txt
app/[locale]/page.tsx
app/[locale]/pricing/page.tsx
```

Production considerations:

- Locale detection
- Fallback language
- Translated metadata
- Localized slugs
- Currency/timezone handling

## Example

```tsx
// app/[locale]/layout.tsx
export default async function LocaleLayout({ children, params }) {
  const { locale } = await params
  const messages = await getMessages(locale)

  return (
    <html lang={locale}>
      <body>
        <I18nProvider messages={messages}>{children}</I18nProvider>
      </body>
    </html>
  )
}
```

## Most Asked Interview Questions

### Q1. What must be localized besides text?

**Answer:** Dates, numbers, currencies, URLs, metadata, validation messages, and sometimes images.

### Q2. Why is localized metadata important?

**Answer:** It improves SEO and social sharing for each language/region.

### Q3. What is a fallback locale?

**Answer:** The default language used when a translation is missing.

---

# 44. MDX and Content Sites

## Definition and Information

MDX lets you write Markdown with JSX components.

## Use Case

Use MDX for:

- Blogs
- Docs
- Tutorials
- Changelogs
- Developer documentation
- Long-form educational content

## Production Usage

MDX is useful when non-engineers or content writers need readable content files, while developers still want reusable React components inside content.

## Example

```mdx
# Getting Started

This is a normal paragraph.

<Callout type="warning">
  Server-side validation is mandatory.
</Callout>
```

## Most Asked Interview Questions

### Q1. Why use MDX instead of Markdown?

**Answer:** MDX allows React components inside Markdown content.

### Q2. Where is MDX useful?

**Answer:** Documentation, blogs, tutorials, changelogs, and content-heavy sites.

### Q3. What is a risk of MDX?

**Answer:** Unsafe or uncontrolled MDX can introduce security and maintainability issues. Treat content sources carefully.

---

# 45. Performance and Core Web Vitals

## Definition and Information

Core Web Vitals measure user-perceived performance:

- LCP: Largest Contentful Paint
- INP: Interaction to Next Paint
- CLS: Cumulative Layout Shift

Performance also includes:

- TTFB
- Hydration cost
- Bundle size
- Image loading
- Font loading
- Caching
- Server response time

## Use Case

Performance matters for:

- SEO
- Conversion
- User retention
- SaaS dashboard usability
- Mobile users

## Production Usage

Common optimizations:

- Prefer Server Components
- Reduce Client Component boundaries
- Optimize images/fonts/scripts
- Use caching carefully
- Stream slow sections
- Avoid unnecessary hydration
- Monitor Web Vitals
- Use bundle analyzer

## Example

```tsx
// Good: only interactive piece is client-side
export default async function ProductPage() {
  const product = await getProduct()

  return (
    <>
      <ProductInfo product={product} />
      <AddToCartButton productId={product.id} />
    </>
  )
}
```

## Most Asked Interview Questions

### Q1. What usually hurts Next.js performance?

**Answer:** Too many Client Components, large JS bundles, unoptimized images, blocking third-party scripts, no caching, slow server data fetching, and request waterfalls.

### Q2. How do Server Components help performance?

**Answer:** They reduce browser JavaScript and move data fetching/rendering work to the server.

### Q3. What is hydration?

**Answer:** Hydration is React attaching event handlers to server-rendered HTML so Client Components become interactive.

---

# 46. Bundle Optimization

## Definition and Information

Bundle optimization means reducing the JavaScript shipped to the browser.

## Use Case

Use for:

- Faster initial load
- Faster hydration
- Better mobile performance
- Lower bandwidth usage

## Production Usage

Best practices:

- Keep pages/layouts as Server Components by default.
- Use `'use client'` only where needed.
- Dynamically import heavy client-only libraries.
- Avoid importing large libraries into client boundaries.
- Use tree-shakable imports.
- Analyze bundles periodically.

## Example

```tsx
import dynamic from 'next/dynamic'

const Chart = dynamic(() => import('./Chart'), {
  ssr: false,
})

export function AnalyticsPanel() {
  return <Chart />
}
```

## Most Asked Interview Questions

### Q1. How can `'use client'` increase bundle size?

**Answer:** Imports inside the client boundary can become part of the browser bundle.

### Q2. Why dynamically import heavy components?

**Answer:** To avoid loading expensive code until it is actually needed.

### Q3. What tools help inspect bundle size?

**Answer:** Bundle analyzers, build output inspection, Lighthouse, and performance monitoring tools.

---

# 47. Deployment

## Definition and Information

Next.js can be deployed on:

- Vercel
- Node servers
- Docker containers
- Cloud platforms
- Static export for limited use cases

## Use Case

Choose deployment based on:

- Need for SSR
- Need for ISR/cache persistence
- Edge/runtime needs
- Cost
- Team infra capability
- Vendor preference

## Production Usage

Vercel is the easiest managed platform for Next.js, but it is not required. Self-hosting requires careful handling of:

- Reverse proxy
- CDN
- Cache consistency
- Environment variables
- Build artifacts
- Horizontal scaling
- Logging/monitoring

## Example Dockerfile

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
CMD ["npm", "run", "start"]
```

## Most Asked Interview Questions

### Q1. Is Vercel required for Next.js?

**Answer:** No. Next.js can be self-hosted, but Vercel provides the smoothest managed experience.

### Q2. What is difficult about self-hosting?

**Answer:** Caching, ISR consistency, scaling, observability, image optimization, environment management, and deployment automation.

### Q3. When is static export valid?

**Answer:** For fully static sites that do not need SSR, dynamic server features, Route Handlers, or server-side runtime behavior.

---

# 48. Observability and Instrumentation

## Definition and Information

Observability means understanding what your app is doing in production through:

- Logs
- Metrics
- Traces
- Error reporting
- Web Vitals
- Request IDs
- Alerting

## Use Case

Use observability to debug:

- Slow routes
- Failed Server Actions
- API errors
- Hydration issues
- High TTFB
- Cache misses
- Third-party failures

## Production Usage

A mature app tracks:

- Route latency
- API latency
- Error rate
- Cache hit rate
- Revalidation failures
- LCP/INP/CLS
- Bundle size
- Server Action failures
- Database query latency

## Example

```ts
// instrumentation.ts
export async function register() {
  // Initialize tracing/logging/monitoring SDK here
}
```

## Most Asked Interview Questions

### Q1. What should you monitor in a Next.js app?

**Answer:** Web Vitals, server route latency, API latency, error rate, cache behavior, bundle size, and server action failures.

### Q2. Why are request IDs useful?

**Answer:** They let you trace one user request across frontend, backend, logs, and services.

### Q3. What is the difference between logs, metrics, and traces?

**Answer:** Logs are events, metrics are numeric time-series, and traces show request flow across systems.

---

# 49. Security

## Definition and Information

Security in Next.js means protecting against:

- XSS
- CSRF
- Data leakage
- Secret exposure
- Auth bypass
- Open redirects
- Insecure cookies
- Dependency vulnerabilities
- Unsafe file uploads
- Overly permissive CORS

## Use Case

Every production app needs security controls.

## Production Usage

Security checklist:

- Validate all external input.
- Authorize on the server.
- Keep secrets server-only.
- Use secure `httpOnly` cookies where appropriate.
- Sanitize user-generated HTML.
- Use CSRF protection where needed.
- Configure security headers.
- Avoid open redirects.
- Rate-limit sensitive endpoints.
- Never trust client-provided role/user IDs.

## Example

```ts
function getSafeRedirectUrl(url: string | null) {
  if (!url) return '/dashboard'
  if (!url.startsWith('/')) return '/dashboard'
  if (url.startsWith('//')) return '/dashboard'

  return url
}
```

## Most Asked Interview Questions

### Q1. Biggest security mistake in Next.js?

**Answer:** Trusting client-side checks and forgetting server-side authorization.

### Q2. What is XSS?

**Answer:** Cross-site scripting, where malicious scripts run in a user’s browser through unsafe rendering or injection.

### Q3. What is CSRF?

**Answer:** Cross-site request forgery, where another site tricks a user’s browser into making an authenticated request.

---

# 50. Testing

## Definition and Information

Testing validates that your app works and prevents regressions.

Types:

- Unit tests
- Component tests
- Integration tests
- E2E tests
- Visual regression tests
- Accessibility tests

## Use Case

Use testing for:

- Critical flows
- Forms
- Auth
- Checkout/payment
- Dashboard loading
- API behavior
- Server Actions
- Route Handlers

## Production Usage

Recommended stack:

- Vitest/Jest for unit tests
- React Testing Library for components
- Playwright/Cypress for E2E
- MSW for API mocks
- CI checks before merge

## Example

```ts
import { test, expect } from '@playwright/test'

test('home page loads', async ({ page }) => {
  await page.goto('/')
  await expect(page.getByRole('heading')).toBeVisible()
})
```

## Most Asked Interview Questions

### Q1. What should E2E tests cover?

**Answer:** Critical user journeys such as login, signup, checkout, onboarding, form submission, and dashboard loading.

### Q2. What should unit tests cover?

**Answer:** Pure business logic, validation, utility functions, and service behavior.

### Q3. Why not test everything with E2E?

**Answer:** E2E tests are slower, more expensive, and more brittle. Use the right test level for the risk.

---

# 51. Essential Frontend Topics for Next.js Interviews

These are not Next.js-specific, but they are frequently asked because Next.js is built on top of React and browser fundamentals.

---

## 51.1 React Fundamentals

### Definition and Information

React is a UI library based on components, props, state, rendering, and reconciliation.

Key concepts:

- Components
- Props
- State
- Hooks
- Effects
- Context
- Reconciliation
- Controlled/uncontrolled inputs
- Composition

### Use Case

Every Next.js app is still a React app. Next.js adds architecture; React powers the UI.

### Production Usage

Use React composition well:

- Keep components small.
- Avoid unnecessary global state.
- Do not misuse `useEffect` for derived state.
- Memoize only when there is a real performance reason.

### Example

```tsx
import { useState } from 'react'

function Counter() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

### Interview Q/A

**Q: When should you use `useEffect`?**

**Answer:** For synchronizing with external systems, subscriptions, browser APIs, timers, or network effects. Do not use it for simple derived values that can be calculated during render.

---

## 51.2 JavaScript Fundamentals

### Definition and Information

Important JavaScript topics:

- Closures
- Scope
- Hoisting
- Promises
- Async/await
- Event loop
- Microtasks/macrotasks
- Modules
- Array methods
- Object references
- Prototypes

### Use Case

Next.js code uses JavaScript on both the server and client.

### Production Usage

Understand async behavior deeply. Many frontend bugs come from race conditions, stale closures, and incorrect promise handling.

### Example

```ts
const [user, posts] = await Promise.all([
  fetchUser(),
  fetchPosts(),
])
```

### Interview Q/A

**Q: `Promise.all` vs sequential `await`?**

**Answer:** `Promise.all` runs independent async tasks concurrently. Sequential `await` waits for one to finish before starting/continuing the next.

---

## 51.3 TypeScript Fundamentals

### Definition and Information

TypeScript adds static typing to JavaScript.

Important topics:

- `type` vs `interface`
- Generics
- Union/intersection types
- Utility types
- Narrowing
- Type guards
- Inference
- `unknown` vs `any`

### Use Case

TypeScript helps prevent bugs in props, API responses, forms, database models, and route params.

### Production Usage

Avoid excessive `any`. Use generated types and runtime schemas where possible.

### Example

```ts
type User = {
  id: string
  name: string
  role: 'user' | 'admin'
}
```

### Interview Q/A

**Q: `unknown` vs `any`?**

**Answer:** `any` disables type checking. `unknown` forces you to narrow the value before using it.

---

## 51.4 CSS and Responsive Design

### Definition and Information

Core CSS concepts:

- Box model
- Specificity
- Cascade
- Flexbox
- Grid
- Positioning
- Media queries
- Container queries
- Responsive units

### Use Case

Next.js apps still require strong CSS fundamentals for production UI quality.

### Production Usage

Use responsive layouts from the beginning. Do not build desktop-only UI and patch mobile later.

### Example

```css
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1rem;
}
```

### Interview Q/A

**Q: Flexbox vs Grid?**

**Answer:** Flexbox is best for one-dimensional layout. Grid is best for two-dimensional rows and columns.

---

## 51.5 Browser Rendering

### Definition and Information

The browser rendering pipeline includes:

1. Parse HTML
2. Build DOM
3. Parse CSS
4. Build CSSOM
5. Build render tree
6. Layout
7. Paint
8. Composite

### Use Case

Understanding this helps optimize performance and avoid layout issues.

### Production Usage

Avoid:

- Layout thrashing
- Massive DOM trees
- Unoptimized images
- Late-loading fonts
- Expensive animations on layout-affecting properties

Prefer animating `transform` and `opacity`.

### Example

```css
.card:hover {
  transform: translateY(-4px);
}
```

### Interview Q/A

**Q: What causes layout shift?**

**Answer:** Images without dimensions, late-loading fonts, injected content above visible content, ads, and dynamically resized components.

---

## 51.6 HTTP and APIs

### Definition and Information

Important HTTP topics:

- Methods: GET, POST, PUT, PATCH, DELETE
- Status codes
- Headers
- Cookies
- Caching
- CORS
- Authentication
- Idempotency

### Use Case

Next.js apps communicate with APIs, Route Handlers, backend services, and third-party systems.

### Production Usage

Use correct status codes and request methods. Do not misuse GET for mutations.

### Example

```ts
await fetch('/api/profile', {
  method: 'PATCH',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ name: 'Harsh' }),
})
```

### Interview Q/A

**Q: 401 vs 403?**

**Answer:** 401 means unauthenticated. 403 means authenticated but not authorized.

---

## 51.7 State Management

### Definition and Information

State management means deciding where data/state should live.

Types:

- Local component state
- URL state
- Server state
- Global client state
- Form state
- Persistent storage state

### Use Case

Use state management for:

- Forms
- Filters
- Modals
- User preferences
- Cached API data
- Multi-step flows

### Production Usage

Recommended decision tree:

1. Can it be derived? Do not store it.
2. Is it local to one component? Use local state.
3. Should it be shareable/bookmarkable? Use URL state.
4. Is it server data? Use server fetching or a server-state library.
5. Is it truly global UI state? Use Zustand/Redux/Context carefully.

### Example

```tsx
'use client'

import { useSearchParams } from 'next/navigation'

export function SearchLabel() {
  const params = useSearchParams()
  const query = params.get('q') || ''

  return <p>Searching for: {query}</p>
}
```

### Interview Q/A

**Q: Should everything go into Redux?**

**Answer:** No. Most state should be local, URL-based, server-rendered, or handled by server-state tools. Global stores are for truly shared client state.

---

## 51.8 Accessibility

### Definition and Information

Accessibility means making the app usable by people using keyboards, screen readers, alternative input methods, or different visual settings.

Important concepts:

- Semantic HTML
- Keyboard navigation
- Focus management
- ARIA
- Color contrast
- Form labels
- Accessible modals

### Use Case

Accessibility is required for professional-grade UI and often legally important.

### Production Usage

Start with semantic HTML. Use ARIA only when native HTML is not enough.

### Example

```tsx
<button aria-label="Close modal">×</button>
```

Better when visible label exists:

```tsx
<button>Close</button>
```

### Interview Q/A

**Q: What is the first rule of accessibility?**

**Answer:** Use semantic HTML before adding ARIA.

---

## 51.9 Frontend Security

### Definition and Information

Frontend security overlaps with backend security but includes browser-specific risks:

- XSS
- CSRF
- Token storage risks
- Clickjacking
- Dependency attacks
- Unsafe redirects
- Leaking secrets into bundles

### Use Case

Any Next.js app with auth, forms, user-generated content, or APIs needs frontend security awareness.

### Production Usage

Avoid storing sensitive tokens in `localStorage` if XSS risk matters. Avoid `dangerouslySetInnerHTML` unless sanitized.

### Example

```tsx
// Avoid this with untrusted user HTML
<div dangerouslySetInnerHTML={{ __html: userProvidedHtml }} />
```

### Interview Q/A

**Q: Why is `dangerouslySetInnerHTML` risky?**

**Answer:** It can render untrusted HTML/JS and create XSS vulnerabilities if content is not sanitized.

---

# High-Priority Interview Revision Checklist

If you have limited time, prioritize these:

1. App Router mental model
2. Server Components vs Client Components
3. Layouts, dynamic layouts, and route groups
4. Dynamic routes, catch-all routes, and `generateStaticParams`
5. Loading/error/not-found boundaries
6. Data fetching and avoiding waterfalls
7. Caching and revalidation
8. Rendering strategies: SSR, SSG, ISR, CSR, streaming
9. Server Actions vs Route Handlers
10. Auth and server-side authorization
11. Performance: bundle size, hydration, images, fonts, scripts
12. Deployment and observability
13. Frontend fundamentals: React, JS, TS, CSS, HTTP, accessibility

---

# Common Production Mistakes to Avoid

1. Adding `'use client'` too high in the component tree.
2. Fetching all data sequentially and creating waterfalls.
3. Caching private user-specific data globally.
4. Relying only on client-side auth checks.
5. Putting heavy database logic in Proxy.
6. Using spinners everywhere instead of skeletons and streaming.
7. Mixing Pages Router and App Router conventions without discipline.
8. Exposing secrets through `NEXT_PUBLIC_` variables.
9. Using Route Handlers for all app-internal form mutations when Server Actions would be simpler.
10. Ignoring Core Web Vitals until the end.
11. Overusing global state.
12. Forgetting runtime validation because TypeScript “looks safe.”
13. Not logging/monitoring Server Actions, Route Handlers, and critical user flows.
14. Creating inaccessible custom components instead of using semantic HTML.
15. Treating Next.js as only a frontend framework and ignoring its server/runtime behavior.

---

# Practical Production Architecture Recommendation

For most modern Next.js SaaS apps:

```txt
app/
  (marketing)/          -> static/ISR public pages
  (auth)/               -> login/signup/forgot-password
  (dashboard)/          -> authenticated app shell
  api/                  -> webhooks/public API/BFF routes
  layout.tsx            -> root providers and HTML shell

server/
  auth.ts               -> session helpers
  db.ts                 -> database client
  services/             -> business logic
  validators/           -> runtime schemas
  permissions/          -> authorization rules

components/
  ui/                   -> reusable UI components
  features/             -> feature-specific components
```

Rules:

- Keep pages and layouts mostly server-side.
- Use Client Components only for interactivity.
- Keep Route Handlers thin.
- Keep Server Actions validated and authorized.
- Put domain logic in server services.
- Cache public/stable data intentionally.
- Avoid global caching for private data.
- Use route groups to separate app areas cleanly.
- Use nested layouts for persistent shells.
- Use streaming and Suspense for slow sections.
- Track performance and errors from day one.

---

# Official References

- Next.js App Router: https://nextjs.org/docs/app
- Server and Client Components: https://nextjs.org/docs/app/getting-started/server-and-client-components
- Route Groups: https://nextjs.org/docs/app/api-reference/file-conventions/route-groups
- Parallel Routes: https://nextjs.org/docs/app/api-reference/file-conventions/parallel-routes
- Intercepting Routes: https://nextjs.org/docs/app/api-reference/file-conventions/intercepting-routes
- Loading UI and Streaming: https://nextjs.org/docs/app/guides/streaming
- Route Handlers: https://nextjs.org/docs/app/getting-started/route-handlers
- Server Actions and Mutations: https://nextjs.org/docs/14/app/building-your-application/data-fetching/server-actions-and-mutations
- Fetching Data: https://nextjs.org/docs/app/getting-started/fetching-data
- Revalidating: https://nextjs.org/docs/app/getting-started/revalidating
- ISR: https://nextjs.org/docs/app/guides/incremental-static-regeneration
- Proxy: https://nextjs.org/docs/app/api-reference/file-conventions/proxy
- React Server Components: https://react.dev/reference/rsc/server-components

