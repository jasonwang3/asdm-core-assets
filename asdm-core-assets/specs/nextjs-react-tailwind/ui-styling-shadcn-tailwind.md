# UI & Styling with Shadcn UI and Tailwind

Rules for UI development and styling using Shadcn UI and Tailwind CSS, emphasizing responsive design and a mobile-first approach.

## Core Technologies

### Shadcn UI
- Collection of reusable components
- Built on Radix UI primitives
- Styled with Tailwind CSS
- Copy-paste approach (not a package)

### Tailwind CSS
- Utility-first CSS framework
- Mobile-first responsive design
- JIT compiler for optimized bundles

## Styling Principles

### Use Shadcn UI and Tailwind
- Use Shadcn UI for pre-built components
- Use Tailwind for custom styling
- Combine both for maximum flexibility

```tsx
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader } from '@/components/ui/card';

export function ProductCard({ product }: ProductCardProps) {
  return (
    <Card className="hover:shadow-lg transition-shadow">
      <CardHeader>{product.name}</CardHeader>
      <CardContent>
        <p className="text-sm text-gray-600">{product.description}</p>
        <Button className="mt-4 w-full">Add to Cart</Button>
      </CardContent>
    </Card>
  );
}
```

## Responsive Design

### Mobile-First Approach
Start with mobile styles, then add responsive breakpoints:

```tsx
// ✅ Good: Mobile-first responsive design
<div className="
  p-4              // Mobile: padding 4
  md:p-6           // Tablet: padding 6
  lg:p-8           // Desktop: padding 8
">
  <h1 className="
    text-xl         // Mobile: text-xl
    md:text-2xl     // Tablet: text-2xl
    lg:text-3xl     // Desktop: text-3xl
  ">
    Welcome
  </h1>
</div>
```

### Responsive Breakpoints
Tailwind CSS breakpoints:
- `sm`: 640px
- `md`: 768px
- `lg`: 1024px
- `xl`: 1280px
- `2xl`: 1536px

### Responsive Patterns
```tsx
// Stack on mobile, row on desktop
<div className="flex flex-col md:flex-row gap-4">
  <div className="w-full md:w-1/2">Left</div>
  <div className="w-full md:w-1/2">Right</div>
</div>

// Hide/show on different screens
<div className="hidden md:block">Desktop only</div>
<div className="block md:hidden">Mobile only</div>

// Grid responsive
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {items.map(item => <Card key={item.id} />)}
</div>
```

## Component Styling

### Using cn() Utility
Combine class names with conditionals:

```tsx
import { cn } from '@/lib/utils';

interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
}

export function Button({ 
  variant = 'primary', 
  size = 'md',
  className 
}: ButtonProps) {
  return (
    <button
      className={cn(
        // Base styles
        "rounded-md font-medium transition-colors",
        // Variant styles
        variant === 'primary' && "bg-blue-500 text-white hover:bg-blue-600",
        variant === 'secondary' && "bg-gray-200 text-gray-800 hover:bg-gray-300",
        variant === 'danger' && "bg-red-500 text-white hover:bg-red-600",
        // Size styles
        size === 'sm' && "px-3 py-1.5 text-sm",
        size === 'md' && "px-4 py-2 text-base",
        size === 'lg' && "px-6 py-3 text-lg",
        // Custom override
        className
      )}
    >
      Click me
    </button>
  );
}
```

## Common Patterns

### Card Component
```tsx
<Card className="w-full max-w-sm">
  <CardHeader>
    <CardTitle>Card Title</CardTitle>
    <CardDescription>Card description</CardDescription>
  </CardHeader>
  <CardContent>
    <p>Card content goes here</p>
  </CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>
```

### Form Layout
```tsx
<form className="space-y-4">
  <div className="space-y-2">
    <Label htmlFor="email">Email</Label>
    <Input id="email" type="email" placeholder="you@example.com" />
  </div>
  <Button type="submit" className="w-full">
    Submit
  </Button>
</form>
```

### Layout Patterns
```tsx
// Page container
<div className="container mx-auto px-4 py-8">
  {/* Content */}
</div>

// Section
<section className="py-12 md:py-24">
  <div className="container">
    {/* Section content */}
  </div>
</section>

// Flex center
<div className="flex items-center justify-center min-h-screen">
  {/* Centered content */}
</div>
```

## Tailwind Best Practices

### Do's
- ✅ Use Tailwind utility classes
- ✅ Follow mobile-first approach
- ✅ Extract repeated patterns to components
- ✅ Use `cn()` for conditional classes
- ✅ Use Shadcn UI components as base

### Don'ts
- ❌ Use inline styles
- ❌ Create custom CSS classes for Tailwind utilities
- ❌ Use arbitrary values excessively
- ❌ Ignore responsive design

## Theme Customization

### Tailwind Config
```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#...',
          // ...
          900: '#...',
        },
      },
    },
  },
};
```

### CSS Variables
Shadcn UI uses CSS variables for theming:
```css
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  /* ... */
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ... */
}
```

## Best Practices Summary

1. ✅ Use Shadcn UI for pre-built components
2. ✅ Style with Tailwind CSS
3. ✅ Start with mobile-first approach
4. ✅ Use responsive breakpoints (sm, md, lg, xl, 2xl)
5. ✅ Use `cn()` for conditional styling
6. ✅ Extract repeated patterns to components
7. ✅ Customize theme via Tailwind config
8. ✅ Use CSS variables for theming
