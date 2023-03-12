# 8ception

Because CHIP-8 interpreters have been written in *every* programming language. Except for CHIP-8 💡😄

## TODO list

  * Implement quirks -> pass quirks test for both platforms -> Success! 🎉
    * [x] vfQuirk
    * [x] memQuirk
    * [x] disp wait
    * [x] clipping
      * [ ] 16x16 sprite clipping
    * [x] shiftQuirk
    * [x] jumpQuirk
  * Test hires and other SCHIP features

## Idea list

  * Run multiple programs at once?
  * Make a "OS" like GUI?
    * Use hires, then lores programs can be shown in windows floating on a "desktop"
    * Needs memory management to allocate space for a starting program
    * Needs annotations for how much memory a program needs to run (maybe in CBF?)
    * Can also have "native" programs, that use specific opcodes to have different resolutions and stuff
    * DOS style UI? Or Win3.1? Or Win95? Or MacOS?
  * Allow seleting interpreter settings in menu
  * Show interpreter internals in overlay on plane 2
  * Support CBF files?
  