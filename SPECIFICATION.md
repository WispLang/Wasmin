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

| Jasmin   | WAT            | Wasmin         |
|----------|----------------|----------------|
|          | `local.get $i` | `LocalGet $i`  |
|          | `local.set $i` | `LocalSet $i`  |
|          | `local.tee $i` | `LocalTee $i`  |
| `ILOAD`  | `local.get $i` | `ILocalGet $i` |
| `ISTORE` | `local.set $i` | `ILocalSet $i` |
|          | `local.tee $i` | `ILocalTee $i` |
| `LLOAD`  | `local.get $i` | `LLocalGet $i` |
| `LSTORE` | `local.set $i` | `LLocalSet $i` |
|          | `local.tee $i` | `LLocalTee $i` |
| `FLOAD`  | `local.get $i` | `FLocalGet $i` |
| `FSTORE` | `local.set $i` | `FLocalSet $i` |
|          | `local.tee $i` | `FLocalTee $i` |
| `DLOAD`  | `local.get $i` | `DLocalGet $i` |
| `DSTORE` | `local.set $i` | `DLocalSet $i` |
|          | `local.tee $i` | `DLocalTee $i` |

#### Load & Store / Get & Set Globals

| WAT             | Wasmin          |
|-----------------|-----------------|
| `global.get $i` | `GlobalGet $i`  |
| `global.set $i` | `GlobalSet $i`  |
| `global.get $i` | `IGlobalGet $i` |
| `global.set $i` | `IGlobalSet $i` |
| `global.get $i` | `LGlobalGet $i` |
| `global.set $i` | `LGlobalSet $i` |
| `global.get $i` | `FGlobalGet $i` |
| `global.set $i` | `FGlobalSet $i` |
| `global.get $i` | `DGlobalGet $i` |
| `global.set $i` | `DGlobalSet $i` |

#### Load & Store Memory

| WAT            | Wasmin       |
|----------------|--------------|
| `i32.load`     | `IMemLoad`   |
| `i64.load`     | `LMemLoad`   |
| `f32.load`     | `FMemLoad`   |
| `f64.load`     | `DMemLoad`   |
| `i32.load8_S`  | `BIMemLoad`  |
| `i32.load16_S` | `SIMemLoad`  |
| `i32.load8_U`  | `UBIMemLoad` |
| `i32.load16_U` | `USIMemLoad` |
| `i64.load8_S`  | `BLMemLoad`  |
| `i64.load16_S` | `SLMemLoad`  |
| `i64.load32_S` | `ILMemLoad`  |
| `i64.load8_U`  | `UBLMemLoad` |
| `i64.load16_U` | `USLMemLoad` |
| `i64.load32_U` | `UILMemLoad` |
| `i32.store`    | `IMemStore`  |
| `i64.store`    | `LMemStore`  |
| `f32.store`    | `FMemStore`  |
| `f64.store`    | `DMemStore`  |
| `i32.store8`   | `BIMemStore` |
| `i32.store16`  | `SIMemStore` |
| `i64.store8`   | `BLMemStore` |
| `i64.store16`  | `SLMemStore` |
| `i64.store32`  | `ILMemStore` |

#### Pushing Constants

| Jasmin   | WAT            | Wasmin      | Semantics                              |
|----------|----------------|-------------|----------------------------------------|
| `BIPUSH` | `i32.const $i` | `BConst $i` | Push byte on stack as I32 (Comp-time)  |
| `SIPUSH` | `i32.const $i` | `SConst $i` | Push short on stack as I32 (Comp-time) |
| `LDC I`  | `i32.const $i` | `IConst $i` | Push integer on stack as I32           |
| `LDC2 J` | `i64.const $i` | `LConst $i` | Push long on stack as I64              |
| `LDC F`  | `f32.const $i` | `FConst $i` | Push float on stack as F32             |
| `LDC2 D` | `f64.const $i` | `DConst $i` | Push double on stack as F64            |

#### Conditional operators

