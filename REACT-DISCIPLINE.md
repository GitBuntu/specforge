# React Discipline in SpecForge

**Purpose**: Provide React-specific implementation guidance for LLMs and developers executing SpecForge on React projects. Map React component patterns, hooks, and testing to SpecForge's DDD aggregates, invariants, and assertion-based test specs.

**Scope**: This governance applies to all React projects using SpecForge during implementation (Tasks 1-7 execution and post-SpecForge development).

---

## Why React-Specific Guidance Matters in SpecForge

SpecForge enforces domain-driven test specifications that must map cleanly to React implementation. Without React-specific patterns:

- **Aggregate → Component mapping is unclear** — Which React component represents a DDD aggregate? Is it a page, a form, a hook, or a context provider?
- **Invariant enforcement is ambiguous** — How do React hooks (useState, useReducer, useContext) enforce domain invariants without test verification?
- **Test assertions don't translate** — Jest syntax (expect, toBe, toBeCalledWith) doesn't automatically map to SpecForge's 4 assertion patterns (==, is true, !=, emitted with).
- **Folder structure collision** — Where do tests live? src/components/_, tests/_, __tests__/?

**With React guidance**: Component structure becomes traceable, invariants are validated through test assertions, and folder organization matches SpecForge's artefact chain.

---

## Component-Aggregate Mapping

### DDD Aggregate → React Component Pattern

A DDD **Aggregate** is a cluster of domain objects (entities + value objects) treated as a single unit of consistency. In React, aggregates map to:

| DDD Element | React Implementation | SpecForge Mapping |
|------------|----------------------|-------------------|
| **Aggregate** | Custom hook (`useAggregate`) + Context provider | Domain entity state container |
| **Aggregate ID** | Context/hook state property (e.g., `userId`) | Primary key, scoped to aggregate |
| **Entity** | React component or hook state object | Mutable root within aggregate |
| **Value Object** | Immutable TypeScript interface or frozen object | Immutable properties (invariant protected) |
| **Aggregate Invariant** | Pre-condition + post-condition in hook logic OR useReducer validation | Test assertion that is always true |
| **Domain Event** | Custom hook callback or event emitter (observer pattern) | Return value from hook, emission in test |
| **Repository** | Custom hook (e.g., `useRepository`) + API layer | Fetch/save logic, external to SpecForge domain |

### Concrete Example: Order Aggregate

**DDD Domain Model** (from 00-context.md):
```
Aggregate: Order
├─ Aggregate ID: orderId (UUID)
├─ Entity: Line Items (mutable array)
├─ Value Object: OrderStatus (enum: Pending, Confirmed, Shipped, Delivered)
├─ Invariant: "Total quantity must be > 0"
├─ Invariant: "Order total must match sum of line item prices"
├─ Domain Event: OrderConfirmed (emits orderId, total, timestamp)
└─ Repository: OrderRepository (load by orderId, save)
```

**React Implementation** (React component + hook):
```jsx
// src/aggregates/useOrder.ts (Custom hook = Aggregate container)
interface Order {
  orderId: string;                      // Aggregate ID
  lineItems: LineItem[];                // Entity: mutable array
  orderStatus: 'Pending' | 'Confirmed'; // Value Object: immutable enum
  total: number;
}

export const useOrder = (orderId: string) => {
  const [order, setOrder] = useState<Order | null>(null);
  const [events, setEvents] = useState<DomainEvent[]>([]);

  // Invariant 1: Total quantity > 0
  const isValidQuantities = () => order?.lineItems.reduce((sum, li) => sum + li.qty, 0) > 0;

  // Invariant 2: Order total = sum of line item prices
  const isValidTotal = () => {
    const calculated = order?.lineItems.reduce((sum, li) => sum + (li.price * li.qty), 0) ?? 0;
    return calculated === order?.total;
  };

  const confirmOrder = () => {
    if (!isValidQuantities() || !isValidTotal()) throw new Error('Invariant violated');
    setOrder(prev => ({ ...prev, orderStatus: 'Confirmed' }));
    setEvents(prev => [...prev, { type: 'OrderConfirmed', payload: { orderId: order.orderId, total: order.total } }]);
  };

  return { order, events, confirmOrder };
};

// src/components/OrderForm.tsx (Component = Aggregate boundary)
export const OrderForm: React.FC<{ orderId: string }> = ({ orderId }) => {
  const { order, confirmOrder } = useOrder(orderId);
  return (/* JSX that calls confirmOrder */)
};
```

