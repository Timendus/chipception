# 8ception

Because CHIP-8 interpreters have been written in *every* programming language. Except for CHIP-8 💡😄

## TODO list

  * 16x16 sprite clipping
  * Test hires and other SCHIP features
  * Key test not halting
  * Framerate test doesn't clear screen (and doesn't run as fast as it should..?)

  * Interpreter multiplexing
    * `clear` opcode clears the whole screen
    * `scroll` opcodes shift the whole screen
    * `lores` and `hires` opcodes mess with rendering
    * `key` opcode halts all interpreters

## Idea list

  * Run multiple programs in parallel
  * Make an "OS" like GUI
    * Use hires, then lores programs can be shown in windows floating on a "desktop"
    * Needs memory management to allocate space for a starting program
    * Needs annotations for how much memory a program needs to run (maybe in CBF?)
    * Can also have "native" programs, that use specific opcodes to have different resolutions and stuff
  * Allow selecting interpreter settings in menu
  * Show interpreter internals in overlay on plane 2
  * Support CBF files
  