<div class="article">

# Logger

*Pillars* uses [Serilog](https://github.com/serilog/serilog) to provide you with detailed information in the console.

## Configuration

Via configuration you can pipe the output to files, databases, w/e you like. The logger configuration can be changed in the [loggersettings.json](../../guides/configuration.md#loggersettingsjson).

## Usage

Using the logger requires some additional work, to correctly bind the context, e.g. the controller using the logger. After that you can use the logger to print with different log levels:

* `Information` - Prints out information log
* `Debug` - Prints out a debug message, only applicable if the *Debug Mode* is enabled
* `Error` - Prints an error message, accepts `Exception` as parameters
* `Warning` - Prints out a warning message
* and more ...

```cs
public sealed class TestController(ILogger l)
{
	private readonly ILogger _logger = l.ForThisContext();
}
```


</div>