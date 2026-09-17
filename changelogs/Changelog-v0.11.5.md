# v0.11.5

Released 17th September 2026.

This version includes protocol support for 1.26.50. Worlds saved by earlier versions are upgraded to the new block states as they are loaded.

NetherNet is added alongside RakNet: RakNet is removed in the next version, so migrating now is recommended

## Changes

### **world**
- Added automatic upgrading of worlds saved before 1.26.50: fences, glass panes, stained glass panes, iron bars, copper bars, tripwire and stairs are given the states 1.26.50 adds to them, and the ones they derive from the blocks around them are recalculated once the chunks around them are loaded
- Upgrading runs once per chunk as it is loaded and costs nothing afterwards: no conversion step is needed before starting a server on an existing world
- Added the `StateDeriver` interface for blocks deriving part of their state from the blocks around them
- Added `LegacyStates()` and `MarkStatesUpgraded()` to `Chunk` for tracking chunks that still hold pre-1.26.50 block entries

### **server**
- Added NetherNet as a transport alongside RakNet, selected through the new `Network.Transport` list
- RakNet remains the default in this version but is removed in the next, so moving to NetherNet now is recommended
- Added `Network.NetherNet` configuration for the signaling address, identity key file, identity domain and UDP port range
- Added `NetherNetConfig` for building a NetherNet listener with a custom `http.Server` and ICE credentials
- Added `ListenNetwork` for listening on any `minecraft.Network` implementation from a `Config.Listeners` function

### **block**
- Implemented the blocks added in 1.26.50:
  - Poplar wood type, covering planks, logs, wood, stairs, slabs, fences, fence gates, doors, trapdoors and signs
  - Poplar trees grown from saplings, each carrying one of the orange, red and yellow leaf colours throughout
  - Shelf mushrooms, attaching to the side of any block with a full surface and bouncing entities that land on them
  - Red shrubs, spread to the blocks around them with bone meal
  - Straw beds, which are slept in without setting a spawn point and break once their sleeper wakes
  - Wool and concrete slabs and stairs in all sixteen colours
- Added connection states to fences, glass panes, stained glass panes, iron bars, copper bars and tripwire, which are now recalculated as the blocks around them change
- Added the corner state to stairs, forming inner and outer corners with the stairs around them
- Implemented ice, with its friction and melting behaviour
- Implemented mycelium
- Implemented trees grown from saplings for every existing sapling type

### **session**
- Ignored the `SetPlayerFurnaceOptions` packet rather than reporting it as unhandled

### **world/biome**
- Added the dappled forest biome and aligned biome definitions and tags with BDS

### **item/recipe**
- Updated item tags for 1.26.50

## Breaking Changes

### **block**
- `Bed` now carries an `Occupied` state alongside its `Sleeper`, so that occupied beds reach the client
- Sleeping is now driven by the `Sleepable` interface rather than `Bed` directly, allowing `StrawBed` to be slept in
- Stairs, fences, glass panes, stained glass panes, iron bars, copper bars and tripwire carry new state fields

### **chunk**
- `CurrentBlockVersion` updated from 1.19.70.15 to 1.26.50: chunks written by earlier versions are upgraded when loaded, and chunks written by this version are not readable by earlier ones

## Bug Fixes

### **block**
- Fixed stairs and tripwire losing their facing and other states when loaded from a world saved before 1.26.50
- Fixed a panic when cloning a registry holding compound block hashes

### **item**
- Fixed the attack damage component not being sent for custom weapons

### **server**
- Fixed a listener that fails to be created being kept in the server, causing a panic when it was accepted on
