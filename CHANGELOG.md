# MEGA65 ROM Change Log

This change log lists user-facing changes to the MEGA65 ROM between numbered versions. See README.md for more information about version numbering and releases.

The latest stable ROM release is **ROM 920413**, in release package v0.97. It was declared stable on April 27, 2025. ROM 920413 requires the v0.97 core, or the Xemu emulator "stable" version released on April 23, 2025 or later.

## The latest ROM beta version

We release beta versions of the ROM that are newer than the latest stable release, to solicit help with testing from the community and to provide early previews of new features. Be aware that beta versions may require a newer core, and may have known issues. Please [file bugs](https://github.com/MEGA65/mega65-rom-public/issues) as you find them.

The latest ROM beta version is **ROM 920417**. Changes since release v0.97 (ROM 920413):

* 920417
  * New: TYPE now supports the U12 pseudo-device for displaying files from the SD card. Note that it still expects the file to contain PETSCII text, and does not display ASCII files correctly.
  * New: VERSIONQ KERNAL routine loads the ROM version number into the Q virtual register (A/X/Y/Z).
  * Improvement: GO64 sets the system palette to C64 colors, instead of leaving them at their current settings. This change may be noticeable to people who make regular use of GO64 mode, because the C64 palette is more nuanced than the MEGA65 default palette. The new C64 palette matches the default palette used by other entry paths into GO64 mode, so it's more consistent.
  * Improvement: Run/Stop + Restore deallocates 80x50 mode memory.
  * Fix: SAVEIFF now works correctly when saving an image with bit depth 8.
  * Fix: TI no longer accidentally relies on a CIA bug in the core.
  * Fix: Restore number parsing edge cases involving the letter E.

* 920416
  * **Important change:** The division operator was not rounding the floating point result, causing rounding errors when integerizing the quotient. E.g. `INT(LOG(64)/LOG(2))=5` (should be 6). This now rounds the same way multiplication does. I'm flagging this as "important" because it's changing the result of math operations, so existing programs may exhibit different behavior. If anyone notices a hardship, please [file a new bug](https://github.com/MEGA65/mega65-rom-public/issues).
  * New: A new KERNAL call for resetting the computer in fancy ways. A program can use this routine to load and run an arbitrary second program by loading it into BASIC memory ($2001) and calling this reset routine with a parameter. This ensures that the new program gets a full warm boot before auto-launching. The routine can reset via KERNAL warm boot or via Hypervisor (similar to pressing the reset button).
  * Improvement: Switching from 80x50 mode to 80x25 mode deallocates bank 4 region 0 (as switching to 40x25 mode already does).
  * Fix: The most recent change to the MEM system accidentally disabled deallocation of screen memory on `GRAPHIC CLR`, which caused screen memory to not get deallocated in common cases. This is fixed.

* 920415
  * New: High resolution sprite mode. The new `SPRITE SYS` command sets parameters that apply to the sprite system. The first such parameter is `R` to enable (1) or disable (0) high resolution sprite mode. In this mode, all sprites double in resolution (halve in size, without losing pixels) in both dimensions. They also use a different coordinate plane. `SPRITE SYS R1`
    * Technically, the VIC-IV allows each sprite to set its vertical hires mode individually, while horizontal hires mode applies to all sprites. This mode is intentionally simpler: all sprites are either full hires or full lowres. If you want to use other combinations, set the registers directly.
    * This works well with the recently introduced hires mouse mode. Be sure to enable hires mode before enabling the mouse. `SPRITE SYS R1 : MOUSE ON,1`
  * New: `RSPRSYS(0)` returns the current sprite hires mode setting. This function will also be an accessor for future sprite system parameters.
  * New: The `MEM` system has been upgraded with more intuitive behavior. `MEM`, `SCREEN`, and 80x50 text mode can all reserve 8KB regions of banks 4 and 5, primarily so that BASIC programs can reserve memory for their own use and still use `SCREEN` to allocate graphics screens. Previously, these commands needed to be called in a specific order. Now, `MEM`, `SCREEN`, and the rest of the system maintain their own reservations, and `SCREEN` honors all of them as they are set at the time it is called. (It's a subtle change, but trust me, the previous way was difficult to use.)
    * 80x50 text mode still forcibly reserves bank 4 region 0.
    * A future sprite mode intends to do something similar for bank 5 region 0.
  * New: `FRE(4)` and `FRE(5)` return `MEM`-style bitfields of the 8KB regions reserved in banks 4 and 5 by any of the mechanisms. A BASIC program can use this to dynamically reserve a free region with `MEM`, even after graphics screens have been allocated.
  * New: `MOUNT` (mount physical drive) and `MOUNT OFF` (mount "no disk") accept an optional `U` parameter to act on only the requested drive. The other drive is unaffected. Omitting the `U` parameter performs the action on both drives. For example: `MOUNT OFF U9`
  * New: KERNAL call `CURSOR` can enable or disable the screen editor cursor from a machine code program, similar to BASIC's `CURSOR ON` / `CURSOR OFF`.
  * Improvement: `INFO` reports bank 4 and bank 5 use accurately with the upgraded `MEM` system.
  * Fix: `RPT$()` was not allocating the return value correctly, preventing it from being stored in a variable or using two `RPT$()` calls in a single expression. This is fixed.
  * Fix: Activating 80x50 text mode when 80x50 text mode is already active no longer resets the screen memory in a weird way. This re-entry behavior was occurring when ESC 5 was typed or printed while already in 80x50 mode, or when opening then closing a graphics screen while in 80x50 text mode.

* 920414
  * New: The `HASBIT(addr, n)` function returns true (-1) if bit `n` is set at address `addr`. This is symmetric with the `SETBIT addr,n` and `CLRBIT addr,n` commands.
  * New: The `RPT$(str, n)` function returns a string consisting of `n` repetitions of the string expression `str`. It throws a String Too Long error if the result would be longer than 255 characters.
  * New: The `RDISK(n)` function returns properties of the most recent disk action. `RDISK(0)` returns the number of bytes read or written. `RDISK(1)` returns the unit number of the last disk operation (e.g. 8). This is useful if a program wants to load data files from its own disk and needs to know which drive it was loaded from.
  * New: The `LOG2(n)` returns the logarithm base 2 of a number. BASIC 65 supports `LOG(n)` natural log, `LOG10(n)` log base 10, and `LOG2(n)` log base 2. All other bases can be derived with this identity: log(n, base) = `LOG(n) / LOG(base)`
  * New: The `SYS` command now has a `SYS TO` variant with three differences: 1) arguments after the address are labeled, not positional; 2) a new "offset" argument makes more addresses accessible, and; 3) a new "interrupts" argument makes it possible to leave BASIC active while calling a machine code subroutine (such as for smooth `PLAY` music playback). These new features are complicated! See the updated User's Guide and Compendium for more information.
  * New: The `MOUSE` driver now supports the VIC-IV hires sprite modes. You must set the sprite modes before enabling the mouse. For example: `SETBIT $D076,0 : SETBIT $D054,4 : MOUSE ON,1`
  * New: The `MOUSE` command accepts a port number of "3," which tells it to detect mouse movement on either port. (See also `JOY(3)` from the v0.97 release.) In this mode, `RMOUSE` reports mouse buttons from both ports.
  * New: The `MOUNT OFF` command sets both virtual disks (default unit numbers 8 and 9) to "no disk."
  * New: As before, `MOUNT` remembers the filenames of mounted disk images, and attempts to re-mount them after you press the reset button. Previously, there was no official way to cancel this behavior. In this version, `MOUNT` (no args) and `MOUNT OFF` cancel the last-remembered disk image filenames, such that the configured default mount will occur after a reset. **Note:** We have more improvements planned to make mounting optionally "sticky" with both the `MOUNT` command and the Freezer. These behaviors may change slightly in future versions.
  * Change: `MOUNT` without arguments sets both virtual disk drives to their corresponding physical drives. There is not yet an implementation for the 1565 external drive, so this effectively un-mounts unit 9.
  * Change: `MOUNT` now uses the Hyppo v1.3 `attach()` function to mount disk images.
  * Change: `INFO` has been updated cosmetically to avoid confusion about the relationship between "used" and "free" bytes.
  * Fix: `FIND` had an edge case where it could find `/DATA/`.
  * Fix: Run/Stop-Restore no longer crashes during long-running graphics commands (e.g. `PAINT`).

