# Chipception

Because CHIP-8 interpreters have been written in *every* programming language,
except in CHIP-8 itself. Until now! 😄

## TODO list

### Compatibility / compliance

* Key test: not released
* Framerate test doesn't clear screen (and doesn't run as fast as it should..?)
* Virtual displays
  * disp wait quirk not working
  * Wrapping (CLIP_QUIRK off) is broken horizontally
  * `scroll` opcodes
  * `lores` and `hires` opcodes -> what to do?
    * `hires` buffers can't be in RAM like it is now

### Features

* Memory management:
  * indexed "array" instead of indirect pointers
  * being able to "allocate" and "free" a block
* Keypad focus management & `Alt + Tab`
* Allow selecting interpreter settings in menu
* Add a program launcher
* Show interpreter internals in overlay on plane 2?
* Support CBF files
* Can also have "native" programs, that use specific opcodes to have different resolutions or communicate with each other. Could be cool.

# Development log

## Wait but why..?

I've been thinking about writing a CHIP-8 interpreter in CHIP-8 for almost two
years or so. Yeah, you read that right. An interpreter for the CHIP-8 bytecode,
itself programmed in CHIP-8. It's just such an insane idea that I can't help
but find myself attracted to it.

To be clear: there is absolutely no benefit to just having a CHIP-8 interpreter
written in CHIP-8. It would only allow you to run the exact same programs, but
slower and probably with more unexpected bugs. Also, it's quite the engineering
challenge to make it actually work. So it's a significant investment of time
that brings absolutely no tangible benefit. Sounds like my kind of project! 🎉

However, thinking about this a bit longer, there are a few cool things we could
do with such an interpreter. One is adaptation: it could act as a mediator
between programs and interpreters that don't play nice otherwise, without
modifying the original program **or** the interpreter running it. A CHIP-8
program that relies on a version of CHIP-8 (or one with specific quirks) that
you don't support in your interpreter could be wrapped in a CHIP-8 CHIP-8
interpreter, and run just fine in your interpreter.

Another kinda cool thing that I wanted to experiment with is multithreaded
CHIP-8. A CHIP-8 interpreter written in CHIP-8 could theoretically run multiple
CHIP-8 programs in parallel on a single host interpreter. This has no real
benefit, but it would be something cool to play with. It also touches on
another itch that I have had for a while and that kinda needed scratching:
writing an operating system for CHIP-8.

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

So I drew this image of a four colour XO-CHIP program running a "desktop" GUI
with two CHIP-8 programs running in parallel to inspire myself:

![A mocked-up "CHIP-8 OS desktop running two CHIP-8 programs in
parallel"](./pictures/concept1-large.png)

Alright, enough rationalizing an irrational project and having crazy pipe dreams
about its potential, let's get to coding 😉

## Getting started

I came up with the project name "Chipception", short for "CHIP-8 Inception",
and got to work.

### So what are we up against?

Writing a CHIP-8 interpreter isn't too hard. This will be the fourth time I
write one. But writing one in a language that **is also** the target language
brings both a couple of challenges and a couple of shortcuts.

The good news is that many opcodes can just be passed on to the host interpreter
as-is. To AND two registers, we just ask the host to AND two registers. Save the
result, save the flag register, and we're done (if we're just ignoring quirks).

The bad news is that we will actually need all the registers that CHIP-8
provides us with for running the interpreter. So every opcode will need to
operate on virtual registers in memory, adding quite a lot of overhead.

Next to that, it's super hard to predict how a CHIP-8 program will use RAM. So
we can't gracefully "share" the 4K of memory space, without a crazy amount of
overhead. Our interpreter would have to keep track of which parts of RAM are
being written to and do some kind of memory management. But even then: many of
the worth while ROMs for CHIP-8 are 80+ percent of the available storage. In
those cases there's just not enough space to add the interpreter at all.

