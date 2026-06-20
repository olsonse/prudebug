Prudebug Version 0.28
=====================

This version is a fork of the original code released by Steven Anderson.  Many
improvements have been added since the original code, including

  - Additional processors supported
  - Additional instruction codes supported
  - Commands to inspect, manipulate more registers
  - Commands to inspect constants table (see documentation for PRU)
  - Improvements to the user interface
  - Addition of Debian packaging
  - etc.

Copyright notice of original 0.25 code:
---------------------------------------
(C) Copyright 2011, 2013 by Arctica Technologies<br>
Written by Steven Anderson

Description
-----------
Prudebug is a very small program that was initially intended to be 100-200 lines
of code to start/stop the Texas Instruments Programmable Realtime Unit (PRU) and
load a binary in the PRU.  It has evolved over time into a fairly functional
commandline debugger for the PRU.  While functional, this is a very low-level
debugger.  There is currently no connection to lines in a high level language.
Instructions are visible, but only presented as disassembled assembly language
instructions.  There is no attempt to make any connection to the register values
and any variables that may have been described in any high level programming
language.  Nevertheless, prudebug is valuable in the development of hard
realtime programs meant to run on a PRU.  One can inspect and also manipulate
the data, instructions, and registers in all of the PRU resources in order to
debug a realtime program.

Prudebug has slowly improved over time and all of the features are not
necessarily tested for every processor.  Although TI seems to use a consistent
memory layout for the PRU across different SoCs that should allow consistent
interaction with PRU cores across several different SoCs, one should approach
this use with caution.

USE AT YOUR OWN RISK.


RELEASE NOTES for prudebug
---------------------------------------------------------------------
Version 0.28

  - Improvements:
    - Commandline improvements using the readline library
    - Support of additional SoCs
    - Support for more instructions
    - Support for realtime guaranteed hardware breakpoints

Version 0.24

  - Improvements:
	- Added support for UIO PRUSS driver
	- Moved to dynamic processor selection - user can pick a processor on the command line
	- Fixed watchpoints and breakpoints to support different values on different PRUs

Version 0.25

  - Bug fixes provided by Shoji Suzuki
    - Correction to the QBA instruction decode
    - Fix backspace code for terminals using 0x7f
    - Corrected issue with writing numbers greater than 0x7fffffff to PRU memory with the wr command


CONTRIBUTORS
---------------------------------------------------------------------
  - Spencer Olson - bug fixes and lots of improvements
  - Giulio Moro - bug fixes and lots of improvements
  - Dhruva Gole - documentation improvements and register updates for AM57x[12]
  - Christian Joly - bug fixes, and modifications to make prudebug work with PRUSSv2.
  - Shoji Suzuki - bug fixes for v0.25


INSTALLATION
---------------------------------------------------------------------
To build just run make in the source code directory (make sure you have the correct cross-compiler in place and in the path -
arm-none-linux-gnueabi-gcc). You may also need to run
```sh
sudo apt-get install libreadline-dev
```
to install the readline library. The binary is called prudebug.


USAGE
---------------------------------------------------------------------
```
Usage: prudebug [-a pruss-address] [-u] [-m] [-p processor]
    -a - pruss-address is the memory address of the PRU in ARM memory space
    -u - force the use of UIO to map PRU memory space
    -m - force the use of /dev/mem to map PRU memory space
    if neither the -u or -m options are used then it will try the UIO first
    -p - select processor to use (sets the PRU memory locations)
        AM1707 - AM1707
        AM335X - AM335x
        AM57X1 - AM57x1
        AM57X2 - AM57x2
```

Generally the -a option should not be used.  If it is used, then prudebug will use the -a address for the PRU base with
the selected processor as the various PRU subsystem offsets.  -u and -m control the way the PRU base address is mapped for
program access (either the /dev/mem or /dev/uio* device).  If -u or -m are selected then it will only used the selected
method or fail.  If neither the -u or -m are selected then prudebug will try to use the UIO device driver, and if that fails
then it will use /dev/mem.  The -p option allows you to select the processor.  If your processor is not listed then determine
if one of the listed processors has compatible PRU (same base address and PRU subsystem offsets).  If not, you'll need to
modify prudbg.c and prudbg.h (see remarks near the beginning of prudbg.c).  If you do add to the list of processors, please
send me the diff so I can add it into future releases.


### COMMAND HELP
The command line takes the command 'help' to provide a detailed help, and 'hb'
for a brief help.  The output of both commands is given below.

