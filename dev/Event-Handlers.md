[![Players](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/server/player#Handler)

## Index
* [Overview](#overview)
* [Using player handlers](#using-player-handlers)
* [Switching player handlers](#switching-player-handlers)
* [Using handlers to store session data](#using-handlers-to-store-session-data)
* [Other handlers](#other-handlers)
* [Multiple simultaneous handlers](#multiple-simultaneous-handlers)

## Overview
Dragonfly takes a different approach with event handling than most existing
Bedrock Edition server software. Instead of having one Listener that handles
all incoming events, Dragonfly uses Handlers that can be assigned per player,
world, inventory or other type that exposes events.

## Using player handlers
The first handler you will generally encounter is the [`player.Handler`](https://pkg.go.dev/github.com/df-mc/dragonfly/server/player#Handler).
A handler that implements this interface can be assigned in the function passed
to `srv.Accept()` in the `main.go`'s `main` function.

The first step is to create a new struct type, which we will initially
put at the bottom of `main.go`:
```go
type MyHandler struct {
	p *player.Player
}
```
`MyHandler` has a `p` field. This will later hold the player that this
handler will listen to. You can also add other fields that you want to
use for the duration of a player's session. It is generally recommended 
to have a `New...` function for your handler to initialise it, so we will 
create that:
```go
func NewMyHandler(p *player.Player) *MyHandler {
	return &MyHandler{p: p}
}
```
As you can see, we return a `*MyHandler` (pointer) as opposed to `MyHandler`.
This is so that we can edit `MyHandler`'s fields in its methods. There are
cases where this is not necessary, but for now, that is not important.

In order for `MyHandler` to implement the `player.Handler` interface, we
will embed the `player.NopHandler` type:
```go
type MyHandler struct {
	player.NopHandler
	p *player.Player
}
```
`player.NopHandler` is the default `player.Handler` assigned to a player.
This handler has no behaviour, but it does implement `player.Handler`.
Embedding it means we do not have to implement all of `player.Handler`'s
methods if we do not want to. We can now look at [`player.Handler`'s methods]((https://pkg.go.dev/github.com/df-mc/dragonfly/server/player#Handler))
and see which event we want to handle in `MyHandler`. For this example, we
will cancel a player sending a chat message and print it to the terminal:
```go
func (m *MyHandler) HandleChat(ctx *event.Context, message *string) {
	ctx.Cancel()
	fmt.Println("Player", m.p.Name(), "tried to send:", *message)
}
```
The `*event.Context` may be cancelled using `ctx.Cancel()`. This will stop
the chat message from being sent. The `message` has the type `*string`. It
is a pointer and may, therefore, be edited in this code so that the chat
message sent is changed.

Combining this code:
```go
type MyHandler struct {
	player.NopHandler
	p *player.Player
}

func NewMyHandler(p *player.Player) *MyHandler {
    return &MyHandler{p: p}
}

func (m *MyHandler) HandleChat(ctx *event.Context, message *string) {
    ctx.Cancel()
	fmt.Println("Player", m.p.Name(), "tried to send:", *message)
}
```

Now that our `player.Handler` implementation is ready, we will want to
attach it to the player when it joins. Moving back to the `main` function,
we change:
```go
for srv.Accept(nil) {
}
```
to:
```go
for srv.Accept(func(p *player.Player) {
	p.Handle(NewMyHandler(p))
}) {
}
```
Players joining the server will now immediately get `MyHandler` assigned as
their handler.

It might be useful to put your handler in a different file to organise your
code in a clean way. By creating a new package (folder) and importing it,
you can keep your handler code in a different file.

## Switching player handlers
Some servers, such as minigame servers, have very different behaviour in
one place (the games) than another (the hub). It is useful to be able to
switch `player.Handler`s at runtime to reflect this. Switching a player's
handler works the same as assigning the initial handler:
```go
type OtherHandler struct {
	player.NopHandler
}

func NewOtherHandler() *OtherHandler {
	return &OtherHandler{}
}

// ... 

p.Handle(NewOtherHandler())
```
`p.Handle(...)` can be called at any point in time to change a player's
handler, such as when the player enters a minigame or returns to the hub.

## Using handlers to store session data
Servers often need to store data for the duration of a player's session. 
Handlers can be seen as a built-in system for this: Simply by adding fields
to the handler, like so:
```go
type MyHandler struct {
	player.NopHandler
	p *player.Player
	
	deaths int
	kills int
}
```
By implementing the `HandleDeath(damage.Source)` method, you can detect
when the player quits and, for example, store the data to disk. The
handler of a player can be retrieved by calling `p.Handler()`, and asserting
it to the correct type like this:
```go
handler, ok := p.Handler().(*MyHandler)
if !ok {
	// The player's handler is not *MyHandler
	return
}
fmt.Println(handler.deaths)
```

## Other handlers
While player handlers are the most commonly used, this system of attaching
handlers is also used in other places throughout Dragonfly, such as [worlds](https://pkg.go.dev/github.com/df-mc/dragonfly/server/world#Handler)
and [inventories](https://pkg.go.dev/github.com/df-mc/dragonfly/server/item/inventory#Handler).
These systems all function the same way, with a `NopHandler` to embed and
a `Handle()` method to attach and/or switch the handler.

## Multiple simultaneous handlers
Dragonfly's system does not support adding multiple handlers simultaneously
out of the box, where other Bedrock Edition server software might. There 
are libraries that implement solutions to this however, such as [Peex](https://github.com/AndreasHGK/Peex). 
Whether you want to use these or not depends on your preference and 
requirements for your server.