# TypeScript Migration Patterns

This document captures common patterns encountered during the JS-to-TS migration and the recommended approaches for handling them.

---

## 1. PropTypes → TypeScript Interfaces

### Before (JavaScript)
```jsx
import PropTypes from 'prop-types';

const MyComponent = ({ name, count, onClick }) => (
  <div onClick={onClick}>{name}: {count}</div>
);

MyComponent.propTypes = {
  name: PropTypes.string.isRequired,
  count: PropTypes.number,
  onClick: PropTypes.func.isRequired,
};

MyComponent.defaultProps = {
  count: 0,
};
```

### After (TypeScript)
```tsx
type Props = {
  name: string;
  count?: number;
  onClick: () => void;
};

const MyComponent = ({ name, count = 0, onClick }: Props) => (
  <div onClick={onClick}>{name}: {count}</div>
);
```

**Notes:**
- Move `defaultProps` into destructuring defaults
- Use `type` for props unless extension is needed (then `interface`)
- `isRequired` → non-optional field; optional PropTypes → `?` modifier

---

## 2. Redux `connect()` Typing

### Before
```jsx
const mapStateToProps = state => ({
  user: state.user,
  isLoading: state.app.isLoading,
});

const mapDispatchToProps = dispatch => ({
  fetchUser: id => dispatch(fetchUserAction(id)),
});

export default connect(mapStateToProps, mapDispatchToProps)(MyComponent);
```

### After
```tsx
import { RootState } from 'src/types';

type StateProps = ReturnType<typeof mapStateToProps>;
type DispatchProps = ReturnType<typeof mapDispatchToProps>;
type OwnProps = { /* component-specific props */ };
type Props = StateProps & DispatchProps & OwnProps;

const mapStateToProps = (state: RootState) => ({
  user: state.user,
  isLoading: state.app.isLoading,
});

const mapDispatchToProps = (dispatch: Dispatch) => ({
  fetchUser: (id: number) => dispatch(fetchUserAction(id)),
});

export default connect<StateProps, DispatchProps, OwnProps>(mapStateToProps, mapDispatchToProps)(MyComponent);
```

---

## 3. Event Handlers

### Common Event Types
```tsx
// Input change
onChange: (event: React.ChangeEvent<HTMLInputElement>) => void

// Select change
onChange: (event: React.ChangeEvent<HTMLSelectElement>) => void

// Form submit
onSubmit: (event: React.FormEvent<HTMLFormElement>) => void

// Button click
onClick: (event: React.MouseEvent<HTMLButtonElement>) => void

// Generic click
onClick: React.MouseEventHandler<HTMLElement>
```

---

## 4. Async Functions & Promises

### Before
```js
async function fetchData(url) {
  const response = await fetch(url);
  const data = await response.json();
  return data;
}
```

### After
```ts
async function fetchData<T>(url: string): Promise<T> {
  const response = await fetch(url);
  const data: T = await response.json();
  return data;
}
```

---

## 5. Object / Dictionary Types

```ts
// Avoid: object (too broad)
const config: object = {};

// Prefer: index signature
const config: Record<string, unknown> = {};

// Or explicit shape
const config: { timeout: number; retries: number } = { timeout: 5000, retries: 3 };

// Dynamic keys with known value type
const labelMap: Record<string, string> = {};
```

---

## 6. Superset-Specific Types

### QueryFormData
```ts
import { QueryFormData } from '@superset-ui/core';
```

### Chart Props
```ts
import { ChartProps } from '@superset-ui/core';
```

### Filter / Adhoc Filters
```ts
import { AdhocFilter, Filter } from 'src/types/Filter';
```

### Dataset
```ts
import { Dataset } from 'src/views/CRUD/types';
```

---

## 7. Type Assertions — Use Sparingly

```ts
// Avoid unless necessary
const el = document.getElementById('root') as HTMLDivElement;

// Prefer type narrowing
const el = document.getElementById('root');
if (el instanceof HTMLDivElement) {
  // el is HTMLDivElement here
}

// Non-null assertion — only when you're certain
const el = document.getElementById('root')!;
```

---

## 8. Enums vs Union Types

```ts
// Avoid numeric enums (they have footguns)
enum Status { Active, Inactive } // ❌

// Prefer string union types
type Status = 'active' | 'inactive'; // ✅

// Or const object pattern
const Status = {
  Active: 'active',
  Inactive: 'inactive',
} as const;
type Status = typeof Status[keyof typeof Status];
```

---

## 9. `any` Escape Hatch Policy

- **Never** use `any` in new code
- When migrating, prefer `unknown` over `any`
- If a third-party type is missing, use `// @ts-ignore` with a comment explaining why
- Document all `any` usages with `// TODO: type this properly`

```ts
// Acceptable temporary usage
const legacyUtil = (window as any).__legacyUtil; // TODO: type this properly
```

---

## 10. Re-exporting Types

```ts
// Use 'export type' to avoid runtime imports
export type { MyType } from './types';

// In index files
export type { ChartData, ChartConfig } from './ChartTypes';
```

---

## Common Pitfalls

| Pitfall | Solution |
|---|---|
| `React.FC` hides `children` type issues | Use explicit `Props` type instead |
| Forgetting `void` return type on event handlers | Always annotate event handler return types |
| Using `{}` as empty object type | Use `Record<string, never>` or `Record<string, unknown>` |
| Mutating state directly | TypeScript won't catch this — use immer or spread |
| Circular type imports | Use `import type` to break cycles |
