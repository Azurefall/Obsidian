
# ✍️ How to use

### 🧩Model
---
##### 제외할 프로퍼티에 [NoMap] Attribute 설정
```ad-note
collapse: close
title: Employee.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class Employee
{
    public int ID { get; set; }
    public string Name { get; set; }
    [NoMap]
    public string Address { get; set; }
    [NoMap]
    public string Email { get; set; }
}
~~~
```

### 🧩Data object
---
```ad-note
collapse: close
title: EmployeeDTO.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class EmployeeDTO
{
    public int ID { get; set; }
    public string Name { get; set; }
    public string Address { get; set; }
    public string Email { get; set; }
}
~~~
```

### 🧩Classes
---
```ad-note
collapse: close
title: NoMapAttribute.cs
~~~csharp
using AutoMapper;

namespace AutoMapperDemo;

public class NoMapAttribute : System.Attribute
{
}
~~~
```

```ad-note
collapse: close
title: IgnoreNoMapExtensions.cs
~~~csharp
using AutoMapper;

namespace AutoMapperDemo;

public static class IgnoreNoMapExtensions
{
    public static IMappingExpression<TSource, TDestination> IgnoreNoMap<TSource, TDestination>(
        this IMappingExpression<TSource, TDestination> expression)
    {
        var sourceType = typeof(TSource);
        foreach (var property in sourceType.GetProperties())
        {
            PropertyDescriptor descriptor = TypeDescriptor.GetProperties(sourceType)[property.Name];
            NoMapAttribute attribute = (NoMapAttribute)descriptor.Attributes[typeof(NoMapAttribute)];
            if (attribute != null)
                expression.ForMember(property.Name, opt => opt.Ignore());
        }
        return expression;
    }
}
~~~
```

### 🧩Program
---
```ad-note
collapse: open
title: Program.cs
~~~csharp
using AutoMapper;

namespace AutoMapperDemo;

class Program
{
    static void Main(string[] args)
    {
        var mapper = InitializeAutomapper();

        Employee employee = new Employee()
        {
            ID = 101,
            Name = "James",
            Address = "Mumbai"
        };

        var empDTO = mapper.Map<Employee, EmployeeDTO>(employee);

        Console.WriteLine("After Mapping : Employee");
        Console.WriteLine("ID : " + employee.ID + ", Name : " + employee.Name + ", Address : " + employee.Address + ", Email : " + employee.Email);
        Console.WriteLine();
        Console.WriteLine("After Mapping : EmployeeDTO");
        Console.WriteLine("ID : " + empDTO.ID + ", Name : " + empDTO.Name + ", Address : " + empDTO.Address + ", Email : " + empDTO.Email);
        Console.ReadLine();
    }

    static Mapper InitializeAutomapper()
    {
        var config = new MapperConfiguration(cfg =>
        {
            cfg.CreateMap<Employee, EmployeeDTO>()
            .IgnoreNoMap();
        });

        var mapper = new Mapper(config);
        return mapper;
    }
}
~~~
```

```ad-success
collapse: open
title: Result
~~~powershell
After mapping : Employee
ID : 101, Name : James, Address : Mumbai, Email:

After mapping : EmployeeDTO
ID : 101, Name : James, Address : , Email :
~~~
```
