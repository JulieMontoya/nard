# nard
Not A Real Disassembler

Not a real dissasembler for 6502 machine code, with an unashamed focus on the BBC Micro  (because that's the system I'm
targeting)  but probably adaptable to anything 6502-based.

It takes in a file of 6502 machine code  (which might well contain chunks of non-executable data interspersed with chunks
of executable code);  and generates a JSON file assigning temporary names to the labels it gathered from the code, and a
BeebAsm-compatible assembly language output file, which will assemble byte-for-byte identical to the original input file,
It can optionally read in a JSON file edited to give more meaningful names to the memory locations to which they refer,
and include those names in the assembly language output.

It does _not_ do any stateful analysis.  All it is doing is looking for a block of valid instructions, that ends with
**RTS**, **RTI**, **JMP** or **Bxx** (i.e., a conditional branch).  But _not_ **JSR**, because the next **RTS**
encountered will cause execution to resume at the next location. Only when a block of code ends cleanly, with a change
in the flow of execution, is it considered legitimate.

## OPTIONS

+ **-i _filename_** -- specifies the input filename.
+ **-l _load_addr_** -- specifies the load address.
+ **-e _exec_addr_** -- specifies the execution address.
+ **-g _gap_start_addr_,_gap_end_addr_** -- specifies the starting and after-ending addresses of a gap of non-code.
+ **-o _json_filename_** -- specifies a file to which to write label and gap definitions.
+ **-j _json_filename_** -- specifies a file from which to read label and gap definitions.
+ **-a _asm_filename_** -- specifies a file to which to write the assembly language output.
+ **-A _dfs_filename_** -- specifies the DFS filename within the eventual disc image.

The input file should be a raw 6502 machine code file, as extracted from a disc image.  Addresses can be
supplied in decimal, or prefixed with **0x**  (or **&**, but that must be escaped with a backslash or inside
speech marks).  Multiple gaps may be specified as a space-separated list in speech marks, such as
**-g "_gap_start_addr1_,_gap_end_addr1_ _gap_start_addr2_,_gap_end_addr2_ _gap_start_addr3_,_gap_end_addr3_ "**.
Note that gap ending addresses are actually the first address that is _not_ part of the gap  (i.e., like `*SAVE`).

## EXAMPLES

```
$ ./nard -i HELLO -l 0x900
```
This takes an input file called `HELLO` intended to load and execute at address hex 900.

```
$ ./nard -i HELLO -l 0x900 -g0x90e,0x91e -o hello-labels.json
```
This takes the same input file called `HELLO` intended to load and execute at address hex 900,
but creates a gap from hex 90e to  hex 91**d**  (remember, the address after the comma is the
first address _after_ the gap); and outputs a file `hello-labels.json` with the gap and
label definitions.

After editing the .json file with more meaningful label names  (this is the bit where you will
need to do some old-fashioned detective work), you can run
```
$ ./nard -i HELLO -l 0x900 -j hello-labels.json -a hello2.6502 -A HELLO2
$ beebasm -i hello.6502
```
The assembled code in the file `HELLO2` will be byte-for-byte identical with the
original input file `HELLO`.  You can prove this using the `hd` command to view a
hex dump of each file.  If you don't trust your eyeballs, run
```
$ diff <(hd HELLO) <(hd HELLO2)
```
and let the computer prove it.

The assembly language file can be further edited, and comments added
