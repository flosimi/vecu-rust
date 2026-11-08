# vecu-rust
# V-ECU: A Virtual ECU Host in Rust

> **Documentation-only repository.** The source code is private. This repo describes the architecture and design decisions of a personal proof of concept. Work in progress.

## What it is

A virtual ECU (vECU) host written in Rust. It runs embedded C application code on a regular PC, unmodified, so software can be executed, tested and debugged long before target hardware is available. The host provides a multi-core execution model (two cores) that mirrors how real multi-core ECU configurations are laid out.

## Why

Testing embedded software on real hardware or on closed, commercial simulation setups comes with limits:

- **Hardware is a bottleneck.** Target boards and rigs are scarce, slow to set up and hard to share, so software often waits for them.
- **Closed tools hide what happens inside.** When an application misbehaves, a black-box simulation environment gives little visibility. It is hard to step into the code, inspect state at the exact moment something goes wrong, or change the environment to reproduce the problem.
- **Multi-core makes it harder.** Timing and data exchange between cores are a common source of bugs, and they are the first thing lost when a setup is simplified to a single core.

This project explores a different approach: own the host. If the execution environment is open and written in a memory-safe language, the same C code can run on a PC with full debugger access, a configurable core layout, and a short edit, run and debug loop. The aim is to find problems earlier and to understand them faster, not to replace testing on the real target.

## Goals

- **Run the real C code.** The application under test stays in C. It is compiled for the host and is not rewritten or mocked in Rust.
- **Multi-core by design.** A configuration with two cores maps onto the structure of a real two-core target: each core has its own tasks, and the cores exchange data through defined mechanisms.
- **Debug where the code lives.** Press F5 in VS Code to run the ECU and step from the Rust host straight into the C source, in one workspace.
- **No heavy toolchain.** Everything builds and runs natively on Windows with VS Code. No Linux, WSL or virtual machine is needed.

## Architecture

```
┌──────────────────────────────────────────────┐
│              Rust host (V-ECU)               │
│                                              │
│  ┌────────────┐      ┌────────────┐          │
│  │   Core 0   │◀────▶│   Core 1   │          │
│  │ scheduler  │ data │ scheduler  │          │
│  │  + tasks   │      │  + tasks   │          │
│  └─────┬──────┘      └─────┬──────┘          │
│        │                   │                 │
│        ▼                   ▼                 │
│  ┌────────────────────────────────────────┐  │
│  │      C application code (unmodified)   │  │
│  └────────────────────────────────────────┘  │
│        ▲                                     │
│  ┌─────┴──────────────────────────────────┐  │
│  │   Host layer: timing, I/O, config      │  │
│  └────────────────────────────────────────┘  │
└──────────────────────────────────────────────┘
```

## Design decisions

- **Rust as host, C as payload.** Rust owns scheduling, timing, memory and I/O, which makes the host safe and predictable. The C code runs as the thing being tested.
- **Cores as a first-class concept.** The core layout comes from a configuration, so the same application can be run in a one-core or two-core setup and compared.
- **One workspace for host and application.** Rust and C are built and debugged together, which shortens the loop when the application misbehaves.
- **Built step by step.** Each part of the host (scheduler, core model, I/O layer) is its own module with a clear boundary, so every step can be tested on its own.

## Project structure (planned)

```
vecu/
├── Cargo.toml
├── src/
│   ├── main.rs        # Entry point, configuration loading
│   ├── core.rs        # Core model: tasks and per-core scheduler
│   ├── scheduler.rs   # Task scheduling and timing
│   ├── ipc.rs         # Data exchange between cores
│   └── ffi.rs         # Bridge to the C application
├── c/                 # Demo C application
├── .vscode/           # launch.json and tasks.json for F5 debugging
└── docs/
```

## Status

Proof of concept. A working version runs on two cores. The source code is not public; this repository exists to show how the system is designed and organised.

## Disclaimer

Personal learning and demonstration project. It is not affiliated with any employer and does not contain employer code or confidential information.
