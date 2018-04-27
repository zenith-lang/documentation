Execution Environment
=====================

Page Loading
^^^^^^^^^^^^

Before the program can be started, all of the pages need to be loaded into
memory.
Each page is loaded separately, so there is no guarantee that the pages will be
contiguous, but the content within a single page will always be stored
contiguously in memory.
However, non-code pages will always be loaded contiguously with the first code
page so that the addresses can be found by reference.

Address Relocation
^^^^^^^^^^^^^^^^^^

After all pages are allocated and loaded, any addresses that need to be
relocated are replaced with the corrected addresses for the differing page
locations.
In addition, any addresses marked as relocatable that have a special symbol in
the symbol table pointing to the address of the relocation information will be
relocated to the address of that special variable.

Value Stack
^^^^^^^^^^^

A value stack will always be allocated for storing arguments to functions and
local variables.
Each element in this stack will always take up eight bytes, even if the data
of the element does not use the entire space.
Any unused space in a single element of the stack will be zeroed when the
element is pushed.

Call Stack
^^^^^^^^^^

The call stack is similar to the value stack, but it stores addresses for
calling functions instead of local variables.

Starting the Program
^^^^^^^^^^^^^^^^^^^^

Once all of the program has been loaded into memory, the execution environment
will jump to the symbol '_main' and then start executing instructions.
