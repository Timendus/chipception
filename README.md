# 8ception

Because CHIP-8 interpreters have been written in *every* programming language. Except for CHIP-8 💡😄

## TODO list

  * Test other SCHIP features
  * Key test: not released
  * Framerate test doesn't clear screen (and doesn't run as fast as it should..?)

  * Virtual displays
    * disp wait quirk not working
    * Wrapping (CLIP_QUIRK off) is broken horizontally
    * `scroll` opcodes
    * `lores` and `hires` opcodes -> what to do?
      * `hires` buffers can't be in RAM

## Idea list

  * Make an "OS" like GUI
    * Use hires, then lores programs can be shown in windows floating on a "desktop"
    * Needs memory management to allocate space for a starting program
    * Could use annotations for how much memory a program needs to run (maybe in CBF?)
    * Can also have "native" programs, that use specific opcodes to have different resolutions and stuff
  * Allow selecting interpreter settings in menu
  * Show interpreter internals in overlay on plane 2
  * Support CBF files
  