# v0.10.11

Released 21st March 2026.

This version includes protocol support for 1.26.0, a major version jump that brings significant new blocks, gameplay features, inventory fixes, and requires Go 1.24.

## Changes

### **block**
- Implemented beds with full functionality
- Implemented azalea leaves, slime blocks, and smooth basalt blocks
- Fixed skull blocks to not determine hanging state from NBT decode
- Drop blocks without unintended NBT data

### **chunk**
- Improved `compact()` performance and reduced allocations in paletted storage

### **cube/pos**
- Added `Min()`, `Max()`, and `Range3D()` functions for spatial queries
- Added `Within()` method to check if position is within a range

### **enchantment**
- Implemented fortune enchantment with proper ore drop multipliers
- Fixed silk touch to not drop experience from mined blocks

### **entity**
- Fixed spectators being able to move experience orbs
- Added baby entity interface for young entity variants

### **model**
- Fixed incorrect bounding boxes for cocoa bean, wall, tilled grass, and brewing stand models

### **player**
- Allow placing blocks that intersect with experience orbs
- Only resend blocks if within player render distance
- Fixed container and inventory desync issues

### **player/debug**
- Added missing scale field to text shape
- Implemented attaching debug shapes to entities

### **player/form**
- Implemented divider and header form elements
- Allow label element on menu forms

### **server/conf**
- Added `SaveInterval` and `ChunkUnloadInterval` configuration options

### **server/session**
- Export method to build `AvailableCommands` packet
- Decouple `debugShapeToProtocol` from session
- Detect enum changes and promote to soft enums dynamically

### **session/entity_metadata**
- Added baby interface for entities

### **world**
- Fixed spawn points in other dimensions
- Do not allow disabling chunk unloading via configuration
- Fixed block state decoding

### **world/chunk**
- Force send `NetworkChunkPublisherUpdate` when session world switches
- Only send `NetworkChunkPublisherUpdate` on chunk position change

### **world/mcdb**
- Bumped chunk version from 41 to 42

### **item/stack**
- Make empty items return empty data

### **bossbar**
- Updated colour values

### **server**
- Use British English spelling throughout codebase

### **session**
- Fixed various container and inventory desync issues

## Breaking Changes
- Minimum Go version requirement updated to 1.24
