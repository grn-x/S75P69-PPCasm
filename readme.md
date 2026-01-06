# Wii Monopoly Streets Europe (S75P69) ASM Modifications

Stuff ive been working on the past week 
Modifying Monopoly Savestates and Gamesaves 

## Example: Player 1 Max-Balance:
```asm
loc_0x0:
  cmpwi r30, 0x0
  bne- loc_0x14
  lis r12, 0x7FFF
  ori r12, r12, 0xFFFF
  stw r12, 2504(r3)

loc_0x14:
  lwz r3, 2504(r3)
```
To Overwrite 4 Instructions at `80019FB0`

#### Resulting Gecko-Code:
```
C2019FB0 00000004
2C1E0000 40820010
3D807FFF 618CFFFF
918309C8 806309C8
60000000 00000000
```

## Example: Player 2 Zero Currency:
```asm
loc_0x0:
  cmpwi r30, 0x1
  bne- loc_0x14
  lis r12, 0x0
  ori r12, r12, 0x0
  stw r12, 2504(r3)

loc_0x14:
  lwz r3, 2504(r3)
```
At same injection point:

#### Resulting Gecko-Code:
```
C2019FB0 00000004
2C1E0001 40820010
3D800000 618C0000
918309C8 806309C8
60000000 00000000
```

## Player Selection & Balance Pattern

The current player is identified by the value in `r30`.

Player indices are zero-based:

- 0 = Player 1
- 1 = Player 2
- 2 = Player 3
- 3 = Player 4

The player’s bank balance is stored at `2504(r3) (0x09C8)`.

### Player Check:

| ASM | Opcode | Description |
| --- | ------ | ----------- |
| `cmpwi r30, 0x1` | `2C1E0001` | Compare current player index with Player 2
| `bne- loc_0x14` | `40820010` | Skip Code if not the selected Player

As mentioned changing the immediate value (0x0, 0x1, 0x2, 0x3) being compared with the contents of the r30 register, selects which player is affected.

### Balance Construction

| ASM | Opcode | Description |
| --- | ------ | ----------- |
| `lis r12, 0x7FFF` | `3D807FFF` | Load upper 16 bits of balance
| `ori r12, r12, 0xFFFF` | `618CFFFF` | Load lower 16 bits of balance

This specific example leads to the construction of this 32 bit val:

```
0x7FFFFFFF (2,147,483,647)
```

### Trailling instructions:
| ASM | Opcode | Description |
| --- | ------ | ----------- |
| `stw r12, 2504(r3)` | `918309C8` | Store new balance into player bank
| `lwz r3, 2504(r3)` | `806309C8` | Load player balance (original game logic)

The first gecko code lines dictate the injection point and the number of instructions to overwrite. 
In these cases, `C2019FB0 00000004` injects custom ASM at `0x80019FB0` and replaces `4` original instructions, which must be preserved or re-executed in the injected code if required.


## Example: Player 2 +5000 & Player 3 5000

```asm
loc_0x0:
  cmpwi r30, 0x1
  bne- loc_0x18
  lwz r12, 2504(r3)
  subi r12, r12, 0x1388
  stw r12, 2504(r3)
  b loc_0x28

loc_0x18:
  cmpwi r30, 0x2
  bne- loc_0x28
  li r12, 0x1388
  stw r12, 2504(r3)

loc_0x28:
  lwz r3, 2504(r3)

```
->
```
C2019FB0 00000006
2C1E0001 40820014
818309C8 398CEC78
918309C8 48000014
2C1E0002 4082000C
39801388 918309C8
806309C8 00000000
```

This routine again modifies player bank balances based on the current player index stored in `r30`.

If the current player is Player 2 (`r30 == 0x1`), the existing balance is first loaded from memory and then reduced by `5000` using an `addi` instruction with a negative immediate value. The updated balance is written back to the player structure.

If the current player is Player 3 (`r30 == 0x2`), the balance is overwritten with a fixed value of 5000. This uses the `li` pseudo-instruction, which assembles to a single `addi` instruction because the value fits within a signed 16-bit immediate.

