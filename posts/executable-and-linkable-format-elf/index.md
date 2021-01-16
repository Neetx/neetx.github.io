<!--
.. title: Executable and Linkable Format (ELF)
.. slug: executable-and-linkable-format-elf
.. date: 2021-01-16 16:29:26 UTC+01:00
.. tags: #reverse,#security,#malware,#linux,#elf
.. category: Reverse Engineering
.. link: 
.. description: 
.. type: text
-->

# Headers, sections and segments

ELF is a file format for executables file, Linux uses ELF files.

The components are:

- ELF Header
- Sections
- Segments

__________________________________________________________________________

## ELF Header

The ELF header is described by the type Elf32_Ehdr or Elf64_Ehdr, it 
information about the binary.

```c
/* The ELF file header.  This appears at the start of every ELF file.  */
#define EI_NIDENT (16)
typedef struct
{
  unsigned char  e_ident[EI_NIDENT];  /* Magic number and other info    */
  Elf32_Half	 e_type;             /* Object file type                */
  Elf32_Half     e_machine;         /* Architecture                     */
  Elf32_Word     e_version;         /* Object file version              */
  Elf32_Addr     e_entry;           /* Entry point virtual address      */
  Elf32_Off      e_phoff;           /* Program header table file offset */
  Elf32_Off      e_shoff;           /* Section header table file offset */
  Elf32_Word     e_flags;           /* Processor-specific flags         */
  Elf32_Half     e_ehsize;          /* ELF header size in bytes         */
  Elf32_Half     e_phentsize;       /* Program header table entry size  */
  Elf32_Half     e_phnum;           /* Program header table entry count */
  Elf32_Half     e_shentsize;       /* Section header table entry size  */
  Elf32_Half     e_shnum;           /* Section header table entry count */
  Elf32_Half     e_shstrndx;        /* Section header string table index*/
} Elf32_Ehdr;

typedef struct
{
  unsigned char   e_ident[EI_NIDENT];  /* Magic number and other info   */
  Elf64_Half      e_type;             /* Object file type               */
  Elf64_Half      e_machine;        /* Architecture                     */
  Elf64_Word      e_version;        /* Object file version              */
  Elf64_Addr      e_entry;          /* Entry point virtual address      */
  Elf64_Off       e_phoff;          /* Program header table file offset */
  Elf64_Off       e_shoff;          /* Section header table file offset */
  Elf64_Word      e_flags;          /* Processor-specific flags         */
  Elf64_Half      e_ehsize;         /* ELF header size in bytes         */
  Elf64_Half      e_phentsize;      /* Program header table entry size  */
  Elf64_Half      e_phnum;          /* Program header table entry count */
  Elf64_Half      e_shentsize;      /* Section header table entry size  */
  Elf64_Half      e_shnum;          /* Section header table entry count */
  Elf64_Half      e_shstrndx;       /* Section header string table index*/
} Elf64_Ehdr;
```

**e_ident[EI_NIDENT]** is a 16 bytes array that contains identification flags
about the file, these flags are used to interpret and decode the file
Flags:

- EI_MAG0-3    : ELF magic number
- EI_CLASS     : File class.
- EI_DATA      : File’s data encoding.
- EI_VERSION   : File’s version.
- EI_OSABI     : OS/ABI identification.
- EI_ABIVERSION: ABI version
- EI_PAD       : Start of padding bytes.
- EI_NIDENT    : Size of ei_ident.

Command to get elf header
```bash
readelf -h elf_name
```
__________________________________________________________________________

## Sections

Sections contains all the information for linking (link-time and not at
runtime).
There is a Section Header Table, an array of structures Elf64_Shdr (and
Elf32_Shdr for i386 ABI), there is an array entry for each section.

