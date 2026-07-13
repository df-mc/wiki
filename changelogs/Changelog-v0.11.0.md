# v0.11.0

Released 13th July 2026.

This version introduces a major breaking change: the public blocking transaction API has been removed in favour of owner-safe scheduling APIs. See the [v0.11.0 Migration Guide](https://github.com/df-mc/dragonfly/wiki/v0.11.0-Migration-Guide) before upgrading. It also adds nether and End portals, shulker boxes, bamboo blocks, candles and a world-owned redstone engine.

## Changes

### **block**
- Implemented candles and candle cakes
- Implemented shulker boxes
- Implemented bamboo and bamboo sapling
- Implemented bamboo building blocks and items
- Block break duration now accounts for status effects and the surrounding environment
- Added block state accessors (`FacingDirection`/`WithFacing`, `PillarAxis`/`WithAxis`, `DyeColour`/`WithColour`) along with a `cmd/blockaccessor` generator to produce them

### **player**
- Implemented a client input lock system for selectively restricting player input
- Natural regeneration now matches vanilla behaviour

### **server**
- Implemented Nether portals
- Implemented End portals

### **world**
- Redstone is now driven by a world-owned engine
- Added a synchronous world mode via `Config.Synchronous` for deterministic, in-process worlds, with time advanced through `World.AdvanceTick`, enabling behaviour to be unit tested without a running server

### **chunk**
- Added `RuntimeIDToHash()` to the `BlockRegistry` interface

### **world/mcdb/leveldat**
- Added missing level.dat fields

## Breaking Changes

### **world**
- Removed the public blocking transaction API (`World.Exec` and `ExecFunc`) in favour of owner-safe scheduling APIs:
  - `World`, `EntityHandle` and `Player` now expose `Do()` and `DoAfter()` for fire-and-forget scheduling
  - `world.Call()`, `world.CallEntity()`, `world.CallRef()` and `player.Call()` schedule work and await a result
  - Added `world.EntityRef[T]` and `player.Ref` for owner-safe entity handles, created via `world.NewEntityRef()` and `player.NewRef()`
  - Added `Tx.Defer()` and `Tx.DeferErr()` for scheduling follow-up work from within a transaction
  - Scheduling calls return a `world.Task`, exposing `Wait()`, `Done()`, `Err()`, `Cancel()` and `OnDone()`
  - Added `world.RethrowPanic()` and `world.PanicError` for surfacing panics raised inside scheduled tasks
  - See the [v0.11.0 Migration Guide](https://github.com/df-mc/dragonfly/wiki/v0.11.0-Migration-Guide) for a full before/after walkthrough of updating existing code

## Bug Fixes

### **player**
- Fixed missing axe and hoe swing animations
- Fixed player fall state handling

### **server**
- Fixed floating arrow bug and incorrect crossbow charging check
- Guarded against out-of-range anvil index, unknown loom pattern and unopened NPC dialogue

### **entity**
- Items collected by hoppers now stop ticking

### **session**
- Fixed panic when a sessionless player logs in
- Fixed panic on out-of-range `CreateStackRequestAction` result slot
- Guarded against a nil session handle in entity metadata
- Fixed missing entity body yaw rotation
- Persistent sneaking input is now synchronised correctly

### **chunk**
- Rejected invalid network subchunk bounds during decoding

### **world/mcdb/leveldat**
- level.dat is now always read in full to prevent potential reading errors
