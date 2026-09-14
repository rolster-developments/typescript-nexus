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

Extend `Store` to encapsulate your domain logic. The protected `reduce` and
`select` methods let you express updates and derived reads declaratively:

```typescript
import { Store } from '@rolster/nexus';

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

## Contributing

- Daniel Andrés Castillo Pedroza :rocket:
