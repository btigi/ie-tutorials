# A Familiar Tutorial by bt_igi
NB. I know this might appear a little confusing, but its not. Honestly.

# Outline
This tutorial outlines a method for creating/using companions which move with the party between areas, are storable in the player inventory, controllable, and can be spoken to. The method should be compatible with all mods, and TC's, and everything. Please make sure if you use/release your own derivation of this work to use your own prefix (mine is 'ii' and is used in this tutorial, and the example provided for download).

# How it works I:
The main summoning spell summons an invisible creature. This creature checks to find the first empty slot, and summons the relevant creature. The creature adds itself to the .gam file, if necessary. Whenever the player moves to a new area, the companion is moved to the same area.

# How it works II:
The process works on the basis of "slots". Duplication the default BG2 system would require 1 slot (ie. it would use the creature iifam0, and posses only checks for this creature), and would allow 1 companion to be summoned at any 1 time.
To allow 2 companions to be summoned at any one time would require 2 slots (ie. checks for iifam0, and iifam1). Similarly, 3 companions at once would require 3 slots (checks for iifam0, iifam1, and iifam2). To allow the possibility of every joinable NPC having their own unique companion, would take as many slots as there are NPC's. This number can be lowered if only the 6 members of the party can have a companion at once, and the companion is killed when its owner is removed from the party, thereby freeing a slot for the owners replacement.
Slots are re-used upon the companions death (hence the 2nd block being appended to the dplayer scripts, and the minhp item).

In general, the method needs:
1 script `<manager_scriptname>`
1 script `<appending_script>`
1 creature `<manager_cre>`
1 spell `1<sum_spell>`
1 item `<minhp_item>`

Each companion needs (eg. for fam0):
1 script `<script_fam0>`
1 cre file `<cre_fam0>`
1 dialog `<companion_dialog_1>`
1 dialog `<companion_dialog_2>`
1 item `<item_fam0>`
1 spell `<summon_fam0>`

----------------------------------------------------------------------

# Making the general files:
1) Create the item, `<minhp_item>`, which is simply a minimum HP item, effect #208, (ie. set the Min HP to 1). This is needed to make sure a global gets set when the companion dies. The Die/Died triggers didnt seem to work.

2) Create the manager creature, `<manager_cre>`. This creature will be summoned whenever a companion is summoned. It should be made immortal and invisible. Set the creatures override script to `<manager_scriptname>` (we will make in a moment).

3) Create the main Companion Summoning Spell, `<sum_spell>`. This spell summons `<manager_cre>`, which sorts everything out. The spell has with 1 extended header, Target = Self. To allow the spell to be cast by anyone, create it as an innate ability. The extended header has 1 feature block, which contains:
Opcode: #67 Creature Summoning
Target: Self
Timing: Permanent
Prob: 0-100
Resource: `<manager_cre>`

4) Create the script for `<cre_manager>`, `<manager_scriptname>`. For each simultaneous companion, add the following 2 blocks, updating the "iifam0" references (ie. iifam1, iifam2...). This script is placed on `<manager_cre>`, and is therefore executed whenever `<sum_spell>` is cast. I've commented the code a little, so it should be clear how it works.
```
IF
!Exists("iifam0") // We haven't ever summoned iifam0
THEN
RESPONSE #100
ReallyForceSpellRES("<summon_fam0>",LastSummonerOf(Myself)) // get the caster of the summon spell to summon iifam0 (this sets the companions LastSummonerOf tag correctly
ActionOverride(LastSummonerOf(Myself),SetGlobal"iiMyFamIs0","LOCALS",1)) // tell the summoner which companion they have
SetGlobal("iiFam0Free","GLOBAL",0) // marker global, slot 0 is now used
DestroySelf() // get rid of myself
END

IF
Exists("iifam0") // We have summoned iifam0 before
Dead("iifam0") // iifam0 dead
Global("iiFam0Free","GLOBAL",1) // the 0 slot is free
THEN
RESPONSE #100
SetGlobal("iiFam0Free","GLOBAL",0) // marker global, slot 0 is used
ReallyForceSpellRES("<summon_fam0>",LastSummonerOf(Myself)) // get the caster of the summon spell to summon iifam0 (this sets the companions LastSummonerOf tag correctly
ActionOverride(LastSummonerOf(Myself),SetGlobal("iiMyFamIs0","LOCALS",1)) // tell the summoner which companion they have
DestroySelf() // get rid of myself
END
```

The final block in the script should be:

```
IF
True()
THEN
RESPONSE #100
DisplayStringHead(LastSummonerOf(Myself),~You are not permitted a companion at this time~)
DestroySelf()
END
```

This occurs is all the companion slots are full.

5) Create `<appending_script>`, which must be appended to dplayer2 and dplayer3.

