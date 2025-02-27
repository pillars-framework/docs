<div class="article">

# Schedules Module

The *ScheduleModule* is a server-side only module, that provides the feature to run methods on a time-based manner, either for a specific point in time (*Schedule*), or in a certain time interval (*Cycle*) using attributes.

## Feature Overview

1. Declare schedule type using attributes ✅
2. Track history of either type ✅
3. Perform _"catch up"_ on server start for missed runs ✅

## History / Catchup

Schedules and cycles offer a form of catching up missed triggers (e.g. while the server was offline). To do this, it is necessary to track / log certain executions in the database. To not clutter the database, several levels of @Pillars.Schedules.Data.CATCHUP mechanisms are provided

```cs
/// <summary>
/// The enum describes the behavior what should happen if a schedule or cycle
/// was missed, while the server was offline.
/// </summary>
public enum CATCHUP
{
	/// <summary>
	/// No History will be added for this schedule
	/// </summary>
	NONE = 0,

	/// <summary>
	/// Does nothing when server starts, but keeps a history in the database
	/// </summary>
	NEVER = 1,

	/// <summary>
	/// If any number of cycles or schedules were missed, the method will be run ONCE as a catchup
	/// </summary>
	ONCE = 2,

	/// <summary>
	/// Performs the schedule / cycle for exactly the amounts of missed cycles or schedules
	/// </summary>
	ALL = 3
}
```

# Schedule

A @Pillars.Schedules.Models.Schedule describes the **exact** point in time a method should be executed / run, using the @Pillars.Schedules.Models.ScheduleAttribute . It uses the [Cronos](https://github.com/HangfireIO/Cronos) package to evaluate a given cron-expression.

Each Schedule **must** return a `bool` value, indicating if the execution was correctly performed (`true`) or if an error occured (`false`).

> [!TIP]
> As a general advice, even though everything is async and multi-threaded, it is **not** recommended to have a lot of schedules being triggered at the same time.


## Registration

Registering a new schedule is as easy as annotating a certain function with the @Pillars.Schedules.Data.SCHEDULE attribute.

```cs
[Schedule(SCHEDULE.TEST, "0 16 * * *", CATCHUP.ALL)]
public async Task<bool> TestSchedule()
{
	_logger.Information("Testing schedule triggered!");
	return true;
}
```

Let's have a look at that.

### Schedule Id

The `SCHEDULE` enum is used for identification, logging and tracking purposes in the database, therefore unique.

> [!NOTE]
> The server will not start if a schedule id is present multiple times.

```cs
public enum SCHEDULE
{
	TEST = 1
}
```

### Cron Format

The cron format in the attribute describes **when** the schedule should be executed, meaning `"0 16 * * *"` determines the execution of the function everyday at `16:00`

> [!NOTE]
> The server will not start if a given cron expression is invalid.

```         
┌───────────── minute                   
│ ┌───────────── hour      
│ │ ┌───────────── day of month
│ │ │ ┌───────────── month 
│ │ │ │ ┌───────────── day of week
│ │ │ │ │
* * * * *
```

> [!IMPORTANT]
> The `seconds` are omitted from the cron expressions

### History

Based on the given catchup `ALL`, the server will check on startup, how often it missed the `16:00` time since the server shutdown.

If a history is tracked, it will be saved as a @Pillars.Schedules.Models.ScheduleHistory in the database.

> [!NOTE]
> If a catchup trigger fails during server start (returning false), the server will not continue to start.

# Cycle

A @Pillars.Schedules.Models.Cycle describes a loose interval with capabilities of history and catchup. All cycles share a single timer to reduce memory footprint. Due to this nature there are some technical limitations.

> [!NOTE]
> On server start, if a new cycle is detected, it will start tracking it. If the exact time is required, use a **Schedule**

## Registration

Registering a new cycle is as easy as annotating a certain function with the @Pillars.Schedules.Data.CYCLE attribute.

```cs
[Cycle(CYCLE.TEST, 5, CATCHUP.ALL)]
public async Task<bool> TestCycle()
{
	_logger.Information("Testing cycle triggered!");
	return true;
}
```

### Cycle Id

The `CYCLE` enum is used for identification, logging and tracking purposes in the database, therefore unique.

> [!NOTE]
> The server will not start if a cycle id is present multiple times.

```cs
public enum CYCLE
{
	TEST = 1
}
```

### CycleTimer

The cycle timer describes the time in **minutes** between each trigger. Due to the nature of the cycle, the shortest considerable timespan is `2` minutes.

### History

See history section above.

# Questions ?

Have any questions or remarks? Feel free to reach out 🙂.

</div>