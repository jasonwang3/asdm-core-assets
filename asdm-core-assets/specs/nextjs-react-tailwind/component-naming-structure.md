# Component Naming & Directory Structure

Rules for naming components and structuring directories within the src/components folder, including conventions for lowercase names with dashes.

## Directory Structure

### Base Structure
All components should go in `src/components` and be named like `new-component.tsx`:

```
src/
├── app/
├── components/
│   ├── ui/
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   └── input.tsx
│   ├── forms/
│   │   ├── search-form.tsx
│   │   └── contact-form.tsx
│   └── layout/
│       ├── navbar.tsx
│       ├── footer.tsx
│       └── sidebar.tsx
└── lib/
```

### Naming Convention
- Use lowercase with dashes for file names
- Use PascalCase for component names

```
// ✅ Good
components/
├── auth-wizard/
│   ├── index.tsx
│   └── auth-wizard-form.tsx
├── user-profile/
│   └── index.tsx
└── product-card/
    └── index.tsx

// ❌ Bad
components/
├── AuthWizard/
├── UserProfile/
└── ProductCard/
```

## Component Organization

### By Type
Group components by their type:
```
src/components/
├── ui/              # Basic UI elements
│   ├── button/
│   ├── modal/
│   └── card/
├── forms/           # Form components
│   ├── text-field/
│   ├── select/
│   └── checkbox/
└── layout/          # Layout components
    ├── navbar/
    ├── footer/
    └── sidebar/
```

### By Feature
For larger applications, group by feature:
```
src/components/
├── auth/
│   ├── login-form/
│   ├── register-form/
│   └── password-reset/
├── products/
│   ├── product-list/
│   ├── product-card/
│   └── product-detail/
└── cart/
    ├── cart-item/
    ├── cart-summary/
    └── checkout-form/
```

## Export Patterns

### Named Exports
Favor named exports for components:

```tsx
// ✅ Good: Named export
export function Button({ children }: ButtonProps) {
  return <button>{children}</button>;
}

// ❌ Avoid: Default export
export default function Button({ children }: ButtonProps) {
  return <button>{children}</button>;
}
```

### Index Files
Use index files for cleaner imports:

```tsx
// components/ui/button/index.tsx
export { Button } from './button';
export type { ButtonProps } from './button.types';

// Usage
import { Button, type ButtonProps } from '@/components/ui/button';
```

## Component File Structure

### Single File Component
For simple components:
```
components/
└── button.tsx
```

```tsx
// button.tsx
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
}

export function Button({ children, onClick }: ButtonProps) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}
```

### Multi-File Component
For complex components:
```
components/
└── auth-wizard/
    ├── index.tsx              # Main component
    ├── auth-wizard-form.tsx   # Sub-component
    ├── auth-wizard.types.ts   # Types
    ├── auth-wizard.utils.ts   # Utilities
    └── auth-wizard.test.tsx   # Tests
```

## Shadcn UI Components

### Location
Shadcn UI components go in `components/ui/`:

```
components/
└── ui/
    ├── button.tsx
    ├── card.tsx
    ├── input.tsx
    ├── dialog.tsx
    └── dropdown-menu.tsx
```

### Customization
Extend Shadcn components in separate files:

```tsx
// components/ui/button.tsx (Shadcn)
import { Button as ShadcnButton } from '@/components/ui/button';

// components/custom-button.tsx (Custom)
import { Button } from '@/components/ui/button';

export function CustomButton({ variant, ...props }: CustomButtonProps) {
  return <Button variant={variant} {...props} />;
}
```

## Directory Organization Examples

### E-commerce Application
```
src/components/
├── ui/                    # Shadcn UI components
│   ├── button.tsx
│   ├── card.tsx
│   └── input.tsx
├── layout/               # Layout components
│   ├── navbar.tsx
│   ├── footer.tsx
│   └── sidebar.tsx
├── products/             # Product feature
│   ├── product-list.tsx
│   ├── product-card.tsx
│   └── product-detail.tsx
├── cart/                 # Cart feature
│   ├── cart-item.tsx
│   ├── cart-summary.tsx
│   └── checkout-form.tsx
└── auth/                 # Auth feature
    ├── login-form.tsx
    └── register-form.tsx
```

### SaaS Dashboard
```
src/components/
├── ui/                   # Shadcn UI components
├── layout/
│   ├── dashboard-layout.tsx
│   ├── sidebar.tsx
│   └── header.tsx
├── dashboard/
│   ├── stats-card.tsx
│   ├── recent-activity.tsx
│   └── charts.tsx
├── settings/
│   ├── profile-settings.tsx
│   └── notification-settings.tsx
└── shared/
    ├── data-table.tsx
    └── filter-bar.tsx
```

## Best Practices Summary

1. ✅ Place all components in `src/components`
2. ✅ Use lowercase with dashes for file names
3. ✅ Use PascalCase for component names
4. ✅ Favor named exports over default exports
5. ✅ Organize by type for small projects
6. ✅ Organize by feature for large projects
7. ✅ Use index files for cleaner imports
8. ✅ Keep Shadcn components in `ui/` directory
9. ✅ Create subdirectories for complex components
