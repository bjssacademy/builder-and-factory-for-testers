# Using the Factory for Testing (API or UI)  

Factories are a tester's best friend when it comes to creating **dynamic, reusable, and valid test data**. They help streamline the setup process and avoid repetitive, hard-coded values in test cases - or worse, reading from static files. 

Let's take a look at how you might use the factory to test **user registration** — whether you're working with an API, UI, or both.  

---

## Testing an API Endpoint for User Registration

Let’s say you’re testing an API that allows users to register. The API expects the following request body:  

```json
{
  "name": "John Doe",
  "age": 30,
  "email": "john.doe@example.com",
  "role": "User"
}
```

You can use the `UserFactory` we created to generate test users dynamically. 

> :exclamation: This code doesn't work, it's an example! To work you'd need an actual endpoint and it would depend in the HTTP client you are using.

```typescript
// Import the factory
import { UserFactory } from "./UserFactory"; 

async function testRegisterUser(apiClient: any) {
  
  //Arrange
  // Generate a random user using the factory
  const newUser = UserFactory.createUser();

  //Act
  // Make the API request
  const response = await apiClient.post("/register", newUser);

  //Assert
  // Validate the response
  expect(response.status).toBe(201); 
  expect(response.data).toMatchObject({
    name: newUser.name,
    email: newUser.email,
    role: newUser.role,
  });

  console.log("User successfully registered:", response.data);
}
```

---

### **Why Use the Factory for API Testing?**  

1. **Dynamic Data**

> Using randomised data (via `faker.js` in the factory) helps uncover edge cases and prevents relying on hard-coded values.  

2. **Reusable Setup**

> The factory centralises test data creation, reducing duplication across tests.  


3. **Controlled Randomness**

> You can customise factory output to test specific scenarios (e.g., creating users with specific roles).  

4. **Test Atomicity**

> We don't rely on seed data, static data, or other tests to run before the test. The test is responsible for its own data.

---

## **2. Testing the UI for User Registration**  

When testing the **UI** for user registration, the factory simplifies test setup by ensuring valid user data is always available. In this example we are using Playwright.

```typescript
import { UserFactory } from "./UserFactory";
import { test, expect } from "@playwright/test";

test("User can register via the registration form", async ({ page }) => {

  const newUser = UserFactory.createUser();

  await page.goto("https://example.com/register");

  await page.fill("#name", newUser.name);
  await page.fill("#age", newUser.age.toString());
  await page.fill("#email", newUser.email);
  await page.selectOption("#role", newUser.role);

  await page.click("#register-button");

  await expect(page).toHaveURL("https://example.com/welcome");
  await expect(page.locator("#success-message")).toContainText("Registration successful");

  console.log("User registered via UI:", newUser);
});
```

---

### **Why Use the Factory for UI Testing?**  
1. **Consistency Across Tests**

> The factory ensures that all test data is valid and consistent with the application’s requirements.  

2. **Ease of Maintenance**

> If the user model changes (e.g., adding new fields), you only update the factory, not every individual test.  
3. **Customisation for Scenarios**

>You can tweak the factory to produce specific user data for testing different roles, invalid inputs, or edge cases.  

---

## **3. Example: Testing Edge Cases with the Factory**  

Sometimes, you want to simulate specific scenarios, such as registering users with unique roles or invalid inputs. The factory makes it easy!  

### **Testing Specific Roles**:
   ```typescript
   const adminUser = UserFactory.createUserForRole("Admin");
   const guestUser = UserFactory.createUserForRole("Guest");

   // Use these in your API or UI tests
   ```

### **Testing Invalid Data**:  
   If you want to test how the system handles invalid inputs, you can extend the factory to generate such data:  

   ```typescript
   // Extending the factory for invalid data
   const invalidUser = { ...UserFactory.createUser(), email: "invalid-email" };

   // Use invalidUser in your test
   const response = await apiClient.post("/register", invalidUser);
   expect(response.status).toBe(400); // Validate error handling
   ```

---

## **Summary**  

So how do factories and builders help when testing?

1. **They Save Time**
> A factory reduces boilerplate and makes test setup faster.  
2. **They Increase Test Coverage**
> By generating dynamic and varied data, you test *more* scenarios with *less* effort.  
3. **They are Reusable and Scalable**
> Factories can adapt as the application grows, ensuring tests remain maintainable.  

Using a factory like this helps you focus on what matters — testing behaviour — while letting the factory handle the grunt work of creating valid (or invalid) data.  

---

[How To Never Write a Builder Class Again >>](./chapter9.md)