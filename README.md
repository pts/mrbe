# Modern Retro Build Environment (MRBE)

This is the design document for Modern Retro Build Environment, which (when implemented) can run modern and retro build tools on modern host systems. Build tools include assemblers, compilers, compressors, make (incremental build) tools (.mak), script interpreters (.bat, .cmd etc.) written for DOS (16-bit and 32-bit) and for Win32. Modern host systems are modern Windows (i386 or amd64 only, e.g. Windows 10 and some earlier versions of Windows, possibly also ReactOS), Linux (i386 and amd64 only), macOS (i386 and amd64 only, build tools running within Docker). MRBE comes with many free build tools preinstalled (e.g. OpenWatcom V2R C/C++ compiler and linker, WMAKE, TCC C compiler, NASM assembler), and it can also run some popular free-to-use and proprietary build tools (e.g. TASM32 assembler, TLINK32 linker, TLINK linker, Turbo Pascal compiler, LZASM assembler, MASM32 assembler, A86 assembler).

The goal of MRBE is to make it possible to rebuild and modify PC (with processors 8086 ... i386 ... Pentium...) software for DOS, Win16 and Win32. Typically such software was originally designed and implemented between 1985 and 1999, for DOS 3.0 .. 6.22, Windows 3.x, Windows 95, Windows NT 3.1 &ndash; Windows 2000. MRBE makes it convenient to rebuild such software on a modern host system, either the original software or with minor modifications, with the original (retro) build tools (but some modern build tools are also easily available within MRBE). MRBE also facilitates reproducible builds.

MRBE doesn't contain a debugger or an emulator to run the software built, but those are easily available elsewhere, e.g. unmodified DOSBox, Wine or QEMU. If the software executable built happens to be compatible with the MRBE inner environment (i.e. it's a text-mode, non-interactive DOS program or a Win32 console application), then MRBE can run it (so there is no need for an additional emulator), even as part of the current invocation.

For software builds, MRBE provides the following benefits over direct use of emulators (virtual machines):

* Quick installation (less than a minute). No need to install virtual machines and guest operating systems within a virtual machine. Such an installation could take hours, and include frustrating experimentation with emulator settings, until it finally works. Wine is easier to install (because there is no separate step for guest operating system installation), but it also needs some configuration. In contrast, MRBE comes with Wine ready to use (preinstalled and preconfigured).
* Reproducible operation. It's very easy to archive the build files (e.g. source files and files built from them) and share them with others. Work on the build can continue on another host system, without the system transition breaking it.
* High performance. Win32 build tools run at almost native speed. (For implementation reasons, DOS build tools run slower than native in the initial design because of the architecture and emulation speed of DOSBox.)
* Reasonable defaults. No need to create config files or to specify long emulator command-line flags.
* No need to manually stop the emulator as soon as the build has finished.
* Quick startup and shutdown time. It's possible to run many small builds in less than a second each.
* Parallel operation. Multiple builds can be run in parallel on the same host system.
* Convenient read-write sharding of files and directories between the host system and the MRBE inner environment (where the build tools run).
* More convenient environment variable and standard stream (standard input, standard output and standard error) propagation.

## Invocation environment

MRBE provides a command-line tool `bmr`, which can be invoked from a terminal (command prompt window) on the host system. There is no GUI. A terminal emulator may be provided in the future, but as of now any of the native terminals are used on Linux and macOS, and the native black console window (in which cmd.exe also runs) is used on Windows.

In the command-line `bmr` expects the name of the build tool (usually and .exe file) and its command line arguments. It accepts some `bmr`-specific command-line flags and environment variables. It also accepts standard input (can be redirected), which it forwards to the build tool. Whatever the build tool writes to its standard output and standard error, `bmr` forwards to the host system, respectively. These standard streams are not binary-safe (not even when redirected), but ASCII text goes through with possibly some LF <-> CRLF translation. (In a future version, the standard streams will be binary-safe if redirected to a file or pipe.) It's strongly recommended to run build tools in non-interactive mode, i.e. by specifying input and output files in command-line arguments rather than typing them interactively &ndash; however, limited interactive use (ASCII only, without colors or cursor positioning) is also supported.

