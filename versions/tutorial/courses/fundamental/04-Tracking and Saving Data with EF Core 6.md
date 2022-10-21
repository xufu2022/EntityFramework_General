# Tracking and Saving Data with EF Core 6

> DbContext : Represents a session with the database
- Contains EntityEntry objects with reference pointers to in-memory objects
> DbContext.ChangeTracker : Manages a collection of EntityEntry objects
> EntityEntry : State info for each entity: 
- CurrentValues, 
- OriginalValues, 
- State enum, 
- Entity & more
> Entities : In-memory objects with key (identity) properties that the DbContext is aware of

## Understanding Tracking and Saving Workflow

> Tracking and Saving Workflow

1. EF Core creates tracking object for each entity
   - Tracking States:
     - Unchanged (Default)
     - Added
     - Modified
     - Deleted
2. DbContext maintains EntityEntries
3. On SaveChanges
   - DbContext updates State for tracked entities
   - Works with provider to compose SQL statements
   - Executes statements on database
4. Returns # Affected & new PKs
5. Updates entity PKs and FKS
6. Resets state info

## Add from DbSet or DbContext

```csharp

    //DbContext will discover type Knows it’s Added
    context.Authors.Add(…)

    //Track via DbSet
    //DbSet knows the type => DbContext knows that it’s Added
    context.Add(…)
```

## How DbContext Discovers About Changes

> DbContext.ChangeTracker.DetectChanges()

Reads each object being tracked and updates state details in EntityEntry object

> DbContext.SaveChanges()

Always calls DetectChanges for you

> You can call DetectChanges() in your code if needed

The entities are just plain objects and don’t communicate their changes to the DbContext

## Various Paths for Tracking to Detect Edits

- If DbContext is aware of object, it will detect changes on SaveChanges
- DbSet and DbContext also have an Update() method
- More advanced: modify state directly

## Begin Tracking and Set State to Modified

context.Authors.Update(…)

- Track via DbSet
- DbSet knows the type
- Knows right away that it’s Modified

context.Update(…)

- Track via DbContext
- DbContext will discover the type
- Knows right away that it’s Modified
  
## More Pointers for Disconnected Data

- Update is not typically needed when object is tracked
- Update and Add and Remove are important for untracked objects
- Your app logic will need to know when to call Update, Add or Remove

## Deleting May Seem a Little Weird

1. Context needs to track the entity
1. Then you Delete (“Remove”) the entity
1. EF Core sends relevant SQL to database on SaveChanges

## Tracking Multiple Objects and Bulk Support

> AddRange Method
    _context.Authors.AddRange(author1,author2,author3,author4, etc.)

> Tracking Multiple Entities
- context.Authors.AddRange(…)
- context.Authors.UpdateRange(…)
- context.Authors.RemoveRange(…)
- context.AddRange(…)
- context.UpdateRange(…)
- context.RemoveRange(…)
  
## Combined EF Core + SQL Server Effort

1. Individual Commands <=3
    Faster to send individual commands for 1-3 entities
1. Batched Commands 4+
    Faster to send batched commands for 4+ entities

## Batching can combine types and operations

Default Batch Constraints of SQL Server Provider