| WAT        | Wasmin             | Semantics                      |
|------------|--------------------|--------------------------------|
| `i32.eqz`  | `IEqualZero`       | Pop integer and return 1 if 0. |
| `i32.eq`   | `IEqual`           |
| `i32.ne`   | `INotEqual`        |
| `i32.lt_s` | `ILessThan`        |
| `i32.gt_s` | `IGreaterThan`     |
| `i32.le_s` | `ILessOrEqual`     |
| `i32.ge_s` | `IGreaterOrEqual`  |
| `i32.lt_u` | `UILessThan`       |
| `i32.gt_u` | `UIGreaterThan`    |
| `i32.le_u` | `UILessOrEqual`    |
| `i32.ge_u` | `UIGreaterOrEqual` |
| `i64.eqz`  | `LEqualZero`       | Pop long and return 1 if 0.    |
| `i64.eq`   | `LEqual`           |
| `i64.ne`   | `LNotEqual`        |
| `i64.lt_s` | `LLessThan`        |
| `i64.gt_s` | `LGreaterThan`     |
| `i64.le_s` | `LLessOrEqual`     |
| `i64.ge_s` | `LGreaterOrEqual`  |
| `i64.lt_u` | `ULLessThan`       |
| `i64.gt_u` | `ULGreaterThan`    |
| `i64.le_u` | `ULLessOrEqual`    |
| `i64.ge_u` | `ULGreaterOrEqual` |
| `f32.eq`   | `FEqual`           |
| `f32.ne`   | `FNotEqual`        |
| `f32.lt`   | `FLessThan`        |
| `f32.gt`   | `FGreaterThan`     |
| `f32.le`   | `FLessOrEqual`     |
| `f32.ge`   | `FGreaterOrEqual`  |
| `f64.eq`   | `DEqual`           |
| `f64.ne`   | `DNotEqual`        |
| `f64.lt`   | `DLessThan`        |
| `f64.gt`   | `DGreaterThan`     |
| `f64.le`   | `DLessOrEqual`     |
| `f64.ge`   | `DGreaterOrEqual`  |

#### Mathematics & Computing

| WAT                 | Wasmin                | Semantics                                      |
|---------------------|-----------------------|------------------------------------------------|
| `i32.clz`           | `ICountLeadingZeros`  | Count leading zeros in integer                 |
| `i32.ctz`           | `ICountTrailingZeros` | Count trailing zeros in integer                |
| `i32.popcnt`        | `IPopulationCount`    | Count 1 bits in integer                        |
| `i32.add`           | `IAdd`                | Add 2 integers together in stack               |
| `i32.sub`           | `ISub`                | Subtract 2 integers in stack                   |
| `i32.mul`           | `IMul`                | Multiply 2 integers in stack                   |
| `i32.div_s`         | `IDiv`                | Divide 2 integers in stack                     |
| `i32.div_u`         | `UIDiv`               | Unsigned divide 2 integers in stack            |
| `i32.rem_s`         | `IRem`                | Remainder of 2 integers in stack               |
| `i32.rem_u`         | `UIRem`               | Unsigned remainder of 2 integers               |
| `i32.and`           | `IAnd`                | And of 2 integers                              |
| `i32.or`            | `IOr`                 | Or of 2 integers                               |
| `i32.xor`           | `IXor`                | Xor of 2 integers                              |
| `i32.shl`           | `IShiftLeft`          | Shift integer to the left                      |
| `i32.shr_s`         | `IShiftRight`         | Shift integer to the right, preserving sign    |
| `i32.shr_u`         | `UIShiftRight`        | Shift integer to the right, unsigned           |
| `i32.rotl`          | `IRotateLeft`         | Rotate integer to the left                     |
| `i32.rotr`          | `IRotateRight`        | Rotate integer to the right                    |
| `i64.clz`           | `LCountLeadingZeros`  | Count leading zeros in long                    |
| `i64.ctz`           | `LCountTrailingZeros` | Count trailing zeros in long                   |
| `i64.popcnt`        | `LPopulationCount`    | Count 1 bits in long                           |
| `i64.add`           | `LAdd`                | Add 2 longs together in stack                  |
| `i64.sub`           | `LSub`                | Subtract 2 longs in stack                      |
| `i64.mul`           | `LMul`                | Multiply 2 longs in stack                      |
| `i64.div_s`         | `LDiv`                | Divide 2 longs in stack                        |
| `i64.div_u`         | `ULDiv`               | Unsigned divide 2 longs in stack               |
| `i64.rem_s`         | `LRem`                | Remainder of 2 longs in stack                  |
| `i64.rem_u`         | `ULRem`               | Unsigned remainder of 2 longs                  |
| `i64.and`           | `LAnd`                | And of 2 longs                                 |
| `i64.or`            | `LOr`                 | Or of 2 longs                                  |
| `i64.xor`           | `LXor`                | Xor of 2 longs                                 |
| `i64.shl`           | `LShiftLeft`          | Shift long to the left                         |
| `i64.shr_s`         | `LShiftRight`         | Shift long to the right, preserving sign       |
| `i64.shr_u`         | `ULShiftRight`        | Shift long to the right, unsigned              |
| `i64.rotl`          | `LRotateLeft`         | Rotate long to the left                        |
| `i64.rotr`          | `LRotateRight`        | Rotate long to the right                       |
| `f32.abs`           | `FAbsolute`           | Absolute of the float in stack                 |
| `f32.neg`           | `FNegate`             | Negate the float in stack                      |
| `f32.ceil`          | `FRoundCeiling`       | Round float in stack towards the next integer  |
| `f32.floor`         | `FRoundFloor`         | Round float in stack towards the last integer  |
| `f32.trunc`         | `FRoundTruncate`      | Round float in stack towards 0; TODO: verify   |
| `f32.nearest`       | `FRoundNearest`       | Round float in stack towards nearest integer   |
| `f32.sqrt`          | `FSqrt`               | Square root of float in stack                  |
| `f32.add`           | `FAdd`                | Add 2 floats together in stack                 |
| `f32.sub`           | `FSub`                | Subtract 2 floats in stack                     |
| `f32.mul`           | `FMul`                | Multiply 2 floats in stack                     |
| `f32.div`           | `FDiv`                | Divide 2 floats in stack                       |
| `f32.min`           | `FMin`                | Minimum of 2 floats in stack                   |
| `f32.max`           | `FMax`                | Maximum of 2 floats in stack                   |
| `f32.copysign`      | `FCopySign`           | For `p1, p2`, return `p1` with sign of `p2`    |
| `f64.abs`           | `DAbsolute`           | Absolute of the double in stack                |
| `f64.neg`           | `DNegate`             | Negate the double in stack                     |
| `f64.ceil`          | `DRoundCeiling`       | Round double in stack towards the next integer |
| `f64.floor`         | `DRoundFloor`         | Round double in stack towards the last integer |
| `f64.trunc`         | `DRoundTruncate`      | Round double in stack towards 0; TODO: verify  |
| `f64.nearest`       | `DRoundNearest`       | Round double in stack towards nearest integer  |
| `f64.sqrt`          | `DSqrt`               | Square root of double in stack                 |
| `f64.add`           | `DAdd`                | Add 2 doubles together in stack                |
| `f64.sub`           | `DSub`                | Subtract 2 doubles in stack                    |
| `f64.mul`           | `DMul`                | Multiply 2 doubles in stack                    |
| `f64.div`           | `DDiv`                | Divide 2 doubles in stack                      |
| `f64.min`           | `DMin`                | Minimum of 2 doubles in stack                  |
| `f64.max`           | `DMax`                | Maximum of 2 doubles in stack                  |
| `f64.copysign`      | `DCopySign`           | For `p1, p2`, return `p1` with sign of `p2`    |

