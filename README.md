# PIPE/5 — Pipelined RISC Simulator

**Live:** [pipe5.onrender.com](https://pipe5.onrender.com)

## The idea

Most pipeline simulators for ARM/Thumb-style architectures are the same thing: a text console, a command list, and an instruction trace. You read numbers scrolling by and infer what the hardware is doing. There's no visual sense of a pipeline actually moving.

PIPE/5 exists to fix that. It's a 5-stage MIPS-style pipelined RISC CPU where you can *watch* the hardware think — instructions physically flowing through IF → ID → EX → MEM → WB, forwarding paths lighting up when they resolve a hazard, the front end freezing on a load-use stall, a branch flush clearing the pipeline in real time. The goal was to make pipelining legible instead of abstract — something you build intuition for by watching, not just by reading a trace.

## How it was made

The project has two halves that mirror each other.

**The hardware.** A real 5-stage pipelined CPU written in Verilog — program counter, instruction and data memories, a 32-register file, ALU, and dedicated hazard detection and forwarding units, wired together with four clocked inter-stage register banks. This is the actual source of truth for how the processor behaves. It's verified against directed testbenches and inspected with GTKWave waveforms, the same way you'd validate real RTL.

**The simulator.** Rather than trying to run the Verilog itself in a browser, I built a separate, dependency-free JavaScript engine that behaves identically — its own assembler, its own cycle-stepping pipeline model, its own hazard and forwarding logic — as a behavioral twin of the RTL. This is what actually powers the interactive site: assemble a program, step through clock edges or run it continuously, and see every register, memory word, stall, and flush update live alongside a diagram of the datapath itself.

Building the engine's hazard logic was the hardest and most interesting part. A same-cycle register file bug — a register written back in WB and read in ID on the same tick — was silently returning stale values instead of the new result. It didn't crash anything; it just quietly produced wrong answers that still looked plausible. That kind of bug is exactly why cycle-accurate simulation matters: you can watch the exact cycle where a value is wrong instead of guessing from an output.

**The AI layer.** On top of the engine sits an assistant (Gemini-powered) that can generate, fix, explain, or optimize code for this processor's exact instruction subset. It is never trusted blindly — every AI response is fed back through the same deterministic assembler that powers the simulator, and only code that passes validation is allowed to load. The AI writes; the engine checks.

## What it demonstrates

- A cycle-accurate model of classic RISC hazards: RAW dependencies resolved via EX/MEM and MEM/WB forwarding, load-use hazards resolved via stalling and bubble insertion, and control hazards resolved via branch/jump flushing.
- A simulation engine built independently from the RTL it mirrors, rather than a wrapper around it.
- A concrete pattern for using an LLM to generate code for a constrained target language safely — generate, then validate deterministically before trusting the output.

## License

MIT — see `LICENSE`.
