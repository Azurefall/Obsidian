
# ✍️ How to use

### 🧩Model
---
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
	public string Address { get; set; }
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
}
~~~
```

### 🧩Program
---
##### 특정 프로퍼티를 제외하고 매핑
* Address 프로퍼티는 제외
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
        Console.WriteLine("ID : " + employee.ID + ", Name : " + employee.Name + ", Address : " + employee.Address);
        Console.WriteLine();
        Console.WriteLine("After Mapping : EmployeeDTO");
        Console.WriteLine("ID : " + empDTO.ID + ", Name : " + empDTO.Name + ", Address : " + empDTO.Address);
        Console.ReadLine();
    }

    static Mapper InitializeAutomapper()
    {
        var config = new MapperConfiguration(cfg =>
        {
            cfg.CreateMap<Employee, EmployeeDTO>()

                //Ignoring the Address property of the destination type
                .ForMember(dest => dest.Address, act => act.Ignore());
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
ID : 101, Name : James, Address : Mumbai

After mapping : EmployeeDTO
ID : 101, Name : James, Address : 
~~~
```
