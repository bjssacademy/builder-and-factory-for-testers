# Builder for Pre-Existing Classes

Let’s explore how to use `StrictBuilder` from [Vincent Pang’s builder-pattern library](https://github.com/vincent-pang/builder-pattern) to build objects in TypeScript, without having to create a separate builder class!

We’ll walk through what `StrictBuilder` does, how to use it with an **existing class**, and why it’s beneficial.

---

## **What is a StrictBuilder?**

The `StrictBuilder` enforces:
1. **Immutability during construction**

Fields must be explicitly defined step by step.

2. **Type safety**

Ensures that all required fields are set at compile time before the object can be built.

3. **Fluent Interface**

Provides a chainable API for a clean, readable, and developer-friendly way to construct objects.

> :exclamation: This is particularly useful when dealing with complex objects or classes where you want to ensure that the end object is always in a valid state.

---

## **Example**

```typescript
import { StrictBuilder } from "builder-pattern";

interface UserInfo {
  id: number;
  userName: string;
  email: string;
}

const userInfo = StrictBuilder<UserInfo>()
  .id(1) 
  .userName("Alice") 
  .email("alice@example.com") 
  .build(); 

console.log(userInfo);
// Output: { id: 1, userName: 'Alice', email: 'alice@example.com' }
```

---

## **Key Features of `StrictBuilder`**

1. **Type Safety at Compile Time**
   - The builder requires that all fields defined in the target type (`UserInfo` in this case) are explicitly initialized before calling `build()`.
   - If you forget to set a field, TypeScript will immediately throw an error, for example:

   ```typescript
   const incompleteUser = StrictBuilder<UserInfo>()
     .id(1)
     .userName("Alice")
     .build(); // Error: Property 'email' is missing
   ```

2. **Named Methods for Each Field**
   - `StrictBuilder` generates methods based on the field names in the target type. For example, for `id`, `userName`, and `email`, it creates `.id()`, `.userName()`, and `.email()` methods, respectively.

3. **Order of Initialization Does Not Matter**
   - You can set the fields in any order, as long as all fields are set before calling `.build()`.

   ```typescript
   const userInfo = StrictBuilder<UserInfo>()
     .email("charlie@example.com")
     .id(42)
     .userName("Charlie")
     .build();
   ```

4. **Immutability**
   - Each method (`id`, `userName`, etc.) returns a new builder instance with the updated field, ensuring immutability.

---

## **Benefits of Using `StrictBuilder`**

1. **Strong Compile-Time Validation**

   Developers are guaranteed that objects created using the builder are always valid because all required fields are initialised before `build()`.

2. **Improved Developer Experience**

   With named methods for each field, the builder provides an intuitive and fluent interface that makes object construction clear and readable.

3. **Reduced Risk of Errors**:
   - Forgetting to set a required field is caught at compile time, eliminating a class of runtime bugs.

4. **Flexible and Reusable**
   
   You can reuse the `StrictBuilder` instance to create multiple valid objects by chaining the appropriate methods.

---

## **Practical Example with Nested Types**

You can use `StrictBuilder` with types that have nested objects. Let’s say `UserInfo` has an `address` field.

```typescript
interface Address {
  street: string;
  city: string;
}

interface UserInfo {
  id: number;
  userName: string;
  email: string;
  address: Address;
}

// Using StrictBuilder
const userInfo = StrictBuilder<UserInfo>()
  .id(1)
  .userName("Diana")
  .email("diana@example.com")
  .address( StrictBuilder<Address>()
    .street("123 Main St")
    .city("Metropolis")
    .build()
    ) 
  .build();

console.log(userInfo);
// Output: { id: 1, userName: 'Diana', email: 'diana@example.com', address: { street: '123 Main St', city: 'Metropolis' } }
```

---

## **Summary**

The `StrictBuilder` is a great utility for enforcing type-safe object creation in TypeScript, without having to create a specific builder class.

It ensures that
1. **All fields are set before building** the object, preventing invalid or incomplete instances.
2. **Each field has its own method**, offering clarity and a fluent interface.
3. **Immutability is maintained** throughout the building process.

### When should you use `StrictBuilder`?
- When working with objects that have many required fields.
- When you need a clear and structured way to create objects.
- When you want to guarantee that every object created is valid and complete.

---

[Builder Examples in C# >>](./addenda.md)