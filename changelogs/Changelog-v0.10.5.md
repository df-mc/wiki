# v0.10.5

Released 3rd July 2025.

This version includes protocol support for 1.21.93.

## Changes
- Packets are now processed in batches, allowing for up to 256 packets to be processed in a single world transaction. This provides a big boost in performance however if you notice this causing any problems, please create an issue on GitHub.
- Creative items are now registered on startup, allowing users to register vanilla items and retain order in the inventory.

### **item**
- `Stack.WithValue()` now sets data to nil when last value is removed, fixing comparable checks.
- Implemented the `OffHand` interface for vanilla items that can be held in the off-hand.
- Added `DiscTears()` and `DiscLavaChicken()` music disc types.

### **item/inventory**
- Added support for slot validation to restrict items that can be placed in the off-hand inventory.

### **player**
- Added the following methods relating to debug shapes:
  - `AddDebugShape(debug.Shape)`
  - `RemoveDebugShape(debug.Shape)`
  - `VisibleDebugShapes() []debug.Shape`
  - `RemoveAllDebugShapes()`
- Added the following methods relating to on-screen HUD elements:
  - `ShowHudElement(hud.Element)`
  - `HideHudElement(hud.Element)`
  - `HudElementHidden(hud.Element) bool`

### **player/debug**
- Added a new package implementing debug shapes which can be sent to a player to be drawn in the world.
- The following types have been added:
  - `interface Shape`
  - `struct Arrow`
  - `struct Box`
  - `struct Circle`
  - `struct Line`
  - `struct Sphere`
  - `struct Text`

### **player/hud**
- Added a new package implementing on-screen HUD elements that can be hidden/shown to players.
- The following functions have been added:
  - `PaperDoll() Element`
  - `Armour() Element`
  - `ToolTips() Element`
  - `TouchControls() Element`
  - `Crosshair() Element`
  - `HotBar() Element`
  - `Health() Element`
  - `ProgressBar() Element`
  - `Hunger() Element`
  - `AirBubbles() Element`
  - `HorseHealth() Element`
  - `StatusEffects() Element`
  - `ItemText() Element`

### **world**
- Added a `IgnoreTotem() bool` method to the `DamageSource` interface, allowing for void damage to ignore Totems of Undying.

### **world/sound**
- Added `LightningExplode` and `LightningThunder` sound types.

## Bug Fixes
- Fixed a rare edge case where blocks due to false collision detections.
- UI Inventory is resent after every update to prevent desync.
- Removed extra velocity updates to firework trails causing them to go in front of the player.
- Effects are deleted from the EffectManager before ending, fixing invisibility issues.
- Remove unsetting of invisible flag when in Spectator mode.
- Players can no longer attack dead entities.
