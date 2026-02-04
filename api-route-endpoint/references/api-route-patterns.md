# API Route Patterns (apps/api)

## File layout

- Feature route folder:
  - `apps/api/src/routes/<group>/<feature>/<feature>.index.ts`
  - `apps/api/src/routes/<group>/<feature>/<feature>.handler.ts`
  - `apps/api/src/routes/<group>/<feature>/<feature>.schema.ts`

Example:

- `apps/api/src/routes/admin/members/members.index.ts`
- `apps/api/src/routes/admin/members/members.handler.ts`
- `apps/api/src/routes/admin/members/members.schema.ts`

## Route wiring

```ts
import { createRouter } from '@/api/app';
import { requireAuth } from '@/api/middlewares/auth.middleware';
import { requireAdminRole } from '@/api/middlewares/role.middleware';

import {
  createMember,
  getMember,
  getMembers,
  getMemberOptions,
  updateMember,
} from './members.handler';

const membersRoute = createRouter()
  .use('*', requireAuth)
  .use('*', requireAdminRole)
  .get('/', ...getMembers)
  .get('/options', ...getMemberOptions)
  .get('/:id', ...getMember)
  .post('/', ...createMember)
  .put('/:id', ...updateMember);

export default membersRoute;
```

## Handler pattern

```ts
import { createHandlers } from '@/api/app';
import { initializePrisma } from '@/api/lib/db';
import { validate } from '@/api/lib/validator';

import { memberListQuerySchema } from './members.schema';

export const getMembers = createHandlers(
  validate('query', memberListQuerySchema),
  async (c) => {
    const { search, page, limit } = c.req.valid('query');
    const prisma = initializePrisma(c.env.DATABASE_URL);
    const skip = (page - 1) * limit;

    const [members, total] = await Promise.all([
      prisma.member.findMany({ skip, take: limit }),
      prisma.member.count(),
    ]);

    return c.json({
      meta: { code: 200, message: 'Members retrieved successfully' },
      data: {
        members,
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
        },
      },
    }, 200);
  },
);
```

## Schema patterns

```ts
import { z } from 'zod';

export const memberIdParamSchema = z.object({
  id: z.string().trim().min(1),
});

export const memberListQuerySchema = z.object({
  search: z.string().trim().optional(),
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
});

export const memberCreateSchema = z.object({
  firstName: z.string().trim().min(1),
  lastName: z.string().trim().min(1),
});
```

## Standard response structure

Success:

```ts
return c.json({
  meta: { code: 200, message: 'Members retrieved successfully' },
  data: { members },
}, 200);
```

Validation errors (from `validate`):

```ts
return c.json({
  meta: { code: 422, message: 'Validation failed' },
  data: {
    name: result.error.name,
    message: result.error.message,
    issues: result.error.issues,
    cause: result.error?.cause,
  },
}, 422);
```

Not found:

```ts
return c.json({
  meta: { code: 404, message: 'Member not found' },
  data: {},
}, 404);
```

Conflict/constraint errors (Prisma):

```ts
if (error.code === 'P2002') {
  return c.json({
    meta: { code: 409, message: 'A member with this name already exists' },
    data: {},
  }, 409);
}
```

## Top-level route registration

Register new top-level routes in `apps/api/src/routes/index.ts`:

```ts
return app.route('/admin', adminRoutes);
```

## Shared helpers

- `createRouter()` and `createHandlers()` from `apps/api/src/app.ts`
- `validate()` from `apps/api/src/lib/validator.ts`
- `initializePrisma()` from `apps/api/src/lib/db.ts`
- `requireAuth` from `apps/api/src/middlewares/auth.middleware.ts`
- `requireAdminRole` from `apps/api/src/middlewares/role.middleware.ts`
