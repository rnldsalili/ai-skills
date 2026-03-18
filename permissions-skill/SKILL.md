---
name: permissions-skill
description: Implement and maintain authorization and permission features in this repo. Use this whenever the user asks to add or change roles, permissions, CASL ability rules, route authorization, access-management UI, permission-gated navigation, or frontend/backend permission checks. Also use it when permission logic should be refactored to follow the repo's enum-based authorization model.
---

# Permissions Skill

This repo uses a shared CASL-based permission system. The source of truth is the `@workspace/permissions` package and the app/API are expected to consume the same enums, helpers, and ability wiring.

## Core rules

- Use `PermissionModule`, `PermissionAction`, and `SystemRoleSlug` enums from `@workspace/permissions`.
- Do not introduce magic strings like `'view'`, `'loans'`, or `'super-admin'` in authorization code.
- Keep permission definitions centralized in `packages/permissions`.
- Treat `super-admin` as the full-access bypass role.
- Keep backend enforcement in place even if the UI hides unauthorized actions.

## Frontend conventions

- Use CASL React `Can` from `@workspace/permissions/react` for action-level rendering.
- If a user lacks permission for an action, do not render the control.
- Do not use permission-based `disabled` states.
- Keep non-permission disables only for actual UI state:
  - loading or pending submission
  - incomplete or invalid input
  - business constraints like protected system roles
- For route-level access, use the existing unauthorized state pattern when `VIEW` is missing.
- Prefer `useCan(PermissionModule.X, PermissionAction.Y)` for route/page checks and `Can` for local UI sections.

## Backend conventions

- Use `authorize(PermissionModule.X, PermissionAction.Y)` in route registration.
- Build abilities from the shared permission snapshot only.
- Validate incoming permission payloads against the shared enums and helpers from `@workspace/permissions`.
- Keep DB values as strings that correspond to enum values; do not invent alternate wire formats.

## Access management rules

- Roles are reusable and many-to-many with users.
- `super-admin` is protected and should not be editable/deletable through normal role management flows.
- Role permission matrices should render from the shared catalog.
- User-facing permission labels should come from `permissionCatalog`.

## Implementation checklist

When changing permissions:

1. Update `packages/permissions` first.
2. Update API route guards and validation to use the shared enums.
3. Update app route checks and `Can` wrappers.
4. Remove any permission-driven disabled UI introduced by the change.
5. Run:
   - `bun run typecheck`
   - `bun run lint`

## Good patterns

```ts
authorize(PermissionModule.LOANS, PermissionAction.VIEW)
```

```tsx
<Can I={PermissionAction.CREATE} a={PermissionModule.CLIENTS}>
  <Button>New Client</Button>
</Can>
```

```ts
const canViewDocuments = useCan(
  PermissionModule.DOCUMENTS,
  PermissionAction.VIEW,
);
```

## Avoid

- Inline permission strings
- Permission booleans passed deeply through multiple component layers when `Can` or `useCan` can be used locally
- Showing unauthorized actions in a disabled state
- Duplicating permission catalogs in app or API code
