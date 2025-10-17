# 🏢 Enterprise Next.js Application Architecture (with Nx & Vitest)

This document describes the architecture, design principles, and folder structure for our enterprise-grade **Next.js** application powered by **Nx** and **Vitest**.  
The goal is to ensure **scalability, maintainability, and performance** across multiple teams, domains, and shared libraries.

---

## ⚙️ Core Principles

- **Monorepo First:** Nx for workspace orchestration and dependency graphing.  
- **Modularity:** Clear domain separation via `apps/` and `libs/` boundaries.  
- **Scalability:** Built for multiple teams, environments, and applications.  
- **Type Safety:** Full TypeScript coverage across client and server.  
- **Performance:** SSR with Server Components, edge caching, and ISR.  
- **Testing Excellence:** Fast and isolated testing using Vitest.  
- **Security:** Encapsulated APIs and middleware for auth and validation.  
- **Observability:** Centralized logging, metrics, and error tracking.

---

## 🧱 Architectural Overview

```
Presentation Layer (UI)
  ├── Pages / Routes / Layouts
  ├── Components, UI primitives
  └── Client-side state / hooks

Application / Domain Layer (Business Logic)
  ├── Use cases, services, orchestration
  ├── Domain entities, validation, and rules
  └── Application interfaces (ports)

Infrastructure Layer (Adapters)
  ├── Repositories, data access (DB / APIs)
  ├── Integrations (HTTP, GraphQL, storage)
  └── Logging, caching, monitoring

Interface Layer (Boundaries)
  ├── Next.js API routes / Route handlers
  ├── Middleware (auth, validation)
  └── Controllers invoking the Application Layer

Cross-cutting Concerns
  ├── Config / Environment handling
  ├── Error handling / Observability
  ├── Shared utilities / helpers
  └── Design system and UI components
```

---

## 🧰 Nx Monorepo Setup

**Nx** manages our monorepo with multiple apps and libraries:

```
apps/
  web/             → Main Next.js app
  admin/           → Admin dashboard app

libs/
  ui/              → Shared design system components
  domain/          → Business logic and types
  utils/           → Shared utilities
  config/          → Environment and feature flags
```

### Benefits
- **Dependency Graph Visualization** (`npx nx graph`)
- **Smart Task Orchestration** (`npx nx affected:build`)
- **Incremental & Cached Builds** (Nx Cloud)
- **Reusable Generators & Executors**
- **Clear Ownership Across Teams**

### Common Commands
```bash
npx nx graph
npx nx affected:test
npx nx run web:serve
npx nx run admin:build
```

---

## 🧪 Testing with Vitest

**Vitest** provides ultra-fast and modern testing for Next.js apps.

### Features
- ⚡ Blazing fast runs with Vite under the hood  
- 🧩 TypeScript support out of the box  
- 🧠 Jest-compatible API (describe, it, expect)  
- 🧪 Integration with React Testing Library  
- 🧱 Works seamlessly with Nx targets  

### Example `vitest.config.ts`
```ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './tests/setup.ts',
    coverage: {
      reporter: ['text', 'html'],
    },
  },
});
```

### Example Command
```bash
npx nx run web:test
```

---

## 📂 Folder Structure

```
apps/
  web/
    src/
      app/
        api/
          auth/
            route.ts
          users/
            route.ts
        dashboard/
          page.tsx
          layout.tsx
        layout.tsx
        page.tsx
      components/
        ui/
          Button/
            index.tsx
            Button.test.tsx
        layout/
          Header.tsx
          Footer.tsx
      hooks/
        useUser.ts
        useFetch.ts
      services/
        authService.ts
        userService.ts
      config/
        env.ts
        featureFlags.ts
      middleware/
        authMiddleware.ts
        errorMiddleware.ts

libs/
  ui/
    Button.tsx
    Modal.tsx
  domain/
    user/
      User.ts
      validators.ts
  utils/
    format.ts
    date.ts
  config/
    env.ts
    featureFlags.ts

tests/
  unit/
  integration/
  e2e/

nx.json
package.json
tsconfig.base.json
vite.config.ts
```

---

## 💻 Example Code

### `src/usecases/loginUser.ts`
```ts
import { authRepo } from "@org/domain/authRepo";
import { validateCredentials } from "@org/domain/auth/validators";

export async function loginUser(email: string, password: string) {
  validateCredentials(email, password);
  const user = await authRepo.login(email, password);
  if (!user) throw new Error("Invalid credentials");
  return user;
}
```

### `apps/web/src/app/api/auth/route.ts`
```ts
import { NextRequest, NextResponse } from "next/server";
import { loginUser } from "@org/domain/usecases/loginUser";

export async function POST(req: NextRequest) {
  const { email, password } = await req.json();
  try {
    const user = await loginUser(email, password);
    return NextResponse.json({ user });
  } catch (err: any) {
    return NextResponse.json({ error: err.message }, { status: 400 });
  }
}
```

---

## 🚀 Deployment & Scaling

- **Incremental Builds:** Nx Cloud caching and remote execution.  
- **Environment Config:** Strict validation via `libs/config/env.ts`.  
- **Feature Flags:** Controlled rollout in `libs/config/featureFlags.ts`.  
- **Monitoring:** Sentry / OpenTelemetry for metrics and tracing.  
- **CI/CD:** Nx + GitHub Actions for affected pipelines only.  
- **Microfrontends:** Independent `apps/` deploys via Nx projects.  
- **Monorepo Governance:** Shared linting, formatting, and type rules.

---

## 🧠 Tech Stack

| Layer | Technology | Purpose |
|-------|-------------|----------|
| Framework | **Next.js 14+ (App Router)** | SSR, static generation, edge rendering |
| Monorepo | **Nx** | Workspace orchestration, caching, and builds |
| Language | **TypeScript** | Static typing and contracts |
| UI | **React + TailwindCSS** | UI component system |
| State | **SWR / React Query / Zustand** | Async and local state |
| Testing | **Vitest + Playwright** | Unit, integration, e2e |
| API Layer | **Next.js Route Handlers / tRPC** | BFF-style communication |
| Database | **Prisma / REST / GraphQL** | ORM and API adapters |
| CI/CD | **Nx Cloud + GitHub Actions** | Scalable pipelines |
| Monitoring | **Sentry / OpenTelemetry** | Logging and tracing |

---

## 🧩 Author & Ownership

Architectural maintained by: **Valentyn Yakymenko**  
Contact: `vale.yakymenko@gmail.com`

---

> _“Architecture is not about frameworks, it's about boundaries.”_ — Uncle Bob Martin
