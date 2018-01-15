# HPK Archiver for Haemimont Engine game files
Supported games:
* Tropico 3-5
* Omerta: City of Gangsters
* Grand Ages: Rome
* Victor Vran

## Usage

### hpk help
```bash
$ hpk help
hpk

USAGE:
    hpk <SUBCOMMAND>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

SUBCOMMANDS:
    create     Create a new hpk archive
    extract    Extract files from a hpk archive
    list       List the content of a hpk archive
    print      Print information of a hpk archive
    help       Prints this message or the help of the given subcommand(s)
```

### hpk list
```bash
$ hpk list files/omerta/Packs/TextureLists.hpk
entities.lst
fallback.lst
fx.lst
misc.lst
sky.lst
terrains.lst
water.lst
```

### hpk create
```bash
$ hpk create -h
hpk-create
Create a new hpk archive

USAGE:
    hpk create <dir> <file>

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

ARGS:
    <dir>     input directory
    <file>    hpk output file
```

### hpk extract
```bash
$ hg extract -h
hpk-extract
Extract files from a hpk archive

USAGE:
    hpk extract [FLAGS] <file> <dest>

FLAGS:
        --force      Force extraction if destination folder is not empty
    -h, --help       Prints help information
    -V, --version    Prints version information
    -v               Verbosely list files processed

ARGS:
    <file>    hpk archive
    <dest>    destination folder
```

### hpk print
```bash
$ hpk print files/omerta/Packs/TextureLists.hpk
reading file: files/omerta/Packs/TextureLists.hpk
header:
  data_offset: 0x24
  fragments_residual_offset: 0x0
  fragments_residual_count: 0
  fragments_per_file: 1
  fragments_filesystem_offset: 0x1459
  fragments_filesystem_length: 64
filesystem entries: 8
filesystem fragments:
  0x13D1   len: 136
  0x24     len: 3876
  0xF48    len: 278
  0x105E   len: 82
  0x10B0   len: 134
  0x1136   len: 140
  0x11C2   len: 304
  0x12F2   len: 223
dir:  index=1 depth=0 ""
 fragment: 0x13D1 len: 136
file: index=2 depth=1 "entities.lst"
 fragment: 0x24 len: 3876
 compressed: inflated_length=30917 chunk_size=32768 chunks=1
  chunks: 0x10     len: 3860
file: index=3 depth=1 "fallback.lst"
 fragment: 0xF48 len: 278
 compressed: inflated_length=2956 chunk_size=32768 chunks=1
  chunks: 0x10     len: 262
file: index=4 depth=1 "fx.lst"
 fragment: 0x105E len: 82
 compressed: inflated_length=290 chunk_size=32768 chunks=1
  chunks: 0x10     len: 66
file: index=5 depth=1 "misc.lst"
 fragment: 0x10B0 len: 134
 compressed: inflated_length=344 chunk_size=32768 chunks=1
  chunks: 0x10     len: 118
file: index=6 depth=1 "sky.lst"
 fragment: 0x1136 len: 140
 compressed: inflated_length=536 chunk_size=32768 chunks=1
  chunks: 0x10     len: 124
file: index=7 depth=1 "terrains.lst"
 fragment: 0x11C2 len: 304
 compressed: inflated_length=2268 chunk_size=32768 chunks=1
  chunks: 0x10     len: 288
file: index=8 depth=1 "water.lst"
 fragment: 0x12F2 len: 223
 compressed: inflated_length=1978 chunk_size=32768 chunks=1
  chunks: 0x10     len: 207
```

## HPK File Format

### Header

| Offset | Size | Value                                         |
|--------|------|-----------------------------------------------|
| 0      | 4    | Magic number; `0x4C555042` (`"BPUL"`)         |
| 4      | 4    | Data offset; `0x24` (`36`)                    |
| 8      | 4    | Number of fragments per file; `1`, `8`        |
| 12     | 4    | Unknown; `0xFFFFFFFF`                         |
| 16     | 4    | Offset of the residual fragments in bytes     |
| 20     | 4    | Number of residual fragments                  |
| 24     | 4    | Unknown; `0x1`                                |
| 28     | 4    | Offset of the filesystem fragments in bytes   |
| 32     | 4    | Size of the filesystem fragments in bytes     |

### Fragment (Filesystem & Residual)

* Offsets of fragments are relative from the start of the file.
* The first filesystem fragment is the root directory.

| Offset | Size | Value                                     |
|--------|------|-------------------------------------------|
| 0      | 4    | Offset of a fragment in bytes             |
| 4      | 4    | Size of a fragment in bytes               |

### Fragment Type: Dir

| Offset | Size | Value                                     |
|--------|------|-------------------------------------------|
| 0      | 4    | Fragment Index; Index starts with `1`     |
| 4      | 4    | Fragment Type; File=`0x0` Dir=`0x1`       |
| 8      | 2    | Size of the following name in bytes       |
| 10     | ?    | Name data                                 |

### Fragment Type: File (zlib compressed)

Offsets of compressed chunks are relative from the start of a fragment.

| Offset | Size | Value                                         |
|--------|------|-----------------------------------------------|
| 0      | 4    | Magic number, `0x42494C5A` (`"ZLIB"`)         |
| 4      | 4    | Size of the inflated data in bytes            |
| 8      | 4    | Inflated chunk size, `0x0800` (`32768`)       |
| 12     | 4    | Chunk offset in bytes, `0x10` for one chunk   |
| *      | 4    | Additional chunk offsets (optional)           |
