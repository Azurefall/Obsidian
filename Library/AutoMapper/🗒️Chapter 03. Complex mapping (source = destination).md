
# ✍️ How to use

### 🧩Models
---
```ad-note
collapse: close
title: Employee.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class Employee
{
	public string Name { get; set; }
    public int Salary { get; set; }
    public string Department { get; set; }
    public Address address { get; set; }
}
~~~
```

```ad-note
collapse: close
title: Address.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class Address
{
	public string City { get; set; }
	public string State { get; set; }
	public string Country { get; set; }
}
~~~
```

### 🧩Data objects
---
##### Employee.address(Address type) - EmployeeDTO.address(AddressDTO type)
```ad-note
collapse: close
title: EmployeeDTO.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class EmployeeDTO
{
	public string Name { get; set; }
    public int Salary { get; set; }
    public string Department { get; set; }
    public AddressDTO address { get; set; }
}
~~~
```

```ad-note
collapse: close
title: AddressDTO.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class AddressDTO
{
    public string City { get; set; }
    public string State { get; set; }
    public string Country { get; set; }
}
~~~
```

### 🧩Program
---
##### 프로퍼티명이 동일한 경우
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
        Address empAddres = new Address()
        {
            City = "Mumbai",
            State = "Maharashtra",
            Country = "India"
        };

        Employee emp = new Employee
        {
            Name = "James",
            Salary = 20000,
            Department = "IT",
            address = empAddres
        };

        var mapper = InitializeAutomapper();
        var empDTO = mapper.Map<EmployeeDTO>(emp);

        Console.WriteLine("Name:" + empDTO.Name + ", Salary:" + empDTO.Salary + ", Department:" + empDTO.Department);
        Console.WriteLine("City:" + empDTO.address.EmpCity + ", State:" + empDTO.address.EmpState + ", Country:" + empDTO.address.Country);
        Console.ReadLine();
    }
    
    static Mapper InitializeAutomapper()
    {
        var config = new MapperConfiguration(cfg => {
            cfg.CreateMap<Address, AddressDTO>()
            cfg.CreateMap<Employee, EmployeeDTO>()
        });

        var mapper = new Mapper(config);
        return mapper;
    }
}
~~~
```