**SpecForge Test Mapping** (from TEST-001.md):
```
Assertion 1: order.lineItems.length > 0  [is true]  ← Invariant 1
Assertion 2: order.total == (sum of line item prices)  [==]  ← Invariant 2
Assertion 3: events emitted with { type: 'OrderConfirmed', payload: { orderId, total } }  [emitted with]  ← Domain Event
```

---

## Testing Patterns: Jest/RTL → SpecForge Assertions

### The 4 SpecForge Assertion Patterns

SpecForge enforces exactly 4 assertion patterns in tests (from TEST-{{TEST_ID}}.template.md):

1. **Equality (`==`)** — Value comparison (state, computed properties, return values)
2. **Boolean (`is true`)** — Invariant validation (condition always holds)
3. **Inequality (`!=`)** — Negation (state is not in forbidden state)
4. **Event (`emitted with`)** — Domain event emission (observable side effect)

### Jest/React Testing Library → SpecForge Mapping

| SpecForge Pattern | Jest Assertion | RTL Method | Example |
|------------------|----------------|-----------|---------|
| **`==`** | `expect(value).toBe(...)` | `expect(screen.getByText(...)).toBeInTheDocument()` | `expect(order.total).toBe(100)` |
| **`is true`** | `expect(condition).toBeTruthy()` | N/A (domain-level) | `expect(order.lineItems.length > 0).toBeTruthy()` |
| **`!=`** | `expect(value).not.toBe(...)` | `expect(screen.queryByText(...)).not.toBeInTheDocument()` | `expect(order.status).not.toBe('Cancelled')` |
| **`emitted with`** | `expect(mock).toHaveBeenCalledWith(...)` | Custom observer hook or callback | `expect(onOrderConfirmed).toHaveBeenCalledWith({ orderId, total })` |

### Concrete Test Example

**SpecForge Test (TEST-001.md)**:
```
## Assertions

### Assertion 1: Order Invariant — Quantity > 0
- **Pattern**: is true
- **Statement**: lineItems.length > 0 is true after addLineItem() is called

### Assertion 2: Order Invariant — Total Calculation
- **Pattern**: ==
- **Statement**: order.total == (sum of lineItem.price * lineItem.qty)

### Assertion 3: Domain Event — OrderConfirmed
- **Pattern**: emitted with
- **Statement**: events emitted with { type: 'OrderConfirmed', payload: { orderId, total } }
```

**Jest Test (src/tests/useOrder.test.ts)**:
```typescript
import { renderHook, act } from '@testing-library/react';
import { useOrder } from '../aggregates/useOrder';

describe('Order Aggregate', () => {
  it('TEST-001: Assertion 1 — lineItems.length > 0 is true after addLineItem()', () => {
    const { result } = renderHook(() => useOrder('order-123'));
    act(() => {
      result.current.addLineItem({ productId: 'p1', qty: 1, price: 50 });
    });
    expect(result.current.order.lineItems.length > 0).toBeTruthy(); // ✓ is true
  });

  it('TEST-001: Assertion 2 — order.total == sum of lineItem.price * lineItem.qty', () => {
    const { result } = renderHook(() => useOrder('order-123'));
    act(() => {
      result.current.addLineItem({ productId: 'p1', qty: 2, price: 50 });
    });
    const calculatedTotal = result.current.order.lineItems.reduce((sum, li) => sum + (li.price * li.qty), 0);
    expect(result.current.order.total).toBe(calculatedTotal); // ✓ ==
  });

  it('TEST-001: Assertion 3 — events emitted with OrderConfirmed', () => {
    const { result } = renderHook(() => useOrder('order-123'));
    act(() => {
      result.current.addLineItem({ productId: 'p1', qty: 1, price: 100 });
      result.current.confirmOrder();
    });
    expect(result.current.events).toContainEqual(
      expect.objectContaining({ type: 'OrderConfirmed', payload: expect.objectContaining({ orderId: 'order-123', total: 100 }) })
    ); // ✓ emitted with
  });

  it('TEST-002: Assertion 1 — order.status != Cancelled after confirm', () => {
    const { result } = renderHook(() => useOrder('order-123'));
    act(() => {
      result.current.addLineItem({ productId: 'p1', qty: 1, price: 50 });
      result.current.confirmOrder();
    });
    expect(result.current.order.orderStatus).not.toBe('Cancelled'); // ✓ !=
  });
});
```

