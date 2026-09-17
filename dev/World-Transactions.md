Dragonfly serialises access to each world through transactions. This page
explains how to schedule world and entity work as of Dragonfly v0.11.

## Index

* [World owners and transactions](#world-owners-and-transactions)
* [Scheduling world work](#scheduling-world-work)
* [Entities and stable references](#entities-and-stable-references)
* [Deferred work](#deferred-work)
* [Handlers and commands](#handlers-and-commands)
* [Tasks and failures](#tasks-and-failures)
* [Blocking calls with results](#blocking-calls-with-results)
* [Troubleshooting](#troubleshooting)

## World owners and transactions

Each world has an owner: the single goroutine that runs all transactions for
that world, including ticks. Code that receives a `*world.Tx` is already running
on the owner and may use the transaction directly:

```go
func update(tx *world.Tx, pos cube.Pos) {
	tx.SetBlock(pos, block.Dirt{}, nil)
}
```

A `*world.Tx`, `world.Entity` or `*player.Player` is only valid inside the
callback that supplied it. Do not store these values or capture them in a
goroutine:

```go
w.Do(func(tx *world.Tx) {
	// tx is valid here.
	go func() {
		// tx is not valid here.
	}()
})
// tx is not valid here either.
```

Use a world, entity handle or typed entity reference to schedule another
callback instead.

## Scheduling world work

From any goroutine, use `World.Do` to schedule fire-and-forget work:

```go
var w *world.World

task := w.Do(func(tx *world.Tx) {
	tx.SetBlock(pos, block.Dirt{}, nil)
})
```

`Do` returns immediately and is safe to call from anywhere, including another
owner callback. Use `World.DoAfter` when the work should not run until a delay
has passed:

```go
w.DoAfter(time.Second, func(tx *world.Tx) {
	tx.SetBlock(pos, block.Air{}, nil)
})
```

Both methods return a `*world.Task`. Ignoring the task is fine when the caller
does not need to observe completion or failure.

`World.Exec`, which was used before v0.11, no longer exists. Replace
fire-and-forget uses with `Do`; use the off-owner `world.Call` helper when a
result is required.

## Entities and stable references

An entity handle is a stable identity that may be stored outside a transaction.
Schedule work through the handle to access the live entity safely:

```go
var handle *world.EntityHandle

handle.Do(func(tx *world.Tx, e world.Entity) {
	tx.RemoveEntity(e)
})
```

Entity work follows the entity between worlds. If a player travels through a
portal before delayed work runs, for example, the callback runs on the owner of
the player's new world:

```go
handle.DoAfter(time.Second, func(tx *world.Tx, e world.Entity) {
	// e is valid only inside this callback.
})
```

Typed references avoid type assertions. Use `world.EntityRef[T]` for any entity
type and `player.Ref` for players:

```go
type Arena struct {
	players []player.Ref // not []*player.Player
}

for _, ref := range arena.players {
	ref.Do(func(tx *world.Tx, p *player.Player) {
		p.Message("Round over")
	})
}
```

Create refs with `world.NewEntityRef[T](handle)` or `player.NewRef(handle)`. A
one-off player operation can use `player.Do` directly:

```go
player.Do(handle, func(tx *world.Tx, p *player.Player) {
	p.Message("Hello")
})
```

Compare entities using their stable handles, not by comparing short-lived
`world.Entity` or `*player.Player` values.

## Deferred work

When code already has a transaction and needs work to run immediately after the
current callback, use `Tx.Defer`:

```go
tx.Defer(func(next *world.Tx) {
	// next is a fresh transaction.
})
```

Deferred callbacks run on the same owner, ahead of the world's normal queue,
and in registration order (FIFO). Never capture the current transaction in the
deferred callback; use the fresh transaction passed to it.

This is useful for mutating entities after iterating `tx.Entities()` or acting
after an event's default behaviour has applied. Use `World.Do` instead when the
work should go through the normal queue.

| Situation | Use |
|---|---|
| Immediately after this callback, in the same world | `tx.Defer` |
| Soon, from anywhere | `w.Do` |
| After a delay | `DoAfter` |
| Wherever an entity is when the work runs | `handle.Do` |

Player event contexts also have `Context.Defer`, which re-resolves the player
for the deferred callback:

```go
ctx.Defer(func(next *player.Context) {
	next.Player().Message("Done")
})
```

## Handlers and commands

Handlers and commands already run on a world owner, so use the transaction or
event context they receive rather than scheduling and waiting for more work.
Player handlers receive a `*player.Context`; call `ctx.Player()` to get the
player, and use embedded world operations directly:

```go
func (h Handler) HandleBlockBreak(ctx *player.Context, pos cube.Pos, ...) {
	ctx.Player().Message("Broken")
	ctx.SetBlock(pos.Side(cube.FaceUp), block.Air{}, nil)
}
```

Cancellable `world.Handler` events receive a `*world.Context`, which embeds the
transaction. Non-cancellable `HandleEntitySpawn`, `HandleEntityDespawn` and
`HandleClose` events receive a `*world.Tx`.

For commands, `Runnable.Run` receives a nil transaction when the source is not
attached to a world, such as a console source. Nil-check the transaction before
using it.

`Server.Accept` and `Server.Players` also run each loop body on the player's
world owner. Keep loop bodies short and do not call `world.Call*` or wait for a
task from inside them. To act on players later, collect their handles:

```go
var handles []*world.EntityHandle
for p := range srv.Players(nil) {
	handles = append(handles, p.H())
}
for _, handle := range handles {
	player.Do(handle, func(tx *world.Tx, p *player.Player) {
		p.Message("Hello")
	})
}
```

## Tasks and failures

`Do`, `DoAfter` and `Defer` return a `*world.Task`. A task records whether its
callback ran successfully or failed because the target closed, the task was
cancelled or the callback panicked:

```go
task := handle.DoAfter(time.Second, func(tx *world.Tx, e world.Entity) {
	// ...
})
task.OnDone(func(err error) {
	switch {
	case err == nil:
		// The callback completed.
	case errors.Is(err, world.ErrEntityClosed):
		// The entity closed before the callback ran.
	case errors.Is(err, world.ErrWorldClosed):
		// The world closed before the callback ran.
	case errors.Is(err, world.ErrTaskPanicked):
		// The callback panicked.
	}
})
```

`OnDone` always invokes its hook on a fresh goroutine. The hook is therefore
off-owner and must schedule more work before touching world state.

`Task.Wait`, `Task.Done` and `Task.Err` are intended for off-owner code such as
tests and shutdown paths. `Task.Cancel` prevents a pending task from starting.
Calling `Wait` from the target owner blocks that owner on itself and deadlocks.

Panics in fire-and-forget callbacks are recovered, logged with their stack and
stored as a `*world.PanicError` on the task. A task may also report
`ErrEntityType`, `ErrEntityNotInWorld` or `ErrTaskCancelled`.

## Blocking calls with results

Off-owner code that needs a result may use `world.Call`:

```go
count, err := world.Call(ctx, w, func(tx *world.Tx) (int, error) {
	return len(slices.Collect(tx.Entities())), nil
})
```

The equivalent helpers are `world.CallEntity` for an entity handle,
`world.CallRef` for a typed entity reference and `player.Call` for a player.

These functions are only for code outside the target owner, such as background
goroutines, startup code and tests. Never call them from a handler, command,
scheduled callback or other code that already has a transaction: waiting there
deadlocks the owner. Use the existing transaction directly, or schedule
follow-up work with `Defer` or `Do`.

## Troubleshooting

### A callback never finishes

Check whether owner code calls `world.Call*` or `Task.Wait`. The owner cannot
process the scheduled task while it is blocked waiting for that same task.
Use the current transaction directly, `tx.Defer` for immediate follow-up work
or `w.Do` for normal queued work.

### A task reports a closed-world or closed-entity error

Scheduled work has defined lifetime failures. Handle `world.ErrWorldClosed` and
`world.ErrEntityClosed` when the operation must be retried or reported. A
player-context defer can also return `world.ErrEntityNotInWorld` if the player
moved to another world; use `player.Do` when the operation should follow the
player.

### A typed reference reports `world.ErrEntityType`

The handle still exists, but its live entity no longer has the type expected by
the reference. Treat the reference as stale and stop scheduling through it.

### Work runs in an unexpected order

Use `tx.Defer` for strict follow-up work on the same owner. It runs ahead of the
normal queue in FIFO order. `w.Do` uses the normal queue and is the appropriate
choice when other already-queued work may run first.

Transactions prevent concurrent access to world state while making scheduling
explicit. For a concise list of changes from v0.10, see the
[v0.11.0 migration guide](v0.11.0-Migration-Guide).
