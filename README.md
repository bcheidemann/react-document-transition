# React Document Transitions

This package enables creating complex state transitions by using simple hooks. It uses `document.createDocumentTransition` from the [page transition API](https://github.com/WICG/shared-element-transitions/blob/main/developer-guide.md). This API is currently only available in [Chrome Canary](https://www.google.com/chrome/canary/) and requires the `chrome://flags/#document-transition` flag to be enabled.

## Basic Usage

The `useTransitionState` hook is a thin wrapper around the `useState` hook and exposes a nearly identical API.

```tsx
import React, { useState } from 'react';
import { useTransitionState } from 'react-document-transition';

function Example() {
  const [count, setCount] = useTransitionState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

However, in browsers which support the `document.createDocumentTransition` API, any state changes will be animated.

## Advanced Usage

![ToDo List Example](https://raw.githubusercontent.com/bcheidemann/react-document-transition/main/assets/example-todo.gif)

```tsx
import { useTransitionState } from 'react-document-transition';

type TodoItem = {
  name: string;
  done: boolean;
}

type TodoListState = {
  items: TodoItem[],
}

function Item(item: TodoItem & { toggle: (item: TodoItem) => void}) {
  const slug = item.name.replaceAll(" ", "-").toLowerCase();
  const toggle = () => {
    item.toggle(item);
  };

  return (
    <div
      onClick={toggle}
      className={`item ${item.done ? 'done' : ''}`}
      style={{
        pageTransitionTag: slug,
      } as React.CSSProperties}
    >
      <h1>{item.name}</h1>
    </div>
  );
}

function App() {
  const [state, setState] = useTransitionState<TodoListState>({
    items: [
      { done: false, name: "Buy Milk" },
      { done: false, name: "Publish NPM Package" },
      { done: false, name: "Become Rich and Famous" },
      { done: true, name: "Pay Rent" },
      { done: true, name: "Create React Document Transition Hook" },
    ],
  });

  function toggle(item: TodoItem) {
    let items: TodoItem[] = Array.from(state.items);

    const foundItem = items.find(maybeItem => maybeItem.name === item.name);
    if (foundItem) {
      foundItem.done = !foundItem.done;
    }
    
    setState({
      items,
    });
  }

  return (
    <div className="App">
      <div className="todo-col">
        {state.items.filter(item => !item.done).map((item) => (
          <Item {...item} toggle={toggle} key={item.name} />
        ))}
      </div>
      <div className="done-col">
        {state.items.filter(item => item.done).map((item) => (
          <Item {...item} toggle={toggle} key={item.name} />
        ))}
      </div>
    </div>
  );
}

export default App;
```

```css
.App {
  text-align: center;
  display: flex;
  min-height: 100vh;
}

.todo-col {
  display: flex;
  flex-direction: column;
  row-gap: 12px;
  padding: 12px;
  flex: 1;
  background: red;
}

.done-col {
  display: flex;
  flex-direction: column;
  row-gap: 12px;
  padding: 12px;
  flex: 1;
  background: green;
}

.item {
  contain: paint;
  background: lightpink;
  border-radius: 12px;
  cursor: pointer;
}

.item.done {
  background: lightblue;
}

.item.done h1 {
  font-size: 16px;
}
```
