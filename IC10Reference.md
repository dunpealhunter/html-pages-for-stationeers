# IC10 Programming Reference

## Table of Contents

- [IC10 Programming Reference](#ic10-programming-reference)
  - [Table of Contents](#table-of-contents)
  - [Argument Type Definitions](#argument-type-definitions)
  - [Macros](#macros)
  - [Comments](#comments)
  - [Constants](#constants)
  - [Misc Instructions](#misc-instructions)
  - [Device IO](#device-io)
    - [Special Argument Semantics](#special-argument-semantics)
  - [Flow Control](#flow-control)
  - [Select (Test) Instructions](#select-test-instructions)
  - [Mathematical Operations](#mathematical-operations)
  - [Bitwise Operations](#bitwise-operations)
  - [Stack](#stack)
    - [Sending Commands to Other Devices](#sending-commands-to-other-devices)

## Argument Type Definitions

- `r?` = Register or `alias`ed register identifier
  - `r0` - `r15` = General purpose registers
  - `ra` = Return address register (written by `*al` instructions)
  - `sp` = Stack pointer (used by stack manipulation instructions)
- `d?` = Device or `alias`ed device identifier
  - `d0` - `d5` = Connected device
  - `db` = Device this IC is socketed in
- `str` = String literal
- `num` = Numeric literal or `define`d constant
- `id` = Device reference ID (numeric)

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `alias` | **aliasName**(str) **target**(r?\|d?) | Create named alias for register/device |
| `define` | **constantName**(str) **value**(num) | Create constant replaced throughout program |

## Macros

Replace `...` with content.

| Macro | Description |
|-------|-------------|
| `$...` | Hex number, optionally separated with underscores (`$F0`) |
| `%...` | Binary number, optionally separated with underscores (`%1111_0000`) |
| `HASH("...")` | Computes a CRC32 hash of a string (`HASH("StructureWallLight")`) |
| `STR("...")` | Packs a string into up to 53 bits (6 characters) for drawing text |

## Comments

Comments are indicated by a `#` character. All text on a line following a `#` is interpreted as a comment and will not be executed.

## Constants

The following named constants can be used in place of numeric literals:

| Constant | Value | Description |
|----------|-------|-------------|
| `nan` | NaN | Not a number (quiet NaN - signal NaN from operations will halt execution) |
| `pinf` | +∞ | Positive infinity |
| `ninf` | -∞ | Negative infinity |
| `pi` | π (≈3.14159) | Ratio of circle circumference to diameter |
| `tau` | 2π (≈6.28318) | Ratio of circle circumference to radius |
| `deg2rad` | π/180 (≈0.01745) | Degrees to radians conversion factor |
| `rad2deg` | 180/π (≈57.2958) | Radians to degrees conversion factor |
| `epsilon` | ~4.94×10⁻³²⁴ | Smallest positive subnormal number greater than 0 |
| `rgas` | 8.31446 | Universal gas constant (J/(mol·K)) |

## Misc Instructions

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `hcf` | | Halt and catch fire (stop execution) |
| `move` | **targetRegister**(r?) **sourceValue**(r?\|num) | Copy value to register |
| `sleep` | **durationSeconds**(r?\|num) | Pause execution for specified seconds |
| `yield` | | Pause execution for 1 tick |

## Device IO

### Special Argument Semantics

Arguments with these names expect certain kinds of values.

- `deviceHash` = An object's prefab hash (e.g. `HASH("StructureWallLight")`)
- `nameHash` = The hash of a specific name given to an object with a Labeler (e.g. `HASH("Workshop Light")`)
- `batchMode` = Batch processing mode enum. One of:
  - 0/Average (`nan` if no devices)
  - 1/Sum (`0` if no devices)
  - 2/Minimum (`pinf` if no devices)
  - 3/Maximum (`ninf` if no devices)
- `reagentMode` = Reagent query mode enum. One of:
  - 0 = Contents
  - 1 = Required
  - 2 = Recipe
- `reagentHash` = Reagent hash
- `logicType` = Device variable name (e.g., Temperature, Pressure, On)
- `logicSlotType` = Slot variable name (e.g., Occupied, Quantity, Growth)

**Device References:**

- Network connections can be referenced as `device:connection` (e.g., `d0:0` for connection 0)
- Cable networks have 8 channels (Channel0-Channel7) for data storage/transfer

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `l` | **result**(r?) **device**(d?\|r?\|num) **logicType**(str) | Load device variable |
| `lb` | **result**(r?) **deviceHash**(r?\|num) **logicType**(str) **batchMode**(r?\|num) | Load variable from network devices by type |
| `lbn` | **result**(r?) **deviceHash**(r?\|num) **nameHash**(r?\|num) **logicType**(str) **batchMode**(r?\|num) | Load variable from network devices by type and name |
| `lbns` | **result**(r?) **deviceHash**(r?\|num) **nameHash**(r?\|num) **slotIndex**(num) **logicType**(str) **batchMode**(r?\|num) | Load slot variable from network devices by type and name |
| `lbs` | **result**(r?) **deviceHash**(r?\|num) **slotIndex**(num) **logicType**(str) **batchMode**(r?\|num) | Load slot variable from network devices by type |
| `lr` | **result**(r?) **device**(d?) **reagentMode**(r?\|num) **reagentHash**(r?\|num) | Load reagent from device |
| `ls` | **result**(r?) **device**(d?) **slotIndex**(num) **logicType**(str) | Load slot variable from device |
| `rmap` | **result**(r?) **device**(d?) **reagentHash**(r?\|num) | Get prefab hash for reagent requirement |
| `s` | **device**(d?\|r?\|num) **logicType**(str) **sourceRegister**(r?) | Store register to device variable |
| `sb` | **deviceHash**(r?\|num) **logicType**(str) **sourceRegister**(r?) | Store register to variable on all network devices by type |
| `sbn` | **deviceHash**(r?\|num) **nameHash**(r?\|num) **logicType**(str) **sourceRegister**(r?) | Store register to variable on network devices by type and name |
| `sbs` | **deviceHash**(r?\|num) **slotIndex**(num) **logicType**(str) **sourceRegister**(r?) | Store register to slot variable on network devices by type |
| `ss` | **device**(d?\|r?\|num) **slotIndex**(num) **logicType**(str) **sourceRegister**(r?) | Store register to slot variable on device |

## Flow Control

Most jump/branch instructions have three variants:

- Absolute - `targetLine` is an absolute line number or label in the program
- Relative - `targetLine` is how many lines to jump relative to the current line
- Return Address - stores the next line in the `ra` register before branching to an exact line of the program

Labels may be used to refer to lines by name. For example, an infinite loop could look like:

```asm
loopStart:
# some code here
j loopStart
```

| Absolute | Relative | Return Address | Arguments | Description |
|------------------|------------------|------------------------|--------|-------------|
| `j` | `jr` | `jal` | **targetLine**(r?\|num) | Unconditional jump to line |
| `bap` | `brap` | `bapal` | **a**(r?\|num) **b**(r?\|num) **tolerance**(r?\|num) **targetLine**(r?\|num) | Branch if a ≈ b (with tolerance c) |
| `bapz` | `brapz` | `bapzal` | **value**(r?\|num) **tolerance**(r?\|num) **targetLine**(r?\|num) | Branch if a ≈ 0 |
| `beq` | `breq` | `beqal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a == b |
| `beqz` | `breqz` | `beqzal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a == 0 |
| `bge` | `brge` | `bgeal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a ≥ b |
| `bgez` | `brgez` | `bgezal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a ≥ 0 |
| `bgt` | `brgt` | `bgtal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a > b |
| `bgtz` | `brgtz` | `bgtzal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a > 0 |
| `ble` | `brle` | `bleal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a ≤ b |
| `blez` | `brlez` | `blezal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a ≤ 0 |
| `blt` | `brlt` | `bltal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a < b |
| `bltz` | `brltz` | `bltzal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a < 0 |
| `bna` | `brna` | `bnaal` | **a**(r?\|num) **b**(r?\|num) **tolerance**(r?\|num) **targetLine**(r?\|num) | Branch if a !≈ b (with tolerance c) |
| `bnaz` | `brnaz` | `bnazal` | **value**(r?\|num) **tolerance**(r?\|num) **targetLine**(r?\|num) | Branch if a !≈ 0 |
| `bne` | `brne` | `bneal` | **a**(r?\|num) **b**(r?\|num) **targetLine**(r?\|num) | Branch if a ≠ b |
| `bnez` | `brnez` | `bnezal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if a ≠ 0 |
| `bnan` | `brnan` | `bnanal` | **value**(r?\|num) **targetLine**(r?\|num) | Branch if value is NaN |
| `bdns` | `brdns` | `bdnsal` | **device**(d?) **targetLine**(r?\|num) | Branch if device not set |
| `bdse` | `brdse` | `bdseal` | **device**(d?) **targetLine**(r?\|num) | Branch if device is set |
| `bdnvl` | - | - | **device**(d?\|r?\|num) **logicType**(str) **targetLine**(r?\|num) | Branch if device not valid for load |
| `bdnvs` | - | - | **device**(d?\|r?\|num) **logicType**(str) **targetLine**(r?\|num) | Branch if device not valid for store |

## Select (Test) Instructions

All instructions in this section store 1 (true) or 0 (false) in the specified register based on the condition.

For instructions using approximate equality (`≈` and `!≈`):

- Two values are approximately equal if: `abs(a-b) ≤ max(c×max(abs(a),abs(b)), ε×8)`
- Single value is approximately zero if: `abs(a) ≤ ε×8`

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `sap` | **result**(r?) **a**(r?\|num) **b**(r?\|num) **tolerance**(r?\|num) | True if a ≈ b (with tolerance c) |
| `sapz` | **result**(r?) **value**(r?\|num) **tolerance**(r?\|num) | True if a ≈ 0 |
| `sdns` | **result**(r?) **device**(d?) | True if device is not set |
| `sdse` | **result**(r?) **device**(d?) | True if device is set |
| `select` | **result**(r?) **condition**(r?\|num) **trueValue**(r?\|num) **falseValue**(r?\|num) | Select b if a ≠ 0, else c |
| `seq` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a == b |
| `seqz` | **result**(r?) **value**(r?\|num) | True if a == 0 |
| `sge` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a ≥ b |
| `sgez` | **result**(r?) **value**(r?\|num) | True if a ≥ 0 |
| `sgt` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a > b |
| `sgtz` | **result**(r?) **value**(r?\|num) | True if a > 0 |
| `sle` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a ≤ b |
| `slez` | **result**(r?) **value**(r?\|num) | True if a ≤ 0 |
| `slt` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a < b |
| `sltz` | **result**(r?) **value**(r?\|num) | True if a < 0 |
| `sna` | **result**(r?) **a**(r?\|num) **b**(r?\|num) **tolerance**(r?\|num) | True if a !≈ b (with tolerance c) |
| `snaz` | **result**(r?) **value**(r?\|num) **tolerance**(r?\|num) | True if a !≈ 0 |
| `sne` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | True if a ≠ b |
| `snez` | **result**(r?) **value**(r?\|num) | True if a ≠ 0 |
| `snan` | **result**(r?) **value**(r?\|num) | True if value is NaN |
| `snanz` | **result**(r?) **value**(r?\|num) | True if value is not NaN |

## Mathematical Operations

Trigonometric instructions work in radians, not degrees.

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `abs` | **result**(r?) **value**(r?\|num) | Absolute value of a |
| `acos` | **result**(r?) **value**(r?\|num) | Arccosine of a |
| `add` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | a + b |
| `asin` | **result**(r?) **value**(r?\|num) | Arcsine |
| `atan` | **result**(r?) **value**(r?\|num) | Arctangent |
| `atan2` | **result**(r?) **y**(r?\|num) **x**(r?\|num) | Returns angle (radians) whose tangent is the quotient of y and x |
| `ceil` | **result**(r?) **value**(r?\|num) | Smallest integer ≥ value |
| `cos` | **result**(r?) **value**(r?\|num) | Cosine |
| `div` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | a ÷ b |
| `exp` | **result**(r?) **value**(r?\|num) | e^a |
| `floor` | **result**(r?) **value**(r?\|num) | Largest integer ≤ value |
| `log` | **result**(r?) **value**(r?\|num) | Natural logarithm |
| `max` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Maximum of a and b |
| `min` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Minimum of a and b |
| `mod` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | a modulo b (NOT remainder) |
| `mul` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | a × b |
| `pow` | **result**(r?) **base**(r?\|num) **exponent**(r?\|num) | base^exponent |
| `rand` | **result**(r?) | Random value 0 ≤ x < 1 |
| `round` | **result**(r?) **value**(r?\|num) | round to nearest integer |
| `sin` | **result**(r?) **value**(r?\|num) | Sine |
| `sqrt` | **result**(r?) **value**(r?\|num) | Square root |
| `sub` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | a - b |
| `tan` | **result**(r?) **value**(r?\|num) | Tangent |
| `trunc` | **result**(r?) **value**(r?\|num) | value with fractional part removed |
| `lerp` | **result**(r?) **a**(r?\|num) **b**(r?\|num) **ratio**(r?\|num) | Linear interpolation between a and b by ratio (clamped 0-1) |

## Bitwise Operations

Bitwise operations work on the binary representation of numbers.

**Note:** Arithmetic shifts preserve the sign of the number (most significant bit), logical shifts do not.

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `not` | **result**(r?) **value**(r?\|num) | Bitwise NOT |
| `and` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Bitwise AND |
| `nor` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Bitwise NOR |
| `or` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Bitwise OR |
| `xor` | **result**(r?) **a**(r?\|num) **b**(r?\|num) | Bitwise XOR |
| `sla` | **result**(r?) **value**(r?\|num) **shiftAmount**(r?\|num) | Shift left arithmetic |
| `sll` | **result**(r?) **value**(r?\|num) **shiftAmount**(r?\|num) | Shift left logical |
| `sra` | **result**(r?) **value**(r?\|num) **shiftAmount**(r?\|num) | Shift right arithmetic |
| `srl` | **result**(r?) **value**(r?\|num) **shiftAmount**(r?\|num) | Shift right logical |
| `ext` | **dest**(r?) **source**(r?\|num) **sourceOffset**(r?\|num) **length**(r?\|num)  | Zero `dest` and copy `length` bits from `source` to `dest`, starting at `source` bit `sourceOffset` and `dest` bit 0. `sourceOffset` must be >= 0, `length` must be >= 1, and `sourceOffset + length` must not exceed 53. |
| `ins` | **dest**(r?) **destOffset**(r?\|num) **length**(r?\|num) **source**(r?\|num) | Copy `length` bits from `source` to `dest`, starting at `source` bit 0 and `dest` bit `destOffset`.  `destOffset` must be >= 0, `length` must be >= 1, and `destOffset + length` must not exceed 53. |

## Stack

| Instruction | Arguments | Description |
|-------------|--------|-------------|
| `clr` | **device**(d?) | Clear stack memory for device |
| `clrd` | **deviceId**(r?\|num) | Clear stack memory for device by ID |
| `get` | **result**(r?) **device**(d?\|r?\|num) **address**(r?\|num) | Read from device stack at address |
| `peek` | **result**(r?) | Read top of stack (no pop) |
| `poke` | **address**(r?\|num) **value**(r?\|num) | Write value to stack at address |
| `pop` | **result**(r?) | Pop value from stack |
| `push` | **value**(r?\|num) | Push value to stack |
| `put` | **device**(d?\|r?\|num) **address**(r?\|num) **value**(r?\|num) | Write value to device stack at address |

### Sending Commands to Other Devices

You can control some devices by reading and writing from their stack. These commands are defined in the Stationpedia entry for each device.

Example of forming a command to print 1 `Kit (Pipe)`:

```mips
# define register aliases for readability
alias printer d0
alias command r0
alias itemQuantity r1
alias itemHash r2

# set command options
move itemQuantity 1
move itemHash HASH("ItemKitPipe")

# build command
move command PrinterInstruction.ExecuteRecipe
ins command 8 8 itemQuantity
ins command 16 32 itemHash

# send command to printer
clr printer
put printer 0 command
```
