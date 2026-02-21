# Private vs Shared Components

Rules for determining if a component should be private or shared, and where to place them based on their use-case.

## Component Categories

### Private Components
Components used only within specific pages or features.

### Shared Components
Reusable components used across multiple pages or features.

## Private Components

### Location
Create a `_components` folder within the relevant `/app` subdirectory:

```
src/app/
├── (auth)/
│   ├── login/
│   │   ├── page.tsx
│   │   └── _components/
│   │       ├── login-form.tsx
│   │       └── social-login.tsx
│   └── register/
│       ├── page.tsx
│       └── _components/
│           └── register-form.tsx
└── dashboard/
    ├── page.tsx
    └── _components/
        ├── stats-card.tsx
        └── recent-activity.tsx
```

### When to Use Private Components
- Component is only used in one page/route
- Component is tightly coupled to specific page logic
- Component doesn't need to be shared

### Example
```tsx
// src/app/dashboard/_components/stats-card.tsx
interface StatsCardProps {
  title: string;
  value: string;
  change: string;
}

export function StatsCard({ title, value, change }: StatsCardProps) {
  return (
    <Card>
      <CardHeader>{title}</CardHeader>
      <CardContent>
        <div className="text-2xl font-bold">{value}</div>
        <div className="text-sm text-gray-500">{change}</div>
      </CardContent>
    </Card>
  );
}

// src/app/dashboard/page.tsx
import { StatsCard } from './_components/stats-card';

export default function DashboardPage() {
  return (
    <div className="grid grid-cols-3 gap-4">
      <StatsCard title="Users" value="1,234" change="+12%" />
      <StatsCard title="Revenue" value="$12,345" change="+8%" />
      <StatsCard title="Orders" value="456" change="+5%" />
    </div>
  );
}
```

## Shared Components

### Location
The `/src/components` folder should contain reusable components:

```
src/components/
├── ui/              # Shadcn UI components
│   ├── button.tsx
│   ├── card.tsx
│   └── input.tsx
├── forms/           # Shared form components
│   ├── search-form.tsx
│   └── date-picker.tsx
└── layout/          # Shared layout components
    ├── navbar.tsx
    └── footer.tsx
```

### When to Use Shared Components
- Component is used across multiple pages
- Component is generic and reusable
- Component is a UI primitive (button, input, etc.)

### Example
```tsx
// src/components/ui/card.tsx (Shadcn UI)
export function Card({ children }: CardProps) {
  return <div className="rounded-lg border bg-card">{children}</div>;
}

// Can be used in multiple places:
// - src/app/dashboard/_components/stats-card.tsx
// - src/app/products/product-card.tsx
// - src/app/users/user-card.tsx
```

## Decision Flow

```
Is the component used in only one page/feature?
├── Yes → Use private component in _components/
│          src/app/feature/_components/component.tsx
│
└── No → Is it a UI primitive (button, input)?
    ├── Yes → Place in src/components/ui/
    │
    └── No → Is it a form component?
        ├── Yes → Place in src/components/forms/
        │
        └── No → Is it a layout component?
            ├── Yes → Place in src/components/layout/
            │
            └── No → Create feature folder in src/components/feature/
```

## Examples

### Private Component Example
```tsx
// src/app/products/[id]/_components/product-gallery.tsx
// Only used in product detail page

interface ProductGalleryProps {
  images: string[];
}

export function ProductGallery({ images }: ProductGalleryProps) {
  const [selectedImage, setSelectedImage] = useState(0);
  
  return (
    <div className="space-y-4">
      <img src={images[selectedImage]} alt="Product" />
      <div className="flex gap-2">
        {images.map((image, index) => (
          <button onClick={() => setSelectedImage(index)}>
            <img src={image} alt={`Thumbnail ${index}`} />
          </button>
        ))}
      </div>
    </div>
  );
}
```

### Shared Component Example
```tsx
// src/components/forms/search-form.tsx
// Used across multiple pages (products, users, orders)

interface SearchFormProps {
  placeholder?: string;
  onSearch: (query: string) => void;
}

export function SearchForm({ placeholder = "Search...", onSearch }: SearchFormProps) {
  const [query, setQuery] = useState('');
  
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      onSearch(query);
    }}>
      <Input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder={placeholder}
      />
    </form>
  );
}
```

## Migration Path

### When to Move Private → Shared
If a private component starts being used in multiple places:

1. **Identify usage**: Check where the component is imported
2. **Generalize**: Make component more flexible with props
3. **Move**: Relocate to `src/components/`
4. **Update imports**: Fix import paths in all files
5. **Test**: Verify all usages work correctly

### Migration Example
```tsx
// Before: Private component
// src/app/dashboard/_components/stats-card.tsx

// After: Shared component
// src/components/stats/stats-card.tsx

// Update imports in:
// - src/app/dashboard/page.tsx
// - src/app/analytics/page.tsx (new usage)
```

## Best Practices Summary

1. ✅ Use `_components/` for private components
2. ✅ Use `src/components/` for shared components
3. ✅ Private components are page-specific
4. ✅ Shared components are reusable
5. ✅ Start with private, migrate to shared when needed
6. ✅ Keep UI primitives in `components/ui/`
7. ✅ Use underscore prefix for `_components` directories
8. ✅ Review component usage periodically
