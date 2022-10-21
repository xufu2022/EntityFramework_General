# Using EF Core 6 to Query a Database

## Query Workflow

example:

```csharp
    context.Authors.ToList() 
```
Express and execute query flow

1. Express and execute query 
1. EF Core reads model, works with provider to work out SQL 
1. Sends SQL to database 
1. Receives tabular results 
1. Materializes results as objects
1. Adds tracking details to DbContext instance

## Two Ways to Express LINQ Queries

| **LINQ Methods**   | **LINQ Operators** |
|:-----------------------------------|------------|
| context.Authors.ToList();                               | (from a in context.Authors select a).ToList() |
| context.Authors.Where(a=>a.FirstName==“Julie”).ToList() | (from a in context.Authors where a.FirstName==“Julie” select a).ToList() |

Queries are Composable

## Triggering Queries via Enumeration

```csharp
    var query=_context.Authors;
    foreach (var a in query)
    {
        Console.Writeline(a.LastName);
    }
```


Database connection remains open during query enumeration

## Enumerate vs. Execute

```csharp
    foreach (var a in context.Authors){
        Console.WriteLine(a.FirstName);
    }

    foreach (var a in context.Authors){
        RunSomeValidator(a. FirstName);
        CallSomeService(a.Id);
        GetSomeMoreDataBasedOn(a.Id);
    }

    var authors=context.Authors.ToList()
    foreach (var a in authors){
        RunSomeValidator(a.FirstName);
        CallSomeService(a.Id);
        GetSomeMoreDataBasedOn(a.Id);
    }
```
 ## Filtering Queries Securely by Default

Parameterized queries protect your database from SQL Injection attacks

## Benefiting From Additional Filtering Features

> Filtering Partial Text in Queries

SQL LIKE(%abc%)

| **EF.Functions.Like**   | **LINQ Contains** |
|:-----------------------------------|------------|
|EF.Functions.Like(property, %abc%)     | property.Contains(abc) |
|_context.Authors.Where(a=>EF.Functions.Like(a.Name,“%abc%”) ) |_context.Authors.Where(a=>a.Name.Contains(“abc”)) |

## Finding an Entity Using its Key Value

- DbSet.Find(keyvalue)
- This is the only task that Find can be used for
- Not a LINQ method
- Executes immediately
- If key is found in change tracker, avoids unneeded database query

## No Track Queries and DbContexts

AsNoTracking() returns a query, not a DbSet

```csharp
    optionsBuilder
    .UseSqlServer(myconnectionString)
    .UseQueryTrackingBehavior
    QueryTrackingBehavior.NoTracking);
```
All queries for this DbContext will default to no tracking
Use DbSet.AsTracking() for special queries that need to be tracked

