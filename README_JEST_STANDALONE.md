# 🏢 Enterprise Next.js Application Architecture

This document describes the architecture, design principles, and folder structure for our enterprise-grade **Next.js** application.  
The goal is to ensure **scalability, maintainability, and performance** across multiple teams and feature domains.

---

## ⚙️ Core Principles

- **Modularity:** Features and concerns are separated into well-defined layers and modules.  
- **Scalability:** Designed to grow with new features, teams, and services.  
- **Maintainability:** Predictable folder structure and consistent patterns across the codebase.  
- **Type Safety:** Full TypeScript coverage across client and server.  
- **Performance:** Leverages Next.js Server Components, caching, ISR, and CDN layers.  
- **Security:** Encapsulated API routes and middleware for auth, validation, and logging.  
- **Observability:** Centralized logging, metrics, and error tracking (Sentry, OpenTelemetry).

---

## 🧱 Architecture Overview

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
  └── Logging, caching, email, monitoring

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

## 📂 Folder Structure

```
src/
  app/
    api/
      auth/
        route.ts
      users/
        route.ts
    (auth)/
      login/
        page.tsx
      register/
        page.tsx
    dashboard/
      layout.tsx
      page.tsx
      settings/
        page.tsx
    layout.tsx
    page.tsx

  components/
    ui/
      Button/
        index.tsx
        Button.test.tsx
      Modal/
    layout/
      Header.tsx
      Footer.tsx

  features/
    auth/
      components/
      hooks/
      services/
      types/
    profile/
      ...

  lib/
    httpClient.ts
    logger.ts
    cache.ts

  hooks/
    useUser.ts
    useFetch.ts

  services/
    authService.ts
    userService.ts

  repositories/
    userRepo.ts
    authRepo.ts

  domain/
    user/
      User.ts
      types.ts
      validators.ts

  usecases/
    registerUser.ts
    loginUser.ts
    fetchProfile.ts

  utils/
    date.ts
    format.ts

  config/
    env.ts
    featureFlags.ts

  middleware/
    authMiddleware.ts
    errorMiddleware.ts

tests/
  unit/
  integration/
  e2e/

public/
next.config.js
tsconfig.json
.env
```

---

## 💻 Example Code

### `src/usecases/loginUser.ts`
```ts
import { authRepo } from "../repositories/authRepo";
import { validateCredentials } from "../domain/auth/validators";

export async function loginUser(email: string, password: string) {
  validateCredentials(email, password);
  const user = await authRepo.login(email, password);
  if (!user) throw new Error("Invalid credentials");
  return user;
}
```

### `src/app/api/auth/route.ts`
```ts
import { NextRequest, NextResponse } from "next/server";
import { loginUser } from "@/usecases/loginUser";

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

### `src/hooks/useAuth.ts`
```ts
import useSWR from "swr";
import { fetcher } from "@/lib/httpClient";

export function useAuth() {
  const { data, error, mutate } = useSWR("/api/auth/me", fetcher);
  return {
    user: data,
    isLoading: !error && !data,
    isError: error,
    mutate,
  };
}
```

### `src/app/dashboard/page.tsx`
```tsx
import { redirect } from "next/navigation";
import { getCurrentUser } from "@/services/authService";

export default async function DashboardPage() {
  const user = await getCurrentUser();
  if (!user) redirect("/login");
  return <div>Welcome, {user.name}</div>;
}
```

---

## 🚀 Deployment & Scaling

- **Caching & ISR:** Use Incremental Static Regeneration and Edge Caching for optimal performance.  
- **Environment Config:** Centralized via `/src/config/env.ts` with strict schema validation.  
- **Monitoring:** Sentry / OpenTelemetry integration for logs, metrics, and traces.  
- **Feature Flags:** Enable gradual rollout of new features (`/src/config/featureFlags.ts`).  
- **CI/CD:** Automated pipelines for testing, linting, and deployment (GitHub Actions / CircleCI).  
- **Microfrontends (optional):** Each `feature/` can evolve into an independently deployable module.  
- **Monorepo Ready:** Supports Turborepo for shared packages, APIs, and UI libraries.

---

## 🧪 Testing Strategy

| Type          | Framework | Location             | Description                             |
|----------------|------------|----------------------|-----------------------------------------|
| Unit Tests     | Jest / Vitest | `tests/unit/`        | Tests individual functions & components |
| Integration    | Playwright / Supertest | `tests/integration/` | API route and service layer testing     |
| End-to-End     | Cypress / Playwright | `tests/e2e/`         | Full browser simulation of user flows   |

---

## 🧠 Tech Stack

- **Framework:** [Next.js 14+ (App Router)](https://nextjs.org)  
- **Language:** TypeScript  
- **UI Library:** React + TailwindCSS  
- **State Management:** SWR / React Query / Context / Zustand  
- **API Layer:** Next.js Route Handlers (BFF)  
- **Database Layer:** Prisma / REST / GraphQL clients (pluggable)  
- **Testing:** Jest, Playwright  
- **CI/CD:** GitHub Actions  
- **Monitoring:** Sentry / OpenTelemetry

---

## 📖 References

- [Next.js Docs](https://nextjs.org/docs)  
- [Clean Architecture Concepts](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html)  
- [Next.js Enterprise Structure – Dennis O’Keeffe](https://blog.dennisokeeffe.com/blog/2021-12-06-nextjs-enterprise-project-structure)  
- [Design Patterns for Scalable Next.js Applications](https://dev.to/nithya_iyer/5-design-patterns-for-building-scalable-nextjs-applications-1c80)

---

## 🧩 Author & Ownership

Architectural maintained by: **Valentyn Yakymenko**  
Contact: `vale.yakymenko@gmail.com`  

---

> _“Architecture is not about frameworks, it's about boundaries.”_ — Uncle Bob Martin
