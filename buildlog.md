


using System.Text.Json;
using Microsoft.EntityFrameworkCore;

public static class Configure
{
    public static void ConfigureServices(this IServiceCollection services, IConfiguration configuration)
    {
        services.AddOpenApi();
        // Add DbContext with SQLite provider
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlite(configuration.GetConnectionString("DefaultConnection")));

        // Add other services here
        services.AddScoped<StudentService>();
        services.AddControllers();
    }
    
    public static void ConfigureApp(this WebApplication app)
    {
        // Configure the HTTP request pipeline.
        if (app.Environment.IsDevelopment())
        {
            app.MapOpenApi();

            // Serves the Swagger UI at /swagger
            app.UseSwaggerUI(options =>
            {
                options.SwaggerEndpoint("/openapi/v1.json", "Cj API V1");
            });
        }

        // app.UseHttpsRedirection();
        app.MapControllers();
    }

    // New method to log data right after build/startup
    public static async Task LogStudentsAtStartupAsync(this WebApplication app)
    {
        // Create a scope to resolve Scoped services (like AppDbContext)
        using var scope = app.Services.CreateScope();
        // var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        var studentservice=scope.ServiceProvider.GetRequiredService<StudentService>();

        try
        {
            // just call the method
            var studentList=await studentservice.GetAllStudentsAsync();
        
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error logging students at startup: {ex.Message}");
        }
    }

}
===============================================
extention method


using System.Text.Json;

public static class Logging
{
    public static string LogspecData<T>(this T data )
    {
        var logdata=JsonSerializer.Serialize(data,new JsonSerializerOptions{WriteIndented=true});
        return logdata;
    }
}
=======================================================
the service method
  public async Task<List<Student>> GetAllStudentsAsync()
    {

        var studentlist=await context.Students.ToListAsync();
        var groupstud=studentlist.GroupBy(obj=>obj.Age);
        var datatolog=groupstud.LogspecData();

       var secondway= await Logvalue<string>(datatolog);
        Console.WriteLine($"cool1==={datatolog}");
        Console.WriteLine($"cool2==={secondway}");



        // now hw do i log the studentlist to the console when the app run?
        // Console.WriteLine("====Retrieved students:=====");
        // foreach (var student in studentlist)
        // {
        //     Console.WriteLine($" - {student.Name}, Age: {student.Age}");
        // }   
        // what of when the app build? i want to console wen the app build only
        return studentlist;


    }
    =============================================
    program .cs
    // Call the startup logging method before running the app
await app.LogStudentsAtStartupAsync();