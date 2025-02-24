<div class="article">

# Database

As of this writing, _Pillars_ only supports _MongoDB_ database, using the data access library [MongoDB.Entities](https://mongodb-entities.com/). The goal is to provide a **simple**, **frictionless** and **easy-to-use** experience, which excludes _Schema Migrations_ and the need to maintain them.

Please refer to the _MongoDB.Entities_ [documentation](https://mongodb-entities.com/wiki/Get-Started.html) to get a better understanding.

## Entities

Pillars comes with several _Entities_ already included. An _Entity_ defines the structure of a database collection, see [here](https://mongodb-entities.com/wiki/Entities.html).

> [!WARNING]
> For easier reference and extending of entities, it is recommended to have a common namespace `Pillars.Entities` across all entities.

Example:

```c#
namespace Pillars.Entities;

[Collection("accounts")]
public sealed partial class Account : Entity, ICreatedOn, IModifiedOn
{
	public ulong DiscordId { get; set; }

	public DateTime CreatedOn { get; set; }

	public DateTime ModifiedOn { get; set; }
}
```

### Extending / Embedding Entities 

To add additional fields to an existing _Entity_, a partial class is preferred.

The following examples adds a `LastLogin` field to the @Pillars.Entities.Account , using the `IgnoreDefault` mechanism, so `null` values don't get saved. The implementation is done in the corresponding [controller](di-controller.md).

> [!TIP]
> When extending / embedding entities, please also refer to [Entity Relationships](https://mongodb-entities.com/wiki/Relationships-Embeded.html).

```c#
namespace Pillars.Entities;

public sealed partial class Account
{
	[IgnoreDefault]
	public DateTime? LastLogin { get; set; }
}
```

</div>