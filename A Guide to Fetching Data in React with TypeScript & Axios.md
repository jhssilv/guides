This guide provides a complete pattern for fetching data from an API, handling loading and error states, and displaying the data in your React application using TypeScript and the `axios` library.

We will build a `UserList` component that fetches a list of users from an API and displays them using the `UserProfileCard` component from our previous guide.

### 1. API Contract & Type Definition

Before making an API call, define the shape of the expected data. This ensures type safety throughout your component. Let's assume our API endpoint (`/api/users`) returns an array of user objects.

We can reuse and adapt the `UserProfileCardProps` for this.

```ts
// src/types/api.types.ts

// This describes the shape of a single user object from our API
export interface UserData {
  userId: string;
  name: string;
  email: string;
  avatarUrl?: string;
}
```

### 2. State Management for Async Operations

Fetching data involves three key states you must manage to create a good user experience:

1. **`data`**: The data successfully fetched from the API.
2. **`loading`**: A boolean to indicate if the request is currently in progress.
3. **`error`**: An object or string to hold any error information if the request fails.

We'll manage these with `useState` hooks.

```tsx
// src/components/UserList/UserList.tsx

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { UserData } from '../../types/api.types';
import { UserProfileCard } from '../UserProfileCard';

export const UserList: React.FC = () => {
  const [users, setUsers] = useState<UserData[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  // Fetching logic will go here next...

  return (
    {/* JSX for rendering will go here */}
  );
};
```

### 3. Fetching Data with `useEffect` and `axios`

We use the `useEffect` hook to perform the data fetch when the component first renders. The empty dependency array (`[]`) ensures this effect runs only once.

Inside `useEffect`, we define an `async` function to handle the `axios` request.

```tsx
// Continuing inside UserList.tsx

useEffect(() => {
  // We define an async function inside the effect
  const fetchUsers = async () => {
    try {
      // Set loading to true before the request
      setLoading(true);
      
      const response = await axios.get<UserData[]>('/api/users');
      
      // On success, update the users state and clear errors
      setUsers(response.data);
      setError(null);
    } catch (err) {
      // On error, update the error state
      setError('Failed to fetch users. Please try again later.');
      // It's also good practice to log the actual error for debugging
      console.error(err);
    } finally {
      // Set loading to false after the request is complete
      setLoading(false);
    }
  };

  fetchUsers();
}, []); // Empty dependency array means this runs once on mount
```

### 4. Conditional Rendering (Handling UI States)

Now, use the `loading` and `error` states to render the correct UI. This gives the user immediate feedback about the status of the application.

```tsx
// The return statement inside UserList.tsx

if (loading) {
  return <div className="text-center p-8">Loading users...</div>;
}

if (error) {
  return <div className="text-center p-8 text-red-600">{error}</div>;
}

return (
  <div className="space-y-4">
    <h2 className="text-2xl font-bold">Our Users</h2>
    {/* We will map over the data next */}
  </div>
);
```

### 5. Displaying the Fetched Data

Once the data is successfully loaded, map over the `users` array and render a `UserProfileCard` for each user.

```tsx
// The final return statement inside UserList.tsx

return (
  <div className="space-y-4">
    <h2 className="text-2xl font-bold mb-4">Our Users</h2>
    {users.map((user) => (
      <UserProfileCard
        key={user.userId}
        userId={user.userId}
        name={user.name}
        email={user.email}
        avatarUrl={user.avatarUrl}
        onSelect={(id) => console.log(`Selected user: ${id}`)}
      />
    ))}
  </div>
);
```

### 6. Refactoring to a Reusable `useFetch` Hook (Advanced)

To avoid repeating this logic in every component that needs to fetch data, you can extract it into a custom hook.

```ts
// src/hooks/useFetch.ts

import { useState, useEffect } from 'react';
import axios from 'axios';

interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

export function useFetch<T>(url: string): FetchState<T> {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null,
  });

  useEffect(() => {
    const fetchData = async () => {
      setState({ data: null, loading: true, error: null });
      try {
        const response = await axios.get<T>(url);
        setState({ data: response.data, loading: false, error: null });
      } catch (err) {
        setState({ data: null, loading: false, error: 'Fetch failed' });
        console.error(err);
      }
    };

    fetchData();
  }, [url]); // Re-fetch if the URL changes

  return state;
}

// Now your component becomes much simpler:
// const { data: users, loading, error } = useFetch<UserData[]>('/api/users');
```