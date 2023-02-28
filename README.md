# simpleprofiler
Simple disk, network and database operations performance measurement tool

# Purpose
Simple utility to measure the performance the following operations:
* Writes to disk (MB writen per second)
* Reads from disk (MB read per second)
* http(s) transfer of a file to a server, which may either discard the contents or write them to a file (Mb/s)
* mysql database writes

Also may be used to perform database operations in an infinite loop in order to test resiliency scenarios (that
is, how does my database behave in case of disk failure, or node failure)
* Loop of deletion/insertion/queries
* Loops of queries only (for read-only databases)

# Local installation quick start

Requires go 18 or higher

Build the executable

```
go build
```

Execute a default test with a single instance as client and server



