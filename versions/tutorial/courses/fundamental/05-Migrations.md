# Controlling Database Creation and Schema Changes with Migrations

> Migration Commands Lead to Migrations APIs

PowerShell migrations commands => add-migration
dotnet CLI migrations commands => dotnet ef migrations add


> Migration Commands Lead to Migrations APIs

PowerShell migrations commands => Microsoft.EntityFrameworkCore.Tools
dotnet CLI migrations commands => dotnet tool install dotnet-ef

> Tools depends on Design => Microsoft.EntityFrameworkCore.Design
> Design depends on Relational => Microsoft.EntityFrameworkCore.Relational

## Migrations Recommendation

Development database => update-database
Production database  => script-migration

## Seeding a Database via Migrations

modelBuilder.Entity<EntityType>().HasData(parameters)
modelBuilder.Entity<Author>().HasData(new Author {Id=1, FirstName=“Julie”, .. };

## Specify Seed Data with ModelBuilder HasData Method

- Provide all non-nullable parameters including keys and foreign keys
- HasData will get translated into migrations
- Inserts will get interpreted into SQL
- Data will get inserted when migrations are executed 
  
## Add-Migration Tasks for Model Changes

1. Read DbContext to determine data model
1. Compare data model to snapshot
1. Generate migration file to apply the deltas
1. Create updated snapshot file

Seeding with HasData will not cover all use cases for seeding

Use Cases for Seeding with HasData

- Mostly static seed data 
- Sole means of seeding
- No dependency on anything else in the database
- Provide test data with a consistent starting point


HasData will also be recognized and applied by **EnsureCreated**

> Some Scripting Options

| **Scripting Options**   | **Description** |
|:-----------------------------------|------------|
| script-migration            |    Default: Scripts every migration |
| script-migration-idempotent |    Scripts all migrations but checks for each object first e.g, table already exists |
| script-migration FROM TO |       Specifies last migration run, so start at the next one TO Arg: final one to apply |

> Reverse Engineer with the Scaffold Command
- PowerShell migrations commands => Scaffold-DbContext
- dotnet CLI migrations commands => dotnet ef dbcontext scaffold
