# v0.10.14

Released 16th June 2026.

This version includes protocol support for 1.26.30 and introduces a major breaking change: the block registry is now instance-scoped to allow different worlds to have different block configurations

## Changes

### **block**
- Bonemeal huge growth particles are now displayed when bone meal is used on applicable blocks
- Smelter blocks now have correct light emission levels
- Connected fence and pane collision has been aligned with vanilla

### **chunk**
- Added `Clone()` methods to `Chunk`, `SubChunk`, and `Palette` for easier duplication

### **cube/trace**
- Added `BlockIntersects()` and `BBoxIntersects()` methods for collision detection

### **item**
- Implemented the piercing enchantment, allowing projectiles to pass through multiple entities
- Added support for applying incompatible enchantments to stacks via `Stack.WithIncompatibleEnchantments()`

### **server/block**
- Implemented basic redstone components:
  - Redstone wire
  - Redstone torch
  - Redstone block
  - Lever
  - Redstone ore
  - Note block redstone activation

### **server**
- Golden apple absorption effects now stack correctly and clear upon depletion

### **world**
- Block registry is now instance-scoped, allowing different worlds to have different block configurations
- Generators now have a `DefaultSpawn()` method to define custom spawn positions
- Added `RuntimeIDToHash()` method to block registry for efficient block identification

### **world/chunk**
- Optimized light calculations by replacing linked-list queue with ring-buffer and removing iterator-based traversal

### **player**
- Fixed incorrect eye height for swimming, crawling, gliding, and prone positions

## Bug Fixes
- Fixed bone meal duplication glitch
- Fixed crossbow charging animation playing even without suitable projectiles available
- Fixed crossbow critical flag to always match vanilla behaviour
- Fixed fence and pane collision misalignment with vanilla
