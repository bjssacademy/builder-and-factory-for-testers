# More Encapsulation and Type Safety

The current implementation relies on the caller to explicitly set the permissions for a role, which introduces the risk of creating a `UserProfile` with **invalid or mismatched permissions**. By introducing a `Role` type that encapsulates the role's name and associated permissions, we can ensure that every role always has a valid set of permissions.

This approach adheres to **encapsulation** and **type safety**, improving the reliability of the code by preventing invalid states.

---

## **Refactored Design**

1. **Introduce a `Role` Class**
   - Encapsulates both the role name and its associated permissions.
   - Ensures that permissions are tightly coupled to roles and cannot be mismatched.

2. **Update the Builder**
   - Replace the `withRole(role: string, permissions: Permission[])` method with `withRole(role: Role)`, removing the need to set permissions separately.

3. **Adjust the Factory**
   - The factory now uses `Role` instances to assign roles to user profiles, ensuring correctness.

---

## **Updated Code**

### **1. `Role` Class**

This class acts as a type-safe representation of a role and its permissions.

```typescript
// models/Role.ts

import { Permission } from "./UserProfile";

export class Role {
  constructor(public readonly name: string, public readonly permissions: Permission[]) {}

  static admin(): Role {
    return new Role("admin", ["read", "write", "delete"]);
  }

  static editor(): Role {
    return new Role("editor", ["read", "write"]);
  }

  static viewer(): Role {
    return new Role("viewer", ["read"]);
  }
}
```

---

### **2. Updated `UserProfileBuilder`**

The builder now accepts a `Role` object instead of manually setting the role and permissions, ensuring type safety.

```typescript
// builders/UserProfileBuilder.ts

import { UserProfile } from "../models/UserProfile";
import { Role } from "../models/Role";

export class UserProfileBuilder {
  private name?: string;
  private age?: number;
  private role?: Role;

  withName(name: string): this {
    this.name = name;
    return this;
  }

  withAge(age: number): this {
    this.age = age;
    return this;
  }

  withRole(role: Role): this {
    this.role = role;
    return this;
  }

  build(): UserProfile {
    if (!this.name || !this.age || !this.role) {
      throw new Error("Incomplete profile: name, age, and role are required.");
    }
    return new UserProfile(this.name, this.age, this.role.name, this.role.permissions);
  }
}
```

---

### **3. Updated `UserProfileFactory`**

The factory uses predefined methods from the `Role` class to assign valid roles to user profiles.

```typescript
// factories/UserProfileFactory.ts

import { faker } from "@faker-js/faker";
import { UserProfile } from "../models/UserProfile";
import { UserProfileBuilder } from "../builders/UserProfileBuilder";
import { Role } from "../models/Role";

export class UserProfileFactory {
  static createAdmin(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole(Role.admin())
      .build();
  }

  static createEditor(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole(Role.editor())
      .build();
  }

  static createViewer(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole(Role.viewer())
      .build();
  }

}
```

---

## **Example Usage**

Here’s how the updated design works in practice:

```typescript
// main.ts

import { UserProfileFactory } from "./factories/UserProfileFactory";
import { Role } from "./models/Role";

// Generate users for specific roles
const adminUser = UserProfileFactory.createAdmin();
const editorUser = UserProfileFactory.createEditor();
const viewerUser = UserProfileFactory.createViewer();

console.log(adminUser.toString());
console.log(editorUser.toString());
console.log(viewerUser.toString());
```

---

### **Why This Is Better**

1. **Encapsulation**:  
   - The `Role` class encapsulates the role's name and permissions, ensuring they cannot be accidentally mismatched.

2. **Type Safety**:  
   - Roles are now represented by a dedicated type (`Role`), which eliminates invalid states like assigning incorrect permissions to a role.

3. **Simpler Builder**:  
   - The builder is "dumb" and focuses on its core responsibility: assembling a `UserProfile`. It no longer needs to manage role-specific logic.

4. **Centralised Logic**:  
   - Role definitions and permissions are managed within the `Role` class. Adding or modifying roles happens in one place, improving maintainability.

5. **Cleaner API**:  
   - Using `Role.admin()`, `Role.editor()`, etc., is more expressive and readable than manually specifying permissions everywhere.

---

## **Why Use Static Methods in the Factory?**

The use of **static methods** in the factory is intentional and provides several **benefits**.

A **static method** belongs to the class itself rather than to an instance of the class. This means you can call the method directly on the class without needing to create an object of that class. In our `UserProfileFactory`, all the methods (`createAdmin`, `createEditor`, etc.) are static. This has benefits, but you need to understand why it's okay here but not in **all** cases!

---

### **1. No Need for State**

The `UserProfileFactory` doesn’t maintain any *internal state or store data*. Its sole purpose is to create `UserProfile` objects based on predefined logic. Since it doesn’t rely on instance-specific data, **creating an instance of the factory would be redundant**.

For example:
```typescript
const factory = new UserProfileFactory(); // No need for this.
const adminUser = factory.createAdmin();
```

Instead, static methods let us call the factory methods directly:
```typescript
const adminUser = UserProfileFactory.createAdmin(); // Cleaner and more efficient.
```

---

### **2. Global Access Without Instantiation**

Factories are often utility classes—designed to be **reusable** from anywhere in the codebase. By making the methods static, the factory becomes a **globally accessible utility**. You don’t have to create an instance of the factory every time you want to use it.

For example:
- A `UserProfileFactory` can be used in multiple places without worrying about creating or managing its instances.

---

### **3. Simplifies Usage**

Static methods simplify the API by removing the need to instantiate the class. This makes the code more concise and easier to use, especially in cases where the factory is frequently invoked.

Example:
```typescript
// Static approach (simple)
const viewerUser = UserProfileFactory.createViewer();

// Non-static approach (verbose)
const factory = new UserProfileFactory();
const viewerUser = factory.createViewer();
```

With static methods, you skip the unnecessary boilerplate of creating a factory instance.

---

### **4. Encourages Stateless Design**

Using static methods enforces **stateless design**. A `UserProfileFactory` doesn't need to hold state because its purpose is purely functional — it creates objects based on inputs or predefined rules. 

Stateless design has several advantages:
- **Thread-safety**: Since no state is maintained, static methods are inherently thread-safe.
- **Reusability**: The factory can be reused without worrying about side effects from lingering state.

---

### **5. Reduced Memory Usage**

By avoiding instantiation, we save memory. A single static class is loaded once into memory, and its methods can be called repeatedly without creating multiple objects.

- Instantiating the factory multiple times would use memory unnecessarily.
- With static methods, no instances are created, resulting in **lower memory consumption**.

---

### **When Should You Avoid Static Methods?**

While static methods work great here, they’re not always the best choice. You should avoid static methods if:
1. **Stateful Logic Is Needed**

   If the factory needs to maintain state (e.g., caching created objects or keeping track of some configuration), instance methods would be more appropriate.

2. **Extensibility**

   Static methods can be harder to override in subclasses. If you anticipate needing to extend the factory for custom logic, instance methods might be preferable.

3. **Dependency Injection**

   Static methods **don’t work well with dependency injection frameworks**. If your factory needs to rely on injected dependencies, you’d typically use instance methods.

---

## **Summary**

Using static methods in the `UserProfileFactory` ensures that it remains a lightweight, globally accessible utility for object creation. It enforces stateless design, simplifies usage, and reduces unnecessary memory consumption. 

---

[Next Chapter >>](./chapter7.md)