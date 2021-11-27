# Stack Tokens
Stack Tokens is a mechanism used for storing Tokens (or Colored Coins) on the Bitcoin Blockchain in the Script Field.

## Basic Concept
Stack Tokens stores data in "stack bytes" stored in the `TxOut-script` field. This is accomplished by using one of the opcodes in the range `0x01-0x4b`, which pushes the next N bytes, where N is the value, to the stack, and then ending that sequence with the `OP_DROP` (0x75) opcode, to pop that value off the stack.

The result of this is up to 75 bytes of data can be used to represent the token (or tokens).

## Identification, versioning, and differentiating multiple apps
If Stack Tokens is to be used on an existing blockchain we may run into an issue where others have used the stack pushing operators. We don't want to treat those transactions as Stack Tokens, so we will use an additional 6 bytes to prevent this.

We start by preceeding our stack push opcode with two `OP_NOP` (0x61) opcodes.
Then after the stack push opcode we do the following:
- 1st byte: Is the special sequence 0x55, which indicates this is a Stack Token structure (note: nothing is done to prevent collisions beyond this sequence).
- 2nd byte: Is reserved to represent the version of Stack Tokens, the first version will always be 0x00.
- 3rd and 4th bytes: Will be used to identify the application, allowing over 65,536 different applications on the same blockchain. The version field will allow us to go beyond this count if necessary.

The hope is that the double `OP_NOP` and the 0x55 sequence can help quickly identify if a transaction represents a Stack Token, with a relatively low risk of collisions.

*Note: The initial sequence of 0x61, 0x61, 0x01-0x4b, 0x55, must be the first entries in the `TxOut-script` field to identify as a Stack Token.*

## Application Based Data Representation
As long as the initial 7 bytes are as described above, and the last byte is an `OP_DROP`, the application can store data as desired in the remaining data space.

## Example

The following is an example for if an application with Application Id of 1 simply wanted to store the UTF-8 string 'Hello World' as the data package.

| NOPS | OP_15 | Stack Token | Version | Application | Data  | Footer |
| --- | --- | --- | --- | --- | --- | --- |
| `OP_NOP` `OP_NOP` | `OP_15` | `0x55` | `0x00` | `0x0001` | Hello World | `OP_DROP` |

Resulting in the following hex version:
`0x61610f5500000148656c6c6f20576f726c6475`
