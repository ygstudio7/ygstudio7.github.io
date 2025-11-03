참고: https://www.ibm.com/docs/en/aix/7.2?topic=t-test-command



| Description                   |                                                              |
| ----------------------------- | ------------------------------------------------------------ |
| **-b** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a block special file. |
| **-c** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a character special file. |
| **-d** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a directory. |
| **-e** *FileName*             | Returns a True exit value if the specified *FileName* exists. |
| **-f** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a regular file. |
| **-g** *FileName*             | Returns a True exit value if the specified *FileName* exists and its Set Group ID bit is set. |
| **-h** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a symbolic link. |
| **-k** *FileName*             | Returns a True exit value if the specified *FileName* exists and its sticky bit is set. |
| **-L** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a symbolic link. |
| **-n** *String1*              | Returns a True exit value if the length of the *String1* variable is nonzero. |
| **-p** *FileName*             | Returns a True exit value if the specified *FileName* exists and is a named pipe (FIFO). |
| **-r** *FileName*             | Returns a True exit value if the specified *FileName* exists and is readable by the current process. |
| **-s** *FileName*             | Returns a True exit value if the specified *FileName* exists and has a size greater than 0. |
| **-t** *FileDescriptor*       | Returns a True exit value if the file with a file descriptor number of *FileDescriptor* is open and associated with a terminal. |
| **-u** *FileName*             | Returns a True exit value if the specified *FileName* exists and its Set User ID bit is set. |
| **-w** *FileName*             | Returns a True exit value if the specified *FileName* exists and the write flag is on. However, the *FileName*will not be writable on a read-only file system even if **test** indicates true. |
| **-x** *FileName*             | Returns a True exit value if the specified *FileName* exists and the execute flag is on. If the specified file exists and is a directory, the True exit value indicates that the current process has permission to search in the directory. |
| **-z** *String1*              | Returns a True exit value if the length of the *String1* variable is 0 (zero). |
| *String1***=** *String2*      | Returns a True exit value if the *String1* and *String2* variables are identical. |
| *String1***!=***String2*      | Returns a True exit value if the *String1* and *String2* variables are not identical. |
| *String1*                     | Returns a True exit value if the *String1* variable is not a null string. |
| *Integer1* **-eq** *Integer2* | Returns a True exit value if the *Integer1* and *Integer2* variables are algebraically equal. Any of the comparisons **-ne**, **-gt**, **-ge**, **-lt**, and **-le** can be used in place of **-eq**. |
| *file1* **-nt** *file2*       | True if *file1* is newer than *file2*.                       |
| *file1* **-ot** *file2*       | True if *file1* is older than *file2*.                       |
| *file1* **-ef** *file2*       | True if *file1* is another name for *file2*.                 |