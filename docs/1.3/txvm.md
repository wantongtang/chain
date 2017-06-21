# Stacks

## Types

There are three types of items on the VM stacks, with the following numeric identifiers.

Int64 (33)
String String (34)
Tuple (35)

"Boolean" is not a separate type. Operations that produce booleans produce the two int64 values `0` (for false) and `1` (for true). Operations that consume booleans treat `0` as false, and all other values (including all strings and tuples) as `true`.

### Int64

An integer between 2^-63 and 2^63 - 1.

### String

A bytestring with length between 0 and 2^31 - 1 bytes.

### Tuple

An immutable collection of items of any type.

There are several named types of tuples.

#### Value

0. `type`, a string, "value"
2. `referencedata`, a tuple of strings (initially empty)
1. `history`, a string
3. `amount`, an int64
4. `assetID`, a string

#### Output

0. `type`, a string, "output"
1. `referencedata`, a tuple of strings (initially empty)
2. `history`, a 32-byte string
3. `values`, a tuple of `Value`s
4. `history`, a tuple (TBD)

#### Nonce

0. `type`, a string, "nonce"
1. `referencedata`, a tuple of strings (initially empty)
2. `program`, a string
3. `mintime`, an int64
4. `maxtime`, an int64

#### Retirement

0. `type`, a string, "retirement"
1. `referencedata`, a tuple of strings (initially empty)

#### Anchor

0. `type`, a string, "anchor"
1. `referencedata`, a tuple of strings (initially empty)
1. `history`, a string

#### Asset Definition

0. `type`, a string, "assetdefinition"
1. `referencedata`, a tuple of strings
2. `issuanceprogram`, a string

#### Transaction Header

