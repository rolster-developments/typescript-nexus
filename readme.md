# Rolster Nexus

Library that allows you to manage the status of applications.

## Installation

```
npm i @rolster/nexus
```

## Configuration

You must install the `@rolster/types` to define package data types, which are configured by adding them to the `files` property of the `tsconfig.json` file.

```json
{
  "files": ["node_modules/@rolster/types/index.d.ts"]
}
```

## Features

A small, framework-agnostic state container built on top of the observable from
`@rolster/commons`. The state is always a `LiteralObject`, it is kept immutable
(frozen) and every update notifies its subscribers.

### Basic usage

Instantiate a `Store` with its initial state, then read, update and subscribe:

```typescript
import { Store } from '@rolster/nexus';

interface CounterState {
  count: number;
  step: number;
}

const store = new Store<CounterState>({ count: 0, step: 1 });

// Read the current (read-only) value
store.value; // { count: 0, step: 1 }

// React to changes — `subscribe` fires immediately with the current value
const unsubscribe = store.subscribe((state) => {
  console.log('count is', state.count);
});

// Partially update the state (shallow merge)
store.setValue({ count: 5 }); // subscribers receive { count: 5, step: 1 }

// Restore the initial state
store.reset();

unsubscribe();
```

`subscribe` vs `listen`: `subscribe` emits the current value right away and on
every change; `listen` only emits on future changes.

### Custom stores with actions

Extend `Store` to encapsulate your domain logic. The protected
`reduce(reducer: Reducer<T>)` and `select(selector: Selector<T, V>)` methods
let you express updates and derived reads declaratively:

```typescript
import { Store } from '@rolster/nexus';

interface Product {
  name: string;
  price: number;
}

interface CartState {
  items: Product[];
  total: number;
}

class CartStore extends Store<CartState> {
  constructor() {
    super({ items: [], total: 0 });
  }

  public addItem(product: Product): void {
    this.reduce((state) => ({
      items: [...state.items, product],
      total: state.total + product.price
    }));
  }

  public get count(): number {
    return this.select((state) => state.items.length);
  }
}

const cart = new CartStore();
cart.addItem({ name: 'Mouse', price: 25 });
cart.count; // 1
```

### Types

| Type               | Description                                                                                                                                                          |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AbstractStore<T>` | Abstract contract implemented by `Store`: `value`, `subscribe`, `listen` and `reset`. Depend on it (for example when injecting a store) instead of a concrete class. |
| `Reducer<T>`       | `(value: Readonly<T>) => T` — builds the next state from the current one; argument of `reduce`.                                                                      |
| `Selector<T, V>`   | `(value: Readonly<T>) => V` — derives a value from the current state; argument of `select`.                                                                          |

```typescript
import { AbstractStore } from '@rolster/nexus';

class CartView {
  constructor(private store: AbstractStore<CartState>) {}

  public render(): Unsubscription {
    return this.store.subscribe((state) => console.log(state.total));
  }
}

new CartView(new CartStore());
```

## Contributing

- Daniel Andrés Castillo Pedroza :rocket:
