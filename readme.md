# Wii Monopoly Streets Europe (S75P69) ASM Modifications

Stuff ive been working on the past week 
Modifying Monopoly Savestates and Gamesaves 

## Player 1 Max-Balance:
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
Resulting Gecko-Code:

```
C2019FB0 00000004
2C1E0000 40820010
3D807FFF 618CFFFF
918309C8 806309C8
60000000 00000000
```

## Player 2 Zero:
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
Same Insertion-Adress
Resulting in following Gecko-Codes:

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