```
IF
Global("iiMyFamIs0","LOCALS",1) // I own iifam0
!Global("iiFam0InPack","LOCALS",1) // iifam0 not in my inventory
Exists("iifam0") // iifam0 does exist
!InMyArea("iifam0") // iifam0 isnt in my area
THEN
RESPONSE #100
MoveGlobalObject("iifam0",Myself) // bring iifam0 to my location
END

IF
Global("iiMyFamIs0","LOCALS",1) // I own iifam0
Dead("iifam0") // but it's dead
THEN
RESPONSE #100
SetGlobal("iiMyFamIs0","LOCALS",0) // I no longer own iifam0
END
```

That concludes the general section, now we move onto the files specific to each companion.

----------------------------------------------------------------------

Making the companion files:
1) Create the creature to represent the companion, `<cre_fam0>`. Make sure to sets its override script to `<script_fam0>`, and its death_variables to `<cre_fam0>`.

2) Create the companions script, `<script_fam0>`.

```
IF
Global("iiFamSetupDone","LOCALS",0) // global not set
THEN
RESPONSE #100
// set up the companion by use of spells etc
// based on gender, alignment, class etc of LastSummonerOf
SetGlobal("iiFamSetupDone","LOCALS",1) // set global
END

IF
Global("iiAddedToGAM","LOCALS",0) // if global not set
THEN
RESPONSE #100
MakeGlobal() // add to gam file (allows MoveGlobal to work)
SetGlobal("iiAddedToGAM","LOCALS",1) // set global
END

IF
InParty(LastSummonerOf(Myself)) // My owner is in the party
!Allegiance(Myself,5) // I am not controllable
THEN
RESPONSE #100
ChangeEnemyAlly(Myself,5) // I should be controllable (sometimes I become neutral)
END

IF
HPLT(Myself,5) // Almost dead
THEN
RESPONSE #100
DestroyItem("<minhp_item>") // destroy the min HP item - allow my death
ApplyDamage(Myself,100,MAGIC) // kill myself
SetGlobal("iiFam0Free","GLOBAL",1) // and mark my slot as free
END
```

3) Create the item, `<item_fam0>`, to represent the companion when in the players inventory. The item should be marked as conversible, and have its dialog file set to `<companion_dialog_2>`.

4) The companion requires 2 dialog, one for when it is in the players inventory (`<companion_dialog_2>`), and one for when it is roaming freely (`<companion_dialog_1>`).

We'll start with `<companion_dialog_1>`. To ensure the companion only talks to its owner, duplicate the following block, 6 times, obviously changing the slot number (and the state name).

```
IF ~InPartySlot(LastSummonerOf(Myself),0)
!InPartySlot(LastTalkedToBy(Myself),0)~ THEN BEGIN initial_0
SAY ~You did not summon me~
IF ~~ THEN REPLY ~Okay.~ EXIT
END
```

Next is the block to enable the owner to talk to the companion:

```
IF ~OR(2)
NumTimesTalkedTo(0)
NumTimesTalkedToGT(0)~ THEN BEGIN main
SAY ~Hello!~
IF ~!InventoryFull(LastSummonerOf(Myself))~ THEN REPLY ~Come into my pack.~ GOTO put_in_pack
IF ~~ THEN REPLY ~Do you have any advice?~ EXIT
IF ~~ THEN REPLY ~Nevermind.~ EXIT
END
```

And something like this to allow the companion to enter the owners inventory.

```
IF ~~ THEN BEGIN put_in_pack
SAY ~Okay, I will go in your pack.~
IF ~~ THEN DO ~GiveItemCreate("<item_fam0>",LastSummonerOf(Myself),1,0,0) // give the companion item
ActionOverride(LastSummonerOf(Myself),SetGlobal("iiFam0InPack","LOCALS",1)) // note companion is inside - prevents MoveGlobal being carried out
DestroySelf()~ EXIT // we dont need the creature anymore
END
```

And now `<companion_dialog_2>`, for when the companion is inside the player inventory.

```
IF ~True()~ THEN BEGIN main
SAY ~Hello!~
IF ~~ THEN REPLY ~Come out of my pack.~ GOTO out_of_pack
IF ~~ THEN REPLY ~Do you have any advice?~ EXIT
IF ~~ THEN REPLY ~Nevermind.~ EXIT
END

IF ~~ THEN BEGIN out_of_pack
SAY ~Okay, I will come out of pack.~
IF ~~ THEN DO ~ReallyForceSpellRES("<summon_fam0>",Myself) //
DestroyItem("<item_fam0>")
Wait(2)
SetGlobal("iiFam0InPack","LOCALS",0)~ EXIT // we no longer carry the companion
END
```

5) Finally, create a spell to summon this particular companion, `<summon_fam0>`. This is set up identical to `<sum_spell>`, except it summons `<cre_fam0>`.