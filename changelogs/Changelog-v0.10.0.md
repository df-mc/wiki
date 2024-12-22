# v0.10.0

Released December xxrd 2024.

This version introduces significant changes to Dragonfly, making world access happen through
synchronised transactions.

Dragonfly v0.10.0 requires **Go 1.23** at the least.

## Changes

### **biome**
- Added pale garden biome.

### **block**
- Added pale oak blocks.
- Added resin blocks.
- Added end rods.
- Added pink petals.
- Leaves now have a chance to drop sticks when breaking.
- Crops now drop their respective seeds/items when "popping off" as a result of low light levels.
- Fixed door placement consuming 3 door items.
- Sugar canes now also break without water nearby on `(SugarCane).RandomTick`.

### **cmd**
- Command names and aliases now must be lowercase. (https://github.com/df-mc/dragonfly/issues/946)

### **creative**
- Fixed several items, such as banners and skulls, not showing up correctly in the creative inventory. ()

### **dialogue**
dialogue is a new package for sending entity dialogues to players. These are UI windows with text, buttons and an
entity. The entity may be rotated. Dialogues may be created using `New` and can be sent to players using 
`(*player.Player).SendDialogue`.

### **effect**
- Effects now start their tick count the moment they are added to `*entity.EffectManager`. This means that effects now 
  always tick predictably and consistently. (https://github.com/df-mc/dragonfly/issues/944)
- Changed effect types to variables. `Poison{}` is now `Poison`.
- Potency fields were removed from `InstantHealth` and `InstantDamage`, instead having been added to a new
  `NewInstantWithPotency` function.

### **item**
- Added `NightVisionTorchflowerStew`, `BlindnessEyeblossomStew` and `NauseaStew` suspicious stew types.
- Added resin item.
- Removed armour trims from `Stack`. To replace it, a `Trim` field has been added to `Helmet`, `Chestplate`, 
  `Leggings` and `Boots`.
- Changed maximum stack count of written books to 16.
- Fixed bottles not being filled up with water when using them on a water source. (https://github.com/df-mc/dragonfly/issues/945)
- Removed `DisplayName` function.

### **world**
- Added a new `(*World).Exec` method that is used to open a transaction on the world. Many methods on `*World`, such
  as `(*World).Block`, have now been moved to `(*Tx).Block` instead. More information may be found at the 
  [wiki page for world transactions](https://github.com/df-mc/dragonfly/wiki/World-Transactions).
- Added a new `*EntityHandle` type that uniquely identifies an entity. `Entity` implementations are only valid in the
  context of a transaction now.
- Added a `EntityAnimation` type that allows for complex entity animations to be played.
- Added a new `HandleLeavesDecay` method to `Handler`, called when leaves blocks decay.
- Added a new `SaveInterval` field to `Config` that specifies how often worlds should auto-save.
- Scheduled block updates are now tied to the specific block type that they are called for.
- Scheduled block updates are now saved and loaded to/from disk.
- Entities are now stored on disk using the new format. (https://github.com/df-mc/dragonfly/issues/516)
- Worlds with a void generator no longer panic when being loaded. (https://github.com/df-mc/dragonfly/issues/481)
- Fixed panic when closing a world immediately after opening it. (https://github.com/df-mc/dragonfly/issues/801)

### **enchantment**
- Changed enchantment types to variables. `Sharpness{}` is now `Sharpness`.

### **player**
- Added `HandleHeldSlotChange` method to `Handler`, called when a player changes held slot. Additionally, a new
  `(*Player).SetHeldSlot` method was added.
- Added `HandleItemRelease` method to `Handler`, called when a player releases and item such as a bow.
- `Data` has been refactored in a more easily usable `Config` that can be used to create a player entity.
- Attack immunity was changed to match vanilla. If `(*Player).Hurt` is called twice in the same tick, damage is still
  dealt if the second damage was higher than the first. Additionally, attack immunity now extends to all damage sources. (https://github.com/df-mc/dragonfly/issues/950)
- Saturation and food tick are now reset when `(Handler).HandleFoodLoss` is cancelled.
- Releasable items such as bows no longer fire when switching held slots. (https://github.com/df-mc/dragonfly/issues/676)

### **cube**
- `Rotation` has a new `Neg` method that returns the negative of both `Rotation` values, used for projectiles.

### **server**
- `(*Server).Accept` now returns an `iter.Seq` of joining players that can be iterated over.
- Disconnecting players returned by `(*Server).Accept` no longer deadlocks the server. (https://github.com/df-mc/dragonfly/issues/527)
- Added `StatusProvider` field to `Config` for custom MOTD/player count values.