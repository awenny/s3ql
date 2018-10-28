============================================
 # 46 - Support concurrent read-only mounts
============================================

The idea behind this enhancement is the ability to mount the same file system
multiple times, but for read only.
While S3QL does not allow concurrent mounts in write mode, having one mount
able to change the file system, we could have others for read only. Necessary
to regularly download the metadata and replacing the local metadata.


Definition
==========

RW client = client holding writeable mount (only one possible)
RO client = client holding read-only mount (many possible)


Requirements
============

The RW client must upload the metadata more often, than if "alone" connected to
the mounted file system. (once in 24 h is probably not suitable anymore)

The metadata must be replaced in the RO client while being mounted.

Downloading the metadata regularly without knowing it has changed in the
backend is not an option. This will probably cause extensive transfer costs,
if it is mounted outside the datacenter. Costs do apply for eg. Amazon S3, if
data is transferred externally to the datacenter. The transfer costs depend on
size of metadata, number of read only clients and interval.


Possible issues
===============

The RO client will only see changes in the file system, if the changed metadata
has been pushed to the backend.

This leads us to the question, how do changes to data effect the RO client
before the client downloads the new metadata?

1. A new file is created with new blocks, the RO client will see nothing
2. A file is deleted, will the data blocks get deleted immediately, so the
   RO client will see data which does not exist anymore? -> dirty metadata
3. A file is changed, are the changed blocks overwritten, or created as new
   blocks and the replaced block deleted?


Nice to have
============

It would be great, if the RO clients could be notified somehow, if the metadata
has changed in the backend and being ready for download.
