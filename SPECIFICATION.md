# Wasmin Text Specification

Text specification for the Wasmin assembler. Do note, at current, this is work in progress and may not reflect the final
design.

## Types

WASM has only 4 major types, `i32`, `i64`, `f32` and `f64`, which can be mapped to Java's `int`, `long`, `float` and
`double`. The Java types, `boolean`, `byte`, `char`, and `short` will be ultimately mapped to `int`, and validation will
only be best effort. The `U` prefix will be used for marking types as intended to be unsigned, or the instruction is the
unsigned variant.

### Numeric types

- `Z` - `boolean` as `i32 & 1`
- `B` - `byte` as `i32 << 24 >> 24`; signed
- `C` - `char` as `i32 & 65535`; unsigned
- `S` - `short` as `i32 << 16 >> 16`; signed
- `I` - `int` as `i32`; signed
- `L` - `long` as `i64`; signed
- `F` - `float` as `f32`
- `D` - `double` as `f64`
- `U` - `unsigned` prefix

### Objective types

WASM does not support object types at the bytecode level at this time. These are only here as potential reserved.

For reference, the following types may be...

- `O` - `Object`. Reserved.
- `P` - `Pointer`. Reserved. May only be used for validation. Will be mapped to the closest fitting integer.
- `Q` - `Struct`. Reserved.

### Miscellaneous types

WASM doesn't have the concept of void at the bytecode level, and as such, `V` will essentially mean nothing when
assembled.

- `V` - `void`. May mean no return.

## Instructions

The Jasmin, WAT and Wasmin instruction names. Jasmin names are only provided as a reference to the JVM, and may be
incomplete due to not mapping 1:1 to WebAssembly.

Some instructions may emit additional instructions as part of checking & casting, if it is detected that the original
may not fit the constraints.

### Numeric Instructions

#### Load & Store / Get & Set Locals

| Jasmin   | WAT            | Wasmin             | Semantics                       |
|----------|----------------|--------------------|---------------------------------|
|          | `local.get $i` | `local.get $i`     | Get local                       |
|          | `local.set $i` | `local.set $i`     | Set local                       |
|          | `local.tee $i` | `local.tee $i`     | Dup & set local                 |
| `ILOAD`  | `local.get $i` | `i32.local.get $i` | Get local as i32 enforced       |
| `ISTORE` | `local.set $i` | `i32.local.set $i` | Set local as i32 enforced       |
|          | `local.tee $i` | `i32.local.tee $i` | Dup & set local as i32 enforced |
| `LLOAD`  | `local.get $i` | `i64.local.get $i` | Get local as i64 enforced       |
| `LSTORE` | `local.set $i` | `i64.local.set $i` | Set local as i64 enforced       |
|          | `local.tee $i` | `i64.local.tee $i` | Dup & set local as i64 enforced |
| `FLOAD`  | `local.get $i` | `f32.local.get $i` | Get local as f32 enforced       |
| `FSTORE` | `local.set $i` | `f32.local.set $i` | Set local as f32 enforced       |
|          | `local.tee $i` | `f32.local.tee $i` | Dup & set local as f32 enforced |
| `DLOAD`  | `local.get $i` | `f64.local.get $i` | Get local as f64 enforced       |
| `DSTORE` | `local.set $i` | `f64.local.set $i` | Set local as f64 enforced       |
|          | `local.tee $i` | `f64.local.tee $i` | Dup & set local as f64 enforced |

#### Load & Store / Get & Set Globals

| WAT             | Wasmin              | Semantics                  |
|-----------------|---------------------|----------------------------|
| `global.get $i` | `global.get $i`     | Get global                 |
| `global.set $i` | `global.set $i`     | Set global                 |
| `global.get $i` | `i32.global.get $i` | Get global as i32 enforced |
| `global.set $i` | `i32.global.set $i` | Set global as i32 enforced |
| `global.get $i` | `i64.global.get $i` | Get global as i64 enforced |
| `global.set $i` | `i64.global.set $i` | Set global as i64 enforced |
| `global.get $i` | `f32.global.get $i` | Get global as f32 enforced |
| `global.set $i` | `f32.global.set $i` | Set global as f32 enforced |
| `global.get $i` | `f64.global.get $i` | Get global as f64 enforced |
| `global.set $i` | `f64.global.set $i` | Set global as f64 enforced |

