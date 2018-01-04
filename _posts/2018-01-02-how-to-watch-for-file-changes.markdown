---
layout: post
title:  "How To Watch For File Changes On Linux"
date:   2017-01-03 19:50:11 -0700
categories: [snippets]
---

If you're trying to perform a particular action whenever a file changes in a directory,
`inotifywait` is likely the tool you're looking for.

On Ubuntu, you can get `inotifywait` by installing `inotify-tools`.

```bash
sudo apt-get install inotify-tools
```

Below is an example bash script utilizing `inotifywait` to run `elm-make` on any changed file
the current directory.


```bash
#!/bin/bash
set -ouv

# Change the following to suit your needs.
EXTENSION="\.elm\$"
ACTION="elm-make"

inotifywait \
    -r \
    -q \
    -m \
    -e move,modify,create \
    --format '%w%f' \
    ./ | while read file; do
    echo ${file} | grep -q ${EXTENSION} # Check that the extension matched.
    if [ $? -eq 0 ]; then
        ${ACTION} ${file} # Perform the action.
    fi
done
```

The following flags are employed:

| Flag       | Description                                                                 |
| -----------|:----------------------------------------------------------------------------|
| `-r`       | Recursively search for changes in the provided directory.                   |
| `-q`       | Don't output anything beyond the file name that we're capturing.            |
| `-m`       | Monitor for file changes, don't quit after the first change.                |
| `-e`       | List of events to watch for. You need `move` if you're using intellij.`*`   |
| `--format` | `%w%f` is the formatter used to grab either the file or directory modified. |

<br />

<small>
`*` The reason you need `move` if you're using intellij (or likey other editors as well) is that
intellij doesn't modify the files in place. It writes them to a temporary directory and then
copies the file to the destination once you save. This counts as a `move`, not a `modify`.
</small>

I hope this helped!