## Release 0.97: ROM 920413

Release 0.97 is the [current stable release package](https://files.mega65.org?id=a0276005-e71c-4b2d-8d17-2aa92e492c50), dated April 2025. It includes ROM **920413** and core `aaf4542`.

Changes since release 0.96 (920395):

* 920413
  * New: `AND`, `OR`, `XOR`, and `NOT` operators now accept operands in the range \[-32768, 65535\], so that a program can treat the operands as unsigned 16-bit integers. Previously, only signed 16-bit integers could be operands. The operators still evaluate to a number value in the signed 16-bit integer range: `32768 AND 65535 = -32768`
  * Fix: A disk operation given a filename or filename pattern longer than the maximum 16 characters truncates the pattern to 16 characters. This resolves an issue where it was possible to create two files with the same name using an overly-long string.
  * Fix: Improvement to how the `SPEED` command parses its argument.
* 920412
  * New: `BSAVE` now has a "raw" mode (`,R` flag) that excludes the 16-bit starting address header from the file, symmetric with the `BLOAD` "raw" mode. This header is only appropriate PRG files. Use "raw" mode when writing data to a SEQ file: `BSAVE "MYFILE,S",P($1600) TO P($16FF),R`
  * New: There is a new KERNAL `SAVEFL` routine that can do "raw" mode saves from machine code programs. It is similar to `SAVE` with a new precondition that accepts a flags argument. See the updated Jump Table appendix in the Compendium.
  * Fix: `VSYNC` was accidentally accepting a value one past the NTSC range, which hangs. It now rejects the out-of-range value.
  * Fix: Numbers were printing incorrectly in some cases. This bug was noticed in cases of compounding floating point inaccuracies, e.g. `?.75+.05+.05+.05+.05+.05` should print `" 1"` but was printing `" 0"`.
  * Fix: `MOUNT` was causing the internal floppy drive head to be reset incorrectly, causing subsequent disk read errors.
  * Fix: `MOUNT` was mishandling disk image file names of length 5 (`A.D64`).
  * Fix: `MOUNT` to mount the internal floppy after a D64 disk image was previously mounted caused errors.
  * Fix: Attempting to fast-format a D64 disk image now returns an error instead of confusing behavior. Fast-formatting a D64 disk image is currently unsupported.
  * Fix: D64 support now handles an edge case involving a non-standard DOS version byte value.
* 920411
  * Fix: The BASIC 65 bitplane graphics subsystem now stores palette data for screens correctly, fixing multiple issues from a regression all the way back in 920255. Symptoms of the issue included incorrect palettes after switching between screens with `SCREEN SET`, and incorrect storage of colors >128 with 8 bitplanes. This went unnoticed because changing the palette of the visible screen sets the VIC palette correctly.
  * Fix: `CHAR` now consistently sets its default charset address correctly, fixing a regression in 920410. This was most noticeable when using `CHAR` with the default charset in Xemu, which notices and reports an illegal memory access in a pop-up message, in some cases. It may also have caused broken character glyphs on real hardware in those cases.
  * Fix: `MOUNT` command follows a procedure similar to the Freezer when mounting the internal floppy drive. (Thanks to johnwayner.)
  * Fix: DOS message 73 is just a welcome message from the VDC and is no longer printed as if it were a disk error. (Thanks to johnwayner.)
* 920410
  * **Important change:** The BASIC `INT()` function now always rounds toward negative infinity, also known as [the floor function](https://en.wikipedia.org/wiki/Floor_and_ceiling_functions). This is a change to the behavior when the argument is negative and has a fractional part: `INT(-1.5) = -2`, `INT(-1) = -1`. All versions of Commodore BASIC, and many other BASICs derived from Microsoft BASIC, define `INT()` as "floor," and other parts of BASIC depend on this definition internally. This was inadvertently changed starting at ROM 920177 to mean "round toward zero," breaking dependent behaviors. Restoring this also fixes issues with trigonometric functions.
  * New: `DIR U12` to list the contents of the SD card now accepts a pattern matching string, similar to using `DIR` with DOS devices. All wildcard characters are supported. For example: `DIR "M*",U12`
  * Improvement: `MONITOR`'s Disassembly command will display a screenful at a time by default, adjusted for the current screen size. Thanks to jwa1974 for this contribution!
  * Improvement: Run/Stop + Restore now resets the system palette.
  * Improvement: `CHAR` now accepts a 28-bit charset address argument. This was previously limited to 20-bit addresses.
    * This is mostly useful for `CHAR ...,$FF7E000`, which tells `CHAR` to use the custom charset defined by `FONT` and `CHARDEF` statements when drawing text to the graphics screen. Use `$FF7E800` for lowercase.
  * Fix: The `IMPORT` and `MERGE` commands don't work when run from within a program. These now report an error in this case, instead of corrupting program memory.
  * Fix: `LOADIFF` and `SAVEIFF` report disk errors correctly, instead of crashing.
* 920409
  * New: `JOY()` now reports up to five game controller buttons, as defined by the [5-button protocol](https://dansanderson.com/mega65/up-up-down-down/) and as implemented by several modern game controllers and adapters. The C64GS controller reports two buttons; Commotron and several others report three buttons; mouSTer and Unijoysticle report five buttons. This protocol works with both R3 and R6 mainboards.
    * While technically this changes the range of return values for `JOY()`, it only does so when a multi-button controller is connected to the port and the other buttons are pressed. (It may also report a value when paddle pairs are connected and both paddle buttons are pressed, but paddle games should use `POT()`.) Given how `JOY()` is defined, programs are unlikely to use the return value in a way where this interferes with program behavior. To be extra sure, I did an audit of Filehost: of the 31 BASIC programs that use `JOY()`, only two would respond to the upper buttons, and they would treat them as button 1.
  * Improvement: `POT()` now uses the MEGA65 paddle registers, instead of the vintage method of reading paddle values through the SID chip. The vintage method requires a slow wait for the SID value to stabilize after updating the SID connection, and `POT()` out of necessity paused execution for the full duration. The new method reads a MEGA65 hardware register and returns immediately.
    * (The new 5-button game controller support also uses the MEGA65 hardware registers to read buttons 2 and 3 over the paddle lines.)
  * Fix: TI is no longer reset by `RUN <line>`.
  * Fix: When in `CURSOR ON` mode, relocating the `CURSOR` no longer leaves stray blinked-on cursor marks. This is a partial solution for a general issue: any printing of cursor movement PETSCII codes can still have this unintended effect. The documentation has been updated to recommend calling `CURSOR OFF` before calling `PRINT`.
  * Fix: `IMPORT` now reports File Not Found correctly.
  * Fix: The `USING` directive in `PRINT` can now be preceded by other directives, such as semicolon and comma.
  * Fix: `JOY(1)` no longer reports a spurious signal when top row keys are pressed.

* 920408
  * Fix: DECBIN was also mishandling string lengths, similar to DEC.
  * Fix: KEY macro sizes were not limited correctly, very long macros were corrupting memory.
  * Fix: Text highlight register was not getting initialized correctly, was causing disk error highlight colors to stick (orange text) under specific circumstances.
  * Fix: RMOUSE with non-0 sprite was reporting large X coordinates incorrectly. (Thanks Pimau!)

* 920407
  * Fix: DEC() was mishandling string lengths, sometimes returning wrong values.
  * Fix: OPEN now clears DS when opening a disk device. This fixes an edge case in Eleven where a failure to load the last-loaded file on start-up would cause subsequent successful loads to report failure.

* 920406
  * Fix: LOAD KERNAL API in "raw" mode now returns a File Not Found error. One caveat to this fix: when loading a PRG-type file in raw mode, it must be at least two bytes long, even though the bytes are interpreted as data. Technically all PRG-type files must be at least two bytes long, so this is fairly complete, but maybe doesn't fully capture the use case in edge cases. (IMO, a file type that must allow sizes of 0 or 1 ought to be SEQ files.) I added this limitation to the docs.
  * Fix: CONT now works when the program stopped during a subroutine, either with the STOP command or with Run/Stop.
  * Fix: FOR with byte variable now throws Illegal Quantity Error on an over-large TO, instead of infinite looping.
  * Fix: FOR with byte variable can use TO targets of 0 and 255.

* 920405
  * Change: Support for the `RENUMBER100`-style syntax for "crunch" mode has been removed, in favor of `RENUMBER C 100`. If it is any consolation, `RENUMBERC100` also works. 😅 Hopefully this is different enough to prevent accidental activation.
  * Fix: `FN X()` argument can now be an array element, was previously reporting Syntax Error.
  * Fix: `FOR` index variable can now be a byte variable (`B&`). The variable truncates on assignment after `STEP` is added. Take care that the `TO` value is in byte range, to prevent an infinite loop.
  * Fix: `DLOAD` failure during Edit mode now reports `?OK ERROR`. Previously it was silent. It's not perfect—`DS` still isn't set—but it's an improvement.
  * Fix: `MOVSPR` no longer overshoots the target position.

* 920404
  * New: `PRINT USING` `%`-style patterns now support left zero padding for positive values in float and integer patters: `%07.2F` `%05D` Left zero padding is cancelled if the value is negative. Right-of-decimal fields and hexadecimal patterns are always left zero padded, as before.
  * New: `RWINDOW(3)` returns the screen height in rows, regardless of window size, similar to how RWINDOW(2) returns the screen width in columns.
  * Improvement: `RENUMBER` "crunch" mode that elides leading whitespace before line numbers now has an explicit flag: `RENUMBER C 100` I would like to remove the `RENUMBER100`-style syntax in a future version because this is a destructive action and too easy to trigger accidentally this way. This ROM version supports both syntaxes for now.
  * Improvement: When in 80x50 mode, ESC+X goes to 80x25, then toggles between 80x25 and 40x25 thereafter. Previously, this did nothing in 80x50 mode. This also applies to the (mostly useless) "SWAPPER" KERNAL call.
  * Fix: Physical floppy drive resets the drive head properly on next directory read after an error, fixing a regression from 920403. This is believed to be the cause of the oddball floppy drive issues with "D64 support" in ROM 920381, even though that specific case was not reproduceable in 920403. Wayne found a reproduceable case this time around, and we determined that the logic that "D64 support" replaced was doing a sector read that the new version wasn't doing. There is now a dummy sector read in this call path to force a reset. It should only affect the internal floppy drive, and only in this way.
  * Fix: `DS`/`DS$` disk error variables are now reset correctly for all DOS commands. `DOPEN`, `MERGE`, and `CHDIR U12` were not resetting them when called, causing confusion when a previous disk command failed and the newer command succeeded.

* 920403
  * New: D64 support. This change is identical to the one for ROM 920381, but the oddball problems of ROM 920381 + core 0.95 seem to no longer be present with ROM 920403 and core 0.96. We're excited to open this to wider testing.

* 920402
  * Fix: Regression in PLAY and SOUND from 920401.

* 920401
  * Fix: BLOAD loads correctly to Attic RAM addresses.
  * Fix: SETBNK setting persists between LOAD calls, was failing in edge cases.
  * Fix: SETBNK more thoroughly respected by BASIC.
  * Fix: Corrects BLOAD and BVERIFY address reporting in edge cases.

* 920400
  * Fix: Restore the VECTOR change. RESTOR and Run/Stop+Restore correctly use a conditional test for the FDC to choose the default serial/DOS vectors. User calls to VECTOR require a 56-byte table, and can customize the serial and editor vectors.
  * Fix: BLOAD now uses the new SETBNK implementation for both B-argument calls and 28-bit P-argument addresses, resolving a clash between the previous 28-bit implementation and SETBNK.
  * Fix: BSAVE now uses the B-argument correctly.

* 920399
  * Temporarily revert VECTOR change. The new table length isn't an issue, it was just implemented incorrectly, interfering with DOS's jump table (not yet public) after Run/Stop + Restore. Until this is fixed, VECTOR will only do the first 32 bytes of the vector table. Callers should continue to expect a 56-byte table.

* 920398
  * **Important change:** The KERNAL routine VECTOR was previously only copying the first 32 bytes of the vector table. It now copies all 56 bytes. This is a backwards incompatible change, but we believe existing MEGA65 software is not using this routine. If you believe this change is causing an issue with existing software, please [file a bug](https://github.com/MEGA65/mega65-rom-public/issues).
  * New: Four new KERNAL routines provide safe access to KERNAL state:
    * GETIO: Read the current input and output devices
    * GETLFS: Read file, device, secondary address
    * KEYLOCKS: Read/set keyboard locks
    * ADDKEY: Add a character to the soft keyboard input buffer
  * New: KERNAL routine SETBNK has been reimplemented to support 28-bit addresses for file data and filenames. The API is backwards compatible with the original 24-bit SETBNK. This routine has been disabled for multiple ROM versions. It is now the official way to set file data upper address bits.
  * New: KERNAL vector KEYSCAN gives a program an opportunity to intercept reading from the keyboard by GETIN, after a key has been read and before it has been interpreted. This replaces C65 vectors that were originally intended for a similar purpose that had to be removed for the new MEGA65 keyboard scanner.
  * Fix: Some programs, especially those compiled with llvm-mos, were crashing when screen printing scrolled the screen.
  * Fix: Run/Stop-Restore (and IOINIT) were not resetting the CPU speed correctly. It now resets to 40 MHz consistently.
  * **Note:** As a general rule, direct access to KERNAL memory by a program is not supported (for now, at least). We are working to document all supported KERNAL integration techniques, and support new tasks through KERNAL calls. If your program has a need to access KERNAL state that is not covered by the KERNAL jump table routines, please [file a feature request](https://github.com/MEGA65/mega65-rom-public/issues).

* 920397
  * New: `JOY(3)` returns the combined status of joysticks in either port, allowing for a program to easily support a single joystick in either port without calling `JOY()` twice.
  * Fix: `SPEED` no longer trips VIC-IV "hot registers."
  * Fix / workaround: `VSYNC` no longer hangs when given a value lower than 6 in NTSC mode. The root cause is [a core bug](https://github.com/MEGA65/mega65-core/issues/362) in the vertical raster position register. This change causes `VSYNC` with a value less than 6 in NTSC mode only to act as if the argument is 6. When the core bug is fixed, this workaround can be removed.
  * Fix: Extend RTC detection to notice broken amp.
  * Fix: Restore `GO65`.

* 920396
  * New: Binary literals! Use `%` for binary literal values such as `%1010`. New `DECBIN("...")` function converts a binary string of `0` and `1` characters to a number. `STRBIN$(number)` converts a number to a string of `0` and `1` characters.
  * New: `DIR P` displays a directory listing, pausing for a key press after each screenful (page). `DIR U12,P` for listing files on the SD card is also supported.
  * New: `TYPE P "filename"` displays a text file, pausing for a key press after each screenful (page).
  * New: `BLOAD` prints the starting and ending address of the loaded memory region when used in immediate mode (at the `READY` prompt).
  * New: When `RENUMBER` encounters an Unresolved Reference Error, it also prints the line where the error occurs. (For various reasons, this is not available via the `HELP` command, as with runtime errors.)
  * New: `RPALETTE()` now accepts a screen number of -1 to refer to the primary system palette (palette 0). It also accepts -2, -3, and -4 for the other three system palettes (palettes 1-3). Support for using the other system palettes from BASIC is planned for a future release.
  * New: It is now possible to disable "line pushing" in the screen editor, using the escape sequence ESC R (`PRINT CHR$(27)+"R"`). Use ESC N to re-enable. This allows the `PRINT` command to write a character to the right-most column without pushing other lines downward. Note that you must still use ESC M (`PRINT CHR$(27)+"M"`) to disable screen scrolling to `PRINT` a character in the bottom-right corner without scrolling the screen. (Use ESC L to re-enable scrolling.)
  * New: The BASIC sprite subsystem now recognizes and honors the VIC-IV high resolution sprite registers `sprenv400` (high-res vertical, per sprite) and `sprh640` (high-res horizontal, system-wide). `MOVSPR` accepts 10-bit X and Y coordinates, and will correctly truncate coordinates in low-res modes. We will be adding a mode for matching the sprite resolution to the screen resolution in a future release. It will always be supported for a BASIC program to set these registers directly. We will also add support for a hires mouse pointer in a future release ([tracking issue](https://github.com/MEGA65/mega65-rom-public/issues/126)).
  * Change: The BASIC ROM has been reorganized to make better use of available space. This introduces a new memory map mode to access a new region of ROM space that was previously unused. The BASIC 65 bitplane graphics subsystem has been relocated to this new region, and future additions may also use it.
  * Change: The boot state of register $0001 has changed to start with the datasette lines turned off. The datasette port will be supported in the upcoming expansion board.
  * Change: The v0.96 release ROM included a boot-time error message for when it used with an older core or Xemu version that didn't support the hardware typing queue, to make it easier for upgraders and testers to notice when this happens. This error message has been removed. We may reinstate this for the next release package where the ROM depends on a new feature of the core.
  * Change: The ROM version string in the ROM data is now null terminated, to support external tools reading a variable-length version string.
  * Change: The repeat delay for Mega+Shift, No Scroll, and Ctrl-S has been slowed down. This repeat delay was inadvertently sped up with the new typing system in release v0.96, causing accidental repeats.
  * Fix / Change: The `MOUSE` command was intended by Fred Bowen to accept arguments to adjust the location of the "hot spot" on the mouse pointer sprite, as well as set the pointer's initial screen position. These arugments were incorrectly documented and also were not functioning correctly. The features have been fixed, using the original intended argument order: `MOUSE ON,port,sprite,hotspot-x,hotspot-y,location-x,location-y`
  * Fix / Change: Using `MOUSE ON` when the mouse is already enabled using a different sprite number will disable the previous sprite. (There is only ever one active mouse pointer.)
  * Fix: `INFO` and `RSPEED()` detect the CPU speed correctly in an edge case.
  * Fix: Improved `CLR TI` to reset the `TI` timer variable.
  * Fix: Edge case where `VAL` reads one too many characters for signed values.
  * Fix: Edge case where typing a quote in the last position of an inserted space leaves the screen editor in quote mode.
  * Fix: DOS memory map bug.
  * Fix: Monitor requires an argument for the `J` command.
  * Fix: Monitor restores the base page B CPU register when resuming from a `brk`. B register is supported by the `jmp_far` routine (also available as a KERNAL call).
  * Fix: In v0.96, scroll pausing was interrupting regular typing. This now behaves as before: No Scroll and Ctrl-S do nothing at a blinking cursor.
  * Fix: The `R` Shift+`U` shortcut for the `RUN` command has been restored. In general, we can't support command abbreviations as an API, but we will make a best effort to not clobber commonly used ones. See [this wiki page](https://mega65.atlassian.net/wiki/spaces/MEGA65/pages/22609921/BASIC+65+Keyword+Abbreviations).
  * Fix: Run/Stop-Restore sets sprite 0's color to 1 (white) instead of 0 (black), agreeing with the boot state.
  * Note: Similar to the undocumented `MOUSE` parameters, we noticed that the `VOL` command was documented incorrectly. It accepts two arguments: a volume for the right stereo channel SIDs, and a volume for the left stereo channel SIDs (`VOL 9,9`). The ROM has always implemented it this way. We have elected to change the documentation to match the implementation (with no change to the implementation).

## Release 0.96: ROM 920395

Release 0.96 was the stable release package dated February 2024, factory installed for MEGA65s shipped starting June 2024. It includes ROM **920395** and core `20240224.00,3c10488`.

**Important:** The release v0.96 ROM requires the v0.96 core, or the Xemu emulator from January 2024 or later.

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

Release 0.95 was the factory installed release package for MEGA65s shipped in October 2022. It includes ROM **920377** and core `20221012.18,93d55f0`.

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

Release 0.9 was the first release package, dated January 2022. It
included ROM **920287** and core `20220109.11,1586ad4`. This release was factory installed with MEGA65s that shipped in May 2022.

See [this history of the MEGA65 ROM](https://www.m-e-g-a.org/mega65-rom-history/) for change notes from the beginning (January 2021) to ROM 920376 (July 2022), including a few notes on the known Commodore prototype versions.
