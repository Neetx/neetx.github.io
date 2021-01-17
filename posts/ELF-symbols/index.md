<!--
.. title: ELF symbols
.. slug: ELF-symbols
.. date: 2021-01-17 21:26:58 UTC+01:00
.. tags: #elf, #linux, #reverse, #malware, #security
.. category: Reverse Engineering
.. link: 
.. description: 
.. type: text
-->

Function and variable names are called symbols, they must be translated 
into offsets and addresses at compilation time.
The compiler exports **symbolic information** into the output object, that
holds symbols and interfaces to them (local, external).

Linkers and debuggers works with symbols accessing the interfaces and the
Symbol Table.
Debugging is possible, but harder, without symbols, as in a stripped elf.
It's possible to remove symbol tables or its entries

__________________________________________________________________________

## ELF symbol

In an ELF file there is a Symbol Table (max two) as an array of structures
called Elf32_Sym or Elf64_Sym

```c
/* Symbol table entry.  */
typedef struct
{
    Elf32_Word      st_name;          /* Symbol name (string tbl index) */
    Elf32_Addr      st_value;         /* Symbol value                   */
    Elf32_Word      st_size;          /* Symbol size                    */
    unsigned char   st_info;          /* Symbol type and binding        */
    unsigned char   st_other;         /* Symbol visibility              */
    Elf32_Section   st_shndx;         /* Section index                  */
} Elf32_Sym;

typedef struct
{
    Elf64_Word     st_name;           /* Symbol name (string tbl index) */
    unsigned char  st_info;           /* Symbol type and binding        */
    unsigned char  st_other;          /* Symbol visibility              */
    Elf64_Section  st_shndx;          /* Section index                  */
    Elf64_Addr     st_value;          /* Symbol value                   */
    Elf64_Xword    st_size;           /* Symbol size                    */
} Elf64_Sym;
```

__________________________________________________________________________

### st_name

If **st_name** isn't unitialized the the symbol hasn't a name.

### st_info

In **st_info** there are binding attributes (linkage visibility and
behavior when the symbol is externally referenced)
Common binding:

- STB_LOCAL : invisible from outside
- STB_GLOBAL: visible from outside
- STB_WEAK  : global symbol but it can be redefined

Common types:

- STT_NOTYPE : type not specified
- STT_OBJECT : symbol is a data object (variable).
- STT_FUNC   : symbol is a code object (function).
- STT_SECTION: symbol is a section.

To get these fields a set of bitmasks are used

- ELF64_ST_BIND(info) ((info) >> 4 )
- ELF64_ST_TYPE(info) ((info) & 0xf)

### st_other

Information about symbol visibility, how a symbol can be accessed after it
become part of an object:

- STV_DEFAULT  : default visibility, it is equal to symbol’s binding type
- STV_PROTECTED: it is visible by other objects, but cannot be preempted
- STV_HIDDEN   : invisible
- STV_INTERNAL : internal visiblity

**Symbol binding** refers to an external object referencing a symbol at
linking time
**Symbol visibility** refers on the original object and to an external
object referencing the symbol of interest, at runtime

Macro to get visibility information: 

- ELF64_ST_VISIBLITY(o) ((o) & 0x3)

### st_shndx

Every symbol is associated with a section, **st_shndx** contains its index
in the Section Header Table

Common section type:

- SHT_UNDEF: section is not present in the object. The symbol is imported
- SHT_PROGBITS: section is defined in current object
- SHT_SYMTAB, SHT_DYNSYM: Symbol Table (.symtab, .dynsym)
- SHT_STRTAB, SHT_DYNSTR:  String Table (.strtab, .dynstr)
- SHT_REL: Relocation Table without explicit addends (.rel.dyn, .rel.plt)
- SHT_RELA: Relocation Table with explicit addends (.rela.dyn, .rela.plt)
- SHT_HASH: Hash Table for dynamic symbol resolution (.gnu.hash)
- SHT_DYNAMIC: section holding Dynamic Linking information (.dynamic)
- SHT_NOBITS: section takes no space in disk (.bss)

### st_value

Symbol value for an entry

- For ET_REL files this value represents a section offset 
- For ET_EXEC or ET_DYN files this value represents a virtual address
  If it is equal to 0 and the section pointed by st_shndx has a sh_type
  equal to SHT_UNDEF, then this symbol is imported (so its value is
  resolved at runtime by RTLD(ld.so))

### st_size

**st_size** contains symbol's size

__________________________________________________________________________


##String Table

In an ELF are commont two Symbol Tables: .symtab and .dynsym.
The .symtab is the global Symbol Table of the object, it has a Sting Table
called .strtab (they have the same number of entries).

The .dynsym Symbol Table contains symbols for Dynamic Linking, its strings
are in the .dynstr String Table

__________________________________________________________________________

A **stripped** binary hasn't .symtab and .strtab, so reversing and
debugging is harder, but some symbol can be retrieved by the .dynsym
located at DT_SYMTAB entry in the PT_DYNAMIC segment.

Dynamically linked binaries can be stripped but .dynsym and .dynstr cannot
be removed

__________________________________________________________________________

References:

- [intezer](https://www.intezer.com/blog/elf/executable-linkable-format-101-part-2-symbols/)

