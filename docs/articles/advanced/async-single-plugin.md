<div class="article">

# Async / Single Plugin

Within *Pillars* we try to provide an asynchronous operation, wherever possible, but you may ask yourself *"Why use a single plugin and make everything async?"* . That's a valid question, so in this article we will look at some behavior of *HogWarp* and provide some examples, why some decisions were made.

Let's say we have **two** plugins - *PluginA* and *PluginB*:

* Both subscribe to the same native *PlayerSystem.PlayerJoinEvent*.
* Both have an expensive / long running process

Have a look at the output of the console.

## Everything Sync

# [PluginA](#tab/plugina-sync)

```cs
public class Plugin: IPlugin
{
	private readonly Logger _logger = new("PluginA");
	public string Author { get; } = "PluginA";
	public string Name { get; } = "PluginA";
	public Version Version { get; } = new Version(1, 0, 0);

	public void PostLoad()
	{
		Server.PlayerSystem.PlayerJoinEvent += PlayerJoined;
	}

	private void PlayerJoined(Player id)
	{
		_logger.Info($"PluginA joined {id}, waiting...");
		Task.Delay(2_000).Wait(); // Some expensive stuff
		_logger.Info($"PluginA waited");
	}	
}
```

# [PluginB](#tab/pluginb-sync)

```cs
public class Plugin: IPlugin
{
	private readonly Logger _logger = new("PluginB");
	public string Author { get; } = "PluginB";
	public string Name { get; } = "PluginB";
	public Version Version { get; } = new Version(1, 0, 0);

	public void PostLoad()
	{
		Server.PlayerSystem.PlayerJoinEvent += PlayerJoined;
	}

	private void PlayerJoined(Player id)
	{
		_logger.Info($"PluginB joined {id}, waiting...");
		Task.Delay(2_000).Wait(); // Some expensive stuff
		_logger.Info($"PluginB waited");
	}	
}
```

# [Output](#tab/output-sync)

![Console Output](/docs/images/advanced/sync-output.jpg)

---

As you can see in the output, *PluginB* will wait for the completion of *PluginA* (yes even skipping the process entirely in case of an exception), even though *PluginB* has no idea of the existance of *PluginA* and that it's own execution may be delayed.

> [!TIP]
> That's why we recommend a *single-plugin* solution, where no outside plugins interfere with the execution of *Pillars*.

So, let's try to make it Async!

## Async All The Things

In the following example, we wrap the entire execution in an own `Task`, makeing it _async_ (at least to some extend.)

# [PluginA](#tab/plugina-async)

```cs
public class Plugin: IPlugin
{
	private readonly Logger _logger = new("PluginA");
	public string Author { get; } = "PluginA";
	public string Name { get; } = "PluginA";
	public Version Version { get; } = new Version(1, 0, 0);

	public void PostLoad()
	{
		Server.PlayerSystem.PlayerJoinEvent += (player) =>
		{
			Task.Run(async () => await PlayerJoined(player));
		};
	}

	private async Task PlayerJoined(Player id)
	{
		_logger.Info($"PluginA joined {id}, waiting...");
		await Task.Delay(2_000); // Some expensive stuff
		_logger.Info($"PluginA waited");
	}	
}
```

# [PluginB](#tab/pluginb-async)

```cs
public class Plugin: IPlugin
{
	private readonly Logger _logger = new("PluginB");
	public string Author { get; } = "PluginB";
	public string Name { get; } = "PluginB";
	public Version Version { get; } = new Version(1, 0, 0);

	public void PostLoad()
	{
		Server.PlayerSystem.PlayerJoinEvent += (player) =>
		{
			Task.Run(async () => await PlayerJoined(player));
		};
	}

	private async Task PlayerJoined(Player id)
	{
		_logger.Info($"PluginB joined {id}, waiting...");
		await Task.Delay(2_000); // Some expensive stuff
		_logger.Info($"PluginB waited");
	}	
}
```

# [Output](#tab/output-async)

![Console Output](/docs/images/advanced/async-output.jpg)

---

As you can see in the output, both plugins start their own execution, independent from one another 👍. Now this is a very valid approach, but depends on that **all** plugins implement this behavior.

## Solution

By design, the @Pillars.Bootstrapper and host is running async, managing it's own threadpool:

```cs
// Runs the host asynchronously and continues with exiting the environment with a code of -1
return _host.RunAsync().ContinueWith(_ => Environment.Exit(-1));
```

Now we can subscribe to native events with the simple `async` keyword:
```cs
HogWarpSdk.Server.PlayerSystem.PlayerJoinEvent += async p => PlayerJoined(p);
```

And together with *event delegates* instead of actions, e.g. in the @Pillars.Core.Player.Events.PlayerConnectionEvents , you get somewhat true async behavior:

```cs
public TestController(PlayerConnectionEvents pce)
{
	pce.OnPlayerConnected += OnPlayerConnect;
}

private async Task OnPlayerConnect(PiPlayer player)
{
	_logger.Debug("Player {pid} connected - Connection Id: {cid}", player.Id,
		player.ConnectionId);
	await Task.Delay(5_000);
	_logger.Debug("Sending message");
}
```

> [!IMPORTANT]
> It is **highly** recommended to avoid using the native events and use the corresponding, correctly wrapped events from *Pillars*

</div>