
# ✍️ How to use

### 🧩Models
---
```ad-note
collapse: close
title: Order.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class Order
{
	public string OrderNo { get; set; }
    public int NumberOfItems { get; set; }
    public int TotalAmount { get; set; }
    public int CustomerId { get; set; }
    public string Name { get; set; }
    public string Postcode { get; set; }
    public string MobileNo { get; set; }
}
~~~
```

```ad-note
collapse: close
title: Customer.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class Customer
{
    public int CustomerID { get; set; }
    public string FullName { get; set; }
    public string Postcode { get; set; }
    public string ContactNo { get; set; }
}
~~~
```

### 🧩Data objects
---
```ad-note
collapse: close
title: OrderDTO.cs
~~~csharp
using Automapper;

namespace AutomapperDemo;

public class OrderDTO
{
    public int OrderId { get; set; }
    public int NumberOfItems { get; set; }
    public int TotalAmount { get; set; }
	public Customer Customer { get; set; } 
}
~~~
```

### 🧩Program
---
##### Model → Data object → 데이터 변경 → Model (ReverseMap 설정)
* 변경 : OrderNo, NumberOfItems, TotalAmount
* ForMember의 new Customer 수행으로 원본이 유지(Customerld, FullName, Postcode, ContactNo)
	→ ReverseMap 후 customer에 대한 ForMember 수행을 통해 정상 변경
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
        //Step1: Initialize the Mapper
        var mapper = InitializeAutomapper();

        //Step2: Create the Order Request
        var OrderRequest = CreateOrderRequest();

        //Step3: Map the OrderRequest object to Order DTO
        var orderDTOData = mapper.Map<Order, OrderDTO>(OrderRequest);

        //Step4: Print the OrderDTO Data
        Console.WriteLine("After Mapping - OrderDTO Data");
        Console.WriteLine("OrderId : " + orderDTOData.OrderId);
        Console.WriteLine("NumberOfItems : " + orderDTOData.NumberOfItems);
        Console.WriteLine("TotalAmount : " + orderDTOData.TotalAmount);
        Console.WriteLine("CustomerId : " + orderDTOData.customer.CustomerID);
        Console.WriteLine("FullName : " + orderDTOData.customer.FullName);
        Console.WriteLine("Postcode : " + orderDTOData.customer.Postcode);
        Console.WriteLine("ContactNo : " + orderDTOData.customer.ContactNo);
        Console.WriteLine();

        //Step5: modify the OrderDTO data
        orderDTOData.OrderId = 10;
        orderDTOData.NumberOfItems = 20;
        orderDTOData.TotalAmount = 2000;
        orderDTOData.CustomerId = 5;
        orderDTOData.Name = "James Smith";
        orderDTOData.Postcode = "12345";

        //Step6: Reverse Map
        mapper.Map(orderDTOData, OrderRequest);

        //Step7: Print the Order Data
        Console.WriteLine("After Reverse Mapping - Order Data");
        Console.WriteLine("OrderNo : " + OrderRequest.OrderNo);
        Console.WriteLine("NumberOfItems : " + OrderRequest.NumberOfItems);
        Console.WriteLine("TotalAmount : " + OrderRequest.TotalAmount);
        Console.WriteLine("CustomerId : " + OrderRequest.CustomerId);
        Console.WriteLine("Name : " + OrderRequest.Name);
        Console.WriteLine("Postcode : " + OrderRequest.Postcode);
        Console.WriteLine("MobileNo : " + OrderRequest.MobileNo);
        Console.ReadLine();
    }

    private static Order CreateOrderRequest()
    {
        return new Order
        {
            OrderNo = 101,
            NumberOfItems = 3,
            TotalAmount = 1000,
			CustomerID = 777,
			FullName = "James Smith",
			Postcode = "755019",
			ContactNo = "1234567890"
        };
    }

    static Mapper InitializeAutomapper()
    {
        var config = new MapperConfiguration(cfg => {
            cfg.CreateMap<Order, OrderDTO>()
                //OrderId is different so map them using For Member
                .ForMember(dest => dest.OrderId, act => act.MapFrom(src => src.OrderNo))

                //Customer is a Complex type, so Map Customer to Simple type using For Member
                .ForMember(dest => dest.customer, act => act.MapFrom(src => new Customer()
                {
	                CustomerID = src.CustomerId,
	                FullName = src.Name,
	                PostCode = src.PostCode,
	                ContactNo = src.MobileNo
                }))
                .ReverseMap();
				.ForMember(dest => dest.CustomerId, act => act.MapFrom(src => src.customer.CustomerID))
				.ForMember(dest => dest.Name, act => act.MapFrom(src => src.customer.FullName))
				.ForMember(dest => dest.MobileNo, act => act.MapFrom(src => src.customer.ContactNo))
				.ForMember(dest => dest.Postcode, act => act.MapFrom(src => src.customer.Postcode));
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
After mapping - OrderDTO Data
Orderld : 101
NumberOfItems : 3
Tota1Amount : 1000
Customerld : 777
Name : James Smith
Postcode : 755019
Mobi1eNo : 1234567890

After Reverse mapping - Order Data
OrderNo : 10
NumberOfItems : 20
Tota1Amount : 2000
Customerld : 5
FullName : James Smith
Postcode : 12345
ContactNo : 1234567890
~~~
```
