# Type Safety for Permissions

Let's enhance the code by adding **type safety for permissions**. This ensures that only *valid* permissions can be assigned, reducing errors and making our code more robust.

---

## **Step 1: Define Permissions as a Type**

**What we're doing**  
We’ll define valid permissions as a union of string literals (`"read" | "write" | "delete"`) and ensure roles can only use these permissions.

**Why**  
This prevents typos or invalid permissions from being accidentally used.

### Refactored Code:

```typescript
type Permission = "read" | "write" | "delete";

const roleConfigurations: Record<string, Permission[]> = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};

const createUserProfile = (
  name: string,
  age: number,
  role: keyof typeof roleConfigurations
): UserProfile => {
  const baseProfile = createBaseProfile(name, age);
  return {
    ...baseProfile,
    role,
    permissions: roleConfigurations[role],
  };
};

// Example Usage
const user1 = createUserProfile("Alice", 30, "admin");
const user2 = createUserProfile("Bob", 25, "editor");
const user3 = createUserProfile("Charlie", 20, "viewer");

// Invalid Role or Permission Example (Uncomment to see TypeScript error)
// const invalidUser = createUserProfile("Dave", 40, "superadmin"); // Error: "superadmin" is not assignable to "admin" | "editor" | "viewer"
// roleConfigurations.admin.push("execute"); // Error: Type '"execute"' is not assignable to type 'Permission'

console.log(user1);
console.log(user2);
console.log(user3);
```

## Type Safety?

Let me break down the important parts here.

### 1. **Defining the `Permission` Type:**
```typescript
type Permission = "read" | "write" | "delete";
```

- Here, we're creating a custom **type** called `Permission`. This is a **union type**, meaning that it can only have one of the specific string values: `"read"`, `"write"`, or `"delete"`.
- This limits the possible values that a `Permission` can take. For example, you can't accidentally assign `"edit"` or `"view"` to a `Permission` because those aren't part of the defined set of possible values.
- TypeScript ensures type safety by restricting any variable of type `Permission` to one of these three exact string values. If you try to assign a different string to a `Permission`, TypeScript will give an error.

### 2. **Using `Permission[]` in `roleConfigurations`:**
```typescript
const roleConfigurations: Record<string, Permission[]> = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};
```

- **`Record<string, Permission[]>`**:
  - `Record` is a built-in utility type in TypeScript. It's used to define an object type where:
    - The **keys** are of type `string` (in this case, the role names like `"admin"`, `"editor"`, `"viewer"`).
    - The **values** are of type `Permission[]` (an array of valid `Permission` values, like `["read", "write"]`).

- The object `roleConfigurations` contains roles (like `"admin"`, `"editor"`, `"viewer"`) as keys, and each key maps to an array of permissions that the role has.
  
TypeScript ensures that the values in the arrays are only valid `Permission` values. So, if you try to add something like `"edit"` to one of these arrays, TypeScript will raise an error, because `"edit"` isn't part of the `Permission` type.
  
  Example of incorrect usage:
  ```typescript
  roleConfigurations.admin = ["read", "write", "edit"]; // Error: 'edit' is not assignable to type 'Permission'.
  ```

### How Does TypeScript Enforce Type Safety?

During development TypeScript will check that the values assigned to `roleConfigurations` are valid `Permission` values. For example, if someone tries to assign a value like `"edit"` to the array, TypeScript will immediately flag it as an error.

This is particularly helpful for catching bugs early in the development process, because we will be notified about invalid values before running the code.

---

### **Why This Is Better**

1. **Type Safety**  
   The `Permission` type ensures that only `"read"`, `"write"`, or `"delete"` can be used in `roleConfigurations`.

2. **Compile-Time Errors**  
   Any attempt to add invalid permissions or use an undefined role will result in a TypeScript error, catching issues early (a major benefit of TypeScript of JS).

3. **Ease of Maintenance**  
   If new permissions are introduced, they can be added to the `Permission` type, making changes straightforward. We love straightforward.

---

## Summary

Okay,we've made our code a bit less brittle, and made it clearer that roles are associated with a permission type. This stops people just cramming in random magic strings somewhere else in the code.

---

[Next Chapter >>](./chapter3.md)