#### Truncating, extending, casting & reinterpreting

| WAT                   | Wasmin       | Semantics                                    |
|-----------------------|--------------|----------------------------------------------|
| `i32.wrap_i64`        | `IWrapL`     | Wrap long on stack to integer                |
| `i32.trunc_f32_s`     | `ITruncF`    | Truncate float on stack to signed integer    |
| `i32.trunc_f32_u`     | `ITruncUF`   | Truncate float on stack to unsigned integer  |
| `i32.trunc_f64_s`     | `ITruncD`    | Truncate double on stack to signed integer   |
| `i32.trunc_f64_u`     | `ITruncUD`   | Truncate double on stack to unsigned integer |
| `i64.extend_i32_s`    | `LExtendI`   | Extend signed integer on stack to long       |
| `i64.extend_i32_u`    | `LExtendUI`  | Extend unsigned integer on stack to long     |
| `i64.trunc_f32_s`     | `LTruncF`    | Truncate float on stack to signed long       |
| `i64.trunc_f32_u`     | `LTruncUF`   | Truncate float on stack to unsigned long     |
| `i64.trunc_f64_s`     | `LTruncD`    | Truncate double on stack to signed long      |
| `i64.trunc_f64_u`     | `LTruncUD`   | Truncate double on stack to unsigned long    |
| `f32.convert_i32_s`   | `FConvertI`  | Convert signed integer on stack to float     |
| `f32.convert_i32_u`   | `FConvertUI` | Convert unsigned integer on stack to float   |
| `f32.convert_i64_s`   | `FConvertL`  | Convert signed long on stack to float        |
| `f32.convert_i64_u`   | `FConvertUL` | Convert unsigned long on stack to float      |
| `f32.demote_f64`      | `FDemoteD`   | Convert double on stack to float             |
| `f64.convert_i32_s`   | `DConvertI`  | Convert signed integer on stack to double    |
| `f64.convert_i32_u`   | `DConvertUI` | Convert unsigned integer on stack to double  |
| `f64.convert_i64_s`   | `DConvertL`  | Convert signed long on stack to double       |
| `f64.convert_i64_u`   | `DConvertUL` | Convert unsigned long on stack to double     |
| `f64.promote_f32`     | `DPromoteF`  | Convert float on stack to double             |
| `i32.reinterpret_f32` | `IReintF`    | Reinterpret float on stack as integer        |
| `i64.reinterpret_f64` | `LReintD`    | Reinterpret double on stack as long          |
| `f32.reinterpret_i32` | `FReintI`    | Reinterpret integer on stack as float        |
| `f64.reinterpret_i64` | `DReintL`    | Reinterpret long on stack as double          |
| `i32.extend8_s`       | `IExtendB`   | Extend sign of byte as integer on stack      |
| `i32.extend16_s`      | `IExtendS`   | Extend sign of short as integer on stack     |
| `i64.extend8_s`       | `LExtendB`   | Extend sign of byte as long on stack         |
| `i64.extend16_s`      | `LExtendS`   | Extend sign of short as long on stack        |
| `i64.extend32_s`      | `LExtendI`   | Extend sign of integer as long on stack      |

