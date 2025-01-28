# Other Examples

In C# you have to might some slight changes, but it's pretty similar.

---

### **1. Role Class**

```csharp
// Models/Role.cs
using System.Collections.Generic;

namespace UserManagement.Models
{
    public class Role
    {
        public string Name { get; }
        public List<string> Permissions { get; }

        private Role(string name, List<string> permissions)
        {
            Name = name;
            Permissions = permissions;
        }

        public static Role Admin()
        {
            return new Role("Admin", new List<string> { "Read", "Write", "Delete" });
        }

        public static Role Editor()
        {
            return new Role("Editor", new List<string> { "Read", "Write" });
        }

        public static Role Viewer()
        {
            return new Role("Viewer", new List<string> { "Read" });
        }

    }
}
```

---

### **2. UserProfile Class**

```csharp
// Models/UserProfile.cs
namespace UserManagement.Models
{
    public class UserProfile
    {
        public string Name { get; }
        public int Age { get; }
        public string RoleName { get; }
        public List<string> Permissions { get; }

        public UserProfile(string name, int age, string roleName, List<string> permissions)
        {
            Name = name;
            Age = age;
            RoleName = roleName;
            Permissions = permissions;
        }

        public override string ToString()
        {
            return $"{Name}, Age: {Age}, Role: {RoleName}, Permissions: {string.Join(", ", Permissions)}";
        }
    }
}
```

---

### **3. UserProfileBuilder Class**

```csharp
// Builders/UserProfileBuilder.cs
using UserManagement.Models;

namespace UserManagement.Builders
{
    public class UserProfileBuilder
    {
        private string _name;
        private int _age;
        private Role _role;

        public UserProfileBuilder WithName(string name)
        {
            _name = name;
            return this;
        }

        public UserProfileBuilder WithAge(int age)
        {
            _age = age;
            return this;
        }

        public UserProfileBuilder WithRole(Role role)
        {
            _role = role;
            return this;
        }

        public UserProfile Build()
        {
            if (string.IsNullOrEmpty(_name) || _age <= 0 || _role == null)
            {
                throw new System.Exception("Incomplete profile: Name, Age, and Role are required.");
            }

            return new UserProfile(_name, _age, _role.Name, _role.Permissions);
        }
    }
}
```

---

### **4. UserProfileFactory Class**

```csharp
// Factories/UserProfileFactory.cs
using UserManagement.Models;
using UserManagement.Builders;
using Bogus; // Faker library for C#

namespace UserManagement.Factories
{
    public static class UserProfileFactory
    {
        private static Faker _faker = new Faker();

        public static UserProfile CreateAdmin()
        {
            return new UserProfileBuilder()
                .WithName(_faker.Name.FullName())
                .WithAge(_faker.Random.Int(18, 65))
                .WithRole(Role.Admin())
                .Build();
        }

        public static UserProfile CreateEditor()
        {
            return new UserProfileBuilder()
                .WithName(_faker.Name.FullName())
                .WithAge(_faker.Random.Int(18, 65))
                .WithRole(Role.Editor())
                .Build();
        }

        public static UserProfile CreateViewer()
        {
            return new UserProfileBuilder()
                .WithName(_faker.Name.FullName())
                .WithAge(_faker.Random.Int(18, 65))
                .WithRole(Role.Viewer())
                .Build();
        }

    }
}
```

---

### **5. Example Usage**

```csharp
// Program.cs
using System;
using UserManagement.Factories;
using UserManagement.Models;

class Program
{
    static void Main(string[] args)
    {
        var adminUser = UserProfileFactory.CreateAdmin();
        var editorUser = UserProfileFactory.CreateEditor();
        var viewerUser = UserProfileFactory.CreateViewer();

        Console.WriteLine(adminUser);
        Console.WriteLine(editorUser);
        Console.WriteLine(viewerUser);
    }
}
```
