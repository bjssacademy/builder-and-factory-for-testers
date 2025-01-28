# Let's Build It

Let’s introduce a **builder pattern** to make the process of constructing user profiles more flexible and readable. The builder pattern is especially useful when the construction process involves multiple steps or optional configurations.

---

## **Step 1: Define the Builder**

**What we're doing**  
We'll create a `UserProfileBuilder` class that allows step-by-step construction of a `UserProfile`. The builder will include *methods* to set the `name`, `age`, and `role`, and will handle adding the correct permissions based on the role.

**Why**  
This approach:
- Improves readability when constructing profiles.
- Reduces the risk of errors in specifying the profile’s attributes.
- Makes it easier to add optional or conditional steps in the future.

---

### Refactored Code with Builder

```typescript
type Permission = "read" | "write" | "delete";

const roleConfigurations: Record<string, Permission[]> = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};

type UserProfile = {
  name: string;
  age: number;
  role: string;
  permissions: Permission[];
};

class UserProfileBuilder {
  private profile: Partial<UserProfile> = {};

  setName(name: string): this {
    this.profile.name = name;
    return this;
  }

  setAge(age: number): this {
    this.profile.age = age;
    return this;
  }

  setRole(role: keyof typeof roleConfigurations): this {
    this.profile.role = role;
    this.profile.permissions = roleConfigurations[role];
    return this;
  }

  build(): UserProfile {
    if (!this.profile.name || !this.profile.age || !this.profile.role || !this.profile.permissions) {
      throw new Error("Incomplete profile: name, age, and role are required.");
    }
    return this.profile as UserProfile;
  }
}

// Example Usage
const user1 = new UserProfileBuilder()
  .setName("Alice")
  .setAge(30)
  .setRole("admin")
  .build();

const user2 = new UserProfileBuilder()
  .setName("Bob")
  .setAge(25)
  .setRole("editor")
  .build();

const user3 = new UserProfileBuilder()
  .setName("Charlie")
  .setAge(20)
  .setRole("viewer")
  .build();

console.log(user1);
console.log(user2);
console.log(user3);
```

---

### **Why This Is Better**

1. **Step-by-Step Construction**  
   The builder allows profiles to be constructed step-by-step, improving readability and reducing mistakes.

2. **Validation**  
   The `build` method ensures all required fields (`name`, `age`, `role`) are set before the profile is returned. This avoids creating incomplete objects.

3. **Flexible Extensibility**  
   It’s easy to add new methods (e.g., `setAddress` or `setCustomPermissions`) to extend the builder without affecting existing code.

---

## **Step 2: Add Optional Features**

We can now easily add optional features, such as custom permissions, without altering the role-based logic.

#### Adding Custom Permissions

```typescript
class UserProfileBuilder {
  private profile: Partial<UserProfile> = {};

  setName(name: string): this {
    this.profile.name = name;
    return this;
  }

  setAge(age: number): this {
    this.profile.age = age;
    return this;
  }

  setRole(role: keyof typeof roleConfigurations): this {
    this.profile.role = role;
    this.profile.permissions = roleConfigurations[role];
    return this;
  }

  addCustomPermission(permission: Permission): this {
    if (!this.profile.permissions) {
      this.profile.permissions = [];
    }
    this.profile.permissions.push(permission);
    return this;
  }

  build(): UserProfile {
    if (!this.profile.name || !this.profile.age || !this.profile.role || !this.profile.permissions) {
      throw new Error("Incomplete profile: name, age, and role are required.");
    }
    return this.profile as UserProfile;
  }
}

// Example Usage with Custom Permissions
const user4 = new UserProfileBuilder()
  .setName("Dave")
  .setAge(40)
  .setRole("editor")
  .addCustomPermission("delete")
  .build();

console.log(user4);
```

---

### **Why Add Custom Permissions?**

- **Granular Control**  
   You can now add specific permissions for users without overriding the role-based defaults.

- **Reusability**  
   The builder remains flexible for future changes, such as additional configuration options or logic.

---

## Step 3: Refactor Our Builder

Let's refactor the `UserProfileBuilder` class to replace the `setXX` methods with `withXX`. The term `withXX` is semantically better because it describes the context in which the object is being built, focusing on the **end result** rather than the internal implementation details.

---

### **Refactored Code: `UserProfileBuilder`**

Here’s how the builder will look after the refactor.

#### Code:

```typescript
class UserProfileBuilder {
  private profile: Partial<UserProfile> = {};

  withName(name: string): this {
    this.profile.name = name;
    return this;
  }

  withAge(age: number): this {
    this.profile.age = age;
    return this;
  }

  withRole(role: keyof typeof roleConfigurations): this {
    this.profile.role = role;
    this.profile.permissions = roleConfigurations[role];
    return this;
  }

  addCustomPermission(permission: Permission): this {
    if (!this.profile.permissions) {
      this.profile.permissions = [];
    }
    this.profile.permissions.push(permission);
    return this;
  }

  build(): UserProfile {
    if (!this.profile.name || !this.profile.age || !this.profile.role || !this.profile.permissions) {
      throw new Error("Incomplete profile: name, age, and role are required.");
    }
    return this.profile as UserProfile;
  }
}

// Example Usage with Custom Permissions
const user4 = new UserProfileBuilder()
  .withName("Dave")
  .withAge(40)
  .withRole("editor")
  .addCustomPermission("delete")
  .build();

console.log(user4);
```

---

### **Key Changes**
1. **Renaming Methods**
   - Changed method names from `setName`, `setAge`, etc., to `withName`, `withAge`, etc.
   - This makes the builder's interface more *declarative* and user-friendly.

2. **Semantics:**
   - The `withXX` naming convention aligns more with how the builder is expected to be used, emphasising the intent rather than implementation.

---

### **Benefits of Using `withXX`**

1. **Declarative API**:  
   The `withXX` naming aligns the builder's interface with how it will be consumed. It reads more naturally, e.g., `withName("Alice")` instead of `setName("Alice")`.

2. **Fluent Interface**:  
   The method chain conveys the *what* of the object being built, not the *how*. This abstraction makes the code more expressive and easier to understand.

3. **Encapsulation of Logic**:  
   The focus shifts from mutating internal state (`setXX`) to expressing intent (`withXX`), which is more aligned with the **OOP principle of abstraction**.

---

## Summary

Okay, we've taken somethign that worked, and have written *more* code? Surely we want to write *less* code?

Well, yes and no. We want to make out code to be more semantic and user-friendly, as well as being more flexible.

---

[Next Chapter >>](./chapter4.md)
