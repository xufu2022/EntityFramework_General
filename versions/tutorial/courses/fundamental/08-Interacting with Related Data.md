# Interacting with Related Data

Adding Related Data

Add New Parent and New Child Together

## Change Tracker Response to New Child of Existing Parent

- Add child to child collection of existing tracked parent. SaveChanges
- Add existing tracked parent to ref property of child. SaveChanges
- Set foreign key property in child class to parent’s key value. Add & SaveChanges

[!Note] Beware accidental inserts!
Passing a pre-existing entity into its DbSet Add will cause EF Core to try to insert it into the database!

### EF Core’s Default Entity State of Graph Data

**Add(graph)**

Has Key Value => Added
No Key Value  => Added

*Database will throw an exception (Has Key Value => Added) unless IDENTITY INSERT is explicitly enabled and the key doesn’t already exist

Some of this change tracking behavior is different in disconnected scenarios

## Eager Loading Related Data in Queries

Means to Get Related Data from the Database

Eager Loading => Include related objects in query

```csharp
    // good
    _context.Authors.Where(a=>a.LastName.StartsWith("L"))
        .Include(a=>a.Books).ToList()
    _context.Authors.Where(a => a.LastName == "Lerman")
        .Include(a => a.Books).FirstOrDefault()

    // bad
    _context.Authors.Where(a => a.LastName == "Lerman")
        .FirstOrDefault().Include(a => a.Books)       
    _context.Authors.Find(1).Include(a=>a.Books)
    _context.Authors.Include(a=>a.Books).Find(1)
```

Using Include for Multiple Layers of Relationships

```csharp
    // good
    _context.Authors.Where(a=>a.LastName.StartsWith("L"))
        .Include(a=>a.Books).ToList()
    _context.Authors.Where(a => a.LastName == "Lerman")
        .Include(a => a.Books).FirstOrDefault()

    // bad
    _context.Authors.Where(a => a.LastName == "Lerman")
        .FirstOrDefault().Include(a => a.Books)       
    _context.Authors.Find(1).Include(a=>a.Books)
    _context.Authors.Include(a=>a.Books).Find(1)
```

Using Include for Multiple Layers of Relationships
```csharp
    //Get books for each author
    //Then get the jackets for each book
    _context.Authors
        .Include(a => a.Books)
        .ThenInclude(b=>b.BookJackets)
        .ToList();

    // Get books for each author
    // Also get the contact info each author
    _context.Authors
        .Include(a => a.Books)
        .Include(a=>a.ContactInfo) 
        .ToList();

    //Get the jackets for each author’s books
    //(But don’t get the books)         
    _context.Authors
    .Include(a=>a.Books.BookJackets)
    .ToList();
```

### Performance Considerations with Include

- Composing many Includes in one query could create performance issues. Monitor your queries!
- Include defaults to a single SQL command. Use AsSplitQuery() to send multiple SQL commands instead.

### SQL Generated from Includes

Default => Single query with LEFT JOIN(s)
With **AsSplitQuery()** : Query is broken up into multiple queries sent in a single command

> Means to Get Related Data from the Database
> Query Projections: Define the shape of query results

#### EF Core Can Only Track Entities Recognized by the DbContext

Anonymous types are **not tracked**
Entities that are properties of an anonymous type are **tracked**

### Loading Related Data for Objects Already in Memory

Explicit Loading :  Explicitly request related data for objects in memory
Lazy Loading     :  On-the-fly retrieval of data related to objects in memory

> With author object already in memory, load a collection
```csharp
    _context.Entry(author).Collection(a => a.Books).Load();
```

> With book object already in memory, load a reference (e.g., parent or 1:1)
```csharp
    _context.Entry(book).Reference(b => b.Author).Load();
```

### Explicit Loading

Explicitly retrieve related data for objects already in memory

DbContext.Entry(object).Collection().Load()
DbContext.Entry(object).Reference().Load()

### More on Explicit Loading

**You can only load from a single object**
Profile to determine if LINQ query would be better performance

**Filter loaded data using the Query method**
```csharp
var happyQuotes = context.Entry(samurai) 
        .Collection(b => b.Quotes) 
        .Query()
        .Where(q => q.Quote.Contains(“Happy”)
        .ToList();
```

### Using Lazy Loading to Retrieve Related Data

Lazy Loading : On-the-fly retrieval of data related to objects in memory
Lazy Loading is OFF by default

**Enabling Lazy Loading**

- Every navigation property in every entity must be virtual
- Reference the Microsoft.EntityFramework.Proxies package
- Use the proxy logic provided by that package
    - optionsBuilder.UseLazyLoadingProxies()

> Good Behavior

One command to the db to get the books for this one author

```csharp
    foreach(var b in author.Books)
    {
        Console.WriteLine(b.Title);
    }
```
> Behavior to Avoid
- Retrieves all the Book objects from the database and materialize them and then give you the count.
- Sends N+1 commands to the database as each author’s book is loaded into a grid row
- No data is retrieved

```csharp  
    var bookCount= author.Books.Count();
    //Data bind a grid to lazy-loaded data
    //Lazy loading when no context in scope
```

### Using Related Data to Filter Objects
### Modifying Related Data

Attach starts tracking with state set to Unchanged

DbContext.Entry gives you a lot of fine-grained control over the change tracker

### Understanding Deleting Within Graphs

- Remove from an in-memory collection
- Set State to Deleted in Change Tracker
- Delete from database

Cascade Delete When Dependents Can’t Be “Orphaned”

**EF Core Enforces Cascade Deletes** : Any related data that is also tracked will be marked as Deleted along with the principal object

Cascade Delete Q/A: 
1. Will Remove() remove everything in a graph, just like Update?
    No. Remove will only remove the specific object.
1. What about deleting a dependent that’s not in a graph?
    You’ve actually done this! Just call it’s DbSet Remove method
1. How to move a child from one parent to another (when tracked)?
    1. book.AuthorId=3
    1. newAuthor.Books.Add(book)
    1. book.Author=newAuthor
1. What about optional relationships?
    1. book.AuthorId=null
    1. author.Books.Remove(book)
1. What about deleting relationships in disconnected scenarios?
    1. This is more advanced, but you will see some of it in the web app module
