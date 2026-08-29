# v0.11.3

Released 29th August 2026.

This version includes protocol support for 1.26.45 and is otherwise a small bug fix release.

## Changes

### **block**
- Added `ContainerSize()` to the `Container` interface, exposing the inventory size of barrels, blast furnaces, brewing stands, chests, decorated pots, ender chests, furnaces, hoppers, shulker boxes and smokers

## Bug Fixes

### **block**
- Fixed vines being unable to attach to leaves

### **item**
- Fixed a panic when reading an enchantment with a level below 1 from NBT

### **server**
- Fixed a listener that fails to bind being kept in the server configuration

### **session**
- Fixed a panic when closing a session without a transaction
