---
name: backend-elysia
description: Elysia web framework patterns with Drizzle ORM and Better Auth
---

# Backend — Elysia + Drizzle + Better Auth

## Project Structure (Escalable)

```
apps/backend/src/
├── modules/              # Domain modules (DDD-lite)
│   ├── users/
│   │   ├── repository.ts # Port interface
│   │   ├── service.ts    # Business logic
│   │   ├── routes.ts     # API endpoints
│   │   └── index.ts      # Module aggregation
│   └── orders/
│       ├── repository.ts
│       ├── service.ts
│       ├── routes.ts
│       └── index.ts
├── core/                 # Shared code
│   ├── errors/          # Custom error classes
│   ├── ports/           # Abstract interfaces
│   ├── utils/           # Helpers
│   └── constants.ts
├── db/                   # Database
│   ├── client.ts        # Drizzle client
│   ├── schema.ts        # Table definitions
│   └── migrations/
├── plugins/              # Elysia plugins
│   ├── cors.ts
│   ├── rate-limit.ts
│   └── auth.ts
├── config/               # Environment config
└── index.ts             # Entry point
```

### When to Use This Structure

| Project Size | Structure |
|--------------|-----------|
| < 10 archivos | Todo en `routes/` + `services/` |
| 10-50 archivos | `modules/` sin subcarpetas |
| 50+ archivos | Estructura completa arriba |

**No sobre-ingenierices.** Empieza simple, agrega capas cuando las necesites.

---

## Module Pattern (DDD-lite)

Cada módulo tiene su propia lógica encapsulada:

```typescript
// modules/users/index.ts
import { usersRoutes } from './routes'
import { userService } from './service'

export const usersModule = (service = userService) => usersRoutes(service)

export { userService } from './service'
export type { UserRepository } from './repository'

// modules/users/repository.ts (Port - interfaz)
import type { User, NewUser } from '@/db/schema'

export interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(data: NewUser): Promise<User>
  update(id: string, data: Partial<NewUser>): Promise<User>
  delete(id: string): Promise<void>
}

// modules/users/repository.impl.ts (Adapter - implementación)
import { eq } from 'drizzle-orm'
import { db } from '@/db/client'
import { users } from '@/db/schema'
import type { UserRepository } from './repository'

export class DrizzleUserRepository implements UserRepository {
  async findById(id: string) {
    return db.query.users.findFirst(eq(users.id, id))
  }

  async findByEmail(email: string) {
    return db.query.users.findFirst(eq(users.email, email))
  }

  async create(data: NewUser) {
    const [user] = await db.insert(users).values(data).returning()
    return user
  }

  async update(id: string, data: Partial<NewUser>) {
    const [user] = await db.update(users).set(data).where(eq(users.id, id)).returning()
    return user
  }

  async delete(id: string) {
    await db.delete(users).where(eq(users.id, id))
  }
}

// modules/users/service.ts (Use case)
import type { UserRepository } from './repository'
import { db } from '@/db/client'

export function createUserService(repository: UserRepository) {
  return {
    async getById(id: string) {
      const user = await repository.findById(id)
      if (!user) throw new UserNotFoundError(id)
      return user
    },

    async getByEmail(email: string) {
      return repository.findByEmail(email)
    },

    async create(data: { email: string; name: string }) {
      const existing = await repository.findByEmail(data.email)
      if (existing) throw new UserAlreadyExistsError(data.email)

      return repository.create({
        id: crypto.randomUUID(),
        email: data.email,
        name: data.name,
        role: 'user',
        createdAt: new Date()
      })
    },

    async delete(id: string) {
      const user = await repository.findById(id)
      if (!user) throw new UserNotFoundError(id)
      await repository.delete(id)
    }
  }
}

// modules/users/routes.ts (Controller/Adapter)
import { Elysia, t } from 'elysia'
import type { UserService } from './service'

export const usersRoutes = (service: UserService) => Elysia({ prefix: '/users' })
  .get('/', async ({ query }) => {
    const { email } = query
    if (email) return service.getByEmail(email as string)
    return service.getAll()
  })
  .get('/:id', async ({ params }) => service.getById(params.id))
  .post('/', async ({ body }) => service.create(body as any), {
    body: t.Object({
      email: t.String({ format: 'email' }),
      name: t.String({ minLength: 1 })
    })
  })
  .delete('/:id', async ({ params }) => service.delete(params.id))
```

---

## Core Layer (Errores y Utilidades)

