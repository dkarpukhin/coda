CODA => CO(re) D(ump) A(nalyzer) or C(ore) O(bject) D(ebugger) and A(nalyzer)
--------

This tool probes _Linux-x86\_64_ coredumps in batch mode as well as interactively.
coredump has to be full(not a data/stack segment dump only).
If the coredump is not full, backtraces will be curtailed.
This tool does not require binaries, only full core dump is required.

## Architecture supported
    compile on x86_64 Linux box.

## How to enable full coredump in linux

On bash prompt:

    $> cat /proc/$$/coredump_filter 
    00000033

    $> echo 0x7f > /proc/$$/coredump_filter 
    $> cat /proc/$$/coredump_filter 
    0000007f

For furter info 'man core'.

## Compilation
    $> apt-get install binutils-dev #for opcodes library
    $> make

Help
------
    $> ./coda
    Argument Missing, provide (corefile)

    Usage: ./coda <options> corefile
    options:
      -B Backtrace all
      -m Show, if mini-coredump
      -d Enable de-mangling
      -i Interactive mode

    $> ./coda -i core.segfault.full 
    Welcome to coda interactive command line.
    THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND.
    Supported on x86_64-linux.
    Coredump generated by command line => ./segfault_thr 
    coda > help
    help       - Help for all interactive commands.
    demangle   - demangle 0|1
    dis        - Show disassembly of the function.
    btregs     - Show backtrace of the selected thread with registers.
    bt         - Show backtrace of the selected thread.
    mmap       - Show memory map.
    thread     - thread <number>.
    threads    - Show all threads.
    strings    - Extract strings from coredump under probe.
                     Usage: strings <pattern>
    xuint      - Extract unsigned int.
                     Usage: xuint addr <count>
    xchar      - Extract char.
                     Usage: xchar addr <count>
    xulong     - Extract unsigned long.
                     Usage: xulong addr <count>
    xint       - Extract int.
                     Usage: xint addr <count>
    xlong      - Extract long.
                     Usage: xlong addr <count>
    xaddr      - Extract address.
                     Usage: xaddr addr <count>
    q          - Exit program.


###For better backtrace, compile the binary with -rdynamic compiler flag and then generate the coredump.

Some time back I posted an answer on stackoverflow in response to a question - [get-backtrace-from-core-file-without-gdb](http://stackoverflow.com/questions/22461254/get-backtrace-from-core-file-without-gdb).

##How to extract backtrace from core dump 
 
Getting a backtrace from a core file is very much architecture and OS specific, if you plan to extract it all by yourself. 
gdb is a portable program which works on multiple architectures. 

###gdb uses two techniques to extract the backtrace from coredump - 
    
* Following the link list of frame pointer (ebp in case of x86 architecture). 
* Using the exception handling frame work to do virtual unwinding. gdb requires binaries to extract the backtrace from coredump. 
   Since binaries contains information, how to start the virtual unwinding.

###After this point my answer is specific to linux-x86_64 architecture.

Using the frame pointer i.e. rbp does not work when it comes to extracting backtrace reliably 
if frame pointer is omitted or it is optimized program. On _x86\_64 architecture, compilers e.g. gcc_ emits 
_.eh\_frame_ section which is used by gdb to do _virtual unwinding_(remember I am not talking about source level debugging of gdb). But gdb requires binaries to do virtual unwinding since gdb assumes core is without executable contents since, on linux box, kernel by default dumps only data segments.