#### Load & Store Memory

| WAT            |
|----------------|
| `i32.load`     |
| `i64.load`     |
| `f32.load`     |
| `f64.load`     |
| `i32.load8_S`  |
| `i32.load16_S` |
| `i32.load8_U`  |
| `i32.load16_U` |
| `i64.load8_S`  |
| `i64.load16_S` |
| `i64.load32_S` |
| `i64.load8_U`  |
| `i64.load16_U` |
| `i64.load32_U` |
| `i32.store`    |
| `i64.store`    |
| `f32.store`    |
| `f64.store`    |
| `i32.store8`   |
| `i32.store16`  |
| `i64.store8`   |
| `i64.store16`  |
| `i64.store32`  |

#### Pushing Constants

| Jasmin   | WAT            | Wasmin         | Semantics                              |
|----------|----------------|----------------|----------------------------------------|
| `BIPUSH` | `i32.const $i` | `i8.const $i`  | Push byte on stack as I32 (Comp-time)  |
| `SIPUSH` | `i32.const $i` | `i16.const $i` | Push short on stack as I32 (Comp-time) |
| `LDC I`  | `i32.const $i` | `i32.const $i` | Push integer on stack as I32           |
| `LDC2 J` | `i64.const $i` | `i64.const $i` | Push long on stack as I64              |
| `LDC F`  | `f32.const $i` | `f32.const $i` | Push float on stack as F32             |
| `LDC2 D` | `f64.const $i` | `f64.const $i` | Push double on stack as F64            |

#### Conditional operators

| WAT        | Semantics                      |
|------------|--------------------------------|
| `i32.eqz`  | Pop integer and return 1 if 0. |
| `i32.eq`   |
| `i32.ne`   |
| `i32.lt_s` |
| `i32.gt_s` |
| `i32.le_s` |
| `i32.ge_s` |
| `i32.lt_u` |
| `i32.gt_u` |
| `i32.le_u` |
| `i32.ge_u` |
| `i64.eqz`  | Pop long and return 1 if 0.    |
| `i64.eq`   |
| `i64.ne`   |
| `i64.lt_s` |
| `i64.gt_s` |
| `i64.le_s` |
| `i64.ge_s` |
| `i64.lt_u` |
| `i64.gt_u` |
| `i64.le_u` |
| `i64.ge_u` |
| `f32.eq`   |
| `f32.ne`   |
| `f32.lt`   |
| `f32.gt`   |
| `f32.le`   |
| `f32.ge`   |
| `f64.eq`   |
| `f64.ne`   |
| `f64.lt`   |
| `f64.gt`   |
| `f64.le`   |
| `f64.ge`   |

#### Mathematics & Computing

