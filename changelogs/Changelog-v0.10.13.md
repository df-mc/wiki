# v0.10.13

Released 7th May 2026.

This version includes protocol support for 1.26.20 and introduces new blocks, configuration options, and important fixes.

## Changes

### **block**
- Implemented infested blocks for all stone types
- Implemented cobweb block
- Implemented magma block
- Fixed fortune enchantment not applying to iron and gold ore drops
- Fixed liquid-removable blocks dropping items when removed by lava

### **cube/pos**
- Added `Within()` method to check if position is within a range

### **item**
- Implemented honey bottle

### **player**
- Reduce durability every time player attacks living entity

### **server**
- Added compression support to server configuration
- Avoid holding HUD lock while sending HUD update packets

### **server/block**
- Implement `String()` method for blocks

### **server/world**
- Added per-viewer view layers
- Fixed panic in `entitiesWithin()` by using `slices.Clone()` for safe iteration
- Fixed debug drawer deadlock by queueing mutations

### **session**
- Fixed empty nametag metadata

## Breaking Changes
- Minimum Go version bumped to 1.26.0
