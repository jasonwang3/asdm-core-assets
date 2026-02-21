# ASDM Specs Repository

This directory contains general specifications (rules) for various technology stacks and development practices. These specs are automatically added by `asdm-bootstrapper` as needed.

## Overview

Specs in this directory provide:
- Coding standards and best practices
- Performance guidelines
- Testing strategies
- Architecture recommendations

## Directory Structure

```
specs/
├── specs-registry.json    # Registry file for Admin UI
├── reactjs/               # React.js development specs
│   ├── README.md
│   ├── reactjs-coding-standard.md
│   ├── reactjs-performance-guidelines.md
│   └── reactjs-testing-guidelines.md
├── nextjs-react-tailwind/ # Next.js React Tailwind specs
│   ├── README.md
│   ├── general-typescript-nextjs-rules.md
│   ├── nextjs-conventions-best-practices.md
│   ├── ui-styling-shadcn-tailwind.md
│   ├── component-naming-structure.md
│   ├── private-shared-components.md
│   └── performance-optimization.md
└── {technology}/         # Additional tech stacks
    └── README.md
```

## Available Specs

### React.js
- [Coding Standards](reactjs/reactjs-coding-standard.md)
- [Performance Guidelines](reactjs/reactjs-performance-guidelines.md)
- [Testing Guidelines](reactjs/reactjs-testing-guidelines.md)

### Next.js React Tailwind
- [General TypeScript & Next.js Rules](nextjs-react-tailwind/general-typescript-nextjs-rules.md)
- [Next.js Conventions & Best Practices](nextjs-react-tailwind/nextjs-conventions-best-practices.md)
- [UI & Styling with Shadcn UI and Tailwind](nextjs-react-tailwind/ui-styling-shadcn-tailwind.md)
- [Component Naming & Directory Structure](nextjs-react-tailwind/component-naming-structure.md)
- [Private vs Shared Components](nextjs-react-tailwind/private-shared-components.md)
- [Performance Optimization](nextjs-react-tailwind/performance-optimization.md)

## Adding New Specs

To add a new technology spec:

1. Create a new directory under `specs/`
2. Add README.md with overview and index
3. Add relevant specification documents
4. Update `specs-registry.json` with new entry

Example structure:
```
specs/
├── {tech-name}/
│   ├── README.md
│   ├── {tech-name}-coding-standard.md
│   ├── {tech-name}-performance-guidelines.md
│   └── {tech-name}-testing-guidelines.md
```

## Registry

The `specs-registry.json` file tracks all available specs and is used by the Admin UI to display spec details.

## Usage

AI agents automatically receive relevant specs based on the technology stack detected in the workspace. These specs help maintain consistent code quality and follow best practices.

## Integration

Specs integrate with:
- `asdm-bootstrapper`: Automatically adds specs to workspaces
- AI coding assistants: Provides context and guidelines
- Admin UI: Displays spec information and details

## Support

For questions about specs, visit [ASDM Platform](https://platform.asdm.ai).
