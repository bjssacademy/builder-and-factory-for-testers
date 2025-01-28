# Take Some Bad Code, And Make It Better
### Refactoring Functional TypeScript to Type Safe OOP
---

> Examples in this tutorial will be in TypeScript as I find people coming from a JS background tend to use TS directly like JS. Other examples will be provided at the end.

Here's an example of some code you've probably written more than once if you are working with a system that has users:

```ts
type UserProfile = {
  name: string;
  age: number;
  role: string;
  permissions: string[];
};

const createAdmin = (name: string, age: number): UserProfile => {
  return {
    name,
    age,
    role: "admin",
    permissions: ["read", "write", "delete"],
  };
};

const createEditor = (name: string, age: number): UserProfile => {
  return {
    name,
    age,
    role: "editor",
    permissions: ["read", "write"],
  };
};

const createViewer = (name: string, age: number): UserProfile => {
  return {
    name,
    age,
    role: "viewer",
    permissions: ["read"],
  };
};

// Example Usage
const user1 = createAdmin("Alice", 30);
const user2 = createEditor("Bob", 25);
const user3 = createViewer("Charlie", 20);

console.log(user1);
console.log(user2);
console.log(user3);

```

### For those new to TypeScript

```ts
type UserProfile = {
  name: string;
  age: number;
  role: string;
  permissions: string[];
};
```

This TypeScript code defines a "type" called `UserProfile`. A "type" is like a blueprint for what a user profile should look like. Imagine a profile card you fill out with details about a person. In this case, the profile card will have the following sections:

1. name - This is where you'll put the person's name. It has to be a text (string).
2. age - This section will contain the person's age, but it has to be a number.
3. role - This section holds the person's job title or role, like "Admin" or "User", and it must be a text (string).
4. permissions - This part is a list of things the person can do, like "view", "edit", or "delete". It's a list of texts (strings).

So, this code is saying: "A user profile must have a name, age, role, and a list of permissions, where name and role are text, age is a number, and permissions is a list of text items."

It's a way to make sure every user profile follows this structure, helping to avoid errors when working with data about users.

And this works, it's fine, but we can see a lot of duplication - and if there is a new role or permission we'll just be copy-pasting a lot.

## Why is this bad?

### Duplication of Logic
Each function (`createAdmin`, `createEditor`, `createViewer`) has repeated logic, like setting the name, age, and permissions.

### Tightly Coupled to Roles
The logic for creating different roles is scattered across multiple functions. If roles or permissions need updating, *every function* must be changed.

### Lack of Flexibility
Adding a new role or permission structure requires creating another function, which could lead to even more duplication.

### No Abstraction
There is no abstraction to separate common properties (e.g., name and age) from role-specific configurations.

---

## Ok, so what do we do?

