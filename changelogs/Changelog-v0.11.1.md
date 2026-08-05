# v0.11.1

Released 5th August 2026.

This version includes protocol support for 1.26.40, adds End crystals, custom dimensions and concurrent chunk loading, and reworks the explosion API around `world.ExplosionSource`.

## Changes

### **block**
- Implemented End crystals
- Added cinnabar and sulfur building block sets, including bricks, polished variants, slabs, stairs and walls
- Blast resistance and explosion behaviour now match vanilla
- Reduced explosion block selection overhead

### **cube**
- Bounding boxes are now generic over `float32` and `float64`

### **entity**
- Added support for the `AlwaysShowNameTag` data flag
- Reduced movement collision scan overhead

### **item**
- Moved the item NBT codec out of `internal/nbtconv` into `server/item`, so importing `server/item` on its own no longer depends on another package pulling the codec into the binary

### **player**
- Added the `HandleSetOnFire` player handler, receiving the duration the player is set on fire for
- The airborne mining penalty is no longer applied to flying players

### **server**
- Replaced explosion positions with a `world.ExplosionSource`, giving handlers the originating block or entity alongside the position, and deriving radius, item drop chance and damage from a single size
- `config.Allow` is now passed to gophertunnel for earlier handling, removing redundant resource pack serving

### **session**
- The player's build platform is now included in the player list

### **world**
- Added concurrent chunk loading, generation and light calculations, moving disk reads, generation and lighting off the transaction goroutine:
  - Added `Config.ChunkLoadWorkers` (default 1) to generate multiple chunks in parallel, requiring a concurrency-safe generator
  - Provider calls are serialised internally, so providers need not be concurrency-safe
- Added a custom dimension registry via `RegisterDimension()`, accepting IDs between 1000 and 65535
- Added the `HandlePortalCreate` and `HandlePortalActivate` world handlers for portal lifecycle events

### **world/sound**
- Added `sound.Custom` for playing sounds defined by a resource pack

## Breaking Changes

### **world**
- `Loader.Load` is now asynchronous: chunks appear after it returns
- `Tx.Light` returns 0 for unloaded chunks instead of generating them
- `World.HighestLightBlocker` is now off-owner only: use `Tx.HighestLightBlocker` inside a transaction
- Explosion handlers and `Explodable` now take a `world.ExplosionSource` instead of a position, and `ExplosionConfig` is no longer passed to `Explodable`
- `Handler` gained `HandlePortalCreate` and `HandlePortalActivate`, which custom handlers embedding `NopHandler` inherit

### **cube**
- `cube.BBox` is now generic, so code naming the type explicitly needs a type parameter

### **item**
- `nbtconv.WriteItem` and `nbtconv.MapItem` moved to `server/item`

## Bug Fixes

### **block**
- Fixed panic when calling `BreakInfo()` on unplaced smelter blocks
- Fixed panic when decoding pot sherd NBT
- Fixed outdated custom block components
- Custom block components now send destroy seconds instead of hardness
- Hopper cooldown ticks are no longer broadcast as block changes

### **server**
- Fixed End crystal fire and placement parity with vanilla
- Simplified and fixed End crystal explosion logic
- Empty custom item entries are no longer sent

### **world**
- Fixed the outermost ring of the load radius being dropped when populating the load queue
- Fixed chunk distance overflowing int32 for far-apart positions, such as long teleports