Also; there's this little thing called self-modifying code, where a CHIP-8
program modifies itself to achieve some goal. For some programs, this trick
_may_ depend on the fact that a reset of the program loads in a fresh new
version of the ROM. So this means that to do it right and be a proper
interpreter, we have to make a copy of the ROM before we start execution.

Because of these memory requirements and the CPU overhead I very soon had to
accept the fact that this was going to be an XO-CHIP project. And that it will
only interpret systems that require less memory than XO-CHIP, like CHIP-8 and
SUPER-CHIP.

### Getting things going!

The first version of this program came together in a little over one evening:
initialize virtual registers, fetch an opcode, branch correctly on the opcode,
proxy all the CHIP-8 instructions to operate on the virtual registers. I also
added in a few quirks for good measure and my CHIP-8 CHIP-8 interpreter was
running and passing most of the available tests.

The [quirks test](https://github.com/Timendus/chip8-test-suite#quirks-test) was
the hardest to get right, because I was targeting CHIP-8 and SUPERCHIP, but my
host system was using XO-CHIP quirks. So my Chipception interpreter needed to
implement versions of the required quirks, using the XO-CHIP quirks. Especially
getting the `sprite` opcode to clip properly on the edges of the screen was a
bit of a pain. It also required quite a few cycles per frame.

![Passing the quirks test](./pictures/passing-quirks.png)

So after doing quite a lot of work on this project, I arrived at this rather
unsatisfying point. I could successfully run a single CHIP-8 or SUPERCHIP
program in an XO-CHIP interpreter. Just a little bit slower. And precisely no
one could actually see the difference with **not** having done all of this extra
work.

## One is none

So to get some return on investment, I decided that I would immediately take
this project in the direction of running multiple interpreters in parallel. If
you can run multiple programs next to each other, surely that's obviously
something new. Something different.

Because all my code was already operating on virtual registers in memory, it
was actually not that hard to get it to run multiple virtual CPUs in parallel.
Making use of the fact that SUPER-CHIP `hires` is twice the CHIP-8 `lores`
resolution, I could pretty quickly get Chipception to render this:

![Running multiple interpreters in split-screen](./pictures/split-screen.gif)

Which is clearly much more cool! A single unmodified CHIP-8 interpreter that's
running four distinct ROMs at once? Now we're talking! 😄

However, these programs are chosen quite carefully. If any program tries to
clear the screen or shift it, everything totally breaks. Also, when you press a
button, all programs register the same keypress. Which is probably not very
useful behaviour.

Virtualising the display turned out to be a lot more work than doing the CPU. I
had to -- once again -- write a custom double buffered rendering system with a
custom sprite drawing routine in CHIP-8 that could draw 8xN and 16x16 sprites
as well as get all the quirks right. Then I had to implement the `clear` opcode
and all the scrolling opcodes to operate on those buffers. Just so that each
interpreter could have its own isolated display behaviour.

And then I needed to composit those discrete display buffers together to form
the full display output. Because I wanted to be able to move virtual windows
over each other, those windows also needed to occlude each other. The normal
`sprite` behaviour does not support that kind of rendering. So the composite
step also renders everything to **yet another** virtual display buffer, that is
then finally drawn to the screen.

After all of that, I decided it was time for some lighter work: I wanted to
render some colourful borders around the virtual windows, to make it look a
little bit more like my inspirational drawing. How hard can it really be to
render a few lines and squares to a virtual display buffer in an unusual memory
lay-out?

Yeah. So that took me way longer than I care to admit. I put the project down
for a couple of days, just because I was too annoyed I couldn't get the damn
routine to draw a bloody horizontal line 😂 But after taking a few breaths and
a couple of days off, I managed to produce some primitive drawing routines and
now all of a sudden 8ception could do this:

![Running multiple interpreters in windows with borders](./pictures/bordered-windows.gif)

This is still just a single CHIP-8 ROM running in Octo, with a custom colour
scheme. This is starting to go somewhere!
