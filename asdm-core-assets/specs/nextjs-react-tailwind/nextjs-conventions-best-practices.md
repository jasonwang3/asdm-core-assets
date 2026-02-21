# Next.js Conventions & Best Practices

Rules for adhering to Next.js conventions, including data fetching, rendering, routing, and using 'nuqs' for URL search parameter state management.

## App Router Architecture

### Directory Structure
- Place both the `/app` and `/components` folders under a `/src` directory
- Follow Next.js file conventions:
  - `page.tsx` - Page component
  - `layout.tsx` - Layout component
  - `loading.tsx` - Loading UI
  - `error.tsx` - Error UI
  - `not-found.tsx` - 404 page

```
src/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── components/
└── lib/
```

## Server vs Client Components

### When to Use 'use client'
Limit `'use client'` directive:
- Favor server components and Next.js SSR
- Use only for Web API access in small components
- Avoid for data fetching or state management

### Server Components (Default)
```tsx
// ✅ Good: Server Component (default)
async function ProductList() {
  const products = await fetchProducts();
  
  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}
```

### Client Components (When Necessary)
```tsx
// ✅ Good: Client Component for interactivity
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

## Data Fetching

### Best Practices
- Follow Next.js docs for Data Fetching, Rendering, and Routing
- Fetch data directly in Server Components using async/await
- Use React's `cache()` for request deduplication

```tsx
import { cache } from 'react';

// ✅ Good: Cached data fetch
export const getUser = cache(async (id: string) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
});
```

## URL State Management with nuqs

### Why nuqs?
- Type-safe URL search parameter state management
- Automatic serialization/deserialization
- Works with Next.js App Router

### Usage
```tsx
import { useQueryState } from 'nuqs';

function SearchPage() {
  const [query, setQuery] = useQueryState('q');
  
  return (
    <input
      value={query ?? ''}
      onChange={(e) => setQuery(e.target.value || null)}
      placeholder="Search..."
    />
  );
}
```

## Web Vitals Optimization

### Core Metrics
Optimize Web Vitals:
- **LCP** (Largest Contentful Paint): < 2.5s
- **CLS** (Cumulative Layout Shift): < 0.1
- **FID** (First Input Delay): < 100ms

### Optimization Strategies
1. Use Server Components for faster initial load
2. Implement proper caching strategies
3. Optimize images with Next.js Image component
4. Minimize client-side JavaScript

## Routing Best Practices

### Route Groups
Use parentheses to organize routes without affecting URLs:
```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx
│   └── contact/
│       └── page.tsx
└── (auth)/
    ├── login/
    │   └── page.tsx
    └── register/
        └── page.tsx
```

### Dynamic Routes
```tsx
// app/products/[id]/page.tsx
interface PageProps {
  params: { id: string };
}

export default async function ProductPage({ params }: PageProps) {
  const product = await getProduct(params.id);
  return <ProductDetails product={product} />;
}
```

## Error Handling

### Error Boundaries
Use `error.tsx` for route-level error handling:
```tsx
// app/products/error.tsx
'use client';

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong!</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## Loading States

### Loading UI
Use `loading.tsx` for better UX during data fetching:
```tsx
// app/products/loading.tsx
export default function Loading() {
  return <ProductsSkeleton />;
}
```

## Best Practices Summary

1. ✅ Use `/src` directory for app and components
2. ✅ Default to Server Components
3. ✅ Limit 'use client' to interactive components
4. ✅ Use nuqs for URL state management
5. ✅ Optimize Web Vitals (LCP, CLS, FID)
6. ✅ Follow Next.js data fetching patterns
7. ✅ Use route groups for organization
8. ✅ Implement error and loading states
