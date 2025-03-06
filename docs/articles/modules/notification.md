<div class="article">

# Notification Module

The _NotificationModule_ is a module that provides the feature for triggering notifications.

## Feature Overview

1. Single *Actor* for triggering notifications to players ✅
2. Over 20 different icons by default

# Trigger a Notification

Triggering a notification to a target or set of targets is as easy as

* Injecting the @Pillars.Notifications.Actors.NotificationActor as a depedency
* Determinig the target player
* Calling `Notify` in @Pillars.Notifications.Actors.NotificationActor with the target, icon, title and message

```c#
[RegisterSingleton]
public class MyController(PlayerController _playerController, NotificationActor _notifyActor) {
	
	public async Task Something() 
	{
		// Determining the player
		var player = _playerController...
		_notifyActor.Notify(player, NOTIFICATIONICON.UI_T_COLLECTIONCOMPLETEICON, "Success!", "Here is your first notification");
	}
}
```

# Icons

The icons are maintained in the client mod files in an array. The selected icon is determined by the given icon index of that array.
@Pillars.Notifications.Data.NOTIFICATIONICON provides an easy way to differentiate between the icons. The name of the enum members resemble the _"UI Texture"_ as in the unreal editor / creator kit.

> [!IMPORTANT]
> When adding new icons, make sure to add them at the end to ensure backwards compatibility.

## Overview

> [!NOTE]
> The list may become outdated as more and more icons are added. Always check with the latest source.

![Icon Overview](/docs/images/modules/notification/iconoverview.jpg)

# Improvements / TODOs

A list of improvements / todos for the notifications :

* Add possibility to determine position:
	* Top Left, stacking top to bottom
	* Center Left, stacking bottom to top
	* Bottom Left, stacking bottom to top
	* Same for right side
* Extend the title / message style to include colors (font extensions)
* Dynamically load asset icon by string (failed on 1st try - maybe not possible)

</div>