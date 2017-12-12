# Outline of this workshop

In this workshop we will build three of the libraries that are needed to
build a new version of the GNU compilers.  We got their name from the
source code of the compilers.  It will often be the case that some
library will not be of interest, per se, but will be required by something
that is.

The libraries that we need are:  MPC, MPFR, and GMP

This tour will be a little artificial, but all the steps that we would
normally take to install software on the cluster for you are included.
Once you have finished this workshop, you can compete for our job!

## Where to install your software

We recommend that you install your software into a directory called
`local` in your home directory.  This is the closest you are likely
to get to `/usr/local` which is the default location, and choosing
this name will make reading documentation easier.  Some software may
want to install instead into `/opt`, in which case, you can create
an `/opt` directory in your home.  These examples will use `$HOME/local`.

Under `$HOME/local` create a directories called `src` and `build`
and the change to the `src` directory.  This is where we will download
the source code.
```
$ mkdir -p local/src
$ cd src
```
## Getting the source code

It's often a good bet to search for `<package_name> download source` to
find where to get the source code for your software.  The following were
found be searching for `mpfr download source`, `mpc download source`, and
`gmp download source`, respectively.

```
$ wget http://www.mpfr.org/mpfr-current/mpfr-3.1.6.tar.gz
$ wget ftp://ftp.gnu.org/gnu/mpc/mpc-1.0.3.tar.gz
$ wget https://ftp.gnu.org/gnu/gmp/gmp-6.1.2.tar.bz2
```

The GNU people are wildly impressed with their own cleverness and have made
the default compression program a non-standard format rather than using
their very own `gzip`.  We fished around and found a more standard, `bzip2`
format source code.  If that were not available, then you might first have
to compile the `lzip` package.  OK, that's enough venting.  On with the show!

## Looking for dependencies

All software has something else on which it depends.  Finding those
dependencies can sometimes be frustrating.  Most software will come with
a file called `INSTALL`.  The archives are uncompressed and expanded using
the `tar` command.  For `.tar.gz` or `.tgz` files you will use the `z`
option to `tar`.  The `t` option says to just get the names but not to
actually expand, the `v` option says 'be verbose', i.e., print the names,
and the `f` option must be _immediately_ followed by the name of the file
containing the archive.

For `bzip2` files, it is almost the same, but use a `j` instead of a `z`.

We want to look for the `INSTALL` files, so we can do something like this.

```
$ tar tzvf mpfr-3.1.6.tar.gz | grep INSTALL
-rw-r--r-- 1000/1000     24769 2016-12-31 20:39 mpfr-3.1.6/INSTALL

$ tar tzvf mpc-1.0.3.tar.gz | grep INSTALL
-rw-r--r-- 1001/1001      3362 2015-02-16 07:37 mpc-1.0.3/INSTALL
```
```
$ tar tjvf gmp-6.1.2.tar.bz2 | grep INSTALL
-rw-r--r-- tege/wheel     2489 2016-12-16 10:45 gmp-6.1.2/INSTALL
-rw-r--r-- tege/wheel     9220 2016-12-16 10:45 gmp-6.1.2/INSTALL.autoconf
-rw-r--r-- tege/wheel     2435 2016-12-16 10:45 gmp-6.1.2/demos/perl/INSTALL
```
We will assume that `gmp-6.1.2/INSTALL` is the one we really want, so
we can remove the `t` option and replace it with the `x` option to extract.
```
$ tar xzvf mpfr-3.1.6.tar.gz mpfr-3.1.6/INSTALL
mpfr-3.1.6/INSTALL

$ tar xzvf mpc-1.0.3.tar.gz mpc-1.0.3/INSTALL
mpc-1.0.3/INSTALL

$ tar xjvf gmp-6.1.2.tar.bz2 gmp-6.1.2/INSTALL
gmp-6.1.2/INSTALL
```
Using `less`, we look in the `INSTALL` file for MPC, and we find that it says
```
0. You first need to install GMP, the GNU Multiprecision Arithmetic Library,
   see <http://gmplib.org/>, and GNU MPFR, see <http://www.mpfr.org>.
   GNU MPC requires GMP version 4.3.2 or later
   and GNU MPFR version 2.4.2 or later.
```
Ah, Hah!  There, we saved ourselves some kind of time by noting that this has
additional prerequisites.

On to MPFR, which says,
```
0. You first need to install GMP. See <http://www.gnu.org/software/gmp/>.
   MPFR requires GMP version 4.1 or later.
```
So, looks like GMP has to be done first, then MPFR, and then MPC.  So, let's
get started.

## Extracting and configuring your software

We saw above how to extract single files from an archive, to extract all
files, we just leave off the filename from the `tar` command.

