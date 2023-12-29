
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
	public string Name { get; set; }
	public int Salary { get; set; }
	public string Address { get; set; }
	public string Department { get; set; }
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
	public string Name { get; set; }
	public int Salary { get; set; }
	public string Address { get; set; }
	public string Department { get; set; }
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
		//Initialize the mapper
		var config = new MapperConfiguration(cfg =>
				cfg.CreateMap<Employee, EmployeeDTO>()
			);

		//Creating the source object
		Employee emp = new Employee
		{
			Name = "James",
			Salary = 20000,
			Address = "London",
			Department = "IT"
		};

		//Using automapper
		var mapper = new Mapper(config);
		var empDTO = mapper.Map<EmployeeDTO>(emp);
		//OR
		//var empDTO2 = mapper.Map<Employee, EmployeeDTO>(emp);

		Console.WriteLine("Name:" + empDTO.FullName + ", Salary:" + empDTO.Salary + ", Address:" + empDTO.Address + ", Department:" + empDTO.Dept);
		Console.ReadLine();
	}
}
~~~
```
