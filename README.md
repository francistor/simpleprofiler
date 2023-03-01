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

```
# The -sync option will force a flush per slice written to file. Writting performance is typically one order of mangnitude poorer
./simpleprofiler -client -server [-sync]
```

A typical output is like this

```
francisco@h2:~/simpleprofiler$ ./simpleprofiler -client -server -sync
[INFO] writing to file /tmp/simpleprofiler/profilerfile
[INFO] listening in port 8080
[INFO] using http
[INFO] sending data to http:/localhost:8080
-------------------------------------------------
starting test
-------------------------------------------------
[INFO] existing file /tmp/simpleprofiler/profilerfile.server deleted
[INFO] existing file /tmp/simpleprofiler/profilerfile.client deleted
[RESULT] write local file. Speed: 210.77 MByte/sec
[RESULT] read local file. Speed: 11217.31 MByte/sec
[RESULT] data transfer with discard at destination. Speed 27252.05 Mbit/sec
[RESULT] data transfer with write at destination. Speed 1495.48 Mbit/sec = 186.94 MByte/sec
[INFO] client terminated
```

Some options
* `-client` to launch the client, that reads and writes a file locally and then sends http requests to the server
* `-server` to launch the server, which publishes some endpoints for receiving the file from the client
* `-sync` will force a flush per slice written to file
* Specify the location of the file to be written or read with `-filename`. The default is `/tmp/simpleprofiler/profilerfile[.client|.server]`
* `-serverhost` and `-serverport` to change the server endpoint. 
* `-secure` use https instead of http for client to server communication. Self-signed certificates are automatically generated in the home
directory

# Database testing

Currently works with mysql database only

This type of test is launched if `-sqlcredentials` is specified, with the format `username:password`, in this case, also `-sqlhostport` is
required, with the format `host:port`. The credentials need administrator permissions, since writes are performed in the `mysql` database.

If no other switch is specified, an infinite loop of insertions, selects and deletions of batches of `-sqlloopsize` is performed. For
this purpose, a `test` table is created under the `mysql` database. A `+` is written to the console for each write batch, a `O` is written
per select batch and a `-` per delete batch. If `-debug` is specified, the operation rates are written also. In case of error, a message
is printed and the operations halted for one second.

This default type of testing is intended to generate some load while the database is forced into unavailabilty, to check its behaviour.

To support a similar use case but for databases that are read only (replicas), the `-sqlquery` parameter may be specified. This is be a
select sentence that will be executed in an infinite loop, sleeping for 200ms between queries.

To test the insertion performance, the `-sqlinsertthreads` switch must be specified. In this case, only this type of test is performed and
the client will exit.

# Kubernetes testing

The simpleprofiler docker image is published as `francistor/simpleprofiler:0.2`

The `descriptors` folder contains some kubernetes resources to launch a client and server pods, either with local storage or with Persistent
Volume Claims.

The `profiler.sh` script executes a full test set in which pods are first created with local storage, performance measured and then deleted
and created again with Persistent Volume Claims.

To re-generate the docker files, use `dockerBuild.sh`