We highly recommend that you check the available space in the `/tmp` directory,
```
$ df -h /tmp
```
and if there is plenty of room, expand and build your software there.  This will
help prevent problems with space in your home directory.
```
$ mkdir -p -m 700 /tmp/$USER/build
$ cd /tmp/$USER/build
$ tar xjvf $HOME/local/src/gmp-6.1.2.tar.bz2
```
(The `-m 700` is a nicety that sets permissions on that new directory so only
you can read its contents.)

That will create a `gmp-6.1.2` directory, and we go there next.
```
$ cd gmp-6.1.2
$ ls -l configure
-rwxr-xr-x 1 grundoon hpcstaff 945400 Dec 16  2016 configure
```
That `configure` program will run a bunch of checks on your computer, and
then set appropriate (it is hoped) options for the computer you have and
the other software you have installed.  It's always a good idea, if
somewhat daunting, to at least _look_ at the options, which can be found
with
```
$ ./configure --help | less
```
Paging through the ones for GMP, there are a couple that might bear
further investigation:  `--with-cxx` (C++ is kinda popular),
`--enable-profiling` (that might help if you were trying to make things
go faster).  The one we _know_ we want is `--prefix`, which says where
to install the compiled software.  Let's just do the defaults for this
first time around, and that will look like this.
```
$ ./configure --prefix=$HOME/local
. . . . [ lots of output ] . . . .
configure: summary of build options:

  Version:           GNU MP 6.1.2
  Host type:         westmere-pc-linux-gnu
  ABI:               64
  Install prefix:    /home/bennet/local
  Compiler:          gcc -std=gnu99
  Static libraries:  yes
  Shared libraries:  yes
```
Not all packages will print that nice summary, though, so you should
look in `config.log`, which will almost always have the actual
`configure` command you used.  We recommend that you put that into
a file and keep it with the installed software, in case you ever need
to know what you used.

The  `configure` program creates a `Makefile`, so you can now run
`make`, which will do what says.  You can save substantial time by
running `make` in parallel:  It will try to compile several things
at once.  We recommend that you stick to four parallel compilations.
```
$ make -j 4
. . . . [ lots of output ] . . . .
```
When I ran this on a lightly used machine, it took about 29 seconds
with four processes.  Using only one, it took about 1 minute 30
seconds.

Again, most software will come with some kind of `test` or `check`
option for `make`.  You should always run it.  It may fail, and that
may be important, or not.  But if the tests/checks pass, then you
can feel a little more confident.  Mais, pas trop confiant!  So!
```
$ make test
make: *** No rule to make target `test'.  Stop.
$ make check
. . . . [ lots of output ] . . . .
```
You should see a bunch of impression looking compilations followed
by some lines that look something like
```
PASS: t-sqrtrem
```
Finally, it will end, and if it doesn't have any errors printed, and
the return code is 0, then you've passed.  This will print the return
code from `make`.
```
$ echo $?
```
Now you need to install it,
```
$ make install
. . . . [ lots of output ] . . . .
```
That will create a bunch of new directories under `$HOME/local`.
```
$ ls $HOME/local
include  lib  share  src
```

If you scroll up from the bottom of the output, you'll see something
like this (for this package, anyway).
```
Libraries have been installed in:
   /home/bennet/local/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the '-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the 'LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the 'LD_RUN_PATH' environment variable
     during linking
   - use the '-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to '/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
```

## Compiling a test program

So, we have it installed, but now we need to take note of what the
installation told us about how to use it, and there are several
choices.  To see whether (!) and how they work, we need to have
a sample program.  Sometimes you'll get lucky and find samples
in the software documentation, maybe a tutorial section.  Other
times you are force to ask the Goog: "sample programs for gmp".



We have one for you, though:  [gmp_sample.c](./gmp_sample.c)

We are going to compile it a couple of different ways.

### Dependent on runtime environment

This is a program in C, so our chosen compiler is `gcc`.  If there
were no external libraries needed, we would compile with
```
$ gcc -o factorial factorial.c 
/tmp/cctCGE1w.o: In function `fact':
factorial.c:(.text+0x18): undefined reference to `__gmpz_init_set_ui'
factorial.c:(.text+0x3a): undefined reference to `__gmpz_mul_ui'
factorial.c:(.text+0x77): undefined reference to `__gmpz_out_str'
factorial.c:(.text+0x83): undefined reference to `__gmpz_clear'
collect2: error: ld returned 1 exit status
```
but there are external dependencies.  It's complaining that it can't
find the functions in GMP, and that's because we didn't tell it to
use that library.  Let's try again.
```
$ gcc -o factorial -l gmp factorial.c
$ ldd factorial
	linux-vdso.so.1 =>  (0x00007ffd1c992000)
	libgmp.so.10 => /lib64/libgmp.so.10 (0x00002acdae737000)
	libc.so.6 => /lib64/libc.so.6 (0x00002acdae9ae000)
	/lib64/ld-linux-x86-64.so.2 (0x00002acdae514000)
```
That went more weller, so we run the `ldd`, which will list the references
to external libraries in the program we compiled, and, oh, dear, it's
not finding the version of `libgmp.so` that we just compiled!  Instead
it found the one on the system.
```
libgmp.so.10 => /lib64/libgmp.so.10
```
How do we fix that?  We have to do that in two steps.  First, we have to
give it a search path for the library when we compile it, then we have
to give it a path to look in when we run it.  The `LD_LIBRARY_PATH` variable
is used to instruct programs to look in a set of directories for libraries
before trying to hunt them up on the system.  You can put a colon-separated
list of directories in that variable.  If it is empty, then
```
$ echo $LD_LIBRARY_PATH
$ export LD_LIBRARY_PATH=$HOME/local/lib
```
If there were something in it already, then you would use
```
$ export LD_LIBRARY_PATH=$HOME/local/lib:$LD_LIBRARY_PATH
```
which adds `$HOME/local/lib` to the front of what's there. Now,
we need to tell the compiler where to look when compiling, and do that with
the `-L` option.

