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
  
# Development log

## Wait but why..?

I've been thinking about writing a CHIP-8 interpreter in CHIP-8 for a year or
so. It's just such an insane idea that I can't help but find myself attracted to
it.

To be clear: there is absolutely no benefit to just having a CHIP-8 interpreter
written in CHIP-8. It would only allow you to run the exact same programs, but
slower and probably with more unexpected bugs. Also, it's quite the engineering
challenge to make it actually work. So it's a significant investment of time
that brings absolutely no tangible benefit. Sounds like my kind of project! 🎉

However, thinking about this a bit longer, there are a few cool things we could
do with such an interpreter. One is adaptation: a CHIP-8 program that relies on
a version of CHIP-8 (or one with specific quirks) that you don't support in your
interpreter could be wrapped in my CHIP-8 CHIP-8 interpreter, and run just fine
in your interpreter. It could act as a mediator between programs and
interpreters that don't play nice otherwise, without modifying the original
program.

Another kinda cool thing that I wanted to experiment with is multithreaded
CHIP-8. A CHIP-8 interpreter written in CHIP-8 could theoretically run multiple
CHIP-8 programs in parallel on a single host interpreter. This has no real
benefit, but it would be something cool to play with. It also touches on another
itch that I had for a while and that kinda needed scratching: writing an
operating system for CHIP-8.

Last Octojam sysl made [a program that was a mock-up of a CHIP-8 PDA operating
system](https://sysl.itch.io/bim-logo-animation), that they called BIM. It makes
sense for them to model a "CHIP-8 OS" on a PDA-type device, since you can't
really "download" or "install" new programs into a running CHIP-8 interpreter.
PDAs used to come with a fixed set of "applications" like a calendar or a
calculator that were just baked into the ROM of the device, each just running
directly on the hardware and taking over full control of the CPU. However, when
I dream about a "CHIP-8 OS", I think about something that resembles the early
Unixes, that can run programs in "user mode", with a "kernel" that can do
"scheduling" or kill misbehaving programs and maybe some way for programs to
communicate with each other.

So maybe, just maybe, starting with a CHIP-8 interpreter in CHIP-8 is the first
step in the journey towards a true CHIP-8 operating system. One that runs CHIP-8
programs, that you can switch between or run in parallel. Maybe a few custom
opcodes could allow for the program to communicate with the host OS or other
programs?

I drew this image to inspire myself:

![A mocked-up "CHIP-8 OS desktop running two CHIP-8 programs in
parallel"](./pictures/concept1-large.png)

Alright, enough rationalizing an irrational project and having crazy pipe dreams
about its potential, let's get to coding 😉

## Getting started

I came up with the project name "8ception", short for "CHIP-8 Inception", and
got to work.

Writing a CHIP-8 interpreter isn't too hard. This will be the fourth time I
write one. But writing one in a language that **is also** the target language
brings both a couple of challenges and a couple of shortcuts.

The good news is that many opcodes can just be passed on to the host interpreter
as-is. To AND two registers, we just ask the host to AND two registers. Save the
result, save the flag register, and we're done.

The bad news is that we will actually need all the registers that CHIP-8
provides us with for running the interpreter. So every opcode will need to
operate on virtual registers in memory, adding quite a lot of overhead.

Also, we have no way of knowing how a CHIP-8 program uses RAM. So we can't
gracefully "share" the 4K of memory space. We could try to offer the guest
program just 2K of memory, but many programs would just not work with that.
Also: many of the worth while ROMs for CHIP-8 are 80+ percent of the available
storage. In those cases there's just not enough space to add the interpreter at
all. Also; there's this thing that's called self-modifying code, where a CHIP-8
program modifies itself to achieve some goal. For some programs, this trick
_may_ depend on the fact that a reset of the program loads in a fresh new
version of it. Because of all of this I very soon had to accept the fact that,
just for the memory space, this was going to be an XO-CHIP project. And that it
will interpret systems that require less memory than XO-CHIP, like CHIP-8 and
SUPERCHIP.

The first version of this program came together in a little over one evening:
initialize virtual registers, fetch an opcode, branch correctly on the opcode,
proxy all the CHIP-8 instructions but now operating on the virtual registers. I
also added in a few quirks for good measure and my CHIP-8 CHIP-8 interpreter was
running and passing most of the available tests.

The [quirks test](https://github.com/Timendus/chip8-test-suite#quirks-test) was
the hardest to get right, because I was targeting CHIP-8 and SUPERCHIP, but my
host system was using XO-CHIP quirks. So my 8ception interpreter needed to
implement versions of the required quirks, using the XO-CHIP quirks. Especially
getting the `sprite` opcode to clip properly on the edges of the screen was a
bit of a pain. It also required quite a few cycles per frame to be playable.

![Passing the quirks test](./pictures/passing-quirks.png)

So after doing quite a lot of work on this project, I arrived at this rather
unsatisfying point. I could successfully run a single CHIP-8 program in an
XO-CHIP interpreter. Just a little bit slower. And no one could actually see the
difference with **not** having all of this extra code.