| WAT            | Semantics                                      |
|----------------|------------------------------------------------|
| `i32.clz`      | Count leading zeros in integer                 |
| `i32.ctz`      | Count trailing zeros in integer                |
| `i32.popcnt`   | Count 1 bits in integer                        |
| `i32.add`      | Add 2 integers together in stack               |
| `i32.sub`      | Subtract 2 integers in stack                   |
| `i32.mul`      | Multiply 2 integers in stack                   |
| `i32.div_s`    | Divide 2 integers in stack                     |
| `i32.div_u`    | Unsigned divide 2 integers in stack            |
| `i32.rem_s`    | Remainder of 2 integers in stack               |
| `i32.rem_u`    | Unsigned remainder of 2 integers               |
| `i32.and`      | And of 2 integers                              |
| `i32.or`       | Or of 2 integers                               |
| `i32.xor`      | Xor of 2 integers                              |
| `i32.shl`      | Shift integer to the left                      |
| `i32.shr_s`    | Shift integer to the right, preserving sign    |
| `i32.shr_u`    | Shift integer to the right, unsigned           |
| `i32.rotl`     | Rotate integer to the left                     |
| `i32.rotr`     | Rotate integer to the right                    |
| `i64.clz`      | Count leading zeros in long                    |
| `i64.ctz`      | Count trailing zeros in long                   |
| `i64.popcnt`   | Count 1 bits in long                           |
| `i64.add`      | Add 2 longs together in stack                  |
| `i64.sub`      | Subtract 2 longs in stack                      |
| `i64.mul`      | Multiply 2 longs in stack                      |
| `i64.div_s`    | Divide 2 longs in stack                        |
| `i64.div_u`    | Unsigned divide 2 longs in stack               |
| `i64.rem_s`    | Remainder of 2 longs in stack                  |
| `i64.rem_u`    | Unsigned remainder of 2 longs                  |
| `i64.and`      | And of 2 longs                                 |
| `i64.or`       | Or of 2 longs                                  |
| `i64.xor`      | Xor of 2 longs                                 |
| `i64.shl`      | Shift long to the left                         |
| `i64.shr_s`    | Shift long to the right, preserving sign       |
| `i64.shr_u`    | Shift long to the right, unsigned              |
| `i64.rotl`     | Rotate long to the left                        |
| `i64.rotr`     | Rotate long to the right                       |
| `f32.abs`      | Absolute of the float in stack                 |
| `f32.neg`      | Negate the float in stack                      |
| `f32.ceil`     | Round float in stack towards the next integer  |
| `f32.floor`    | Round float in stack towards the last integer  |
| `f32.trunc`    | Round float in stack towards 0; TODO: verify   |
| `f32.nearest`  | Round float in stack towards nearest integer   |
| `f32.sqrt`     | Square root of float in stack                  |
| `f32.add`      | Add 2 floats together in stack                 |
| `f32.sub`      | Subtract 2 floats in stack                     |
| `f32.mul`      | Multiply 2 floats in stack                     |
| `f32.div`      | Divide 2 floats in stack                       |
| `f32.min`      | Minimum of 2 floats in stack                   |
| `f32.max`      | Maximum of 2 floats in stack                   |
| `f32.copysign` | For `p1, p2`, return `p1` with sign of `p2`    |
| `f64.abs`      | Absolute of the double in stack                |
| `f64.neg`      | Negate the double in stack                     |
| `f64.ceil`     | Round double in stack towards the next integer |
| `f64.floor`    | Round double in stack towards the last integer |
| `f64.trunc`    | Round double in stack towards 0; TODO: verify  |
| `f64.nearest`  | Round double in stack towards nearest integer  |
| `f64.sqrt`     | Square root of double in stack                 |
| `f64.add`      | Add 2 doubles together in stack                |
| `f64.sub`      | Subtract 2 doubles in stack                    |
| `f64.mul`      | Multiply 2 doubles in stack                    |
| `f64.div`      | Divide 2 doubles in stack                      |
| `f64.min`      | Minimum of 2 doubles in stack                  |
| `f64.max`      | Maximum of 2 doubles in stack                  |
| `f64.copysign` | For `p1, p2`, return `p1` with sign of `p2`    |

#### Truncating, extending, casting & reinterpreting

