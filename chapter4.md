# A Bit of OOP Does You Good

Refactoring the current codebase to follow a more **object-oriented programming (OOP)** approach in TypeScript while splitting the code into separate files improves **modularity, maintainability, and scalability**. 

Let's get into it!

---

## **Refactoring Plan**

1. **Separate Concerns**

   Split the code into logically distinct components:
   - `UserProfile` model (data structure and validation logic).
   - `RoleConfiguration` for managing roles and permissions.
   - `UserProfileBuilder` for constructing `UserProfile` objects.

2. **Leverage TypeScript Classes**

   Introduce classes to encapsulate functionality, enforce contracts (via types/interfaces), and provide clear responsibilities.

3. **File Organisation**
   - Each file will focus on a single responsibility.
   - This separation ensures that changes to one part of the system (e.g., role definitions) don’t inadvertently affect others.

---

### Folder Structure

Make sure you have the following folders and files:

```bash
-main.ts
|-models
    -UserProfile.ts
|-config
    -RoleConfiguration.ts
|-builders
    -UserProfileBuilder.ts
```

---

## **1. `models/UserProfile.ts`**

This file defines the `UserProfile` class, encapsulating all properties of a user profile. It also includes type definitions for `Permission`.

### Code:

```typescript
export type Permission = "read" | "write" | "delete";

export class UserProfile {
  constructor(
    public name: string,
    public age: number,
    public role: string,
    public permissions: Permission[]
  ) {}

  toString(): string {
    return `UserProfile(name: ${this.name}, age: ${this.age}, role: ${this.role}, permissions: ${this.permissions.join(", ")})`;
  }
}
```

### **Why This Way?**
- **Encapsulation** The `UserProfile` class focuses solely on the structure of the user profile.
- **Convenience Method** `toString` provides a human-readable representation of the object for debugging/logging.

---

## **2. `config/RoleConfiguration.ts`**

This file manages the roles and their associated permissions. It provides a `RoleConfiguration` class to centralise role definitions.

### Code:

```typescript
import { Permission } from "../models/UserProfile";

export class RoleConfiguration {
  private static roles: Record<string, Permission[]> = {
    admin: ["read", "write", "delete"],
    editor: ["read", "write"],
    viewer: ["read"],
  };

  static getPermissions(role: string): Permission[] {
    const permissions = this.roles[role];
    if (!permissions) {
      throw new Error(`Role "${role}" is not defined.`);
    }
    return permissions;
  }

  static addRole(role: string, permissions: Permission[]): void {
    if (this.roles[role]) {
      throw new Error(`Role "${role}" already exists.`);
    }
    this.roles[role] = permissions;
  }
}
```

### **Why This Way?**
- **Centralised Role Management**  
  `RoleConfiguration` ensures all roles are defined and accessed in one place.
- **Extensibility**  
  New roles can be added using `addRole`, ensuring a single source of truth.

---

## **3. `builders/UserProfileBuilder.ts`**

This file implements the `UserProfileBuilder` class, which encapsulates the step-by-step process of creating `UserProfile` objects.

### Code:

```typescript
import { UserProfile, Permission } from "../models/UserProfile";
import { RoleConfiguration } from "../config/RoleConfiguration";

export class UserProfileBuilder {
  private name?: string;
  private age?: number;
  private role?: string;
  private permissions?: Permission[];

  withName(name: string): this {
    this.name = name;
    return this;
  }

  withAge(age: number): this {
    this.age = age;
    return this;
  }

  withRole(role: string): this {
    this.role = role;
    this.permissions = RoleConfiguration.getPermissions(role);
    return this;
  }

  addCustomPermission(permission: Permission): this {
    if (!this.permissions) {
      throw new Error("Role must be set before adding custom permissions.");
    }
    this.permissions.push(permission);
    return this;
  }

  build(): UserProfile {
    if (!this.name || !this.age || !this.role || !this.permissions) {
      throw new Error("Incomplete profile: name, age, and role are required.");
    }
    return new UserProfile(this.name, this.age, this.role, this.permissions);
  }
}
```

### **Why This Way?**
- **Encapsulation of Profile Creation Logic**  
  The builder ensures that the creation process is structured and validated.
- **Step-by-Step Configuration**  
  The builder allows incremental configuration of `UserProfile` objects.

---

## **4. `main.ts`**

This is the entry point of the application, demonstrating how everything ties together.

### Code:

```typescript
import { UserProfileBuilder } from "./builders/UserProfileBuilder";
import { RoleConfiguration } from "./config/RoleConfiguration";

// Add a new role dynamically (optional)
RoleConfiguration.addRole("moderator", ["read", "write"]);

// Create user profiles using the builder
const user1 = new UserProfileBuilder()
  .withName("Alice")
  .withAge(30)
  .withRole("admin")
  .build();

const user2 = new UserProfileBuilder()
  .withName("Bob")
  .withAge(25)
  .withRole("editor")
  .addCustomPermission("delete")
  .build();

const user3 = new UserProfileBuilder()
  .withName("Charlie")
  .withAge(20)
  .withRole("viewer")
  .build();

console.log(user1.toString());
console.log(user2.toString());
console.log(user3.toString());
```

That's pretty cool, right? Now that we have implemented the builder pattern, we can see how each user object is built and we aren't hiding details.

> :exclamation: You might be asking "hey, why can't we just write a function with parameters?" Well, you could but then code that looks like `createUser("Bob", "admin", 20)` isn't as expressive as a builder. Also, this is a trivial example - imagine hundreds of properties in a class...


---

## **Why Split Into Files?**

1. **Separation of Concerns**  
   Each file has a single responsibility (model, role configuration, builder, or application logic). This makes the code easier to understand and maintain.

2. **Reusability**  
   By separating roles, models, and the builder, these components can be reused in other parts of the system or applications.

3. **Scalability**  
   As the system grows (e.g., adding more roles, features, or builders), the codebase remains organised and manageable.

---

## **OOP and TypeScript - Why?**

1. **Encapsulation**  
   Classes encapsulate data and behaviour, making it easy to manage complexity.

2. **Type Safety**  
   TypeScript ensures errors (e.g., invalid permissions or missing required properties) are caught during development.

3. **Extensibility**  
   Adding new roles, permissions, or profile features is straightforward without affecting existing functionality.

---

## Summary

So that's a simple builder. Those of you paying attention might still be wondering the same thing as me - why is role a string rather than a type `Role`? Well the answer is that we haven't got that far yet! 

Keep reading and we'll get to it!

---

[Next Chapter >>](./chapter5.md)