```c
/* Section header.  */
typedef struct
{
  Elf32_Word    sh_name;        /* Section name (string tbl index)      */
  Elf32_Word    sh_type;        /* Section type                         */
  Elf32_Word    sh_flags;       /* Section flags                        */
  Elf32_Addr    sh_addr;        /* Section virtual addr at execution    */
  Elf32_Off     sh_offset;      /* Section file offset                  */
  Elf32_Word    sh_size;        /* Section size in bytes                */
  Elf32_Word    sh_link;        /* Link to another section              */
  Elf32_Word    sh_info;        /* Additional section information       */
  Elf32_Word    sh_addralign;   /* Section alignment                    */
  Elf32_Word    sh_entsize;     /* Entry size if section holds table    */
} Elf32_Shdr;

typedef struct
{
  Elf64_Word    sh_name;        /* Section name (string tbl index)      */
  Elf64_Word    sh_type;        /* Section type                         */
  Elf64_Xword   sh_flags;       /* Section flags                        */
  Elf64_Addr    sh_addr;        /* Section virtual addr at execution    */
  Elf64_Off     sh_offset;      /* Section file offset                  */
  Elf64_Xword   sh_size;        /* Section size in bytes                */
  Elf64_Word    sh_link;        /* Link to another section              */
  Elf64_Word    sh_info;        /* Additional section information       */
  Elf64_Xword   sh_addralign;   /* Section alignment                    */
  Elf64_Xword   sh_entsize;     /* Entry size if section holds table    */
} Elf64_Shdr;
```

Some sections:

- .text   : code.
- .data   : initialised data.
- .rodata : initialised read-only data.
- .bss    : uninitialized data.
- .plt    : PLT (Procedure Linkage Table) (IAT equivalent).
- .got    : GOT entries dedicated to dynamically linked global variables.
- .got.plt: GOT entries dedicated to dynamically linked functions.
- .symtab : global symbol table.
- .dynamic: Holds all needed information for dynamic linking.
- .dynsym : symbol tables dedicated to dynamically linked symbols.
- .strtab : string table of .symtab section.
- .dynstr : string table of .dynsym section.
- .interp : RTLD embedded string.
- .rel.dyn: global variable relocation table.
- .rel.plt: function relocation table.

Command to get section headers
```bash
readelf -S elf_name
```
__________________________________________________________________________

## Segments

Also called Programm Headers, segments split the binary into chunks, they
aren't useful at linktime.
There is a Program Header Table, an array of Elf64_Phdr (or Elf32_Phdr for
i386 ABI), there is an array entry for each segment.

```c
/* Program segment header.  */

typedef struct
{
  Elf32_Word        p_type;                 /* Segment type             */
  Elf32_Off         p_offset;               /* Segment file offset      */
  Elf32_Addr        p_vaddr;                /* Segment virtual address  */
  Elf32_Addr        p_paddr;                /* Segment physical address */
  Elf32_Word        p_filesz;               /* Segment size in file     */
  Elf32_Word        p_memsz;                /* Segment size in memory   */
  Elf32_Word        p_flags;                /* Segment flags            */
  Elf32_Word        p_align;                /* Segment alignment        */
} Elf32_Phdr;

typedef struct
{
  Elf64_Word        p_type;                 /* Segment type             */
  Elf64_Word        p_flags;                /* Segment flags            */
  Elf64_Off         p_offset;               /* Segment file offset      */
  Elf64_Addr        p_vaddr;                /* Segment virtual address  */
  Elf64_Addr        p_paddr;                /* Segment physical address */
  Elf64_Xword       p_filesz;               /* Segment size in file     */
  Elf64_Xword       p_memsz;                /* Segment size in memory   */
  Elf64_Xword       p_align;                /* Segment alignment        */
} Elf64_Phdr;

```

**p_type** common values are:

- PT_NULL   : unassigned segment (first entry of Program Header Table)
- PT_LOAD   : Loadable segment
- PT_INTERP : Segment holding .interp section
- PT_TLS    : Thread Local Storage segment (Common in static binaries)
- PT_DYNAMIC: Holding .dynamic section

**important**: Only PT_LOAD segments will be loaded in memory, the others
are mapped in the memory range of one of the loaded segments.

Command to get segment/program header:
```bash
readelf -I elf_name
```
__________________________________________________________________________

So sections contain information about linking and Program Headers split
binary into segments.

Segments have offsets and virtual addresses that must be congruent modulo
the sytem page size, and p_align must be a multiple of the system page
size.

With this alignment the segment mapping can't overlap in a single memory
page. A problem for the overlap are different access attributes, these 
cannot be forced if two segments are mapped in the same memory page.

__________________________________________________________________________

References:
[intezer](https://www.intezer.com/blog/research/executable-linkable-format-101-part1-sections-segments/)

