# Blazor Server - Entity Framework Core MySQL Setup

## Package Installation

```bash
Install-Package Microsoft.EntityFrameworkCore.Design -Version 7.0.20
Install-Package Pomelo.EntityFrameworkCore.MySql -Version 7.0.0
```

## EF Core Tools

```bash
dotnet new tool-manifest
dotnet tool install --global dotnet-ef --version 7.0.20
```

## Migrations

```bash
dotnet ef migrations add InitialCommit
dotnet ef database update
```

## Program.cs Configuration

```csharp
builder.Services.AddDbContext<AppDbContext>(options => 
    options.UseMySql(
        builder.Configuration.GetConnectionString("DefaultConnection"), 
        new MySqlServerVersion(new Version(8, 0, 33))
    )
);
```

## appsettings.json

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Port=3307;Database=db;User=root;Password=root;"
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
