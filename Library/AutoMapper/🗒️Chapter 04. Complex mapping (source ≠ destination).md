
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
    public AddressDTO addressDTO { get; set; }
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
    public string EmpCity { get; set; }
    public string EmpState { get; set; }
    public string Country { get; set; }
}
~~~
```

### 🧩Program
---
##### 프로퍼티명이 다른 경우 ForMember를 통해 매핑
* Address.City - AddressDTO.EmpCity
* Address.State - AddressDTO.EmpState
* Employee.address - EmployeeDTO.addressDTO
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
            .ForMember(dest => dest.EmpCity, act => act.MapFrom(src => src.City))
            .ForMember(dest => dest.EmpState, act => act.MapFrom(src => src.State));
            cfg.CreateMap<Employee, EmployeeDTO>()
            .ForMember(dest => dest.addressDTO, act => act.MapFrom(src => address));
        });

        var mapper = new Mapper(config);
        return mapper;
    }
}
~~~
```
