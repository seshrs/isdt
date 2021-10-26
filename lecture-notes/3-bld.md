---
---

# Lecture Notes: Build Systems

## Lecture 2

### Targets and rules

*Targets* are the core of a Makefile. They are the results you need to
eventually build; the names of the files on disk you want to see by the end of
the build process[^phony]. In the example below, a Makefile has a target called
`mybinary` -- the name preceding the colon (`:`).

[^phony]: Not always; sometimes there are "phony" targets that are names for
    batches of rules and do not produce files.

```
mybinary:
    gcc main.c
    mv a.out mybinary
```

It also has *rules* to build that target: `gcc main.c`. The rules section
consists of a list of shell commands that are run in
succession[^different-shells] to build the target. In this case, the two rules
-- `gcc main.c` and `mv a.out mybinary` will run one after another.

[^different-shells]: They are run in different shell processes, so state like
    variables and working directory do not persist between commands.

This is a bit of a silly example since `gcc` has a `-o` option to specify the
output filename, but there are certainly situations where multiple shell
commands might be necessary.

### Dependency relations

Let's look again at the Makefile from above:

```
mybinary:
    gcc main.c
    mv a.out mybinary
```

We can run this once with `make mybinary` and then we will have a `mybinary` on
disk. During the course of development, though, we will likely make a change to
`main.c` and try to rebuild with `make mybinary`. Make will give an unhelpful
response and do nothing:

```
$ make mybinary
gcc main.c
mv a.out mybinary
$ vim main.c
...
$ make mybinary
make: 'mybinary' is up to date.
$
```

This is because there is no way for Make to know about the implicit
relationship between `mybinary` and `main.c`. As far as Make is concerned, the
contents of the rules are opaque shell commands; it does not attempt to
introspect on them, nor does it have an innate idea of what `gcc` means. You,
the programmer, have to specify the relationship manually.

In order to instruct Make to rebuild `mybinary` when `main.c` is modified, add
`main.c` as a *dependency* of `mybinary`:

```
mybinary: main.c
    gcc main.c
    mv a.out mybinary
```

Now Make will look at the <dfn><abbr title="modification time">m-time</abbr></dfn>
of both `main.c` and `mybinary`. If `main.c` is newer, it will rebuild
`mybinary`.

It is possible to specify more than one dependency per target. If `main.c` also
included a header, `myheader.h`, you should add `myheader.h` to the
dependencies for `main.c`. Otherwise the binary and the header will be out of
sync. For libraries use headers to specify interfaces, this is bad. At best,
you might get a wrong answer. At worst, a crash or memory corruption.

### Notes on split compilation (object files)

For languages like C that have a notion of split compilation, Make is even more
useful. If you have, say, 10 C files that all need to be compiled together, it
is possible to run:

```
$ gcc file0.c file1.c ... file9.c -o mybinary
$
```

This will compile each file and then link the results together at the end into
`mybinary`.

Unfortunately, this compilation process will throw all of the intermediate
results away every time. If you only change `file2.c` and nothing else, you
will still end up building all of the other C files again[^c-vs-header]. For
this reason, it is possible to compile each file into a corresponding *object
file* and then link those together:

[^c-vs-header]: TODO note here about changing C files vs header files

```
$ gcc -c file0.c file1.c ... file9.c
$ gcc file0.o file1.o ... file9.o -o mybinary
$
```

Now if you change `file2.c`, you need only re-run `gcc -c file2.c` and the
linking step, which together should be much faster than recompiling everything.

It's hard to keep track of this manually, so we can build Make rules to handle
this for us:

```
mybinary: file0.o file1.o file2.o  # and so on
    gcc file0.o file1.o file2.o -o mybinary

file0.o: file0.c
    gcc -c file0.c

file1.o: file1.c
    gcc -c file1.c

file2.o: file2.c
    gcc -c file2.c
```

Now it is possible to modify any one C file and have the binary rebuilt
automatically with the least amount of steps.

You will notice that all of this typing is getting cumbersome. We will talk
about a solution to this repetition later!

TODO: Maybe notes on Rust here?

### The dependency graph

a DAG, just like Git uses!

```
┌───────────────────┐      
│mybinary           │      
└┬────────┬────────┬┘      
┌▽──────┐┌▽──────┐┌▽──────┐
│file0.o││file1.o││file2.o│
└┬──────┘└┬──────┘└┬──────┘
┌▽──────┐┌▽──────┐┌▽──────┐
│file0.c││file1.c││file2.c│
└───────┘└───────┘└───────┘
```

```
$ cat Makefile
mybinary: mybinary
	touch mybinary
$ make mybinary
make: Circular mybinary <- mybinary dependency dropped.
$
```

### What happens when you type `make`

* Make reads `Makefile`
* determines target(s) to execute
* builds DAG/topo sort
* execute in order (potentially in parallel)

### Why is a Makefile better than a shell script?

As demonstrated, it's possible to build your projects using a simple shell
script that builds every file every time. This is a fine solution, to a point;
if the number of files or the compilation time for individual files grows, your
build script will get slower over time.

You might get fancy and add some features to compare m-times in your shell
script. Maybe you add a function called `build_if_newer` and get 60% of the
functionality of Make. This will work. But now you have to reason about an
ever-growing shell script and if it perfectly implements your ideal build
semantics. And Make will do it better still -- Make already has built-in
parallelism. Does your shell script?

Make has been around a long time and its performance and correctness are
well understood. Its interface is well understood, too; people who know Make
are at home in most other projects that use Make. Not so for a custom shell
script.
