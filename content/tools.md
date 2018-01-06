# PL/Swift Tools

PL/Swift comes with a set of CLI tools that integrate with the 
Swift Package Manager.
The tools enhance the Swift Package Manager to be able to build native
PostgreSQL extensions, configure them and run them.
All of them are invoked like:

    swift pl <subcommand>

If the toolname is omitted, you get a small help:

```
$ swift pl
usage: swift pl <subcommand>

Available subcommands are:
   init      Setup directory as a Swift PostgreSQL Package.
   build     Build Swift Package as a PostgreSQL loadable module.
   install   Install module into PostgreSQL server.
   validate  Check PostgreSQL build environment.

Try 'swift pl <subcommand> help' for details.
```

## swift pl init

Prepare a directory as a Swift PostgreSQL extension.

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

This creates a set of files:
- the control file for the package
- the SQL file to register the package functions w/ PostgreSQL
- a boilerplate file which contains the C function registration
- a file where the actual Swift functions are kept

## swift pl build

`swift pl build` first invokes `swift build` and subsequently converts the
build results into an PostgreSQL extension shared library.

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

## swift pl install

The `swift pl install` command will install the extension into your local
PostgreSQL server.

That is, it will copy the build binary extension,
the control file
and the SQL registration file into your PostgreSQL server.

```
$ swift pl install
```

## swift pl validate

`swift pl validate` just checks whether a build is likely to be successful
and prints out the configuration assumptions it has.

```
$ swift pl validate
The Swift PostgreSQL build environment looks sound.

  srcroot:   /Users/helge/trump/base36
  module:    base36
  config:    debug
  product:   /Users/helge/trump/base36/.build/base36.so
  version:   0.0.1
  sql-setup: base36--0.0.1.sql
  pg_config: /Applications/Postgres.app/Contents/Versions/9.4/bin/pg_config
  moddir:    /Applications/Postgres.app/Contents/Versions/9.4/lib/postgresql
  extdir:    /Applications/Postgres.app/Contents/Versions/9.4/share/postgresql/extension/
  PL/Swift:  /usr/local
  swift:     4.0.3
```