| WAT                   | Semantics                                    |
|-----------------------|----------------------------------------------|
| `i32.wrap_i64`        | Wrap long on stack to integer                |
| `i32.trunc_f32_s`     | Truncate float on stack to signed integer    |
| `i32.trunc_f32_u`     | Truncate float on stack to unsigned integer  |
| `i32.trunc_f64_s`     | Truncate double on stack to signed integer   |
| `i32.trunc_f64_u`     | Truncate double on stack to unsigned integer |
| `i64.extend_i32_s`    | Extend signed integer on stack to long       |
| `i64.extend_i32_u`    | Extend unsigned integer on stack to long     |
| `i64.trunc_f32_s`     | Truncate float on stack to signed long       |
| `i64.trunc_f32_u`     | Truncate float on stack to unsigned long     |
| `i64.trunc_f64_s`     | Truncate double on stack to signed long      |
| `i64.trunc_f64_u`     | Truncate double on stack to unsigned long    |
| `f32.convert_i32_s`   | Convert signed integer on stack to float     |
| `f32.convert_i32_u`   | Convert unsigned integer on stack to float   |
| `f32.convert_i64_s`   | Convert signed long on stack to float        |
| `f32.convert_i64_u`   | Convert unsigned long on stack to float      |
| `f32.demote_f64`      | Convert double on stack to float             |
| `f64.convert_i32_s`   | Convert signed integer on stack to double    |
| `f64.convert_i32_u`   | Convert unsigned integer on stack to double  |
| `f64.convert_i64_s`   | Convert signed long on stack to double       |
| `f64.convert_i64_u`   | Convert unsigned long on stack to double     |
| `f64.promote_f32`     | Convert float on stack to double             |
| `i32.reinterpret_f32` | Reinterpret float on stack as integer        |
| `i64.reinterpret_f64` | Reinterpret double on stack as long          |
| `f32.reinterpret_i32` | Reinterpret integer on stack as float        |
| `f64.reinterpret_i64` | Reinterpret long on stack as double          |
| `i32.extend8_s`       | Extend sign of byte as integer on stack      |
| `i32.extend16_s`      | Extend sign of short as integer on stack     |
| `i64.extend8_s`       | Extend sign of byte as long on stack         |
| `i64.extend16_s`      | Extend sign of short as long on stack        |
| `i64.extend32_s`      | Extend sign of integer as long on stack      |

### Reference instructions

| WAT           | Semantics                                           |
|---------------|-----------------------------------------------------|
| `ref.null`    | Push null on stack                                  |
| `ref.is_null` | Pop stack and return 1 if null                      |
| `ref.func $i` | Push reference from function address table on stack |

### Branch instructions

| WAT                    | Semantics                                                  |
|------------------------|------------------------------------------------------------|
| `unreachable`          | Unconditionally trap the VM immediately.                   |
| `nop`                  | No-operation.                                              |
| `block`                | Denotes a block to branch out of.                          |
| `loop`                 | Denotes a loop to branch to the beginning of.              |
| `if`                   | Conditional. Must be followed by `Else` and `End`.         |
| `else`                 | Alternative branch of `If`. Must be followed by `End`.     |
| `try`                  | Block that may throw and handle an exception.              |
| `catch`                | Block that handles an exception.                           |
| `catch_all`            | Block that handles all uncaught exceptions.                |
| `delegate`             | Forward to a specific catch block.                         |
| `throw`                | Throws an exception.                                       |
| `rethrow`              | Rethrows what was caught by the catch block.               |
| `end`                  | End of `Block`, `Loop`, `If`, `Else`, `Try` and function.  |
| `br`                   | Branches to the end of a block or the beginning of a loop. |
| `br_if`                | Conditional version of above.                              |
| `br_table`             | Table switch.                                              |
| `return`               | Return values in stack.                                    |
| `call`                 | Call to function.                                          |
| `call_indirect`        | Call to function indirectly.                               |
| `return_call`          | Return values in stack to caller.                          |
| `return_call_indirect` | Return values in stack to indirect caller.                 |
| `drop`                 | Pop the value from stack                                   |
| `select`               | For `a, b, i`, if `i` is `0`, return `b`, else `a`.        |
| `select $t`            | Above but `a` and `b` must be of type `t`.                 |

## Functions

Functions can be declared like the following:

```wsmn
.func RESULT name(PARAM a) LOCAL b
	; Instructions here.
.end
```

The equivalent in WAT is the following:

```wat
(func (param $a PARAM) (local $b LOCAL) (result RESULT)
	;; Instructions here.
)
```

A more complete example:

<details><summary>Wasmin</summary>

