# About
This tutorial aims to show a new, flexible way to implement traps in BG2. The traps will be usable by the player, in a similar way to standard traps, and will also be easily usable by creature AI scripts. The traps will be detectable (like area traps, though unlike player traps), removable (like area traps, though unlike player traps), will respond to the Find Traps ability, and the Find Traps spell.

# Setting the Trap
Cast spell, summon invis trap cre.

# Detecting the Trap - Thieves Find Traps
Append to the dplayer scripts:

```plaintext
IF 
  Class(Myself,205)
  ModalState(0)
  Global("iiTrapsDetect","GLOBAL",1)
THEN
  RESPONSE #100
    SetGlobal("iiTrapsDetect","GLOBAL",0)
END

IF 
  Class(Myself,205)
  ModalState(2)
  Global("iiTrapsDetect","GLOBAL",0)
THEN
  RESPONSE #100
    SetGlobal("iiTrapsDetect","GLOBAL",1)
    FindTraps()
END
```

# Triggering the Trap
Trap cre script:

```plaintext
IF
  Global("iiTrapsDetect","GLOBAL",1)
THEN
  RESPONSE #100
    ReallyForceSpellReS("iidetecD",Myself)
END

IF
  Global("iiReveal","GLOBAL",1)
THEN
  RESPONSE #100
    ReallyForceSpellReS("iidetecD",Myself)
    SetGlobal("iiReveal","GLOBAL",0)
END
```

# Removing the Trap
Trap cre script:

```plaintext
IF
  Global("iiRemove","LOCALS",1)
THEN
  RESPONSE #100
    CreateVisualEffectObject("iiremovB",Myself)
    DestroySelf()
END

IF
  OR(3)
    Range([0.0.0.0.0.MALE],10)
    Range([0.0.0.0.0.FEMALE],10)
    Range([0.0.0.0.0.SUMMONED],10)
THEN
  RESPONSE #100
    ReallyForceSpellRES("spwi101",Myself)  // ~Human~
    DestroySelf()
END
```

# Summary
This tutorial has shown how to implement a new trap system in BG2. The new trap system works in a similar fashion to the standard trap system, but has added advantages (script targeting abilities and scripting actions, trap removal).