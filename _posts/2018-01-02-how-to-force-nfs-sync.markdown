---
layout: post
title:  "How to Force NFS Sync"
date:   2017-01-02 18:42:15 -0500
categories: [snippets]
---

At work, I'm forced to develop on a remote linux box and run windows locally. My files are
synchronized through an NFS mount. I ran into issues with
[webpack-dev-server](https://github.com/webpack/webpack-dev-server) not seeing my updates even
with [watchOptions.poll](https://webpack.js.org/configuration/watch/#watchoptions-poll). It turned
out that my files were not being synchronized over the NFS share as often as I had hoped.

Unfortunately, I don't have the privileges to modify my NFS
mount options (if you do, this [Stack Overflow post](https://unix.stackexchange.com/a/23425) is
probably what you want). For the rest of us, we'll have to abuse the rules of the Linux NFS
client.

> Note that only open and fopen need to guarantee that they get a consistent handle to a
> particular file for reading and writing. stat and friends are not required to retrieve fresh
> attributes, in fact. Thus, for the sake of close-to-open cache coherence, only open and fopen
> are considered an "open event" where fresh attributes need to be fetched immediately from the
> server.
>
> -- Chuck Lever, [Close-To-Open Cache Consistency in the Linux NFS Client](https://goo.gl/mRYbHj)

Since `cat` naturally uses `fopen()`, the below shell script will force NFS sync.

```bash
while [ true ]; do sleep 1 && find ./ -type f -exec cat {} \; >/dev/null ; done
```

Inelegant, though it works when your hands are tied or you're too lazy to tweak your NFS mount
configuration.

For me, this method traverses over around 3000 files per second. Adjust the sleep as needed for
your project.

If you're looking to perfect your bash skills, I recommend checking out
[The Linux Command Line: A Complete Introduction](http://amzn.to/2CGdnhM).
