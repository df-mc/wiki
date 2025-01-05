This page is a short introduction on the way worlds and entities function as of
Dragonfly v0.10.

## Index
* [Worlds and transactions](#worlds-and-transactions)
* [Entities and transactions](#entities-and-transactions)
* [Handlers and transactions](#handlers-and-transactions)
* [Updating from older versions](#updating-from-older-versions)
* [Troubleshooting](#troubleshooting)
* [Conclusion](#conclusion)

## Worlds and transactions
As of v0.10, all code that modifies worlds must be executed from within a
transaction (`*world.Tx`) to ensure synchronisation with the world. Transactions
can be run as such:
```go
var w *world.World
w.Exec(func(tx *world.Tx) {
	// Use tx to edit the world, for example:
	tx.SetBlock(pos, block.Dirt{})
})
```
A `*world.Tx` is only valid inside of this transaction function. Using it outside 
of this scope (e.g. on a different goroutine) is not permitted:
```go
var w *world.World
w.Exec(func(tx *world.Tx) {
    // tx is valid here.
    go func() {
        // tx is invalid here.
    }()
})
// tx is also invalid here
```
Trying to use the `*world.Tx` in these invalid cases leads to the following panic:
```
world.Tx: use of transaction after transaction finishes is not permitted
```

`w.Exec()` returns a `<-chan struct{}` which may be used to wait for the finishing
of the transaction's execution. In cases where code needs to await a transaction,
it can do so as such:
```go
var w *world.World
<-w.Exec(func(tx *world.Tx) {})
```

## Entities and transactions
Following the introduction of transactions to worlds, entities have undergone some
significant changes as well. A new `*world.EntityHandle` was introduced which is
a persistent identifier of a `world.Entity`. Meanwhile, `world.Entity`s are now
only valid in the context of a transaction. Let us examine an example:
```go
var w *world.World

// Create a snowball entity using spawn options.
opts := world.EntitySpawnOpts{Position: pos}
handle := entity.NewSnowball(opts, nil) // handle is a *world.EntityHandle

var snowball world.Entity // Don't do this! See the explanation below.
<-w.Exec(func(tx *world.Tx) {
	snowball = tx.AddEntity(handle) // AddEntity adds the handle to the world and returns a world.Entity.
	fmt.Println(snowball.Position()) // Will equal 'pos'
})
// snowball is no longer valid here.
```
In this example, a snowball is created as a `*world.EntityHandle`. As explained before, 
these handles are persistent and always valid. The `world.Entity` returned by tx.AddEntity(),
however, is only valid while the transaction is active. This means that `snowball` in
this example is no longer valid when the transaction ends.

If we were to correctly get access to the snowball again using the handle we have
stored, we can open a new transaction using the following:
```go
var handle *world.EntityHandle

handle.ExecWorld(func(tx *world.Tx, e world.Entity) {
	tx.RemoveEntity(e)
})
```
`ExecWorld()` obtains the entity's world in a thread-safe way and opens a transaction
in it when it does. If the entity is not added to a world, `ExecWorld()` will block until
the entity is added to a world and run the transaction function once it is. If the
entity is closed before `ExecWorld()` is called, `ExecWorld()` will return false and not
run the transaction function.

In short: Avoid storing `world.Entity` implementations (includes `*player.Player`) in
any field that lasts longer than a transaction. Instead, store a `*world.EntityHandle` and
open a new transaction when needed. Another consequence is that `world.Entity` implementations,
such as `*player.Player`, should not be compared for equality with `==`. Instead, compare their
respective handles by checking, e.g., if `(world.Entity).H() == (*player.Player).H()`.

## Handlers and transactions
Handlers, such as `player.Handler` and `world.Handler`, have the respective `*player.Player`/
`*world.Tx` passed to them in an `event.Context[T]`, like so:
```go
func (h Handler) HandleMove(ctx *event.Context[*player.Player], newPos mgl64.Vec3, newRot cube.Rotation) {
	p := ctx.Val() // *player.Player
}
```
As explained in the section [Entities and transactions](#entities-and-transactions), these
values are not valid when moved to another goroutine or when stored in a field of the Handler.

## Updating from older versions
Libraries and servers using Dragonfly written before v0.10 require several changes.
Some of the biggest changes and updating suggestions are listed below.

### Transactions
Editing worlds now requires opening a transaction in advance. This means that code
such as this:
```go
var w *world.World
w.SetBlock(pos, block.Dirt{})
w.AddEntity(ent)
```
must be refactored to be run in a transaction like this:
```go
var w *world.World
w.Exec(func(tx *world.Tx) {
	tx.SetBlock(pos, block.Dirt{})
	tx.AddEntity(ent)
})
```

Most places, such as methods in `world.Block` implementations, will now have a 
`*world.Tx` passed around instead of a `*world.World` and should otherwise require
relatively few changes, other than changing `*world.World` => `*world.Tx`.

Pay additional attention in places where your code creates goroutines or calls
`time.AfterFunc()`, because these require a new transaction.

### Handlers
Storing a `*player.Player` in a field of the Handler is no longer valid, as player
entities don't last the entire lifetime. Instead, the player is now passed in all
handle functions as shown in [Handlers and transactions](#handlers-and-transactions).
Consider storing only the data necessary in your handler and, if needed, store
the `*world.EntityHandle` of the player instead of the player itself.

As a result of these transaction changes, handlers are now entirely thread safe.
This means that data stored in handlers, unless accessed from different goroutines
elsewhere, does not need to be protected using e.g. a mutex.

### Commands
Commands now have a transaction passed to them too, and players/entities returned by
a `[]Target` parameter will only include those from the same world as the caller of
the command.

### Forms
Like with commands, forms are opened within a transaction now. The transaction is
passed to all Submit methods and Close methods.

### Accepting players
Accepting players was changed from the following:
```go
for srv.Accept(func(p *player.Player) {
	// Use p.
})
```
to:
```go
for p := range srv.Accept() {
	// Use p.
}
```
Like in other places, using `p` outside of this context (different goroutine or
time.AfterFunc()) is not permitted.

## Troubleshooting
You might run into some issues trying to update to the new world transactions. Here are some of the most common issues
and what you can do to solve them:

#### panic: `world.Tx: use of transaction after transaction finishes is not permitted`:
This panic happens when a transaction is used after it is closed. Common causes for this include using entities outside
of transactions, e.g. when storing `*player.Player` as opposed to its `*world.EntityHandle`. Solving this issue involves
making sure that a `*world.Tx` is not used outside a transaction. This also includes entities that implement
`world.Entity`, such as `*player.Player`. Instead, store their `*world.EntityHandle`, which is persistent, unique and 
safe for use outside a transaction.

#### Deadlock/freeze while opening a transaction:
You might run into deadlocks trying to create world transactions. This results from opening a transaction from within
a transaction. Let's say we have the following function:
```go
func Do(p *player.Player, w *world.World) {
    p.Message("Do called")
    <-w.Exec(func(tx *world.Tx) {
        fmt.Printf("Range: %v\n", tx.Range())
    })
}
```
This function will cause a deadlock because the presence of a `*player.Player`, which implements `world.Entity`, means
that this function is called while a transaction is already opened. Because we wait for the new transaction we create to
finish (**`<-`**`w.Exec()`), a deadlock is created. There are two ways to solve this:

If, in this example, `w` is always equal to the player's world, we can simply do the following:
```go
func Do(p *player.Player, w *world.World) {
    p.Message("Do called")
    fmt.Printf("Range: %v\n", p.Tx().Range())
}
```

If, on the other hand, `w` is not guaranteed to be the player's world, we can simply remove the arrow so that we do not
wait for the transaction to end inside the current transaction:
```go
func Do(p *player.Player, w *world.World) {
    p.Message("Do called")
    w.Exec(func(tx *world.Tx) {
        fmt.Printf("Range: %v\n", tx.Range())
    })
}
```
Now, the transaction is run asynchronously once our current transaction ends.

## Conclusion
Transactions help prevent many race conditions and make it easier to optimise 
particularly hot code paths. They do make code slightly more complicated and 
you may have questions about them. Do not hesitate to ask any questions in the 
Bedrock Gophers Discord.