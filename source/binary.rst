Binary Format
=============

File Alignment
^^^^^^^^^^^^^^

The content of the file is organized into 4 KiB pages that are kept completely
separate.
Each page has a specific type of data, and will likely have lots of zeros at the
end.
The first page is always a header, and the following pages are defined based on
the header.

Header Page
^^^^^^^^^^^

The first page is always a header page.
In addition, further header pages will be allocated afterwards if the first page
is too small to contain all header information.
The first 256 bytes of the header are a signature from the compiler that
generated the file.
The first 22 bytes of any Zenith file are as follows:

.. code-block:: none

    0x000: 23 21 2F 75 73 72 2F 62  69 6E 2F 65 6E 76 20 7A  #!/usr/b in/env z
    0x010: 65 6E 69 74 68 0A                                 enith.

For a binary file, the next byte is either 0x00 or 0x01, then for the rest of
the 256 byte signature, the compiler will insert data to identify which compiler
and version was used to create the binary.

If the 23rd byte is 0x00, this is the last header page.
Otherwise, if the 23rd byte is 0x01, there is another header page that follows
this page.

Here is an example of a possibility for the first 256 bytes of a binary built by
the standard Zenith compiler, version 1.0:

.. code-block:: none

    0x000: 23 21 2F 75 73 72 2F 62  69 6E 2F 65 6E 76 20 7A  #!/usr/b in/env z
    0x010: 65 6E 69 74 68 0A 00 7A  65 6E 69 74 68 2D 62 6F  enith..z enith-bo
    0x020: 6F 74 73 74 72 61 70 20  31 2E 30 00 00 00 00 00  tstrap 1 .0......
    0x030: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0f0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x050: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x060: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x070: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x080: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x090: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0A0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0B0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0C0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0D0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0E0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........
    0x0F0: 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  ........ ........

Following that block (so 0x100 - 0xFFF) contains flags for all of the remaining
pages in the file.
Each page is represented by a two-byte structure representing the type of page
and the attributes of the page.

The attributes are defined as follows:

.. code-block:: none

    ........ ....0000    Page contains a symbol table index
    ........ ....0001    Page contains symbol table values
    ........ ....0010    Page contains a relocations list
    ........ ....1111    Page contains code
    ........ ....XXXX    Other values are reserved for future use

    ........ ...1....    The page can be executed
    ........ ..1.....    The page can be written to
    ........ .1......    The page can be written to by other processes
    ........ 1.......    The page can be read from by other processes

    .......1 ........    The page should not be read into memory while loading the process

    XXXXXXX. ........    Reserved for future use

Most programs will not require more than one header page, since each header page
can handle 1920 pages in the body of the program.

Symbol Tables
^^^^^^^^^^^^^

The symbol table contains the addresses of all symbols that were in the original
source code.
They are organized to be optimial for a binary search tree so that the names of
symbols can be looked up from an address as quickly as possible.

The runtime does not directly use this symbol table (except to find the special
symbols), but some code will rely on this symbol table being loaded into memory
for the lookup of symbols.
One example of this would be to generate stack traces from the addresses
contained on the stack.

Special Symbols
^^^^^^^^^^^^^^^
=========== ========= ==========================================================
Symbol Name Required? Purpose
=========== ========= ==========================================================
_start      Yes       Defines where the execution of the code should start.
_stack      No        Becomes a pointer to the base of the value stack.
_pc_stack   No        Becomes a pointer to the base of the call stack.
_program    No        Becomes a pointer to the first page of the program.
=========== ========= ==========================================================

Symbol Table Index Page
^^^^^^^^^^^^^^^^^^^^^^^

The symbol table contains an index that is sorted by address in ascending order
to allow for binary searching.
The page contains a series of eight-byte addresses followed by an eight-byte
offset of the start of the string that contains the name of a symbol.
Those strings should always be stored inside a page marked to be a symbol table
values page.
Unused portions of this page are zeroed.

Symbol Table Values Page
^^^^^^^^^^^^^^^^^^^^^^^^

This page contains the strings required for the symbol table, and is just the
strings ordered as close together as possible, with a null byte (0x00) at the
end of every string.

Relocations List Page
^^^^^^^^^^^^^^^^^^^^^

In order for the program still to function when it has been loaded into a random
memory offset, some values have to be relocated in order for the references to
work.
This page provides a series of eight-byte addresses that need to be relocaed
because the addresses are currently relative to the file and not to the physical
memory.
Unused portions of this page are zeroed.

