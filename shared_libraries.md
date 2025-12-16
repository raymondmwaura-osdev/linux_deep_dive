# Shared Libraries

## Overview and Purpose

In modern operating systems, applications often rely on functionality beyond what is defined in their own source code. **Shared libraries**, also known as **dynamic libraries** or **shared objects** (`.so` files on Unix/Linux), serve as reusable modules of compiled code that can be referenced by multiple programs at runtime. Shared libraries:

* Allow multiple applications to use the same code without duplication.
* Reduce overall memory and disk footprint by avoiding embedding common routines into each executable.
* Support modularity and easier updates of third-party and system code.

A shared library is not a complete application; rather, it contains relocatable and linkable code that the **dynamic linker/loader** will map into a program’s address space when the program runs.

---

## Static vs. Shared Libraries

### Libraries

* Contained in archives (`.a` files).
* Linked into the executable at **compile time**.
* Result in a larger binary because all used code is copied into the final executable.
* Do not require presence of external files at runtime.

### Shared Libraries

* Have extension `.so` (shared object).
* Linked primarily at **runtime**, not embedded into the executable.
* A single copy can be shared across multiple programs both on disk and in memory.
* Enable updates without recompiling dependent executables.

---

## Naming Conventions and Versioning

Shared libraries follow specific conventions that facilitate compatibility and dependency resolution:

### Filenames

Typically of the form:

```
libfoo.so        (linker name used during build)
libfoo.so.1      (soname indicating ABI version)
libfoo.so.1.2.3  (real filename with full version)
```

* The **soname** (shared object name) is embedded inside the file and represents the ABI version. It tells the dynamic linker what interface the library supports.
* The linker and runtime use the soname to ensure compatibility even if the underlying implementation version changes.

### Version Compatibility

A soname like `libfoo.so.1` can correspond to multiple actual versions (`libfoo.so.1.0.0`, `libfoo.so.1.1.2`, etc.) as long as they maintain binary compatibility with the ABI expected by dependent programs.

---

## Compilation and Linking

### Creating a Shared Library

To create a shared library from C source files:

1. Compile source files into **position-independent code (PIC)**:

   ```
   gcc -fPIC -c foo.c -o foo.o
   ```

   PIC allows the code to be loaded at arbitrary memory locations, which is required for shared libraries.

2. Link object files into a shared object:

   ```
   gcc -shared -o libfoo.so foo.o
   ```

This produces a shared object `libfoo.so` that other programs can use.

### Linking Against a Shared Library

When compiling a program that uses the shared library:

```
gcc -o app main.c -L/path/to/lib -lfoo
```

* `-L` specifies the directory where the library resides.
* `-lfoo` instructs the linker to link against a library named `libfoo.so`.

---

## The Role of the Dynamic Linker/Loader

At runtime, the operating system uses the **dynamic linker/loader** (e.g., `ld.so` or `ld-linux.so`) to:

1. Locate required shared libraries listed in the executable’s ELF header.
2. Load those libraries into memory, mapping them into the program’s address space.
3. Resolve symbol references (function and variable names) in the executable to their definitions in the libraries.

The dynamic linker consults a sequence of search paths to locate shared libraries:

* Standard trusted directories (`/lib`, `/usr/lib`, etc.).
* Paths specified in `/etc/ld.so.conf` and its includes.
* Environment variables such as `LD_LIBRARY_PATH`.
* Hardcoded search paths embedded in the executable (via **RPATH** or **RUNPATH**).

#### ld.so.cache and ldconfig

`ldconfig` scans shared library directories and builds the **ld.so.cache** file, which the dynamic linker uses to speed up lookups at load time. Administrators update this cache when libraries are installed or removed.

---

## Symbol Resolution and Execution

During loading:

* A program’s undefined symbols are resolved by matching them to definitions in the shared libraries.
* Resolution includes function pointers in the Global Offset Table (GOT) that the program uses for calls to shared functions.

Symbol resolution strategies include:

* **Lazy binding**: Resolve symbols only when they are first called.
* **Eager binding**: Resolve all symbols before program execution begins (can be enforced by environment variables like `LD_BIND_NOW`).

---

## Runtime Inspection and Tools

### ldd

`ldd` lists shared library dependencies of a binary:

```
ldd /bin/ls
```

Output shows each `.so` file required, the path where it was found, and the load address.

### readelf and objdump

These utilities examine ELF headers, symbol tables, and dynamic section entries to understand how a program references shared libraries.

---

## Administrative Considerations

### Library Installation

* Libraries should be placed in standard directories (`/usr/lib`, `/usr/lib64`) or in custom directories registered in `/etc/ld.so.conf`.
* After adding or removing a shared library, run `ldconfig` to update the cache.

### Security and Stability

* Uncontrolled use of environment variables like `LD_LIBRARY_PATH` can introduce security risks.
* Administrators should avoid conflicts between library versions that can break dependent programs.

---

## Advanced Topics

### Dynamic Loading at Runtime

Beyond implicit linking at program start, programs can load libraries explicitly at runtime using APIs like `dlopen`, `dlsym`, and `dlclose` to manage symbols dynamically. This enables plugin architectures and extensible components.

### Dependency Chains and Transitive Libraries

Shared libraries may themselves depend on other shared libraries. The dynamic linker resolves these chains according to specified runpaths or system defaults.

---
