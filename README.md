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

### -i _filename_

Specifies the input filename.  This should be a raw 6502 machine code file, as extracted from a
disc image.

### -l _load_addr_

Specifies the load address.  This can be supplied in decimal, or in hex if prefixed with **0x**  (or
**&**, but that would have to be escaped with a backslash or inside speech marks).

### -e _exec_addr_

Specifies the execution address.

### -g _gap_start_addr_,_gap_end_add_

Specifies the starting and after-ending addresses of a gap of non-code  (i.e., text, or a table
of data or addresses).    Multiple gaps may be specified as a space-separated list in speech marks,
such as
`-g "_gap_start_addr1_,_gap_end_addr1_ _gap_start_addr2_,_gap_end_addr2_ _gap_start_addr3_,_gap_end_addr3_ "`
Note that gap ending addresses are actually the first address that is _not_ part of the gap  (i.e.,
just like `*SAVE`).

### -o _json_filename_

Specifies a file to which to write label and gap definitions, in JSON format.  This saves the effort
of repeatedly entering multiple gaps, and can be edited to change the temporary labels  (of the form
`tl090d`, i.e. the letters "tl" for "temporary label" and its hexadecimal address)  to something more
meaningful as deduced from studying the code.

### -j _json_filename

Specifies a file from which to read label and gap definitions.

### -a _assembly_filename_

Specifies a file to which to write the assembly language output.

### -A _dfs_filename_

Specifies the filename to use in the `SAVE` statement in the assembly language output
file.  Note that if BeebAsm's `-do` option is used to create a disc image, this filename
will become the DFS filename within that disc image; so in this case, it must be a valid
DFS filename.  Otherwise, if `-do` is not given, it will be saved in the default folder.
The default `SAVE` name is based on the input filename, but with the extension ".rec" for
"recreation".

## EXAMPLES

The file `HELLO`, as included above with its BeebAsm source code `hello_world.6502`, is
used for these examples.

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

The assembly language file can be further edited, and comments added to suit.

# Further Work

The assembly language output of non-code sections needs to be tidied up.  At present, every
byte is specified individually with `EQUB`.  This will eventually be improved by gathering
up strings of printable characters into `EQUS` statements.

