# Atomic.S Generator

This V program generates an architecture-independent assembly file (`atomic.S`) implementing atomic operations by extracting disassembled code from pre-compiled `libatomic.a` libraries for multiple CPU architectures.

## Features
- Supports 5 CPU architectures: `i386`, `x86_64`, `arm`, `aarch64`, `riscv`
- Implements 16 size-suffixed atomic operations (`__atomic_*_1/2/4/8`)
- Includes 6 non-size-suffixed atomic operations (`atomic_*`)
- Generates both raw machine code (for TinyCC) and assembly mnemonics
- Handles symbol name translation and label formatting

## Supported Functions
### Size-Suffixed Operations (1/2/4/8 bytes)
```
__atomic_load
__atomic_store
__atomic_compare_exchange
__atomic_exchange
__atomic_fetch_add
__atomic_add_fetch
__atomic_fetch_sub
__atomic_sub_fetch
__atomic_fetch_and
__atomic_and_fetch
__atomic_fetch_or
__atomic_or_fetch
__atomic_fetch_nand
__atomic_nand_fetch
__atomic_fetch_xor
__atomic_xor_fetch
__atomic_test_and_set
```

### Non-Size-Suffixed Operations
```
atomic_thread_fence
atomic_signal_fence
atomic_flag_test_and_set
atomic_flag_test_and_set_explicit
atomic_flag_clear
atomic_flag_clear_explicit
```

## Requirements
- V compiler
- `objdump` supporting all target architectures
```bash
tar -zxf binutils-2.44.tar.gz
cd binutils-2.44
mkdir build
cd build
../configure --enable-targets=all
make
sudo make install
```
- Pre-compiled `libatomic.a` files for each architecture:
  - `libatomic.a.i386`
  - `libatomic.a.x86_64`
  - `libatomic.a.arm`
  - `libatomic.a.aarch64`
  - `libatomic.a.riscv`

## Usage
1. Place pre-compiled `libatomic.a.[arch]` files in the working directory
2. Compile and run the generator:
```bash
v run gen_atomic.v
```
3. Output file `atomic.S` will be generated containing atomic implementations

## File Structure
- `output_global_header()`: Global header with metadata
- `output_arch_header()/footer()`: Architecture-specific sections
- `gen_objdump_file()`: Extracts disassembly via objdump
- `fix_objdump_asm()`: Formats assembly labels
- `output_f_*()`: Generates function headers/bodies/footers
- `process_arch()`: Processes all functions for an architecture

## Verification (Optional)
Enable the `verify()` step in `main()` to:
1. Assemble `atomic.S` to object files
2. Compare disassembly with original libatomic
3. Requires architecture-specific GCC toolchains

## Special Notes
- ARM architectures require symbol name fixes (`libat_` → `__atomic_`)
- TinyCC compatibility mode uses raw machine code (`.int`/`.short`)
- Label format conversion: `44 <.L1^B2>` → `.L___atomic_fetch_add_8_44`

