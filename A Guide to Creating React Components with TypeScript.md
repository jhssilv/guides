This guide provides a blueprint for building high-quality, type-safe functional components in React using TypeScript. Following this structure will help you write clean, maintainable, and reusable UI code.

We'll use the practical example of creating a `UserProfileCard` component to illustrate each step.

### 1. Defining Props (The Component's API)

The first step is to define the "contract" for your component using a TypeScript `interface` or `type`. This specifies what data the component expects to receive from its parent.

- **`interface`**: Best for defining the shape of objects or classes. Can be extended by other interfaces.
- **`type`**: More flexible; can represent unions, intersections, or primitives.

For component props, `interface` is often preferred for its clarity and extensibility.

```ts
// src/components/UserProfileCard/UserProfileCard.types.ts

export interface UserProfileCardProps {
  /** The user's unique identifier */
  userId: string;
  /** The full name of the user */
  name: string;
  /** The user's email address */
  email: string;
  /** The URL for the user's avatar image */
  avatarUrl?: string; // Optional property
  /** A function to call when the card is clicked */
  onSelect: (userId: string) => void;
}
```

### 2. Component Scaffolding (The Basic Structure)

Create the component file and define its basic structure as a functional component. Use `React.FC` (Functional Component) to type the component itself, which provides type-checking for its props and a clear definition of its function.

```tsx
// src/components/UserProfileCard/UserProfileCard.tsx

import React from 'react';
import { UserProfileCardProps } from './UserProfileCard.types';

// Using React.FC to type the component
export const UserProfileCard: React.FC<UserProfileCardProps> = ({
  userId,
  name,
  email,
  avatarUrl,
  onSelect
}) => {
  // Component logic and JSX will go here
  
  return (
    <div>
      {/* JSX content will be added in a later step */}
      Hello, {name}!
    </div>
  );
};
```

### 3. State Management (Handling Internal Data)

For any internal data that can change, use the `useState` hook. TypeScript's type inference often works here, but you can also explicitly provide a type for more complex state.

```tsx
// Continuing inside UserProfileCard.tsx

const [isSelected, setIsSelected] = React.useState<boolean>(false);
// TypeScript infers the type of `isSelected` as boolean

const [lastInteraction, setLastInteraction] = React.useState<Date | null>(null);
// For complex types like unions, explicitly define the type
```

### 4. Handling Events (Making it Interactive)

Correctly typing event handlers is crucial. React provides types for most synthetic events, like `React.MouseEvent` for clicks or `React.ChangeEvent` for inputs.

```tsx
// Continuing inside UserProfileCard.tsx

const handleCardClick = (event: React.MouseEvent<HTMLDivElement>) => {
  // `event` is now fully typed, with properties like `event.currentTarget`
  console.log('Card was clicked!');
  
  setIsSelected(!isSelected);
  setLastInteraction(new Date());

  // Call the function passed down in props
  onSelect(userId);
};
```

### 5. JSX and Styling (The Visual Output)

Write the JSX that renders the component's UI. It's best practice to use a consistent styling solution, such as CSS Modules, styled-components, or a utility-first framework like Tailwind CSS.

This example uses Tailwind CSS classes for styling.

```tsx
// The return statement inside UserProfileCard.tsx

return (
  <div
    onClick={handleCardClick}
    className={`p-4 border rounded-lg shadow-sm cursor-pointer transition-all ${
      isSelected ? 'border-blue-500 ring-2 ring-blue-200' : 'border-gray-200'
    }`}
  >
    <div className="flex items-center">
      <img
        src={avatarUrl || `https://i.pravatar.cc/150?u=${email}`} // Fallback avatar
        alt={`${name}'s avatar`}
        className="w-16 h-16 rounded-full mr-4"
      />
      <div>
        <h3 className="text-lg font-bold text-gray-800">{name}</h3>
        <p className="text-gray-600">{email}</p>
      </div>
    </div>
    {lastInteraction && (
        <p className="text-xs text-gray-400 mt-2">
            Last interaction: {lastInteraction.toLocaleTimeString()}
        </p>
    )}
  </div>
);
```

### 6. Default Props and Exporting

Provide default values for any optional props to prevent runtime errors and ensure predictability. With functional components, this is easily done using default parameter syntax.

Finally, you can add a `default` export for convenience if it's the main export of the file.

```ts
// src/components/UserProfileCard/index.ts

// It's good practice to have an index file to manage exports

export { UserProfileCard } from './UserProfileCard';
export type { UserProfileCardProps } from './UserProfileCard.types';

// You can also set default props directly in the component signature:
/*
export const UserProfileCard: React.FC<UserProfileCardProps> = ({
  name,
  avatarUrl = 'default-avatar-url.png', // Default value here
  ...
}) => { ... }
*/
```

### Summary: The 6-Step Checklist

Use this checklist for a quick reminder when building a new React component with TypeScript.

1. **Define Props:** Create a `Props` interface in a `.types.ts` file to define the component's public API.
2. **Scaffold Component:** Create the component as a `React.FC` and destructure its props.
3. **Manage State:** Use the `useState` hook with explicit types for any complex internal state.
4. **Handle Events:** Use React's built-in event types (e.g., `React.MouseEvent`) for event handlers.
5. **Write JSX:** Build the visual structure with clean JSX and a consistent styling approach.
6. **Set Defaults & Export:** Provide default values for optional props and manage exports cleanly, often with an `index.ts` file.