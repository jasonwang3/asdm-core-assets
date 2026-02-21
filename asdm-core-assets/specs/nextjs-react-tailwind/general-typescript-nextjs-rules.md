# General TypeScript & Next.js Rules

Rules for TypeScript, Node.js, and Next.js projects, covering code style, naming conventions, and TypeScript usage.

## Core Principles

### Expertise Areas
You should be proficient in:
- TypeScript
- Node.js
- Next.js App Router
- React
- Shadcn UI
- Tailwind CSS
- Framer Motion

## Code Style

### Writing Code
- Write concise, technical TypeScript code with accurate examples
- Use functional and declarative programming patterns
- Avoid classes; prefer functions
- Prefer iteration and modularization over code duplication

### Naming Conventions
- Use descriptive variable names with auxiliary verbs:
  - ✅ `isLoading`
  - ✅ `hasError`
  - ✅ `canEdit`
  - ✅ `shouldRender`
  - ❌ `loading`
  - ❌ `error`

## TypeScript Usage

### Types vs Interfaces
- Prefer interfaces over types for object shapes
- Use types for unions, intersections, and utility types

```tsx
// ✅ Good: Use interface for object shapes
interface User {
  id: string;
  name: string;
  email: string;
}

// ✅ Good: Use type for unions
type Status = 'loading' | 'success' | 'error';

// ✅ Good: Use type for utility types
type UserPartial = Partial<User>;
```

### Avoid Enums
- Use maps or objects instead of enums

```tsx
// ❌ Bad: Using enum
enum Role {
  Admin = 'admin',
  User = 'user',
}

// ✅ Good: Using object
const Role = {
  Admin: 'admin',
  User: 'user',
} as const;

type Role = typeof Role[keyof typeof Role];
```

### Functional Components
- Use functional components with TypeScript interfaces
- Always define prop interfaces

```tsx
// ✅ Good
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
}

export function Button({ children, onClick, disabled }: ButtonProps) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}
```

### Pure Functions
- Use the `function` keyword for pure functions
- Avoid arrow functions for top-level function declarations

```tsx
// ✅ Good
function formatPrice(amount: number): string {
  return `$${amount.toFixed(2)}`;
}

// ❌ Avoid for top-level functions
const formatPrice = (amount: number): string => {
  return `$${amount.toFixed(2)}`;
};
```

## Conditional Statements

### Concise Syntax
- Avoid unnecessary curly braces in conditionals
- Use concise syntax for simple statements

```tsx
// ✅ Good: Simple one-liner
if (isLoading) return <Spinner />;

// ✅ Good: Multiple statements need braces
if (error) {
  logError(error);
  return <ErrorFallback error={error} />;
}

// ❌ Bad: Unnecessary braces for single statement
if (isLoading) {
  return <Spinner />;
}
```

## JSX Style

### Declarative JSX
- Write declarative JSX
- Prefer early returns over nested conditionals

```tsx
// ✅ Good: Declarative with early returns
function UserCard({ user }: UserCardProps) {
  if (!user) return null;
  if (user.isBanned) return <BannedNotice />;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// ❌ Bad: Nested conditionals
function UserCard({ user }: UserCardProps) {
  return (
    <div>
      {user && !user.isBanned && (
        <>
          <h2>{user.name}</h2>
          <p>{user.email}</p>
        </>
      )}
    </div>
  );
}
```

## Best Practices Summary

1. ✅ Use TypeScript for all code
2. ✅ Prefer interfaces over types for objects
3. ✅ Use functional components
4. ✅ Avoid enums; use const objects
5. ✅ Use descriptive names with auxiliary verbs
6. ✅ Prefer iteration and modularization
7. ✅ Write declarative JSX
8. ✅ Use concise conditional syntax
9. ✅ Avoid classes; prefer functions
10. ✅ Use `function` keyword for pure functions
