# Hello World

But lets do a simple `base36` module for demonstration purposes.

## Setup Module

Setup a new directory and initialize it as an PostgreSQL extension:

```
$ mkdir base36 && cd base36
$ swift pl init
The Swift PostgreSQL build environment looks sound.

  module:    base36
  config:    debug
  product:   /Users/helge/trump/base36/.build/base36.so
  pg_config: /Applications/Postgres.app/Contents/Versions/9.4/bin/pg_config
  PL/Swift:  /usr/local
```

This creates a Swift Package Manager module and places relevant extension
files into it:

```
$ tree
.
├── Package.swift
├── Sources
│   ├── base36-ext.swift
│   └── base36.swift
├── base36--0.0.1.sql
└── base36.control

1 directory, 5 files
```

The `Package.swift` just loads the PL/Swift wrapper module we provide:

```swift
import PackageDescription

let package = Package(
    name: "base36",

    dependencies: [
      .Package(url: "https://github.com/PL-Swift/PLSwift.git", 
               majorVersion: 0)
    ]
)
```

The `base36.swift` source file contains a a sample Swift SQL function. The
file can be named anything (but main.swift, which would produce a tool instead
of a library ;-)

```swift
import Foundation

func hello() -> String {
  return "Hello Schwifty World!"
}
```

The `base36-ext.swift` file contains all the boilerplate to export the
Swift function towards PostgreSQL:

```swift
import Foundation
import CPLSwift
import PLSwift

// MARK: - Hello Function Declaration

/*
 * To add more functions, you need to:
 * - add a main `func xyz(fcinfo: FunctionCallInfo) -> Datum`
 * - add a mandatory ABI function (just returns PG_FUNCTION_INFO_V1)
 * - load the function in the base36.sql file
 *
 * The functions need proper "C names", so that PostgreSQL can find them. The
 * names are assigned using the `@_cdecl` attribute.
 */

@_cdecl("pg_finfo_base36_hello")
public func hello_abi() -> UnsafeRawPointer { return PG_FUNCTION_INFO_V1 }

@_cdecl("base36_hello")
public func hello(fcinfo: FunctionCallInfo) -> Datum {
  return hello().pgDatum
}

// MARK: - PostgreSQL Extension Marker

@_cdecl("Pg_magic_func") public func PG_MAGIC_BLOCK() -> UnsafeRawPointer {
  return PGExtensionMagicStruct
}
```

### Intermission: `@_cdecl`

PostgreSQL is written in C and when loading objects, expects the exported
functions, data, etc to be in "C-style" (conforming to the platforms C ABI).
While Swift interoperates with C ABI code just fine,
it does not use the C ABI, but - like C++ - "mangles" the names. This is to
support type overloading, module, etc - the details don't matter here.

The important thing is that Swift functions need to be given a "C name" to be
accessible by a C caller like PostgreSQL.
This is what the `@_cdecl` function attribute is good for.
Consider this:

    @_cdecl("base36_hello") func hello(...) {}

Within the Swift module, the name of the function is just `hello`.
In the compiled Swift binary this becomes something like (run `nm -gU`
on the shared library to check that):

    __T006base365helloSuSpySC20FunctionCallInfoDataVG6fcinfo_tF

This can't be consumed by PostgreSQL.
By adding the `@_cdecl("base36_hello")`, the binary will also contain a plain
reference:

    _base36_hello

Which is what PostgreSQL will find and load.

### Explanation of the Source

The boilerplate contains the things exported by the extension using the
C/PG ABI. When PostgreSQL loads the dynamic object, it will use the
`dlsym` family of functions to locate entry points into the extension.

There is one 'marker' entry point, the `PG_MAGIC_BLOCK`. This has to be declared
exactly once in the extension and tells PG that this is indeed a valid
PostgreSQL extension, plus the PostgreSQL API the module was compiled against
etc:

		@_cdecl("Pg_magic_func") public func PG_MAGIC_BLOCK() -> UnsafeRawPointer {
		  return PGExtensionMagicStruct
		}

In addition to that, there are *two* functions for each SQL function, the
actual function which is called by PostgreSQL when the function is used
from within PostgreSQL:

		@_cdecl("base36_hello")
		public func hello(fcinfo: FunctionCallInfo) -> Datum

and an "ABI version function". The ABI version function is always the same,
and PostgreSQL only supports this one ABI v1. Declaring the function is still
mandatory:

    @_cdecl("pg_finfo_base36_hello")
    func hello_abi() -> UnsafeRawPointer { return PG_FUNCTION_INFO_V1 }

Notice that the C name is the same like the actual function name, but
with a `pg_finfo_` prefix. Just return the `PG_FUNCTION_INFO_V1` constant.

Back to the actual function, it receives a `FunctionCallInfo` pointer.
The struct this is pointing to, contains the arguments the SQL function was
called with and a few more things.
For example if the first argument is an int, it can be extracted like this:

    let firstArgument = fcinfo.pointee[int: 0]

The function returns a `Datum`. `Datum` is an opaque PostgreSQL type which can
carry all kinds of values. Same thing like a Swift `Any` essentially.
`PLSwift` contains a protocol which can turn various Swift types into `Datum`
values. For example to return a SQL TEXT, you can simply do this:

    return "Hello".pgDatum

That is what the boilerplate does. We recommend to keep it in a separate file
from the actual implementation of the function.

## Build Module

Now that we looked at the source, lets build the module:
`swift pl build` first invokes `swift pl` and subsequently converts the
build results into an PostgreSQL extension shared library (`base36.so`).

```
$ swift pl build
Fetching https://github.com/PL-Swift/PLSwift.git
Fetching https://github.com/PL-Swift/CPLSwift.git
Cloning https://github.com/PL-Swift/PLSwift.git
Resolving https://github.com/PL-Swift/PLSwift.git at 0.0.4
Cloning https://github.com/PL-Swift/CPLSwift.git
Resolving https://github.com/PL-Swift/CPLSwift.git at 0.0.1
Compile Swift Module 'PLSwift' (3 sources)
Compile Swift Module 'base36' (2 sources)

$ ls -hl .build/base36.so
-rwxr-xr-x  1 helge  staff    74K Jan  6 14:52 .build/base36.so
```

## Install Module

The `swift pl install` command will install the extension into your local
PostgreSQL server.

That is, it will copy the build binary extension,
the control file
and the SQL registration file into your PostgreSQL server.

```
$ swift pl install
```

## Load Module into PostgreSQL

Once you installed the module, you can simply load it into your PostgreSQL
like that:

    CREATE EXTENSION "base36";

and then call functions, like for example:

    SELECT * FROM base36_hello();

When you are in `psql`, you can use `\df` to list the registered functions.
