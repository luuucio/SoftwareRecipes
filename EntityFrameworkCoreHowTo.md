# Entity Framework Core on .Net Core HOW-TO

## Installation
Install from NuGet:
- Microsoft.EntityFrameworkCore.SqlServer
- Microsoft.EntityFrameworkCore.Design

## Entities
We need to create entities, usually in the Models folder

```csharp
    public class Product
    {
        // Use "Id" or "ProductId" to use the naming convention, otherwise use the [Key] attribute
        public int Id { get; set; }
        
        [Required]
        public string Name { get; set; }
        
        [Column(TypeName="decimal(18,2)")]
        public decimal Price { get; set; }
    }
```

For nullable types use the nullable context:

```csharp
    public class Customer
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
#nullable enable
        public string? Address { get; set; }
        public string? Phone { get; set; }
        public string? Email { get; set; }
#nullable disable
        // Navigation property, the customer may have zero or more orders. It generates a one-to-many relationship in the database
        public ICollection<Order> Orders { get; set; }
    }
```    

The Order class:

```csharp
    public class Order
    {
        public int Id { get; set; }
        public DateTime OrderPlaced { get; set; }
        public DateTime? OrderFulfilled { get; set; }
        
        // The foreign key for the relationship. If we didn't include the property, Entity Framework Core
        // would create it anyway as a shadow property
        public int CustomerId { get; set; }
        
        // Navigation property, one customer per order
        public Customer Customer { get; set; }
        
        // Another navigation property for the junction table
        public ICollection<ProductOrder> ProductOrders { get; set; }
    }
```

The junction table:

```csharp
    public class ProductOrder
    {
        public int Id { get; set; }
        public int Quantity { get; set; }
        
        // Foreign key, it isn't strictly required
        public int ProductId { get; set; }
        
        public Product Product { get; set; }       
        
        // Foreign key, it isn't strictly required      
        public int OrderId { get; set; }
        
        public Order Order { get; set; }
    }
```

## Context
We need to create a context, which represent a database session, usually in the Data folder

```csharp
    public class StoreContext : DbContext
    {
        public DbSet<Customer> Customers { get; set; }
        public DbSet<Product> Products { get; set; }
        public DbSet<Order> Orders { get; set; }
        public DbSet<ProductOrder> ProductOrders { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            // Never hardcode the connection string
            optionsBuilder.UseSqlServer(@"Data Source=(localdb)\MSSQLLocalDB;Database=StoreContext;Trusted_Connection=True;");
        }
    }
```


## Migrations
Migrations are tool to create and evolve the database. From the command line:

```
dotnet ef migrations add MIGRATION_NAME
```

We should look at the generated code in the Migrations folder to be sure it performs what we need. Then we run the migration:

```
dotnet ef database update MIGRATION_NAME
```

## CRUD - Create

```csharp
    var tennisBall = new Product
    {
        Name = "Tennis Ball 3-Pack",
        Price = 9.99M
    };
    
    context.Add(tennisBall);
    
    context.SaveChanges();
```

## CRUD - Read and modify a single value

```csharp
    var squeakyBone = context.Products.Where(x => x.Name == "Squeaky Dog Bone").FirstOrDefault();
    
    if (squeakyBone is Product)
    {
        squeakyBone.Price = 7.99m;
    }
    context.SaveChanges();
```    
    
## CRUD - Read multiple values

```csharp
    var products = context.Products.Where(x => x.Price > 5.00m).OrderBy(x => x.Name);
    
    foreach (var product in products)
    {
        Console.WriteLine($"{product.Name}: {product.Price} $");
    }
```

## CRUD - Delete a value

```csharp
    var squeakyBone = context.Products.Where(x => x.Name == "Squeaky Dog Bone").FirstOrDefault();
    
    if (squeakyBone is Product)
    {
        context.Remove(squeakyBone);
    }
    context.SaveChanges();
```    

## Working with an existing database

From the command line, launch:

```
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Database=StoreContext;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -o Models -c StoreContext -d
```

If the -d flag is omitted, annotations will not be generated and the behavior will be implemented in the fluent API

## Lazy loading

Because of the use of the Include method, this statement will load all the orders for a specific customer:

```csharp
var customer = await _context.Customers.Include(c => c.Orders).SingleAsync(c => c.Id == id);
```

If we want lazy loading, we have to:
- get rid of the Include method:

```csharp
var customer = await _context.Customers.SingleAsync(c => c.Id == id);
```

- install from NuGet the Microsoft.EntityFrameworkCore.Proxies package
- in Startup.cs, modify the ConfigureServices method, adding UseLazyLoadingProxies:

```csharp
services.AddDbContext<StoreContext>(options => options.UseLazyLoadingProxies().UseSqlServer(CONNECTION_STRING);
```

- modify the navigation properties, marking them as **virtual**, so Entity Framework Core can override them with the proxies it generates. The orders will be now loaded only when actuallty needed by the code
