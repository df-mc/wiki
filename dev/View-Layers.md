[![World](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/server/world#ViewLayer)

## Index
* [Overview](#overview)
* [Accessing a view layer](#accessing-a-view-layer)
* [Name tags](#name-tags)
* [Score tags](#score-tags)
* [Visibility](#visibility)
* [Removing overrides](#removing-overrides)
* [Lifecycle](#lifecycle)
* [Examples](#examples)

## Overview
A `world.ViewLayer` holds per-viewer overrides for how a single session sees other entities.
Each session owns one view layer, and Dragonfly merges its overrides into the entity metadata
the session receives. View layers do not change any state on the entity itself — they only
change what is sent to one viewer.

Each view layer supports three kinds of overrides:
* Name tag
* Score tag
* Visibility (forced visible or forced invisible)

## Accessing a view layer
Every player has a view layer attached to its session. The most convenient way to set overrides
is through the helpers on `*player.Player`, which forward to the player's view layer:
```go
p.ViewNameTag(target, text.Colourf("<yellow>[Admin]</yellow> %s", target.Name()))
p.ViewScoreTag(target, text.Colourf("<green>100 HP</green>"))
p.ViewVisibility(target, world.EnforceInvisible())
```
The underlying `*world.ViewLayer` is also available if you want to read the current overrides
or pass it around:
```go
layer := p.ViewLayer()
```

## Name tags
`ViewNameTag` overrides what name tag a viewer sees above an entity. Passing an empty string
hides the name tag for the viewer. The override also toggles the `AlwaysShowName` and `ShowName`
flags so the tag renders (or stops rendering) for that viewer:
```go
p.ViewNameTag(target, text.Colourf("<aqua>Custom Name</aqua>"))
p.ViewNameTag(target, "") // hides the name tag for this viewer
```
To return to the public name tag, call `ViewPublicNameTag`:
```go
p.ViewPublicNameTag(target)
```
The current override, if any, can be read back from the view layer:
```go
if name, ok := p.ViewLayer().NameTag(target); ok {
    // name is the override applied for this viewer
}
```

## Score tags
Score tags work the same way as name tags. `ViewScoreTag` sets the override and
`ViewPublicScoreTag` clears it:
```go
p.ViewScoreTag(target, text.Colourf("<red>1.2k kills</red>"))
p.ViewPublicScoreTag(target)
```
Reading back:
```go
if score, ok := p.ViewLayer().ScoreTag(target); ok {
    // score is the override applied for this viewer
}
```

## Visibility
Visibility is controlled with a `world.VisibilityLevel`. There are three levels:
* `world.PublicVisibility()` — no override; the viewer sees the entity exactly as it is publicly.
  This is the default.
* `world.EnforceVisible()` — force the entity visible for this viewer, even if it is publicly invisible.
* `world.EnforceInvisible()` — force the entity invisible for this viewer, even if it is publicly visible.

```go
p.ViewVisibility(target, world.EnforceInvisible())
p.ViewVisibility(target, world.EnforceVisible())
p.ViewVisibility(target, world.PublicVisibility()) // removes the override
```
The current visibility level can be read back as:
```go
level := p.ViewLayer().Visibility(target)
if level.EnforceVisibility() {
    // an override is in effect for this viewer
}
```

## Removing overrides
To clear all overrides (name tag, score tag, and visibility) for a single entity in one call,
use `RemoveViewLayer`:
```go
p.RemoveViewLayer(target)
```

## Lifecycle
A view layer is created with the session and closed when the session ends. Overrides are also
cleared automatically when the entity they target despawns. Players are an exception: their
overrides are preserved across temporary removals such as respawn or world transfer.

Whenever an override changes, Dragonfly resends the affected entity's metadata to the viewer.

## Examples

### Vanish a player from one viewer
```go
package main

import (
    "github.com/df-mc/dragonfly/server/player"
    "github.com/df-mc/dragonfly/server/world"
)

func vanish(viewer, target *player.Player) {
    viewer.ViewVisibility(target, world.EnforceInvisible())
}

func unvanish(viewer, target *player.Player) {
    viewer.ViewVisibility(target, world.PublicVisibility())
}
```
