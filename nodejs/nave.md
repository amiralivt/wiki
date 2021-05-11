# Virtual Environments for Node

## Install nave
```
npm install -g nave
```

## Usage
To use a version of node, you do this:
```
nave use <some version>
```

If you want to name a virtual env, you can do this:

```
nave use <some name>
```

If that virtual env doesn't already exist, it'll prompt you to choose
a version.

Both of these commands drop you into a subshell.  Exit the shell with
`exit` or `^D` to go back from whence you came.

Here's the full usage statement:

```
Usage: nave <cmd>

Commands:

install <version>     Install the version specified (ex: 12.8.0)
install <name> <ver>  Install the version as a named env
use <version>         Enter a subshell where <version> is being used
use <ver> <program>   Enter a subshell, and run "<program>", then exit
use <name> <ver>      Create a named env, using the specified version.
                      If the name already exists, but the version differs,
                      then it will update the link.
usemain <version>     Install in /usr/local/bin (ie, use as your main nodejs)
clean <version>       Delete the source code for <version>
uninstall <version>   Delete the install for <version>
ls                    List versions currently installed
ls-remote             List remote node versions
ls-all                List remote and local node versions
latest                Show the most recent dist version
cache                 Clear or view the cache
help                  Output help information
auto                  Find a .naverc and then be in that env
auto <dir>            cd into <dir>, then find a .naverc, and be in that env
auto <dir> <cmd>      cd into <dir>, then find a .naverc, and run a command
                      in that env
get <variable>        Print out various nave config values.
exit                  Unset all the NAVE environs (use with 'exec')

Version Strings:
Any command that calls for a version can be provided any of the
following "version-ish" identifies:

- x.y.z       A specific SemVer tuple
- x.y         Major and minor version number
- x           Just a major version number
- lts         The most recent LTS (long-term support) node version
- lts/<name>  The latest in a named LTS set. (argon, boron, etc.)
- lts/*       Same as just "lts"
- latest      The most recent (non-LTS) version
- stable      Backwards-compatible alias for "lts".

To exit a nave subshell, type 'exit' or press ^D.
To run nave *without* a subshell, do 'exec nave use <version>'.
To clear the settings from a nave env, use 'exec nave exit'
```

