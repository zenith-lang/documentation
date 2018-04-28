Base Language Instructions
==========================

add
^^^

============ ===========
Stack Before Stack After
============ ===========
Addend 1
Addend 2     Sum
============ ===========

Adds two numbers together.

and
^^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise and on the two operands.

break
^^^^^

============ ===========
Stack Before Stack After
============ ===========
============ ===========

Notifies a debugger that a breakpoint has been reached.

call
^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
============ ===========

Calls the method located at the specified address.

deref
^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address      Value
============ ===========

Determines the value of the memory at a specified address.

div
^^^

============ ===========
Stack Before Stack After
============ ===========
Divisor
Dividend     Quotient
============ ===========

Divides two numbers.
This always rounds down.

divmod
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Divisor      Quotient
Dividend     Remainder
============ ===========

Divides two numbers and stores the remainder.

dup
^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
Value        Value
============ ===========

Duplicates the top value of the stack.

dupn
^^^^

=============== ===============
Stack Before    Stack After
=============== ===============
Index           Value
(Index entries) (Index entries)
Value           Value
=============== ===============

Duplicates a value the specified value from the top to the top.

ignore
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Value
============ ===========

Removes the top value from the stack.

jump
^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
============ ===========

Jumps to the specified address.

jump_eq
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is equal to zero.

jump_ge
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is greater than or equal
to zero.

jump_gt
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is greater than zero.

jump_le
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is less than or equal to
zero.

jump_lt
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is less than zero.

jump_ne
^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Address
Condition
============ ===========

Jumps to the specified address if the condition value is not equal to zero.

mod
^^^

============ ===========
Stack Before Stack After
============ ===========
Divisor
Dividend     Remainder
============ ===========

Computes the remainder of division.

mul
^^^

============== ===========
Stack Before   Stack After
============== ===========
Multiplicand 1
Multiplicand 2 Product
============== ===========

Multiplies two numbers together.

nand
^^^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise nand on the two operands.

nop
^^^

============ ===========
Stack Before Stack After
============ ===========
============ ===========

Does nothing.

nor
^^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise nor on the two operands.

not
^^^

============ ===========
Stack Before Stack After
============ ===========
Operand      Result
============ ===========

Computes a bitwise not on the operand.

or
^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise or on the two operands.

pcref
^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Address
============ ===========

Gets a reference to the top of the call stack.

pop <address>
^^^^^^^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Value
============ ===========

Removes a value from the stack and stores it in the specified address.

push <address>
^^^^^^^^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
============ ===========

Adds a value to the top of the stack from the specified address.

push_b <value>
^^^^^^^^^^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
============ ===========

Adds a value to the top of the stack.

push_i <value>
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
============ ===========

Adds a value to the top of the stack.

push_l <value>
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
============ ===========

Adds a value to the top of the stack.

push_s <value>
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
             Value
============ ===========

Adds a value to the top of the stack.

ref
^^^

============ ================
Stack Before Stack After
============ ================
             Address of value
Value        Value
============ ================

Gets a reference to the top of the stack and stores it on the top of the stack.

ret
^^^

============ ===========
Stack Before Stack After
============ ===========
============ ===========

Returns to the location that the code was running before the current method was
called.

shiftl
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Value
Amount       Result
============ ===========

Shifts a value a specified number of bits to the left.

shiftr
^^^^^^

============ ===========
Stack Before Stack After
============ ===========
Value
Amount       Result
============ ===========

Shifts a value a specified number of bits to the right.

stub
^^^^

============ ===========
Stack Before Stack After
============ ===========
============ ===========

Internally used for the bootstrap to be able to have a full compilation
environment.
Do not use this instruction from user code.

sub
^^^

============ ===========
Stack Before Stack After
============ ===========
Subtrahend
Minuend      Difference
============ ===========

Subtracts two numbers.

syscall
^^^^^^^

============ ============
Stack Before Stack After
============ ============
Identifier
Argument 1
Argument 2
...
Argument n   Return Value
============ ============

Calls a method that is implemented in the system.

xnor
^^^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise xnor on the two operands.

xor
^^^

============ ===========
Stack Before Stack After
============ ===========
Operand 1
Operand 2    Result
============ ===========

Computes a bitwise xor on the two operands.
