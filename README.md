# ASP.NET MVC with EF Core

Based on <https://docs.microsoft.com/en-us/aspnet/core/data/ef-mvc/intro?view=aspnetcore-5.0>
but EF is in a separate assembly

## Option 1: Original - all in 1 project

- create now ASP.NET MVC Web app: AspNetMvcEfcore.WebApp
- add an Class Library assembly which contains EF Core: AspNetMvcEfcore.Data

`dotnet new mvc --name ContosoUniversity`
`dotnet new sln --name ContosoUniversity`
`dotnet sln .\ContosoUniversity.sln add .\ContosoUniversity\ContosoUniversity.csproj`

The `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` NuGet package provides ASP.NET Core middleware for EF Core error pages. This middleware helps to detect and diagnose errors with EF Core migrations.

## Option 2: Separate project for ef (data)

> Serivice is optional (TODO: implement Repository pattern in Service assembly)

`dotnet new mvc --name ContosoUniversity`
`dotnet new classlib --name ContosoUniversity.Data`
`dotnet new classlib --name ContosoUniversity.Service`

`dotnet new sln --name ContosoUniversity`
`dotnet sln .\ContosoUniversity.sln add .\ContosoUniversity\ContosoUniversity.csproj .\ContosoUniversity.Data\ContosoUniversity.Data.csproj .\ContosoUniversity.Service\ContosoUniversity.Service.csproj`

ContosoUniversity.Service: Add reference to ContosoUniversity.Data
ContosoUniversity: Add reference to ContosoUniversity.Service, ContosoUniversity.Data

Add EF core to ContosoUniversity.Data:

## For both install the following packages

Design is needed for scaffolding:

```cmd
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Design
```

The `Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` NuGet package provides ASP.NET Core middleware for EF Core error pages. This middleware helps to detect and diagnose errors with EF Core migrations.

Add Context:

```cs
using ContosoUniversity.Models;
using Microsoft.EntityFrameworkCore;

namespace ContosoUniversity.Data
{
    public class SchoolContext : DbContext
    {
        public SchoolContext(DbContextOptions<SchoolContext> options) : base(options)
        {
        }

        public DbSet<Course> Courses { get; set; }
        public DbSet<Enrollment> Enrollments { get; set; }
        public DbSet<Student> Students { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Course>().ToTable("Course");
            modelBuilder.Entity<Enrollment>().ToTable("Enrollment");
            modelBuilder.Entity<Student>().ToTable("Student");
        }
    }
}
```

Register context in Startup.cs:

```cs
services.AddDbContext<SchoolContext>(options =>
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
```

Add to AppSettings.json to use your Sql Server:

```json
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost\\sqlexpress;Database=ContosoUniversity1;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  ```

 or use LocalDB:

By default, LocalDB creates .mdf DB files in the `C:/Users/<user>` directory.

```json
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=ContosoUniversity1;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
```

Add Database Exception Filter:

```cs
services.AddDatabaseDeveloperPageExceptionFilter();
```

The AddDatabaseDeveloperPageExceptionFilter provides helpful error information in the development environment.

Now - in essential - call `context.Database.EnsureCreated()`:

(more detailled steps see base document)

```cs
// create context manually
var context = services.GetRequiredService<SchoolContext>();

// create database and tables according context:
context.Database.EnsureCreated();
```

Run the applications and ensure the databases and tables with some content have been created.

## Add Controller + View

Right-click on Controller folder: Add.. New Scaffolded item...

In the Add Scaffold dialog box:

- Select MVC controller with views, using Entity Framework.
- Click Add. The Add MVC Controller with views, using Entity Framework

Run the app and checkout the "Student" menu item:

- show items in list
- add item
- view details page
- edit item
- delete item

> Note: in the original example it works as expected. But in the second example (EF in another assembly) I am getting the following issue:

```windows
---------------------------
Microsoft Visual Studio
---------------------------
Error

There was an error running the selected code generator:

'Package restore failed. Rolling back package changes for 'ContosoUniversity'.'
---------------------------
OK   
---------------------------
```

Temporary fix: I copied the scaffolded files from \Controllers and \Views from the single-assembly solution.

## Add EF Core Loging to appsettings.json

```json
"Microsoft.EntityFrameworkCore.Database.Command": "Information"
```
