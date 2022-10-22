# Logging EF Core Activity and SQL

EF Core’s logging is an extension of .NET Logging APIs

## Adding Logging to EF Core’s Workflow

DbContextOptionsBuilder.LogTo Method
```csharp
protected override void OnConfiguring (DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(someconnectionstring)
        .LogTo(target);
}
```

## Filtering Log Output with EF Core Message Categories and LogLevel

EF Core Message Categories

## .NET Logging API Configurations

> Output targets 
e.g., console, file

> Log Levels
e.g., critical, debug

## Configuring LogLevel via .NET Logging API

> LogLevel enums
- Debug (default)
- Error
- Critical
- Information
- Trace
- Warning
- None

## Additional Filtering Capabilities

- Formatting
- Detailed query error information
- Filter on event types
- What information goes in a log message
- Show sensitive information e.g., parameters

## Protecting Your Command Parameters
## Command Parameters in Your Logs

Protected by default 
Expose them with explicit code

Enabling Sensitive Data to Show in Your Logs

- Parameters hidden by default
- Parameters exposed
- .using this OptionsBuilder method

## Specifying the Target of the Log Output
## Configuring Targets via .NET Logging API

- Console
- File via Streamwriter
- Debug window
- External logger APIs

LogTo Target Examples

```csharp
    // Delegate to Console.WriteLine
    optionsBuilder.LogTo(Console.WriteLine)

    // Delegate to StreamWriter.WriteLine 
    // Be sure to dispose StreamWriter to save file
    private StreamWriter _writer = new StreamWriter(“EFCoreLog.txt", append: true); 
    optionsBuilder.LogTo(_writer.WriteLine)

    // Lambda expression for Debug.WriteLine
    optionsBuilder.LogTo(log=>Debug.WriteLine(log));
```

## Console Won’t Output Some Encodings

Default encoding won’t display special characters
Override encoding displays the special characters



