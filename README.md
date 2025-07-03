## Bulletproof C: A Style Guide for Extremely Secure Code

**Philosophy:**

This guide defines a subset of C (using C23 standards) intended for
mission-critical systems where failure is unacceptable. The goal is
**zero bugs** through simplicity, predictability, and rigorous
verification. Clarity trumps cleverness. Code is written primarily
for human understanding and secondarily for the machine. Data structures
are fundamental; get them right, and the code often becomes self-evident.
Every rule exists to eliminate potential errors and enhance static and
dynamic analysis.

**Core Rules:**

1.  **Simplicity is Paramount:**

    - **Function Size:** Functions *must not* exceed 60 lines of code (excluding comments and blank lines). If a function is longer, refactor it.

    - **Control Flow:** Use only simple constructs.

      - **Forbidden:** goto, setjmp, longjmp, recursion (direct or indirect).

      - **Loops:** All loops *must* have a fixed, statically verifiable upper bound. Prove this bound is never exceeded. Infinite loops (e.g., schedulers) must be proven to *never* terminate unintentionally.

      - **Branching:** Prefer branchless code *only* if it maintains clarity and simplicity. Otherwise, use simple if/else or switch.

2.  **Memory Safety is Non-Negotiable:**

    - **Allocation:** Use only static memory or dedicated memory arenas allocated *before* the main application logic begins. No dynamic allocation (malloc, calloc, realloc, free) after initialization.

    - **Bounds Checking:** *All* memory access (arrays, pointers) *must* be preceded by explicit bounds checks.

    - **Check Placement:** Bounds checks *must* be the first executable statements within a function for all relevant parameters and indices. Refactor code aggressively to meet this requirement.

    - **Pointers vs. Indices:** Strongly prefer array indices over pointers.

    - **Pointer Restrictions:**

      - Limit pointer dereferencing to a *single* level (\*p is okay, \*\*p is forbidden).

      - Do not hide dereferences in macros or typedefs.

      - Pointer arithmetic is forbidden except implicitly via array indexing (a\[i\]).

      - Function pointers are forbidden.

3.  **Data Handling & Integrity:**

    - **Scope:** Declare variables at the smallest possible level of scope.

    - **Initialization:** Initialize all variables before use.

    - **Return Values:** Check the return value of *every* non-void function. If a return value is intentionally ignored, explicitly cast it to (void) and add a comment justifying why.

    - **Parameters:** Check the validity of parameters inside each function (e.g., non-null pointers, valid enum ranges) *after* bounds checks.

    - **Assertions:** Use assertions liberally (aim for \>= 2 per function) to verify pre-conditions, post-conditions, invariants, and assumptions. Assertions *must* be side-effect free. A failed assertion indicates an unrecoverable error state.

4.  **Language & Preprocessing:**

    - **C Standard:** Use C23. Leverage safety-enhancing features where available (e.g., \[\[nodiscard\]\], nullptr).

    - **Preprocessor:** Limit preprocessor use strictly to:

      - Header file inclusion (#include).

      - Simple, constant-like macro definitions (#define MAX_VAL 100).

      - Standard include guards.

      - **Forbidden:** Token pasting (##), variadic macros (\...), recursive macros, complex conditional compilation (#if/#ifdef beyond include guards or essential platform differences -- justify any use). Macros must expand to complete syntactic units.

**Tooling & Process:**

1.  **Compilation:**

    - Compile with *all* warnings enabled at the highest/most pedantic level (e.g., -Wall -Wextra -pedantic -Werror for GCC/Clang).

    - Treat *all* warnings as errors. Code *must* compile completely clean. Rewrite code to satisfy the compiler/analyzer, even if you suspect a false positive.

2.  **Static Analysis:**

    - Integrate *at least one* state-of-the-art static analyzer (e.g., Clang Static Analyzer, PVS-Studio, Cppcheck) into the development process.
    - Run analysis daily (or on commit).
    - Code *must* pass with zero warnings.

3.  **Dynamic Analysis:**

    - *Always* build and run tests with AddressSanitizer (ASAN) enabled.
    - Fix all reported issues immediately.

4.  **Build System:**

    - Avoid build systems.
    - Use well-documented Unity Builds with manual compilation.

5.  **Debugging:**
  
    - Code structure and rules should facilitate easy debugging.
    - Avoid complex expressions or side effects that obscure state.

8.  **Documentation:**

    - Not meaningfully seperated from code.
    - Must be simple, correct, consistant, and complete.
    - Failure to meet any of these standards is treated as a bug.
