# v0.9.19

Released 23rd November 2024.

This version includes protocol support for 1.21.40 as well as a number of bug fixes and new features.

## Bug Fixes

- Fixed incorrect interface definition causing Bows enchanted with Punch not to work
  correctly (https://github.com/df-mc/dragonfly/pull/927).
- Fixed a nil pointer that could occur with Ender Chests (https://github.com/df-mc/dragonfly/pull/929).
- Fixed incorrect calculations with the Efficiency enchantment.
- Players will no longer fall through blocks when being
  teleported (https://github.com/df-mc/dragonfly/pull/931).
- Fixed various incorrect item identifiers being returns by some blocks.
- Updated various hardness and blast resistance values for blocks.
- Fletching Tables no longer cause a panic when being picked.
- Fixed cost of creating multiple banners in a Loom (https://github.com/df-mc/dragonfly/pull/934).
- Enchanted Golden Apples were updated to now provide Regeneration II to match vanilla.
- Calculations for damage caused by TNT explosions have been fixed.
- Spectators can no longer pick up arrows (https://github.com/df-mc/dragonfly/pull/879)

## Changes

### **block**

- The `Mangrove` and `Cherry` functions have been renamed to `MangroveWood` and `CherryWood` respectively.
- The new `SneakingActivatable` interface has been added to allow blocks to be activated when sneaking.
- The new `Oxidizable` interface has been added for blocks that can oxidise.
- Many new blocks have been implemented, including `BrewingStand` `Copper`, `CopperDoor`, `CopperGrate`,
  `CopperTrapdoor`, `PolishedTuff` and `TuffBricks`.

### **block/cube**

- Three new methods have been introduced to `BBox`: `Corners`, `Mul` and `Volume`.

### **recipe**

- The new `Potion and `PotionContainerChange` structs have been introduced to allow for potion recipes to be
  registered.
- The `Perform` and `ValidBrewingReagent` functions have been introduced for working with potion recipes.

### **player**

- The `entity.SplashableEntity` interface has been implemented on the `Player` struct to allow for water
  bottles to be thrown at players to extinguish them when on fire.

### **session**

- Three new methods have been introduced to the `Controllable` interface, `StartCrawling`, `Crawling` and
  `StopCrawling`.

### **world/sound**

- The new `PotionBrewed` struct can be used to play the sound of a potion being brewed.
- The new `WaxRemoved` struct can be used to play the sound of wax being removed from a Copper block.
- The new `CopperScraped` struct can be used to play the sound of a Copper block being scraped to reduce
  oxidation.
