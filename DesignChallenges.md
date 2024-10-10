# SLVM: Scripting Language Virtual Machine

## Challenge #1: VVOST

1. Values
2. Variables
3. Operations
4. Selective Type Checking

A simple VM must be able to store, and manipulate values in variables based on operators.

A challenge in dynamic languages is ensuring that values in variables are of correct type, and
that operators being used on them **can** be used on them.

We also need to have a mechanism for foreign objects, aka the objects shared by SLVM and host program.

On another note, we would have to take care of garbage collection.

### Memory Representation Proposal

Example for Values - No Type Checking:

1. int

   `{4 bytes}`

2. double

   `{8 bytes}`

3. class (Example: {int,double}):

   `{int: 4 bytes + double: 10 bytes}`

4. reference:

   `{void *: 32 or 64 bytes}`

Example for Foreign Objects / With Type Checking:

1. int

```txt
{
    Type: { void*: 32 or 64 bytes },
    Data: { 4 bytes }
}
```

Same pattern applies for all other types.