```asm
.import "console" "str" .func str
.import "console" "num" .func num

.start main

.func start() i32
		i32.const 0
		local.set 0
	loop:
		local.get 0
		i32.const 4096
		i32.gt_u
		br_if $exit
		local.get 0
		i32.const 1
		i32.add
		local.set 0
		local.get 0
		i32.const 3
		i32.rem_s
		i32.const 0
		i32.ne
		br_if $skipFizz
		i32.const 1
		call $str
		i32.const 0
		local.get 0
		i32.sub
		local.set 0
	skipFizz:
		local.get 0
		i32.const 5
		i32.rem_s
		i32.const 0
		i32.ne
		br_if $skipBuzz
		i32.const 2
		call $str
		local.get 0
		i32.const 0
		i32.lt_s
		br_if $skipBuzz
		i32.const 0
		local.get 0
		i32.sub
		local.set 0
	skipBuzz:
		local.get 0
		i32.const 0
		i32.gt_s
		br_if $skipToPrint
		i32.const 0
		local.get 0
		i32.sub
		local.set 0
		i32.const 0
		call $str
		br $loop
	skipToPrint:
		local.get 0
		call $num
		br $loop
	exit:
.end
```

</details>

<details><summary>WAT</summary>

```wat
(module
	(import "console" "str" (func $str (param i32)))
	(import "console" "num" (func $num (param i32)))
	(start $start)
	(func $start (local i32)
		i32.const 0
		local.set 0
		(block $exit
		(loop $loop
			local.get 0
			i32.const 4096
			i32.gt_u
			br_if $exit
			local.get 0
			i32.const 1
			i32.add
			local.set 0
			(block $skipFizz
				local.get 0
				i32.const 3
				i32.rem_s
				i32.const 0
				i32.ne
				br_if $skipFizz
				i32.const 1
				call $str
				i32.const 0
				local.get 0
				i32.sub
				local.set 0
			)
			(block $skipBuzz
				local.get 0
				i32.const 5
				i32.rem_s
				i32.const 0
				i32.ne
				br_if $skipBuzz
				i32.const 2
				call $str
				local.get 0
				i32.const 0
				i32.lt_s
				br_if $skipBuzz
				i32.const 0
				local.get 0
				i32.sub
				local.set 0
			)
			(block $skipToPrint
				local.get 0
				i32.const 0
				i32.gt_s
				br_if $skipToPrint
				i32.const 0
				local.get 0
				i32.sub
				local.set 0
				i32.const 0
				call $str
				br $loop
			)
			local.get 0
			call $num
			br $loop
		))
	))
```

</details>

## Meta

### Import

| WAT                                  | Wasmin                          | Semantics       |
|--------------------------------------|---------------------------------|-----------------|
| `(import "module" "name" ...)`       | `.import "module" "name" ...`   | Base of import  |
| `(import ... (func $name ...))`      | `.import ... .func ...`         | Import function |
| `(import ... (table ))`              | `.import ... .table `           | Import table    |
| `(import ... (memory pages))`        | `.import ... .memory pages`     | Import memory   |
| `(global $name (import ...) (type))` | `.import ... .global name type` | Import global   |

### Export

| WAT                              | Wasmin                           | Semantics                   |
|----------------------------------|----------------------------------|-----------------------------|
| `(func (export "name") ...)`     | `.func export ...`               | Exports & declares function |
| `(export "name" (func $name))`   | `.export "optional" func name`   | Exports a function by name  |
| `(table (export "name") ...)`    | `.table export ...`              | Exports & declares table    |
| `(export "name" (table $name))`  | `.export "optional" table name`  | Exports a table by name     |
| `(memory (export "name") ...)`   | `.memory export ...`             | Exports & declares memory   |
| `(export "name" (memory $name))` | `.export "optional" memory name` | Exports memory by name      |
| `(global (export "name") ...)`   | `.global export ...`             | Exports & declares global   |
| `(export "name" (global $name))` | `.export "optional" global name` | Exports a global by name    |

### Miscellaneous

| WAT                            | Wasmin                   | Semantics                                      |
|--------------------------------|--------------------------|------------------------------------------------|
| `memory pages`                 | `.memory pages`          | Creates a memory page.                         |
| `data (const offset) "string"` | `.data offset, "string"` | Fills the memory with the requested contents.  |
| `table size type`              | `.table size type`       | Decalres a table of a given size and type.     |
| `elem (const offset) ...`      | `.elem name offset, ...` | Fills the preceding table with elements.       |
| `start $name/id`               | `.start name`            | Declares the starting function.                |
|                                | `.include "./file.wsmn"` | Imports a Wasmin file to be bundled in output. |
