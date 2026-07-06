To keep your Program.cs completely clear of LINQ logic, scopes, and boilerplate, you can create a custom extension property or method directly on the app object.
Since C# does not allow custom nested properties like app.DebugHelper.DumpResult(...) natively through extensions, the cleanest, most standard C# approach is a direct extension method on app.
This achieves exactly what you want in just one single line of code inside Program.cs:

app.DumpRepositoryData<IProductRepository>("extension_test_output.json");

Here is how to set this up so all the dirty work stays hidden inside your helper files.
## Step 1: Create the Clean Extension Method
Create a file named Extensions/DebugExtensions.cs. This single method handles opening the safe scope, fetching your repository from dependency injection, running your specific LINQ query, and writing the JSON output.

using System;using System.IO;using System.Linq;using System.Text.Json;using Microsoft.AspNetCore.Builder;using Microsoft.Extensions.DependencyInjection;
public static class DebugExtensions
{
    private static readonly JsonSerializerOptions JsonOptions = new() { WriteIndented = true };

    /// <summary>
    /// One-liner to extract a repository, run queries, and dump results to a file and console.
    /// </summary>
    public static WebApplication DumpRepositoryData<TRepository>(this WebApplication app, string fileName = "debug_output.json")
        where TRepository : class
    {
        // 1. Automatically handle the scope behind the scenes
        using (var scope = app.Services.CreateScope())
        {
            // 2. Automatically resolve the repository
            var repository = scope.ServiceProvider.GetRequiredService<TRepository>();

            // 3. Cast or call your specific query logic safely inside the scope
            // Note: If TRepository is IProductRepository, execute your logic here:
            if (repository is IProductRepository productRepo)
            {
                var queryResult = productRepo.GetAllProducts()
                                             .Where(p => p.Price > 50)
                                             .OrderByDescending(p => p.Price)
                                             .Select(p => new { p.Id, p.Name, p.Price })
                                             .ToList();

                // 4. Serialize to JSON
                string jsonString = JsonSerializer.Serialize(queryResult, JsonOptions);

                // 5. Print to Console
                Console.WriteLine("\n========== LINQ QUERY RESULT ==========");
                Console.WriteLine(jsonString);
                Console.WriteLine("=======================================\n");

                // 6. Save to File
                File.WriteAllText(fileName, jsonString);
                Console.WriteLine($"[Debug] Output successfully written to: {Path.GetFullPath(fileName)}");
            }
            else
            {
                Console.WriteLine($"[Debug Error] Query logic not defined for repository type: {typeof(TRepository).Name}");
            }
        }

        return app;
    }
}

## Step 2: Your Ultra-Clean Program.cs
Now your Program.cs stays perfectly pristine. You do not need to look at lambda expressions, nested grouping logic, or service resolution code here.
Just drop this single line right before app.Run();:

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddScoped<IProductRepository, ProductRepository>();var app = builder.Build();
// ----- THE ONLY LINE YOU NEED FOR TESTING -----
app.DumpRepositoryData<IProductRepository>("extension_test_output.json");// ----------------------------------------------

app.Run();

## How to adapt this for your complex grouping logic
If you want this single line to run your complex reportList grouping query instead, simply swap out the inner query block inside DebugExtensions.cs like this:

// Inside DebugExtensions.cs -> if (repository is ISalesRepository salesRepo)var flatData = salesRepo.GetFlatSalesData();var reportList = flatData
    .GroupBy(x => new { x.DateCreated, x.RevenueCenterName })
    .Select(group => new SalesItemReportDtov2 { /* ... your grouping query ... */ })
    .ToList();
string jsonString = JsonSerializer.Serialize(reportList, JsonOptions);
File.WriteAllText(fileName, jsonString);

If you want to keep expanding this tool belt, let me know:

* Do you want to see how to pass runtime arguments into this single line (like app.DumpRepositoryData<IProductRepository>("output.json", minimumPrice: 100))?
* Should we configure this extension to only execute during local development so it safely ignores itself on production servers?