```sh
PRU0> hb
Command help

    BR [breakpoint_number [address [s]]] - View or set an instruction breakpoint, "s" makes it a software breakpoint
    D <address> [length] - Raw dump of PRU data memory (byte offset from beginning of full PRU memory block - all PRUs)
    DD memory_location_wa [length] - Dump data memory (32-bit word offset from beginning of PRU data memory)
    DD <address> [length] - Dump data memory (byte offset from beginning of PRU data memory)
    DI <address> [length] - Dump instruction memory (byte offset from beginning of PRU instruction memory)
    DIS <32bit-address> [length] - Disassemble instruction memory (32-bit word offset from beginning of PRU instruction memory)
    G - Start processor execution of instructions (at current IP)
    GSS - Start processor execution using automatic single stepping - this allows running a program with breakpoints
    TRACE [<stop_on_halt> [<filename>]] - Start processor execution while sampling its program counter]
    HALT - Halt the processor
    L <32bit-address> file_name - Load program file into instruction memory
    PRU pru_number - Set the active PRU where pru_number ranges from 0 to 1
    Q - Quit the debugger and return to shell prompt.
    R - Display the current PRU registers.
    RESET - Reset the current PRU
    SS - Single step the current instruction.
    WA [watch_num [address [ (len | : value0 [value1 ...]) ]]] - Clear or set a watch point
    WR <address> value1 [value2 [value3 ...]] - Write a byte value to a raw (offset from beginning of full PRU memory block)
    WRD <address> value1 [value2 [value3 ...]] - Write a byte value to PRU data memory for current PRU
    WRI <address> value1 [value2 [value3 ...]] - Write a byte value to PRU instruction memory for current PRU

```

```sh
PRU0> help
Command help

General hints:
    - Commands are case insensitive
    - Address and numeric values can be dec (ex 12), hex (ex 0xC), octal
      (ex 014), or binary (ex 0b101010)
    - Memory addresses are byte addressed.
    - Pressing 'Enter' without a command will rerun a previouscommand.  For the
      `d`, `dd`, or `di` commands subsequent iterations will display the next
      block

BR [breakpoint_number [address [s]]]
    View or set an instruction breakpoint
     - 'b' by itself will display current breakpoints
     - breakpoint_number is the breakpoint reference and ranges from 0 to 9
     - address is the instruction word address that the processor should stop
       at (instruction is not executed)
     - "s" forces it to be a software breakpoint that requires single-stepping
       and may break real-time performance
     - if no address is provided, then the breakpoint is cleared

CYCLE [clear | off | on ]
    Display, clear, disable, or enable the cycle count register.

D <address> [length]
    Raw dump of PRU data memory (byte offset from beginning of full PRU memory
    block - all PRUs)

DD <address> [length]
    Dump data memory (byte offset from beginning of PRU data memory)

DI <address> [length]
    Dump instruction memory (byte offset from beginning of PRU instruction
    memory)

DIS <32bit-address> [length]
    Disassemble instruction memory (32-bit word offset from beginning of PRU
    instruction memory)

G
    Start processor execution of instructions (at current IP)

GSS [<count>]
    Start processor execution using automatic single stepping - this allows
    running a program with breakpoints.  If the optional <count> parameter is
    given, only <count> steps will be made.  If <count> is either not specified
    or given as '0', stepping will continue until otherwise interrupted.

TRACE [<k_elements>] [<stop_on_halt> [<filename>]]
    Start processor execution while sampling its program counter]
    - <k_elements> how many thousand elements to store (defaults to 1)
    - if <stop_on_halt> is true, it will stop automatically when a HALT
      instruction is encountered. This will result in lower sampling rate.
    - if <filename> is passed, the trace will be written to it instead of stdout

HALT
    Halt the processor

L <32bit-address> file_name
    Load program file into instruction memory at 32-bit word address provided
    (offset from beginning of instruction memory

J address
    Move the program counter to the specified address (absolute or relative).
    If <address> is not provided, jumps to +1

PRU <pru_number>
    Set the active PRU where pru_number ranges from 0 to 1
    Some debugger commands do action on active PRU (such as halt and reset)

Q
    Quit the debugger and return to shell prompt.

R
    Display current PRU registers.

Rx [value]
    Display or modify register value, e.g.:
    R2 // prints R2
    R2 0x01234 // set R2 to 0x1234

C
    Display current PRU constant table.

Cx [value]
    Display value from the constant table, e.g.:
    C2 // prints C2

RESET
    Reset the current PRU

SS [n_steps]
    Single step the current instruction.

WA [watch_num [<address> [ (len | : value0 [value1 ...]) ]]]
    Clear or set a watch point
    For the `WA` command, the <address> may also utilize aliases to certain
    registers:
     * rN    -- address of Nth register (N is in range 0..31)
     * cycle -- address of CYCLE count register, which, if enabled, provides a
       count of PRU cycles that go by for each instruction executed
    The various formats have the following effects
     - format 1:  wa -- print watch point list
     - format 2:  wa watch_num -- clear watch point watch_num
     - format 3:  wa watch_num address -- set a watch point (watch_num) so any
       change at that byte address in data memory will be printed during program
       execution with gss command
     - format 4:  wa watch_num address len -- set a watch point (watch_num) so
       any change at that byte address for <len> bytes in data memory will be
       printed during program execution with gss command
     - format 5:  wa watch_num address : value0 value1 ... -- set a watch point
       (watch_num) so that the program (run with gss) will be halted when the
       memory span at that location location equals the values specified
     NOTE: for watchpoints to work, you must use gss command to run the program

WR <address> value1 [value2 [value3 ...]]
    Write a byte value to a raw (offset from beginning of full PRU memory
    block--all PRUs)
    <address> is a byte address from the beginning of the PRU subsystem memory
    block

WRD <address> value1 [value2 [value3 ...]]
    Write a byte value to PRU data memory (byte offset from beginning of PRU
    data memory)

WRI <address> value1 [value2 [value3 ...]]
    Write a byte value to PRU instruction memory (byte offset from beginning of
    PRU instruction memory)

A brief version of help is available with the command hb

```
