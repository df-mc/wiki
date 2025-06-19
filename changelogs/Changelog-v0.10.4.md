# v0.10.4

Released 19th June 2025.

This version includes protocol support for 1.21.90.

## Changes
- Packet sending & encoding is now handled in a different go-routine for each player, providing performance improvements.

### **world**
- A new `Blocks()` method has been added, returning a list of all registered blocks.


## Bug Fixes
- Crafting and other recipes have now been fixed.
- Fixed a race condition where the "Server closed" message would sometimes not be logged.
- Fixed players taking fall damage when sprinting down stairs.
- Legacy item slots are now being resent to the client, preventing inventory desyncs.
- NBT data returned from `unknownBlock` is now cloned to prevent issues when being modified.
- Fixed a panic when loading chunks from vanilla with unregistered items.
- `minecraft:air` items are considered empty.
- Fixed `BBox`'s `ExtendTowards` method flipping behaviour in the Y axis.
- Custom sounds now play properly
- Fixed a deadlock that can occur when closing a world
