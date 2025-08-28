# Lecture 01: Introduction to Operating Systems

## Why OS matter
- Almost every CPU runs one.
- They are very complex.
- Provide **services** (process, file, pipe, directory, driver).
- Manage **resources**.
- Coordinate **parallelism**.
- Base for **security**.

## History
- **1940s**: no OS (ENIAC).
- **1960s**: Mainframes (IBM System/360 → OS/360 → z/OS).
- **1970s–80s**: Minicomputers → UNIX, VMS.
- **1980s–90s**: Workstations (HP, Sun, etc.).
- **1978–today**: PCs → DOS, Windows, Linux, macOS.
- **2000s–today**: Mobiles → Symbian, iOS, Android.

## UNIX
- Not one OS → family.
- Born 1969 by Ken Thompson & Dennis Ritchie.
- Portable (C language).
- POSIX standard defines compatibility.
- UNIX Philosophy: "Do one thing well", "Small is beautiful", portability, use text files, etc.

## Licensing
- Closed source → only binaries.
- Open source → code public, modifiable.
- **GPL (Stallman, 1989)** → freedom to use, modify, distribute, with copyleft.

## OS Classification
- Single vs Multi-user.
- Single vs Multi-tasking.
- Batch vs Interactive vs Autonomous.
- Local vs Distributed.
- Purpose: server, embedded, RTOS, PC, educational, etc.

## OS Models
- **Monolithic**: all-in-one, fast but messy.
- **Layered**: structured, but can be inefficient.
- **Quasi-layered**: MS-DOS.
- **Ring model**: UNIX.
- **Client-server**: microkernels, services via messages.
