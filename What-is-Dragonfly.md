Dragonfly is a server software for Minecraft that's powered by [Go](https://golang.org/).

## Intended audiences
Dragonfly is intended to be used by developers rather than server owners, unlike other server software, most of the behavior has to be controlled by code.

## Differences from other server software
Dragonfly is a library that has to be included by another program, where the actual logic would be controlled, altered and extended by the main program.

This differs in contrast to other Plug and Play server software, where the server exists as the main program that which includes user-provided plugins that allow the server's behavior to be modified.

Dragonfly on the other hand works as a library as opposed to a Plug and Play server, therefore it has no plugin system built-in.

## Philosophy of Dragonfly
This section covers the design philosophy of Dragonfly and it's differences from other server software, these details are mostly written for developers.

### Abstracted Packets
Dragonfly believes in abstraction of the packet layer, thus it does not provide any method to access the underlying packet layer, if there's a missing API for something you want, you are free to open an [issue](https://github.com/df-mc/dragonfly/issues?q=is%3Aissue+is%3Aopen+sort%3Aupdated-desc) or a [pull request](https://github.com/df-mc/dragonfly/pulls?q=is%3Apr+is%3Aopen+sort%3Aupdated-desc).

### Per-Player Event Handling
Dragonfly does not have a traditional event handling system, instead a per-player handler can be registered to handle all actions originated from said player.

## Community
If you have any other questions, come join us in the [Bedrock Gophers](https://discord.gg/U4kFWHhTNR) Discord server!
