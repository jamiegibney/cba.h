# `cba.h`

For when you CBA with CMake.

This is a [stb-style header-only library](https://github.com/nothings/stb)
designed for writing build recipes in C, and for useful C utilities.

The idea is chiefly based on tsoding's [nob.h
library](https://github.com/tsoding/nob.h), but with a variety changes that I
either prefer or which I'm experimenting with.

Grab only the header with:

```sh
wget https://raw.githubusercontent.com/jamiegibney/cba.h/refs/heads/master/cba.h
```

## Usage

Add `#define CBA_IMPLEMENTATION` before including `cba.h` in ONE C/C++ file to
generate the implementation:

```c
#include ...
#include ...
#define CBA_IMPLEMENTATION
#include "cba.h"
```

And otherwise just `#include "cba.h` for its definitions.

### Simple example

```c
// File: cba.c

#define CBA_VERBOSE          // See internal logging.
#define CBA_PRINT_ON_REBUILD // Print a message when rebuilding.
#define CBA_IMPLEMENTATION   // Generate implementations.
#include "cba.h"

int main(int argc, char** argv) {
    // Allow the program to rebuild itself when modified.
    CBA_REBUILD(argc, argv);

    // Recursively create a directory.
    assert(try_mkdir("build/artefacts"), "failed to create build directory");

    // A command is an array of arguments which can be run via the shell.
    Command cmd = {0};

    // Use the CBA_COMPILER_* macros for compiler-specific flags.
    cmd_append(&cmd,
        CBA_COMPILER_C,
        CBA_COMPILER_DEBUG_FLAGS,
        CBA_COMPILER_COMMON_FLAGS,
        CBA_COMPILER_OUTPUT("build/artefacts/main"),
        CBA_COMPILER_INPUTS("main.c"),
    );

    // With GCC, the above forms:
    //   gcc -ggdb -DDEBUG -Wall -Wextra -o build/artefacts/main main.c
    //
    // And with MSVC:
    //   cl.exe /ZI /DDEBUG /W4 /nologo /Fe:build/artefacts/main main.c

    // Run the command, block until it terminates, and assert that it exits normally.
    cmd_run(cmd);

    return 0;
}
```

Compile the above once with, e.g.:
```sh
gcc -o cba cba.c
```

And when the source file is modified, the program will automatically rebuild itself when run.

## Other utilities

`cba.h` also provides various utilties for C programs, including an arena
allocator, file operations, string operations, and more.

- See the `alloc`, `alloc_bytes`, and `alloc_array` macros for allocating via the arena.
- See `begin_temp_memory()` and `end_temp_memory()` for temporary arena allocations.
- See the `str_*` functions for string operations.
- See the `str_arr_*` functions for string arrays.
- See the `file_*` functions for file operations.
- See the `print`, `assert`, and `panic` macros for printing/assertions/panics.
- See the `info`, `warn`, and `error` macros for user-facing logs.

## Todo

- [ ] Flesh out Windows support
- [ ] Dynamic array (and perhaps a #define for enabling dynamic allocation)
- [ ] Hash table/set
- [ ] Piping mechanism
- [ ] Optional function prefixes (perhaps with `#define CBA_STRIP_PREFIXES`)

<!--
## API

### Files

```c
b32 file_move(const char* path, const char* new_path);
```

Moves (renames) a file to a new path.



```c
b32 file_copy(const char* path, const char* new_path, b32 symbolic_link);
```

Copies a file to a new path, optionally creating a symbolic link. On Windows,
trying to create a symbolic link will have no effect.



```c
b32 file_delete(const char* path);
```

Deletes a file. If the file is a directory, it will be removed recursively.



```c
b32 file_exists(const char* path);
```

Whether a file exists.



```c
FileType file_get_type(const char* path);
```

Returns the type of a file:
- `FILE_TYPE_UNKNOWN`: the file type could not be detected.
- `FILE_TYPE_FILE`: the file is a regular file.
- `FILE_TYPE_DIRECTORY`: the file is a directory.
- `FILE_TYPE_SYMLINK`: the file is a symbolic link.
- `FILE_TYPE_OTHER`: the file type was detected, but is not covered.



```c
b32 try_mkdir(const char* path);
```

Attempts to create a directory, returning `true` if successful or if the
directory already exists. If `path` describes nested directories which don't
exist, it will create all of them.



```c
i32 files_need_rebuild(String output_path, StringArray input_paths);
i32 file_need_rebuild(String output_path, String input_path);
```

Whether the input file(s) have been modified sooner than the output file.
Returns `1` if true, `0` otherwise, and `-1` in case of an error.

-->
