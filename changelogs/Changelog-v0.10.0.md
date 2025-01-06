# v0.10.0

Released January 5th, 2025.

This version introduces significant changes to Dragonfly, making world access happen through
synchronised transactions. More information on this system can be found at the [wiki page for world transactions](https://github.com/df-mc/dragonfly/wiki/World-Transactions)

Dragonfly v0.10.0 requires **Go 1.23** or above.

## Changes

### **block**
- Added pale oak blocks.
- Added resin blocks.
- Added end rods.
- Added pink petals.
- Added vines.
- Decorated pots can now have sherds and be filled with a stack of items.
- Leaves now have a chance to drop sticks when breaking.
- Crops now drop their respective seeds/items when "popping off" as a result of low light levels.
- Fixed door placement consuming 3 door items.
- Sugar canes now also break without water nearby on `(SugarCane).RandomTick`.
- Explosions now correctly calculate exposure for entities behind blocks with non-full bounding boxes. (https://github.com/df-mc/dragonfly/issues/709)
- Explosions from TNT now drop 100% of drops. (https://github.com/df-mc/dragonfly/issues/969)
- Fixed fire created by lava or lightning not updating. (https://github.com/df-mc/dragonfly/issues/977)
- Fixed note blocks retaining their last note when dropped. (https://github.com/df-mc/dragonfly/issues/854)
- Fixed dead bushes not being placeable on grass/mud. 

### **block/cube**
- `Rotation` has a new `Neg` method that returns the negative of both `Rotation` values, used for projectiles.

### **cmd**
- Command names and aliases now must be lowercase. (https://github.com/df-mc/dragonfly/issues/946)
- Target selectors (`[]Target`) now only yields players from the same world. This is a consequence of the world
  transactions and is likely to be reverted in a future Dragonfly version.
- The `*world.Tx` in which a command is run is now passed to `(Runnable).Run`.

### **entity**
- Fixed splash/lingering potions and bottles o' enchanting throwing velocity not matching vanilla. (https://github.com/df-mc/dragonfly/issues/746)
- Fixed projectile knockback to reflect projectile velocity rather than collision position.

### **entity/effect**
- Effects now start their tick count the moment they are added to `*entity.EffectManager`. This means that effects now
  always tick predictably and consistently. (https://github.com/df-mc/dragonfly/issues/944)
- Changed effect types to variables. `Poison{}` is now `Poison`.
- Potency fields were removed from `InstantHealth` and `InstantDamage`, instead having been added to a new
  `NewInstantWithPotency` function.
- Fixed effect colours not matching the latest Minecraft version. (https://github.com/df-mc/dragonfly/issues/746)

### **item**
- Added `NightVisionTorchflowerStew`, `BlindnessEyeblossomStew` and `NauseaStew` suspicious stew types.
- Added `Resin` item.
- Added `Crossbow` item.
- Added new `(Stack).WithItem` method to return an identical `Stack` with a different `world.Item` type.
- Removed armour trims from `Stack`. To replace it, a `Trim` field has been added to `Helmet`, `Chestplate`, 
  `Leggings` and `Boots`.
- Changed maximum stack count of written books to 16.
- Fixed bottles not being filled up with water when using them on a water source. (https://github.com/df-mc/dragonfly/issues/945)
- Removed `DisplayName` function.

### **item/creative**
- Fixed several items, such as banners and skulls, not showing up correctly in the creative inventory. ()

### **item/enchantment**
- Changed enchantment types to variables. `Sharpness{}` is now `Sharpness`.

### **player**
- Added `HandleHeldSlotChange` method to `Handler`, called when a player changes held slot. Additionally, a new
  `(*Player).SetHeldSlot` method was added.
- Added `HandleItemRelease` method to `Handler`, called when a player releases and item such as a bow.
- Added a new `Messaget` method to send a `chat.Translation`.
- `Data` has been refactored in a more easily usable `Config` that can be used to create a player entity.
- Attack immunity was changed to match vanilla. If `(*Player).Hurt` is called twice in the same tick, damage is still
  dealt if the second damage was higher than the first. Additionally, attack immunity now extends to all damage sources. (https://github.com/df-mc/dragonfly/issues/950)
- Saturation and food tick are now reset when `(Handler).HandleFoodLoss` is cancelled.
- Releasable items such as bows no longer fire when switching held slots. (https://github.com/df-mc/dragonfly/issues/676)
- Fixed an issue where `(*Player).HideEntity` would leak entities. (https://github.com/df-mc/dragonfly/issues/978)
- Fixed `(*Player).ReleaseItem` not using items in the left hand as projectiles.
- Fixed no block break particles/progress/arm swinging happening. (https://github.com/df-mc/dragonfly/issues/960) Block
  cracking is now fully server-side.
- Fixed a bug where colliding with a wall while falling could negate or trigger fall damage. (https://github.com/df-mc/dragonfly/issues/779)
- Stopped block resending when placing blocks close to the player, leading to reduced synchronisation issues.

### **player/chat**
- Added a new `Translate` function used for (client-side) translations. Passing a `TranslationString` to `Translate`
  returns a `Translation`. `Translation`s may require arguments that can be passed in `(Translation).F`.
- Added a new `Translator` interface for `Subscriber`s that can translate messages.
- Added a new `(*Chat).Messaget` function to broadcast a translation that is localised for each subscriber that also
  implements `Translator`.

### **player/dialogue**
dialogue is a new package for sending entity dialogues to players. These are UI windows with text, buttons and an
entity. The entity shown may be rotated. Dialogues may be created using `New` and can be sent to players using
`(*player.Player).SendDialogue`.

### **server**
- `(*Server).Accept` now returns an `iter.Seq` of joining players that can be iterated over.
- Disconnecting players returned by `(*Server).Accept` no longer deadlocks the server. (https://github.com/df-mc/dragonfly/issues/527)
- Added `StatusProvider` field to `Config` for custom MOTD/player count values.

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

### **world/biome**
- Added `PaleGarden` biome.

### **world/sound**
- Added `CrossbowLoad` and `CrossbowShoot`
