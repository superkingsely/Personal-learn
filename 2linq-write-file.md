To access your injected context, repository, or service inside Program.cs, you must extract them from the application's built-in dependency injection container (app.Services) after the application is built (builder.Build()), but before it runs (app.Run()). [1] 
Because these database contexts and services usually depend on a specific web request lifetime (Scoped), you must wrap your test code in a CreateScope() block to avoid memory leaks or crash errors. [2, 3] 
Here is exactly how to update your Program.cs and your static method to pass and use your injected dependencies.
## Step 1: Update Program.cs to Fetch the Injected Service
Place this block right after var app = builder.Build();.

var builder = WebApplication.CreateBuilder(args);
// Register your actual services/repositories here as usual
builder.Services.AddDbContext<MyDbContext>(); // Example context
builder.Services.AddScoped<IProductRepository, ProductRepository>(); // Example repo
var app = builder.Build();
// ----- QUICK SERVICE/DB TESTING BLOCK -----// 1. Create a scope to safely request scoped services (like DbContext or Repositories)using (var scope = app.Services.CreateScope())
{
    // 2. Resolve your injected service, repository, or context from the provider
    var myInjectedRepo = scope.ServiceProvider.GetRequiredService<IProductRepository>();

    // 3. Execute your static test method, passing the injected repo down into it
    DebugHelper.TestMyLinqQuery(myInjectedRepo);
}// ------------------------------------------

app.Run();

## Step 2: Update Your Helper Class to Accept the Service
Modify your DebugHelper class to accept your service or repository interface as a parameter. This allows you to write and run your LINQ queries directly inside the helper method.

using System;using System.IO;using System.Linq;using System.Text.Json;
public static class DebugHelper
{
    private static readonly JsonSerializerOptions JsonOptions = new() { WriteIndented = true };

    // Pass your injected repository or context into this method
    public static void TestMyLinqQuery(IProductRepository repository)
    {
        // 1. Run your actual LINQ query using the injected repository
        var queryResult = repository.GetAllProducts()
                                    .Where(p => p.Price > 50)
                                    .OrderByDescending(p => p.Price)
                                    .Select(p => new { p.Id, p.Name, p.Price })
                                    .ToList();

        // 2. Serialize the LINQ results to JSON string
        string jsonString = JsonSerializer.Serialize(queryResult, JsonOptions);

        // 3. Log to Console
        Console.WriteLine("========== LINQ QUERY RESULT ==========");
        Console.WriteLine(jsonString);
        Console.WriteLine("=======================================");

        // 4. Save to File
        File.WriteAllText("linq_db_output.json", jsonString);
    }
}

## Why this works perfectly:

* No Manual Mocking: It uses your real application settings, connection strings, and database data.
* Safe Lifetime Management: The using statement ensures that the database connection closes automatically as soon as the test finishes logging, preventing resource leaks before your API starts listening for network requests. [4, 5] 

------------------------------
If you'd like, let me know:

* What is the exact name of your repository or context class/interface?
* What specific LINQ query (filters, joins, or groupings) are you trying to run against it?

I can write out the exact C# code block tailored to your real data structure.

[1] [https://blazorschool.com](https://blazorschool.com/tutorial/blazor-server/dotnet7/dependency-injection-785774)
[2] [https://learn.microsoft.com](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-10.0)
[3] [https://blog.devgenius.io](https://blog.devgenius.io/real-life-examples-for-service-lifetimes-in-net-core-60494e3bcdef)
[4] [https://thefullstack.engineer](https://thefullstack.engineer/full-stack-development-series-part-7-unit-and-integration-testing/)
[5] [https://bytehide.com](https://bytehide.com/blog/idisposable-method-csharp)