Let's work in small steps to refactor this. We're going to use [computational thinking](https://www.bbc.co.uk/bitesize/guides/zp92mp3/revision/1) to *decompose* our problem, recognise *patterns*, create *abstractions*, and then apply *algorithms*.

Let’s refactor this code in simple steps, improving it incrementally while explaining each change.

---

## **Step 1: Extract Common Logic**

**What we're doing**  
We'll start by extracting the common properties (`name` and `age`) into a generic function. This will reduce duplication and centralise the shared logic for creating user profiles.

**Why?**  
This simplifies the code by separating the parts that are the same for every user from the parts that differ (e.g., role and permissions).

#### Refactored Code:

```typescript
type UserProfile = {
  name: string;
  age: number;
  role: string;
  permissions: string[];
};

const createBaseProfile = (name: string, age: number): Omit<UserProfile, "role" | "permissions"> => {
  return {
    name,
    age,
  };
};

const createAdmin = (name: string, age: number): UserProfile => {
  const baseProfile = createBaseProfile(name, age);
  return {
    ...baseProfile,
    role: "admin",
    permissions: ["read", "write", "delete"],
  };
};

const createEditor = (name: string, age: number): UserProfile => {
  const baseProfile = createBaseProfile(name, age);
  return {
    ...baseProfile,
    role: "editor",
    permissions: ["read", "write"],
  };
};

const createViewer = (name: string, age: number): UserProfile => {
  const baseProfile = createBaseProfile(name, age);
  return {
    ...baseProfile,
    role: "viewer",
    permissions: ["read"],
  };
};

// Example Usage
const user1 = createAdmin("Alice", 30);
const user2 = createEditor("Bob", 25);
const user3 = createViewer("Charlie", 20);

console.log(user1);
console.log(user2);
console.log(user3);
```

### For those wondering how this works...

In this TypeScript code, the spread operator (`...`) is used to copy the properties from one object into another. Let me break it down in detail:

#### 1. **The `createBaseProfile` function**:
This function takes a `name` and an `age` and returns an object that contains these two properties but **without the `role` and `permissions`** properties. The `Omit<UserProfile, "role" | "permissions">` part ensures that the returned object will be missing the `role` and `permissions` properties from the `UserProfile` type.

Example output from `createBaseProfile`:
```ts
{
  name: "John",
  age: 30,
}
```

#### 2. **The `createAdmin` function**:
This function calls `createBaseProfile` to create a basic profile, which contains `name` and `age`. The spread operator is then used in the return statement:

```ts
return {
  ...baseProfile,
  role: "admin",
  permissions: ["read", "write", "delete"],
};
```

Here’s what happens:
- **`...baseProfile`**: This takes the properties of the object returned by `createBaseProfile` (i.e., `name` and `age`), and "spreads" them into the new object.
- **`role: "admin"`** and **`permissions: ["read", "write", "delete"]`**: These are then added to the object.

The spread operator essentially says: "Take everything from `baseProfile` and copy it into the new object, then add or overwrite these specific properties (`role` and `permissions` in this case)."

Example output from `createAdmin`:
```ts
{
  name: "John",
  age: 30,
  role: "admin",
  permissions: ["read", "write", "delete"],
}
```

#### Why use the spread operator here?
- It avoids repeating the code for the `name` and `age` properties.
- It allows you to "extend" an object by adding or modifying properties (like `role` and `permissions`) without needing to manually copy over everything from `baseProfile`.

So the spread operator is a way to combine or extend objects in a clean and readable way...for now.

---

### **Why This Is Better**

- **Reduced Duplication**  
  `createBaseProfile` eliminates the repetitive creation of `name` and `age` fields.
  
- **Centralised Base Logic**  
  If something about `name` or `age` changes (e.g., adding validation), we only need to update it in one place.

---

## **Step 2: Generalise Role and Permissions Creation**

**What we're doing**  
We'll replace the hardcoded `createAdmin`, `createEditor`, and `createViewer` functions with a *single* function that accepts the role and permissions as parameters.

**Why**  
This eliminates the need to write a separate function for each role, making it more flexible and maintainable.

### Refactored Code

```typescript
const createUserProfile = (
  name: string,
  age: number,
  role: string,
  permissions: string[]
): UserProfile => {
  const baseProfile = createBaseProfile(name, age);
  return {
    ...baseProfile,
    role,
    permissions,
  };
};

// Example Usage
const user1 = createUserProfile("Alice", 30, "admin", ["read", "write", "delete"]);
const user2 = createUserProfile("Bob", 25, "editor", ["read", "write"]);
const user3 = createUserProfile("Charlie", 20, "viewer", ["read"]);

console.log(user1);
console.log(user2);
console.log(user3);
```

---

### **Why This Is Better**

- **Simpler Logic**  
  Only one function (`createUserProfile`) handles all roles, reducing the need for separate functions.

- **More Flexible**  
  Adding a new role (e.g., "moderator") no longer requires creating a new function—just call `createUserProfile` with the appropriate parameters.

---

## **Step 3: Introduce a Factory for Role Configurations**

**What we're doing**  
Instead of passing `role` and `permissions` manually each time, we'll use a factory to encapsulate the logic for role-specific configurations. This avoids hardcoding roles in multiple places.

**Why**  
This centralises the role definitions, making it easier to manage and extend the code.

### Refactored Code:

```typescript
const roleConfigurations = {
  admin: ["read", "write", "delete"],
  editor: ["read", "write"],
  viewer: ["read"],
};

const createUserProfile = (name: string, age: number, role: keyof typeof roleConfigurations): UserProfile => {
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

console.log(user1);
console.log(user2);
console.log(user3);
```

---

### **Why This Is Better**

- **Centralised Role Definitions**  
  All roles and their permissions are defined in `roleConfigurations`. Adding or updating a role is now straightforward.

- **Reduced Human Error**  
  We no longer need to remember or manually input permissions when creating users.

---

## Summary

Okay, so far, so good. We've managed to create a single function rather than multiple functions with duplicated code, and we've abstracted and encapsulated our roles and permissions, making it easier to extend for new permissions and only have one place where roles specified.

---

[Next Chapter >>](./chapter2.md)