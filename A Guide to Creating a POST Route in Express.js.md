This guide provides a step-by-step blueprint for building robust and type-safe `POST` routes in an Express application using TypeScript. Following this structure will help you write clean, predictable, and maintainable API endpoints.

We'll use the practical example of creating a new "product" to illustrate each step.

### 1. Defining Interfaces (The Contract)

Interfaces are the "contract" for your API. They define the expected shape of data for both the request and the response, ensuring type safety.

First, define an interface for the incoming request body. This describes what the client _must_ send to you.

```ts
// src/interfaces/ProductInterfaces.ts

/**
 * Describes the data required to create a new product.
 * This is the shape of the `req.body` for our POST request.
 */
export interface CreateProductRequestBody {
  name: string;
  price: number;
  inStock: boolean;
}
```

Next, define an interface for the successful response payload. This describes what you promise to send back upon success.

```ts
// src/interfaces/ProductInterfaces.ts

/**
 * Describes the data sent back to the client after a
 * product has been successfully created.
 */
export interface CreateProductResponse {
  productId: string;
  message: string;
  productUrl: string;
}
```

### 2. Payload Unwrapping (Receiving the Data)

In your controller, you'll receive the request and safely unwrap the data from the request body. Typing the `req` and `res` objects with your interfaces connects them to your controller's logic.

```ts
// src/controllers/productController.ts

import { Request, Response } from 'express';
import { CreateProductRequestBody, CreateProductResponse } from '../interfaces/ProductInterfaces';

export const createProductController = (
  // Apply the interfaces to the request and response objects
  req: Request<{}, {}, CreateProductRequestBody>,
  res: Response<CreateProductResponse | { error: string }> // Response can be success or error
) => {
  // Unwrapping the payload from the request body.
  // Thanks to our interface, TypeScript knows that `name`, `price`,
  // and `inStock` exist on `req.body`.
  const { name, price, inStock } = req.body;

  console.log(`Received new product: ${name} - Price: ${price}`);

  // Next steps will go here...
};
```

### 3. Data Validation (Trust, but Verify)

**Never trust data from the client.** Always validate the payload on the server before performing any operations. This prevents bad data from corrupting your database and makes your API more secure.

```ts
// Continuing inside createProductController...

// --- Validation Step ---
if (!name || typeof name !== 'string' || name.trim().length < 3) {
  return res.status(400).json({ error: 'Product name must be a string with at least 3 characters.' });
}

if (!price || typeof price !== 'number' || price <= 0) {
  return res.status(400).json({ error: 'Price must be a positive number.' });
}

if (typeof inStock !== 'boolean') {
  return res.status(400).json({ error: 'inStock must be a boolean value (true or false).' });
}

// If we reach this point, the data is considered valid.
```

**Pro Tip:** For more complex validation, consider using a dedicated library like [Zod](https://zod.dev/ "null") or [Joi](https://joi.dev/ "null"), which can simplify this step significantly.

### 4. Operations (The Core Logic)

This is where you perform the main action of your route, such as interacting with a database, calling another service, or executing a business process. Wrap this logic in a `try...catch` block to handle unexpected errors.

```ts
// Continuing inside createProductController...

try {
  // --- Operations Step ---
  // This is a placeholder for your actual database logic.
  // e.g., const newProduct = await ProductModel.create({ name, price, inStock });
  console.log('Simulating database operation...');
  const newProductId = `prod_${Math.random().toString(36).substring(2, 9)}`;
  console.log(`Product created with ID: ${newProductId}`);

  // The next step is to send the success response...

} catch (error) {
  // The next step is to handle this potential error...
}
```

### 5. Success Responses (Confirming It Worked)

If the operation is successful, send back a `201 Created` status code. The response body should conform to your `CreateProductResponse` interface. This confirms to the client that their request was successful and provides them with useful information, like the ID of the newly created resource.

```ts
// Inside the `try` block, after the operation...

// --- Success Response Step ---
const responsePayload: CreateProductResponse = {
  productId: newProductId,
  message: 'Product created successfully.',
  productUrl: `/api/products/${newProductId}`,
};

return res.status(201).json(responsePayload);
```

### 6. Error Responses (Handling Failures)

If anything goes wrong during the operation (e.g., a database connection fails), the `catch` block will execute. You should log the error for debugging purposes and send a generic `500 Internal Server Error` response to the client. Avoid sending detailed system error messages to the client for security reasons.

```ts
// The `catch` block from the operations step...

// --- Error Response Step ---
} catch (error) {
  console.error('Error creating product:', error); // Log the actual error for your team

  // Send a generic error message to the client
  return res.status(500).json({ error: 'An unexpected error occurred on the server.' });
}
```

### Summary: The 6-Step Checklist

Use this checklist for a quick reminder of the process when building a new POST route.

1. **Define Interfaces:** Create types for the request `body` and the successful response payload.
2. **Unwrap Payload:** Destructure the `body` from the `req` object in your controller.
3. **Validate Data:** Check every field from the payload. Return a `400` error for invalid data.
4. **Perform Operations:** Place your core logic (e.g., database calls) inside a `try...catch` block.
5. **Send Success Response:** On success, return a `201` status with a response body matching your interface.
6. **Send Error Response:** In the `catch` block, log the error and return a `500` status with a generic error message.