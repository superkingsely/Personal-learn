<!-- 
=============================================================
// New method to log data right after build/startup
    public static async Task LogStudentsAtStartupAsync(this WebApplication app)
    {
        // Create a scope to resolve Scoped services (like AppDbContext)
        using var scope = app.Services.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();

        try
        {
            var studentlist = await context.Students.ToListAsync();
            
            Console.WriteLine("\n=====================================");
            Console.WriteLine($"APP STARTED: Found {studentlist.Count} students:");
            Console.WriteLine("=====================================");
            foreach (var student in studentlist)
            {
                Console.WriteLine($" -> Name: {student.Name}, Age: {student.Age}");
            }
            Console.WriteLine("=====================================\n");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error logging students at startup: {ex.Message}");
        }
    }

============================================================

// Call the startup logging method before running the app
await app.LogStudentsAtStartupAsync(); -->