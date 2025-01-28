# Conclusion: Refactoring from Bad Code

Refactoring the initial, poorly designed implementation into the current, structured solution highlights the power of software design principles. 

Here's a summary of the journey, key benefits, and important takeaways to remember when you undertake a similar refactoring.

---

## **Key Improvements**

1. **Encapsulation and Separation of Concerns**
   - In the initial implementation, all logic was crammed into a single function, violating **separation of concerns**.  
   - The refactor split responsibilities across classes:
     - `UserProfile` models the user.
     - `Role` encapsulates role-related logic and permissions.
     - `UserProfileBuilder` focuses solely on assembling a `UserProfile` instance.
     - `UserProfileFactory` provides a high-level API for creating preconfigured users.

2. **Type Safety**
   - The initial implementation used raw strings and arrays, leaving room for invalid or mismatched data (e.g., incorrect permissions for roles).  
   - By introducing types (`Role`, `Permission`, and `UserProfile`), we ensured that invalid states are impossible to represent. 
     - Roles now include predefined permissions.
     - Builders enforce the completeness of user profiles before building them.

3. **Flexibility Through Extensibility**
   - Adding new roles, permissions, or user configurations is now straightforward.  
   - The `Role` class centralises role-related logic, making it the single source of truth for role definitions and permissions.

4. **Readability and Maintainability**
   - Code readability improved significantly by adopting patterns like the **Builder** and **Factory**.
     - The **Builder** provides a step-by-step mechanism for creating complex objects.
     - The **Factory** abstracts repetitive creation logic, making user creation simpler and more consistent.
   - The separation of concerns ensures each component is easier to test, debug, and modify without unintended side effects.

---

### **Key Benefits of Refactoring**

1. **Reduced Risk of Bugs**

   The initial implementation was prone to errors because it relied on manual data passing and lacked guardrails. Now, roles and permissions are centralised and validated by design.

2. **Scalability**

   The codebase is now extensible. Adding new roles, permissions, or user types is easy without modifying existing code, adhering to the **Open-Closed Principle**.

3. **Improved Developer Experience**

   By providing clear abstractions (builder, factory, role), the API is intuitive and difficult to misuse. Developers can focus on business logic rather than worrying about invalid configurations.

---

### **Things to Remember When Refactoring**

1. **Identify Pain Points**
   
   Before refactoring, understand what makes the current implementation problematic. In this case, it was the lack of type safety, poor organisation, and risk of invalid states.

2. **Introduce Strong Types Early**
   
   Strong typing in TypeScript is one of its greatest strengths. Use it to your advantage to enforce constraints, reduce errors, and improve code clarity.

3. **Start Small and Incrementally Improve**
   
   Refactor step by step, ensuring each change simplifies the code, improves readability, or eliminates potential bugs. For example, we started by splitting the code into smaller functions and progressed to introducing builders and factories.

4. **Keep Each Component Focused**
   - Adhere to **Single Responsibility Principle**: Each class or function should have one well-defined purpose.  
   - For example, the `Role` class handles roles and permissions, while the `UserProfileBuilder` focuses on assembling user objects.

5. **Builders Are Dumb, Factories Are Smart**
   - Builders should be simple and focused only on assembling objects.  
   - Factories can contain logic to decide how to create objects (e.g., assigning roles based on predefined configurations).

---

## Final Thought

Refactoring is about more than just improving the structure of your code — it’s about making the system easier to understand, maintain, and extend. The journey from bad code to a robust implementation illustrates key software engineering principles, including **encapsulation**, **type safety**, **separation of concerns**, and **readability**. 

When undertaking this yourself, always keep the following in mind:
- Refactor incrementally and iteratively.
- Prioritise strong typing and constraints to avoid invalid states.
- Leverage proven patterns (Builder, Factory, etc.) to simplify object creation.

I hope you enjoyed the journey! 

---

[Next Chapter >>](./chapter8.md)