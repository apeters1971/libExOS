# libExOS
This library implements large object IO on top of RADOS objects storage.
The model is to define a meta data pool which stores logical names and an erasure coded pool to store chunked payload. The library maintains an increasing inode counter which is stored
in the meta data pool as *oid=EXOS/ROOT*.

The library uses some default settings for chunking and object naming:
```
#define EXOSMANAGER_OBJECT "EXOS/ROOT"
#define EXOSMANAGER_INODE_KEY "exos.inode"
#define EXOSMANAGER_POOL_KEY "exos.pool"
#define EXOSMANAGER_SIZE_KEY "exos.size"
#define EXOSMANAGER_MTIME_KEY "exos.mtime"
#define EXOSMANAGER_XATTR_RESERVED_PREFIX "exos."
#define EXOSMANAGER_DEFAULT_BLOCKSIZE 33554432
```

The meta data OID entries store the inode, the data pool and the modification time as OMAP attributes with the given OMAP keys. The library supports automatic static read-ahead, which disabled 
once a read cannot be satisfied by the used read-ahead strategy. To improve small writes the library maintains a write-back buffer to aggregate small aio_writes into the given chunk size.

An example how to use the library:
```
exosfile exosf("/eos/file.aioseqwritetruncate","rados.md=mdpool_replicated&rados.data=datapool_ec&rados.user=username");
char* buffer = (char*) malloc(128*1024*1024);
exosf.open(O_CREAT);
for (size_t i=0; i< 1024; ++i)
{
  exosf.aio_write(buffer,i* 1024*1024, 1024*1024);
}
free(buffer);
exosf.unlink();
```

The supported CGI variables are:
````
rados.md=<poolname>
rados.data=<poolname>
rados.user=<username>
rados.config=<configfilename>

When using the library for writing, you have to take care to avoid parallel writers using the *lock* interface.

Please be aware that the library is currently only in a *proof-of-concept* state.