```typescript
// core/errors/index.ts
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message)
    this.name = 'AppError'
  }
}

export class UserNotFoundError extends AppError {
  constructor(id: string) {
    super(`User ${id} not found`, 'USER_NOT_FOUND', 404)
  }
}

export class UserAlreadyExistsError extends AppError {
  constructor(email: string) {
    super(`User with email ${email} already exists`, 'USER_EXISTS', 400)
  }
}

export class UnauthorizedError extends AppError {
  constructor() {
    super('Unauthorized', 'UNAUTHORIZED', 401)
  }
}

// Error handler global
app.onError(({ code, error }) => {
  if (error instanceof AppError) {
    return {
      error: error.message,
      code: error.code
    }
  }
  console.error(error)
  return { error: 'Internal server error', code: 'INTERNAL_ERROR' }
})
```

---

## DB Schema (Multi-file)

```typescript
// db/schema.ts
import { pgTable, uuid, varchar, timestamp, pgEnum } from 'drizzle-orm/pg-core'

// Enums
export const roleEnum = pgEnum('role', ['admin', 'user'])
export const orderStatusEnum = pgEnum('order_status', ['pending', 'completed', 'cancelled'])

// Users
export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 100 }),
  role: roleEnum('role').default('user'),
  createdAt: timestamp('created_at').defaultNow()
})

// Orders (relacionado con users)
export const orders = pgTable('orders', {
  id: uuid('id').primaryKey().defaultRandom(),
  userId: uuid('user_id').references(() => users.id).notNull(),
  total: varchar('total', { length: 20 }).notNull(),
  status: orderStatusEnum('status').default('pending'),
  createdAt: timestamp('created_at').defaultNow()
})

// Types
export type User = typeof users.$inferSelect
export type NewUser = typeof users.$inferInsert
export type Order = typeof orders.$inferSelect
export type NewOrder = typeof orders.$inferInsert
```

---

## Dependency Injection Simple

```typescript
// Inyección básica - sin contenedor complejo
import { DrizzleUserRepository } from './modules/users/repository.impl'
import { createUserService } from './modules/users/service'

// Instancias singleton
const userRepository = new DrizzleUserRepository()
const userService = createUserService(userRepository)

// Wiring en index.ts
const app = new Elysia()
  .use(usersModule(userService))
  .use(ordersModule(ordersService))
  .listen(3000)
```

---

## Legacy Structure (Solo para proyectos existentes)

Si tenés el proyecto viejo, no refactorices todo de golpe. Dejá la estructura vieja:

```
apps/backend/src/
├── routes/           # Endpoints
├── services/         # Lógica de negocio
├── db/
│   ├── index.ts
│   └── schema.ts
├── plugins/
├── lib/
└── index.ts
```

**Refactor gradual:** Cuando toque agregar algo a `routes/users.ts`, mové esa lógica a `modules/users/` siguiendo el patrón nuevo. See `refactor` skill para cómo hacerlo sin romper.

## Elysia Basic Pattern

```typescript
import { Elysia, t } from 'elysia'

const app = new Elysia()
  .use(corsPlugin)
  .use(rateLimitPlugin)
  .use(authPlugin)
  .use(userRoutes)
  .use(authRoutes)
  .listen(3000)
```

## Drizzle Schema Pattern

```typescript
// db/schema.ts
import { pgTable, uuid, varchar, timestamp, pgEnum } from 'drizzle-orm/pg-core'

export const roleEnum = pgEnum('role', ['admin', 'user'])

export const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 100 }),
  role: roleEnum('role').default('user'),
  createdAt: timestamp('created_at').defaultNow()
})

export type User = typeof users.$inferSelect
export type NewUser = typeof users.$inferInsert
```

## Better Auth Setup

```typescript
// plugins/auth.ts
import { betterAuth } from 'better-auth'
import { drizzleAdapter } from 'better-auth/adapters/drizzle'
import { db } from '../db'

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: 'pg' }),
  emailAndPassword: { enabled: true },
  socialProviders: {
    github: { clientId: process.env.GITHUB_ID, clientSecret: process.env.GITHUB_SECRET }
  }
})
```

## Eden API Generation

```typescript
// Generate typed client for frontend
// Run: bunx elysia-eden
// Output: frontend/src/lib/backend.ts
```

Then frontend imports:
```typescript
import { client } from '@/backend'
const { users } = client
```

## Common Patterns

### Protected Route
```typescript
app.get('/profile', ({ cookie }) => {
  const session = cookie['better-auth'].value
  return getProfile(session)
})
```

### Validation with TypeBox
```typescript
app.post('/users', ({ body }) => createUser(body), {
  body: t.Object({
    email: t.String({ format: 'email' }),
    name: t.String({ minLength: 1 })
  })
})
```

### Error Handling
```typescript
app.onError(({ code, error }) => {
  if (code === 'NOT_FOUND') return { error: 'Not found' }
  return { error: 'Internal error' }
})
```

## Security

- CORS: Restrict to `FRONTEND_URL`
- Rate limit: `@elysiajs/rate-limit`
- Input: TypeBox validation always
- Auth: HttpOnly cookie with opaque token