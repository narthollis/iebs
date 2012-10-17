# IEBS Specification #
Orignal https://docs.google.com/document/d/1Sj2y3bLzxvY8wrxKMiozvA26_DRxXepsVthzzIReRnI/edit

## Requirements ##
* Encrypt Backup-ed Files
* Produce Incremental backups without needing to decrypt existing backup-ed files
* Prefer to not require decryption key to encrypt files (openssl? gnupg?)
* Able to be easy uploaded into a non-traditional file system (Eg. RackSpace Cloud Files) (ie. don't rely on the existance of a traditional file system on the remote host)

## Ideas ##

### Filesystem Layout ###
File blocks are stored in a tar file (possibly compressed? Optional? Job of the output engine?) - this will ensure that we have something akin to a traditional file system to utilize.

### Full backups ###
Force 1 full backup for no more than 50 increments ?
```
system_yyyy-mm-dd_full.files.tar
system_yyyy-mm-dd_full.manifest.bz2
```

### Backups Increments ###
The first date in this list referes to the full backup. The second data (with time) is used to uniquly identify this backup.
```
system_yyyy-mm-dd_increment_yyyy-mm-dd-hh-mm.files.tar
system_yyyy-mm-dd_increment_yyyy-mm-dd-hh-mm.manifest.bz2
```

### Backup Internal Format ###
Internal format is the first 2 characters of the UUID as a directory containing the UUID of a file as as a folder, then the blocks are stored in folders with 10,000 blocks per folder.
```
55/550e8400-e29b-41d4-a716-446655440000/00000/00000311
a5/a5413a39-1531-ee54-2944-592522344224/00001/00001325
e8/e877ce98-ce99-8a90-ece0-8e9ae0ce90a8/00003/00003214
```
The internal format of both full and incremental backups is identical, it is the contents of the files and the manifests that differ

## Full backup Manifest Layout ##

### Archive Lines ###
The first line of the manifest should point to the file containing all of the files listed in the manifest followed by a time stamp (RFC3339 format) for the manifest and the FQDL of system that manifest is created for eg:
```
ka-dargo/2010-02-11_full.files.tar 2010-02-11T12:29:12+10:30 ka-dargo.moya.narthollis.net
```
The next line should be the sha384 of the archive file
```
sha384(d063457705d66d6f016e4cdd747db3af8d70ebfd36badd63de6c8ca4a9d8bfb5d874e7fbd750aa804dcaddae7eeef51e)
```
The archive lines are terminated by a blank line

### File Lines ###
All files in the archive are broken into blocks - to furthur reduce the size of incrimental backups.
The first line for each file describes the file and provides the identifiers. The fields used are: UUID, which is used to identify the file in the archive; File Path, which is how we remember the file name and location; File Size in bytes, this is used in part for integrity checks and in part for any program that lists the files in the archive for indervidual retrevial; Modification Date, which is used only for human retreival reasons; Owner, used for human retreival reasons; Block Size, used to record the block size decided on for this file. Order is important and all fields are seperated by a white space character, filenames sould be surrounded in quotes.
 
The next lines record the information on the indervidual blocks for that file: Block index, starting at 0; Block size, because not all blocks are going to be the full size of the block (ie the last one); sha384 hash in the format hashtype(hexdigest).
 
The file block is be concluded with '-- END OF FILE --' and then the sha1 and md5 sum of all the manifest information for the file (excluding -- END OF FILE –)
```
550e8400-e29b-41d4-a716-446655440000 'Documents/rfc/iebs.odt' 9842 2010-02-11T12:20:45+10:30 narthollis 1024
0 1024 sha384(5f91550edb03f0bb8917da57f0f8818976f5da971307b7ee4886bb951c4891a1f16f840dae8f655aa5df718884ebc15b)
1 1024 sha384(47f05d367b0c32e438fb63e6cf4a5f35c2aa2f90dc7543f8a41a0f95ce8a40a313ab5cf36134a2068c4c969cb50db776)
2 1024 sha384(d063457705d66d6f016e4cdd747db3af8d70ebfd36badd63de6c8ca4a9d8bfb5d874e7fbd750aa804dcaddae7eeef51e)
3 1024 sha384(6af11c83586822c3c74bb3ccef728bae5cfee67cad82dd7402711e530bec782fc02aff273569d22ddffb3b145f343768)
4 1024 sha384(bcf6e0ebc8c5185372c8d8f0f449727bd52c5284860f1c46ea53dc943fabf40c75067629813f12bd994d75a39f44843d)
5 1024 sha384(1861ebe932dc65c5f04d33f6972b13acc8b1e572344016bf3cc950f60bfad6fdc0e32f0318e8bba57cf756eac0a49fce)
6 1024 sha384(f99c53a03fdd8cbfc806c80d467f488bc7cd7003a8af08a46ca34928aa9241b6cd090353e325d30575b72b03cea996c2)
7 1024 sha384(c2b14cbf3fcb739683383b96157f765c8629dd2415ceb3b277b5f6028e8bb3c8cc5408be8a88254907c6ebb1cd4f1827)
8 626 sha384(5554573d20feb83c21707bc9f346b718a5d0ad55156a1a8cb74ca7263d3f01221aeb0d20305a0aa0793bf0b588de684f)
-- END OF FILE -- sha1(e3dee672b061cc4d47dc42db6d40603cdf640376) md5(0eb41fb024a8bab34905e74b93ebe4cd)
```

### Close Line ###
The manifest should be closed with the -- END OF MANIFEST -- followed by the sha1 and md5 hash of all the above text (excluding -- END OF MANIFEST --)

```
-- END OF MANIFEST -- sha1(7fe84f2db6df95dde3bfbd1e17b14efd8173941c) md5(54e0a7b53091f245e4bb4cdece63c035)
```

### Incremental backup Manifest Format ###

## Archive Lines ##
Incremental backup manifest files include all the information that a full backup manifest as well as the path to the manifest file of the full backup.
```
ka-dargo_2010-02-11_increment_2010-02-12-12-00.files.tar 2010-02-12T12:00:35+10:30 ka-dargo.moya.narthollis.net ka-dargo/2010-02-11_full.manifest.bz2
```
The next line should be the sha384 of the archive file
```
sha384(d063457705d66d6f016e4cdd747db3af8d70ebfd36badd63de6c8ca4a9d8bfb5d874e7fbd750aa804dcaddae7eeef51e)
```
The archive lines are terminated by a blank line

## File Lines ##
Incremental backups only store the changed blocks of changed files and the manifest reflects this, they will only store the information for the changed files and blocks.

```
550e8400-e29b-41d4-a716-446655440000 'Documents/rfc/iebs.odt' 9842 2010-02-11T12:20:45+10:30 narthollis 1024
5 1024 sha384(1861ebe932dc65c5f04d33f6972b13acc8b1e572344016bf3cc950f60bfad6fdc0e32f0318e8bba57cf756eac0a49fce)
-- END OF FILE -- sha1(e3dee672b061cc4d47dc42db6d40603cdf640376) md5(0eb41fb024a8bab34905e74b93ebe4cd)
```

## Close line ##
The Close line is identical in both the incremental and full backups

