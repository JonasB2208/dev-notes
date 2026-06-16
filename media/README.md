# Next.js Fullstack Starter

Moderner Fullstack-Stack mit:

- Next.js (App Router)
- TypeScript
- Tailwind CSS
- PostgreSQL
- Prisma ORM
- Auth.js (NextAuth)
- Zod

---

# Projektstruktur

```txt
my-app/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
│
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── register/
│   │   │
│   │   ├── dashboard/
│   │   ├── api/
│   │   │   └── auth/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   │
│   ├── components/
│   │   ├── ui/
│   │   └── shared/
│   │
│   ├── lib/
│   │   ├── prisma.ts
│   │   ├── auth.ts
│   │   ├── validations/
│   │   └── utils.ts
│   │
│   ├── actions/
│   ├── hooks/
│   └── types/
│
├── public/
│
├── .env
├── .env.example
├── package.json
├── next.config.ts
└── tsconfig.json
```

---

# Installation

## Projekt erstellen

```bash
npx create-next-app@latest my-app
```

Empfohlene Optionen:

```txt
TypeScript     → Yes
ESLint         → Yes
Tailwind       → Yes
src/ directory → Yes
App Router     → Yes
Turbopack      → Yes
```

---

# Dependencies

```bash
npm install prisma @prisma/client
npm install next-auth
npm install zod
npm install bcrypt
```

Optional:

```bash
npm install react-hook-form
npm install @hookform/resolvers
```

---

# Prisma Setup

Initialisierung:

```bash
npx prisma init
```

## .env

```env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"

AUTH_SECRET="your-secret"

GITHUB_ID=""
GITHUB_SECRET=""
```

---

# Prisma Client

`src/lib/prisma.ts`

```ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}
```

---

# Datenmodell

`prisma/schema.prisma`

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  password  String?
  name      String?
  createdAt DateTime @default(now())
}
```

Migration erstellen:

```bash
npx prisma migrate dev --name init
```

---

# Auth.js

`src/lib/auth.ts`

```ts
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";

export const {
  handlers,
  auth,
  signIn,
  signOut
} = NextAuth({
  providers: [
    GitHub({
      clientId: process.env.GITHUB_ID!,
      clientSecret: process.env.GITHUB_SECRET!,
    }),
  ],
});
```

`src/app/api/auth/[...nextauth]/route.ts`

```ts
import { handlers } from "@/lib/auth";

export const { GET, POST } = handlers;
```

---

# Zod Validation

`src/lib/validations/user.ts`

```ts
import { z } from "zod";

export const RegisterSchema = z.object({
  email: z.email(),
  password: z.string().min(8),
});
```

---

# Server Actions

`src/actions/register.ts`

```ts
"use server";

import { prisma } from "@/lib/prisma";
import { RegisterSchema } from "@/lib/validations/user";

export async function register(data: unknown) {
  const parsed = RegisterSchema.parse(data);

  await prisma.user.create({
    data: parsed,
  });
}
```

---

# Scripts

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:generate": "prisma generate"
  }
}
```

---

# .gitignore

```gitignore
node_modules
.next
.env
.env.local
```

---

# Development Workflow

Neue Features immer auf separaten Branches entwickeln:

```bash
git checkout -b feature/auth
git checkout -b feature/dashboard
git checkout -b feature/users
```

Nach Fertigstellung:

```bash
git add .
git commit -m "feat: add authentication"
git push
```

---

# Nächste Schritte

Sinnvolle Projekte zum Lernen:

- Todo App
- Habit Tracker
- Budget Tracker
- Kanban Board
- Notes App
- SaaS Dashboard

Diese Projekte nutzen den kompletten Stack und vermitteln reale Fullstack-Erfahrung mit Next.js.
