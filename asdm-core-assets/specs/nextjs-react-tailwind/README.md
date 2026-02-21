# Next.js React Tailwind Development Specs

This directory contains coding standards, guidelines, and best practices for Next.js development with React, TypeScript, Tailwind CSS, and Shadcn UI.

## Overview

These specs provide comprehensive guidance for:
- TypeScript and Next.js App Router best practices
- Component architecture and organization
- UI styling with Tailwind CSS and Shadcn UI
- Performance optimization techniques

## Available Specifications

### 1. General TypeScript & Next.js Rules
**File**: [general-typescript-nextjs-rules.md](general-typescript-nextjs-rules.md)

Covers:
- Code style and patterns
- Naming conventions
- TypeScript usage guidelines
- Functional programming patterns

### 2. Next.js Conventions & Best Practices
**File**: [nextjs-conventions-best-practices.md](nextjs-conventions-best-practices.md)

Covers:
- App Router architecture
- Server vs Client Components
- Data fetching strategies
- URL state management with nuqs
- Web Vitals optimization

### 3. UI & Styling with Shadcn UI and Tailwind
**File**: [ui-styling-shadcn-tailwind.md](ui-styling-shadcn-tailwind.md)

Covers:
- Shadcn UI component usage
- Tailwind CSS styling patterns
- Responsive design principles
- Mobile-first approach

### 4. Component Naming & Directory Structure
**File**: [component-naming-structure.md](component-naming-structure.md)

Covers:
- Component file naming conventions
- Directory organization
- Export patterns
- Feature-based organization

### 5. Private vs Shared Components
**File**: [private-shared-components.md](private-shared-components.md)

Covers:
- Component placement guidelines
- Private component patterns
- Shared component organization

### 6. Performance Optimization
**File**: [performance-optimization.md](performance-optimization.md)

Covers:
- Server Components best practices
- Suspense boundaries
- Dynamic imports
- Image optimization

## Quick Reference

### Project Structure
```
src/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Route group for auth
│   │   └── login/
│   │       └── page.tsx
│   ├── layout.tsx
│   └── page.tsx
├── components/            # Shared components
│   ├── ui/               # Shadcn UI components
│   ├── forms/            # Form components
│   └── layout/           # Layout components
└── lib/                  # Utilities and helpers
```

### Component Template
```tsx
// Use functional components with TypeScript interfaces
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

export function Button({ 
  children, 
  onClick, 
  variant = 'primary' 
}: ButtonProps) {
  return (
    <button
      onClick={onClick}
      className={cn(
        "rounded-md px-4 py-2",
        variant === 'primary' && "bg-blue-500 text-white",
        variant === 'secondary' && "bg-gray-200 text-gray-800"
      )}
    >
      {children}
    </button>
  );
}
```

### Key Principles

1. **Server Components First**: Default to Server Components, use `'use client'` only when necessary
2. **Mobile-First Design**: Start with mobile styles, add responsive breakpoints
3. **Functional Patterns**: Prefer functional and declarative programming
4. **Descriptive Naming**: Use auxiliary verbs (isLoading, hasError, canEdit)
5. **Modular Organization**: Organize components by type or feature

## Tech Stack

- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **UI Components**: Shadcn UI
- **Animations**: Framer Motion
- **State Management**: nuqs (URL state)

## Related Resources

- [Next.js Documentation](https://nextjs.org/docs)
- [Tailwind CSS](https://tailwindcss.com)
- [Shadcn UI](https://ui.shadcn.com)
- [Framer Motion](https://www.framer.com/motion)
- [nuqs](https://nuqs.47ng.com)

## Version

Current version: 1.0.0

Last updated: 2024-01-01
