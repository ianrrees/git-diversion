A git Diversion
===

When a file is stored in git, the contents of the file are stored in a compressed _blob_, a _tree_ associates that blob with the file name, and each _commit_ refers to a tree.
This scheme means that making a new commit only creates new blobs for any files that are added or changed (and if there isn't already a blob with the new contents in the repo).
Similarly, if multiple files in the same tree have the same contents, git will make multiple entries in that tree object referring to the same single blob.

This clever design means that one could construct a repository containing a whole bunch of references to a single blob object - the size of such a repo by itself wouldn't need to be much bigger than the compressed blob, but a checkout would be the uncompressed size of that blob, multiplied by "a whole bunch".

This is just such a repo; a clone made with `git clone --bare https://github.com/ianrrees/compression.git` will be about 200kB, but omitting the `--bare` takes about 4142568kB (4GB).
Making this was a fun learning experience; it surprised me how simple it was to do following [Git Internals - Git Objects](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects), plus a few basic git commands.

This was constructed like:

```
$ git init
$ echo 'Are we there yet?' | git hash-object -w --stdin
69a24f669025ae4645f27a8313d8c019a82a3a4f
$ for ((i=1;i<=100;i++)); do git update-index --add --cacheinfo 100644 69a24f669025ae4645f27a8313d8c019a82a3a4f file_$i.txt; done
$ git write-tree
2439649cb3063bc7da817ad7f2e6f690ba7f5802
$ git reset
$ for ((i=1;i<=100;i++)); do git read-tree --prefix dir_$i 2439649cb3063bc7da817ad7f2e6f690ba7f5802; done
$ git write-tree
6816aab64cf420b9de77bcf61252f6df2510012d
$ git reset
$ for ((i=1;i<=100;i++)); do git read-tree --prefix dir_$i 6816aab64cf420b9de77bcf61252f6df2510012d; done
$ git write-tree
d6b6aa80750350014c41199041e0e1985b7feaf0
$ git reset
$ git read-tree --prefix a_whole_bunch d6b6aa80750350014c41199041e0e1985b7feaf0
$ git write-tree
455da9a2e21ac6665c62b3952e7fb31c1f87215f
$ echo 'initial commit' | git commit-tree 455da9a2e21ac6665c62b3952e7fb31c1f87215f
deb2cc6ace18363f909da9badbea61a438aaa438
$ git branch master deb2cc6ace18363f909da9badbea61a438aaa438
```