Build tools running inside `bmr` (i.e. in the inner MRBE environment) must be either Win32 console applications (and they see a Win32 environment, possibly emulated with stripped-down Wine 5.0 or later) or text-mode DOS programs (and they see DOS environment emulated with a stripped-down DOSBox 0.74-3 or later, with the actual screen hidden, and the user sees only their standard output and error). For both kinds of build tools, host system files in the current directory where `bmr` was invoked is available (for read-write) as a virtual C: drive. Also, for build tools, the directory specified in the `BMRTOOLDIR` environment variable is available for read-only as a virtual T: drive, and is added to the `PATH`. Filenames are case-insensitive from the build tools' perspective. In case of DOSBox, long filenames (longer than 8.3 or containing whitespace) are supported in Win32, but not in DOSBox. Filenames with non-ASCII characters are not supported. In case of DOSBox, modification to the directory backing the C: drive while `bmr` is running may not be visible to the build tools. In case of Win32 build tools, all file and directory changes propagate instantly between the host system and the build tools. For imlementation reasons, if `bmr` is running on a Win32 host system natively, the C: and T: drives (as described above) are not available for Win32 build tools running under `bmr`, but some other pathnames will be used instead, making them the current drive and directory (instead of `C:\`), and adding them to the `PATH` (instead of `T:\`).

Build tools can run each other within a single `bmr` invocation. A Win32 console application can run another Win32 console application or a DOS program. A DOS program can run another DOS program (but not a Win32 application). Most environment variables propagate between build tools (even if a Win32 console application runs a DOS program); one exceptions is parts of `PATH` which don't exist in the other rmulator.

MRBE shouldn't be used as a security barrier between the host system and the build tools. When running a Win32 console application under `bmr` running on a Win32 host natively, the build too can see and modify all data and all drives the host user has access to. On Linux and macOS a little better isolation is provided: build tools running under `mrbe` can see only the host directories hosting drive C: (read-write) and T: (read-only). Even DOS build tools aren't sufficiently isolated from the host system, because DOSBox can crash or a malicious DOS program can escape by corrupting DOSBox memory.

## Software distribution

MRBE can be easily downloaded (on any Windows, Linux or macOS system with architecture i386 or amd64) by running a few commands and waiting for the download to finish. The `bmr` tool is available right after download, there is no need to do any installation. Adding the `bmr` tool to the `PATH` for convenience is optional. MRBE doesn't leave any files, directories (or other artifacts) on the host system outside the directory it was downloaded to. So when the user has finished using MRBE, they can uninstall it by deleting the MRBE directory. MRBE doesn't need administrator (root, superuser) privileges to install or run. On macOS, Docker has to be installed first manually (and that typically needs administrator privileges).

MRBE comes with some free build tools (see above which) preinstalled to the directory backing the T: drive. The `mrbeget` command-line tool is also provided for convenient, single-command downloading of more build tools, e.g. `mrbeget lzasm` will download the LZASM assembler and put `lzasm.exe` to the directory backing the T: drive, so that it will work when `bmr lzasm ...` is executed. Binaries of free-to-use build tools are hosted in a central repository used by `mrbeget`. Other build tools (e.g. proprietary ones) can be copied manually to the directory backing the T: drive (or even to the C: drive). There is no version management, e.g. if multiple versions of NASM are available in the central repository, then `nasm.exe` used by `bmr nasm ...` will be the one which was installed or copied last. Subsequent installs with `mrbeget` may overwrite previous files, on which `mrbeget` aborts unless the `--overwrite` command-line flag was specified.

The central repository contains precompiled `.exe` files of free software + open source build tools (e.g. the NASM assembler), and it also contains the `.exe` files of some free-to-use, but maybe not free software or open source build tools (e.g. the A86 assembler and the LZASM assembler). Proprietiary build tools (e.g. Turbo Pascal 7.0 compiler) are not available in the central repository, but some documentation may be provided on using them within MRBE after obtaining them from other sources.

