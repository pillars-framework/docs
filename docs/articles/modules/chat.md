<div class="article">

# Chat Module

The _ChatModule_ is a module containing several features and functionality.
It is based on the [HogWarpChat](https://github.com/tiltedphoques/HogWarpChat).

![Chat Window](/docs/images/modules/chat/example.jpg)

## Feature Overview

1. Single *Actor* for sending messages to players ✅
2. Full control with builder pattern for complex chat messages ✅
3. Simple command registration with Attribute ✅
4. Offers different channels (House, Global, ...) ✅
5. Chat message commit history (using `up` / `down` arrow) ✅
6. Player-specific settings ✅

# Sending a Message

Sending a message to a single player is as easy as

* Injecting the @Pillars.Chat.Actors.ChatActor as a depedency
* Determinig the target player
* Calling `SendMessage` in @Pillars.Chat.Actors.ChatActor with the target and provided content of the ChatActor

```c#
[RegisterSingleton]
public class MyController(PlayerController _playerController, ChatActor _chatActor) {
	
	public async Task Something() 
	{
		// Determining the player
		var player = _playerController...
		_chatActor.SendMessage(player, "Hello from MyController!");
	}
}
```

## Building Complex Message

The chat allows for icons to be displayed and styles to be applied for texts using `<>` tags. For easier usage, the @Pillars.Core.Player.Models.ChatMessage.Builder is introduced, so you can build a complex message with many colors as easy as that:

```c#
var builder = new ChatMessage.Builder();
builder.AddIcon(ChatIcon.Gryffindor);
builder.AddText("Default ", ChatTextStyle.Default);
builder.AddText("Gryffindor ", ChatTextStyle.Gryffindor);
builder.AddText("Hufflepuff ", ChatTextStyle.Hufflepuff);
builder.AddText("Ravenclaw ", ChatTextStyle.Ravenclaw);
builder.AddText("Slytherin ", ChatTextStyle.Slytherin);
builder.AddText("Admin ", ChatTextStyle.Admin);
builder.AddText("Dev ", ChatTextStyle.Dev);
builder.AddText("Server ", ChatTextStyle.Server);
builder.AddText("Red ", ChatTextStyle.Red);
builder.AddText("Blue ", ChatTextStyle.Blue);
builder.AddText("Green ", ChatTextStyle.Green);
builder.AddText("Yellow ", ChatTextStyle.Yellow);
builder.AddText("Magenta ", ChatTextStyle.Magenta);
builder.AddText("Cyan ", ChatTextStyle.Cyan);
builder.AddSender(player.Username, ChatTextStyle.Server);
_chatActor.SendMessage(player, builder.Build().Message);
```

Resulting in the output shown in the image in the top.

# Command Registration

A chat command can be registered using the @Pillars.Chat.Attributes.SlashCommandAttribute . Optional args can be used, that are based on the player input, splitted along whitespaces `' '` **without** the command.
The first argument of the method **must** be the @Pillars.Core.Player.Models.PiPlayer .
The second argument is optional.

> [!NOTE]
> The server will not start if an invalid attribute usage was detected

```c#
[RegisterSingleton]
public class MyController(ChatActor _chatActor) {
	
	[SlashCommand("example")]
	public async Task ExampleWithoutArguments(PiPlayer player)
	{
		_chatActor.SendMessage(player, $"You called the example without arguments");
	}

	[SlashCommand("exampleargs")]
	public async Task ExampleWithArguments(PiPlayer player, string[] args)
	{
		_chatActor.SendMessage(player, $"You called the example with a total of {args.Length} arguments");
	}
}
```

# Channels

By default, _Pillars_ offers two channels:
* House
* Global

Switching channels can be done via the `/house` or `/global` command.

There will be improvements to chat in regards to that later 😊.

# Settings

Each player can specify their own settings of the chat, e.g.

* Width
* Height
* FontSize (TextScale)

These settings are saved when the settings window gets closed. These settings are saved in the database using the @Pillars.Entities.ChatSettings entity.

![Settings Window](/docs/images/modules/chat/settings.jpg)

# Improvements / TODOs

A list of improvements / todos for the chat :

* Add some more settings
	* Show background:
		* Always
		* Never
		* Only when active
	* Sound Effects On / Off
* Channels
	* Make channels dynamically (provided by server)
	* Possibility to talk to channel usinig a designated prefix
* Whisper
	* Add whisper functionality with `@username`
* Blocklist
	* Possibility to have a blocklist for whispers
	* Possibility to have a generic blocklist
	* Bind blocklist to account
* Command Auto-Complete
	* When player starts typing with `/`, offer a list of possible commands to possibly select from

</div>