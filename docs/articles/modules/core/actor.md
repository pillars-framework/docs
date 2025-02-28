<div class="article">

# Actor / PiActor

For using *Actors* to send / receive signals to and from a client / player, an abstract wrapper @Pillars.Core.Actors.Models.PiActor`1 is introduced, that is capable of being injected as a depedency, but still remains / holds a reference to the original actor (called *WorldActor*).

The wrapper will automatically spawn its corresponding *WorldActor* instance upon instantiation / registering based on the given generic.

## Usage

If you have a generated actor from a blueprint, you simply create a [Singleton](/docs/articles/advanced/di-controller.md) that provides the blueprint actor as a generic and associate this singleton as the *ParentActor*.

> [!IMPORTANT]
> It is **highly** recommended that the actor only exposes the [PiPlayer](/docs/articles/modules/core/player.md) to the *DI*, using the @Pillars.Core.Player.Controllers.PlayerController , so there is a clear seperation between all *natives* and *pillars* models.

## Example

The following example is taken from the [Chat Module](/docs/articles/modules/chat.md) and its @Pillars.Chat.Actors.ChatActor implementation.

```cs
[RegisterSingleton]
public sealed class ChatActor : PiActor<BpHogWarpChat>
{
	private readonly ChatEvents _chatEvents;
	private readonly PlayerController _playerController;

	public ChatActor(ChatEvents ce, PlayerController pc)
	{
		_chatEvents = ce;
		_playerController = pc;
		_worldActor.ParentActor = this;
	}

	/// <summary>
	/// Sends a message to a specific player by forwarding it to the world actor's ReceiveMsg method.
	/// </summary>
	/// <param name="player">The player to whom the message is sent.</param>
	/// <param name="message">The message to send.</param>
	public async Task SendMessageToPlayer(PiPlayer player, string message) =>
		_worldActor.RecieveMsg(player.Native, message);

	/// <summary>
	/// Handles a received message from a player and forwards it to the chat events system.
	/// </summary>
	/// <param name="player">The player who sent the message.</param>
	/// <param name="message">The message sent by the player.</param>
	public void PlayerMessageReceived(NativePlayer player, string message)
	{
		if (!_playerController.Players.TryGetValue(player, out var piPlayer)) return;
		_chatEvents.ChatMessage(piPlayer, message);
	}
}
```

</div>