All other players bypass the modification logic. The final `lwz` restores one of the original overwritten instructions to preserve normal game execution.

Because all of this is executed every game cycle, and Player 2's balance gets modified relatively to itself, it is constantly changing!

### To fix this looping, the gecko-provided button conditionals could be useful: 
Sets Player 1's balance to max if a+b is pressed.
See [Additional Resources](#additional-resources)

```
28528F18 F3FF0C00 ; Gecko conditional
                  ; Execute only while A+B are pressed

C2019FB0 00000006
2C1E0000 40820020
9421FFF0 91610008
3D607FFF 616BFFFF
916309C8 81610008
38210010 806309C8
60000000 00000000

E2100000 00000000 ; End conditional
04019FB0 806309C8 ; Restore original instruction
E0000000 80008000 ; End code list
```

Instructions without PowerPC-foreign Gecko conditional

```asm
loc_0x0:
  cmpwi r30, 0x0            ; Player 1?
  bne- loc_0x24

  stwu r1, -16(r1)          ; Save stack frame
  stw  r11, 8(r1)           ; Preserve r11

  lis r11, 0x7FFF
  ori r11, r11, 0xFFFF      ; r11 = 0x7FFFFFFF
  stw r11, 2504(r3)         ; Store max balance

  lwz r11, 8(r1)            ; Restore r11
  addi r1, r1, 0x10         ; Restore stack


loc_0x24:
  lwz r3, 2504(r3)          ; Original instruction
  nop 
  .word 0x00000000
  psq_l f16, 0(r16), 0, 0
  .word 0x00000000
  .word 0x04019fb0
  lwz r3, 2504(r3)
  psq_l f0, 0(r0), 0, 0
  lwz r0, -32768(r0)
```
Hastily annotated decompiled gecko conditional code, no warranties on correctness. Some Gecko control data such as the first line: `28528F18 F3FF0C00` had to be cut out since it doesnt represent executable PowerPC instructions. 

# Tools

Dynamic Mememory analysis running monopoly on dolphin using my OWN RIPPED ISO + the dolphin memory engine  

https://github.com/aldelaro5/dolphin-memory-engine
https://github.com/TheGag96/CodeWrite


related for interfacing with the wii without a wiimote
https://github.com/rnconrad/WiimoteEmulator



## Additional Resources
- Gecko Codes S75P69 (Monopoly Streets Europe)
    - https://gamehacking.org/game/134011
      - https://wiird.gamehacking.org/forum/index.php/topic,7988.msg78095.html
          - https://wiird.gamehacking.org/forum/index.php/topic,9173.0.html

- Gecko Codes S75E69 (Monopoly Streets USA)
  - https://gamehacking.org/game/134010
    - https://wiird.gamehacking.org/forum/index.php?topic=7018.0

### PowerPC Assembly
- https://wiigeckocodes.github.io/codetypedocumentation.html
- https://zenith.nsmbu.net/wiki/Custom_Code/PowerPC_Assembly_Cheatsheet
- https://mariokartwii.com/ppc/
- https://math-atlas.sourceforge.net/devel/assembly/ppc_isa.pdf
- https://www.youtube.com/watch?v=8eOGRZrJ4CU
- https://www.youtube.com/watch?v=VmMgfENAURI&list=PL6GfYYW69Pa2L8ZuT5lGrJoC8wOWvbIQv&index=2
- https://smashboards.com/threads/assembly-guides-resources-q-a.397941/



# Intent and Scope

This repository contains original Gecko instructions, links to publicly available Gecko codes, and PowerPC assembly notes produced through independent reverse-engineering and memory analysis.

The project does not distribute game binaries, assets, or proprietary Nintendo code.
All analysis was conducted using a legally obtained copy of the game owned by the author.

All material in this repository is original and transformative in nature.
This work is shared non-commercially for educational and research purposes only.

Nintendo and related trademarks are the property of their respective owners.