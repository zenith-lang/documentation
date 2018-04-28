Base Language
=============

Language Succession Model
^^^^^^^^^^^^^^^^^^^^^^^^^

The main reason for creating the Zenith language is to allow languages to be
modified by the code itself.
Because of this, there are many stages to the "Zenith" language.
The chapter describes very first language that is used by default when there is
not another language specified and when the source file is not detected to be in
a binary format.

Whitespace
^^^^^^^^^^

The base language allows for any amount of whitespace to be between any tokens.
Tokens are defined to be contiguous sections of characters in the source file
of the same type.
Whitespace is considered a space, a tab, or a newline (CR, LF, or both).

Any token that starts with an alphabetic character or an underscore (a-z, A-Z,
_) will continue to be a single token until a character that is not alphanumeric
or an underscore (not a-z, A-Z, 0-9, _) is reached.

Any token that starts with a number (0-9) will continue to be a single token
until a character that is not a number (not 0-9) is reached.

Any token starting with a symbol will be considered a token by itself, and will
always only contain that one character.
All characters between the '!' and the '~' character in the ASCII character set
that have not been described previously are considered symbols.

Any characters outside of this range are not allowed to appear in the source
files, except in string literals.

File Header
^^^^^^^^^^^

All Zenith files must start with the following 22-byte header:

.. code-block:: none

    0x000: 23 21 2F 75 73 72 2F 62  69 6E 2F 65 6E 76 20 7A  #!/usr/b in/env z
    0x010: 65 6E 69 74 68 0A                                 enith.

However, on some systems, the newline character is encoded as CR-LF
(0x0D0A), so that header will be on the same line as the following line.
Therefore, it is helpful to put an additional newline character after the LF
(0x0A) in the header if the file will be opened in an editor that does not
support newlines with only an LF.

Following this header contains the instructions for the code.

Comments
^^^^^^^^

Any '#' character causes itself, with the rest of the line, to be ignored by the
compiler.

Includes
^^^^^^^^

The include directive tells the compiler to read another file and insert its
contents at the current position of the current file.
Includes can be found either in the system include directory or in the current
directory.
The current directory is always searched first, followed by the system include
directory.
If a file extension is not given and the file does not exist without an
extension, the extension '.zen' is assumed.

The syntax for an include directive is as follows:

.. code-block:: none

    !include <path/to/file>

Executing Code while Compiling
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In order to allow the succession of different languages, it has to be possible
to run code inside the compiler.
However, this code has to be before the execution directive is written in the
code.
The code will be run from only what has been compiled sequentially to this
point.
The directive takes an argument, which is the symbol that points to the code
that needs to be run inside the compiler.

The syntax for an execution directive is as follows:

.. code-block:: none

    !call <symbol_name>

Explicit Page Allocation
^^^^^^^^^^^^^^^^^^^^^^^^

Pages do not need to be manually allocated every time, since the compiler will
split pages from the source code into multiple code pages if needed.
After every page that is split, the following footer will be appended to the
page (which means the page can actually only be 4087 bytes):

.. code-block:: none

    01 XX XX XX XX XX XX XX XX 37

where X is the base address of the next page.

This is the equivalent of the following code in the base language:

.. code-block:: none

    #!/usr/bin/env zenith

    # ...
    @exec@
    # ...
    $my_method
        # ...

        # The following code is added by the compiler:
        push second_page
        jump second_page
        # Page ends, next page starts
        $second_page

        # ...

This way, the code will not be broken if a method is split across multiple
pages.

In order to define where a page starts, the '@' token is used twice, with a set
of tokens representing different flags between them.

Here are the possible tokens:

====== =========================================================================
Token  Description
====== =========================================================================
exec   The page contains code that can be executed.
write  The contents of the page can be written to.
swrite The contents of the page can be written to by other processes.
sread  The contents of the page can be read by other processes.
====== =========================================================================

If there is no page explicitly allocated at the top of the file, the first page
is implicitly allocated as:

.. code-block:: none

    @exec write@

Symbols
^^^^^^^

Symbols are references that can refer to either a method, a section of the code,
or to a variable.
At their most basic level, they are just a marker for the compiler to replace in
other sections of the code with the address that they are allocated to.

Symbols are defined as follows:

.. code-block:: none

    $my_symbol_name

When used as a variable, it almost always needs to have a literal value after
directly after it.

Integer Literals
^^^^^^^^^^^^^^^^

When using variables, it is important to have a literal directly following the
symbol so that the space is not used for another symbol or some code.

The syntax for creating an integer literal is as follows:

.. code-block:: none

    # An integer literal
    = 42

    # Common use is right after a symbol
    $my_var = 42

String Literals
^^^^^^^^^^^^^^^

String literals are used just like integer literals, but instead of a number,
these literals provide a way to include binary directly from the source file
into the compiled bytecode.
After the '<<', any token can be used, and all data between the two instances of
that token are literally copied from the source file to the compiled file.
The newline characters directly after the first token and directly before the
second token are omitted from the copy, but any other newline characters will be
copied without modification.

.. code-block:: none

    # A string literal
    << end_of_string_token
    Hello, world!
    This is on a separate line.
    end_of_string_token

    # Common use is right after a symbol
    $my_message = << "
    Hello, world!
    "

Non-Parameterized Instructions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To use an instruction that does not contain any parameters, the name of the
instruction is inserted in the code as its own token.

Here are some examples:

.. code-block:: none

    dup
    call
    add
    ignore

Parameterized Instructions
^^^^^^^^^^^^^^^^^^^^^^^^^^

Parameterized instructions are just like non-parameterized instructions, but the
token immediately following them must be the parameter.
It must either be a number or a token that references a symbol that is defined
somewhere else in the program.

Here are some examples:

.. code-block:: none

    push_i 0
    push_l 0
    pop my_var

Example Program
^^^^^^^^^^^^^^^

Here is an example of one way to write a "Hello, world!" program:

.. code-block:: none

    #!/usr/bin/env zenith

    @@                      # Start a new page

    $ message << "
    Hello, world!

    "                       # Define a symbol called "message", then declare it
                            # to contain the string "Hello, world!" followed by
                            # a newline character.

    $ message_end           # Define a symbol that points to the byte directly
                            # after the message string.

    @exec@                  # Start a new page for the code.

    $ exit                  # Define a method that exits the program.
        push_i 1            # Syscall #1 is exit.
        syscall             # Run the syscall.
                            # A 'ret' instruction is not needed since execution
                            # will never reach here.
    
    $ write                 # Define a method that writes to the console.
                            # Already on the stack should be the length, and the
                            # message.
        push_i 1            # File #1 is STDOUT.
        push_i 4            # Syscall #4 is write.
        syscall             # Run the syscall.
        ignore              # The syscall will return the number of bytes
                            # written, but that number needs to be ignored.
        ret                 # Return to the previous method.
    
    $ _main                 # Execution will start here.
        push message        # First, the length of the message needs to be
        push message_end    # calculated.  To do this, the address of the end of
        sub                 # the message will be subtracted from the address of
                            # the start of the message.
        push message        # Add the message as the other argument to write.
                            # The length is still on the stack, so it will be
                            # the other argument.
        push write          # Push the address that needs to be called
        call                # Call that function.
        push exit           # Push the address of the exit function
        call                # Call the exit function and exit the program.
