# v0.11.2

Released 15th August 2026.

This version includes protocol support for 1.26.44, implements tinted glass and new recipe types, and is largely a bug fix release.

## Changes

### **block**
- Implemented tinted glass

### **item/recipe**
- Added `UserDataShapeless` recipes for items that retain their user data when crafted, such as dyed shulker boxes, bundles and harnesses
- Added `Multi` recipes, identified by UUID, enabling client-side hardcoded behaviour such as map cloning, book cloning, banner duplication and firework crafting
- Registered the vanilla shulker box and multi recipes

### **player**
- Exported `NewEventContext()` for creating a player event context from a transaction and player, panicking if the two do not belong to the same callback

### **session**
- The build platform is now sent as `DeviceUnknown` rather than being left unset
- Removed the deprecated survival and creative spectator game types, which Mojang dropped from the game mode enum in 1.26.40

## Bug Fixes

### **block**
- Fixed blocks that can stack, such as candles, pink petals, sea pickles and slabs, dropping the wrong variant
- Fixed composters consuming the held item when bone meal is dropped, and fixed their break drops
- Fixed lecterns taking the whole stack of books used on them
- Fixed sign and campfire state being lost when saved
- Fixed hoppers destroying items through paired chests and decorated pots
- Fixed explosions dropping both halves of a two block structure
- Fixed custom block ambient occlusion being encoded as an integer instead of a float

### **entity**
- Fixed projectiles being picked up by more than one collector
- Fixed falling blocks dealing no damage when they land
- Fixed area effect cloud effects not applying at the correct rate
- Fixed water bottles only extinguishing newly lit fire blocks
- Fixed durations on entities being lost when they are saved

### **item**
- Corrected armour durability, the respiration enchantment and poison damage against vanilla
- Fixed item duplication caused by `Stack.Grow` being used as a setter in the loom and smithing table handlers

### **player**
- Fixed the block a player stands on being looked up below its centre

### **session**
- Fixed a nil dereference when viewing an entity with no armour
- Fixed client disconnects on player teleport
- Fixed disconnects when viewing sessionless player entities
- Empty skin models are no longer sent
- Fixed the recipe book showing recipes that should be hidden

### **world**
- Fixed a world with no provider raining from the first tick and dereferencing a nil biome
- Fixed lightning eligibility using an entity's Z coordinate as its height
