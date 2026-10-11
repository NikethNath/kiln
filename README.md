# kiln

**An optimizing compiler for a small C subset, written in C++20.** kiln takes source code all the way to a native x86-64 executable through a real compiler pipeline:
- a Pratt parser
- a CFG intermediate representation
- **SSA construction**
- classic optimization passes
- **linear-scan register allocation**
- System V assembly

> **Status: in active development.** This README describes the planned design. Benchmarks and test counts are published only from real runs.

## Pipeline
```
source.kc → Lexer → Parser (recursive descent + Pratt) → AST → Sema
          → IR builder (CFG, locals in stack slots)
          → mem2reg (SSA: Cooper–Harvey–Kennedy dominators, dominance frontiers, phi insertion)
          → SCCP → SimplifyCFG → GVN → LICM → DCE        (IR verifier after every pass)
          → critical-edge splitting → phi elimination (parallel copies)
          → liveness → linear-scan register allocation
          → x86-64 assembly (System V ABI) → as + ld → native executable
```

## The language
A C subset: 64-bit wrapping `int`, fixed-size arrays, functions with up to 6 parameters, recursion, `if`/`while`/`for`/`break`/`continue` and short-circuit `&&`/`||`.
```c
int sieve[1024];

int count_primes(int n) {
    int count = 0;
    for (int i = 2; i < n; i += 1) {
        if (sieve[i] == 0) {
            count += 1;
            for (int j = i * i; j < n; j += i) sieve[j] = 1;
        }
    }
    return count;
}

int main() { print(count_primes(1000)); return 0; }
```

## How correctness is checked
- **Unit tests** (GoogleTest) for the lexer, parser, dominators, each pass and the register allocator
- **IR verifier** after every pass: terminators, phi/predecessor agreement, and SSA dominance
- **Golden programs** run at -O0, -O1 and -O2 with expected output
- **Differential testing against GCC:** randomly generated UB-free programs, compiled by kiln and by `gcc -fwrapv`, must produce identical output

## Stack
C++20, CMake + Ninja, GoogleTest, GCC/Clang, ASan/UBSan, GitHub Actions, and Python for the fuzzer and benchmark scripts.

## References
Cooper, Harvey & Kennedy, *A Simple, Fast Dominance Algorithm* · Cytron et al., *Efficiently Computing SSA Form* · Wegman & Zadeck, *Constant Propagation with Conditional Branches* · Poletto & Sarkar, *Linear Scan Register Allocation* · Cooper & Torczon, *Engineering a Compiler*

## License
MIT
