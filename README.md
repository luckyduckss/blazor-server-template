# Blazor Server + EF Core 7 + MySQL

A minimal setup guide for building a Blazor Server application with Entity Framework Core 7 and MySQL (Pomelo provider).

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Project Setup](#-project-setup)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Database Setup](#-database-setup)
- [File Structure](#-file-structure)
- [Troubleshooting](#-troubleshooting)
- [Best Practices](#-best-practices)

## 📦 Prerequisites

Ensure you have the following installed:

- **.NET 7 SDK** or later - [Download](https://dotnet.microsoft.com/download)
- **MySQL Server** (v5.7 or higher) - [Download](https://www.mysql.com/)
- **Visual Studio 2022** or **Visual Studio Code** with C# extension (recommended)

## 🚀 Project Setup

### Step 1: Create New Blazor Server Project

```bash
dotnet new blazorserver -n PrepInterview1
cd PrepInterview1
```

This creates a new Blazor Server project with the default template structure.

### Step 2: Install Required NuGet Packages

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design --version 7.0.20
dotnet add package Microsoft.EntityFrameworkCore.Tools --version 7.0.20
dotnet add package Pomelo.EntityFrameworkCore.MySql --version 7.0.0
```

Or using Package Manager Console in Visual Studio:

```powershell
Install-Package Microsoft.EntityFrameworkCore.Design -Version 7.0.20
Install-Package Microsoft.EntityFrameworkCore.Tools -Version 7.0.20
Install-Package Pomelo.EntityFrameworkCore.MySql -Version 7.0.0
```

### Step 3: Install EF Core Global Tool (Optional)

For using CLI commands like `dotnet ef migrations add`:

```bash
dotnet tool install --global dotnet-ef --version 7.0.20
```

Or update if already installed:

```bash
dotnet tool update --global dotnet-ef --version 7.0.20
```

## 🛠 Installation & Configuration

### Step 4: Create Models

Create a `Models` folder in your project root, then add your entity models.

**Models/Book.cs:**

```csharp
namespace PrepInterview1.Models
{
    public class Book
    {
        public int BookId { get; set; }
        public string? Title { get; set; }
        public string? Author { get; set; }
    }
}
```

**Models/Category.cs:**

```csharp
namespace PrepInterview1.Models
{
    public class Category
    {
        public int CategoryId { get; set; }
        public string? Name { get; set; }
    }
}
```

### Step 5: Create AppDbContext

Create a new file `Data/AppDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using PrepInterview1.Models;

namespace PrepInterview1.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options)
            : base(options) { }

        public DbSet<Book> Books { get; set; }
        public DbSet<Category> Categories { get; set; }
    }
}
```

### Step 6: Configure Program.cs

Update your `Program.cs` file:

```csharp
using Microsoft.EntityFrameworkCore;
using PrepInterview1.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();

// Configure DbContext with MySQL
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(
        builder.Configuration.GetConnectionString("DefaultConnection"),
        ServerVersion.AutoDetect(builder.Configuration.GetConnectionString("DefaultConnection"))
    ));

var app = builder.Build();

// Configure the HTTP request pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

### Step 7: Setup appsettings.json

Update your `appsettings.json` with the MySQL connection string:

```json
{
    "ConnectionStrings": {
        "DefaultConnection": "Server=localhost;Port=3306;Database=PrepInterviewDb;User=root;Password=root;"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Information",
            "Microsoft.AspNetCore": "Warning"
        }
    },
    "AllowedHosts": "*"
}
```

**⚠️ Security Note:** For production, use secure configuration methods like Azure Key Vault or User Secrets:

```bash
# For development only
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your_connection_string"
```

## 🗄️ Database Setup

### Step 8: Create and Apply Migrations

Create your initial migration:

```bash
dotnet ef migrations add InitialCommit
```

Update the database:

```bash
dotnet ef database update
```

### Manual Database Creation (Alternative)

If you prefer to create the database manually first:

```sql
CREATE DATABASE PrepInterviewDb;
USE PrepInterviewDb;

CREATE TABLE Books (
    BookId INT PRIMARY KEY AUTO_INCREMENT,
    Title NVARCHAR(255),
    Author NVARCHAR(255)
);

CREATE TABLE Categories (
    CategoryId INT PRIMARY KEY AUTO_INCREMENT,
    Name NVARCHAR(255)
);
```

Then run migrations to sync with your models.

## 📁 File Structure

```
PrepInterview1/
├── Data/
│   └── AppDbContext.cs          # Database context
├── Models/
│   ├── Book.cs
│   └── Category.cs
├── Pages/
│   ├── _Host.cshtml
│   ├── Error.cshtml
│   └── Index.razor
├── Migrations/                   # Auto-generated
│   ├── 20240101000000_InitialCommit.cs
│   └── AppDbContextModelSnapshot.cs
├── Properties/
│   └── launchSettings.json
├── wwwroot/                      # Static files
├── appsettings.json
├── appsettings.Development.json
├── Program.cs                    # Entry point
├── App.razor
├── _Imports.razor
└── PrepInterview1.csproj
```

## ▶️ Running the Application

```bash
dotnet run
```

The application will start on `https://localhost:7000` by default (port may vary).

## 📝 EF Core Commands Reference

### Using CLI (dotnet-ef)

```bash
# Create migration
dotnet ef migrations add MigrationName

# Update database
dotnet ef database update

# Remove last migration
dotnet ef migrations remove

# List migrations
dotnet ef migrations list

# Script migrations (generate SQL)
dotnet ef migrations script

# Drop database
dotnet ef database drop
```

### Using Package Manager Console (Visual Studio)

```powershell
Add-Migration MigrationName
Update-Database
Remove-Migration
Get-DbContext
```

## 🔧 Troubleshooting

### Issue: "The Entity Framework tools version '7.0.x' is older than the runtime version '7.0.y'"

**Solution:** Update the global EF Core tool:

```bash
dotnet tool update --global dotnet-ef --version 7.0.20
```

### Issue: MySQL Connection Error

**Verify:**

- MySQL server is running
- Connection string is correct
- Database exists (or will be created by migrations)
- Port 3306 is accessible
- Username and password are correct

**Check connection string:**

```csharp
// Test in Program.cs
using var connection = new MySqlConnection(connectionString);
connection.Open(); // Will throw if connection fails
```

### Issue: "No database provider has been configured"

**Solution:** Ensure DbContext is registered in Program.cs:

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseMySql(...));
```

### Issue: Migration Command Not Found

**Solution:** Make sure you have the EF Core tool installed:

```bash
dotnet tool install --global dotnet-ef --version 7.0.20
```

### Issue: SqlServer Package Conflict

**Solution:** Remove any SQL Server packages if using only MySQL:

```bash
dotnet remove package Microsoft.EntityFrameworkCore.SqlServer
```

## 📚 Best Practices

### 1. **Version Consistency**

Always keep EF Core versions consistent:

```bash
# Check installed versions
dotnet package search Microsoft.EntityFrameworkCore --exact-match
```

### 2. **Connection String Management**

Never hardcode connection strings. Use `appsettings.json` or User Secrets:

```csharp
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

### 3. **Lazy Loading Prevention**

Use explicit loading or eager loading to avoid N+1 queries:

```csharp
// Eager loading
var books = await context.Books.Include(b => b.Author).ToListAsync();

// Explicit loading
await context.Entry(book).Reference(b => b.Author).LoadAsync();
```

### 4. **Async/Await**

Always use async methods in Blazor:

```csharp
var books = await context.Books.ToListAsync();
```

### 5. **Disposal**

DbContext is registered as scoped and automatically disposed. No need to dispose manually in Blazor components.

### 6. **Migration Naming**

Use descriptive migration names:

```bash
dotnet ef migrations add AddBookCategoryRelationship
dotnet ef migrations add RenameAuthorToWriterColumn
```

### 7. **Data Annotations**

Use data annotations for validation and schema configuration:

```csharp
public class Book
{
    public int BookId { get; set; }

    [Required]
    [StringLength(255)]
    public string Title { get; set; }

    [MaxLength(255)]
    public string? Author { get; set; }
}
```

## 🔒 Security Considerations

- **Never commit connection strings** with passwords to version control
- Use **User Secrets** for development:
    ```bash
    dotnet user-secrets init
    dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your_string"
    ```
- Use **environment variables** in production
- Implement **authentication** before deploying
- Use **parameterized queries** (EF Core does this by default)
- Validate and **sanitize user inputs**

## 📖 Additional Resources

- [Microsoft Docs: Blazor](https://docs.microsoft.com/en-us/aspnet/core/blazor/)
- [Entity Framework Core Documentation](https://docs.microsoft.com/en-us/ef/core/)
- [Pomelo MySQL Provider](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql)
- [MySQL Connection Strings](https://www.connectionstrings.com/mysql/)

## ✅ Quick Checklist

- [ ] .NET 7 SDK installed
- [ ] MySQL Server installed and running
- [ ] Created Blazor Server project
- [ ] Installed required NuGet packages
- [ ] Created Models
- [ ] Created AppDbContext
- [ ] Configured Program.cs
- [ ] Setup appsettings.json with connection string
- [ ] Created initial migration
- [ ] Updated database
- [ ] Verified application runs

## 📄 License

This project is open source and available under the MIT License.

## 👤 Author

Add your name or GitHub profile here.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Last Updated:** February 2026  
**Framework Version:** .NET 7 with EF Core 7  
**Database:** MySQL with Pomelo Provider
