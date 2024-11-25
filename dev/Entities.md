[![cmd](https://badges.fyi/static/go.dev/Documentation/29BEB0)](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/entity)

## Index
* [Animations](#entity-animations)
* [Resources](#resources)

## Entity Animations
In order to play an animation one must be defined in one of your resource packs. This is out of scope for this guide so look at the [resources](#resources) for more information. But at the minimum you will need to create an animation and define it in your client-entity file. Once you have these requirements you can now play an animation for your entity.

Note: any changes made to your [animation](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/entity/animation#Animation) will not be automatically applied. You will need to call [PlayAnimation](https://pkg.go.dev/github.com/df-mc/dragonfly/dragonfly/world#World.PlayAnimation) again with your new animation.

Example of playing an animation:
```go
// var p *player.Player

// Play our sit animation and end the animation once the player jumps
p.World().PlayAnimation(p, animation.New("animation.player.sit").WithStopCondition("query.is_jumping"))
```

## Resources
* [learn.microsoft.com animation guide](https://learn.microsoft.com/en-us/minecraft/creator/reference/content/animationsreference/examples/animationgettingstarted)
* [wiki.bedrock.dev animation controller guide](https://wiki.bedrock.dev/animation-controllers/animation-controllers-intro.html)