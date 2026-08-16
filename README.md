<div align="center">

# 🔧 REVIVED — Working on V Rising 1.1.13

### Fixed and maintained by [Fartonice](https://github.com/Fartonice-Dev)

<!-- PHOTO: replace the URL below with your image link.
     Easiest way: drag your photo into any GitHub issue comment,
     GitHub uploads it and gives you a link, paste that link here. -->
<img src="PUT_YOUR_PHOTO_LINK_HERE" width="150" />

**[github.com/Fartonice-Dev/LeadAHorseToWater](https://github.com/Fartonice-Dev/LeadAHorseToWater)**

*This mod sat broken for over two years. It works again.*

</div>

---

## ⚠️ READ THIS FIRST — Bloodstone is GONE

**This version does NOT use Bloodstone. Do not install it.**

The old version required `deca-Bloodstone`. That library is deprecated and it is
**actively harmful** on V Rising 1.1.x:

- On your **server** it breaks *every network event*. Chat commands stop working,
  achievement rewards can't be claimed, items can't be dropped or moved. The server
  looks fine but nothing gets through.
- On your **client** it crashes you straight to desktop the moment you connect.

If you have Bloodstone installed for this mod, **remove it.** This version doesn't
need it and runs better without it.


---

## 🔍 What was broken, and what I changed

Total time to diagnose and fix: **about 7 and a half hours.** Three separate faults,
stacked on top of each other. Fixing any one alone would not have worked.

### 1. The mod hooked a game system that no longer exists

It patched `ProjectM.FeedableInventorySystem_Update`. Stunlock removed it. So the
mod's entire engine never started:

```
TypeLoadException: Could not load type 'ProjectM.FeedableInventorySystem_Update'
   at LeadAHorseToWater.Plugin.OnGameInitialized()
```

The mod loaded, showed no error to the player, and then quietly did nothing.

**Fix:** hook `CheckInSunSystem` instead — a system that still exists and updates
constantly. This solution is **cheesasaurus's**, from their open PR #21.

### 2. It was built against ancient game bindings

```
VRising.Unhollowed.Client   0.6.5          ->  1.1.*
BepInEx                     6.0.0-be.668   ->  6.0.0-be.733
```

### 3. Bloodstone — the one that really mattered

This was the hard one to find, because Bloodstone *reports* that it loaded fine:

```
[Info :Bloodstone] Bloodstone v0.2.2 loaded.
```

Then under real traffic it fails on every single network packet:

```
MissingMethodException: Method not found:
'Void Stunlock.Network.NetBufferOut.Write(Byte)'
   at Bloodstone.Network.SerializationHooks.SerializeAndSendServerEventsSystem_Patch

MissingMethodException: Method not found:
'UInt32 Stunlock.Network.NetBufferIn.ReadUInt32()'
   at Bloodstone.Network.SerializationHooks.DeserializeHook
```

Hundreds per minute. Bloodstone hooks the network serialization layer, and those
game methods don't exist anymore.

**Fix:** removed Bloodstone entirely. The mod only ever used three things from it —
`VWorld.Server`, `VExtensions.ActionRef`, and `Entity.WithComponentData`. All three
are now local replacements in `Compat/`, roughly 60 lines total. Behaviour unchanged.

`[BepInProcess("VRisingServer.exe")]` replaces Bloodstone's old IsServer/IsClient
checks, which also means this can no longer be loaded onto a client by accident.

### Files changed

```
NEW      Compat/VWorld.cs
NEW      Compat/VExtensions.cs
NEW      Compat/EntityExtensions.cs
RENAMED  Patches/FeedableInventorySystem_Update_Patch.cs
            -> Patches/LeaadAHorseToWaterSystem.cs
EDITED   Plugin.cs, Commands.cs, HorseUtil.cs, ECSExtensions.cs,
         Settings.cs, LeadAHorseToWater.csproj, Processes/*, Patches/*
```


---

## 📦 Install

**This is a SERVER-SIDE mod. It goes on the server only. Never on your game client.**

### What you need on the server

| Mod | Required? |
|---|---|
| BepInEx (BepInExPack_V_Rising) | Yes |
| VampireCommandFramework | Yes — for the commands |
| ~~Bloodstone~~ | **NO. Removed. Do not install.** |

### Steps

1. Install **BepInEx** on your server if you haven't already
2. Install **VampireCommandFramework** into `BepInEx/plugins`
3. Drop `LeadAHorseToWater.dll` into `BepInEx/plugins`
4. Start the server

That's it. There is no client-side step.

### Check it worked

In the server log you should see:

```
[Info :   BepInEx] Loading [LeadAHorseToWater 0.9.0]
[Info :LeadAHorseToWater] Plugin LeadAHorseToWater is loaded! (Bloodstone-free build)
[Info :LeadAHorseToWater] Trying to find VCF:
[Warning:LeadAHorseToWater] VCF Version: 0.11.0
```

Then in game type `.help horse` — if you get a command list back, you're running.

---

## 🙏 Credits

This is a community revival. **Most of the code here is not mine.**

**[decaprime](https://github.com/decaprime/LeadAHorseToWater) — original author.**
Wrote the mod. The idea, the well detection, breeding, all the commands. Without
decaprime there is nothing here to fix. Thank you for building it.

**[cheesasaurus](https://github.com/cheesasaurus/LeadAHorseToWater) — V Rising 1.0 compatibility ([PR #21](https://github.com/decaprime/LeadAHorseToWater/pull/21)).**
Did the hardest diagnostic work. Found the dead `FeedableInventorySystem_Update`
hook and worked out that `CheckInSunSystem` could replace it — and left a comment
listing every other system they tried that *didn't* work, which saved me hours of
repeating it. Also improved breeding and horse-killing. Their PR has been open and
unmerged since September 2024. **I built directly on it.**

**The tools underneath:** the **BepInEx** team, **deca** for VampireCommandFramework,
and **Thunderstore** for hosting the V Rising modding community.

**Fartonice (me):** removed the Bloodstone dependency, rebuilt against V Rising 1.1.x,
tested on a live dedicated server, packaged it back up.

### A note on licensing

decaprime's repository has no LICENSE file. This fork exists to keep a good mod alive
for the community, with full credit and links back to the original authors.

**decaprime — if you'd prefer this taken down, changed, or merged back into your repo,
just say the word and I'll do it.** No argument. It's your mod.


---
---

# 🐴 Lead A Horse To Water

A V-Rising mod that lets your horses drink water from wells. Now with horse breeding and other commands.

## Motivation
<img src="https://user-images.githubusercontent.com/62450933/175367019-be27ef84-4676-45cc-809c-41e7244d3594.png" width="300" />

> *Bye, bye, Li'L Sebastian* <br>
> *Miss you in the saddest fashion* <br>
> *You're 5000 candles in the wind*


# Commands and Other Features

For commands to load you must have [VampireCommandFramework](https://github.com/decaprime/VampireCommandFramework) installed on the server as well.

**Notes**:
- Commands with `[horse=]` are optionally for specifying a horse by name and will default to the closest horse if not specified.
- 🔒 Requires admin permissions
- <ins>_Underlined_</ins> keys come from the config file.
- Horses should be tamed and not riden by a player for most commands

### This info is also **available in game** with `.help LeadAHorseToWater`

---

#### `.horse breed`

This process takes two horses and consumes <ins>_BreedingRequiredItem_</ins> * <ins>_BreedingCostAmount_</ins> from the player's inventory. The resulting horse will be a random mix of the two parents' stats as a 50/50 chance of inheriting each trait from either parent. Then randomly +/- <ins>_MutationRange_</ins> is applied based on the max stat for each attribute. Finally values are capped at <ins>_MaxSpeed_</ins>, <ins>_MaxAcceleration_</ins>, <ins>_MaxRotation_</ins>. The resulting horse will be named after the first parent.

> **Tip:** the default breeding item is the **twilight snapper** (prefab `-570287766`).
> You can change it to any item via `BreedingRequiredItem` in the config.

![image](https://user-images.githubusercontent.com/62450933/190880543-92d31267-34ec-4292-bb03-b12feee5a95b.png)

#### `.horse tag-stats [horse=]`

This player command will add the current stats of the horse to it's name as a suffix. This is great to see the stats of a horse without having to open the inventory. This command may limited by name length as the game allows one extra character via their UI.

![image](https://user-images.githubusercontent.com/62450933/190880667-fac067fe-764b-4e89-a059-f37ee8221fe1.png)

#### 🔒 `.horse rename [horse=] (newName)`

Powerful admin rename, this allows you do escape normal naming restrictions and use markup. This is useful for special rewards like making a horse's name color, bold, or even use some emojis or unicode.
**PSA**: This command can result in the drinking prefix breaking or tag-stats not fitting on the horse. Please don't report naming issues resulting from this command.

#### 🔒 `.horse whistle [horse=]` / `.horse warp [horse=]`

Whistle tries to brings the horse to you, warp teleports you to the horse.

#### 🔒 `.horse speed [horse=] (speed)`
#### 🔒 `.horse acceleration [horse=] (acceleration)`
#### 🔒 `.horse rotation [horse=] (rotation)`

Set the horse's stats. These values are **not** capped by <ins>_MaxSpeed_</ins>, <ins>_MaxAcceleration_</ins>, or <ins>_MaxRotation_</ins>. Note that the game represents rotation as 10x the value displayed in the UI but the commands handle this for you and you should refer to the values as you see them in the UI.

#### 🔒 `.horse kill [horse=]`

Removes the horse immediately without any loot or corpse.

#### 🔒 `.horse cull [radius=5] [percentage=1]`

**WARNING**: This command will remove horses within the radius of the player. It choose `percentage` of the horses within the `radius`. This command is very useful for cleaning up a large number of horses. It is recommended to use a small radius to start. The default radius of 5 is about 1 tile. The default percentage 1 means 100% of the horses within the radius will be removed.

#### 🔒 `.horse famish [radius=5]`

Make all horses within the `radius` hungry enough to feed. (depletes satiety)

#### 🔒 `.horse spawn [count=1]`

Spawns either one or `count` horses around you.


## 🚰 About the wells

The mod does **not** use wells out in the world. It looks for **castle fountains**
placed inside your castle and connected to a **Castle Heart**.

By default it accepts the **Stone** and **Large** fountains. Options are
`stone, iron, bronze, small, large` — set them in `EnabledWellPrefabs`.

The horse must be within **`DistanceRequired`** of the fountain. Default is `5`,
which is about **one tile** — so right up against it.

When a horse is drinking, a **♻** is added to the front of its name (turn this off
with `EnableRename = false`).

# Configurable Values
```ini
[Breeding]

## This prefab is consumed as a cost to breed horses.
# Setting type: Int32
# Default value: -570287766
BreedingRequiredItem = -570287766

## This is the name of the required item that will be consumed.
# Setting type: String
# Default value: special fish (twilight snapper)
BreedingCostItemName = special fish (twilight snapper)

## This is the amount of the required item consumed.
# Setting type: Int32
# Default value: 1
BreedingCostAmount = 1

## This is the half range +/- this value for applied for mutation.
# Setting type: Single
# Default value: 0.05
MutationRange = 0.05

## The absolute maximum speed for horses including selective breeding and mutations.
# Setting type: Single
# Default value: 14
MaxSpeed = 14

## The absolute maximum rotation for horses including selective breeding and mutations.
# Setting type: Single
# Default value: 16
MaxRotation = 16

## The absolute maximum acceleration for horses including selective breeding and mutations.
# Setting type: Single
# Default value: 9
MaxAcceleration = 9

[Server]

## Horses must be within this distance from well. (5 =1 tile)
# Setting type: Single
# Default value: 5
DistanceRequired = 5

## How many seconds added per drink tick (~1.5seconds), default values would be about 24 minutes for the default max amount at fountain.
# Setting type: Int32
# Default value: 30
SecondsDrinkPerTick = 30

## Time in seconds, default value is roughly amount of time when you take wild horses.
# Setting type: Int32
# Default value: 28800
MaxDrinkAmount = 28800

## If true will rename horses in drinking range with a symbol
# Setting type: Boolean
# Default value: true
EnableRename = true

## This is a comma seperated list of prefabs to use for the well. You can choose from one of (stone, iron, bronze, small, big) or (advanced: at your own risk) you can also include an arbitrary guid hash of of a castle connected placeable.
# Setting type: String
# Default value: Stone, Large
EnabledWellPrefabs = Stone, Large
```

# Demo Video (only viewable on github)
https://user-images.githubusercontent.com/62450933/175365529-f6ade327-dbd0-4500-b840-128ac52cefe7.mp4


---

<div align="center">

## ❤️ Support this revival

This mod was broken for over two years. I spent about **7 and a half hours** digging
through logs, tracing dead game systems, and tearing out a deprecated library to get
it running again — then tested it on a live server until the horses actually drank.

**I'll do my best to keep it up to date as V Rising keeps patching.**

If it saved your server, feel free to throw any amount my way. Never expected,
always appreciated.

# 💵 Cash App — [$Fartonice1081](https://cash.app/$Fartonice1081)

<!-- Optional: drag your Cash App QR image into a GitHub issue comment,
     copy the link it gives you, and paste it below to show the QR code here.
<img src="PUT_YOUR_QR_LINK_HERE" width="220" />
-->

---

*And if you'd rather thank the people who wrote the original mod — please go support*
***[decaprime](https://github.com/decaprime)*** *and* ***[cheesasaurus](https://github.com/cheesasaurus)***.
*They earned it first.*

---

**Tested working on V Rising 1.1.13 with a dedicated server running 13 other mods.**

*Horses drink. Commands work. Breeding works.*

🐴

</div>
