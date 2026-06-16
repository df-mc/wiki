# v0.10.10

Released 22nd December 2025.

This version includes protocol support for 1.21.130, and introduces copper blocks, new enchantments, and major quality of life improvements.

## Changes

### **block**
- Implemented copper age blocks including all variants and oxidation states
- Always ignite TNT instead of burning
- Calculate damage before destroying blocks during explosions

### **enchantment**
- Implemented multishot enchantment

### **item**
- Added copper tools, weapons, and armour with correct defence points
- Implemented dynamic recipes system with decorated pot support

### **player**
- Apply knockback enchantment before passing damage to handler
- Play sound when equipping armour from inventory

### **session**
- Fixed entity action particles

### **server/cmd**
- Added `ParamDescriber` interface to allow non-reflect parameter schemas

### **entity**
- Destroy items on explosion
- Add count option to `EnchantedHitAction` and `CriticalHitAction`

### **world**
- Fix time flickering when time cycle is disabled

## Bug Fixes
- Fixed missing defence points for copper armour
