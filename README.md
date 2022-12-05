# elfcopy

A tool for copying ELF binaries. The most notable feature is the implementation
of the `--redefine-syms` from objcopy that allows for renaming dynamic symbols
in a shared library.

99% of this code comes from the object crate's "elfcopy"
[example](https://github.com/gimli-rs/object/blob/master/crates/examples/src/bin/elfcopy.rs).
As such, most of the credit goes to the original author, philipc.

### Usage

```
# Usage: target/debug/elfcopy [--redefine-syms <file>] <infile> <outfile>
#
$ cargo run --redefine-syms mangle-list.txt libfoo.so liboutput.so
```

Where mangle-list.txt is of the form:
```
original_name new_name
original_name1 new_name1
original_name2 new_name2
...
```

## Redefining symbols in a shared library

Renaming symbols in dynamic shared objects is difficult, there are a couple
things that have to be done:
* Actually rename the symbol, updating the `.dynstr` section and setting the
  new offset in the symbol's entry in `.dynsym`. (medium difficulty)
* Regenerate the hashes for dynamic symbols (mostly easy)
* Recalculate all offsets (very hard)

Tools such as objcopy and ELF toolkits usually only implement reading dynamic
ELF objects due to the difficulty of the third point, and writing or modifying
is not allowed. The authors of the object crate bailed the rest of us out by
actually having support for writing DSOs, and the elfcopy example was created
to demonstrate this.

The base elfcopy reads a binary into a series of `in_*` variables that contain
the string/symbol tables, section information, etc. These are then used to initialize
`out_*` variables in the `copy_file` function to construct a copy of the data. The
`out_*` data is then used to generate a valid ELF binary, including recalculating
all memory offsets as part of generating.

The redefinition step occurs while filling in the `out` entries from the `in` data.
Symbols are looked up in a table to find their redefined name, and that is
inserted into `out` instead of the original name.
