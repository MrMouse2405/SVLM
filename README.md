# SLVM: Scripting Language Virtual Machine

SLVM is a high-performance, language-agnostic bytecode virtual machine implemented with low engineering overhead.

## How SLVM is designed

### ON ENGINEERING COST

The SLVM source code is intentionally kept simple, ensuring that, while the algorithms may be complex, the code remains easy to read and debug. The primary goal of SLVM is to rely on stable compiler features and minimize additional engineering costs.

We avoid code generation[^1], compiler hacks, and experimental features of any kind.

#### Memory Safety and Security: Rust

Although Rust may seem like an unconventional choice for implementing a virtual machine, it's important to note that we are intentionally avoiding complex type or pointer manipulations.

We selected Rust because it can significantly reduce engineering costs and technical debt when the codebase is well-structured. Additionally, Rust inherently prevents memory corruption and eliminates memory-related CVEs[^2].

### ON PERFORMANCE

#### More Assumptions = More Optimizations

SLVM achieves high runtime-performance by making more assumptions about the
programming language environment and the supplied blueprints (SLVM bytecode).

| Dynamically Typed Languages |                SLVM                |  Statitcally Typed Languages  |
| :-------------------------: | :--------------------------------: | :---------------------------: |
|      More Flexibility       |          More Flexiblity           |        Less Flexiblity        |
|      Less Assumptions       |          More Assumptions          |       More Assumptions        |
|      Hard to Optimize       | Relatively Less Harder to Optimize | Relatively Easier to Optimize |

#### Awesome O(n)

Virtual machine performance is often measured in **startup time** and **runtime**

SLVM is designed to be:

##### Startup

|      Phase       | SLVM Complexity (Time & Space) |
| :--------------: | :----------------------------: |
|     Parsing      |  Defined by Front End. ∴ N/A   |
|     Compile      |            O(n)[^3]            |
| Load (if cached) |              O(n)              |

##### Runtime

|    Type    | SLVM Complexity (Time & Space) |
| :--------: | :----------------------------: |
| Execution  |              O(n)              |
| Allocation |            O(1)[^4]            |

### Simulated Stack

Often vms are stack based (push + pop) or register based. SLVM's simulated stack on heap is similar to how a compiler would arrange stack. There are no push/pops.

SLVM's blueprint frame arranges memory just like how a C function call is arranged. It reuses the same memory.

This means, stack / heap are real concepts in SLVM. You can store integers in stack, and quickly add them together. While large data structures can be stored in simulated heaps, with a pointer to them in simulated stacks.

### Selective Stack Simulation

SLVM can decide to retain debugging / stack simulation for a blueprint frame. In a nutshell, this means we can optimizing away a lot of smoke and mirrors needed for embedding SLVM into a host program.

### Selective or No Type Checking at all

SLVM can eliminate typechecking or insert selective type checking. Allowing SLVM to generate fast blueprints.

### Foreign & Native Objects

Not all code in a blueprint will get manipulated by a host program. In SLVM, blueprints communicate with
host program via Foreign / Native Objects.

Foreign Frames / Objects are called by host program from SLVM. (SLVM Hooks)

Native Functions are called by SLVM from the host program. (SLVM FFI, SLVM Native)

Any Frame / Object, that is not Foreign or Native, will not be simulated, and will be executed as if
it's assembly, not bytecode.

## ON MEMORY

### Blueprint Frames

Just like C function have a fixed stack size, SLVM frames have a fixed stack size. This allows to us
to precalculate the amount of memory a function call would take[^5]. This opens up windows for various
optimizations such as TCO.

### Dynamic Allocations

Dynamic allocations are done using arena allocators, and have zero overhead, unless they are Native / Foreign Objects.

[^1]: A multi-tiered JIT engine can be created from SLVM. However, any kind of code generation / JIT is out of scope of SLVM. SLVMJ, not yet realized, aims to accomplish this.
[^2]: SLVM aims avoid unsafe rust code. However the exceptions to this could include OS-interfacing code.
[^3]:
    SLVM is a single pass compiler. SLVM does not have to support complex grammars, since that's the
    responsibility of front End.

[^4]:
    SLVM computes memory required by each blueprint frame (function) during compile time.
    Therefore, allocations required by the VM are O(1). In some cases, SLVM can just allocate memory once and would never have to reallocate again. However, any dynamic allocations, by executing
    logic, may not be O(1).

[^5]:
    This includes the function calls the function call would make. Only exception is with recursive
    functions, unless they can be TCO'ed away.
