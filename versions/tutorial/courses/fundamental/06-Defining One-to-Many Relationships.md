# Defining One-to-Many Relationships

> Shadow Properties 

Properties that existing the data model but not the entity class. These can be inferred by EF Core or you can explicitly define them in the DbContext

> Relationship Terminology
- Parent / Child
- Principal / Dependent

Reference from parent to child is sufficient

```csharp
    public class Author
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
        public List<Book> Books { get; set; }
    }

    // This Child has no references back to Parent
    // Foreign Key (e.g, AuthorId) will be inferred in database
    public class Book
    {
        public int Id { get; set; }
        public string Title { get; set; }
    }
```

Child has navigation prop pointing to Parent

```csharp
    public class Author
    {
        ...
        public List<Book> Books { get; set; }
    }

    // This child has a reference back to parent aka “Navigation Property”
    // It already understands the one-to-many because of the parent’s List<Child>
    // The reference back to parent aka “Navigation Property” is a bonus
    // AuthorId FK will be inferred in the database
    public class Book
    {
        ...
        public Author Author { get; set; }
    }
```

FK is recognized because of reference from parent
```csharp
    public class Author
    {
        ...
        public List<Book> Books { get; set; }
    }

    // AuthorId is recognized as foreign key for two reasons:
    //      the known relationship from parent
    //      follows FK naming convention (type + Id)
    public class Book
    {
        ...
        public int AuthorId {get; set; }
    }
```

Child has navigation prop pointing to Parent & an FK
```csharp
    public class Author
    {
        ...
        public List<Book> Books { get; set; }
    }

    // One to many is known because of List<Child>
    // Navigation and FK are bonus   
    public class Book
    {
        ...
        public Author Author { get; set; }
        public int AuthorId {get;set;}
    }
```

No detectable relationship

```csharp
    // Parent has no knowledge of children
    public class Author
    {
        public int Id { get; set; }
    }

    // This AuthorId property is just a random integer   
    public class Book
    {
        ...
        public int AuthorId {get;set;}
    }
```

You can achieve so much with mappings
Here’s a peek at an example: mapping a relationship that is completely unconventional


## Configuring a One-to-Many

When convention can’t discover it because there are no references

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
                .HasMany<Book>()
                .WithOne();
}
```

## Benefitting from Foreign Key Properties

- Default to no navigation property ==> Only add it if your biz logic requires it
- EF Core is now smarter about missing navigation data, but not in all scenarios.

> Samples
- Only a navigation property to Author.You must have an author in memory to connect a book
- With a foreign key property, you don’t need an Author object
- ...and you can even eliminate the navigation property if your logic doesn’t need it.

## Mapping Unconventional Foreign Keys

FK is tied to a relationship, so you must first describe that relationship

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Author>()
                .HasMany(a => a.Books)
                .WithOne(b => b.Author)
                .HasForeignKey(b => b.AuthorFK)
    }
```

## Understanding Nullability and Required vs. Optional Principals

By default, every dependent must have a principal, but EF Core does not enforce this

```csharp
// Specify the FK property is nullable
public int? AuthorId { get; set; }

// Map the FK property as not required
    .HasForeignKey(b => b.AuthorId)
    .IsRequired(false);

// When FK property doesn’t exist, map the inferred (“shadow”) property as not required
    .HasForeignKey("AuthorId")
    .IsRequired(false);
```


