# AGENTS.md - Agentic Coding Guidelines for APITable

This file provides guidelines for AI agents working on the APITable codebase.

## Project Overview

APITable is a monorepo using pnpm workspaces, nx, TypeScript, React, and NestJS. It contains:

- Frontend packages: `datasheet`, `components`, `widget-sdk`, `core`, `ai-components`
- Backend packages: `room-server` (NestJS)
- Shared packages: `api-client`, `icons`, `i18n-lang`, `databus-client`

## Package Manager & Node

- **Package Manager**: pnpm (v8)
- **Node Version**: 16.15.0
- **Install dependencies**: `pnpm install`
- **Pre-install check**: Uses `only-allow pnpm` to enforce pnpm

---

## Build Commands

```bash
# Build all packages
pnpm build

# Build specific packages
pnpm build:web          # Build datasheet (frontend)
pnpm build:room-server # Build NestJS backend
pnpm build:api-client  # Build API client
pnpm build:datasheet   # Build datasheet package

# Start development servers
pnpm start:datasheet   # Start datasheet dev server
pnpm start:room-server # Start room-server dev server
pnpm start:components  # Start components dev server
pnpm start:core        # Start core dev server

# Shortcuts
pnpm sd  # start:datasheet
pnpm sr  # start:room-server
pnpm sss # start:components
```

---

## Test Commands

```bash
# Run all tests (excluding cypress)
pnpm test

# Run tests for specific packages
pnpm test:core           # @apitable/core
pnpm test:datasheet      # @apitable/datasheet
pnpm test:widget-sdk     # @apitable/widget-sdk
pnpm test:nest           # @apitable/room-server (NestJS)
pnpm test:ut:room        # Room server unit tests
pnpm test:ut:room:cov    # Room server unit tests with coverage

# Run single test file (within a package)
cd packages/core && pnpm test -- --testPathPattern=filename
cd packages/room-server && pnpm test -- --testPathPattern=filename

# Watch mode
pnpm test:watch          # Watch mode (core)
pnpm test:core:cov       # Coverage report

# Other test commands
pnpm cy:open             # Open Cypress UI
pnpm cy:run              # Run Cypress tests
```

For running a **single test**, use `--testPathPattern` or `--testNamePattern`:

```bash
# In package directory
pnpm test -- --testPathPattern=SpecificTest
pnpm test -- --testNamePattern="test description"
```

---

## Lint & Format Commands

```bash
# ESLint
pnpm lint           # Run full lint
pnpm lint:fix       # Auto-fix lint issues
pnpm lint:check     # Check without fixing (quiet mode)

# Package-specific lint
pnpm lint:datasheet

# Prettier
pnpm prettier:check  # Check formatting
pnpm prettier:fix    # Fix formatting

# Both together (recommended)
pnpm prettier:fix
```

---

## Code Style Guidelines

### TypeScript Configuration

- **TypeScript Version**: 4.8.2
- Strict typing preferred; avoid `any`, `!` non-null assertions, and `@ts-ignore`
- Use proper type annotations for function parameters and return types

### Naming Conventions

| Element    | Convention                 | Example                                    |
| ---------- | -------------------------- | ------------------------------------------ |
| Interfaces | PascalCase with `I` prefix | `IUser`, `IConfig`                         |
| Types      | PascalCase                 | `UserType`, `Result`                       |
| Enums      | PascalCase or UPPER_CASE   | `StatusEnum.ACTIVE` or `StatusEnum.active` |
| Variables  | camelCase                  | `userName`, `isActive`                     |
| Constants  | UPPER_CASE                 | `MAX_RETRIES`, `API_URL`                   |
| Functions  | camelCase                  | `getUser()`, `fetchData()`                 |
| Components | PascalCase                 | `UserCard`, `ButtonGroup`                  |
| Files      | kebab-case or PascalCase   | `user-service.ts`, `UserCard.tsx`          |

### Import Order (ESLint plugin-import)

1. External libraries (React, NestJS, etc.)
2. Internal packages (`@apitable/...`)
3. Relative imports (local modules)
4. Type imports (`import type { ... }`)

```typescript
// Good import order
import React from 'react';
import { useState, useEffect } from 'react';
import { Injectable } from '@nestjs/common';
import { UserService } from '@apitable/core';
import { MyComponent } from './components/MyComponent';
import type { IUserConfig } from './types';
```

### Formatting (Prettier + ESLint)

| Rule                   | Value                             |
| ---------------------- | --------------------------------- |
| Print width            | 150 characters                    |
| Tab width              | 2 spaces                          |
| Single quotes          | Yes (with exception for escaping) |
| Semicolons             | Yes                               |
| Trailing commas        | ES5 style                         |
| Object bracket spacing | Yes                               |
| JSX boolean values     | `disabled` not `disabled={true}`  |

### React Best Practices

- Use functional components with hooks
- Self-closing tags for empty elements: `<Component />` not `<Component></Component>`
- Use `useMemo`, `useCallback` for expensive computations
- Avoid inline functions in render when possible
- Prefer composition over inheritance

### Error Handling

- Use try-catch with specific error types
- Prefer typed errors over generic `Error`
- Never swallow errors without logging
- Use proper HTTP status codes in NestJS controllers

```typescript
// Good
try {
  await service.doSomething();
} catch (error) {
  if (error instanceof NotFoundException) {
    throw error;
  }
  this.logger.error('Failed to do something', error);
  throw new InternalServerErrorException('Operation failed');
}
```

### General Rules

- 2-space indentation (no tabs)
- No `console.log` in production code (use logger)
- Maximum 1 empty line between statements
- Maximum 150 characters per line
- Use `const` over `let`, never use `var`
- Prefer arrow functions for callbacks
- Use async/await over raw promises
- Always await async functions

---

## Architecture Notes

### Package Structure

```
packages/
├── core/           # Datasheet core logic (Redux, state management)
├── datasheet/      # Main React frontend application
├── components/     # Shared UI components (Storybook available)
├── widget-sdk/     # Widget development SDK
├── room-server/    # NestJS backend server
├── api-client/     # HTTP API client
├── icons/          # Icon components
└── i18n-lang/     # Internationalization
```

### State Management

- Core uses Redux with Redux Toolkit
- Use `createSlice` for new slices
- Use `createAsyncThunk` for async operations
- Follow existing patterns for selectors

### Testing Patterns

- Jest for unit tests
- React Testing Library for React components
- Cypress for E2E tests
- Follow existing test file naming: `*.spec.ts` or `*.test.ts`

---

## Important Notes

1. **Never commit secrets**: Don't commit `.env` files, credentials, or API keys
2. **Husky hooks**: Pre-commit linting is enabled via husky
3. **Workspace dependencies**: Use `workspace:*` for internal packages
4. **Commit messages**: Follow conventional commits (handled by husky)
