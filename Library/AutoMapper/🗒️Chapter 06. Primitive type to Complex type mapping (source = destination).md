
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
    public string City { get; set; }
    public string State { get; set; }
    public string Contry { get; set; }
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
    public Address address { get; set; }
}
~~~
```

### 🧩Program
---
##### ForMember를 통한 Type 변환 
* Employee.City - EmployeeDTO.address.City
* Employee.State - EmployeeDTO.address.State
* Employee.Country - EmployeeDTO.address.Country
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
        Employee emp = new Employee();
		emp.Name = "James";
		emp.Salary = 20000;
		emp.Department = "IT";
		emp.City = "Mumbai";
		emp.State = "Maharashtra";
		emp.Country = "India";

        var mapper = InitializeAutomapper();
        var empDTO = mapper.Map<EmployeeDTO>(emp);

        Console.WriteLine("Name:" + empDTO.Name + ", Salary:" + empDTO.Salary + ", Department:" + empDTO.Department);
        Console.WriteLine("City:" + empDTO.address.EmpCity + ", State:" + empDTO.address.EmpState + ", Country:" + empDTO.address.Country);
        Console.ReadLine();
    }
    
    static Mapper InitializeAutomapper()
    {
        var config = new MapperConfiguration(cfg => {
            cfg.CreateMap<Employee, EmployeeDTO>()
            .ForMember(dest => dest.address, act => act.MapFrom(src => new Address()
            {
	            City = src.City,
	            State = src.State,
	            Country = src.Country
            }));
        });

        var mapper = new Mapper(config);
        return mapper;
    }
}
~~~
```
