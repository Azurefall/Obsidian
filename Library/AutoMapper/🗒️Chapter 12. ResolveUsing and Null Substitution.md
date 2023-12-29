
# ✍️ How to use

### 🧩Model
---
```ad-note
collapse: close
title: A.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class A
{
	public string Name { get; set; }
	public string AAddress { get; set; }
	public string AMobileNo { get; set; }
	public string FixedValue { get; set; }
	public DateTime DOJ { get; set; }
}
~~~
```

### 🧩Data object
---
```ad-note
collapse: close
title: B.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class B
{
	public string Name { get; set; }
	public string BAddress { get; set; }
	public string BMobileNo { get; set; }
	public string FixedValue { get; set; }
	public DateTime DOJ { get; set; }
}
~~~
```

### 🧩Program
---
##### Defult value 설정
* AMobileNo는  NullSubstitute를 통해 Null이면 "00011112222"로 매핑
* B.FixedValue는 UseValue를 통해 강제로 "Hello" 매핑
* B.DOJ는 ResolveUsing을 통해 정의된 DateTime.Now 로 매핑
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
		InitializeAutomapper();

		A aObj = new A()
		{
			Name = "Pranaya",
			AAddress = "Mumbai",
			AMobileNo = "01234567890"
		};

		var bObj = Mapper.Map<A, B>(aObj);
		Console.WriteLine("After Mapping : ");

		//Here FixedValue and DOJ will be empty for aObj
		Console.WriteLine("aObj.Member : " + aObj.Name);
		Console.WriteLine("aObj.FixedValue : " + aObj.FixedValue);
		Console.WriteLine("aObj.DOJ : " + aObj.DOJ);
		Console.WriteLine("aObj.AAddress : " + aObj.AAddress);
		Console.WriteLine("aObj.AMobile : " + aObj.AMobile);
		
		Console.WriteLine("bObj.Member : " + bObj.Name);
		Console.WriteLine("bObj.FixedValue : " + bObj.FixedValue);
		Console.WriteLine("bObj.DOJ : " + bObj.DOJ);
		Console.WriteLine("bObj.BAddress : " + bObj.BAddress);
		Console.WriteLine("bObj.BMobile : " + bObj.BMobile);

		Console.ReadLine();

		bObj.Name = "Rout";
		bObj.BAddress = "Delhi";
		bObj.BMobile = null;
		Mapper.Map(bObj, aObj);

		Console.WriteLine("After ReverseMap : ");
		Console.WriteLine("aObj.Member : " + aObj.Name);
		Console.WriteLine("aObj.FixedValue : " + aObj.FixedValue);
		Console.WriteLine("aObj.DOJ : " + aObj.DOJ);
		Console.WriteLine("aObj.AAddress : " + aObj.AAddress);
		Console.WriteLine("aObj.AMobile : " + aObj.AMobile);
		
		Console.WriteLine("bObj.Member : " + bObj.Name);
		Console.WriteLine("bObj.FixedValue : " + bObj.FixedValue);
		Console.WriteLine("bObj.DOJ : " + bObj.DOJ);
		Console.WriteLine("bObj.BAddress : " + bObj.BAddress);
		Console.WriteLine("bObj.BMobile : " + bObj.BMobile);

		Console.ReadLine();
	}

	static void InitializeAutomapper()
	{
		Mapper.Initialize(config =>
		{
			config.CreateMap<A, B>()
				.ForMember(dest => dest.BAddress, act => act.MapFrom(src => src.AAddress))

				.ForMember(dest => dest.BMobileNo, act => act.AMobileNo)
				//You need to use NullSubstitute method to substitute null value
				.ForMember(dest => dest.BMobileNo, act => act.NullSubstitute("00011112222"))

				//To Store Static Value use the UseValue() method
				.ForMember(dest => dest.FixedValue, act => act.UseValue("Hello"))

				//To Store DateTime value use ResolveUsing() method
				.ForMember(dest => dest.DOJ, act => act.ResolveUsing(src =>
				{
					return DateTime.Now;
				}))

				.ReverseMap();
		});
	}
}
~~~
```

```ad-success
collapse: open
title: Result
~~~powershell
After mapping
aObj.Member : Pranaya
aObj.FixedVa1ue : 
aObj.DOJ : 01-01-0001 00:00:00
aObj.AAddress : Mumbai
aObj.AMobileNo : 01234567890
bObj.Member : Pranaya
bObj.FixedValue : Hello
bObj.DOJ : 08-10-2018 12:34:56
bObj.AAddress : Mumbai
bObj.AMobileNo : 01234567890

After ReverseMap
aObj.Member : Rout
aObj.FixedVa1ue : Hello
aObj.DOJ : 08-10-2018 12:34:56
aObj.AAddress : Delhi
aObj.AMobileNo : 00011112222
bObj.Member : Rout
bObj.FixedValue : Hello
bObj.DOJ : 08-10-2018 12:34:56
bObj.AAddress : Delhi
bObj.AMobileNo : 
~~~
```
