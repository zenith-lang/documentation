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
    ........ ....0010    Page contains code
    ........ ....XXXX    Other values are reserved for future use

    ........ ...1....    The page can be executed
    ........ ..1.....    The page can be written to
    ........ .1......    The page can be written to by other processes
    ........ 1.......    The page can be read from by other processes

    .......1 ........    The page should not be read into memory while loading the process

    XXXXXXX. ........    Reserved for future use

Most programs will not require more than one header page, since each header page
can handle 1920 pages in the body of the program.

Symbol Table
^^^^^^^^^^^^

The symbol table contains the addresses of all symbols that were in the original
source code.
They are organized to be optimial for a binary search tree so that the names of
symbols can be looked up from an address as quickly as possible.

The runtime does not directly use this symbol table, but some code will rely on
this symbol table being loaded into memory for the lookup of symbols.
One example of this would be to generate stack traces from the addresses
contained on the stack.

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
