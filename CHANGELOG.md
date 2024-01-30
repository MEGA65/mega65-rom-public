# MEGA65 ROM Change Log

This change log lists user-facing changes to the MEGA65 ROM between numbered
versions. See README.md for more information about version numbering and
releases.

## Current ROM beta

The current ROM beta version is **920395**. This is the version described by
the HEAD of the "master" branch of this repo.

**Important:** Starting with ROM beta 920387, the ROM depends on a new feature of the MEGA65 core not present in the last stable core (release package v0.95). To test the latest ROM beta versions, you must use a recent [MEGA65 development core](https://builder.mega65.org/job/mega65-core/job/development/), at least `20230922.14-develo-dea350f`. Using a newer ROM with an older core will result in a broken typing experience. Newer cores continue to support older ROMs, and the latest ROM beta is intended to be fully backwards compatible with all existing software. (If it isn't, please [file a bug](https://github.com/MEGA65/mega65-rom-public/issues).)

**For Xemu users:** To use the latest ROM beta versions with Xemu, you must use the "next" version, which emulates the new core feature. If you'd prefer to stay with Xemu "stable," please use ROM 920386 or earlier for now.

Changes since release 0.95 (920377):

* 920395
  * Fix: Run/Stop was not being detected correctly under some circumstances.

* 920394
  * Change: ROM detects whether it is running on a core or Xemu version that does not support the hardware typing queue, and halts boot with a message. This is a short-term convenience for v0.96 release testers and upgraders that might accidentally run the new ROM with an old core/Xemu version, and will be removed after v0.96 is released to reclaim code space. (This is not a long-term solution for forward compatibility testing; see discussion on [this issue in the private repo](https://github.com/MEGA65/mega65-rom/issues/65).)

* 920393
  * Fix: Previous change broke Run/Stop interrupting a BASIC program, now fixed.

* 920392
  * Fix: BASIC joystick handling now works correctly with the R5 mainboard.

* 920391
  * DOS and disk command improvements:
    * Fix: Reset `DS`/`DS$` disk status after initial autoboot.
    * Fix: DOS correctly writes the final byte (C64 and MEGA65 modes).
    * Fix: Report correct error for `LOAD` with inappropriate device numbers.
    * Fix: Repair DOS command channel bugs interfering with U1 and U2 commands.
    * Fix: Repair disk full detection.
    * Fix: Repair disk commands that start with a slash.
    * Improvements to `CHDIR`, `MKDIR`, `DISK`
    * Improvements to `MONITOR` `@` disk command handling
  * Fix: `RMOUSE` reports mouse movement more precisely.
  * Fix: `DOT` was corrupting graphics subsystem internal state that interfered with other commands.
  * Fix: `TI` timer accurate for NTSC, was previously hard-coded for PAL.
  * Fix: `SLEEP` edge case caused hanging, noticeable on very long delays.
  * Fix: ioinit flushes the hardware keyboard buffer, fewer spurious key events after state transitions.
  * Fix: Improvements to restoring the CPU speed after disk operations.

* 920390
  * MONITOR (BSMON) improvements:
    * New: Support for assembling and disassembling Q instructions.
    * New: Support for assembling and disassembling MAP and EOM instructions.
    * New: Load/Save/Verify support upper memory addresses.
    * New: Assemble supports editing previously entered lines.
    * New: Assemble can assemble to upper memory addresses.
    * Fix: M with range < 16 bytes shows 16 bytes.
    * Fix: Register handling in Verify.
  * Fix: BSAVE handles internal address variables correctly.
  * Fix: Disk status special variables `DS` and `DS$` retain their values until next disk operation.
  * Fix: GO64 mode preserves ZP memory `$B0-$B3` during disk operations. This fixes a regression introduced in ROM 920379 that was interfering with some C64 software that assumed these addresses were safe to use.
  * Many thanks to Wayne (johnwayner) for contributing most of these improvements!

* 920389
  * Fix: Keyboard scanner was broken in KEY OFF mode, affected MegaAssembler and Coffeebreak Compiler

* 920388
  * Fix: Keyboard scanner issues with Ctrl and Function keys
  * Fix: TI$ detecting board revision incorrectly

* 920387 — REQUIRES [THE LATEST DEVELOPMENT CORE](https://builder.mega65.org/job/mega65-core/job/development/), at least `20230922.14-develo-dea350f`
  * Change: An overhaul of the keyboard scanner to make typing faster and more accurate. This collaborates with a new core feature to avoid dropped keystrokes.
  * Older ROMs will work with the latest core, using the legacy keyboard scanner. This new ROM requires the latest core. If you run this ROM with an earlier core, typing will not work.

* 920386
  * Change: Restores original screen code-based cursor rendering. A recent ROM beta release updated cursor rendering to use the reverse character attribute instead of the traditional screen code method. Unfortunately, the original VIC-III character attributes are broken when used in combination, so upper palette characters were rendering the cursor incorrectly. There's a core update to fix character attributes, but for backwards compatibility with the C65 prototype ROM this must be requested by setting a register, and the ROM must leave this turned off by default. It's just easier to go back to the screen code cursor.
  * Fix: Issue with reading from color memory special array C@&(x,y)
  * Fix: MID$() could not assign to last character in string
  * Fix: Incorrect TOD50 setting in NTSC mode

* 920385
  * Change: Implements the [MEGA65 cartridge protocol proposal](https://mega65.atlassian.net/wiki/spaces/MEGA65/pages/36962324/MEGA65+Style+Cartridge+Work+in+Progress). This should be considered experimental for now just in case the proposal needs to be revised. The full implementation of auto-boot MEGA65 cartridges requires a newer core.
  * Change: Run/Stop-Restore resets the font to the PETSCII default (font "C"), to make text readable when resetting from programs that replace the font with graphics characters
  * Fix: Adding a string and a number is a Type Mismatch Error
  * Fix: RENUMBER supports COLLISION statements
  * Fix: RENUMBER fixes for whitespace handling around THEN and ELSE
  * Fix: (Requires new core) 80x50 mode no longer switches to the wrong screen memory when CPU speed changes. This manifested as half a screen of garbage and instability after DOS operations and the ringing of the bell (Ctrl-G) while 80x50 mode was active.
  * Fix: Repaired instability caused by calling PLAY without arguments within a program
  * Fix: Monitor's Memory and Disassemble commands support full 32-bit ranges consistently

* 920384
  * Change: The MEGA65 boot state does not bank in the C64 ROM via the $0001 register by default. The boot state configuration of $0001 is now %101 instead of %111. GO64 re-banks the C64 ROM explicitly instead of relying on the boot state.
  * Change: GO64 mode displays a proprietary banner to differentiate between GO64 mode and the C64 core, and to reinforce expectations around software compatibility and availability of MEGA65 features.
  * Fix: PEN throws an error (and does not hang) if no graphics screen is active
  * Fix: SAVEIFF uses correct IFF-ILBM header values to be compatible with modern image tools
  * Fix: CHAR correctly supports all PETSCII printable characters
  * Fix: RENUMBER supports a line number range without an end (`RENUMBER 1000,10,2500-`)
  * Fix: FOR accepts an initializer expression containing an array element (`FOR X=AR(0) TO 20`)
  * Some internal BASIC routines have been relocated to make more room for bug fixes above a jump table.
  * An unused region of the ROM space is now filled with $FF instead of a copy of other ROM code, to clarify intent and to avoid an accidental dependency on the copy.

* 920383
  * Repair for RMOUSE bug introduced in 920382
  * Rollback of "D64 disk image support" introduced in 920381 that was causing
    floppy drive read errors in rare cases

* 920382
  * BASIC 80x50 mode: pressing ESC then 5 (or printing CHR$(27) + "5") switches
    the BASIC text display to 80x50, complementing 80x25 (ESC+8) and 40x25
    (ESC+4) text modes

* 920381
  * D64 disk image support
  * Initialize BASIC sprites: MOUSE ON immediately shows an arrow mouse cursor,
    without having to install a sprite first; sprites 1-7 are a simple pattern

* 920380
  * Run Stop + Restore during an ML routine breaks to the monitor with accurate
    registers
  * SET DISK &lt;old&gt; TO &lt;new&gt; swaps unit numbers of disk drives
  * Code clean-up for kernel routines

* 920379
  * Refactor CBDOS variable area

* 920378
  * CHARDEF supports all 512 screen codes (uppercase and lowercase)
  * Improvements to MONITOR handling of PC on entry
  * DSAVE explicitly rejects U12 device (not yet supported)
  * INFO command shows PAL/NTSC video mode
  * Improvements to SYS, LET

## Release 0.95: ROM 920377

Release 0.95 is the [current stable release
package](https://files.mega65.org?id=a0276005-e71c-4b2d-8d17-2aa92e492c50),
dated October 2022. It includes ROM **920377** and core `20221012.18,93d55f0`.

Changes since release 0.9 (920287):

* 920377: Change BASIC startup message
* 920376: Fix GET bug
* 920369: Fix DIM not executed issue
* 920368: Fix no error report if trying to add string and numeric constant
* 920367: RESET button triggers cold start, LIST uses blank character after line number
* 920364: Implement VSYNC
* 920363: Fix COPY bug
* 920362: Add @dddaaannn's cursor save/restore
* 920361: Allow the usage of the "save-and-replace" character `@` in EDIT
* 920360: Fix FN bug
* 920359: Improve parameter check for FONT command
* 920357: PEEKW -> WPEEK
* 920356: Fix POKEW -> WPOKE
* 920355: Fix IMPORT issue
* 920354: Map byte arrays to screen memory
* 920353: Fixed EDIT mode
* 920352: fix COPY command
* 920351: fix RAW mode flag for BLOAD
* 920350: Debounce RTC readout
* 920349: Fix INT()
* 920348: PLAY & SOUND (improvements); BASIC-initiated sounds play in the background, do not block
* 920347: CHARDEF; Allowed for PLAY command to use SID1+SID3 (on 6 voice channels), while SOUND command uses SID2+SID4 (on 6 additional voice channels)
* 920342: More circle/ellipse improvements (?)
* 920340: ESC X to toggle 40/80 column mode (?); Was this there before? Looks new here, but not documented
* 920329: ELLIPSE and CIRCLE; These already existed (or at least they appear in the print manual.) Dunno what changed.
* 920320: CBDOS bug fixes
* 920319: Fix U2 bug
* 920318: Fix of the bug: DS (Disk Status) was not reset to OK after reading it from the error channel.
* 920316: No changes except the version number
* 920314: Extend CBDOS pattern matching with `#` and `$`
* 920313: Fix APPEND bug in CBDOS
* 920311: Add partial support for D64 image files
* 920310: Fix SLEEP
* 920309: CBDOS optimisations
* 920308: Fix ROM/FDC/XEMU issue with DIR U12
* 920306: Suspend BASIC IRQ during SYS
* 920304: Hassle free SYS command
* 920303: Fix "Save And Replace" bug
* 920302: DOT and LINE clip negative coordinates
* 920301: LOCK & UNLOCK, fix CBDOS bugs
* 920300: New number parser speeds up BASIC65.
* 920299: Add commands BITSET and BITCLR
* 920298: Add command INFO
* 920297: Improve SIN, COS, TAN functions
* 920296: Improve COPY command for internal floppy drive and SD-card images
* 920295: Fix RENUMBER for line number 65535
* 920294: Bug fixes and optimisation of CBDOS
* 920292: CHDIR implemented.
* 920291: The functions FNA() - FNZ() are now fast functions.
* 920290: Fix MONITOR assembler to accept BEQ mnemonic. Patches now require using the older 910828 as the base instead of 911011.
* 920289: Sprites can now be used in 320 x 200 bitplane graphics with their usual storage at $600 - $7ff
* 920288: Pressing STOP at power on activates MONITOR, leaving it with X skips AUTOBOOT

## Release 0.9: ROM 920287

Release 0.9 was the first mastered release package, dated January 2022. It
included ROM **920287** and core `20220109.11,1586ad4`.

See [this history of the MEGA65 ROM](https://www.m-e-g-a.org/mega65-rom-history/) for change notes from the beginning (January 2021) to ROM 920376 (July 2022), including a few notes on the known Commodore prototype versions.
