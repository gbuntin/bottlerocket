DESCRIPTION
-----------

BottleRocket is an interface to the X10 FireCracker home automation
kit.  The FireCracker kit consists of a dongle-like RF transmitter
that connects to the serial port of a PC, and a receiver that plugs
into a wall socket and intercepts the signals, passing them on
through house wiring to other units that turn appliances on/off or
dim/brighten lamps.  It doesn't support any kind of 2-way
communication, unlike some of the other X10 products.

This is version 0.05b3.  It includes a different internal way of handling
commands (allowing an unlimited # of commands to be specified, until of course
you run out of memory), a library is now built with the command handling
functions, and more.

I have had reports that version 0.04c worked on:

Linux/x86
Linux/Alpha
Linux/ARM
Linux/Sparc
Digital Unix/Alpha
FreeBSD/x86
NetBSD/x86

This one should too; if you have problems, let me know.  I do know that
it's not Minix friendly, though, so don't even ask.  Also there seem to
be problems with controlling DTR/CTS lines under Irix; I'm not sure if
this is an OS or hardware issue.

Version 0.04 incorporated a bunch of code by Ashley Clark for x10-amh 
compatability, code cleanups and other features, as well as more commands
(ALL_ON, ALL_OFF, LAMPS_ON, LAMPS_OFF), security fixes and more.

Thanks to Warner Losh for portability/security info.
Thanks also to Christian Gafton for patches for cleanups/fixes for 0.04a and
for supplying RedHat 6.0 RPMs. 

Version 0.03 started the x10-amh compatability and nicer serial port usage
as well as the ability to repeat commands, and environment/command line
selectable port (for root, at least). 0.03a was a bugfix release.

Version 0.02 moved command handling into its own file so it can be linked
into other things.  0.02a and 0.02b were bug fixes.

This version has accidentally been tested while my modem was online, and
everything worked just fine.  I still wouldn't suggest it though.


FILES
-----

README   - you're looking at it.

Makefile - decided to add one of these to make things easier now that
           there is more than one file in the collection

br.c     - The command line interface; uses the br_cmd stuff (below)
           to carry out commands.

br.h     - Header file for br.c; includes macros and definitions used
           by the program.

br_cmd.c - Functions for executing X10 comands using the FireCracker
           home automation system.  Basically only one function you
           need in this: x10_br_out which takes the file descriptor
           of the serial port to which the Dynamite interface is
           connected, an address (high 4 bits correspond to the letter
           of the device address, lower 4 correspond to the numeric
           part) and a command (these are defined in br_cmd.h; valid
           commands are ON, OFF, DIM and BRIGHT).  Note that you should
           set the address to 0 if you're using DIM and BRIGHT.

br_cmd.h - The file you should include to make use of br_cmd.c.  It
           holds the numeric IDs of the commands.

br_translate.h - Translation tables used by br_cmd to build commands.
           You shouldn't have to mess with this unless you're messing
           around with innards.  Doesn't need to be #included by other
           programs to use br_cmd stuff.

br_cmd_engine.c - Command handling functions to allow a simpler interface
           to br commands, including building lists of commands to be
           executed and string to device ID parsing functions.

br_cmd_engine.h - Header file for br_cmd_engine.h.  #include this to use
           the command handling functions.


COMPILING
---------
Change to the bottlerocket directory and type "./configure", then "make".
Optionally run "make install".  configure also has some command line
options (try "./configure --help" to list them); the most useful is
probably "--with-x10port=<device>" (I recommend using
--with-x10port=/dev/firecracker and making a link from /dev/firecracker
to whatever device your firecracker is plugged into.  That way you only
have to change the link if you plug your firecracker into another port.


RUNNING
-------

Use is simple.  Just run BottleRocket with the address of the unit
you want to control and the command you want it to execute.  Note
that dim/brighten are only able to address a whole letter group
of units at a time, so only the letter portion should be specified
on the command line for these commands.

e.g.

br A6 on      -- turns on appliance set to be unit "A6"
br P dim      -- dims last selected lamp set to housecode "P"

Also, Ashley Clark has given me code for x10-amh compatability so you can
do things like:

br -c A -f 1,2,3,4,5,6 -n 7

to turn off units 1-6 and unit 7 on.  All 'on' commands will be executed
before 'off', and 'dim' comes last, so br -c A -f 1 -n 1 will turn
A1 on then off.

Note:  You generally have to be root to run this, as it requires
       serial port access.  You may wish to make br setgid to the group
       that owns the serial port ("dialers" under FreeBSD, "tty" under
       Linux), but make sure you understand what you're doing and why
       first.


THE INTERFACE
-------------

The interface is really pretty simple; just wiggle the RTS and DTR lines
on a serial port and the little box hooked up to your machine will
transmit signals to another little box that routes the signal to where
it should go (if everything works).  The module is made as a pass-thru
kind of thing, and X10 claims that you can hook up other serial devices to
the same port without a problem, but I wouldn't suggest it (at least if 
you're using any kind of hardware handshaking, or have a modem that will
drop carrier if it loses DTR... those lines are actually generally pretty
nice to have working without someone jittering bits down them)...


KNOWN BUGS (not as many as it looks)
----------

Some of the macro commands (ALL_ON, LIGHTS_OFF) may not work with all
X10 devices.  It seems that X10 has unofficially added these to the
protocol list recently, and older devices (and maybe some newer ones?)
will not recognize them.

Sometimes commands are lost; this appears to be a hardware issue rather
than a software one.  If you are doing anything important, you may wish
to issue commands multiple times to account for this.

The usage information given by br is not comprehensive; there have been
additions to the command handling code, but what is already displayed is
already rather large, and it's just not worth it to add the new stuff to
it.  You can now add housecodes in the middle of a list of devices that
you specify to br, and the last housecode listed will become the default
for the rest of that command; e.g.

br -c a -n 1,2,b1,2,c1,2 -f 1

will turn on a1, a2, b1, b2, c1 and c2 and then turn off a1.  This was
added largely so that the same parsing function can be used for when
the "users" file is added (specifying which users have access to which
devices).

Documentation may not be completely up to date.  I try to keep things at
least fairly recent, but as this code can change very quickly, it is hard
to keep everything in sync.  If you find inconsistencies, email me and I
will try to fix them.

If you find a bug not listed here, please do let me know and I will try
to get it fixed by the next release (or at least mention it here).  Note
that BottleRocket is still in very much a development stage and I do not
consider it to be "stable" yet; work is still primarily to get in the
features that I want; after these have been added, I plan to review the
code, perhaps rewrite a bit, and security audit it from the base up.
Therefore, moronic bugs can still be expected.

CONTACTING ME
-------------

If you have ideas/comments/code/bug reports, email me at tymm@acm.org.



