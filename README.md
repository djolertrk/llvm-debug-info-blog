# llvm-debug-info-blog

Web Page: https://djolertrk.github.io/llvm-debug-info-blog/

## How to reduce a test case using LLVM bugpoint?

For example, the compilation of the huge-test.c crashes during the CodeGen/LiveDebugValues pass.

Steps:
  1. Convert the test written in a highlevel language to LLVM Assembly
                 
    $ clang -g -O2 huge-test.c -S -emit-llvm
  2. Confirm the bug still exists

    $ llc -O2 huge-test.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...
    
  3. Simplify the case

    $ bugpoint -llc-safe huge-test.ll
  Or additional arguments to the 'llc' tool could be passed as:

    $ bugpoint -llc-safe -safe-tool-args -mtriple=ppc-linux-gnueabi huge-test.ll

  If everything is OK, the 'bugpoint-reduced-simplified.bc' will be created.

  4. Disassemble the output to create 'bugpoint-reduced-simplified.ll' file

    $ llvm-dis bugpoint-reduced-simplified.bc
  
  5. Confirm again the problem still exists

    $ llc -O2 bugpoint-reduced-simplified.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...
  
  6. Simplify the case furthermore

    $ opt -S -strip-dead-prototypes --strip-nondebug bugpoint-reduced-simplified.ll > debug-info-bug-striped-case.ll
  
  7. 
    Confirm again the problem still exists

    $ llc -O2 debug-info-bug-striped-case.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...

