
If you are receiving this error it may due to a poorly configured environment.
<http://gcc-help.gcc.gnu.narkive.com/8AbbmxJN/specs-directory-interferes-with-the-operation-of-gcc-in-configure-scripts>

## Summary

Pulldown-cmark has a toplevel directory named specs. A poorly configured *LIBRARY\_PATH*, *LPATH* or *GCC\_EXEC\_PREFIX* may cause gcc to attempt to read this directory as a file.
Test to verify that this is the cause
Do this anywhere on your system.

```bash
$ mkdir -p test/specs
$ cd test
$ gcc --version
gcc: error: ./specs: Is a directory
```

(in a correctly-configured environment, this would, of course, simply print the version)

## Solution

Ensure that the current directory is not listed in *LIBRARY\_PATH*, *LPATH*, or *GCC\_EXEC\_PREFIX*.
Note that an empty entry is interpreted as the current directory, because that's how POSIX defined PATH, and POSIX hates you. This means that these variables must not begin or end with :, and must not contain :: anywhere in the middle.
In my experience, this can easily occur as a result of the common pattern of using

```bash
export VAR=some/new/path:$VAR
```

to append paths to an environment variable without accidentally clobbering a pre-existing value. Needless to say, the very first time this is done it leaves an empty component at the end.
My suggestion is that you pick a file that you own where you modify the variable, and simply add a line to delete empty components after the fact. (this only needs to be done once, and as long as it is done at some point after the first path you add to the variable, it will work equally well regardless of whether VAR existed prior to the script)

```bash
in e.g. .bashrc

export VAR=some/new/path:$VAR
export VAR=some/other/path:$VAR

# delete any possible empty components
VAR=$(echo $VAR | sed -E -e 's/^:*//' -e 's/:*$//' -e 's/:+/:/g')
```

总结：环境变量中不要带空的