***BIG WARNING***

The `-L` must precede the `-l`.  That is the _where_ specification has to
come before the _what_ specification.  The big ell says where, the little
ell says what.  So,
```
$ gcc -o factorial -L $HOME/local/lib -l gmp factorial.c
$ ldd factorial
	linux-vdso.so.1 =>  (0x00007ffe0639e000)
	libgmp.so.10 => /home/grundoon/local/lib/libgmp.so.10 (0x00002b3fb43bb000)
	libc.so.6 => /lib64/libc.so.6 (0x00002b3fb4631000)
	/lib64/ld-linux-x86-64.so.2 (0x00002b3fb4198000)
$ ./factorial 128
128!  =  385620482362580421735677065923463640617493109590223590278828403276373402575165543560686168588507361534030051833058916347592172932262498857766114955245039357760034644709279247692495585280000000000000000000000000000000

```
The kink in all this is that you somehow have to contrive to always
set `LD_LIBRARY_PATH` every time you want to run the software, otherwise
it will fail because it can't find the library, or it will find the wrong
version.

Why is that?

When compiling this way, the compiler stores only the name of the library.  We
can see this (sometimes) by using
```
$ strings factorial | grep libgmp
libgmp.so.10
```

### Passing the run path to linker via environment variable

If we set the _library run path_ variable, `LD_RUN_PATH` before we compile
our software, that directory will get added to the compiled program. Let's
see.
```
$ export LD_RUN_PATH=$HOME/local/lib
$ gcc factorial -l gmp factorial.c
$ strings factorial | egrep "$USER|libgmp"
libgmp.so.10
/home/grundoon/local/lib

$ unset LD_RUN_PATH
$ strings factorial | egrep "$USER|libgmp"
libgmp.so.10
/home/bennet/local/lib
$ ./factorial 32
32!  =  263130836933693530167218012160000000
```
That's significantly better because we only have to remember to set
the environment variable before we compile the software.

### Passing the run path via compiler option

The final method, as told to us by the installation is to use an
option that tells the compiler to add the runpath to the program
explicitly.  With this method, there are no environment variables
needed at all.
```
$ gcc -Wl,-rpath -Wl,$HOME/local/lib -l gmp factorial.c
$ strings factorial | egrep "$USER|libgmp" 
libgmp.so.10
/home/grundoon/local/lib
$ ./factorial 32
32!  =  263130836933693530167218012160000000
```
What's that doing?  When you compile a program, a lot of things
go on behind the scenes.  The compilation occurs, with the code
you are interested in, but there is a lot of other code lying
about -- stuff that understands how to print to the terminal,
or access a disk, for example, that needs to be included.  So,
your bit gets turned into _object code_, and the all the object
codes get passed to the _linker_, which glues them all together.

The `-Wl,` says to pass along whatever follows it as an instruction
to the linker.  We have two in a row, so when the linker runs, it
will get `-rpath $HOME/local/lib` passed to it, and that tells it
to put that directory into the finished program as a place to look
for libraries before looking elsewhere.

Whew!

### Which is the _right_ way?

There is no "right" way, just different ways.  How to choose?

I think that using the last way -- passing the linker the run path
-- may be the most likely to lead to long term happiness.  There
is nothing hidden from you that might get forgotten later.  It's
right there in the notes you keep about how to build this software,
notes that you put into a `Makefile`, riiiiiight?

We've now learned a lot of the things that we will need to make
a library that depends on another library, to which we turn next.
