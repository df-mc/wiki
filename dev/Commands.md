[![cmd](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/cmd)

## Index
* [Overview](#overview)
* [Creating a command](#creating-a-command)
* [Sub commands](#sub-commands)
* [Registering commands](#registering-commands)
* [Other resources](#other-resources)

## Overview
Commands are a very important part of Minecraft and dragonfly allows developers to use this API to extend the Minecraft command system. A command requires to implement the cmd.Runnable interface where the structure can be registered with the `cmd.Register` function.

## Creating a command
Creating a command requires you to create a struct that implements the [cmd.Runnable](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/cmd#Runnable) interface.
This implementation is the functional component of the command. Its fields specify the parameters of the command and the
Run method is used to run code when a `cmd.Source` (usually a player) executes it.

Example `cmd.Runnable` implementation:

```go
package commands

import (
	"github.com/df-mc/dragonfly/server/cmd"
	"github.com/df-mc/dragonfly/server/world"
)

// ExampleCommand contains our command parameters.
type ExampleCommand struct {
	// Each parameter requires the field to be exported in order for the command to work properly.
	// The `cmd:""` struct tag may be specified to change the name of the parameter and its suffix.
	// Supported type: int*, uint*, float*, string, bool, mgl64.Vec3, []cmd.Target, cmd.Enum ...
	Number int `cmd:"number"`
	// Wrapping the parameter type in `cmd.Optional` makes the parameter optional, meaning a cmd.Source
	// does not need to supply it.
	OptionalMessage cmd.Optional[string]
}

// Run will be called when the player runs the command. In this case, we will print the number back to the player
func (c ExampleCommand) Run(source cmd.Source, output *cmd.Output, tx *world.Tx) {
	msg, ok := c.OptionalMessage.Load()
	output.Printf("%d %s (optional arg set? %v)", c.Number, msg, ok)
}
```

## Registering commands
Registering commands can be done via `cmd.Register`. Note that `cmd.New`, the function to create a command using your
`cmd.Runnable` implementation, accepts multiple `cmd.Runnable`s. We will have a look at this in [Sub commands](#sub-commands).

An example usage of `cmd.Register` for simple commands:
```go
package main

import "github.com/df-mc/dragonfly/server/cmd"

func registerCommands() {
    cmd.Register(cmd.New("example", "An example of using commands", []string{"eg"}, commands.ExampleCommand{}))
}
```

## Sub commands
As explained in [Registering commands](#registering-commands), `cmd.New` accepts multiple `cmd.Runnable` implementations.
This functionality may be used to create sub commands within your commands. Let's say we want to have a `/pet` command with
a `/pet create` and a `/pet delete` subcommand. We will start off by creating a `cmd.Runnable` for both of these sub
commands:

```go
package pet

import (
	"github.com/df-mc/dragonfly/server/cmd"
	"github.com/df-mc/dragonfly/server/world"
)

type CreateCommand struct {
	Name cmd.Varargs `cmd:"name"`
}

func (CreateCommand) Run(source cmd.Source, output *cmd.Output, tx *world.Tx) {}

type DeleteCommand struct {}

func (DeleteCommand) Run(source cmd.Source, output *cmd.Output, tx *world.Tx) {}
```

Now that we have both of these `cmd.Runnable` implementations, we will add a field at the start of these
structs to turn these into sub commands. By default, this will use the field name. A `cmd:"name"` struct tag can be
added here to make the parser use a different sub command name.

```go
package pet

import (
	"github.com/df-mc/dragonfly/server/cmd"
	"github.com/df-mc/dragonfly/server/world"
)

type CreateCommand struct {
	// This sub command will be accessible as /pet create <name>.
	Create cmd.SubCommand `cmd:"create"`
	Name cmd.Varargs `cmd:"name"`
}

func (CreateCommand) Run(source cmd.Source, output *cmd.Output, tx *world.Tx) {}

type DeleteCommand struct {
	// This subcommand will be accessible as /pet Delete.
	Delete cmd.SubCommand
}

func (DeleteCommand) Run(source cmd.Source, output *cmd.Output, tx *world.Tx) {}
```

Having created complete `cmd.Runnable` implementations for both of our subcommands, we can now run `cmd.New` and register the command:

```go
package pet

import "github.com/df-mc/dragonfly/server/cmd"

func registerCommands() {
    // We pass both cmd.Runnable implementations here. Passing nil for the aliases is valid.
    cmd.Register(cmd.New("pet", "Pet management", nil, CreateCommand{}, DeleteCommand{}))
}
```

Both subcommands will now be visible in-game, and the command parser will automatically seek out the right sub command
to execute based on the arguments supplied by the `cmd.Source` executing the command.

## Other resources
* [df-mc/plots](https://github.com/df-mc/plots/tree/master/plot/command)