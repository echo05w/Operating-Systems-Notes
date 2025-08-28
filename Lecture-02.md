# Lecture 02: OS Fundamentals

## Interfaces vs Protocols
- **Interface** = static, what functions/signals exist.
  - Hardware: USB, RJ45, VGA.
  - Software: C library, POSIX, Win32 API.
- **Protocol** = dynamic, the sequence of interactions.
  - File protocol: open() → read()/write() → close().
  - Example: ordering food at McDonald's.

## Activities vs Resources
- Activities = tasks, processes, threads (things that run).
- Resources = memory, CPU, devices, files (things needed).
- OS manages allocation and release.

## Preemptable vs Non-preemptable
- **Preemptable**: can be taken & restored later (CPU, memory).
- **Non-preemptable**: cannot be interrupted safely (printer, disk, network card).

## Context Switch
- Saving CPU state of one process & restoring another.
- Enables multitasking.

## Exclusive vs Shared
- **Exclusive**: only one at a time (printer, writable file).
- **Shared**: many can use (read-only memory, read-only files).
- OS enforces exclusivity with locks/semaphores.

## Resource Transformation
- OS transforms raw hardware → higher-level abstraction.
- Example: disk sector → logical block → file.
- Memory + code + ID = new process.

## CPU Modes
- **Kernel Mode**: full privileges, OS code runs here.
- **User Mode**: restricted, app code runs here.
- Protection mechanism.

## System Calls
- Apps request OS services.
- Examples: fork(), open(), read(), wait().
- Involve switching CPU mode (user ↔ kernel).
- Can block.

## Interrupts
- Signals that stop normal execution.
- Can happen anytime (async).
- Sources: devices, OS, applications.
- CPU saves state → runs Interrupt Service Routine (ISR) → restores state.
- Efficient alternative to polling.
