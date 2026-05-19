# TypeScript Migration Patterns

This document captures common patterns and solutions encountered during the JS-to-TS migration of Apache Superset.

## Common Patterns

### 1. PropTypes to TypeScript Interfaces

**Before (JavaScript):**
```js
import PropTypes from 'prop-types';

const MyComponent = ({ name, count, onClick }) => {
  return <div onClick={onClick}>{name}: {count}</div>;
};

MyComponent.propTypes = {
  name: PropTypes.string.isRequired,
  count: PropTypes.number,
  onClick: PropTypes.func.isRequired,
};

MyComponent.defaultProps = {
  count: 0,
};
```

**After (TypeScript):**
```tsx
interface MyComponentProps {
  name: string;
  count?: number;
  onClick: () => void;
}

const MyComponent = ({ name, count = 0, onClick }: MyComponentProps) => {
  return <div onClick={onClick}>{name}: {count}</div>;
};
```

---

### 2. Redux `connect()` with TypeScript

**Before:**
```js
const mapStateToProps = state => ({
  user: state.user,
  loading: state.loading,
});

const mapDispatchToProps = dispatch => ({
  fetchUser: id => dispatch(fetchUserAction(id)),
});

export default connect(mapStateToProps, mapDispatchToProps)(MyComponent);
```

**After:**
```tsx
type StateProps = {
  user: UserType;
  loading: boolean;
};

type DispatchProps = {
  fetchUser: (id: number) => void;
};

type OwnProps = {
  title: string;
};

type Props = StateProps & DispatchProps & OwnProps;

const mapStateToProps = (state: RootState): StateProps => ({
  user: state.user,
  loading: state.loading,
});

const mapDispatchToProps = (dispatch: Dispatch): DispatchProps => ({
  fetchUser: (id: number) => dispatch(fetchUserAction(id)),
});

export default connect<StateProps, DispatchProps, OwnProps, RootState>(
  mapStateToProps,
  mapDispatchToProps,
)(MyComponent);
```

---

### 3. Event Handlers

**Before:**
```js
const handleChange = (e) => {
  setValue(e.target.value);
};
```

**After:**
```tsx
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};

// For select elements:
const handleSelect = (e: React.ChangeEvent<HTMLSelectElement>) => {
  setOption(e.target.value);
};
```

---

### 4. Async Functions and API Calls

**Before:**
```js
const fetchData = async (id) => {
  const response = await SupersetClient.get({
    endpoint: `/api/v1/dataset/${id}`,
  });
  return response.json;
};
```

**After:**
```tsx
interface DatasetResponse {
  id: number;
  table_name: string;
  schema: string;
}

const fetchData = async (id: number): Promise<DatasetResponse> => {
  const response = await SupersetClient.get({
    endpoint: `/api/v1/dataset/${id}`,
  });
  return response.json as DatasetResponse;
};
```

---

### 5. useRef Typing

**Before:**
```js
const containerRef = useRef(null);
```

**After:**
```tsx
const containerRef = useRef<HTMLDivElement>(null);
// or for mutable refs:
const timerRef = useRef<ReturnType<typeof setTimeout> | null>(null);
```

---

### 6. Enum-like Objects

**Before:**
```js
export const STATUS = {
  PENDING: 'pending',
  SUCCESS: 'success',
  ERROR: 'error',
};
```

**After:**
```tsx
export enum Status {
  Pending = 'pending',
  Success = 'success',
  Error = 'error',
}
// or use const assertion:
export const STATUS = {
  PENDING: 'pending',
  SUCCESS: 'success',
  ERROR: 'error',
} as const;

export type StatusType = typeof STATUS[keyof typeof STATUS];
```

---

## Known Problem Areas

- **Antd component types**: Import from `antd/lib/<component>` for more specific types
- **lodash**: Use `@types/lodash` — prefer named imports (`import { debounce } from 'lodash'`)
- **d3/echarts**: May require `// @ts-ignore` in rare cases; document when used
- **Dynamic object keys**: Use `Record<string, unknown>` or index signatures
- **`as any` usage**: Avoid — use proper generics or `unknown` with type guards instead

## Useful Superset-Specific Types

```tsx
// From @superset-ui/core
import type { JsonObject, JsonValue, QueryFormData } from '@superset-ui/core';

// Common internal types
import type { RootState } from 'src/views/store';
import type { Dataset } from 'src/views/CRUD/types';
```
