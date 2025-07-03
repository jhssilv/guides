This guide explains how to manage global application state in React. We'll solve the common problem of "prop drilling" by using React's built-in Context API combined with the `useReducer` hook for predictable state updates.

We will build a simple global store to manage a theme (e.g., 'light' or 'dark') and a user's authentication status.

### 1. The Problem: Prop Drilling

Imagine you have a `User` object at the top of your app, but a deeply nested `Avatar` component needs the user's name. You would have to pass the `user` prop through every single component in between, even those that don't use it. This is **prop drilling**. It's inefficient and makes components hard to reuse.

**Global state management** solves this by creating a central "store" of data that any component can subscribe to directly.

### 2. Defining State Shape and Actions

First, define the shape of your global state and the actions that can change it. This is the "contract" for your store.

```ts
// src/context/AppContext.types.ts

// The shape of our global state
export interface AppState {
  theme: 'light' | 'dark';
  isAuthenticated: boolean;
  user: { name: string } | null;
}

// The actions that can be dispatched to update the state
export type AppAction =
  | { type: 'TOGGLE_THEME' }
  | { type: 'LOGIN'; payload: { name: string } }
  | { type: 'LOGOUT' };
```

### 3. Creating the Reducer Function

A reducer is a pure function that takes the current `state` and an `action`, and returns the **new state**. It's the only place where state logic should live, making updates predictable.

```ts
// src/context/AppReducer.ts

import { AppState, AppAction } from './AppContext.types';

export const appReducer = (state: AppState, action: AppAction): AppState => {
  switch (action.type) {
    case 'TOGGLE_THEME':
      return {
        ...state,
        theme: state.theme === 'light' ? 'dark' : 'light',
      };
    case 'LOGIN':
      return {
        ...state,
        isAuthenticated: true,
        user: action.payload,
      };
    case 'LOGOUT':
      return {
        ...state,
        isAuthenticated: false,
        user: null,
      };
    default:
      return state;
  }
};
```

### 4. Creating the Context and Provider

Now, we create the React Context and a "Provider" component. The Provider will wrap our application, holding the state logic (`useReducer`) and making the state and the `dispatch` function available to all its children.

```tsx
// src/context/AppContext.tsx

import React, { createContext, useReducer, useContext, ReactNode } from 'react';
import { appReducer } from './AppReducer';
import { AppState, AppAction } from './AppContext.types';

const initialState: AppState = {
  theme: 'light',
  isAuthenticated: false,
  user: null,
};

// Create the context
const AppContext = createContext<{
  state: AppState;
  dispatch: React.Dispatch<AppAction>;
}>({
  state: initialState,
  dispatch: () => null, // Placeholder
});

// Create the Provider component
export const AppProvider = ({ children }: { children: ReactNode }) => {
  const [state, dispatch] = useReducer(appReducer, initialState);

  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
};

// Create a custom hook for easy access to the context
export const useAppContext = () => {
  return useContext(AppContext);
};
```

### 5. Wrapping Your Application

To make the global state available everywhere, wrap your root component (e.g., `App.tsx`) with the `AppProvider`.

```tsx
// src/index.tsx or App.tsx

import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { AppProvider } from './context/AppContext';

ReactDOM.render(
  <React.StrictMode>
    <AppProvider>
      <App />
    </AppProvider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

### 6. Using Global State in Components

Now, any component can access the global state or dispatch actions using our custom `useAppContext` hook, without any prop drilling!

**Example 1: A component that reads state.**

```tsx
// src/components/Navbar.tsx

import React from 'react';
import { useAppContext } from '../context/AppContext';

export const Navbar: React.FC = () => {
  const { state } = useAppContext();

  return (
    <nav>
      <span>Theme: {state.theme}</span>
      {state.isAuthenticated ? <p>Welcome, {state.user?.name}!</p> : <p>Please log in.</p>}
    </nav>
  );
};
```

**Example 2: A component that updates state.**

```tsx
// src/components/SettingsPanel.tsx

import React from 'react';
import { useAppContext } from '../context/AppContext';

export const SettingsPanel: React.FC = () => {
  const { dispatch } = useAppContext();

  const handleLogin = () => {
    dispatch({ type: 'LOGIN', payload: { name: 'Alice' } });
  };

  return (
    <div>
      <button onClick={() => dispatch({ type: 'TOGGLE_THEME' })}>
        Toggle Theme
      </button>
      <button onClick={handleLogin}>
        Log In
      </button>
    </div>
  );
};
```