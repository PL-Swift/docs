# PL/Swift Documentation

**PL/Swift** allows you to write custom SQL functions and types
for the
[PostgreSQL](https://www.postgresql.org) database server
in the 
[Swift](http://swift.org/)
programming language.

## What is an PostgreSQL extension?

The PostgreSQL database server is quite modular and can be extended with
custom SQL functions and custom SQL types.

Such extensions can be written in PSQL itself,
in scripting languages such as Python or TCL,
and in plain C.

Now PL/Swift allows you to write extensions using Swift.

Assume you have a Swift function which turns an integer into a
[base36](https://en.wikipedia.org/wiki/Base36)
encoded String,
to replicate the trick used by URL shorteners
("`goo.gl/QvohfE`"):

```swift
func base36_encode(_ v: Int) -> String {
  return String(v, radix: 36)
}
```

and now you would like to use that in a SQL query, like so:

```sql
helge=# SELECT base36_encode(31337);
 base36_encode 
---------------
 o6h
(1 row)
```

This is what PL/Swift does. It helps you expose your Swift functions to
PostgreSQL.
