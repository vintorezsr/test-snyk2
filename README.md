## Overview
`Lyte.Libraries` is a collection of modular libraries designed to simplify and streamline the development process in .NET applications. Each library focuses on a specific area, providing robust, reusable components that can be easily integrated into your projects.

## Feature
* Permission-based access control for managing user authorization.
* A module system for structuring your application's components and configurations.
* Support for background services and integration with Hangfire for handling long-running tasks.
* Integration with notification services such as MediatR and Microsoft Teams for delivering notifications to users.
* Base classes for creating read-only and CRUD API controllers, along with Refit integration for simplified setup and management of API clients.
* Support for various storage providers, including AWS S3 integration for storing and retrieving files in your application.
* Support for various log providers, including AWS CloudWatch integration for logging and monitoring.
* Supports audit logging for tracking changes to application data.

## Modules
### Lyte.Libraries.Common
Offers a set of common utilities and helpers for application development, such as extensions, converters, and common data structures.

[See the full documentation](https://lyteventures.atlassian.net/wiki/spaces/TECH/pages/1505460504)

### Lyte.Libraries.Domain
Provides a solid foundation for domain-driven design with base classes, interfaces, and essential building blocks for creating rich domain models.

[See the full documentation](https://lyteventures.atlassian.net/wiki/spaces/TECH/pages/1505525998)

### Lyte.Libraries.DataTables
Facilitates seamless integration with jQuery DataTables, offering server-side processing, filtering, and sorting capabilities for efficient data management.

[See the full documentation](https://lyteventures.atlassian.net/wiki/spaces/TECH/pages/1505558787)

### Lyte.Libraries.Web
Enhances web application development with features such as custom authorization, background services, notifications, and storage integrations, as well as utilities for Web API and MVC.

[See the full documentation](https://lyteventures.atlassian.net/wiki/spaces/TECH/pages/1505493233)

### Lyte.Libraries.Web.Mvc
Offers additional support for ASP.NET MVC projects, providing reusable components such as base CRUD controllers, custom components, and custom filters.

[See the full documentation](https://lyteventures.atlassian.net/wiki/spaces/TECH/pages/1505558794)

## Getting Started
### Installation
If you are working on an API project run the following command:

```powershell
Install-Package Lyte.Libraries.Web
```

If you are working on an MVC project run the following command:

```powershell
Install-Package Lyte.Libraries.Web.Mvc
```

### Creating DbContext
Your `DbContext` class should be derived from `DbContextBase` as shown below:

```csharp
using Lyte.Libraries.Domain;

public class ApplicationDbContext : DbContextBase
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> dbContextOptions)
        : base(dbContextOptions)
    {
    }
}
```

### Defining the Entity
Define two domain models, Customer, and Order, which inherit from `EntityBase`. By doing so, each model automatically gains an integer Id property, along with `CreatedAt` and `UpdatedAt` properties for auditing as shown below:

```csharp
using Lyte.Libraries.Domain.Entities;

// Define your domain model by inheriting from EntityBase
public class Customer : EntityBase
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

public class Order : EntityBase
{
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }
    public decimal TotalPrice { get; set; }
    public DateTime OrderDate { get; set; }
}
```

### Creating the Data Transfer Object (DTO)
Create a Data Transfer Object (DTO) for each of the two domain models defined earlier as shown below:

```csharp
using Lyte.Libraries.Domain.Entities;

public class CustomerOutput
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

public class CustomerCreate
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

public class CustomerUpdate : IDefaultIdentity
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

public class OrderOutput
{
    public int Id { get; set; }
    public CustomerOutput Customer { get; set; }
    public decimal TotalPrice { get; set; }
    public DateTime OrderDate { get; set; }
}

public class OrderCreate
{
    public int CustomerId { get; set; }
    public decimal TotalPrice { get; set; }
    public DateTime OrderDate { get; set; }
}

public class OrderUpdate : IDefaultIdentity
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public decimal TotalPrice { get; set; }
    public DateTime OrderDate { get; set; }
}
```

### Creating the Domain Service
Based on the two previously defined domain models, we can now create a domain service for each model, as shown below:

```csharp
using Lyte.Libraries.Domain.Services;

public class CustomerService
    : CrudServiceBase<Customer, CustomerOutput, CustomerCreate, CustomerUpdate>
{
    public CustomerService(DbContext dbContext, IMapper mapper, ILogger<CustomerService> logger)
        : base(dbContext, mapper, logger)
    {
    }
}

public class OrderService
    : CrudServiceBase<Order, OrderOutput, OrderCreate, OrderUpdate>
{
    public OrderService(DbContext dbContext, IMapper mapper, ILogger<OrderService> logger)
        : base(dbContext, mapper, logger)
    {
    }
}
```

### Configure AutoMapper Profile
Set up the `AutoMapper` profile using the DTO classes created earlier:

```csharp
using AutoMapper;

public class DefaultMappingProfile : Profile
{
    public DefaultMappingProfile()
    {
        CreateMap<Customer, CustomerOutput>(MemberList.None);
        CreateMap<CustomerCreate, Customer>(MemberList.None);
        CreateMap<CustomerUpdate, Customer>(MemberList.None);
        
        CreateMap<Order, OrderOutput>(MemberList.None);
        CreateMap<OrderCreate, Order>(MemberList.None);
        CreateMap<OrderUpdate, Order>(MemberList.None);
    }
}
```

### Configure the Bootstrapper
The minimum configuration for the `Bootstrapper` in the `Program.cs` file is as follows:

```csharp
using Lyte.Libraries.Web;

var bootstrapper = new Bootstrapper(args);
bootstrapper.AddEntityFramework<ApplicationDbContext>(AppConstants.MySqlDbConnectionName)
    .AddAutoMapper<DefaultMappingProfile>()
    .MinimalSetup();
```

### The CRUD Controller Implementation
Here's an example of usage:

**CustomersController**

```csharp
using Lyte.Libraries.Web.Controllers;

public class CustomersController
    : CrudControllerBase<CustomerOutput, CustomerCreate, CustomerUpdate>
{
    public CustomersController(CustomerService customerService)
        : base(customerService)
    {
    }
}
```

**OrdersController**

```csharp
using Lyte.Libraries.Web.Controllers;

public class OrdersController
    : CrudControllerBase<OrderOutput, OrderCreate, OrderUpdate>
{
    public OrdersController(OrderService orderService)
        : base(orderService)
    {
    }
}
```