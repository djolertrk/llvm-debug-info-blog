# llvm-debug-info-blog

Web Page: https://djolertrk.github.io/llvm-debug-info-blog/

## How to reduce a test case using LLVM bugpoint?

For example, the compilation of the huge-test.c crashes during the CodeGen/LiveDebugValues pass.
Please find below the steps to reduce the bug.

  Convert the test written in a highlevel language to LLVM Assembly
                 
    $ clang -g -O2 huge-test.c -S -emit-llvm
  
  Confirm the bug still exists

    $ llc -O2 huge-test.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...
    
  Simplify the case

    $ bugpoint -llc-safe huge-test.ll
  
  Or additional arguments to the 'llc' tool could be passed as:

    $ bugpoint -llc-safe -safe-tool-args -mtriple=ppc-linux-gnueabi huge-test.ll

  If everything is OK, the 'bugpoint-reduced-simplified.bc' will be created.

  Disassemble the output to create 'bugpoint-reduced-simplified.ll' file

    $ llvm-dis bugpoint-reduced-simplified.bc
  
  Confirm again the problem still exists

    $ llc -O2 bugpoint-reduced-simplified.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...
  
  Simplify the case furthermore

    $ opt -S -strip-dead-prototypes --strip-nondebug bugpoint-reduced-simplified.ll > debug-info-bug-striped-case.ll
  
  Confirm again the problem still exists

    $ llc -O2 debug-info-bug-striped-case.ll
    ... Assertion 'isTrue(...'
    Stack dump:
    0. Program arguments ...

