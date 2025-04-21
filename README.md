# Memory Segments Analysis

## Overview

This project analyzes the memory segments of a compiled program by inspecting its memory layout using various tools such as `size`, `objdump -t`, and `readelf -S`. The primary goal is to demonstrate how different types of variables (global, local, initialized, and uninitialized) map into distinct memory segments, and how compiler optimization flags (`-O0` for debug and `-O3` for release) influence the memory layout.

---

# Table of Contents

1. [Overview](#overview)
2. [Memory Segments](#memory-segments)
    - [1. .text Segment](#1-text-segment)
    - [2. .data Segment](#2-data-segment)
    - [3. .bss Segment](#3-bss-segment)
    - [4. Heap Segment](#4-heap-segment)
    - [5. Stack Segment](#5-stack-segment)
3. [Example Output](#example-output)
    - [Output from `size` Command](#output-from-size-command)
    - [Output from `objdump -t` Command](#output-from-objdump-t-command)
    - [Output from `readelf -S` Command](#output-from-readelf-s-command)
4. [How to Compile and Run](#how-to-compile-and-run)
    - [Clone the Repository](#clone-the-repository)
    - [Build the Project](#build-the-project)
    - [Run the Program](#run-the-program)
    - [Running the Commands Used in GitHub Actions](#running-the-commands-used-in-github-actions)
        - [1. `size` Command](#1-size-command)
        - [2. `objdump -t` Command](#2-objdump-t-command)
        - [3. `readelf -S` Command](#3-readelf-s-command)

---
  
## Memory Segments

Memory segments in a program refer to specific areas in memory allocated for different types of data and program code during the execution of a program. These segments are defined during the compilation and linking process and can be analyzed using tools like `objdump` or `readelf`. Each segment serves a distinct purpose for managing data and instructions.

Here’s a breakdown of the most common segments you'll encounter in compiled programs:

### **1. .text Segment**

- **Purpose**: The `.text` segment contains the **executable code** of the program. This is where all the machine instructions that make up the program's functionality are stored. The code that gets executed when the program runs is placed in this segment.
  
- **Location in memory**: The `.text` section is typically placed in the lower part of the address space (in a read-only region to prevent accidental modification of code). In most systems, this segment is also marked as **read-only** and **executable**.

### **2. .data Segment**

- **Purpose**: The `.data` segment contains **initialized global and static variables**. These variables are explicitly assigned values at the time of program compilation. They are stored here so that they are available when the program starts execution.

- **Location in memory**: This segment is typically located right after the `.text` segment in memory. The `.data` segment is **read-write**, meaning that variables stored here can be modified at runtime.

### **3. .bss Segment**

- **Purpose**: The `.bss` segment is used for **uninitialized global and static variables**. These variables are not explicitly assigned values in the program, but they must still be allocated space in memory. When the program starts, the operating system initializes these variables to zero (or some other default value).
  
- **Location in memory**: Similar to the `.data` segment, the `.bss` segment is also placed in a **read-write** region of memory. However, unlike the `.data` segment, it does not consume space on disk since the variables are zeroed out at runtime.

### **4. Heap Segment**

- **Purpose**: The **heap** is a region of memory used for **dynamic memory allocation**. When you use `new` or `malloc` to allocate memory during runtime, that memory is allocated from the heap.

- **Location in memory**: The heap grows **upwards** (toward higher memory addresses). Memory from the heap is managed by the program, meaning you can dynamically allocate and deallocate memory as needed (using `delete` or `free`).

### **5. Stack Segment**

- **Purpose**: The **stack** is used for **local variables** and function call management. When a function is called, a new "stack frame" is created to store the function’s local variables and return address. Once the function returns, its stack frame is destroyed.

- **Location in memory**: The stack grows **downwards** (towards lower memory addresses). The size of the stack is typically limited, and if this limit is exceeded (e.g., by deeply recursive function calls), a stack overflow error may occur.

---

## Example Output

### Output from `size` Command

The `size` command displays the memory usage of various sections (text, data, bss) of the compiled program. After running the command, you will get an output like this:

#### For **-O0 (debug)** optimization:

```bash
text    data    bss     dec     hex     filename
2202    664     280     3146    c4a     ./build_O0/main
```

#### For **-O3 (release)** optimization:

```bash
text    data    bss     dec     hex     filename
2244    664     280     3188    c74     ./build_O3/main
```

### Explanation of Output

- **text**: The size of the code segment (executable instructions). 
    - In the `-O0` build, the `text` segment is 2202 bytes, while in the `-O3` build, it's 2244 bytes. This difference is due to the compiler optimizations. With `-O3`, the compiler can apply more aggressive optimizations, such as loop unrolling, inlining, and other techniques that can increase the size of the compiled code.
  
- **data**: The size of initialized global variables, which remains the same in both cases (664 bytes). This segment is not affected by compiler optimizations because initialized data remains fixed.

- **bss**: The size of uninitialized global variables, which is also unchanged (280 bytes) between the two builds. This segment contains variables that have no explicit initial values, and its size is unaffected by optimization levels.

- **dec**: The total size of the program in decimal format. 
    - `-O0` gives a total size of 3146 bytes, while `-O3` gives 3188 bytes, reflecting the changes in the `text` segment due to optimization.
  
- **hex**: The total size in hexadecimal format. The values `c4a` (for `-O0`) and `c74` (for `-O3`) correspond to the same difference as in the decimal representation.

### Why Outputs Differ Between `-O0` and `-O3`

The primary reason for the difference in the output is due to the optimization level set during compilation. Here's a breakdown of how optimizations affect the program:

- **-O0** (No optimization): The compiler does minimal to no optimization of the code. It focuses on making debugging easier and retaining as much information as possible for debugging. The generated machine code is typically larger and less efficient in terms of execution speed.

- **-O3** (Maximum optimization): The compiler applies aggressive optimization techniques, such as:
  - **Inlining functions**: Small functions are embedded directly into the code where they are used, reducing function call overhead but increasing code size.
  - **Loop unrolling**: Loops are optimized by expanding them to avoid repeated tests and jumps, which can increase the size of the code.
  - **Vectorization**: The compiler tries to use SIMD (Single Instruction, Multiple Data) instructions to make operations more efficient, which can sometimes increase the size of the compiled code.
  - **Dead code elimination**: Code that does not affect the program’s behavior is removed, although in some cases, this can still cause size increases depending on other optimizations.

These optimizations improve execution speed and performance but may result in a larger code size as certain optimizations introduce additional instructions, inline code, or precomputed values that increase the overall size of the compiled program.

---

### Output from `objdump -t` Command

The `objdump -t` command provides a detailed listing of the symbols in the compiled binary, showing where each function and variable is located in memory. Here's how to interpret the output from this command:

- **Address**: This field shows the memory address where a symbol (variable or function) is located.
    - For example, a function like `main` could be located at address `0x0401130`.
    
- **Type**: The symbol type indicates whether the entry is a function, a global variable, or a data symbol.
    - For example, `F` indicates a function, and `g` indicates a global variable.

- **Bind**: This field indicates the binding of the symbol, such as whether it’s defined within the current file or shared across files.
    - A binding of `df` means the symbol is defined within the file, while `F` indicates it is a function symbol.

- **Size**: The size in bytes of the symbol, which could represent the number of bytes occupied by a variable or the function code itself.

- **Name**: The name of the symbol, such as `main`, `global_var_initialized`, etc.

This output helps you identify where each function and variable is stored in the memory layout of the program, providing insight into how the program is organized in memory.

---

### Output from `readelf -S` Command

The `readelf -S` command displays the section headers of an ELF binary. These headers provide information about each section in the compiled program, such as `.text`, `.data`, and `.bss`. Here's how to interpret the output:

- **Section Name**: This field displays the name of the section, such as `.text`, `.data`, `.bss`, etc.
    - For example, `.text` contains the executable code, `.data` contains initialized global variables, and `.bss` contains uninitialized global variables.

- **Address**: The memory address where the section begins.
    - For example, the `.text` section might start at `0x0401000`.

- **Size**: The size of the section in bytes.
    - For example, the `.text` section might have a size of `0x2244` bytes.

- **Type**: The type of section, indicating whether it contains program data (`PROGBITS`) or uninitialized data (`NOBITS`).
    - For example, `.text` is of type `PROGBITS` (program data), while `.bss` is of type `NOBITS` (uninitialized data).

The `readelf -S` command helps you understand the structure of the binary and how different sections are organized in memory, which is useful for performance optimization and debugging.

---

## How to Compile and Run

Follow these instructions to compile and run the project:

### 1. Clone the Repository

```bash
git clone https://github.com/LyudmilaKostanyan/Memory-Segments.git
cd Memory-Segments
```

### 2. Build the Project

Ensure you have CMake and a C++ compiler (e.g., g++):

```bash
cmake -S . -B build
cmake --build build
```

### 3. Run the Program

- **Linux/macOS**: 
```bash
./build/main
```

- **Windows**:
```bash
./build/main.exe
```

---

### 4. Running the Commands Used in GitHub Actions

#### 1. **`size` Command**:

- **Linux/macOS**: 
  - The `size` command is generally available by default on Linux and macOS systems. If it's not installed on your macOS machine, you can install it via `brew install binutils` (which includes the `size` tool).
  
  - **Usage**:
    ```bash
    size ./build/main
    ```

- **Windows**:
  - `size` is not available on Windows by default. You may need to use alternative tools like **MinGW-w64** (via Chocolatey or manual installation). Alternatively, if you're using **Windows Subsystem for Linux (WSL)**, you can run the `size` command directly from your WSL terminal.
  
  - **MinGW-w64** (Windows) installation:
    ```bash
    choco install mingw --version=8.1.0 -y
    ```
  
  - **Usage** (after installing MinGW):
    ```bash
    size ./build/main.exe
    ```

  - **WSL**: If you're using **WSL**, you can run the command just as you would on Linux:
    ```bash
    size ./build/main
    ```

#### 2. **`objdump -t` Command**:

- **Linux/macOS**: 
  - The `objdump` command is part of the `binutils` package, which should be installed by default on most Linux distributions and macOS (if you have installed Xcode tools). If not, you can install `binutils` via package managers like `apt` on Linux or `brew` on macOS.
  
  - **Usage**:
    ```bash
    objdump -t ./build/main
    ```

- **Windows**:
  - To use `objdump` on Windows, you must install **MinGW-w64** or **MSYS2** as it provides `objdump` under the `binutils` package. You can install **MinGW-w64** using **Chocolatey** or manually.
  
  - **MinGW-w64** (Windows) installation:
    ```bash
    choco install mingw --version=8.1.0 -y
    ```

  - **Usage** (after installing MinGW):
    ```bash
    objdump -t ./build/main.exe
    ```

  - **WSL**: If you're using **WSL**, you can run `objdump` in the WSL terminal as you would on Linux:
    ```bash
    objdump -t ./build/main
    ```

#### 3. **`readelf -S` Command (Linux only)**:

- **Linux**: 
  - The `readelf` command is part of the `binutils` package and is available on most Linux distributions. If it's not installed, you can install `binutils` using:
    ```bash
    sudo apt-get install binutils
    ```
  
  - **Usage**:
    ```bash
    readelf -S ./build/main
    ```

- **macOS/Windows**: 
  - The `readelf` command is specific to **ELF** binaries, and macOS/Windows do not use ELF binaries by default. If you're working on **macOS**, you can use `otool -hvl` instead (see below).

### 4. **`readelf -S` Command (Linux only)**:

- **Linux**: 
  - The `readelf` command is part of the `binutils` package and is available on most Linux distributions. If it's not installed, you can install `binutils` using:
    ```bash
    sudo apt-get install binutils
    ```
  
  - **Usage**:
    ```bash
    readelf -S ./build/main
    ```

- **macOS**:
  - The `readelf` command is not available on macOS. Instead, you can use `otool` to get similar information about the sections of the binary.
  
  - **Usage**:
    ```bash
    otool -hvl ./build/main
    ```

- **Windows**: 
  - **Windows** does not natively support `readelf`. However, you can use **`objdump -x`** (from MinGW or MSYS2) to get similar information about the sections of the binary. 
  - To use `objdump` on **Windows** with **MinGW** installed (via **Chocolatey**), run the following:
  
    **MinGW-w64 Installation (Windows)**:
    ```bash
    choco install mingw --version=8.1.0 -y
    ```

    **Usage (Windows)**:
    ```bash
    objdump -x ./build/main.exe
    ```

---
