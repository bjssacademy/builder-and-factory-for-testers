# Simple Factory

Now we're going to create a *factory*. A factory is another method that sits on top of our builder and builds pre-determined objects for us. You can think of factories as being the mass-producers of objects, whereas using the builder directly is like being the craftsman making one-off specials.

Where people often become confused though, it that they sometimes try to make the builder responsible for this mass behaviour.

Let me show you.

## God Builder :( 

> :exclamation: Don't change your code, this is just an example of what people sometimes do.

```ts
// builders/UserProfileBuilder.ts

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

  forAdmin(): this {
    this.role = "admin";
    this.permissions = RoleConfiguration.getPermissions("admin");
    return this;
  }

  forEditor(): this {
    this.role = "editor";
    this.permissions = RoleConfiguration.getPermissions("editor");
    return this;
  }

  forViewer(): this {
    this.role = "viewer";
    this.permissions = RoleConfiguration.getPermissions("viewer");
    return this;
  }

  forCustomRole(role: string): this {
    this.role = role;
    this.permissions = RoleConfiguration.getPermissions(role);
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

And you might use this like so:

```ts

const user = new UserProfileBuilder()
    .withName("Adam")
    .withAge(20)
    .forEditor()
    .build();

```

This *looks* like an excellent approach. Until you realise we have violated quite a few principles here. 

Now if there is a new role with certain permissions, we have to add another method. D'oh!

> As a general rule, keep the builder dumb. The builder *only* builds. It shouldn't know about things like specific roles. 
>
> If you find yourself making the builder "smart" like this, then you should probably move that into a factory.

Keeping the builder "dumb" aligns with the principle of **single responsibility**. In the previous example, I violated this by introducing role-specific methods (`forAdmin`, `forEditor`, etc.) directly in the builder. This approach **tightly couples the builder to the roles, breaking the separation of concerns and reducing its flexibility**. 

--- 

## A Proper Factory

### **Why the Previous Design Was Bad**

1. **Builder Knows Too Much**

   By adding `forAdmin`, `forEditor`, etc., the builder now contains knowledge of roles and their configurations. This increases its responsibilities, making it harder to maintain and extend.

2. **Reduced Flexibility**  

   If roles change (e.g., new roles or permissions), we must modify the builder itself. This violates the **open-closed principle** (OCP), which states that code should be open for extension but closed for modification.

3. **Unnecessary Complexity** 


   Adding specific methods for every role clutters the builder's API. It increases the surface area of potential bugs and confuses its purpose (constructing objects step by step).

---

The factory will:

1. Use faker.js to generate random values for name, age, and role.
2. Use the UserProfileBuilder to create a valid UserProfile object.

---

### **Refactored Code**

#### **1. Simplified `UserProfileBuilder`**

The builder is stripped of any role-specific logic and now focuses solely on constructing a `UserProfile` step by step.

```typescript
// builders/UserProfileBuilder.ts

import { UserProfile, Permission } from "../models/UserProfile";

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

  withRole(role: string, permissions: Permission[]): this {
    this.role = role;
    this.permissions = permissions;
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

> :exclamation: You may notice that now the withRole has changed and doesn't directly call the RoleConfiguration. This is part of the process of making our builder just responsible for building.
>
> For those paying attention, you'll also be thinking "hang on...doesn't this mean the builder could build something that isn't valid?". Yep! We'll come onto that in a bit.

---

#### **2. Enhanced `UserProfileFactory`**

The factory now contains all the logic for creating profiles for specific roles. Here we user [fakerjs](https://fakerjs.dev/) for creating a user with a random name, and a random age between 18 and 65.

```typescript
// factories/UserProfileFactory.ts

import { faker } from "@faker-js/faker";
import { UserProfile } from "../models/UserProfile";
import { UserProfileBuilder } from "../builders/UserProfileBuilder";
import { RoleConfiguration } from "../config/RoleConfiguration";

export class UserProfileFactory {
  static createAdmin(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole("admin", RoleConfiguration.getPermissions("admin"))
      .build();
  }

  static createEditor(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole("editor", RoleConfiguration.getPermissions("editor"))
      .build();
  }

  static createViewer(): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole("viewer", RoleConfiguration.getPermissions("viewer"))
      .build();
  }

  static createForRole(role: string): UserProfile {
    return new UserProfileBuilder()
      .withName(faker.name.fullName())
      .withAge(faker.datatype.number({ min: 18, max: 65 }))
      .withRole(role, RoleConfiguration.getPermissions(role))
      .build();
  }
}
```

---

### **3. Updated Example Usage**

The factory is now used to create users for specific roles without bloating the builder.

```typescript
// main.ts

import { UserProfileFactory } from "./factories/UserProfileFactory";
import { RoleConfiguration } from "./config/RoleConfiguration";

// Add a new role dynamically (optional)
RoleConfiguration.addRole("moderator", ["read", "write"]);

// Generate users for specific roles
const adminUser = UserProfileFactory.createAdmin();
const editorUser = UserProfileFactory.createEditor();
const viewerUser = UserProfileFactory.createViewer();
const customRoleUser = UserProfileFactory.createForRole("moderator");

console.log(adminUser.toString());
console.log(editorUser.toString());
console.log(viewerUser.toString());
console.log(customRoleUser.toString());
```

---

## **Why the New Design Is Better**

1. **Separation of Concerns**

   The builder remains a simple, reusable tool for creating `UserProfile` objects without worrying about business logic like roles. The **factory** takes responsibility for assigning roles, permissions, and other contextual logic.

2. **Open-Closed Principle**

   If we need to add a new role, we modify the factory or `RoleConfiguration`, not the builder. This improves maintainability.

3. **Cleaner API**

   The builder’s interface is minimal, focused only on its job—constructing objects. It doesn’t bloat with unnecessary methods.

4. **Improved Reusability**

   The builder can now be used for any kind of `UserProfile` creation, while the factory handles domain-specific details.

---

## **Benefits of This Approach**

1. **Single Responsibility Principle (SRP)**
   - The **builder** focuses on assembling objects step by step without domain-specific logic.
   - The **factory** takes on the responsibility of role-specific and contextual details.

2. **Reduced Coupling**  

   The builder no longer depends on the role system, making it reusable in different contexts.

3. **Extensibility**  

   To add a new role, we only update `RoleConfiguration` and the factory. The builder remains untouched.

4. **Testability**  

   Each component (builder, factory, role configuration) can now be tested in isolation, simplifying unit tests.

---

### **Summary**

By separating concerns between the builder and the factory, we’ve achieved a **cleaner, more modular design**:
- The **builder** is a simple, reusable tool for creating objects.
- The **factory** encapsulates domain-specific logic for creating role-based profiles.

---

[Next Chapter >>](./chapter6.md)