---

## Project Folder Organization

### React + SpecForge Canonical Structure

SpecForge artefacts live in the **root** of the project. React source code lives in **src/**. The two hierarchies coexist:

```
my-react-app/
│
├─ SpecForge Artefacts (root-level)
│  ├── domain/
│  │   └── 00-context.md                        (1 of 7)
│  ├── requirements/
│  │   ├── REQ-001.md
│  │   └── REQ-002.md
│  ├── features/
│  │   └── order-management/                    (Feature: semantic name)
│  │       ├── domain/
│  │       │   └── 00-context.md
│  │       ├── requirements/
│  │       │   ├── REQ-001.md (2 of 7)
│  │       │   └── REQ-002.md
│  │       ├── features/
│  │       │   └── confirm-order/
│  │       │       ├── spec.md (3 of 7)
│  │       │       ├── SCENARIO-001.feature (4 of 7)
│  │       │       └── SCENARIO-002.feature
│  │       ├── tests/
│  │       │   └── confirm-order/
│  │       │       ├── TEST-001.md (5 of 7)
│  │       │       └── TEST-002.md
│  │       ├── planning/
│  │       │   └── PLAN-001.md (6 of 7)
│  │       └── tasks/
│  │           ├── TASK-001.md ([RED]) (7a of 7)
│  │           ├── TASK-002.md ([IMPL]) (7b of 7)
│  │           ├── TASK-003.md ([RED])
│  │           └── TASK-004.md ([IMPL])
│  │
│  ├── SpecForge-Contract.md
│  ├── TDD-DISCIPLINE.md
│  ├── REACT-DISCIPLINE.md
│  └── README.md
│
├─ React Source Code (src/)
│  ├── aggregates/                              (Maps to DDD Aggregates)
│  │   ├── useOrder.ts                         (Aggregate: Order)
│  │   ├── useOrderRepository.ts                (Repository: OrderRepository)
│  │   └── useCustomer.ts                       (Aggregate: Customer)
│  │
│  ├── components/                              (Maps to Aggregate Boundaries)
│  │   ├── OrderForm.tsx                       (Uses useOrder)
│  │   ├── OrderList.tsx                       (Uses useOrderRepository)
│  │   └── CustomerProfile.tsx                 (Uses useCustomer)
│  │
│  ├── pages/                                   (Top-level routes)
│  │   ├── OrdersPage.tsx
│  │   └── CustomersPage.tsx
│  │
│  ├── tests/                                   (Jest test files — mirror aggregates/)
│  │   ├── useOrder.test.ts
│  │   ├── useOrderRepository.test.ts
│  │   └── useCustomer.test.ts
│  │
│  ├── types/                                   (Value Objects, Enums from Domain)
│  │   ├── Order.ts
│  │   ├── Customer.ts
│  │   └── DomainEvents.ts
│  │
│  └── index.tsx
│
├── package.json
├── jest.config.js                              (Jest configuration)
└── tsconfig.json
```

### Folder Responsibilities

| Folder | SpecForge Role | React Role | Ownership |
|--------|----------------|-----------|-----------|
| **domain/** | Context definition | Ubiquitous language reference | Both (1:1 mapping) |
| **requirements/** | Requirement capture | Traceability link to features | SpecForge |
| **features/** | Feature & scenario specs | Traceability link to components | SpecForge |
| **tests/** | Test specs (assertions) | Traceability link to Jest tests | SpecForge |
| **planning/** | Planning & task generation | Developer reference for execution order | SpecForge |
| **tasks/** | Task descriptions & TDD cycle | Developer execution log (timestamps, status) | SpecForge |
| **src/aggregates/** | N/A (post-SpecForge) | Custom hooks implementing aggregates | React |
| **src/components/** | N/A (post-SpecForge) | Components using hooks, boundary enforcement | React |
| **src/tests/** | N/A (post-SpecForge) | Jest tests validating TEST-###.md assertions | React |
| **src/types/** | N/A (post-SpecForge) | TypeScript interfaces for value objects, enums | React |

### Key Rules

1. **SpecForge artefacts are at PROJECT ROOT** — not in src/. They are not shipped with the app; they guide development.
2. **src/tests/** mirrors src/aggregates/ — One test file per aggregate hook (useOrder.test.ts ↔ useOrder.ts).
3. **src/aggregates/** implements SpecForge Aggregates — No business logic in components; hooks contain invariants, events, state.
4. **Domain terminology flows down** — Order aggregate in 00-context.md → Order type in src/types/Order.ts → useOrder hook → OrderForm component.

---

## Tooling & Setup

### Required Packages

Add these to `package.json` for a React + SpecForge project:

```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@types/jest": "^29.0.0",
    "@types/react": "^18.0.0",
    "jest": "^29.0.0",
    "jest-environment-jsdom": "^29.0.0",
    "typescript": "^5.0.0",
    "ts-jest": "^29.0.0"
  }
}
```

### Jest Configuration

Create `jest.config.js` in project root:

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jsdom',
  roots: ['<rootDir>/src'],
  testMatch: ['**/*.test.ts', '**/*.test.tsx'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/index.tsx',
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

### Test Execution in SpecForge Tasks

During **TASK-###.md [RED] phase**:
```bash
npm test -- useOrder.test.ts --testNamePattern="TEST-001"
# Expected: ✗ FAIL (test is new, unimplemented)
# Commit: test(order-management): [RED] TEST-001 — lineItems.length > 0 after addLineItem() called
```

During **TASK-###.md [IMPL] phase**:
```bash
npm test -- useOrder.test.ts --testNamePattern="TEST-001"
# Expected: ✓ PASS (implementation satisfies test)
# Commit: impl(order-management): Order aggregate with addLineItem() and confirmOrder()
```

### TypeScript Configuration

Create `tsconfig.json` in project root:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

### Continuous Integration

Add `.github/workflows/test.yml` for CI/CD:

```yaml
name: SpecForge TDD Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with: { node-version: '18' }
      - run: npm ci
      - run: npm test -- --coverage
      - run: npm test -- --testNamePattern="\[RED\]|\[IMPL\]"
```

---

## Summary: When to Apply REACT-DISCIPLINE

- **SpecForge Phases 1–5**: Use DDD + EARS + Gherkin patterns (technology-agnostic). REACT-DISCIPLINE does NOT apply yet.
- **SpecForge Phase 6 (Planning)**: Reference REACT-DISCIPLINE for folder organization and test assertion mapping.
- **SpecForge Phase 7+ (Tasks, Implementation)**: Follow REACT-DISCIPLINE strictly for component structure, invariant enforcement, hook patterns, and Jest test execution.
- **Post-SpecForge**: Implement Repository interfaces (API calls, database, external services) using React query, SWR, or similar. Out of scope for SpecForge.

---

## Key Takeaways for LLMs

1. **Aggregate = Custom Hook** — Always create useAggregate hooks for DDD aggregates; never put domain logic in components.
2. **Invariants are test assertions** — Every invariant becomes ≥1 test assertion; tests are generated from TEST-###.md.
3. **4 patterns only** — Only use `==`, `is true`, `!=`, `emitted with` in tests. Nothing else.
4. **RED-IMPL pairing** — Odd tasks [RED] write failing tests; even tasks [IMPL] make them pass. No test modification in [IMPL].
5. **Events are callbacks** — Domain events emit via hook callbacks or observer pattern; captured in test as toHaveBeenCalledWith(...).
6. **Tests before components** — Write src/tests/useOrder.test.ts before src/aggregates/useOrder.ts. TDD discipline applies.
7. **Folder mirrors SpecForge** — src/aggregates/ ↔ Aggregates from 00-context.md; src/tests/ ↔ tests/{{FEATURE_NAME}}/; src/types/ ↔ Value Objects from context.