Code Pages
^^^^^^^^^^

These pages are loaded directly into memory after the relocations have been
processed.
The compiled bytecode is stored primarily inside these pages, and these will
account for the largest amount of pages that a binary program uses.

Instructions are put in this section with no padding, since the symbols
addresses are already defined in the symbol tables.

Stack Instructions
^^^^^^^^^^^^^^^^^^

Stack push and pop instructions are the only instructions in the Zenith language
that take arguments, so they are stored in the bytecode in a different way.
The arguments are located directly after the opcode in the binary file.

====== ================ ======================= ========================================================================
Opcode Instruction Name Argument Length (bytes) Description
====== ================ ======================= ========================================================================
01     push             8                       Stores the value at the specified address to the top of the stack.
02     push_l           8                       Stores the literal value to the top of the stack.
03     push_i           4                       Stores the literal value to the top of the stack.
04     push_s           2                       Stores the literal value to the top of the stack.
05     push_b           1                       Stores the literal value to the top of the stack.
08     pop              8                       Stores the top of the stack in the specified address.
09     ingore           0                       Removes the value at the top of the stack.
0C     dup              0                       Duplicates the top value of the stack.
0D     dupn             1                       Duplicates the value a specified depth down to the top of the stack.
====== ================ ======================= ========================================================================

Arithmetic Instructions
^^^^^^^^^^^^^^^^^^^^^^^

Arithmetic instructions all consume the top two values from the stack, and
return the value as a single value onto the top of the stack, with the exception
of the 'divmod' instruction, which returns the remainder at the top of the stack
and the quotient at the second to the top value on the stack.
All division is rounded down.

====== ================
Opcode Instruction Name
====== ================
10     add
11     sub
12     mul
13     div
14     mod
15     divmod
====== ================

Bitwise Instructions
^^^^^^^^^^^^^^^^^^^^

The following instructions all consume the top two values from the stack, and
return the value as a single value onto the top of the stack.

====== ================
Opcode Instruction Name
====== ================
20     or
21     and
22     xor
24     nor
25     nand
26     xnor
27     not
====== ================

The following instructions take the amount to shift as the second value from the
top of the stack, and the value to shift with as the top value.
They also return a single value on the top of the stack.

====== ================
Opcode Instruction Name
====== ================
28     shiftl
29     shiftr
====== ================

Branching Instructions
^^^^^^^^^^^^^^^^^^^^^^

With the exception of 'jump', all of these instructions will compare the second
value from the top of the stack to zero with the condition defined, and will
only branch if the condition is true.
All of these instructions expect the top value of the stack to be the address in
memory to jump to.

====== ================ ========================
Opcode Instruction Name Condition
====== ================ ========================
31     jump_eq          Equal
32     jump_lt          Less than
33     jump_le          Less than or equal to
34     jump_gt          Greater than
35     jump_ge          Greater than or equal to
36     jump_ne          Not equal
37     jump             Always
====== ================ ========================

Pointer Instructions
^^^^^^^^^^^^^^^^^^^^

The 'ref' instruction stores the address of the top of the stack to the new top
of the stack.
Since it adds a new value to the stack, it will actually point to the value
second from the top of the stack by the time the instruction is complete.

The 'pcref' instruction stores the address of the top of the call stack to the
top of the value stack.

The 'deref' instruction stores the value of the memory at the address given by
the top value on the stack over the top value of the stack.

====== ================
Opcode Instruction Name
====== ================
40     ref
41     pcref
42     deref
====== ================

Function Instructions
^^^^^^^^^^^^^^^^^^^^^

These functions modify the call stack to allow functions to be implemented and
used.

The 'call' instruction stores the address of the next instruction onto the top
of the call stack, then jumps to the address in the top value of the value
stack, and removes that value.

The 'ret' instruction jumps to the address on the top of the call stack, then
removes that entry.

The 'syscall' instruction calls the operating system to do a specific task.
The top value on the stack is used for a number that identifies which call to
the system should be made.

====== ================
Opcode Instruction Name
====== ================
50     call
51     ret
52     syscall
====== ================

Non-Operational Instructions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

These instructions do not actually do anything for the code.
However, the 'break' instruction can be used to notify a debugger that a
breakpoint has been reached.

====== ================
Opcode Instruction Name
====== ================
60     nop
61     break
====== ================