## Custom software

* The `bmr` and `mrbeget` tools are custom software written for MRBE. They are both free and open-source. Executables for Win32 (.exe) and Linux i386 (statically linked) are distributed in the MRBE download. There are no macOS-specific executables, macOS will run the MRBE Linux i386 executables within Docker.

* For Win32, a patched build of DOSBox >= 0.74-3 is provided (as `mrbe_dosbox.exe` and some config files).

* For Linux i386 and amd64 (and for macOS with Docker), a patched build of DOSBox >= 0.74-3 and Wine >= 5.0 are provided, as statically linked Linux ELF i386 executables.

### Modifications to DOSBox

* The screen (separate DOSBox) window is removed.
* Standard DOS streams (including characters written with int 21h and int 10h) forwarded to their host system counterparts within the terminal. Color information and cursor positioning is ignored. (Interactive use such as Norton Commander doesn't work. Graphics doesn't work.)
* A command-line flag is added to change available memory to DOSBox, default is increased to 63 MiB (maximum of DOSBox).
* Most hardware drivers (e.g. sound card, mouse) are removed.
* In the future, a driver for long filenames may be added.
* Startup is modified to mount the drives C: and T:.
* Some small changes are made to reduce startup time even more.
* Run at maximum available CPU speed by default.

### Modifications to Wine

* Wine comes with its system directories and files preinstalled, so first startup doesn't take >20 seconds.
* Subsequent startup time is reduced (currectly it's ~4 seconds on modern systems) by either keeping wineserver running for a few minutes, or with other tricks.
* All GUI components are removed, only command-line tools remain.
* The DISPLAY environment variable is forcibly unset, to prevent the startup of GUI programs.
* The C: drive is renamed to the S: drive, leaving C: to the user in the current directory of the `bmr` invocation.
* The T: drive is also mounted (read-only), pointing to the build tools.
* The Z: drive is not mounted (by default it's the host root directory `/`), to prevent accidental unwanted changes to data on the host.
* When Wine encounters a DOS .exe program, it will invoke the DOSBox modified by MRBE, connecting standard streams correctly.

### Modifications to native Win32

* When `bmr` is run natively on a Win32 host (e.g. Windows 10), and when a Win32 build tool running under `bmr` runs a DOS .exe, then the DOSBox modified by MRBE will be run (instead of NTVDM), connecting standard streams correctly. (It's unclear how hard it is to implement, maybe LoadLibraryA and/or CreateProcess has to be hooked for that only for Win32 build tools running under `bmr`).
* The C: and T: drives are not available for Win32 build tools running under `bmr` running on a Win32 host natively, but some other pathnames will be used instead, making them the current drive and directory (instead of `C:\`), and adding them to the `PATH` (instead of `T:\`).

## Build tools that will work

* OpenWatcom V2 C/C++ compiler (and WLINK linker and WASM assembler etc.), also the Fortran compiler
* OpenWatcom V2 WMAKE (incremental build tool) and other tools (e.g. dmpobj, WDIS disassembler)
* TCC C compiler targeting Win32
* Turbo Pascal 7.0 compiler, also the Borland Pascal 7.0 compiler
* NASM assembler
* YASM assembler
* A86 assembler
* TASM assembler with TLINK linker
* TASM32 assembler with TLINK or TLINK32 linker
* MASM assembler and the corresponding linker, various versions from 3.01, 6.14 etc.
* LZASM assembler
* NBASM assembler (both the DOS and the Win32 version)
* Wolfware Assembler
* FASM assembler
* FASMARM assembler (with ARM target)
* JWasm assembler
* GoAsm assembler and GoLink linker
* Poasm assembler and Polink linker
* Solar Assembler
* JWlink liner
* ALINK linker
* OPTLINK linker
* VAL linker
* Sphinx C-- Compiler
* BassPasC (BAPC) v2 and v3 compiler
* (Probably many others, Wine can run many Win32 build tools, DOSBox can run many DOS build tools.)