0. `type`, an int64
1. `referencedata`, a tuple of strings (initially empty)
2. `history`, a string
3. `inputs`, a tuple of [outputs](#output)
4. `outputs`, a tuple of [outputs](#output)
5. `mintime`, an int64
6. `maxtime`, an int64

## Item IDs

The ID of an item on the stack is the SHA3 hash of the [serialization](#serialization) of that item.

## History

When a new [Output](#output), [Value](#Value), [Anchor](#Anchor), [Retirement](#Retirement), or [Transaction Header](#transaction-header)) is created by a VM instruction (including `annotate`), its `history` field is computed based on the instruction that generated it.

The `history` field is the [TupleHash](#TupleHash) of:

0. `opcode`, an int64 (reflecting the opcode that generated the item)
1. `arguments`, a tuple (reflecting the arguments consumed by that instruction, in order)
2. `outputindex`, an int64 (reflecting which output of that instruction this item is)

## Serialization

### Varint encoding

Integers are encoded in [signed LEB128](#https://en.wikipedia.org/wiki/LEB128) format. [TBD: clarify].

### Items

Stack items are serialized as a [push operation](#push-operation) that would push that item to the data stack.

Integers greater than or equal to 0 and less than or equal to 32 are encoded as the appropriate [small integer](#small-integer) opcode. Other integers are encoded using [PushInt](#PushInt) instructions. Strings are serialized using [PushString](#PushString) instructions, and tuples are serialized using [PushTuple](#PushTuple) instructions.

## Stack identifiers

0. Data stack
1. Alt stack
2. Input stack
3. Value stack
4. Output stack
5. Condition stack
6. Nonce stack
7. Anchor stack
8. Retirement stack
9. Transaction header stack


## Data stack

Items on the data stack can be int64s, strings, or tuples.

### Alt stack

Items on the alt stack have the same types as items on the data stack. The alt stack starts out empty. Items can be moved from the data stack to the alt stack with the [toaltstack](#toaltstack) instruction, and from the alt stack to the data stack with the [fromaltstack](#fromaltstack).

### Input stack

Items on the Input stack are [Outputs](#output).

### Value stack

Items on the Value stack are [Values](#value).

### Output stack

Items on the Output stack are [Outputs](#output).

### Nonce stack

Items on the Nonce stack are [Nonces](#nonce).

### Anchors stack

Items on the anchors stack are [Anchors](#anchor).

### Condition stack

Items on the Condition stack are strings, representing programs.

# Encoding formats

## Varint

TODO: Describe rules for encoding and decoding unsigned varints.

## Ed25519 Curve Points

TODO: Describe rules for Ed25519 curve point encoding (including checks that should be done when decoding.)

# Operations

## Push operations

### Small integers

Opcodes 0 through 32 are instructions that push 0 through 32, respectively, to the data stack as int64s.

### PushInt

`pushint` pushes an int64 to the stack.

Followed by a varint `x`. Pushes `x` as an int64 to the stack. Fails if `x` is less than -2^63 or greater than 2^63-1.

### PushString

`pushstring` pushes a string to the stack. 

Followed by a varint `len`, then `len` bytes. Pushes those bytes to the data stack as a [string](#string).

### PushTuple

`pushtuple` pushes a tuple to the data stack. 

`pushtuple` must be followed by a varint `len`, then followed by `len` encoded [push operations](#push-operations). `pushtuple` pushes a tuple of those items, in the same order, to the data stack.

## Control flow operations

# Fail

Halts VM execution, returning false.

# PC

Pushes the current program counter (after incrementing for this instruction) to the data stack.

# JumpIf

Pops an integer `destination`, then a boolean `cond` from the data stack. If `cond` is false, do nothing. If `cond` is true, set program counter to `destination`. Fail if `destination` is negative, if `destination` is greater than or equal to the length of the current program.

## Stack operations 

# Roll

Pops an integer `stackid` from the data stack, representing a [stack identifier](#stacks), and pops another integer `n` from the data stack. On the stack identified by `stackid`, moves the `n`th item from the top from its current position to the top of the stack.

Fails if `stackid` does not correspond to a valid stack, or if the stack has fewer than `n + 1` items.

# Bury

Pops an integer `stackid` from the data stack, representing a [stack identifier](#stacks), and pops a number `n` from the data stack. On the stack identified by `stackid`, moves the top item and inserts it at the `n`th-from-top position.

# Depth

Pops an integer `stackid` from the data stack, representing a [stack identifier](#stacks). Counts the number of items on the stack identified by `stackid`, and pushes it to the data stack.

## Data stack operations

### Equal

Pops two items `val1` and `val2` from the data stack. If they have different types, or if either is a tuple, fails execution. If they have the same type: if they are equal, pushes `true` to the stack; otherwise, pushes `false` to the stack.

### Type

Looks at the top item on the data stack. Pushes a number to the stack corresponding to that item's [type](#type).

### Len

Pops a string or tuple `val` from the data stack. If `val` is a tuple, pushes the number of fields in that tuple to the data stack. If `val` is a string, pushes the length of that string to the data stack. Fails if `val` is a number.

# Drop

Drops an item from the data stack.

# Dup
	
Pops the top item from the data stack, and pushes two copies of that item to the data stack.

# ToAlt

Pops an item from the data stack and pushes it to the alt stack.

# FromAlt

Pops an item from the alt stack and pushes it to the data stack.

## Tuple operations 

### Tuple

Pops an integer `len` from the data stack. Pops `len` items from the data stack and creates a tuple of length `len` on the data stack.

### Untuple

Pops a tuple `tuple` from the data stack. Pushes each of the fields in `tuple` to the stack in reverse order (so that the 0th item in the tuple ends up on top of the stack).

### Field

Pops an integer `stackid` from the data stack, representing a [stack identifier](#stack-identifier), and pops another integer `i` from the top of the data stack. Looks at the tuple on top of the stack identified by `stackid`, and pushes the item in its `i`th field to the top of the data stack.

Fails if the stack identified by `stackid` is empty or does not have a tuple of at least length `i + 1` on top of it, or if `i` is negative.

## Boolean operations

### Not

Pop a boolean `p` from the stack. If `p` is `true`, push `false`. If `p` is `false`, push `true`.

### And

Pops two booleans `p` and `q` from the stack. If both `p` and `q` are true, push `true`. Otherwise, push `false`.

### Or

Pops two booleans `p` and `q` from the stack. If both `p` and `q` are false, push `false`. Otherwise, push `true`.

## Math operations

### Add

Pops two integers `a` and `b` from the stack, adds them, and pushes their sum `a + b` to the stack.

### Sub

Pops two integers `a` and `b` from the stack, subtracts the second frmo the first, and pushes their difference `a - b` to the stack.

### Mul

Pops two integers `a` and `b` from the stack, multiplies them, and pushes their product `a * b` to the stack.

### Div

Pops two integers `a` and `b` from the stack, divides them truncated toward 0, and pushes their quotient `a / b` to the stack.

### Mod

Pops two integers `a` and `b` from the stack, computes their remainder `a % b`, and pushes it to the stack.

TODO: clarify behavior. Should act like Go.

### LeftShift

Pops two integers `a` and `b` from the stack, shifts `a` to the left by `b` bits.

TODO: clarify behavior. Fail if overflows Int64.

### RightShift

Pops two integers `a` and `b` from the stack, shifts `a` to the right by `b` bits.

TODO: clarify behavior. 

### GreaterThan

Pops two numbers `a` and `b` from the stack. If `a` is greater than `b`, pushes `true` to the stack. Otherwise, pushes `false`.

## String operations

### Cat

Pops two strings, `a`, then `b`, from the stack, concatenates them, and pushes the result, `a ++ b` to the stack.

### Slice

Pops two integers, `start`, then `end`, from the stack. Pops a string `str` from the stack. Pushes the string `str[start:end]` (with the first character being the one at index `start`, and the second character being the one at index `end`). Fails if `end` is less than `start`, if `start` is less than 0, or if `end` is greater thanss the length of `str`.

## Bitwise operations

### BitNot

Pops a string `a` from the data stack, inverts its bits, and pushes the result `~a`.

### BitAnd

Pops two strings `a` and `b` from the data stack. Fails if they do not have the same length. Performs a "bitwise and" operation on `a` and `b` and pushes the result `a & b`.

### BitOr

Pops two strings `a` and `b` from the data stack. Fails if they do not have the same length. Performs a "bitwise or" operation on `a` and `b` and pushes the result `a | b`.

### BitXor

Pops two strings `a` and `b` from the data stack. Fails if they do not have the same length. Performs a "bitwise xor" operation on `a` and `b` and pushes the result `a ^ b`.

## Crypto operations

### SHA256

Pops a string `a` from the data stack. Performs a Sha2-256 hash on it, and pushes the result `sha256(a)` to the data stack.

### SHA3

Pops a string `a` from the data stack. Performs a Sha3-256 hash on it, and pushes the result `sha3-256(a)` to the data stack.

### CheckSig

Pops a string `pubKey`, a string `msg`, and a string `sig` from the data stack. Performs an Ed25519 signature check with `pubKey` as the public key, `msg` as the message, and `sig` as the signature. Pushes `true` to the data stack if the signature check succeeded, and `false` otherwise.

TODO: Should we switch order of `pubKey` and `msg`?

### PointAdd

Pops two strings `a` and `b` from the data stack, decodes each of them as [Ed25519 curve points](#ed25519-curve-points), performs an elliptic curve addition `a + b`, encodes the result as a string, and pushes it to the data stack. Fails if `a` and `b` are not valid curve points.

### PointSub

Pops two strings `a` and `b` from the data stack, decodes each of them as [Ed25519 curve points](#ed25519-curve-points), performs an elliptic curve subtraction `a - b`, encodes the result as a string, and pushes it to the data stack. Fails if `a` and `b` are not valid curve points.

### PointMul

Pops an integer `i` and a string `a` from the data stack, decodes `a` as an [Ed25519 curve points](#ed25519-curve-points), performs an elliptic curve scalar multiplication `i*a`, encodes the result as a string, and pushes it to the data stack. Fails if `a` is not a valid curve point.

## Entry operations

TODO: add descriptions.

### Annotate

Pops an integer `stackid` from the data stack, representing a [stack identifier](#stack-identifier), and pops a string, `referencedatastring`. If `stackid` is not a valid stack identifier, or if it identifies the data stack, or if the stack identified by `stackid` is empty, fails. Pops the item on top of the stack identified by `stackid`, replaces the `referencedata` tuple with a new tuple with one additional field containing the string `referencedatastring`, and puts the item back on top of the same stack.

### Defer

Pops a string from the data stack and pushes it as an condition to the Condition stack.

### Satisfy

Pops a condition from the Condition stack and executes it.

### Unlock

Pops a tuple `input` of type [Output](#output) from the data stack. Computes the [id](#Item-ID) of the tuple and pushes it to the Input stack. Pushes each of the `values` to the Value stack, and pushes an [anchor](#anchor) to the Anchors stack. Executes `input.program`.

### UnlockOutput

Pops an output `output` from the Output stack. Pushes each of the `values` to the Value stack. Executes `output.program`.

### Merge

Pops two [Values](#value) from the Value stack. If their asset IDs are different, execution fails. Pushes a new [Value](#value) to the Value stack, whose asset ID is the same as the popped values, and whose amount is the sum of the amounts of each of the popped values.

### Split

Pops a [Value](#value) `value` from the Value stack. Pops an int64 `newamount` from the data stack. If `newamount` is greater than or equal to `value.amount`, fail execution. Pushes a new Value with amount `newamount` and assetID `value.assetID`, then pushes a new Value with amount `newamount - value.amount` and assetID `value.assetID`.

### Lock

Pop a string `referencedata` from the data stack. Pop a number `n` from the data stack. Pop `n` [values](#value), `values`, from the Value stack. Pop a string `program` from the data stack. Push an [output](#output) to the Output stack with `referencedata` as the `referencedata`, a tuple of the `values` as the `values`, and `program` as the `program`.

### Retire

Pops a [Value](#value) `value` from the Value stack. Pushes a [Retirement](#retirement) to the Retirement stack.

### Anchor

Pop a [nonce](#nonce) tuple `nonce` from the data stack. Push `nonce` to the Nonce stack. Push an [anchor](#anchor) to the anchors stack. Execute `nonce.program`.

### Issue

Pop an [asset definition](#asset-definition) tuple `assetdefinition` from the data stack, and pop an int64, `amount`, from the data stack. Pop an [anchor](#anchor) from the Anchor stack. Compute an assetID `assetID` from `assetdefinition`. Push a [value](#value) with amount `amount` and assetID `assetID`. Execute `assetdefinition.issuanceprogram`.

### Header

Fail if the [Transaction Headers stack](#transaction-headers-stack) is not empty.

Pop all items from the Input stack and create a tuple of their IDs, `inputs` (with the top item in the 0th position). Pop all items from the Output stack and create a tuple of their IDs, `outputs`. Pop all items from the Nonce stack and create a tuple of their IDs, `nonces`. Pop all items from the Retirement stack.

Pop a string `referencedata` from the data stack, an int64 `mintime` from the data stack, and an int64 `maxtime` from the data stack. Fail if either `maxtime` or `mintime` is negative, or if `maxtime` is not greater than or equal to `mintime`.

Create a [transaction header](#transaction-header) `header` with `referencedata` set to `referencedata`, `inputs` set to `inputs`, `outputs` set to `outputs`, `mintime` set to `mintime`, and `maxtime` set to `maxtime`. Push `header` to the Transaction Header stack.

[TO BE DONE: Figure out how `annotate` would work in the context of transaction headers without allowing people to add additional annotations after the transaction has been signed.]

### IssueCA

TBD

### ProveRange

TBD

### ProveValue

TBD

### ProveAsset

TBD

### Blind

TBD

## Extension opcodes

### NOPs 0–9

Have no effect when executed.

### Reserved

Causes the VM to halt and fail.

## Encoding operations

### Encode

Pops an item from the data stack. [Serializes](#serialization) it and pushes the result as a string to the data stack.

### Decode

Pops a string `serialized` from the data stack. If it is not a valid [push operation](#push-operations), fails execution. If it is, executes that push operation to push the encoded value onto the data stack.
