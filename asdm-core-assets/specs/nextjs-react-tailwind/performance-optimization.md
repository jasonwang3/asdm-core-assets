# Performance Optimization

Rules for performance optimization in Next.js applications, including minimizing client-side logic, using Suspense, and optimizing images.

## Core Principles

### Server Components First
Minimize `'use client'`, `useEffect`, and `setState`:
- Favor React Server Components (RSC)
- Reduce client-side JavaScript bundle
- Improve initial page load

### Performance Goals
- Optimize Core Web Vitals (LCP, CLS, FID)
- Minimize Time to Interactive (TTI)
- Reduce JavaScript bundle size

## Server Components

### When to Use
Default to Server Components for:
- Static content
- Data fetching
- SEO-critical pages
- Initial page render

### Example
```tsx
// ✅ Good: Server Component
async function ProductList() {
  const products = await fetchProducts(); // Direct fetch
  
  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>{product.name}</li>
      ))}
    </ul>
  );
}

// ❌ Bad: Client Component for data fetching
'use client';

function ProductList() {
  const [products, setProducts] = useState([]);
  
  useEffect(() => {
    fetchProducts().then(setProducts); // Client-side fetch
  }, []);
  
  return <ul>{products.map(...)}</ul>;
}
```

## Suspense Boundaries

### When to Use
Wrap client components in Suspense with fallback:
- Code splitting points
- Async component boundaries
- Progressive loading

### Example
```tsx
import { Suspense } from 'react';

function ProductPage() {
  return (
    <div>
      <h1>Products</h1>
      <Suspense fallback={<ProductListSkeleton />}>
        <ProductList />
      </Suspense>
    </div>
  );
}

function ProductListSkeleton() {
  return (
    <div className="space-y-4">
      {[...Array(5)].map((_, i) => (
        <div key={i} className="animate-pulse">
          <div className="h-20 bg-gray-200 rounded" />
        </div>
      ))}
    </div>
  );
}
```

### Granular Suspense
Use multiple Suspense boundaries for better UX:

```tsx
function Dashboard() {
  return (
    <div className="grid grid-cols-2 gap-4">
      <Suspense fallback={<StatsSkeleton />}>
        <Stats />
      </Suspense>
      <Suspense fallback={<ChartSkeleton />}>
        <Chart />
      </Suspense>
      <Suspense fallback={<TableSkeleton />}>
        <DataTable />
      </Suspense>
    </div>
  );
}
```

## Dynamic Imports

### Code Splitting
Use dynamic loading for non-critical components:

```tsx
import dynamic from 'next/dynamic';

// Dynamic import with loading state
const HeavyChart = dynamic(() => import('./heavy-chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false, // If component doesn't need SSR
});

// Usage
function Dashboard() {
  return (
    <div>
      <h1>Analytics</h1>
      <HeavyChart data={chartData} />
    </div>
  );
}
```

### Conditional Loading
Load components only when needed:

```tsx
'use client';

function ProductDetail({ product }: ProductDetailProps) {
  const [showReviews, setShowReviews] = useState(false);
  
  const Reviews = useMemo(() => {
    if (!showReviews) return null;
    return dynamic(() => import('./reviews'), {
      loading: () => <ReviewsSkeleton />,
    });
  }, [showReviews]);
  
  return (
    <div>
      <h1>{product.name}</h1>
      <button onClick={() => setShowReviews(true)}>
        Show Reviews
      </button>
      {Reviews && <Reviews productId={product.id} />}
    </div>
  );
}
```

## Image Optimization

### Use Next.js Image Component
Optimize images automatically:

```tsx
import Image from 'next/image';

// ✅ Good: Optimized image
<Image
  src="/hero.jpg"
  alt="Hero image"
  width={1200}
  height={600}
  priority // For above-fold images
  placeholder="blur"
  blurDataURL="..." // Optional blur placeholder
/>

// ❌ Bad: Unoptimized image
<img src="/hero.jpg" alt="Hero image" />
```

### Image Best Practices
1. **Use WebP format**: Better compression
2. **Include size data**: Prevent layout shift
3. **Implement lazy loading**: Load images on demand
4. **Use placeholder**: Show blur while loading

```tsx
// Responsive images
<Image
  src="/product.jpg"
  alt="Product"
  width={500}
  height={500}
  sizes="(max-width: 768px) 100vw, 50vw"
  className="w-full h-auto"
/>

// Lazy loading (default for images below fold)
<Image
  src="/gallery/image.jpg"
  alt="Gallery image"
  width={400}
  height={300}
  loading="lazy"
/>
```

### Placeholder Images
For seed data, use https://placekitten.com/:

```tsx
// seed.ts
const placeholderImages = [
  'https://placekitten.com/400/300',
  'https://placekitten.com/400/400',
  'https://placekitten.com/600/400',
];
```

## Bundle Optimization

### Analyze Bundle
Regularly analyze your bundle size:

```bash
# Analyze bundle
ANALYZE=true npm run build
```

### Reduce Bundle Size
1. Tree shake unused code
2. Use named imports
3. Avoid large libraries
4. Use dynamic imports

```tsx
// ✅ Good: Named import
import { Button } from '@/components/ui/button';

// ❌ Bad: Default import of entire library
import UI from '@/components/ui';
```

## Caching Strategies

### Fetch Caching
Use Next.js fetch caching:

```tsx
// Revalidate every hour
const data = await fetch('/api/data', {
  next: { revalidate: 3600 }
});

// Static data
const static = await fetch('/api/static', {
  cache: 'force-cache'
});

// Dynamic data
const dynamic = await fetch('/api/dynamic', {
  cache: 'no-store'
});
```

### ISR (Incremental Static Regeneration)
Combine static and dynamic:

```tsx
// page.tsx
export const revalidate = 3600; // Revalidate every hour

export default async function Page() {
  const data = await fetch('/api/data');
  return <Dashboard data={data} />;
}
```

## Performance Checklist

### Before Deployment
- [ ] Use Server Components where possible
- [ ] Implement Suspense boundaries
- [ ] Use dynamic imports for heavy components
- [ ] Optimize all images with next/image
- [ ] Set appropriate caching headers
- [ ] Minimize client-side JavaScript

### Monitor
- [ ] Core Web Vitals (LCP < 2.5s, CLS < 0.1, FID < 100ms)
- [ ] Bundle size (< 200KB initial)
- [ ] Time to Interactive (< 3.8s)
- [ ] First Contentful Paint (< 1.8s)

## Best Practices Summary

1. ✅ Minimize 'use client', useEffect, setState
2. ✅ Favor React Server Components
3. ✅ Wrap client components in Suspense
4. ✅ Use dynamic imports for code splitting
5. ✅ Optimize images with next/image
6. ✅ Use WebP format
7. ✅ Implement lazy loading
8. ✅ Configure proper caching
9. ✅ Monitor Core Web Vitals
10. ✅ Regularly analyze bundle size
