<div class="article">

# General

The framework comes with a variety of `.template` files for your local environment and configuration.
The following page describes each file and provides some example.

## Directory.Build.props

> [!NOTE]
> Make sure to adjust the `<HogWarpServerDirectory>` in this file to the directory where `Server.Loader.exe` is located.

Template Location: `${repo}\Server\`

This files is responsible for defining the build output path and some other configurations.

Example:

```xml
<Project>
  <PropertyGroup>
    <HogWarpServerDirectory>C:\HogWarp\Server\</HogWarpServerDirectory>
  </PropertyGroup>
</Project>
```

## appsettings.json

Source Location: `${repo}\Server\`

This file defines general AppSettings, see @Pillars.Core.Configuration.Models.AppSettings

Example:

```json
{
  "AppSettings": {
    "IsDebug": true
  }
}
```

## loggersettings.json

Source Location: `${repo}\Server\Core\Logging\`

A _Serilog_ configuration file, see [Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration)


## mongosettings.json

Source Location: `${repo}\Server\Core\Database\`

This file defines required and optional DatabaseSettings, see @Pillars.Core.Database.Configuration.DatabaseSettings

Example **without** credentials:

```json
{
  "DatabaseSettings": {
    "Database": "hogwarp",
    "Host": "localhost",
    "Port": 27017,
    "DebugLog": true
  }
}
```

Example **with** credentials:

```json
{
  "DatabaseSettings": {
    "Database": "hogwarp",
    "Host": "localhost",
    "Port": 27017,
    "DebugLog": true,
    "Credentials": {
      "User": "dbUser",
      "Password": "dbPassword"
    }
  }
}
```

</div>