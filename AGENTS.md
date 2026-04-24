# Developer Agent Guidelines (AGENTS.md)

Welcome, agent. This repository follows a strict modular architecture and high-quality design standards. Follow these rules for all contributions.

## 1. Environment & Tech Stack

- **Runtime/PM**: [Bun](https://bun.sh) (primary), Node.js (fallback).
- **Framework**: Next.js (App Router).
- **Language**: TypeScript (Strict mode).
- **Styling**: Tailwind CSS, CSS Modules, or Styled Components (depending on feature).
- **State/Logic**: Zod for validation, Server Actions for mutations.
- **Database**: Prisma or Drizzle (check `src/lib/db.ts`).

## 2. Critical Commands

Always use `bun` if `bun.lockb` or `bun.lock` is present.

### Build & Lint
- **Install**: `bun install`
- **Build**: `bun run build`
- **Lint**: `bun run lint`
- **Type Check**: `bun x tsc --noEmit`

### Testing
- **Run all tests**: `bun test`
- **Run single test**: `bun test <path-to-file>` (e.g., `bun test src/features/auth/auth.test.ts`)
- **Watch mode**: `bun test --watch`

## 3. Architecture: Feature-Sliced Design (FSD) Simplified

We use a modular structure in `src/`. Avoid placing logic in `src/app/` directly.

### Directory Structure
- `src/features/`: Domain-specific vertical slices (e.g., `auth`, `billing`).
  - `components/`: Feature-specific UI.
  - `actions.ts`: Server Actions for this domain.
  - `types.ts`: Zod schemas and TypeScript types.
  - `hooks/`: Feature-specific React hooks.
- `src/services/`: The Service Layer. Pure, transport-agnostic business logic.
- `src/app/`: Routing layer (App Router). Thin wrappers around features.
- `src/components/`:
  - `ui/`: Generic, reusable UI components (e.g., Shadcn).
  - `shared/`: Components shared across multiple features.
- `src/lib/`: Global utilities, constants, and DB clients.

### Co-location Rule
Code that belongs together should live together. If a component is only used by the `billing` feature, it must stay in `src/features/billing/components/`.

## 4. Code Style Guidelines

### Naming Conventions
- **Directories/Files**: `kebab-case` (e.g., `user-profile`, `auth-service.ts`).
- **React Components**: `PascalCase` (e.g., `UserProfile.tsx`).
- **Functions/Variables**: `camelCase`.
- **Server Actions**: Suffix with `Action` (e.g., `createPostAction`).
- **Constants**: `UPPER_SNAKE_CASE`.

### Imports
- Use absolute paths with the `@/` alias (e.g., `import { db } from '@/lib/db'`).
- Group imports: Built-ins -> External -> Internal (@/) -> Relative.
- Avoid deep relative paths (`../../../../`).

### Types & Validation
- **Always** use Zod for input validation (Server Actions, API routes).
- Prefer `interface` for object shapes, `type` for unions/aliases.
- Avoid `any`. Use `unknown` if a type is truly dynamic.

## 5. Backend Componentization (Service Layer)

- **Rule**: Never write direct DB queries in Route Handlers or Server Actions.
- **Service Layer**: All business logic and DB calls go into `src/services/`.
- **Pattern**:
  ```typescript
  // src/services/post-service.ts
  export const PostService = {
    async create(data: CreatePostDTO) {
      return await db.post.create({ data });
    }
  };
  ```

## 6. Server Actions & Mutations

- Use Server Actions (`'use server'`) for all mutations (POST/PUT/DELETE).
- **Workflow**:
  1. Validate input using Zod.
  2. Call the corresponding Service.
  3. Perform `revalidatePath` or `redirect` as needed.
  4. Return a standardized response object `{ success: boolean, data?: T, error?: string }`.

## 7. Frontend Design & Aesthetics

Avoid "AI Slop" aesthetics. Every UI should feel intentionally designed and visually striking.

- **Typography**: 
  - Choose fonts that are characterful and interesting. 
  - Avoid generic stacks like Arial/Inter/Roboto.
  - Pair a distinctive display font (e.g., Space Grotesk, Cal Sans) with a clean, readable body font.
- **Color & Theme**:
  - Use CSS variables for all colors.
  - Commit to a bold aesthetic direction (e.g., Brutalist, Minimalism, Retro-futurism).
  - Use high-contrast accents and avoid timid, muddy palettes.
- **Motion & Interaction**: 
  - Use `framer-motion` for React or CSS transitions for pure HTML/CSS.
  - Focus on "Delightful Micro-interactions": staggered reveals, smooth hover states, and meaningful transitions.
- **Layout**: 
  - Break the grid when appropriate. 
  - Use generous negative space or controlled density.
  - Experiment with asymmetry, overlapping elements, and diagonal flows.
- **Components**: 
  - Prefer Server Components for data-heavy parts. 
  - Use `'use client'` strictly for interactivity at the leaves.

## 8. Error Handling & Validation

- **Input Validation**: Use Zod for *every* input that crosses a boundary (Server Action, API, Form).
- **Service Errors**: Services should throw specific errors or return a Result type.
- **Action Responses**: Server Actions must return a consistent shape:
  ```typescript
  export type ActionResponse<T = any> = 
    | { success: true; data: T }
    | { success: false; error: string; issues?: z.ZodIssue[] };
  ```
- **UI Feedback**: Use `sonner` or similar toast libraries for success/error feedback. Always provide clear, actionable error messages to the user.

## 9. Implementation Workflow (Step-by-Step)

When implementing a new feature, follow this sequence:

1.  **Analyze**: Identify the domain and create a new feature directory in `src/features/`.
2.  **Scaffold**: Run `.opencode/skills/fullstack-architect/scripts/scaffold.sh` to generate the structure.
3.  **Schema**: Define the Zod schemas and database models first.
4.  **Service**: Implement the business logic in `src/services/`.
5.  **Action**: Create the Server Actions in `src/features/<feature>/actions.ts`.
6.  **UI Components**: Build the UI components, starting with Server Components where possible.
7.  **Verify**: Run `bun test` and `bun run lint`.

## 10. Agent Safety & Proactivity

- **Directory Verification**: Before creating files, verify the parent directory exists.
- **No Reverts**: Do not revert changes unless they cause errors.
- **Self-Correction**: If a build/lint/test fails, fix it immediately before proceeding.
- **Documentation**: Only create documentation if explicitly requested.

---
*This file is a living document. Update it as project conventions evolve.*
