[![Forms](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/player/scoreboard?tab=doc)

## Index
* [Overview](#overview)
* [Examples](#examples)

## Overview
To create a new scoreboard, you can use the `scoreboard.New()` method, with the argument(s) passed used as the
scoreboard name/title. Scoreboards implement `io.Writer` and `io.StringWriter` to write text to the scoreboard,
along with `(*Scoreboard).Set()` and `(*Scoreboard).Remove()` methods to update a line with a specific index.

By default, scoreboards are padded to look more appealing visually. This can be undesired behaviour in some cases
however, so Dragonfly allows this padding to be stripped fairly easily, using the `(*Scoreboard).RemovePadding()`
method.

Scoreboards in Dragonfly may be sent using the `(*Player).SendScoreboard()` method, which will send a scoreboard
to a player. These scoreboards can be removed similarly using the `(*Player).RemoveScoreboard()` method.

## Examples

```go
package main

import (
	"github.com/df-mc/dragonfly/server/player/scoreboard"
	"github.com/df-mc/dragonfly/server/player"
)

func sendScoreboard(p *player.Player) {
	s := scoreboard.New("Title")
	s.Set(0, "Hello,")
	s.Set(1, "World!")
	p.SendScoreboard(s)
}
```